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
ms.openlocfilehash: 1161f7731898221e51a4c7f9f246269b83c6ae80
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551363"
---
::: moniker range=">= aspnetcore-3.0"

Eseguire l' Identity impalcatura:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Da **Esplora soluzioni**, fare clic con il pulsante destro del mouse sul progetto > **Aggiungi** > **nuovo elemento con impalcatura**.
* Dal riquadro sinistro della finestra di dialogo **Aggiungi impalcatura** selezionare **Identity** > **Aggiungi**.
* Nella finestra di dialogo **Aggiungi Identity** Selezionare le opzioni desiderate.
  * Selezionare la pagina layout esistente in modo che il file di layout non venga sovrascritto con un markup errato. Quando si seleziona un file *\_ layout. cshtml* esistente, questo **non** viene sovrascritto. Ad esempio:
    * `~/Pages/Shared/_Layout.cshtml` per Razor pagine o Blazor Server progetti con Razor infrastruttura di pagine esistenti
    * `~/Views/Shared/_Layout.cshtml` per progetti MVC o Blazor Server progetti con infrastruttura MVC esistente
* Per usare il contesto dei dati esistente, selezionare almeno un file di cui eseguire l'override. È necessario selezionare almeno un file per aggiungere il contesto dei dati.
  * Selezionare la classe del contesto dati.
  * Selezionare **Aggiungi**.
* Per creare un nuovo contesto utente ed eventualmente creare una classe utente personalizzata per Identity :
  * Selezionare il **+** pulsante per creare una nuova **classe del contesto dati**. Accettare il valore predefinito o specificare una classe (ad esempio, `MyApplication.Data.ApplicationDbContext` ).
  * Selezionare **Aggiungi**.

Nota: se si sta creando un nuovo contesto utente, non è necessario selezionare un file di cui eseguire l'override.

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

Nella cartella del progetto eseguire l' Identity impalcatura con le opzioni desiderate. Ad esempio, per configurare Identity con l'interfaccia utente predefinita e il numero minimo di file, eseguire il comando seguente. Usare il nome completo corretto per il contesto del database:

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login"
```

PowerShell usa il punto e virgola come separatore di comandi. Quando si usa PowerShell, usare il carattere di escape per i punti e virgola nell'elenco dei file o inserire l'elenco di file tra virgolette doppie. Ad esempio:

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

Se si esegue l' Identity impalcatura senza specificare il `--files` flag o il `--useDefaultUI` flag, Identity verranno create tutte le pagine dell'interfaccia utente disponibili nel progetto.

---

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Eseguire l' Identity impalcatura:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Da **Esplora soluzioni**, fare clic con il pulsante destro del mouse sul progetto > **Aggiungi** > **nuovo elemento con impalcatura**.
* Dal riquadro sinistro della finestra di dialogo **Aggiungi impalcatura** selezionare **Identity** > **Aggiungi**.
* Nella finestra di dialogo **Aggiungi Identity** Selezionare le opzioni desiderate.
  * Selezionare la pagina layout esistente. in alternativa, il file di layout verrà sovrascritto con un markup errato. Quando si seleziona un file *\_ layout. cshtml* esistente, questo **non** viene sovrascritto. Ad esempio:
    * `~/Pages/Shared/_Layout.cshtml` per le Razor pagine
    * `~/Views/Shared/_Layout.cshtml` per i progetti MVC
* Per usare il contesto dei dati esistente, selezionare almeno un file di cui eseguire l'override. È necessario selezionare almeno un file per aggiungere il contesto dei dati.
  * Selezionare la classe del contesto dati.
  * Selezionare **Aggiungi**.
* Per creare un nuovo contesto utente ed eventualmente creare una classe utente personalizzata per Identity :
  * Selezionare il **+** pulsante per creare una nuova **classe del contesto dati**. Accettare il valore predefinito o specificare una classe (ad esempio, `MyApplication.Data.ApplicationDbContext` ).
  * Selezionare **Aggiungi**.

Nota: se si sta creando un nuovo contesto utente, non è necessario selezionare un file di cui eseguire l'override.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Se in precedenza non è stato installato il ASP.NET Core impalcatura, installarlo ora:

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

Aggiungere un riferimento al pacchetto [Microsoft. VisualStudio. Web. CodeGeneration. Design](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/) al file di progetto (con *estensione csproj*). Eseguire i comandi seguenti nella directory del progetto:

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
```

Eseguire il comando seguente per elencare le Identity Opzioni dell'impalcatura:

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

Nella cartella del progetto eseguire l' Identity impalcatura con le opzioni desiderate. Ad esempio, per configurare Identity con l'interfaccia utente predefinita e il numero minimo di file, eseguire il comando seguente. Usare il nome completo corretto per il contesto del database:

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login"
```

PowerShell usa il punto e virgola come separatore di comandi. Quando si usa PowerShell, usare il carattere di escape per i punti e virgola nell'elenco dei file o inserire l'elenco di file tra virgolette doppie. Ad esempio:

```dotnetcli
dotnet aspnet-codegenerator identity -dc MyApplication.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

Se si esegue l' Identity impalcatura senza specificare il `--files` flag o il `--useDefaultUI` flag, Identity verranno create tutte le pagine dell'interfaccia utente disponibili nel progetto.

---

::: moniker-end
