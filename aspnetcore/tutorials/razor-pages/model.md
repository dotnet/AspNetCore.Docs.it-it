---
title: Parte 2, aggiungere un modello
author: rick-anderson
description: Parte 2 della serie di esercitazioni sulle Razor pagine. In questa sezione vengono aggiunte le classi del modello.
ms.author: riande
ms.date: 11/11/2020
ms.custom: contperf-fy21q2
no-loc:
- Index
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
uid: tutorials/razor-pages/model
ms.openlocfilehash: 92bfda330399b43871b3ae0e6b609726f7ad4a91
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564048"
---
# <a name="part-2-add-a-model-to-a-razor-pages-app-in-aspnet-core"></a>Parte 2, aggiungere un modello a un' Razor app di pagine in ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

<!-- In the next update on the CLI version, let the scaffolder do the same work the VS driven scaffolder does. That is, create the DB context, etc -->

In questa sezione si aggiungono alcune classi per la gestione di filmati in un database. Le classi modello dell'app usano [Entity Framework Core (EF Core)](/ef/core) per lavorare con il database. EF Core è un mapper relazionale a oggetti (O/RM) che semplifica l'accesso ai dati. Prima di tutto si scrivono le classi del modello e EF Core crea il database.

Le classi del modello sono note come classi POCO (da "**P** Lain-**o** LD **C** LR **o** gli oggetti") perché non hanno una dipendenza da EF core. Definiscono le proprietà dei dati archiviati nel database.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).

## <a name="add-a-data-model"></a>Aggiungere un modello di dati

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto *Razor PagesMovie* > **Aggiungi**  >  **nuova cartella**. Assegnare il nome *Modelli* alla cartella.
1. Fare clic con il pulsante destro del mouse sulla cartella *modelli* . Selezionare **Aggiungi**  >  **classe**. Denominare la classe *Movie*.
1. Aggiungere le proprietà seguenti alla classe `Movie`:

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * L'utente non deve immettere le informazioni sull'ora nel campo Data.
  * Viene visualizzata solo la data, non le informazioni temporali.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. Aggiungere una cartella denominata *Modelli*.
1. Aggiungere una classe alla cartella *Modello* denominata *Movie.cs*.

Aggiungere le proprietà seguenti alla classe `Movie`:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a>Aggiungere i pacchetti NuGet e gli strumenti EF

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI-5.md)]

### <a name="add-a-database-context-class"></a>Aggiungere una classe di contesto dei dati

1. Nel progetto *Razor PagesMovie* creare una cartella denominata *Data*.
1. Nella cartella *dati* aggiungere un file denominato *Razor PagesMovieContext.cs* con il codice seguente:

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

   Il codice precedente crea una proprietà `DbSet` per il set di entità. Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella. Il codice non verrà compilato fino a quando le dipendenze non verranno aggiunte in un passaggio successivo.

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a>Aggiungere una stringa di connessione del database

Aggiungere una stringa di connessione al *appsettings.json* file come illustrato nel codice evidenziato seguente:

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/appsettings_SQLite.json?highlight=10-12)]

<a name="reg"></a>

### <a name="register-the-database-context"></a>Registrare il contesto del database

1. Aggiungere le istruzioni `using` seguenti all'inizio di *Startup.cs*:

   ```csharp
   using RazorPagesMovie.Data;
   using Microsoft.EntityFrameworkCore;
   ```

1. Registrare il contesto del database con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) in `Startup.ConfigureServices` :

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Nella **finestra degli strumenti della soluzione**, fare clic sul progetto *Razor PagesMovie* e quindi selezionare **Aggiungi** > **nuova cartella.** Assegnare un nome ai *modelli* di cartella.
1. CTRL-fare clic sulla cartella *modelli* , quindi selezionare **Aggiungi** > **nuovo file.**...
1. Nel finestra di dialogo **Nuovo file**:
   1. Selezionare **Generale** nel riquadro a sinistra.
   1. Selezionare **Classe vuota** nel riquadro centrale.
   1. Denominare la classe **Filmato** e selezionare **Nuovo**.

1. Aggiungere le proprietà seguenti alla classe `Movie`:

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

---

L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.

Compilare il progetto per verificare che non siano presenti errori di compilazione.

## <a name="scaffold-the-movie-model"></a>Eseguire lo scaffolding del modello di filmato

In questa sezione viene eseguito lo scaffolding del modello *Movie*. Lo strumento di scaffolding crea quindi le pagine per le operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) per il modello *Movie*.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Creare una cartella *Pages/Movies*:
   1. Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.
   1. Denominare la cartella *Movies*.

1. Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **nuovo elemento con impalcatura**.

   ![Immagine relativa alle istruzioni precedenti.](model/_static/5/sca.png)

1. Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.

   ![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffold.png)

1. Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :
   1. Nell'elenco a discesa **classe modello** selezionare **Movie ( Razor PagesMovie. Models)**.
   1. Nella riga **Classe contesto di dati** selezionare il segno più **+**.
      1. Nella finestra di dialogo **Aggiungi contesto dati** viene generato il nome della classe `RazorPagesMovie.Data.RazorPagesMovieContext` .
   1. Selezionare **Aggiungi**.

   ![Immagine relativa alle istruzioni precedenti.](model/_static/3/arp.png)

Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* Aprire una shell dei comandi nella directory del progetto che contiene i file *Program.cs*, *Startup.cs* e *. csproj* .

* **Per Windows**: eseguire il comando seguente:

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **Per MacOS e Linux**: eseguire il comando seguente:

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> La tabella seguente illustra in dettaglio le opzioni del generatore di codice ASP.NET Core.

| Opzione               | Descrizione|
| ----------------- | ------------ |
| `-m`  | Nome del modello. |
| `-dc`  | Classe `DbContext` da usare. |
| `-udl` | Uso del layout predefinito. |
| `-outDir` | Percorso relativo della cartella di output per creare le viste. |
| `--referenceScriptLibraries` | Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine |

Usare l' `-h` opzione per ottenere informazioni sul `aspnet-codegenerator razorpage` comando:

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

Per ulteriori informazioni, vedere [DotNet-ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).

### <a name="use-sqlite-for-development-sql-server-for-production"></a>Usare SQLite per lo sviluppo, SQL Server per la produzione

Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo. Nel codice seguente viene illustrato come inserire <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> in `Startup` . `IWebHostEnvironment` viene inserito in modo che l'app possa usare SQLite in fase di sviluppo e SQL Server nell'ambiente di produzione.

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Creare una cartella *Pages/Movies*:
   1. CTRL: fare clic sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.
   1. Denominare la cartella *Movies*.

1. CTRL: fare clic sulla cartella *pagine/filmati* > **Aggiungi** > **Nuova impalcatura...**.

   ![Immagine relativa alle istruzioni precedenti.](model/_static/scaMac.png)

1. Nella finestra di dialogo **Nuova impalcatura** selezionare **Razor pagine con Entity Framework (CRUD)** > **Avanti**.

   ![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffoldMac.png)

1. Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :
   1. Nella **classe DbContext da usare:** Row, assegnare un nome alla classe `RazorPagesMovie.Data.RazorPagesMovieContext` .
   1. Selezionare **Fine**.

   ![Immagine relativa alle istruzioni precedenti.](model/_static/5/arpMac.png)

Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.

### <a name="use-sqlite-for-development-sql-server-for-production"></a>Usare SQLite per lo sviluppo, SQL Server per la produzione

Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo. Nel codice seguente viene illustrato come inserire <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> in `Startup` . `IWebHostEnvironment` viene inserito in modo che l'app possa usare SQLite in fase di sviluppo e SQL Server nell'ambiente di produzione.

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

---

### <a name="files-created"></a>File creati

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Il processo di scaffolding crea e aggiorna i file seguenti:

* *Pagine/filmati*: creare, eliminare, Details, Edit e Index .
* *Data/ Razor PagesMovieContext.cs*

### <a name="updated"></a>Aggiornato

* *Startup.cs*

I file creati e aggiornati sono illustrati nella sezione successiva.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Il processo di scaffolding crea i file seguenti:

* *Pagine/filmati*: creare, eliminare, Details, Edit e Index .

I file creati sono illustrati nella sezione successiva.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Il processo di scaffolding crea e aggiorna i file seguenti:

* *Pagine/filmati*: creare, eliminare, Details, Edit e Index .
* *Data/ Razor PagesMovieContext.cs*

### <a name="updated"></a>Aggiornato

* *Startup.cs*

I file creati e aggiornati sono illustrati nella sezione successiva.

---

<a name="pmc"></a>

## <a name="create-the-initial-database-schema-using-efs-migration-feature"></a>Creare lo schema del database iniziale usando la funzionalità di migrazione di EF

La funzionalità migrazioni di Entity Framework Core fornisce un modo per:

* Creare lo schema del database iniziale.
* Aggiornare in modo incrementale lo schema del database per mantenerlo sincronizzato con il modello di dati dell'applicazione.  Vengono conservati i dati esistenti nel database.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

In questa sezione viene usata la finestra **console di gestione pacchetti** (PMC) per:

* Aggiungere una migrazione iniziale.
* Aggiornare il database con la migrazione iniziale.

