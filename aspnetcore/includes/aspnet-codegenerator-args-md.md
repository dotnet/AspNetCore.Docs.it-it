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
ms.openlocfilehash: 41d4fd2e746e08d32d9f666faab55acc56817be2
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551605"
---
<!-- Options common to Razor Pages and Controller -->
| <span data-ttu-id="7ca79-101">Opzione</span><span class="sxs-lookup"><span data-stu-id="7ca79-101">Option</span></span>               | <span data-ttu-id="7ca79-102">Descrizione</span><span class="sxs-lookup"><span data-stu-id="7ca79-102">Description</span></span>|
| ----------------- | ------------ |
| <span data-ttu-id="7ca79-103">--model oppure -m</span><span class="sxs-lookup"><span data-stu-id="7ca79-103">--model or -m</span></span>  | <span data-ttu-id="7ca79-104">Classe di modelli da usare.</span><span class="sxs-lookup"><span data-stu-id="7ca79-104">Model class to use.</span></span> |
| <span data-ttu-id="7ca79-105">--dataContext oppure -dc</span><span class="sxs-lookup"><span data-stu-id="7ca79-105">--dataContext or -dc</span></span>  | <span data-ttu-id="7ca79-106">Classe `DbContext` da usare.</span><span class="sxs-lookup"><span data-stu-id="7ca79-106">The `DbContext` class to use.</span></span> |
| <span data-ttu-id="7ca79-107">--bootstrapVersion oppure -b</span><span class="sxs-lookup"><span data-stu-id="7ca79-107">--bootstrapVersion or -b</span></span>  | <span data-ttu-id="7ca79-108">Specifica la versione di bootstrap.</span><span class="sxs-lookup"><span data-stu-id="7ca79-108">Specifies the bootstrap version.</span></span> <span data-ttu-id="7ca79-109">I valori validi sono `3` o `4`.</span><span class="sxs-lookup"><span data-stu-id="7ca79-109">Valid values are `3` or `4`.</span></span> <span data-ttu-id="7ca79-110">Il valore predefinito è `4`.</span><span class="sxs-lookup"><span data-stu-id="7ca79-110">Default is `4`.</span></span> <span data-ttu-id="7ca79-111">Se necessaria e non presente, viene creata una directory *wwwroot* che include i file di bootstrap della versione specificata.</span><span class="sxs-lookup"><span data-stu-id="7ca79-111">If needed and not present, a *wwwroot* directory is created that includes the bootstrap files of the specified version.</span></span> |
| <span data-ttu-id="7ca79-112">--referenceScriptLibraries oppure -scripts</span><span class="sxs-lookup"><span data-stu-id="7ca79-112">--referenceScriptLibraries or -scripts</span></span> |  <span data-ttu-id="7ca79-113">Librerie di script di riferimento nelle viste generate.</span><span class="sxs-lookup"><span data-stu-id="7ca79-113">Reference script libraries in the generated views.</span></span> <span data-ttu-id="7ca79-114">Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine.</span><span class="sxs-lookup"><span data-stu-id="7ca79-114">Adds `_ValidationScriptsPartial` to Edit and Create pages.</span></span> |
| <span data-ttu-id="7ca79-115">--layout oppure -l</span><span class="sxs-lookup"><span data-stu-id="7ca79-115">--layout or -l</span></span> | <span data-ttu-id="7ca79-116">Pagina di layout personalizzata da usare.</span><span class="sxs-lookup"><span data-stu-id="7ca79-116">Custom Layout page to use.</span></span> |
| <span data-ttu-id="7ca79-117">--useDefaultLayout oppure -udl</span><span class="sxs-lookup"><span data-stu-id="7ca79-117">--useDefaultLayout or -udl</span></span> | <span data-ttu-id="7ca79-118">Usare il layout predefinito per le viste.</span><span class="sxs-lookup"><span data-stu-id="7ca79-118">Use the default layout for the views.</span></span> |
| <span data-ttu-id="7ca79-119">--force oppure -f</span><span class="sxs-lookup"><span data-stu-id="7ca79-119">--force or -f</span></span> | <span data-ttu-id="7ca79-120">Sovrascrivere i file esistenti.</span><span class="sxs-lookup"><span data-stu-id="7ca79-120">Overwrite existing files.</span></span> |
| <span data-ttu-id="7ca79-121">--relativeFolderPath oppure -outDir</span><span class="sxs-lookup"><span data-stu-id="7ca79-121">--relativeFolderPath or -outDir</span></span> | <span data-ttu-id="7ca79-122">Percorso relativo della cartella di output dal progetto in cui vengono generati i file.</span><span class="sxs-lookup"><span data-stu-id="7ca79-122">The relative output folder path from project where the file are generated.</span></span> <span data-ttu-id="7ca79-123">Se non è specificato, i file vengono generati nella cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="7ca79-123">If not specified, files are generated in the project folder.</span></span> |