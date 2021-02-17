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
ms.openlocfilehash: 6808c71b1ca43755eea4958ff9409f40e8685694
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551506"
---
<span data-ttu-id="005c6-101">Questa esercitazione illustra ASP.NET Core MVC ed Entity Framework Core con i controller e le viste.</span><span class="sxs-lookup"><span data-stu-id="005c6-101">This tutorial teaches ASP.NET Core MVC and Entity Framework Core with controllers and views.</span></span> <span data-ttu-id="005c6-102">[ Razor Pagine](xref:razor-pages/index) è un modello di programmazione alternativo.</span><span class="sxs-lookup"><span data-stu-id="005c6-102">[Razor Pages](xref:razor-pages/index) is an alternative programming model.</span></span> <span data-ttu-id="005c6-103">Per nuove attività di sviluppo, è consigliabile eseguire Razor pagine su MVC con controller e visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="005c6-103">For new development, we recommend Razor Pages over MVC with controllers and views.</span></span> <span data-ttu-id="005c6-104">Vedere la versione di questa esercitazione relativa alle [ Razor pagine](xref:data/ef-rp/intro) .</span><span class="sxs-lookup"><span data-stu-id="005c6-104">See the [Razor Pages](xref:data/ef-rp/intro) version of this tutorial.</span></span> <span data-ttu-id="005c6-105">Ogni esercitazione illustra alcuni materiali che l'altra versione non contiene:</span><span class="sxs-lookup"><span data-stu-id="005c6-105">Each tutorial covers some material the other doesn't:</span></span>

<span data-ttu-id="005c6-106">In questa esercitazione su MVC è presente che l'esercitazione sulle pagine non esegue le operazioni Razor seguenti:</span><span class="sxs-lookup"><span data-stu-id="005c6-106">Some things this MVC tutorial has that the Razor Pages tutorial doesn't:</span></span>

* <span data-ttu-id="005c6-107">Implementare l'ereditarietà nel modello di dati</span><span class="sxs-lookup"><span data-stu-id="005c6-107">Implement inheritance in the data model</span></span>
* <span data-ttu-id="005c6-108">Eseguire query SQL non elaborate</span><span class="sxs-lookup"><span data-stu-id="005c6-108">Perform raw SQL queries</span></span>
* <span data-ttu-id="005c6-109">Usare LINQ dinamico per semplificare il codice</span><span class="sxs-lookup"><span data-stu-id="005c6-109">Use dynamic LINQ to simplify code</span></span>

<span data-ttu-id="005c6-110">Per alcune operazioni Razor , l'esercitazione sulle pagine non è la seguente:</span><span class="sxs-lookup"><span data-stu-id="005c6-110">Some things the Razor Pages tutorial has that this one doesn't:</span></span>

* <span data-ttu-id="005c6-111">Usare il metodo Select per caricare i dati correlati</span><span class="sxs-lookup"><span data-stu-id="005c6-111">Use Select method to load related data</span></span>
* <span data-ttu-id="005c6-112">Procedure consigliate per EF.</span><span class="sxs-lookup"><span data-stu-id="005c6-112">Best practices for EF.</span></span>