1. Nel menu **Strumenti** selezionare **Gestione pacchetti NuGet** > **Console di Gestione pacchetti**.

   ![Menu della Console di Gestione pacchetti](model/_static/5/pmc.png)

1. Nella Console di Gestione pacchetti immettere i comandi seguenti:

   ```powershell
   Add-Migration InitialCreate
   Update-Database
   ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

* Eseguire i seguenti comandi dell'interfaccia della riga di comando .NET:

  ```dotnetcli
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```

---

I comandi precedenti generano l'avviso seguente: "nessun tipo specificato per la colonna decimale ' Price ' nel tipo di entità' Movie '. This will cause values to be silently truncated if they do not fit in the default precision and scale. (I valori saranno quindi automaticamente troncati se non rispettano la precisione e la scala predefinite). Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'. (Specificare in modo esplicito il tipo di colonna di SQL Server che può supportare tutti i valori usando 'HasColumnType()')"

Ignorare l'avviso poiché verrà risolto in un passaggio successivo.

Il comando `migrations` genera un codice per creare lo schema del database iniziale. Lo schema è basato sul modello specificato in `DbContext` . L'argomento `InitialCreate` viene usato per denominare le migrazioni. È possibile usare qualsiasi nome, ma per convenzione viene selezionato un nome che descrive la migrazione.

Il `update` comando esegue il `Up` Metodo nelle migrazioni che non sono state applicate. In tal caso, `update` esegue il `Up` metodo nel file *migrations/ \<time-stamp> _InitialCreate. cs* , che crea il database.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>Esaminare il contesto registrato con l'inserimento di dipendenze

ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection). I servizi, ad esempio il contesto di database EF Core, vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione. I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore. Più avanti nell'esercitazione viene illustrato il codice del costruttore che ottiene un'istanza del contesto di database.

Lo strumento di ponteggi crea automaticamente un contesto di database e lo registra con il contenitore di inserimento delle dipendenze.

Esaminare il metodo `Startup.ConfigureServices`. La riga evidenziata è stata aggiunta dallo scaffolder:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

Le `RazorPagesMovieContext` coordinate EF Core funzionalità, ad esempio creazione, lettura, aggiornamento ed eliminazione, per il `Movie` modello. Il contesto dei dati (`RazorPagesMovieContext`) è derivato da [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext). Il contesto dei dati specifica le entità incluse nel modello di dati.

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Data/RazorPagesMovieContext.cs)]

Il codice precedente crea una [proprietà \<Movie> DbSet](xref:Microsoft.EntityFrameworkCore.DbSet%601) per il set di entità. Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database. Un'entità corrisponde a una riga nella tabella.

Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions). Per lo sviluppo locale, il [sistema di configurazione](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Esaminare il metodo `Up`.

---

<a name="test"></a>

## <a name="test-the-app"></a>Testare l'app

1. Eseguire l'app e accodare `/Movies` all'URL nel browser (`http://localhost:port/movies`).

   Se si riceve un messaggio di errore che indica che

   ```console
   SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
   Login failed for user 'User-name'.
   ```

   Non è stato eseguita la [migrazione](#pmc).

1. Eseguire il test del collegamento **Crea**.

   ![Pagina di creazione](model/_static/conan5.png)

   > [!NOTE]
   > Potrebbe non essere possibile immettere virgole decimali nel campo `Price`. Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per impostazioni locali diverse dall'inglese che usano la virgola (",") come separatore decimale e per formati di data diversi da quello dell'inglese (Stati Uniti), è necessario localizzare l'app. Per istruzioni sulla globalizzazione, vedere [questo problema su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).

1. Testare i collegamenti **Modifica**, **Dettagli** ed **Elimina**.

L'esercitazione successiva illustra i file creati tramite scaffolding.

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: introduzione](xref:tutorials/razor-pages/razor-pages-start) 
>  Passaggio [successivo: impalcatura Razor Pagine](xref:tutorials/razor-pages/page) di

::: moniker-end

<!--  ::: End of >= 5.0 moniker version   -->

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<!-- In the next update on the CLI version, let the scaffolder do the same work the VS driven scaffolder does. That is, create the DB context, etc -->

In questa sezione vengono aggiunte le classi per la gestione dei filmati. Le classi modello dell'app usano [Entity Framework Core (EF Core)](/ef/core) per lavorare con il database. EF Core è un mapper relazionale a oggetti (O/RM) che semplifica l'accesso ai dati.

Le classi di modello sono dette classi POCO (da "plain-old CLR objects") perché non hanno alcuna dipendenza in EF Core. Definiscono le proprietà dei dati archiviati nel database.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).

