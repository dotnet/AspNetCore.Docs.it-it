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
ms.openlocfilehash: 378c74c930f6e3f9565f408a2761a8ed867450d3
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551975"
---
## <a name="share-interop-code-in-a-class-library"></a>Condividere il codice di interoperabilità in una libreria di classi

Il codice di interoperabilità JS può essere incluso in una libreria di classi, che consente di condividere il codice in un pacchetto NuGet.

La libreria di classi gestisce l'incorporamento delle risorse JavaScript nell'assembly compilato. I file JavaScript vengono inseriti nella `wwwroot` cartella. Gli strumenti si occupano di incorporare le risorse quando la libreria viene compilata.

Nel file di progetto dell'app viene fatto riferimento al pacchetto NuGet compilato nello stesso modo in cui viene fatto riferimento a un pacchetto NuGet. Dopo il ripristino del pacchetto, il codice dell'app può chiamare JavaScript come se fosse C#.

Per altre informazioni, vedere <xref:blazor/components/class-libraries>.
