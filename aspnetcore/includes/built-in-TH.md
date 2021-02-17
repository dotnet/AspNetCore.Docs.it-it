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
ms.openlocfilehash: 1cc2a01725a2af8f3dbfe1b23ea81e99bab9ef8e
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551652"
---
## <a name="built-in-aspnet-core-tag-helpers"></a><span data-ttu-id="87a38-101">Helper per tag incorporati di ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="87a38-101">Built-in ASP.NET Core Tag Helpers</span></span>

<span data-ttu-id="87a38-102">**[Helper per tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-102">**[Anchor Tag Helper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)**</span></span>

<span data-ttu-id="87a38-103">**[Helper tag di cache](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-103">**[Cache Tag Helper](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)**</span></span>

<span data-ttu-id="87a38-104">**[Helper Tag componente](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-104">**[Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)**</span></span>

<span data-ttu-id="87a38-105">**[Helper tag di cache distribuita](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-105">**[Distributed Cache Tag Helper](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)**</span></span>

<span data-ttu-id="87a38-106">**[Helper per tag di ambiente](xref:mvc/views/tag-helpers/builtin-th/environment-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-106">**[Environment Tag Helper](xref:mvc/views/tag-helpers/builtin-th/environment-tag-helper)**</span></span>

<span data-ttu-id="87a38-107">**[Helper tag form](xref:mvc/views/working-with-forms#the-form-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-107">**[Form Tag Helper](xref:mvc/views/working-with-forms#the-form-tag-helper)**</span></span>

<span data-ttu-id="87a38-108">**[Helper per tag di azione modulo](xref:mvc/views/working-with-forms#the-form-action-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-108">**[Form Action Tag Helper](xref:mvc/views/working-with-forms#the-form-action-tag-helper)**</span></span>

<span data-ttu-id="87a38-109">**[Helper per tag di immagine](xref:mvc/views/tag-helpers/builtin-th/image-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-109">**[Image Tag Helper](xref:mvc/views/tag-helpers/builtin-th/image-tag-helper)**</span></span>

<span data-ttu-id="87a38-110">**[Helper tag di input](xref:mvc/views/working-with-forms#the-input-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-110">**[Input Tag Helper](xref:mvc/views/working-with-forms#the-input-tag-helper)**</span></span>

<span data-ttu-id="87a38-111">**[Helper tag di etichetta](xref:mvc/views/working-with-forms#the-label-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-111">**[Label Tag Helper](xref:mvc/views/working-with-forms#the-label-tag-helper)**</span></span>

<span data-ttu-id="87a38-112">**[Helper tag di collegamento](xref:mvc/views/tag-helpers/builtin-th/link-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-112">**[Link Tag Helper](xref:mvc/views/tag-helpers/builtin-th/link-tag-helper)**</span></span>

<span data-ttu-id="87a38-113">**[Helper tag Partial](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-113">**[Partial Tag Helper](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper)**</span></span>

<span data-ttu-id="87a38-114">**[Helper tag script](xref:mvc/views/tag-helpers/builtin-th/script-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-114">**[Script Tag Helper](xref:mvc/views/tag-helpers/builtin-th/script-tag-helper)**</span></span>

<span data-ttu-id="87a38-115">**[Seleziona Helper Tag](xref:mvc/views/working-with-forms#the-select-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-115">**[Select Tag Helper](xref:mvc/views/working-with-forms#the-select-tag-helper)**</span></span>

<span data-ttu-id="87a38-116">**[Helper tag textarea](xref:mvc/views/working-with-forms#the-textarea-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-116">**[Textarea Tag Helper](xref:mvc/views/working-with-forms#the-textarea-tag-helper)**</span></span>

<span data-ttu-id="87a38-117">**[Helper per tag messaggio di convalida](xref:mvc/views/working-with-forms#the-validation-message-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-117">**[Validation Message Tag Helper](xref:mvc/views/working-with-forms#the-validation-message-tag-helper)**</span></span>

<span data-ttu-id="87a38-118">**[Helper tag di riepilogo convalida](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper)**</span><span class="sxs-lookup"><span data-stu-id="87a38-118">**[Validation Summary Tag Helper](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper)**</span></span>