## <a name="add-a-data-model"></a>Aggiungere un modello di dati

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Fare clic con il pulsante destro del mouse sul progetto **Razor PagesMovie** > **Aggiungi**  >  **nuova cartella**. Assegnare il nome *Modelli* alla cartella.

Fare clic con il pulsante destro del mouse sulla cartella *modelli* . Selezionare **Aggiungi**  >  **classe**. Denominare la classe **Movie**.

Aggiungere le proprietà seguenti alla classe `Movie`:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aggiungere una cartella denominata *Modelli*.
* Aggiungere una classe alla cartella *Modello* denominata *Movie.cs*.

Aggiungere le proprietà seguenti alla classe `Movie`:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a>Aggiungere i pacchetti NuGet e gli strumenti EF

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

### <a name="add-a-database-context-class"></a>Aggiungere una classe di contesto dei dati

* Nel progetto *Razor PagesMovie* creare una nuova cartella denominata *Data*.
* Aggiungere la classe `RazorPagesMovieContext` seguente alla cartella *Data*:

  [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

Il codice precedente crea una proprietà `DbSet` per il set di entità. Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella. Il codice non verrà compilato fino a quando le dipendenze non verranno aggiunte in un passaggio successivo.

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a>Aggiungere una stringa di connessione del database

Aggiungere una stringa di connessione al *appsettings.json* file come illustrato nel codice evidenziato seguente:

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/appsettings_SQLite.json?highlight=10-12)]

<a name="reg"></a>

### <a name="register-the-database-context"></a>Registrare il contesto del database

Aggiungere le istruzioni `using` seguenti all'inizio di *Startup.cs*:

```csharp
using RazorPagesMovie.Data;
using Microsoft.EntityFrameworkCore;
```

Registrare il contesto del database con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) in `Startup.ConfigureServices`.

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Nella **finestra degli strumenti della soluzione**, fare clic sul progetto **Razor PagesMovie** e quindi selezionare **Aggiungi** > **nuova cartella.** Assegnare un nome ai *modelli* di cartella.
* Fare clic con il pulsante destro del mouse sulla cartella *Models* , quindi scegliere **Aggiungi** > **nuovo file**.
* Nel finestra di dialogo **Nuovo file**:

  * Selezionare **Generale** nel riquadro a sinistra.
  * Selezionare **Classe vuota** nel riquadro centrale.
  * Denominare la classe **Filmato** e selezionare **Nuovo**.

Aggiungere le proprietà seguenti alla classe `Movie`:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

---

L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.

Compilare il progetto per verificare che non siano presenti errori di compilazione.

## <a name="scaffold-the-movie-model"></a>Eseguire lo scaffolding del modello di filmato

In questa sezione viene eseguito lo scaffolding del modello *Movie*. Lo strumento di scaffolding crea quindi le pagine per le operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) per il modello *Movie*.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Creare una cartella *Pages/Movies*:

* Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.
* Denominare la cartella *Movies*.

Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **nuovo elemento con impalcatura**.

![Immagine relativa alle istruzioni precedenti.](model/_static/sca.png)

Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffold.png)

Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :

* Nell'elenco a discesa **classe modello** selezionare **Movie ( Razor PagesMovie. Models)**.
* Nella riga della **classe del contesto dati** selezionare il **+** segno (segno più) e modificare il nome generato da Razor PagesMovie.**Modelli**. Razor Da PagesMovieContext a Razor PagesMovie.**Dati**. Razor PagesMovieContext. [Questa modifica](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html) non è obbligatoria. Crea la classe del contesto di database con lo spazio dei nomi corretto.
* Selezionare **Aggiungi**.

![Immagine relativa alle istruzioni precedenti.](model/_static/3/arp.png)

Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* Aprire una finestra di comando nella directory del progetto che contiene i file *Program.cs*, *Startup.cs* e *. csproj* .

* **Per Windows**: eseguire il comando seguente:

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **Per MacOS e Linux**: eseguire il comando seguente:

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> La tabella seguente illustra in dettaglio le opzioni del generatore di codice ASP.NET Core:

| Opzione               | Descrizione|
| ----------------- | ------------ |
| `-m`  | Nome del modello. |
| `-dc`  | Classe `DbContext` da usare. |
| `-udl` | Uso del layout predefinito. |
| `-outDir` | Percorso relativo della cartella di output per creare le viste. |
| `--referenceScriptLibraries` | Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine |

Usare l' `-h` opzione per ottenere informazioni sul `aspnet-codegenerator razorpage` comando:

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

Per ulteriori informazioni, vedere [DotNet-ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).

