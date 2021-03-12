---
title: "Esercitazione parte 5: applicare le migrazioni all'esempio di Contoso University"
description: Parte 5 della serie di esercitazioni di Contoso University. Usare la funzionalità migrazioni di EF Core per gestire le modifiche al modello di dati in un'app ASP.NET Core MVC.
author: rick-anderson
ms.author: riande
ms.custom: contperf-fy21q2
ms.date: 11/13/2020
ms.topic: tutorial
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
uid: data/ef-mvc/migrations
ms.openlocfilehash: aebbc3f29b0356c7993abd83869ab21d3613bf61
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589348"
---
# <a name="tutorial-part-5-apply-migrations-to-the-contoso-university-sample"></a>Esercitazione: parte 5, applicare le migrazioni all'esempio di Contoso University

In questa esercitazione si inizia a usare la funzionalità delle migrazioni EF Core per la gestione delle modifiche al modello di dati. Nelle esercitazioni successive si aggiungeranno altre migrazioni quando si modifica il modello di dati.

In questa esercitazione:

> [!div class="checklist"]
> * Ottenere informazioni sulle migrazioni
> * Creare una migrazione iniziale
> * Esaminare i metodi Up e Down
> * Esaminare lo snapshot del modello di dati
> * Applicare la migrazione

## <a name="prerequisites"></a>Prerequisiti

* [Ordinamento, filtro e paging](sort-filter-page.md)

## <a name="about-migrations"></a>Informazioni sulle migrazioni

Quando si sviluppa una nuova applicazione, il modello di dati cambia di frequente e, a ogni cambiamento, non è più sincronizzato con il database. La serie di esercitazioni è iniziata con la configurazione di Entity Framework per creare il database se non esiste già. Quindi, ogni volta che si modifica il modello di dati, ovvero si aggiungono, rimuovono, o modificano le classi di entità o si modifica la classe DbContext, è possibile eliminare il database. In seguito EF ne crea uno nuovo corrispondente al modello ed esegue il seeding con dati di test.

Questo metodo che consiste nel mantenere il database sincronizzato con il modello di dati funziona bene fino a quando non si distribuisce l'applicazione nell'ambiente di produzione. Quando l'applicazione è in esecuzione nell'ambiente di produzione in genere archivia dati che devono essere mantenuti ed è necessario evitare di perdere tutto ad ogni modifica, come ad esempio all'aggiunta di una colonna. La funzionalità delle migrazioni di EF Core risolve questo problema consentendo a EF Core di aggiornare lo schema del database anziché crearne uno nuovo.

