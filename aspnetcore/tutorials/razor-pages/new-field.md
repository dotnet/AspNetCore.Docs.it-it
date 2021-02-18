---
title: Parte 7, aggiungere un nuovo campo
author: rick-anderson
description: Parte 7 della serie di esercitazioni sulle Razor pagine.
ms.author: riande
ms.custom: mvc, contperf-fy21q2
ms.date: 09/28/2020
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
uid: tutorials/razor-pages/new-field
ms.openlocfilehash: 852c26f89b229d89797d9ce6fce2983319defe9c
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101146"
---
# <a name="part-7-add-a-new-field-to-a-razor-page-in-aspnet-core"></a>Parte 7, aggiungere un nuovo campo a una Razor pagina in ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).

In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:

* Aggiungere un nuovo campo al modello.
* Eseguire la migrazione nel database della modifica al nuovo schema del campo.

Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:

* Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.
* Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.

Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.

## <a name="adding-a-rating-property-to-the-movie-model"></a>Aggiunta di una proprietà Rating al modello Movie

1. Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. Compilare l'app.

1. Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie50/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. Aggiornare le pagine seguenti:
   1. Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).
   1. Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Create.cshtml) con un campo `Rating`.
   1. Aggiungere il campo `Rating` alla pagina Edit (Modifica).

L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo. L'esecuzione dell'app senza un aggiornamento del database genera un'eccezione `SqlException` :

`SqlException: Invalid column name 'Rating'.`

L' `SqlException` eccezione è causata dal fatto che la classe del modello di film aggiornata è diversa dallo schema della tabella dei film del database. Non è presente alcuna `Rating` colonna nella tabella di database.

Per correggere questo errore, esistono alcuni approcci:

1. Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello. Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme. Lo svantaggio è che si perdono i dati esistenti nel database. Non usare questo approccio in un database di produzione. L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.

2. Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello. Il vantaggio di questo approccio consiste nel preservare i dati. Apportare questa modifica manualmente o creando uno script di modifica del database.

3. Usare Migrazioni Code First per aggiornare lo schema del database.

Per questa esercitazione usare Migrazioni Code First.

Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna. Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).

Compilare la soluzione.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>Aggiungere una migrazione per il campo Rating

1. Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.
2. Nella Console di Gestione pacchetti immettere i comandi seguenti:

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

Il comando `Add-Migration` indica al framework di:

* Confrontare il `Movie` modello con lo `Movie` schema del database.
* Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.

Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione. È consigliabile usare un nome significativo per il file di migrazione.

Il `Update-Database` comando indica al Framework di applicare le modifiche dello schema al database e di mantenere i dati esistenti.

<a name="ssox"></a>

Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo. È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).

Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database. Per eliminare il database da SSOX:

1. Selezionare il database in SSOX.
1. Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.
1. Selezionare **Chiudi connessioni esistenti**.
1. Selezionare **OK**.
1. In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:

   ```powershell
   Update-Database
   ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>Eliminare e ricreare il database

> [!NOTE]
> Per questa esercitazione, è possibile usare la funzionalità *migrazioni* di Entity Framework Core, ove possibile. Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati. Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate. Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata. Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo. A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite. In caso di modifiche dello schema, il database viene invece eliminato e ricreato.
>
>La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella. La ricompilazione della tabella comporta:
>
>* Creazione di una nuova tabella.
>* Copia dei dati dalla vecchia tabella alla nuova tabella.
>* Eliminazione della tabella precedente.
>* Ridenominazione della nuova tabella.
>
>Per altre informazioni, vedere le risorse seguenti:
>
> * [Limitazioni del provider di database SQLite per EF Core](/ef/core/providers/sqlite/limitations)
> * [Personalizzare il codice di migrazione](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [Seeding dei dati](/ef/core/modeling/data-seeding)
> * [Istruzione ALTER TABLE di SQLite](https://sqlite.org/lang_altertable.html)

1. Eliminare la cartella della migrazione.  

1. Usare i comandi seguenti per ricreare il database.

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`. Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).

In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:

* Aggiungere un nuovo campo al modello.
* Eseguire la migrazione nel database della modifica al nuovo schema del campo.

Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:

* Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.
* Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.

Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.

## <a name="adding-a-rating-property-to-the-movie-model"></a>Aggiunta di una proprietà Rating al modello Movie

1. Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. Compilare l'app.

1. Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. Aggiornare le pagine seguenti:
   1. Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).
   1. Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml) con un campo `Rating`.
   1. Aggiungere il campo `Rating` alla pagina Edit (Modifica).

L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo. L'esecuzione dell'app senza un aggiornamento del database genera un'eccezione `SqlException` :

`SqlException: Invalid column name 'Rating'.`

L' `SqlException` eccezione è causata dal fatto che la classe del modello di film aggiornata è diversa dallo schema della tabella dei film del database. Non è presente alcuna `Rating` colonna nella tabella di database.

Per correggere questo errore, esistono alcuni approcci:

1. Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello. Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme. Lo svantaggio è che si perdono i dati esistenti nel database. Non usare questo approccio in un database di produzione. L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.

2. Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello. Il vantaggio di questo approccio consiste nel preservare i dati. Apportare questa modifica manualmente o creando uno script di modifica del database.

3. Usare Migrazioni Code First per aggiornare lo schema del database.

Per questa esercitazione usare Migrazioni Code First.

Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna. Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).

