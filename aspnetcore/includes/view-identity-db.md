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
ms.openlocfilehash: ff0502f68546213d76fe33f45574b5d8654b522b
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552699"
---
### <a name="view-the-identity-database"></a>Visualizzare il Identity database

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio) 

* Scegliere **Esplora oggetti di SQL Server** (SSOX) dal menu **Visualizza** .
* Passare a **(local DB) MSSQLLocalDB (SQL Server 13)**. Fare clic con il pulsante destro del mouse su **dbo. AspNetUsers**  >  **visualizzare i dati**:

![Menu contestuale nella tabella AspNetUsers in Esplora oggetti di SQL Server](~/security/authentication/accconfirm/_static/ssox.png)

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Sono disponibili molti strumenti di terze parti che è possibile scaricare per gestire e visualizzare un database SQLite, ad esempio il [browser DB per SQLite](https://sqlitebrowser.org/).

---