Per lavorare con le migrazioni, è possibile usare la **console di gestione pacchetti** (PMC) o l'interfaccia della riga di comando.  Queste esercitazioni illustrano come usare i comandi dell'interfaccia della riga di comando. Per informazioni sulla Console di Gestione pacchetti, vedere [la fine di questa esercitazione](#pmc).

## <a name="drop-the-database"></a>Eliminare il database

Installare EF Core Tools come [strumento globale](/ef/core/miscellaneous/cli/dotnet) ed eliminare il database:

 ```dotnetcli
 dotnet tool install --global dotnet-ef
 dotnet ef database drop
 ```

Nella sezione seguente viene illustrato come eseguire i comandi della CLI.

## <a name="create-an-initial-migration"></a>Creare una migrazione iniziale

Salvare le modifiche e compilare il progetto. Aprire una finestra di comando e passare alla cartella del progetto. Ecco un modo rapido per eseguire questa operazione:

* In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto e scegliere **Apri cartella in Esplora File** dal menu di scelta rapida.

  ![Voce di menu Apri in Esplora file](migrations/_static/open-in-file-explorer.png)

* Immettere "cmd" nella barra degli indirizzi e premere INVIO.

  ![Finestra di comando](migrations/_static/open-command-window.png)

Immettere il comando seguente nella finestra Comando:

```dotnetcli
dotnet ef migrations add InitialCreate
```

Nei comandi precedenti viene visualizzato un output simile al seguente:

```console
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
Done. To undo this action, use 'ef migrations remove'
```

Se viene visualizzato un messaggio di errore che *indica che non è possibile accedere al file... ContosoUniversity.dll perché è in uso da un altro processo.*", trovare l'icona IIS Express nella barra delle applicazioni di Windows e fare clic con il pulsante destro del mouse su di essa, quindi fare clic su **ContosoUniversity > arresta sito**.

## <a name="examine-up-and-down-methods"></a>Esaminare i metodi Up e Down

Quando è stato eseguito il comando `migrations add`, EF ha generato il codice che crea il database da zero. Questo codice si trova nella cartella *migrazioni* , nel file denominato *\<timestamp> _InitialCreate. cs*. Il metodo `Up` della classe `InitialCreate` crea le tabelle di database che corrispondono ai set di entità del modello di dati, e il metodo `Down` le elimina, come illustrato nell'esempio seguente.

[!code-csharp[](intro/samples/cu/Migrations/20170215220724_InitialCreate.cs?range=92-118)]

Le migrazioni chiamano il metodo `Up` per implementare le modifiche al modello di dati per una migrazione. Quando si immette un comando per annullare l'aggiornamento, le migrazioni chiamano il metodo `Down`.

Questo codice è per la migrazione iniziale creata al momento dell'immissione del comando `migrations add InitialCreate`. Il parametro del nome della migrazione, nell'esempio "InitialCreate", viene usato per il nome del file e può essere ciò che si vuole. È consigliabile scegliere una parola o una frase che riepiloghi cosa viene fatto nella migrazione. Ad esempio, è possibile denominare una migrazione successiva "AddDepartmentTable".

Se la migrazione iniziale è stata creata quando il database esisteva già, il codice di creazione del database viene generato ma non è necessario eseguirlo perché il database corrisponde già al modello di dati. Quando si distribuisce l'app in un altro ambiente in cui il database non esiste ancora, questo codice verrà eseguito per creare il database, è consigliabile quindi testarlo prima. Ecco perché in precedenza è stato modificato il nome del database nella stringa di connessione: per far sì che le migrazioni possano crearne uno nuovo da zero.

## <a name="the-data-model-snapshot"></a>Snapshot del modello di dati

Le migrazioni creano uno *snapshot* dello schema del database corrente in *Migrations/SchoolContextModelSnapshot.cs*. Quando si aggiunge una migrazione, EF determina le modifiche apportate confrontando il modello di dati con il file dello snapshot.

Usare il comando [DotNet EF Migrations Remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) per rimuovere una migrazione. `dotnet ef migrations remove` elimina la migrazione e garantisce che lo snapshot venga reimpostato correttamente. Se ha `dotnet ef migrations remove` esito negativo, usare `dotnet ef migrations remove -v` per ottenere altre informazioni sull'errore.

Per altre informazioni sull'uso del file di snapshot, vedere [EF Core Migrations in Team Environments](/ef/core/managing-schemas/migrations/teams) (Migrazioni EF Core in ambienti team).

## <a name="apply-the-migration"></a>Applicare la migrazione

Nella finestra di comando immettere il comando seguente per creare il database e le tabelle al suo interno.

```dotnetcli
dotnet ef database update
```

L'output del comando è simile al comando `migrations add`, a eccezione del fatto che vengono visualizzati i log per i comandi SQL che configurano il database. La maggior parte dei log viene omessa nell'output di esempio seguente. Se si vuole ridurre il livello di dettaglio nei messaggi di log, è possibile modificare i livelli di log nel file *appsettings.Development.json*. Per altre informazioni, vedere <xref:fundamentals/logging/index>.

```text
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (274ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      CREATE DATABASE [ContosoUniversity2];
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (60ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      IF SERVERPROPERTY('EngineEdition') <> 5
      BEGIN
          ALTER DATABASE [ContosoUniversity2] SET READ_COMMITTED_SNAPSHOT ON;
      END;
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (15ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE [__EFMigrationsHistory] (
          [MigrationId] nvarchar(150) NOT NULL,
          [ProductVersion] nvarchar(32) NOT NULL,
          CONSTRAINT [PK___EFMigrationsHistory] PRIMARY KEY ([MigrationId])
      );

<logs omitted for brevity>

info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion])
      VALUES (N'20190327172701_InitialCreate', N'5.0-rtm');
Done.
```

Per controllare il database come è stato fatto nella prima esercitazione, usare **Esplora oggetti di SQL Server**.  Si noterà l'aggiunta di una tabella \_\_EFMigrationsHistory che tiene traccia di quali migrazioni sono state applicate al database. Visualizzare i dati nella tabella: si noterà la presenza di una riga per la prima migrazione. Nell'ultimo log nell'esempio di output della CLI precedente viene visualizzata l'istruzione INSERT che crea tale riga.

Eseguire l'applicazione per verificare che tutto funzioni come prima.

![Pagina Student Index (Indice degli studenti)](migrations/_static/students-index.png)

<a id="pmc"></a>

## <a name="compare-cli-and-pmc"></a>Interfaccia della riga di comando e console di gestione pacchetto a confronto

Gli strumenti di EF per la gestione delle migrazioni sono disponibili dai comandi della CLI di .NET Core o dai cmdlet di PowerShell in Visual Studio nella finestra **Console di Gestione pacchetti**. In questa esercitazione viene illustrato come usare la CLI, ma se si preferisce è possibile usare la console di Gestione pacchetti.

I comandi di EF per la console di Gestione pacchetti sono inclusi nel pacchetto [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools). Questo pacchetto è incluso nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), quindi non è necessario aggiungere un riferimento al pacchetto se l'app dispone di un riferimento al pacchetto per `Microsoft.AspNetCore.App`.

**Importante:** non si tratta dello stesso pacchetto che si installa per l'interfaccia della riga di comando modificando il file con estensione *.csproj*. Il nome di questo pacchetto termina con `Tools`, a differenza del nome del pacchetto della CLI che termina con `Tools.DotNet`.

Per altre informazioni sui comandi della CLI, vedere [Strumenti da riga di comando di EF Core .NET](/ef/core/miscellaneous/cli/dotnet).

Per altre informazioni sui comandi della console di Gestione pacchetti, vedere [Console di Gestione pacchetti (Visual Studio)](/ef/core/miscellaneous/cli/powershell).

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples)

## <a name="next-step"></a>Passaggio successivo

Passare all'esercitazione successiva per iniziare a esaminare argomenti più avanzati sull'espansione del modello di dati. Lungo il percorso verranno create e applicate altre migrazioni.

> [!div class="nextstepaction"]
> [Creare e applicare altre migrazioni](complex-data-model.md)
