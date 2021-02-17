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
ms.openlocfilehash: 1a9e98a84093f89f3859454f482dfe59c8504d9b
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551623"
---
Eseguire i seguenti comandi dell'interfaccia della riga di comando di .NET Core:

```dotnetcli
dotnet tool install --global dotnet-ef --version 3.1.9
dotnet tool install --global dotnet-aspnet-codegenerator --version 3.1.4
dotnet add package Microsoft.EntityFrameworkCore.Sqlite --version 3.1.9
dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore --version 3.1.9
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --version 3.1.4
dotnet add package Microsoft.EntityFrameworkCore.Design --version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 3.1.9
dotnet add package Microsoft.Extensions.Logging.Debug --version 3.1.9
```

I comandi precedenti aggiungono:

* Lo [strumento di ponteggi ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).
* Strumenti di Entity Framework Core per il interfaccia della riga di comando di .NET Core.
* Il provider EF Core SQLite, che installa il pacchetto di EF Core come dipendenza.
* I pacchetti necessari per lo scaffolding: `Microsoft.VisualStudio.Web.CodeGeneration.Design` e `Microsoft.EntityFrameworkCore.SqlServer`.

Per istruzioni su più configurazioni dell'ambiente che consentono a un'app di configurare i relativi contesti di database per ambiente, vedere <xref:fundamentals/environments#environment-based-startup-class-and-methods> .

[!INCLUDE[](~/includes/scaffoldTFM.md)]