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
## <a name="share-interop-code-in-a-class-library"></a><span data-ttu-id="25c68-101">Condividere il codice di interoperabilità in una libreria di classi</span><span class="sxs-lookup"><span data-stu-id="25c68-101">Share interop code in a class library</span></span>

<span data-ttu-id="25c68-102">Il codice di interoperabilità JS può essere incluso in una libreria di classi, che consente di condividere il codice in un pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="25c68-102">JS interop code can be included in a class library, which allows you to share the code in a NuGet package.</span></span>

<span data-ttu-id="25c68-103">La libreria di classi gestisce l'incorporamento delle risorse JavaScript nell'assembly compilato.</span><span class="sxs-lookup"><span data-stu-id="25c68-103">The class library handles embedding JavaScript resources in the built assembly.</span></span> <span data-ttu-id="25c68-104">I file JavaScript vengono inseriti nella `wwwroot` cartella.</span><span class="sxs-lookup"><span data-stu-id="25c68-104">The JavaScript files are placed in the `wwwroot` folder.</span></span> <span data-ttu-id="25c68-105">Gli strumenti si occupano di incorporare le risorse quando la libreria viene compilata.</span><span class="sxs-lookup"><span data-stu-id="25c68-105">The tooling takes care of embedding the resources when the library is built.</span></span>

<span data-ttu-id="25c68-106">Nel file di progetto dell'app viene fatto riferimento al pacchetto NuGet compilato nello stesso modo in cui viene fatto riferimento a un pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="25c68-106">The built NuGet package is referenced in the app's project file the same way that any NuGet package is referenced.</span></span> <span data-ttu-id="25c68-107">Dopo il ripristino del pacchetto, il codice dell'app può chiamare JavaScript come se fosse C#.</span><span class="sxs-lookup"><span data-stu-id="25c68-107">After the package is restored, app code can call into JavaScript as if it were C#.</span></span>

<span data-ttu-id="25c68-108">Per altre informazioni, vedere <xref:blazor/components/class-libraries>.</span><span class="sxs-lookup"><span data-stu-id="25c68-108">For more information, see <xref:blazor/components/class-libraries>.</span></span>
