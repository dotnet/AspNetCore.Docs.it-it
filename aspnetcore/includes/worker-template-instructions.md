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
ms.openlocfilehash: 34e12afd6bc7015e4609fbdf773259ed5545809b
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551928"
---
# <a name="visual-studio"></a>[<span data-ttu-id="98f39-101">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="98f39-101">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="98f39-102">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="98f39-102">Create a new project.</span></span>
1. <span data-ttu-id="98f39-103">Selezionare il **servizio Worker**.</span><span class="sxs-lookup"><span data-stu-id="98f39-103">Select **Worker Service**.</span></span> <span data-ttu-id="98f39-104">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="98f39-104">Select **Next**.</span></span>
1. <span data-ttu-id="98f39-105">Specificare il nome di un progetto nel campo **Nome progetto** oppure accettare il nome predefinito.</span><span class="sxs-lookup"><span data-stu-id="98f39-105">Provide a project name in the **Project name** field or accept the default project name.</span></span> <span data-ttu-id="98f39-106">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="98f39-106">Select **Create**.</span></span>
1. <span data-ttu-id="98f39-107">Nella finestra di dialogo **Crea un nuovo servizio** del ruolo di lavoro selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="98f39-107">In the **Create a new Worker service** dialog, select **Create**.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="98f39-108">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="98f39-108">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="98f39-109">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="98f39-109">Create a new project.</span></span>
1. <span data-ttu-id="98f39-110">Selezionare **app** in **.NET Core** nella barra laterale.</span><span class="sxs-lookup"><span data-stu-id="98f39-110">Select **App** under **.NET Core** in the sidebar.</span></span>
1. <span data-ttu-id="98f39-111">Selezionare **Worker** in **ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="98f39-111">Select **Worker** under **ASP.NET Core**.</span></span> <span data-ttu-id="98f39-112">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="98f39-112">Select **Next**.</span></span>
1. <span data-ttu-id="98f39-113">Selezionare **.NET Core 3,0** o versione successiva per il **Framework di destinazione**.</span><span class="sxs-lookup"><span data-stu-id="98f39-113">Select **.NET Core 3.0** or later for the **Target Framework**.</span></span> <span data-ttu-id="98f39-114">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="98f39-114">Select **Next**.</span></span>
1. <span data-ttu-id="98f39-115">Specificare un nome nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="98f39-115">Provide a name in the **Project Name** field.</span></span> <span data-ttu-id="98f39-116">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="98f39-116">Select **Create**.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="98f39-117">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="98f39-117">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="98f39-118">Usare il servizio di ruolo di lavoro (`worker`) con il comando[dotnet new](/dotnet/core/tools/dotnet-new) da una shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="98f39-118">Use the Worker Service (`worker`) template with the [dotnet new](/dotnet/core/tools/dotnet-new) command from a command shell.</span></span> <span data-ttu-id="98f39-119">Nell'esempio seguente viene creata un'app del servizio di ruolo di lavoro denominata `ContosoWorker`.</span><span class="sxs-lookup"><span data-stu-id="98f39-119">In the following example, a Worker Service app is created named `ContosoWorker`.</span></span> <span data-ttu-id="98f39-120">Una cartella per l'app `ContosoWorker` viene creata automaticamente quando viene eseguito il comando.</span><span class="sxs-lookup"><span data-stu-id="98f39-120">A folder for the `ContosoWorker` app is created automatically when the command is executed.</span></span>

```dotnetcli
dotnet new worker -o ContosoWorker
```

---
