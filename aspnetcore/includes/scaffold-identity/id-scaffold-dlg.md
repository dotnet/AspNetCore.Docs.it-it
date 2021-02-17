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
ms.openlocfilehash: ed6de823b8b860c7d27e932e9ece40d8b43b0893
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551791"
---
Eseguire l' Identity impalcatura:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Da **Esplora soluzioni**, fare clic con il pulsante destro del mouse sul progetto > **Aggiungi**  >  **nuovo elemento con impalcatura**.
* Dal riquadro sinistro della finestra di dialogo **Aggiungi nuovo elemento con impalcatura** selezionare **Identity**  >  **Aggiungi**.
* Nella finestra di dialogo **Aggiungi Identity** Selezionare le opzioni desiderate.
  * Selezionare la pagina layout esistente. in alternativa, il file di layout verrà sovrascritto con un markup errato:
    * `~/Pages/Shared/_Layout.cshtml` per le Razor pagine
    * `~/Views/Shared/_Layout.cshtml` per i progetti MVC
    * Blazor Server per impostazione predefinita, le app create dal Blazor Server modello ( `blazorserver` ) non sono configurate per le Razor pagine o MVC. Lasciare vuota la voce di pagina layout.
  * Selezionare il **+** pulsante per creare una nuova **classe del contesto dati**. Accettare il valore predefinito o specificare una classe (ad esempio, `MyApplication.Data.ApplicationDbContext` ).
* Selezionare **Aggiungi**.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Se in precedenza non è stato installato il ASP.NET Core impalcatura, installarlo ora:

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

Aggiungere i riferimenti ai pacchetti NuGet necessari al file di progetto (con *estensione csproj*). Eseguire i comandi seguenti nella directory del progetto:

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

Eseguire il comando seguente per elencare le Identity Opzioni dell'impalcatura:

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

[!INCLUDE[](~/includes/scaffoldTFM.md)]

Nella cartella del progetto eseguire l' Identity impalcatura con le opzioni desiderate. Ad esempio, per configurare Identity con l'interfaccia utente predefinita e il numero minimo di file, eseguire il comando seguente:

```dotnetcli
dotnet aspnet-codegenerator identity --useDefaultUI
```

---
