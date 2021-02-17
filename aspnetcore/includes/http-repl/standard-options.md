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
ms.openlocfilehash: 30c4c469453f0202fe5310dbe00b3d7214f49b5a
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551706"
---
* `-F|--no-formatting`

  <span data-ttu-id="8dc7c-101">Flag la cui presenza elimina la formattazione della risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-101">A flag whose presence suppresses HTTP response formatting.</span></span>

* `-h|--header`

  <span data-ttu-id="8dc7c-102">Imposta un'intestazione della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-102">Sets an HTTP request header.</span></span> <span data-ttu-id="8dc7c-103">Sono supportati i due formati di valore seguenti:</span><span class="sxs-lookup"><span data-stu-id="8dc7c-103">The following two value formats are supported:</span></span>

  * `{header}={value}`
  * `{header}:{value}`

* `--response:body`

  <span data-ttu-id="8dc7c-104">Specifica un file in cui deve essere scritto il corpo della risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-104">Specifies a file to which the HTTP response body should be written.</span></span> <span data-ttu-id="8dc7c-105">Ad esempio: `--response:body "C:\response.json"`.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-105">For example, `--response:body "C:\response.json"`.</span></span> <span data-ttu-id="8dc7c-106">Se il file non esiste, verrà creato.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-106">The file is created if it doesn't exist.</span></span>

* `--response:headers`

  <span data-ttu-id="8dc7c-107">Specifica un file in cui devono essere scritte le intestazioni della risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-107">Specifies a file to which the HTTP response headers should be written.</span></span> <span data-ttu-id="8dc7c-108">Ad esempio: `--response:headers "C:\response.txt"`.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-108">For example, `--response:headers "C:\response.txt"`.</span></span> <span data-ttu-id="8dc7c-109">Se il file non esiste, verrà creato.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-109">The file is created if it doesn't exist.</span></span>

* `-s|--streaming`

  <span data-ttu-id="8dc7c-110">Flag la cui presenza abilita il flusso della risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="8dc7c-110">A flag whose presence enables streaming of the HTTP response.</span></span>