Compilare la soluzione.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>Aggiungere una migrazione per il campo Rating

1. Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.
2. Nella Console di Gestione pacchetti immettere i comandi seguenti:

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

Il comando `Add-Migration` indica al framework di:

* Confrontare il `Movie` modello con lo `Movie` schema del database.
* Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.

Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione. È consigliabile usare un nome significativo per il file di migrazione.

Il `Update-Database` comando indica al Framework di applicare le modifiche dello schema al database e di mantenere i dati esistenti.

<a name="ssox"></a>

Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo. È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).

Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database. Per eliminare il database da SSOX:

* Selezionare il database in SSOX.
* Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.
* Selezionare **Chiudi connessioni esistenti**.
* Selezionare **OK**.
* In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>Eliminare e ricreare il database

> [!NOTE]
> Per questa esercitazione, usare la funzionalità *migrazioni* di Entity Framework Core laddove possibile. Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati. Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate. Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata. Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo. A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite. In caso di modifiche dello schema, il database viene invece eliminato e ricreato.
>
>La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella. La ricompilazione della tabella comporta:
>
>* Creazione di una nuova tabella.
>* Copia dei dati dalla vecchia tabella alla nuova tabella.
>* Eliminazione della tabella precedente.
>* Ridenominazione della nuova tabella.
>
>Per altre informazioni, vedere le risorse seguenti:
>
> * [Limitazioni del provider di database SQLite per EF Core](/ef/core/providers/sqlite/limitations)
> * [Personalizzare il codice di migrazione](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [Seeding dei dati](/ef/core/modeling/data-seeding)
> * [Istruzione ALTER TABLE di SQLite](https://sqlite.org/lang_altertable.html)

1. Eliminare la cartella della migrazione.  

1. Usare i comandi seguenti per ricreare il database.

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`. Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).

In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:

* Aggiungere un nuovo campo al modello.
* Eseguire la migrazione nel database della modifica al nuovo schema del campo.

Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:

* Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.
* Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.

Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.

## <a name="adding-a-rating-property-to-the-movie-model"></a>Aggiunta di una proprietà Rating al modello Movie

Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateRating.cs?highlight=13&name=snippet)]

Compilare l'app.

Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexRating.cshtml?highlight=40-42,61-63)]

Aggiornare le pagine seguenti:

* Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).
* Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) con un campo `Rating`.
* Aggiungere il campo `Rating` alla pagina Edit (Modifica).

L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo. Se l'app viene eseguita adesso, l'app genera un'eccezione `SqlException` :

`SqlException: Invalid column name 'Rating'.`

Questo errore viene visualizzato perché la classe del modello Movie aggiornata è diversa rispetto allo schema della tabella Movie nel database. Non è presente alcuna `Rating` colonna nella tabella di database.

Per correggere questo errore, esistono alcuni approcci:

1. Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello. Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme. Lo svantaggio è che si perdono i dati esistenti nel database. Non usare questo approccio in un database di produzione. L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.

2. Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello. Il vantaggio di questo approccio consiste nel preservare i dati. Apportare questa modifica manualmente o creando uno script di modifica del database.

3. Usare Migrazioni Code First per aggiornare lo schema del database.

Per questa esercitazione usare Migrazioni Code First.

Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna. Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs).

Compilare la soluzione.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>Aggiungere una migrazione per il campo Rating

Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.
Nella Console di Gestione pacchetti immettere i comandi seguenti:

```powershell
Add-Migration Rating
Update-Database
```

Il comando `Add-Migration` indica al framework di:

* Confrontare il `Movie` modello con lo `Movie` schema del database.
* Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.

Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione. È consigliabile usare un nome significativo per il file di migrazione.

Il comando `Update-Database` indica al framework di applicare le modifiche dello schema al database.

<a name="ssox"></a>

Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo. È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).

Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database. Per eliminare il database da SSOX:

* Selezionare il database in SSOX.
* Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.
* Selezionare **Chiudi connessioni esistenti**.
* Selezionare **OK**.
* In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>Eliminare e ricreare il database

> [!NOTE]
> Per questa esercitazione, usare la funzionalità *migrazioni* di Entity Framework Core laddove possibile. Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati. Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate. Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata. Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo. A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite. In caso di modifiche dello schema, il database viene invece eliminato e ricreato.
>
>La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella. La ricompilazione della tabella comporta:
>
>* Creazione di una nuova tabella.
>* Copia dei dati dalla vecchia tabella alla nuova tabella.
>* Eliminazione della tabella precedente.
>* Ridenominazione della nuova tabella.
>
>Per altre informazioni, vedere le risorse seguenti:
>
> * [Limitazioni del provider di database SQLite per EF Core](/ef/core/providers/sqlite/limitations)
> * [Personalizzare il codice di migrazione](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [Seeding dei dati](/ef/core/modeling/data-seeding)
> * [Istruzione ALTER TABLE di SQLite](https://sqlite.org/lang_altertable.html)

Eliminare il database e usare le migrazioni per ricreare il database. Per eliminare il database, eliminare il file di database (*MvcMovie.db*). Eseguire quindi il comando `ef database update`:

```dotnetcli
dotnet ef database update
```

---

Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`. Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Versione YouTube dell'esercitazione](https://youtu.be/3i7uMxiGGR8)

> [!div class="step-by-step"]
> [Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)

::: moniker-end