### <a name="use-sqlite-for-development-sql-server-for-production"></a>Usare SQLite per lo sviluppo, SQL Server per la produzione

Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo. Il codice seguente illustra come inserire all' <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> avvio. `IWebHostEnvironment` viene inserito in modo `ConfigureServices` che possa usare SQLite per lo sviluppo e la SQL Server nell'ambiente di produzione.

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Creare una cartella *Pages/Movies*:

* Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.
* Denominare la cartella *Movies*.

Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **Nuova impalcatura...**.

![Immagine relativa alle istruzioni precedenti.](model/_static/scaMac.png)

Nella finestra di dialogo **Nuova impalcatura** selezionare **Razor pagine con Entity Framework (CRUD)** > **Avanti**.

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffoldMac.png)

Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :

* Nell'elenco a discesa **classe modello** selezionare o digitare **Movie ( Razor PagesMovie. Models)**.
* Nella riga della **classe del contesto dati** Digitare il nome della nuova classe, Razor PagesMovie.**Dati**. Razor PagesMovieContext. [Questa modifica](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html) non è obbligatoria. Crea la classe del contesto di database con lo spazio dei nomi corretto.
* Selezionare **Aggiungi**.

![Immagine relativa alle istruzioni precedenti.](model/_static/arpMac.png)

Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.

### <a name="add-ef-tools"></a>Aggiungi strumenti EF

Eseguire il comando interfaccia della riga di comando di .NET Core seguente:

```dotnetcli
dotnet tool install --global dotnet-ef
```

Il comando precedente aggiunge gli strumenti Entity Framework Core per l'interfaccia della riga di comando di .NET Core. Per ulteriori informazioni, vedere [riferimento agli strumenti Entity Framework Core-interfaccia della riga di comando di .NET Core](/ef/core/miscellaneous/cli/dotnet).

### <a name="use-sqlite-for-development-sql-server-for-production"></a>Usare SQLite per lo sviluppo, SQL Server per la produzione

Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo. Il codice seguente illustra come inserire all' <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> avvio. `IWebHostEnvironment` viene inserito in modo `ConfigureServices` che possa usare SQLite per lo sviluppo e la SQL Server nell'ambiente di produzione.

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

---

### <a name="files-created"></a>File creati

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Il processo di scaffolding crea e aggiorna i file seguenti:

* *Pagine/filmati*: creare, eliminare, Details, Edit e Index .
* *Data/ Razor PagesMovieContext.cs*

### <a name="updated"></a>Aggiornato

* *Startup.cs*

I file creati e aggiornati sono illustrati nella sezione successiva.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Il processo di scaffolding crea e aggiorna i file seguenti:

* *Pagine/filmati*: creare, eliminare, Details, Edit e Index .
* *Data/ Razor PagesMovieContext.cs*

### <a name="updated"></a>Aggiornato

* *Startup.cs*

I file creati e aggiornati sono illustrati nella sezione successiva.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Il processo di scaffolding crea i file seguenti:

* *Pagine/filmati*: creare, eliminare, Details, Edit e Index .

I file creati sono illustrati nella sezione successiva.

---

<a name="pmc"></a>

## <a name="initial-migration"></a>Migrazione iniziale

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

In questa sezione viene usata la Console di Gestione pacchetti (PMC) per:

* Aggiungere una migrazione iniziale.
* Aggiornare il database con la migrazione iniziale.

Nel menu **Strumenti** selezionare **Gestione pacchetti NuGet** > **Console di Gestione pacchetti**.

  ![Menu della Console di Gestione pacchetti](../first-mvc-app/adding-model/_static/pmc.png)

Nella Console di Gestione pacchetti immettere i comandi seguenti:

```powershell
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

Eseguire i seguenti comandi dell'interfaccia della riga di comando di .NET Core:

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

I comandi precedenti generano l'avviso seguente: "nessun tipo specificato per la colonna decimale ' Price ' nel tipo di entità' Movie '. This will cause values to be silently truncated if they do not fit in the default precision and scale. (I valori saranno quindi automaticamente troncati se non rispettano la precisione e la scala predefinite). Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'. (Specificare in modo esplicito il tipo di colonna di SQL Server che può supportare tutti i valori usando 'HasColumnType()')"

Ignorare l'avviso poiché verrà risolto in un passaggio successivo.

Il comando Migrations genera il codice per creare lo schema del database iniziale. Lo schema è basato sul modello specificato in `DbContext` . L'argomento `InitialCreate` viene usato per denominare le migrazioni. È possibile usare qualsiasi nome, ma per convenzione viene selezionato un nome che descrive la migrazione.

Il `update` comando esegue il `Up` Metodo nelle migrazioni che non sono state applicate. In tal caso, `update` esegue il `Up` metodo nel file  *migrations/ \<time-stamp> _InitialCreate. cs* , che crea il database.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>Esaminare il contesto registrato con l'inserimento di dipendenze

ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection). I servizi, ad esempio il contesto del contesto del database EF Core, vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione. I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore. Il codice del costruttore che ottiene un'istanza del contesto del contesto del database viene illustrato più avanti nell'esercitazione.

Lo strumento di ponteggi crea automaticamente un contesto del contesto del database e lo registra con il contenitore di inserimento delle dipendenze.

Esaminare il metodo `Startup.ConfigureServices`. La riga evidenziata è stata aggiunta dallo scaffolder:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

Le `RazorPagesMovieContext` coordinate EF Core funzionalità, ad esempio creazione, lettura, aggiornamento ed eliminazione, per il `Movie` modello. Il contesto dei dati (`RazorPagesMovieContext`) è derivato da [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext). Il contesto dei dati specifica le entità incluse nel modello di dati.

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

Il codice precedente crea una [proprietà \<Movie> DbSet](/dotnet/api/microsoft.entityframeworkcore.dbset-1) per il set di entità. Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database. Un'entità corrisponde a una riga nella tabella.

Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions). Per lo sviluppo locale, il [sistema di configurazione](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Esaminare il metodo `Up`.

---

<a name="test"></a>

### <a name="test-the-app"></a>Testare l'app

* Eseguire l'app e accodare `/Movies` all'URL nel browser (`http://localhost:port/movies`).

Se viene visualizzato l'errore:

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

Non è stato eseguita la [migrazione](#pmc).

* Eseguire il test del collegamento **Crea**.

  ![Pagina di creazione](model/_static/conan5.png)

  > [!NOTE]
  > Potrebbe non essere possibile immettere virgole decimali nel campo `Price`. Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per impostazioni locali diverse dall'inglese che usano la virgola (",") come separatore decimale e per formati di data diversi da quello dell'inglese (Stati Uniti), è necessario localizzare l'app. Per istruzioni sulla globalizzazione, vedere [questo problema su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).

* Testare i collegamenti **Modifica**, **Dettagli** ed **Elimina**.

L'esercitazione successiva illustra i file creati tramite scaffolding.

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: introduzione](xref:tutorials/razor-pages/razor-pages-start) 
>  Passaggio [successivo: impalcatura Razor Pagine](xref:tutorials/razor-pages/page) di

::: moniker-end

<!--  ::: moniker previous version   -->
::: moniker range="< aspnetcore-3.0"

In questa sezione vengono aggiunte le classi per la gestione dei film in un [database SQLite](https://www.sqlite.org/index.html)multipiattaforma. Le app create da un modello di ASP.NET Core usano un database SQLite. Le classi modello dell'app vengono usate con [Entity Framework Core (EF Core)](/ef/core) ([SQLite EF Core provider di database](/ef/core/providers/sqlite)) per lavorare con il database. EF Core è un framework ORM (Object-Relational Mapping) che semplifica l'accesso ai dati.

Le classi di modello sono dette classi POCO (da "plain-old CLR objects") perché non hanno alcuna dipendenza in EF Core. Definiscono le proprietà dei dati archiviati nel database.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).

## <a name="add-a-data-model"></a>Aggiungere un modello di dati

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Fare clic con il pulsante destro del mouse sul progetto **Razor PagesMovie** > **Aggiungi**  >  **nuova cartella**. Assegnare il nome *Modelli* alla cartella.

Fare clic con il pulsante destro del mouse sulla cartella *modelli* . Selezionare **Aggiungi**  >  **classe**. Denominare la classe **Movie**.

Aggiungere le proprietà seguenti alla classe `Movie`:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aggiungere una cartella denominata *Modelli*.
* Aggiungere una classe alla cartella *Modello* denominata *Movie.cs*.

Aggiungere le proprietà seguenti alla classe `Movie`:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a>Aggiungere i pacchetti NuGet e gli strumenti EF

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

### <a name="add-a-database-context-class"></a>Aggiungere una classe di contesto dei dati

Nel Razor progetto PagesMovie creare una nuova cartella denominata *Data*. 
Aggiungere la classe `RazorPagesMovieContext` seguente alla cartella *Data*:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

Il codice precedente crea una proprietà `DbSet` per il set di entità. Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella. Il codice non verrà compilato fino a quando le dipendenze non verranno aggiunte in un passaggio successivo.

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a>Aggiungere una stringa di connessione del database

Aggiungere una stringa di connessione al *appsettings.json* file come illustrato nel codice evidenziato seguente:

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

### <a name="add-required-nuget-packages"></a>Aggiungere i pacchetti NuGet necessari

