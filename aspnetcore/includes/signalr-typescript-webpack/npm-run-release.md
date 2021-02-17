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
ms.openlocfilehash: 05c94351ee4747813cfa8dc2318a6fc02c3a46bf
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552893"
---
```console
npm run release
```

<span data-ttu-id="23a51-101">Questo comando genera gli asset lato client da servire durante l'esecuzione dell'app.</span><span class="sxs-lookup"><span data-stu-id="23a51-101">This command generates the client-side assets to be served when running the app.</span></span> <span data-ttu-id="23a51-102">Le risorse si trovano nella cartella *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="23a51-102">The assets are placed in the *wwwroot* folder.</span></span>

<span data-ttu-id="23a51-103">Webpack ha completato le attività seguenti:</span><span class="sxs-lookup"><span data-stu-id="23a51-103">Webpack completed the following tasks:</span></span>

* <span data-ttu-id="23a51-104">Ha ripulito il contenuto della directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="23a51-104">Purged the contents of the *wwwroot* directory.</span></span>
* <span data-ttu-id="23a51-105">Il TypeScript è stato convertito in JavaScript in un processo noto come *transpilazione*.</span><span class="sxs-lookup"><span data-stu-id="23a51-105">Converted the TypeScript to JavaScript in a process known as *transpilation*.</span></span>
* <span data-ttu-id="23a51-106">Il codice JavaScript generato è stato modificato per ridurre le dimensioni del file in un processo noto come *minification*.</span><span class="sxs-lookup"><span data-stu-id="23a51-106">Mangled the generated JavaScript to reduce file size in a process known as *minification*.</span></span>
* <span data-ttu-id="23a51-107">Ha copiato i file JavaScript, CSS e HTML elaborati dalla directory *src* alla directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="23a51-107">Copied the processed JavaScript, CSS, and HTML files from *src* to the *wwwroot* directory.</span></span>
* <span data-ttu-id="23a51-108">Ha inserito gli elementi seguenti nel file *wwwroot/index.html*:</span><span class="sxs-lookup"><span data-stu-id="23a51-108">Injected the following elements into the *wwwroot/index.html* file:</span></span>
  * <span data-ttu-id="23a51-109">Un `<link>` tag, che fa riferimento A *wwwroot/Main. \<hash\> . file CSS* .</span><span class="sxs-lookup"><span data-stu-id="23a51-109">A `<link>` tag, referencing the *wwwroot/main.\<hash\>.css* file.</span></span> <span data-ttu-id="23a51-110">Questo tag viene inserito immediatamente prima del tag `</head>` di chiusura.</span><span class="sxs-lookup"><span data-stu-id="23a51-110">This tag is placed immediately before the closing `</head>` tag.</span></span>
  * <span data-ttu-id="23a51-111">Un `<script>` tag, che fa riferimento A minimizzati *wwwroot/Main. \<hash\> . file js* .</span><span class="sxs-lookup"><span data-stu-id="23a51-111">A `<script>` tag, referencing the minified *wwwroot/main.\<hash\>.js* file.</span></span> <span data-ttu-id="23a51-112">Questo tag viene inserito immediatamente prima del tag `</body>` di chiusura.</span><span class="sxs-lookup"><span data-stu-id="23a51-112">This tag is placed immediately before the closing `</body>` tag.</span></span>