---
title: Parte 4, utilizzo di un database
author: rick-anderson
description: Parte 4 della serie di esercitazioni sulle Razor pagine.
ms.author: riande
ms.date: 01/05/2021
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
uid: tutorials/razor-pages/sql
ms.openlocfilehash: fa1060ae1a046a40d55e9fef4a094aa9e51a18af
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101171"
---
# <a name="part-4-of-tutorial-series-on-razor-pages"></a>Parte 4 della serie di esercitazioni sulle Razor pagine

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Joe Audette](https://twitter.com/joeaudette)

::: moniker range=">= aspnetcore-5.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).

L'oggetto `RazorPagesMovieContext` gestisce l'attività di connessione al database e di mapping degli oggetti `Movie` ai record di database. Il contesto del database viene registrato con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) nel metodo `ConfigureServices` di *Startup.cs*:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

La chiave viene letta dal sistema di [configurazione](xref:fundamentals/configuration/index) ASP.NET Core `ConnectionString` . Per lo sviluppo locale, la configurazione ottiene la stringa di connessione dal *appsettings.json* file.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

La stringa di connessione generata è simile al codice JSON seguente:

[!code-json[](razor-pages-start/sample/RazorPagesMovie50/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/appsettings_SQLite.json?highlight=10-12)]

---

