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
Eseguire i seguenti comandi dell'interfaccia della riga di comando .NET:

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

I comandi precedenti aggiungono:

* Lo [strumento di ponteggi ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).
* Strumenti di EF Core per l'interfaccia della riga di comando di .NET.
* Il provider EF Core SQLite, che installa il pacchetto di EF Core come dipendenza.
* I pacchetti necessari per lo scaffolding: `Microsoft.VisualStudio.Web.CodeGeneration.Design` e `Microsoft.EntityFrameworkCore.SqlServer`.

Per istruzioni su più configurazioni dell'ambiente che consentono a un'app di configurare i relativi contesti di database per ambiente, vedere <xref:fundamentals/environments#environment-based-startup-class-and-methods> .

[!INCLUDE[](~/includes/scaffoldTFM-5.md)]
