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
ms.openlocfilehash: 1700adafb58cad57ea1becbf53cad45edd047962
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552025"
---
<span data-ttu-id="4491f-101">Il Identity codice del database generato richiede [Entity Framework Core migrazioni](/ef/core/managing-schemas/migrations/).</span><span class="sxs-lookup"><span data-stu-id="4491f-101">The generated Identity database code requires [Entity Framework Core Migrations](/ef/core/managing-schemas/migrations/).</span></span> <span data-ttu-id="4491f-102">Creare una migrazione e aggiornare il database.</span><span class="sxs-lookup"><span data-stu-id="4491f-102">Create a migration and update the database.</span></span> <span data-ttu-id="4491f-103">Ad esempio, eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="4491f-103">For example, run the following commands:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="4491f-104">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="4491f-104">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="4491f-105">Nella **console di gestione pacchetti** di Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="4491f-105">In the Visual Studio **Package Manager Console**:</span></span>

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
Add-Migration CreateIdentitySchema
Update-Database
```

# <a name="net-core-cli"></a>[<span data-ttu-id="4491f-106">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="4491f-106">.NET Core CLI</span></span>](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
```

---

<span data-ttu-id="4491f-107">Il parametro del Identity nome "create schema" per il `Add-Migration` comando è arbitrario.</span><span class="sxs-lookup"><span data-stu-id="4491f-107">The "CreateIdentitySchema" name parameter for the `Add-Migration` command is arbitrary.</span></span> <span data-ttu-id="4491f-108">`"CreateIdentitySchema"` viene descritta la migrazione.</span><span class="sxs-lookup"><span data-stu-id="4491f-108">`"CreateIdentitySchema"` describes the migration.</span></span>