Quando l'app viene distribuita in un server di test o di produzione, è possibile usare una variabile di ambiente per impostare la stringa di connessione su un server di database di test o di produzione. Per ulteriori informazioni, vedere [configurazione](xref:fundamentals/configuration/index).

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di programmi. Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa. Per impostazione predefinita, Local DB crea file con estensione `*.mdf` nella directory `C:\Users\<user>\`.

<a name="ssox"></a>
1. Dal menu **Visualizzazione** aprire **Esplora oggetti di SQL Server** (SSOX).

   ![Menu Visualizza](sql/_static/5/ssox.png)

1. Fare clic con il pulsante destro del mouse sulla `Movie` tabella e scegliere **Visualizza finestra di progettazione**:

   ![Menu di scelta rapida aperti per la tabella Movie](sql/_static/5/design.png)

   ![Tabelle Movie aperte nella finestra di progettazione](sql/_static/dv.png)

   Si noti l'icona a forma di chiave accanto a `ID`. Per impostazione predefinita, Entity Framework crea una proprietà denominata `ID` per la chiave primaria.

1. Fare clic con il pulsante destro del mouse sulla `Movie` tabella e selezionare **Visualizza dati**:

   ![Tabella Movie aperta con i dati della tabella](sql/_static/vd22.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

## <a name="sqlite"></a>SQLite

Nel sito Web di [SQLite](https://www.sqlite.org/) si dichiara:

> SQLite è un motore di database SQL autonomo, a elevata affidabilità, incorporato, completo e di dominio pubblico. SQLite è il motore di database più usato in tutto il mondo.

Sono disponibili molti strumenti di terze parti che è possibile scaricare per gestire e visualizzare un database SQLite. L'immagine seguente è stata tratta da [DB Browser per SQLite](https://sqlitebrowser.org/). Se si preferisce uno strumento SQLite in particolare, aggiungere un commento sui motivi della preferenza.

![Browser DB per SQLite con database di film](~/tutorials/first-mvc-app-xplat/working-with-sql/_static/dbb.png)

> [!NOTE]
> Per questa esercitazione, viene usata la funzionalità *migrazioni* di Entity Framework Core, laddove possibile. Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati. Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate. Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata. Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo. A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite. Al contrario, quando lo schema viene modificato, il database viene eliminato e ricreato.
>
>La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella. La ricompilazione della tabella comporta:
>
>* Creazione di una nuova tabella.
>* Copia dei dati dalla vecchia tabella alla nuova tabella.
>* Eliminazione della tabella precedente.
>* Ridenominazione della nuova tabella.
>
>Per altre informazioni, vedere le risorse seguenti:
> * [Limitazioni del provider di database SQLite per EF Core](/ef/core/providers/sqlite/limitations)
> * [Personalizzare il codice di migrazione](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [Seeding dei dati](/ef/core/modeling/data-seeding)
> * [Istruzione ALTER TABLE di SQLite](https://sqlite.org/lang_altertable.html)

---

## <a name="seed-the-database"></a>Specificare il valore di inizializzazione del database

Creare una nuova classe denominata `SeedData` nella cartella *Models* usando il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedData.cs?name=snippet_1)]

Se sono presenti film nel database, l'inizializzatore del valore di inizializzazione viene restituito e non viene aggiunto alcun film.

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>Aggiungere l'inizializzatore del valore di inizializzazione

Sostituire il contenuto di *Program.cs* con il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Program.cs)]

Nel codice precedente, il `Main` metodo è stato modificato per eseguire le operazioni seguenti:

* Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.
* Chiamare il `seedData.Initialize` metodo passando all'istanza del contesto di database.
* Eliminare il contesto dopo che il metodo di inizializzazione è stato completato. L' [istruzione using](/dotnet/csharp/language-reference/keywords/using-statement) garantisce che il contesto venga eliminato.

Quando `Update-Database` non è stato eseguito, si verifica l'eccezione seguente:

> `SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.`
> `Login failed for user 'user name'.`

### <a name="test-the-app"></a>Testare l'app

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Eliminare tutti i record nel database. Usare i collegamenti di eliminazione nel browser o da [SSOX](xref:tutorials/razor-pages/new-field#ssox)

1. Forzare l'inizializzazione dell'app chiamando i metodi nella `Startup` classe, in modo che venga eseguito il metodo di inizializzazione. Per forzare l'inizializzazione, IIS Express deve essere arrestato e riavviato. Arrestare e riavviare IIS con uno degli approcci seguenti:

   1. Fare clic con il pulsante destro del mouse sull'icona della barra di sistema IIS Express nell'area di notifica e selezionare **Esci** o **Arresta sito**:

      ![Icona dell'area di notifica di IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

      ![Menu di scelta rapida](sql/_static/stopIIS.png)

   1. Se l'app è in esecuzione in modalità non di debug, premere <kbd>F5</kbd> per l'esecuzione in modalità di debug.
   1. Se l'app è in modalità di debug, arrestare il debugger e premere <kbd>F5</kbd>.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed. Arrestare e avviare l'app per inizializzare il database.

---

L'app visualizza i dati sottoposti a seed:

![Applicazione film aperta nel browser che mostra i dati dei film](sql/_static/5/m55.png)

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: con ponteggi Razor Pagine](xref:tutorials/razor-pages/page) 
>  [successive: aggiornare le pagine](xref:tutorials/razor-pages/da1)

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).

L'oggetto `RazorPagesMovieContext` gestisce l'attività di connessione al database e di mapping degli oggetti `Movie` ai record di database. Il contesto del database viene registrato con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) nel metodo `ConfigureServices` di *Startup.cs*:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

La chiave viene letta dal sistema di [configurazione](xref:fundamentals/configuration/index) ASP.NET Core `ConnectionString` . Per lo sviluppo locale, la configurazione ottiene la stringa di connessione dal *appsettings.json* file.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

La stringa di connessione generata sarà simile alla seguente:

[!code-json[](razor-pages-start/sample/RazorPagesMovie30/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/appsettings_SQLite.json?highlight=10-12)]

---

Quando l'app viene distribuita in un server di test o di produzione, è possibile usare una variabile di ambiente per impostare la stringa di connessione su un server di database di test o di produzione. Per altre informazioni, vedere [Configurazione](xref:fundamentals/configuration/index).

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di programmi. Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa. Per impostazione predefinita, Local DB crea file con estensione `*.mdf` nella directory `C:\Users\<user>\`.

<a name="ssox"></a>
* Dal menu **Visualizzazione** aprire **Esplora oggetti di SQL Server** (SSOX).

  ![Menu Visualizza](sql/_static/ssox.png)

* Fare clic con il pulsante destro del mouse sulla `Movie` tabella e scegliere **Visualizza finestra di progettazione**:

  ![Menu di scelta rapida aperti per la tabella Movie](sql/_static/design.png)

  ![Tabelle Movie aperte nella finestra di progettazione](sql/_static/dv.png)

Si noti l'icona a forma di chiave accanto a `ID`. Per impostazione predefinita, Entity Framework crea una proprietà denominata `ID` per la chiave primaria.

* Fare clic con il pulsante destro del mouse sulla `Movie` tabella e selezionare **Visualizza dati**:

  ![Tabella Movie aperta con i dati della tabella](sql/_static/vd22.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

## <a name="sqlite"></a>SQLite

Nel sito Web di [SQLite](https://www.sqlite.org/) si dichiara:

> SQLite è un motore di database SQL autonomo, a elevata affidabilità, incorporato, completo e di dominio pubblico. SQLite è il motore di database più usato in tutto il mondo.

Sono disponibili molti strumenti di terze parti che è possibile scaricare per gestire e visualizzare un database SQLite. L'immagine seguente è stata tratta da [DB Browser per SQLite](https://sqlitebrowser.org/). Se si preferisce uno strumento SQLite in particolare, aggiungere un commento sui motivi della preferenza.

![Browser DB per SQLite con database di film](~/tutorials/first-mvc-app-xplat/working-with-sql/_static/dbb.png)

> [!NOTE]
> Per questa esercitazione, viene usata la funzionalità *migrazioni* di Entity Framework Core, laddove possibile. Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati. Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate. Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata. Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo. A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite. Al contrario, quando lo schema viene modificato, il database viene eliminato e ricreato.
>
>La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella. La ricompilazione della tabella comporta:
>
>* Creazione di una nuova tabella.
>* Copia dei dati dalla vecchia tabella alla nuova tabella.
>* Eliminazione della tabella precedente.
>* Ridenominazione della nuova tabella.
>
>Per altre informazioni, vedere le risorse seguenti:
> * [Limitazioni del provider di database SQLite per EF Core](/ef/core/providers/sqlite/limitations)
> * [Personalizzare il codice di migrazione](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [Seeding dei dati](/ef/core/modeling/data-seeding)
> * [Istruzione ALTER TABLE di SQLite](https://sqlite.org/lang_altertable.html)

---

## <a name="seed-the-database"></a>Specificare il valore di inizializzazione del database

Creare una nuova classe denominata `SeedData` nella cartella *Models* usando il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedData.cs?name=snippet_1)]

Se sono presenti film nel database, l'inizializzatore del valore di inizializzazione viene restituito e non viene aggiunto alcun film.

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>Aggiungere l'inizializzatore del valore di inizializzazione

Sostituire il contenuto di *Program.cs* con il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Program.cs)]

Nel codice precedente, il `Main` metodo è stato modificato per eseguire le operazioni seguenti:

* Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.
* Chiamare il `seedData.Initialize` metodo passando all'istanza del contesto di database.
* Eliminare il contesto dopo che il metodo di inizializzazione è stato completato. L' [istruzione using](/dotnet/csharp/language-reference/keywords/using-statement) garantisce che il contesto venga eliminato.

Quando `Update-Database` non è stato eseguito, si verifica l'eccezione seguente:

> `SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.`
> `Login failed for user 'user name'.`

### <a name="test-the-app"></a>Testare l'app

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Eliminare tutti i record nel database. Usare i collegamenti di eliminazione nel browser o da [SSOX](xref:tutorials/razor-pages/new-field#ssox).
* Forzare l'inizializzazione dell'app chiamando i metodi nella `Startup` classe, in modo che venga eseguito il metodo di inizializzazione. Per forzare l'inizializzazione, IIS Express deve essere arrestato e riavviato. Arrestare e riavviare IIS con uno degli approcci seguenti:

  * Fare clic con il pulsante destro del mouse sull'icona dell'area di notifica di IIS Express e toccare **Esci** o **Arresta sito**:

    ![Icona dell'area di notifica di IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![Menu di scelta rapida](sql/_static/stopIIS.png)

    * Se l'app è in esecuzione in modalità non di debug, premere <kbd>F5</kbd> per l'esecuzione in modalità di debug.
    * Se l'app è in modalità di debug, arrestare il debugger e premere <kbd>F5</kbd>.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed. Arrestare e avviare l'app per inizializzare il database.

---

L'app visualizza i dati sottoposti a seed:

![App per i film aperta in Chrome con i dati sui film](sql/_static/m55https.png)

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: con ponteggi Razor Pagine](xref:tutorials/razor-pages/page) 
>  [successive: aggiornare le pagine](xref:tutorials/razor-pages/da1)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).

L'oggetto `RazorPagesMovieContext` gestisce l'attività di connessione al database e di mapping degli oggetti `Movie` ai record di database. Il contesto del database viene registrato con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) nel metodo `ConfigureServices` di *Startup.cs*:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-17)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

---

Per altre informazioni sui metodi usati in `ConfigureServices`, vedere:

* [Supporto per il Regolamento generale sulla protezione dei dati (GDPR) dell'Unione Europea in ASP.NET Core](xref:security/gdpr) per `CookiePolicyOptions`.
* [SetCompatibilityVersion](xref:mvc/compatibility-version)

La chiave viene letta dal sistema di [configurazione](xref:fundamentals/configuration/index) ASP.NET Core `ConnectionString` . Per lo sviluppo locale, la configurazione ottiene la stringa di connessione dal *appsettings.json* file.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

La stringa di connessione generata sarà simile alla seguente:

[!code-json[](razor-pages-start/sample/RazorPagesMovie22/appsettings.json?highlight=8-10)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

---

Quando l'app viene distribuita in un server di test o di produzione, è possibile usare una variabile di ambiente per impostare la stringa di connessione su un server di database di test o di produzione. Per altre informazioni, vedere [Configurazione](xref:fundamentals/configuration/index).

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di programmi. Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa. Per impostazione predefinita, Local DB crea file con estensione `*.mdf` nella directory `C:/Users/<user/>`.

<a name="ssox"></a>
* Dal menu **Visualizzazione** aprire **Esplora oggetti di SQL Server** (SSOX).

  ![Menu Visualizza](sql/_static/ssox.png)

* Fare clic con il pulsante destro del mouse sulla `Movie` tabella e scegliere **Visualizza finestra di progettazione**:

  ![Menu di scelta rapida aperto per la tabella Movie](sql/_static/design.png)

  ![Tabella Movie aperta nella finestra di progettazione](sql/_static/dv.png)

Si noti l'icona a forma di chiave accanto a `ID`. Per impostazione predefinita, Entity Framework crea una proprietà denominata `ID` per la chiave primaria.

* Fare clic con il pulsante destro del mouse sulla `Movie` tabella e selezionare **Visualizza dati**:

  ![Tabella Movie aperta con i dati della tabella](sql/_static/vd22.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

---

## <a name="seed-the-database"></a>Specificare il valore di inizializzazione del database

Creare una nuova classe denominata `SeedData` nella cartella *Models* usando il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedData.cs?name=snippet_1)]

Se sono presenti film nel database, l'inizializzatore del valore di inizializzazione viene restituito e non viene aggiunto alcun film.

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a>Aggiungere l'inizializzatore del valore di inizializzazione

Sostituire il contenuto di *Program.cs* con il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Program.cs)]

Nel codice precedente, il `Main` metodo è stato modificato per eseguire le operazioni seguenti:

* Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.
* Chiamare il `seedData.Initialize` metodo passando all'istanza del contesto di database.
* Eliminare il contesto dopo che il metodo di inizializzazione è stato completato. L' [istruzione using](/dotnet/csharp/language-reference/keywords/using-statement) garantisce che il contesto venga eliminato.

Un'app di produzione non chiamerà `Database.Migrate`. Sarà aggiunto al codice precedente per evitare l'eccezione seguente quando `Update-Database` non è stato eseguito:

SqlException: Impossibile aprire il database " Razor PagesMovieContext-21" richiesto dall'account di accesso. Accesso non riuscito.
Accesso non riuscito per l'utente "nome-utente".

### <a name="test-the-app"></a>Testare l'app

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Eliminare tutti i record nel database. Questa operazione può essere eseguita con i collegamenti di eliminazione nel browser o da [SSOX](xref:tutorials/razor-pages/new-field#ssox)
* Forzare l'inizializzazione dell'app chiamando i metodi nella `Startup` classe, in modo che venga eseguito il metodo di inizializzazione. Per forzare l'inizializzazione, IIS Express deve essere arrestato e riavviato. È possibile eseguire questa operazione adottando uno degli approcci seguenti:

  * Fare clic con il pulsante destro del mouse sull'icona dell'area di notifica di IIS Express e toccare **Esci** o **Arresta sito**:

    ![Icona dell'area di notifica di IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![Menu di scelta rapida](sql/_static/stopIIS.png)

    * Se l'app è in esecuzione in modalità non di debug, premere <kbd>F5</kbd> per l'esecuzione in modalità di debug.
    * Se l'app è in modalità di debug, arrestare il debugger e premere <kbd>F5</kbd>.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed. Arrestare e avviare l'app per inizializzare il database.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed. Arrestare e avviare l'app per inizializzare il database.

---

L'app visualizza i dati sottoposti a seed:

![App per i film aperta in Chrome con i dati sui film](sql/_static/m55https.png)

L'esercitazione successiva consentirà di pulire la presentazione dei dati.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Versione YouTube dell'esercitazione](https://youtu.be/A_5ff11sDHY)

> [!div class="step-by-step"]
> [Precedente: con ponteggi Razor Pagine](xref:tutorials/razor-pages/page) 
>  [successive: aggiornare le pagine](xref:tutorials/razor-pages/da1)

::: moniker-end