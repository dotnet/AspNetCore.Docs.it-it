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
Il Identity codice del database generato richiede [Entity Framework Core migrazioni](/ef/core/managing-schemas/migrations/). Creare una migrazione e aggiornare il database. Ad esempio, eseguire i comandi seguenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Nella **console di gestione pacchetti** di Visual Studio:

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
Add-Migration CreateIdentitySchema
Update-Database
```

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
```

---

Il parametro del Identity nome "create schema" per il `Add-Migration` comando è arbitrario. `"CreateIdentitySchema"` viene descritta la migrazione.