Eseguire il comando interfaccia della riga di comando di .NET Core seguente per aggiungere SQLite e CodeGeneration. Design al progetto:

```dotnetcli
dotnet add package Microsoft.EntityFrameworkCore.Sqlite --version 2.2.6
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --version 2.2.4
dotnet add package Microsoft.EntityFrameworkCore.Design --version 2.2.6
```

Il pacchetto `Microsoft.VisualStudio.Web.CodeGeneration.Design` è necessario per lo scaffolding.

<a name="reg"></a>

### <a name="register-the-database-context"></a>Registrare il contesto del database

Aggiungere le istruzioni `using` seguenti all'inizio di *Startup.cs*:

```csharp
using RazorPagesMovie.Models;
using Microsoft.EntityFrameworkCore;
```

Registrare il contesto del database con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) in `Startup.ConfigureServices`.

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

Compilare il progetto per il controllo di eventuali errori.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Nella **finestra degli strumenti della soluzione**, fare clic sul progetto *Razor PagesMovie* e quindi selezionare **Aggiungi**  >  **nuova cartella**. Assegnare il nome *Modelli* alla cartella.
* CTRL: fare clic sulla cartella *modelli* , quindi selezionare **Aggiungi** > **nuovo file**.
* Nel finestra di dialogo **Nuovo file**:

  * Selezionare **Generale** nel riquadro a sinistra.
  * Selezionare **Classe vuota** nel riquadro centrale.
  * Denominare la classe **Filmato** e selezionare **Nuovo**.

Aggiungere le proprietà seguenti alla classe `Movie`:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contiene:

