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
ms.openlocfilehash: 27cb64dc8734fcb6d165795515096c9d9dd2a9cc
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552340"
---
<span data-ttu-id="773dc-101">Eseguire i seguenti comandi dell'interfaccia della riga di comando .NET:</span><span class="sxs-lookup"><span data-stu-id="773dc-101">Run the following .NET CLI commands:</span></span>

```dotnetcli
dotnet tool install --global dotnet-ef
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.Extensions.Logging.Debug
```

<span data-ttu-id="773dc-102">I comandi precedenti aggiungono:</span><span class="sxs-lookup"><span data-stu-id="773dc-102">The preceding commands add:</span></span>

* <span data-ttu-id="773dc-103">Lo [strumento di ponteggi ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span><span class="sxs-lookup"><span data-stu-id="773dc-103">The [aspnet-codegenerator scaffolding tool](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span></span>
* <span data-ttu-id="773dc-104">Strumenti di EF Core per l'interfaccia della riga di comando di .NET.</span><span class="sxs-lookup"><span data-stu-id="773dc-104">The EF Core Tools for the .NET CLI.</span></span>
* <span data-ttu-id="773dc-105">Il provider EF Core SQLite, che installa il pacchetto di EF Core come dipendenza.</span><span class="sxs-lookup"><span data-stu-id="773dc-105">The EF Core SQLite provider, which installs the EF Core package as a dependency.</span></span>
* <span data-ttu-id="773dc-106">I pacchetti necessari per lo scaffolding: `Microsoft.VisualStudio.Web.CodeGeneration.Design` e `Microsoft.EntityFrameworkCore.SqlServer`.</span><span class="sxs-lookup"><span data-stu-id="773dc-106">Packages needed for scaffolding: `Microsoft.VisualStudio.Web.CodeGeneration.Design` and `Microsoft.EntityFrameworkCore.SqlServer`.</span></span>

<span data-ttu-id="773dc-107">Per istruzioni su più configurazioni dell'ambiente che consentono a un'app di configurare i relativi contesti di database per ambiente, vedere <xref:fundamentals/environments#environment-based-startup-class-and-methods> .</span><span class="sxs-lookup"><span data-stu-id="773dc-107">For guidance on multiple environment configuration that permits an app to configure its database contexts by environment, see <xref:fundamentals/environments#environment-based-startup-class-and-methods>.</span></span>

[!INCLUDE[](~/includes/scaffoldTFM-5.md)]