* Il campo `ID` è richiesto dal database per la chiave primaria.
* `[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ). Con questo attributo:

  * l'utente non deve immettere le informazioni temporali nel campo della data.
  * Viene visualizzata solo la data, non le informazioni temporali.

L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.

---

Compilare il progetto per verificare che non siano presenti errori di compilazione.

## <a name="scaffold-the-movie-model"></a>Eseguire lo scaffolding del modello di filmato

In questa sezione viene eseguito lo scaffolding del modello *Movie*. Lo strumento di scaffolding crea quindi le pagine per le operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) per il modello *Movie*.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Creare una cartella *Pages/Movies*:

* Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.
* Denominare la cartella *Movies*.

Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **nuovo elemento con impalcatura**.

![Immagine relativa alle istruzioni precedenti.](model/_static/sca.png)

Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffold.png)

Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :
<!-- In the next section, change 
(plus) sign and accept the generated name 
to use Data, it should not use models. That will make the namespace the same for the VS version and the CLI version
-->

* Nell'elenco a discesa **classe modello** selezionare **Movie ( Razor PagesMovie. Models)**.
* Nella riga della **classe del contesto dati** selezionare il **+** segno (più) e accettare il nome generato **Razor PagesMovie. Models. Razor PagesMovieContext**.
* Selezionare **Aggiungi**.

![Immagine relativa alle istruzioni precedenti.](model/_static/arp.png)

Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* Aprire una finestra di comando nella directory del progetto che contiene i file *Program.cs*, *Startup.cs* e *. csproj* .

* **Per Windows**: eseguire il comando seguente:

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* **Per MacOS e Linux**: eseguire il comando seguente:

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> La tabella seguente illustra in dettaglio le opzioni del generatore di codice ASP.NET Core:

| Opzione               | Descrizione|
| ----------------- | ------------ |
| `-m`  | Nome del modello. |
| `-dc`  | Classe `DbContext` da usare. |
| `-udl` | Uso del layout predefinito. |
| `-outDir` | Percorso relativo della cartella di output per creare le viste. |
| `--referenceScriptLibraries` | Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine |

Usare l' `-h` opzione per ottenere informazioni sul `aspnet-codegenerator razorpage` comando:

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

Per ulteriori informazioni, vedere [DotNet-ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Creare una cartella *Pages/Movies*:

* CTRL: fare clic sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.
* Denominare la cartella *Movies*.

Control: fare clic sulla cartella *pages/Movies* > **aggiungere** un > **nuovo elemento con impalcatura**.

![Immagine relativa alle istruzioni precedenti.](model/_static/scaMac.png)

Nella finestra di dialogo **Aggiungi nuova impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffoldMac.png)

Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :

* Nell'elenco a discesa **classe modello** selezionare o digitare **Movie**.
* Nella riga della **classe del contesto dati** Digitare select the **Razor PagesMovieContext** . verrà creata una nuova classe del contesto di database con lo spazio dei nomi corretto. In questo caso sarà **Razor PagesMovie. Models. Razor PagesMovieContext**.
* Selezionare **Aggiungi**.

![Immagine relativa alle istruzioni precedenti.](model/_static/arpMac.png)

Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.

---

Il processo di scaffolding crea e aggiorna i file seguenti:

### <a name="files-created"></a>File creati

* *Pagine/filmati*: creare, eliminare, Details, Edit e Index .
* *Data/ Razor PagesMovieContext.cs*

### <a name="file-updated"></a>File aggiornato

* *Startup.cs*

I file creati e aggiornati sono illustrati nella sezione successiva.

<a name="pmc"></a>

## <a name="initial-migration"></a>Migrazione iniziale

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

In questa sezione viene usata la Console di Gestione pacchetti (PMC) per:

* Aggiungere una migrazione iniziale.
* Aggiornare il database con la migrazione iniziale.

Nel menu **Strumenti** selezionare **Gestione pacchetti NuGet** > **Console di Gestione pacchetti**.

  ![Menu della Console di Gestione pacchetti](../first-mvc-app/adding-model/_static/pmc.png)

Nella Console di Gestione pacchetti immettere i comandi seguenti:

```powershell
Add-Migration Initial
Update-Database
```

Il comando `Add-Migration` genera un codice per creare lo schema del database iniziale. Lo schema è basato sul modello specificato in `DbContext` , nel file *Razor PagesMovieContext.cs* . L' `InitialCreate` argomento viene usato per assegnare un nome alla migrazione. È possibile usare qualsiasi nome, ma per convenzione viene usato un nome che descrive la migrazione. Per altre informazioni, vedere <xref:data/ef-mvc/migrations>.

Il `Update-Database` comando esegue il `Up` metodo nel file *migrations/ \<time-stamp> _InitialCreate. cs* . Il metodo `Up` crea il database.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

Eseguire i seguenti comandi dell'interfaccia della riga di comando di .NET Core:

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

I comandi precedenti generano l'avviso seguente: "nessun tipo specificato per la colonna decimale ' Price ' nel tipo di entità' Movie '. This will cause values to be silently truncated if they do not fit in the default precision and scale. (I valori saranno quindi automaticamente troncati se non rispettano la precisione e la scala predefinite). Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'. (Specificare in modo esplicito il tipo di colonna di SQL Server che può supportare tutti i valori usando 'HasColumnType()')"

Ignorare l'avviso poiché verrà risolto in un passaggio successivo.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a>Esaminare il contesto registrato con l'inserimento di dipendenze

ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection). I servizi (ad esempio il contesto del contesto del database EF Core) vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione. I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore. Il codice del costruttore che ottiene un'istanza del contesto contextB del contesto di database viene illustrato più avanti nell'esercitazione.

Lo strumento di ponteggi crea automaticamente un contesto del contesto del database e lo registra con il contenitore di inserimento delle dipendenze.

Esaminare il metodo `Startup.ConfigureServices`. La riga evidenziata è stata aggiunta dallo scaffolder:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-18)]

Le `RazorPagesMovieContext` coordinate EF Core funzionalità, ad esempio creazione, lettura, aggiornamento ed eliminazione, per il `Movie` modello. Il contesto dei dati (`RazorPagesMovieContext`) è derivato da [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext). Il contesto dei dati specifica le entità incluse nel modello di dati.

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Data/RazorPagesMovieContext.cs)]

Il codice precedente crea una [proprietà \<Movie> DbSet](/dotnet/api/microsoft.entityframeworkcore.dbset-1) per il set di entità. Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database. Un'entità corrisponde a una riga nella tabella.

Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions). Per lo sviluppo locale, il [sistema di configurazione](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Esaminare il metodo `Up`.

---

<a name="test"></a>

### <a name="test-the-app"></a>Testare l'app

* Eseguire l'app e accodare `/Movies` all'URL nel browser (`http://localhost:port/movies`).

Se viene visualizzato l'errore:

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

Non è stato eseguita la [migrazione](#pmc).

* Eseguire il test del collegamento **Crea**.

  ![Pagina di creazione](model/_static/conan.png)

  > [!NOTE]
  > Potrebbe non essere possibile immettere virgole decimali nel campo `Price`. Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per impostazioni locali diverse dall'inglese che usano la virgola (",") come separatore decimale e per formati di data diversi da quello dell'inglese (Stati Uniti), è necessario localizzare l'app. Per istruzioni sulla globalizzazione, vedere [questo problema su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).

* Testare i collegamenti **Modifica**, **Dettagli** ed **Elimina**.

L'esercitazione successiva illustra i file creati tramite scaffolding.

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: introduzione](xref:tutorials/razor-pages/razor-pages-start) 
>  Passaggio [successivo: impalcatura Razor Pagine](xref:tutorials/razor-pages/page) di

::: moniker-end
