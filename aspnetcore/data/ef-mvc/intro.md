---
title: "Esercitazione: Introduzione a EF Core in un'app Web MVC ASP.NET"
description: Questa pagina è la prima di una serie di esercitazioni che illustrano come compilare l'app EF/MVC di esempio di Contoso University.
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 11/06/2020
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
uid: data/ef-mvc/intro
ms.openlocfilehash: 5bb1a05f5e81730d0785c1d6a69a8cafb68f85ab
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586683"
---
# <a name="tutorial-get-started-with-ef-core-in-an-aspnet-mvc-web-app"></a>Esercitazione: Introduzione a EF Core in un'app Web MVC ASP.NET

Di [Tom Dykstra](https://github.com/tdykstra) e [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE [RP better than MVC](~/includes/RP-EF/rp-over-mvc.md)]

L'app Web di esempio di Contoso University illustra come creare un'app Web MVC ASP.NET Core usando Entity Framework (EF) Core e Visual Studio.

L'app di esempio è un sito Web per una fittizia Contoso University. Include funzionalità, come ad esempio l'ammissione di studenti, la creazione di corsi e le assegnazioni di insegnati. Questa è la prima di una serie di esercitazioni che illustrano come compilare l'app di esempio Contoso University.

## <a name="prerequisites"></a>Prerequisiti

* Se non si ha familiarità con ASP.NET Core MVC, vedere la serie di esercitazioni [introduttive ASP.NET Core MVC](xref:tutorials/first-mvc-app/start-mvc) prima di iniziare.

[!INCLUDE[VS prereqs](~/includes/net-core-prereqs-vs-5.0.md)]

## <a name="database-engines"></a>Motori di database

Le istruzioni per Visual Studio usano [SQL Server Local DB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb), una versione di SQL Server Express eseguita solo in Windows.

<!--
The Visual Studio Code instructions use [SQLite](https://www.sqlite.org/), a cross-platform database engine.

If you choose to use SQLite, download and install a third-party tool for managing and viewing a SQLite database, such as [DB Browser for SQLite](https://sqlitebrowser.org/).
-->

## <a name="solve-problems-and-troubleshoot"></a>Risolvere i problemi e risolvere i problemi

Se si verifica un problema che non si sa come risolvere, è generalmente possibile trovare la soluzione confrontando il codice con il [progetto completato](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples). Per un elenco degli errori comuni e delle relative soluzioni, vedere [la sezione relativa alla risoluzione dei problemi nell'ultima esercitazione della serie](advanced.md#common-errors). Se non si trova la soluzione, è possibile inviare una domanda a StackOverflow.com per [ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) o [Entity Framework Core](https://stackoverflow.com/questions/tagged/entity-framework-core).

> [!TIP]
> In questa serie di 10 esercitazioni ogni esercitazione si basa su quanto viene eseguito nelle esercitazioni precedenti. È consigliabile salvare una copia del progetto dopo aver completato ogni esercitazione. Se si verificano problemi, è possibile ricominciare dall'esercitazione precedente anziché tornare all'inizio della serie.

## <a name="contoso-university-web-app"></a>App Web di Contoso University

L'applicazione compilata in queste esercitazioni è il sito Web di base di un'università.

Gli utenti possono visualizzare e aggiornare le informazioni che riguardano studenti, corsi e insegnanti. Ecco alcune schermate dell'app:

![Pagina Student Index (Indice degli studenti)](intro/_static/students-index.png)

![Pagina Students Edit (Modifica studenti)](intro/_static/student-edit.png)

## <a name="create-web-app"></a>Crea app Web

1. Avviare Visual Studio e selezionare **Crea un nuovo progetto**.
1. Nella finestra di dialogo **Crea un nuovo progetto** selezionare **ASP.NET Core applicazione Web** > **Avanti**.
1. Nella finestra di dialogo **Configura nuovo progetto** immettere `ContosoUniversity` per **nome progetto**. È importante usare questo nome esatto, incluse le maiuscole, in modo che ogni `namespace` corrisponda al momento della copia del codice.
1. Selezionare **Crea**.
1. Nella finestra di dialogo **Crea una nuova applicazione web ASP.NET Core** selezionare:
    1. **.NET Core** e **ASP.NET Core 5,0** negli elenchi a discesa.
    1. **ASP.NET Core app Web (Model-View-Controller)**.
    1.  
       Crea ![ Finestra di dialogo nuovo progetto ASP.NET Core](~/data/ef-mvc/intro/_static/new-aspnet5.png)

## <a name="set-up-the-site-style"></a>Impostare lo stile del sito

Alcune modifiche di base configurano il menu del sito, il layout e home page.

Aprire *Views/Shared/_Layout.cshtml* e apportare le modifiche seguenti:

* Modificare tutte le occorrenze di `ContosoUniversity` in `Contoso University` . Le occorrenze sono tre.
* Aggiungere le voci di menu per **About** (Informazioni su), **Students** (Studenti), **Courses** (Corsi), **Instructors** (Insegnanti) e **Departments** (Dipartimenti) ed eliminare la voce di menu **Privacy**.

Le modifiche precedenti sono evidenziate nel codice seguente:

[!code-cshtml[](intro/samples/5cu/Views/Shared/_Layout.cshtml?highlight=6,24-38,52)]

In *views/Home/index. cshtml* sostituire il contenuto del file con il markup seguente:

[!code-cshtml[](intro/samples/5cu/Views/Home/Index.cshtml)]

Premere CTRL+F5 per eseguire il progetto oppure selezionare **Debug > Avvia senza eseguire debug** dal menu. Il home page viene visualizzato con le schede per le pagine create in questa esercitazione.

![Home page di Contoso University](intro/_static/home-page5.png)

## <a name="ef-core-nuget-packages"></a>EF Core pacchetti NuGet

Questa esercitazione usa SQL Server e il pacchetto del provider è [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/).

Il pacchetto EF SQL Server e le relative dipendenze e `Microsoft.EntityFrameworkCore` `Microsoft.EntityFrameworkCore.Relational` forniscono il supporto di runtime per EF.

Aggiungere il pacchetto NuGet [Microsoft. AspNetCore. Diagnostics. EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore) e il pacchetto NuGet [Microsoft. AspNetCore. Diagnostics. EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore) . Nella console di gestione pacchetti (PMC) immettere i comandi seguenti per aggiungere i pacchetti NuGet:

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

Il `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` pacchetto NuGet fornisce ASP.NET Core middleware per EF Core pagine di errore. Questo middleware consente di rilevare e diagnosticare gli errori con EF Core migrazioni.

Per informazioni su altri provider di database disponibili per EF Core, vedere [provider di database](/ef/core/providers/).

## <a name="create-the-data-model"></a>Creare il modello di dati

Per questa app vengono create le classi di entità seguenti:

![Diagramma modello di dati Course/Enrollment/Student (Corso/Iscrizione/Studente)](intro/_static/data-model-diagram.png)

Le entità precedenti presentano le relazioni seguenti:

* Una relazione uno-a-molti tra `Student` le `Enrollment` entità e. Uno studente può essere registrato in un numero qualsiasi di corsi.
* Una relazione uno-a-molti tra `Course` le `Enrollment` entità e. Un corso può avere un numero qualsiasi di studenti iscritti.

Nelle sezioni seguenti viene creata una classe per ognuna di queste entità.

### <a name="the-student-entity"></a>Entità Student (Studente)

![Diagramma entità Student (Studente)](intro/_static/student-entity.png)

Nella cartella *Models* creare la `Student` classe con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Intro)]

La `ID` proprietà è la colonna chiave primaria (**PK**) della tabella di database che corrisponde a questa classe. Per impostazione predefinita, EF interpreta una proprietà denominata `ID` o `classnameID` come chiave primaria. È ad esempio possibile denominare il PK `StudentID` anziché `ID` .

La proprietà `Enrollments` rappresenta una [proprietà di navigazione](/ef/core/modeling/relationships). Le proprietà di navigazione contengono altre entità correlate a questa entità. `Enrollments`Proprietà di un' `Student` entità:

* Contiene tutte le `Enrollment` entità correlate a tale `Student` entità.
* Se una riga specifica del `Student` database contiene due righe correlate `Enrollment` :
  * La `Student` proprietà di `Enrollments` navigazione dell'entità contiene le due `Enrollment` entità.
  
`Enrollment` le righe contengono un valore PK dello studente nella `StudentID` colonna chiave esterna (**FK**).

Se una proprietà di navigazione può ospitare più entità:

 * Il tipo deve essere un elenco, ad esempio `ICollection<T>` , `List<T>` o `HashSet<T>` .
 * Le entità possono essere aggiunte, eliminate e aggiornate.

Le relazioni di navigazione molti-a-molti e uno-a-molti possono contenere più entità. Quando `ICollection<T>` si usa, EF crea una `HashSet<T>` raccolta per impostazione predefinita.

### <a name="the-enrollment-entity"></a>Entità Enrollment (Iscrizione)

![Diagramma entità Enrollment (Iscrizione)](intro/_static/enrollment-entity.png)

Nella cartella *Models* creare la `Enrollment` classe con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Intro)]

La `EnrollmentID` proprietà è il PK. Questa entità utilizza il `classnameID` modello anziché `ID` da solo. L' `Student` entità ha utilizzato il `ID` modello. Alcuni sviluppatori preferiscono usare un modello in tutto il modello di dati. In questa esercitazione la variazione illustra che è possibile usare uno dei due modelli. In un' [esercitazione successiva](inheritance.md) viene illustrato come l'utilizzo di `ID` senza NomeClasse rende più semplice l'implementazione dell'ereditarietà nel modello di dati.

La proprietà `Grade` è un oggetto`enum`. `?`Dopo la `Grade` dichiarazione del tipo indica che la `Grade` proprietà [ammette i valori null](/dotnet/csharp/language-reference/builtin-types/nullable-value-types). Un livello `null` diverso da un livello zero. `null` indica che un livello non è noto o non è ancora stato assegnato.

La `StudentID` proprietà è una chiave esterna (FK) e la proprietà di navigazione corrispondente è `Student` . Un' `Enrollment` entità è associata a un' `Student` entità, pertanto la proprietà può ospitare solo una singola `Student` entità. Questo comportamento è diverso da quello della `Student.Enrollments` proprietà di navigazione, che può ospitare più `Enrollment` entità.

La `CourseID` proprietà è un FK e la proprietà di navigazione corrispondente è `Course` . Un'entità `Enrollment` è associata a un'entità `Course`.

Entity Framework interpreta una proprietà come proprietà FK se è denominata nome della proprietà della `<` chiave primaria del nome della proprietà di navigazione `><` `>` . Ad esempio, `StudentID` per la `Student` proprietà di navigazione poiché il `Student` PK dell'entità è `ID` . Le proprietà FK possono anche essere denominate  `<` nome della proprietà della chiave primaria `>` . Ad esempio, `CourseID` poiché il `Course` PK dell'entità è `CourseID` .

### <a name="the-course-entity"></a>Entità Course (Corso)

![Diagramma entità Course (Corso)](intro/_static/course-entity.png)

Nella cartella *Models* creare la `Course` classe con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Intro)]

La proprietà `Enrollments` rappresenta una proprietà di navigazione. È possibile correlare un'entità `Course` a un numero qualsiasi di entità `Enrollment`.

L'attributo [DatabaseGenerated](xref:System.ComponentModel.DataAnnotations.DatabaseGeneratedAttribute) è illustrato in un' [esercitazione successiva](complex-data-model.md). Questo attributo consente di immettere il PK per il corso anziché il database per generarlo.

## <a name="create-the-database-context"></a>Creare il contesto di database

La classe principale che coordina la funzionalità EF per un determinato modello di dati è la <xref:Microsoft.EntityFrameworkCore.DbContext> classe del contesto di database. Questa classe viene creata mediante derivazione dalla classe `Microsoft.EntityFrameworkCore.DbContext`. La `DbContext` classe derivata specifica quali entità sono incluse nel modello di dati. È possibile personalizzare alcuni comportamenti di EF. In questo progetto la classe è denominata `SchoolContext`.

Nella cartella del progetto creare una cartella denominata `Data` .

Nella cartella *dati* creare una `SchoolContext` classe con il codice seguente:

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_Intro)]

Il codice precedente crea una `DbSet` proprietà per ogni set di entità. Nella terminologia EF:

* Un set di entità corrisponde in genere alla tabella di un database.
* Un'entità corrisponde a una riga nella tabella.

Le `DbSet<Enrollment>` `DbSet<Course>` istruzioni e possono essere omesse e funzionano allo stesso modo. EF li includerebbe in modo implicito perché:

* L' `Student` entità fa riferimento all' `Enrollment` entità.
* L' `Enrollment` entità fa riferimento all' `Course` entità.

Quando viene creato il database, Entity Framework crea tabelle i cui nomi sono gli stessi della proprietà `DbSet`. I nomi di proprietà per le raccolte sono in genere plurali. Ad esempio, `Students` anziché `Student` . Gli sviluppatori non hanno un'opinione unanime sul fatto che i nomi di tabella debbano essere pluralizzati oppure no. Per queste esercitazioni, viene eseguito l'override del comportamento predefinito specificando nomi di tabella singolari in `DbContext` . A tale scopo, aggiungere il codice evidenziato seguente dopo l'ultima proprietà DbSet.

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_TableNames&highlight=16-21)]

## <a name="register-the-schoolcontext"></a>Registrare il `SchoolContext`

ASP.NET Core include l'[inserimento delle dipendenze](../../fundamentals/dependency-injection.md). I servizi, ad esempio il contesto di database EF, vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'app. I componenti che richiedono questi servizi, ad esempio i controller MVC, vengono forniti tramite i parametri del costruttore. Il codice del costruttore del controller che ottiene un'istanza del contesto viene illustrato più avanti in questa esercitazione.

Per registrare `SchoolContext` come servizio, aprire *Startup.cs* e aggiungere le righe evidenziate al metodo `ConfigureServices`.

[!code-csharp[](intro/samples/5cu-snap/Startup.cs?name=snippet&highlight=1-2,22-23)]

Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto `DbContextOptionsBuilder`. Per lo sviluppo locale, il [sistema di configurazione ASP.NET Core](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.

Aprire il *appsettings.json* file e aggiungere una stringa di connessione come illustrato nel markup seguente:

[!code-json[](./intro/samples/5cu/appsettings1.json?highlight=2-4)]

### <a name="add-the-database-exception-filter"></a>Aggiungere il filtro eccezioni del database

Aggiungere <xref:Microsoft.Extensions.DependencyInjection.DatabaseDeveloperPageExceptionFilterServiceExtensions.AddDatabaseDeveloperPageExceptionFilter%2A> a `ConfigureServices` come illustrato nel codice seguente:

[!code-csharp[](intro/samples/5cu/Startup.cs?name=snippet&highlight=6)]

`AddDatabaseDeveloperPageExceptionFilter`Fornisce informazioni utili sull'errore nell' [ambiente di sviluppo](xref:fundamentals/environments).

### <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

La stringa di connessione specifica [SQL Server Local DB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb). Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di app e non per la produzione. Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa. Per impostazione predefinita, Local DB crea file di database con estensione *mdf* nella directory `C:/Users/<user>`.

## <a name="initialize-db-with-test-data"></a>Inizializzare il database con dati di test

EF crea un database vuoto. In questa sezione viene aggiunto un metodo che viene chiamato dopo la creazione del database per compilarlo con i dati di test.

Il `EnsureCreated` metodo viene utilizzato per creare automaticamente il database. In un' [esercitazione successiva](migrations.md)si vedrà come gestire le modifiche al modello utilizzando migrazioni Code First per modificare lo schema del database anziché eliminare e ricreare il database.

Nella cartella *Data (dati* ) creare una nuova classe denominata `DbInitializer` con il codice seguente:

[!code-csharp[DbInitializer](intro/samples/5cu-snap/DbInitializer.cs)]

Il codice precedente controlla se il database esiste:

* Se il database non viene trovato;
  * Viene creato e caricato con i dati di test. I dati di test vengono caricati in matrici anziché in raccolte `List<T>` per ottimizzare le prestazioni.
* Se il database viene trovato, non viene eseguita alcuna azione.

Aggiornare *Program.cs* con il codice seguente:

[!code-csharp[Program file](intro/samples/5cu-snap/Program.cs?highlight=1-2,14-18,21-37)]

*Program.cs* esegue le operazioni seguenti all'avvio dell'app:

* Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.
* Chiamare il metodo `DbInitializer.Initialize` .
* Eliminare il contesto quando il `Initialize` metodo viene completato come illustrato nel codice seguente:

[!code-csharp[](intro/samples/cu/Program.cs?name=snippet_Seed&highlight=5-18)]

La prima volta che l'app viene eseguita, il database viene creato e caricato con i dati di test. Ogni volta che il modello di dati cambia:

* Eliminare il database.
* Aggiornare il metodo Seed e iniziare di nuovo con un nuovo database.

 Nelle esercitazioni successive il database viene modificato in caso di modifica del modello di dati, senza eliminarlo e ricrearlo. Quando il modello di dati viene modificato, non viene perso alcun dato.

## <a name="create-controller-and-views"></a>Creare controller e visualizzazioni

Usare il motore di ponteggi in Visual Studio per aggiungere un controller e visualizzazioni MVC che useranno EF per eseguire query e salvare i dati.

La creazione automatica di metodi di azione [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) e visualizzazioni è nota come impalcatura.

* In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla `Controllers` cartella e scegliere **Aggiungi > nuovo elemento con impalcatura**.
* Nella finestra di dialogo **Aggiungi scaffolding** eseguire le operazioni seguenti:
  * Selezionare **Controller MVC con visualizzazioni, che usa Entity Framework**.
  * Fare clic su **Aggiungi**. Viene visualizzata la finestra **di dialogo Aggiungi controller MVC con visualizzazioni, usando Entity Framework** : per ![ studenti di ponteggi](intro/_static/scaffold-student2.png)
  * In **classe modello** selezionare **studente**.
  * In **classe contesto dati** selezionare **schoolContext**.
  * Accettare il valore predefinito **StudentsController** come nome.
  * Fare clic su **Aggiungi**.

Il motore di ponteggi di Visual Studio crea un `StudentsController.cs` file e un set di visualizzazioni ( `*.cshtml` file) che funzionano con il controller.

Si noti che il controller accetta `SchoolContext` come parametro del costruttore.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Context&highlight=5,7,9)]

L'inserimento delle dipendenze di ASP.NET Core si occupa di passare un'istanza di `SchoolContext` al controller. Questo è stato configurato nella `Startup` classe.

Il controller contiene un metodo di azione `Index` che consente di visualizzare tutti gli studenti nel database. Il metodo ottiene un elenco degli studenti dal set di entità Students (Studenti) leggendo la proprietà `Students` dell'istanza del contesto di database:

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex&highlight=3)]

Gli elementi di programmazione asincrona in questo codice sono illustrati più avanti nell'esercitazione.

Nella visualizzazione *Views/Students/Index.cshtml* è contenuto l'elenco sotto forma di tabella:

[!code-cshtml[](intro/samples/cu/Views/Students/Index1.cshtml)]

Premere CTRL+F5 per eseguire il progetto oppure selezionare **Debug > Avvia senza eseguire debug** dal menu.

Fare clic sulla scheda Students (Studenti) per visualizzare i dati di test inseriti nel metodo `DbInitializer.Initialize`. A seconda delle dimensione della finestra del browser, il collegamento della scheda `Students` viene visualizzato in alto alla pagina oppure è necessario selezionare l'icona di spostamento in alto a destra per visualizzarlo.

![Home page di Contoso University ristretta](intro/_static/home-page-narrow.png)

![Pagina Student Index (Indice degli studenti)](intro/_static/students-index.png)

## <a name="view-the-database"></a>Visualizzare il database

Quando l'app viene avviata, il `DbInitializer.Initialize` metodo chiama `EnsureCreated` . EF ha visto che non era disponibile alcun database:

* Quindi, è stato creato un database.
* Il `Initialize` codice del metodo ha popolato il database con i dati.

Usare **Esplora oggetti di SQL Server** (SSOX) per visualizzare il database in Visual Studio:

* Selezionare **Esplora oggetti di SQL Server** dal menu **Visualizza** in Visual Studio.
* In SSOX selezionare (local DB **) \MSSQLLocalDB > database**.
* Selezionare `ContosoUniversity1` la voce relativa al nome del database nella stringa di connessione nel *appsettings.json* file.
* Espandere il nodo **tabelle** per visualizzare le tabelle nel database.

![Tabelle in SSOX](intro/_static/ssox-tables.png)

Fare clic con il pulsante destro del mouse sulla tabella **Student** e scegliere **Visualizza dati** per visualizzare i dati nella tabella.

![Tabella Student (Studente) in Esplora oggetti di SQL Server](intro/_static/ssox-student-table.png)

I `*.mdf` `*.ldf` file di database e si trovano nella cartella *C:\Users \\ \<username>*

Poiché `EnsureCreated` viene chiamato nel metodo dell'inizializzatore eseguito all'avvio dell'app, è possibile:

* Apportare una modifica alla `Student` classe.
* Eliminare il database.
* Arrestare, quindi avviare l'app. Il database viene ricreato automaticamente in base alla modifica.

Ad esempio, se una `EmailAddress` proprietà viene aggiunta alla `Student` classe, una nuova `EmailAddress` colonna nella tabella ricreata. La vista con classe non visualizzerà la nuova `EmailAddress` Proprietà.

## <a name="conventions"></a>Convenzioni

La quantità di codice scritta per la creazione di un database completo da parte di EF è minima a causa dell'utilizzo delle convenzioni utilizzate da EF:

* I nomi delle proprietà `DbSet` vengono usate come nomi di tabella. Per le entità a cui non fa riferimento una proprietà `DbSet`, i nomi della classe di entità vengono usati come nome di tabella.
* I nomi della proprietà di entità vengono usati come nomi di colonna.
* Le proprietà delle entità denominate `ID` o `classnameID` vengono riconosciute come proprietà PK.
* Una proprietà viene interpretata come una proprietà FK se è denominata nome della proprietà di `<` navigazione PK nome della `><` proprietà `>` . Ad esempio, `StudentID` per la `Student` proprietà di navigazione poiché il `Student` PK dell'entità è `ID` . Le proprietà FK possono anche essere denominate `<` nome della proprietà della chiave primaria `>` . Ad esempio, `EnrollmentID` poiché il `Enrollment` PK dell'entità è `EnrollmentID` .

È possibile eseguire l'override del comportamento convenzionale. È ad esempio possibile specificare in modo esplicito i nomi di tabella, come illustrato in precedenza in questa esercitazione. I nomi di colonna e qualsiasi proprietà possono essere impostati come PK o FK.

## <a name="asynchronous-code"></a>Codice asincrono

La programmazione asincrona è la modalità predefinita per ASP.NET Core ed Entity Framework Core.

Per un server Web è disponibile un numero limitato di thread e in situazioni di carico elevato tutti i thread disponibili potrebbero essere in uso. In queste circostanze il server non può elaborare nuove richieste finché i thread non saranno liberi. Con il codice sincrono, può succedere che molti thread siano vincolati nonostante in quel momento non stiano eseguendo alcuna operazione. Rimangono tuttavia in attesa che l'operazione I/O sia completata. Con il codice asincrono, se un processo è in attesa del completamento dell'operazione I/O, il thread viene liberato e il server lo può usare per l'elaborazione di altre richieste. Di conseguenza, il codice asincrono consente un uso più efficiente delle risorse e il server è abilitato per gestire una maggiore quantità di traffico senza ritardi.

Il codice asincrono comporta un minimo sovraccarico in fase di esecuzione. In caso di traffico ridotto, il calo delle prestazioni è irrilevante, mentre nelle situazioni di traffico elevato, è essenziale ottimizzare le prestazioni potenziali.

Nel codice seguente,, `async` , `Task<T>` `await` e `ToListAsync` rendono il codice eseguito in modo asincrono.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex)]

* La parola chiave`async` indica al compilatore di generare callback per parti del corpo del metodo e di creare automaticamente l'oggetto `Task<IActionResult>` restituito.
* Il tipo restituito `Task<IActionResult>` rappresenta il lavoro in corso con un risultato di tipo `IActionResult`.
* La parola chiave `await` indica al compilatore di suddividere il metodo in due parti. La prima parte termina con l'operazione avviata in modo asincrono. La seconda parte viene inserita in un metodo di callback che viene chiamato al termine dell'operazione.
* `ToListAsync` è la versione asincrona del metodo di estensione `ToList`.

Di seguito sono riportate alcune considerazioni da tenere presente quando si scrive codice asincrono che utilizza EF:

* Solo le istruzioni che generano query o comandi da inviare al database vengono eseguite in modo asincrono, ad esempio `ToListAsync`, `SingleOrDefaultAsync`, e `SaveChangesAsync`. Non sono incluse le istruzioni che modificano solo un oggetto `IQueryable`, ad esempio `var students = context.Students.Where(s => s.LastName == "Davolio")`.
* Un contesto di Entity Framework non è thread-safe. Non tentare di eseguire più operazioni parallelamente. Quando si chiama un metodo asincrono Entity Framework, usare sempre la parola chiave `await`.
* Per sfruttare i vantaggi in termini di prestazioni del codice asincrono, assicurarsi che tutti i pacchetti di libreria utilizzati usino anche async se chiamano metodi EF che provocano l'invio di query al database.

Per altre informazioni sulla programmazione asincrona in .NET, vedere [Panoramica della programmazione asincrona](/dotnet/articles/standard/async).

## <a name="limit-entities-fetched"></a>Limita entità recuperate

Per informazioni su come limitare il numero o le entità restituite da una query, vedere [considerazioni sulle prestazioni](xref:data/ef-rp/intro) .

Passare all'esercitazione successiva per informazioni su come eseguire operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) di base.

> [!div class="nextstepaction"]
> [Implementare funzionalità CRUD di base](crud.md)

::: moniker-end

::: moniker range="<= aspnetcore-3.1"

[!INCLUDE [RP better than MVC](~/includes/RP-EF/rp-over-mvc.md)]

L'applicazione Web di esempio di Contoso University illustra come creare applicazioni Web ASP.NET Core 2.2 MVC con Entity Framework (EF) Core 2.2 e Visual Studio 2017 o 2019.

Questa esercitazione non è stata aggiornata per ASP.NET Core 3,1. È stato aggiornato per [ASP.NET Core 5,0](xref:data/ef-mvc/intro?view=aspnetcore-5.0).

L'applicazione di esempio è un sito Web per una fittizia Contoso University. Include funzionalità, come ad esempio l'ammissione di studenti, la creazione di corsi e le assegnazioni di insegnati. Questa è la prima di una serie di esercitazioni che illustrano come compilare da zero l'app di esempio di Contoso University.

## <a name="prerequisites"></a>Prerequisiti

* [.NET Core SDK 2.2](https://dotnet.microsoft.com/download)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con i carichi di lavoro seguenti:
  * **ASP.NET e carico di lavoro di sviluppo Web**
  * Carico di lavoro di **sviluppo multipiattaforma .NET Core**

## <a name="troubleshooting"></a>Risoluzione dei problemi

Se si verifica un problema che non si sa come risolvere, è generalmente possibile trovare la soluzione confrontando il codice con il [progetto completato](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples). Per un elenco degli errori comuni e delle relative soluzioni, vedere [la sezione relativa alla risoluzione dei problemi nell'ultima esercitazione della serie](advanced.md#common-errors). Se non si trova la soluzione, è possibile inviare una domanda a StackOverflow.com per [ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) o [Entity Framework Core](https://stackoverflow.com/questions/tagged/entity-framework-core).

> [!TIP]
> In questa serie di 10 esercitazioni ogni esercitazione si basa su quanto viene eseguito nelle esercitazioni precedenti. È consigliabile salvare una copia del progetto dopo aver completato ogni esercitazione. Se si verificano problemi, è possibile ricominciare dall'esercitazione precedente anziché tornare all'inizio della serie.

## <a name="contoso-university-web-app"></a>App Web di Contoso University

L'applicazione che sarà compilata in queste esercitazioni è un semplice sito Web universitario.

Gli utenti possono visualizzare e aggiornare le informazioni che riguardano studenti, corsi e insegnanti. Di seguito sono disponibili alcune schermate che saranno create.

![Pagina Student Index (Indice degli studenti)](intro/_static/students-index.png)

![Pagina Students Edit (Modifica studenti)](intro/_static/student-edit.png)

## <a name="create-web-app"></a>Crea app Web

* Aprire Visual Studio.

* Scegliere **Nuovo > Progetto** dal menu **File**.

* Nel riquadro sinistro selezionare **Installato > Visual C# > Web**.

* Selezionare il modello di progetto **Applicazione Web ASP.NET Core**.

* Immettere **ContosoUniversity** come nome e fare clic su **OK**.

  ![Finestra di dialogo Nuovo progetto](intro/_static/new-project2.png)

* Attendere che venga visualizzata la finestra di dialogo **Nuova applicazione Web ASP.NET Core**.

* Selezionare **.NET Core**, **ASP.NET Core 2.2** e il modello **Applicazione Web (MVC)**.

* Assicurarsi che **l'autenticazione** sia impostata su **Nessuna autenticazione**.

* Selezionare **OK**.

  ![Finestra di dialogo Nuovo progetto ASP.NET Core](intro/_static/new-aspnet2.png)

## <a name="set-up-the-site-style"></a>Impostare lo stile del sito

Con alcune modifiche è possibile impostare il menu del sito, il layout e la home page.

Aprire *Views/Shared/_Layout.cshtml* e apportare le modifiche seguenti:

* Modificare tutte le occorrenze di "ContosoUniversity" in "Contoso University". Le occorrenze sono tre.

* Aggiungere le voci di menu per **About** (Informazioni su), **Students** (Studenti), **Courses** (Corsi), **Instructors** (Insegnanti) e **Departments** (Dipartimenti) ed eliminare la voce di menu **Privacy**.

Le modifiche sono evidenziate.

[!code-cshtml[](intro/samples/cu/Views/Shared/_Layout.cshtml?highlight=6,34-48,63)]

In *Views/Home/Index.cshtml* sostituire il contenuto del file con il codice seguente. In questo modo il testo su ASP.NET e MVC sarà sostituito dal testo relativo a questa applicazione:

[!code-cshtml[](intro/samples/cu/Views/Home/Index.cshtml)]

Premere CTRL+F5 per eseguire il progetto oppure selezionare **Debug > Avvia senza eseguire debug** dal menu. La home page sarà visualizzata con le schede create in queste esercitazioni.

![Home page di Contoso University](intro/_static/home-page.png)

## <a name="about-ef-core-nuget-packages"></a>Informazioni sui pacchetti NuGet di EF Core

Per aggiungere il supporto di Entity Framework Core a un progetto, installare il provider di database di destinazione. Questa esercitazione usa SQL Server e il pacchetto del provider è [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/). Poiché il pacchetto è incluso nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), non è necessario inserire alcun riferimento al pacchetto.

Il pacchetto EF SQL Server e le relative dipendenze (`Microsoft.EntityFrameworkCore` e `Microsoft.EntityFrameworkCore.Relational`) offrono supporto di runtime per Entity Framework. Nell'esercitazione [Migrazioni](migrations.md) viene aggiunto un pacchetto di strumenti.

Per informazioni su altri provider di database disponibili per Entity Framework Core, vedere [Provider di database](/ef/core/providers/).

## <a name="create-the-data-model"></a>Creare il modello di dati

A questo punto è possibile creare le classi delle entità per l'applicazione di Contoso University. Si inizia con le tre entità seguenti.

![Diagramma modello di dati Course/Enrollment/Student (Corso/Iscrizione/Studente)](intro/_static/data-model-diagram.png)

Esiste una relazione uno-a-molti tra le entità `Student` e `Enrollment` ed esiste una relazione uno-a-molti tra le entità `Course` e `Enrollment`. In altre parole, uno studente può iscriversi a un numero qualsiasi di corsi e un corso può avere un numero qualsiasi di studenti iscritti.

Nelle sezioni seguenti viene creata una classe per ognuna di queste entità.

### <a name="the-student-entity"></a>Entità Student (Studente)

![Diagramma entità Student (Studente)](intro/_static/student-entity.png)

Nella cartella *Models* (Modelli) creare un file di classe denominato *Student.cs* e sostituire il codice del modello con il codice seguente.

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Intro)]

La proprietà `ID` diventa la colonna di chiave primaria della tabella di database che corrisponde a questa classe. Per impostazione predefinita, Entity Framework interpreta una proprietà denominata `ID` o `classnameID` come la chiave primaria.

La proprietà `Enrollments` rappresenta una [proprietà di navigazione](/ef/core/modeling/relationships). Le proprietà di navigazione contengono altre entità correlate a questa entità. In questo caso la proprietà `Enrollments` di `Student entity` contiene tutte le entità `Enrollment` correlate all'entità `Student`. In altre parole, se una `Student` riga nel database dispone di due `Enrollment` righe correlate (righe che contengono il valore della chiave primaria di tale studente nella colonna chiave esterna StudentID), la `Student` proprietà di navigazione di tale entità conterrà `Enrollments` tali due `Enrollment` entità.

Se una proprietà di navigazione può contenere più entità (come nel caso di relazioni molti-a-molti e uno-a-molti), il tipo della proprietà deve essere un elenco in cui le voci possono essere aggiunte, eliminate e aggiornate, come ad esempio `ICollection<T>`. È possibile specificare `ICollection<T>` o un tipo come `List<T>` o `HashSet<T>`. Se si specifica `ICollection<T>`, per impostazione predefinita EF crea una raccolta `HashSet<T>`.

### <a name="the-enrollment-entity"></a>Entità Enrollment (Iscrizione)

![Diagramma entità Enrollment (Iscrizione)](intro/_static/enrollment-entity.png)

Nella cartella *Models* creare *Enrollment.cs* e sostituire il codice esistente con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Intro)]

La proprietà `EnrollmentID` sarà la chiave primaria. Questa entità usa il criterio `classnameID` anziché `ID` come descritto per l'entità `Student`. In genere si sceglie un criterio e lo si usa poi in tutto il modello di dati. In questo caso la variazione illustra che è possibile sia uno sia l'altro criterio. In un'[esercitazione successiva](inheritance.md) viene illustrato l'uso di ID senza classname. In questo modo si semplifica l'implementazione dell'ereditarietà nel modello di dati.

La proprietà `Grade` è un oggetto`enum`. Il punto interrogativo dopo la dichiarazione del tipo `Grade` indica che la proprietà `Grade` ammette i valori Null. Un voto il cui valore è Null è diverso da un voto con valore zero. Null significa che un voto è sconosciuto o non è stato ancora assegnato.

La proprietà `StudentID` è una chiave esterna e la proprietà di navigazione corrispondente è `Student`. Un'entità `Enrollment` è associata a un'entità `Student`, pertanto la proprietà può contenere un'unica entità `Student`, a differenza della proprietà di navigazione `Student.Enrollments` vista in precedenza, che può contenere più entità `Enrollment`.

La proprietà `CourseID` è una chiave esterna e la proprietà di navigazione corrispondente è `Course`. Un'entità `Enrollment` è associata a un'entità `Course`.

Entity Framework interpretata una proprietà come proprietà di chiave esterna se è denominata `<navigation property name><primary key property name>`, ad esempio `StudentID` per la proprietà di navigazione `Student` poiché la chiave primaria dell'entità `Student` è `ID`. Le proprietà di una chiave esterna possono anche essere denominate semplicemente `<primary key property name>`, ad esempio `CourseID` poiché la chiave primaria dell'entità `Course` è `CourseID`.

### <a name="the-course-entity"></a>Entità Course (Corso)

![Diagramma entità Course (Corso)](intro/_static/course-entity.png)

Nella cartella *Models* creare *Course.cs* e sostituire il codice esistente con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Intro)]

La proprietà `Enrollments` rappresenta una proprietà di navigazione. È possibile correlare un'entità `Course` a un numero qualsiasi di entità `Enrollment`.

Altre informazioni sull'attributo `DatabaseGenerated` sono disponibili nell'[esercitazione successiva](complex-data-model.md) di questa serie. In pratica, questo attributo consente di immettere la chiave primaria per il corso invece di essere generata dal database.

## <a name="create-the-database-context"></a>Creare il contesto di database

La classe del contesto di database è la classe principale che coordina le funzionalità di Entity Framework per un determinato modello di dati. Questa classe viene creata mediante derivazione dalla classe `Microsoft.EntityFrameworkCore.DbContext`. Nel codice vengono specificate le entità incluse nel modello di dati. È anche possibile personalizzare un determinato comportamento di Entity Framework. In questo progetto la classe è denominata `SchoolContext`.

Creare una cartella denominata *Data* (Dati) nella cartella del progetto.

Nella cartella *Data* (Dati) creare un file di classe denominato *SchoolContext.cs* e sostituire il codice del modello con il codice seguente:

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_Intro)]

Questo codice crea una proprietà `DbSet` per ogni set di entità. Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella.

Se sono state omesse le istruzioni `DbSet<Enrollment>` e `DbSet<Course>`, potrebbe comunque funzionare. Entity Framework le include in modo implicito perché l'entità `Student` fa riferimento all'entità `Enrollment` e l'entità `Enrollment` fa riferimento all'entità `Course`.

Quando viene creato il database, Entity Framework crea tabelle i cui nomi sono gli stessi della proprietà `DbSet`. I nomi di proprietà per le raccolte sono generalmente usati al plurale (Students anziché Student). Gli sviluppatori non hanno tuttavia un'opinione unanime sul fatto che i nomi di tabella debbano essere usati al plurale. Per queste esercitazioni, viene eseguito l'override del comportamento predefinito specificando nomi di tabella al singolare in DbContext. A tale scopo, aggiungere il codice evidenziato seguente dopo l'ultima proprietà DbSet.

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_TableNames&highlight=16-21)]

Compilare il progetto per controllare se sono presenti errori del compilatore.

## <a name="register-the-schoolcontext"></a>Registrare SchoolContext

ASP.NET Core implementa l'[inserimento delle dipendenze](../../fundamentals/dependency-injection.md) per impostazione predefinita. I servizi, ad esempio il contesto di database di Entity Framework, vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione. Questi servizi vengono quindi offerti ai componenti per cui sono necessari (ad esempio i controller MVC) tramite i parametri del costruttore. Il codice del costruttore del controller che ottiene un'istanza di contesto viene illustrato più avanti in questa esercitazione.

Per registrare `SchoolContext` come servizio, aprire *Startup.cs* e aggiungere le righe evidenziate al metodo `ConfigureServices`.

[!code-csharp[](intro/samples/cu/Startup.cs?name=snippet_SchoolContext&highlight=9-10)]

Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto `DbContextOptionsBuilder`. Per lo sviluppo locale, il [sistema di configurazione ASP.NET Core](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.

Aggiungere le istruzioni `using` per gli spazi dei nomi `ContosoUniversity.Data` e `Microsoft.EntityFrameworkCore`, quindi compilare il progetto.

[!code-csharp[](intro/samples/cu/Startup.cs?name=snippet_Usings)]

Aprire il *appsettings.json* file e aggiungere una stringa di connessione come illustrato nell'esempio seguente.

[!code-json[](./intro/samples/cu/appsettings1.json?highlight=2-4)]

### <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

La stringa di connessione specifica un database Local DB di SQL Server. Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di applicazioni e non per la produzione. Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa. Per impostazione predefinita, Local DB crea file di database con estensione *mdf* nella directory `C:/Users/<user>`.

## <a name="initialize-db-with-test-data"></a>Inizializzare il database con dati di test

Entity Framework creerà un database vuoto. In questa sezione viene scritto un metodo che viene chiamato dopo aver creato il database al fine di popolare il database con i dati di test.

Per creare automaticamente il database, viene usato il metodo `EnsureCreated`. In un'[esercitazione successiva](migrations.md) viene spiegato come gestire le modifiche al modello usando Migrazioni Code First, che consente di modificare lo schema del database anziché dover eliminare e ricreare il database.

Nella cartella *Data* (Dati) creare un file di classe denominato *DbInitializer.cs* e sostituire il codice del modello con il codice seguente. Verrà creato un database se necessario e i dati di test saranno caricati nel nuovo database.

[!code-csharp[](intro/samples/cu/Data/DbInitializer.cs?name=snippet_Intro)]

Il codice verifica l'esistenza di studenti nel database. In caso negativo presuppone che il database sia nuovo e che sia necessario eseguire l'inizializzazione con i dati di test. I dati di test vengono caricati in matrici anziché in raccolte `List<T>` per ottimizzare le prestazioni.

In *Program.cs* modificare il metodo `Main` per eseguire le operazioni seguenti all'avvio dell'applicazione:

* Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.
* Chiamare il metodo di inizializzazione, passandolo al contesto.
* Eliminare il contesto dopo che il metodo di inizializzazione è stato completato.

[!code-csharp[](intro/samples/5cu-snap/Program.cs?highlight=1-2,14-18,21-38)]

La prima volta che si esegue l'applicazione, il database verrà creato e sottoposta a seeding con dati di test. Ogni volta che si modifica il modello di dati:

 * Eliminare il database.
 * Aggiornare il metodo Seed e iniziare di nuovo con un nuovo database allo stesso modo.
 
Nelle esercitazioni successive viene illustrato come modificare il database alla modifica del modello di dati, senza eliminare e ricreare il database.

## <a name="create-controller-and-views"></a>Creare controller e visualizzazioni

In questa sezione, il motore di ponteggi in Visual Studio viene usato per aggiungere un controller e visualizzazioni MVC che utilizzeranno EF per eseguire query e salvare i dati.

La creazione automatica di metodi di azione CRUD e di visualizzazioni è nota come scaffolding. Lo scaffolding differisce dalla generazione di codice in quanto il codice di scaffolding è un punto di partenza modificabile per essere adattato ai propri requisiti, mentre generalmente il codice generato non viene modificato. Quando è necessario personalizzare il codice generato, usare classi parziali oppure rigenerare il codice in caso di modifiche.

* Fare clic con il pulsante destro del mouse sulla cartella **Controller** in **Esplora soluzioni** e scegliere **Aggiungi -> Nuovo elemento di scaffolding**.
* Nella finestra di dialogo **Aggiungi scaffolding** eseguire le operazioni seguenti:
  * Selezionare **Controller MVC con visualizzazioni, che usa Entity Framework**.
  * Fare clic su **Aggiungi**. Viene visualizzata la finestra **di dialogo Aggiungi controller MVC con visualizzazioni, usando Entity Framework** : per ![ studenti di ponteggi](intro/_static/scaffold-student2.png)
  * In **Classe modello** selezionare **Student** (Studente).
  * In **Classe contesto di dati** selezionare **SchoolContext**.
  * Accettare il valore predefinito **StudentsController** come nome.
  * Fare clic su **Aggiungi**.

Il motore di ponteggi di Visual Studio crea un file *StudentsController.cs* e un set di visualizzazioni (file con *estensione cshtml* ) che funzionano con il controller.

Si noti che il controller accetta `SchoolContext` come parametro del costruttore.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Context&highlight=5,7,9)]

L'inserimento delle dipendenze di ASP.NET Core si occupa di passare un'istanza di `SchoolContext` al controller. Questa configurazione è stata configurata nel file *Startup.cs* .

Il controller contiene un metodo di azione `Index` che consente di visualizzare tutti gli studenti nel database. Il metodo ottiene un elenco degli studenti dal set di entità Students (Studenti) leggendo la proprietà `Students` dell'istanza del contesto di database:

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex&highlight=3)]

Ulteriori informazioni sugli elementi di programmazione asincrona in questo codice sono disponibili più avanti nell'esercitazione.

Nella visualizzazione *Views/Students/Index.cshtml* è contenuto l'elenco sotto forma di tabella:

[!code-cshtml[](intro/samples/cu/Views/Students/Index1.cshtml)]

Premere CTRL+F5 per eseguire il progetto oppure selezionare **Debug > Avvia senza eseguire debug** dal menu.

Fare clic sulla scheda Students (Studenti) per visualizzare i dati di test inseriti nel metodo `DbInitializer.Initialize`. A seconda delle dimensione della finestra del browser, il collegamento della scheda `Students` viene visualizzato in alto alla pagina oppure è necessario selezionare l'icona di spostamento in alto a destra per visualizzarlo.

![Home page di Contoso University ristretta](intro/_static/home-page-narrow.png)

![Pagina Student Index (Indice degli studenti)](intro/_static/students-index.png)

## <a name="view-the-database"></a>Visualizzare il database

Dopo aver avviato l'applicazione, il metodo `DbInitializer.Initialize` chiama `EnsureCreated`. Entity Framework ha verificato che non esiste un database e ne ha pertanto creato uno. La parte restante di codice del metodo `Initialize` ha popolato il database con i dati. È possibile usare **Esplora oggetti di SQL Server** per visualizzare il database in Visual Studio.

Chiudere il browser.

Se la finestra di Esplora oggetti di SQL Server non è già aperta, selezionarla dal menu **Visualizza** in Visual Studio.

In SSOX fare clic su (local DB **) \MSSQLLocalDB > database**, quindi fare clic sulla voce relativa al nome del database nella stringa di connessione nel *appsettings.json* file.

Espandere il nodo **tabelle** per visualizzare le tabelle nel database.

![Tabelle in SSOX](intro/_static/ssox-tables.png)

Fare clic con il pulsante destro del mouse sulla tabella **Student** (Studente) e fare clic su **Visualizza dati** per visualizzare le colonne create e le righe inserite nella tabella.

![Tabella Student (Studente) in Esplora oggetti di SQL Server](intro/_static/ssox-student-table.png)

I file di database con estensione *mdf* e *ldf* sono contenuti nella cartella *C:\Utenti\\\<username>*.

Poiché si sta chiamando `EnsureCreated` nel metodo di inizializzatore che viene eseguito all'avvio dell'app, è ora possibile modificare la classe `Student`, eliminare il database ed eseguire nuovamente l'applicazione. Il database sarà automaticamente ricreato e rispecchierà la modifica. Ad esempio, se si aggiunge una proprietà `EmailAddress` alla classe `Student`, una nuova colonna `EmailAddress` sarà visualizzata nella tabella ricreata.

## <a name="conventions"></a>Convenzioni

Grazie all'uso di convenzioni o di ipotesi di Entity Framework, la quantità di codice scritto che è necessario a Entity Framework per creare un database completo è minino.

* I nomi delle proprietà `DbSet` vengono usate come nomi di tabella. Per le entità a cui non fa riferimento una proprietà `DbSet`, i nomi della classe di entità vengono usati come nome di tabella.
* I nomi della proprietà di entità vengono usati come nomi di colonna.
* Le proprietà dell'entità vengono denominate ID o classnameID e vengono riconosciute come proprietà della chiave primaria.
* Una proprietà viene interpretata come una proprietà di chiave esterna se è denominata *\<navigation property name>\<primary key property name>* , ad esempio `StudentID` per la `Student` proprietà di navigazione poiché la `Student` chiave primaria dell'entità è `ID` . Le proprietà di chiave esterna possono anche essere denominate semplicemente *\<primary key property name>* , ad esempio `EnrollmentID` poiché la `Enrollment` chiave primaria dell'entità è `EnrollmentID` .

È possibile eseguire l'override del comportamento convenzionale. Ad esempio, è possibile specificare in modo esplicito i nomi di tabella, come illustrato in precedenza in questa esercitazione. È anche possibile impostare i nomi delle colonne e impostare qualsiasi proprietà come chiave primaria o chiave esterna, come sarà spiegato in un'[esercitazione successiva](complex-data-model.md) di questa serie.

## <a name="asynchronous-code"></a>Codice asincrono

La programmazione asincrona è la modalità predefinita per ASP.NET Core ed Entity Framework Core.

Per un server Web è disponibile un numero limitato di thread e in situazioni di carico elevato tutti i thread disponibili potrebbero essere in uso. In queste circostanze il server non può elaborare nuove richieste finché i thread non saranno liberi. Con il codice sincrono, può succedere che molti thread siano vincolati nonostante in quel momento non stiano eseguendo alcuna operazione. Rimangono tuttavia in attesa che l'operazione I/O sia completata. Con il codice asincrono, se un processo è in attesa del completamento dell'operazione I/O, il thread viene liberato e il server lo può usare per l'elaborazione di altre richieste. Di conseguenza, il codice asincrono consente un uso più efficiente delle risorse e il server è abilitato per gestire una maggiore quantità di traffico senza ritardi.

Il codice asincrono comporta un minimo sovraccarico in fase di esecuzione. In caso di traffico ridotto, il calo delle prestazioni è irrilevante, mentre nelle situazioni di traffico elevato, è essenziale ottimizzare le prestazioni potenziali.

Nel codice seguente la parola chiave `async`, il valore restituito `Task<T>`, la parola chiave `await` e il metodo `ToListAsync` consentono di eseguire il codice in modo asincrono.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ScaffoldedIndex)]

* La parola chiave`async` indica al compilatore di generare callback per parti del corpo del metodo e di creare automaticamente l'oggetto `Task<IActionResult>` restituito.
* Il tipo restituito `Task<IActionResult>` rappresenta il lavoro in corso con un risultato di tipo `IActionResult`.
* La parola chiave `await` indica al compilatore di suddividere il metodo in due parti. La prima parte termina con l'operazione avviata in modo asincrono. La seconda parte viene inserita in un metodo di callback che viene chiamato al termine dell'operazione.
* `ToListAsync` è la versione asincrona del metodo di estensione `ToList`.

Alcuni aspetti da considerare quando si scrive codice asincrono usato da Entity Framework:

* Solo le istruzioni che generano query o comandi da inviare al database vengono eseguite in modo asincrono, ad esempio `ToListAsync`, `SingleOrDefaultAsync`, e `SaveChangesAsync`. Non sono incluse le istruzioni che modificano solo un oggetto `IQueryable`, ad esempio `var students = context.Students.Where(s => s.LastName == "Davolio")`.
* Un contesto di Entity Framework non è thread-safe. Non tentare di eseguire più operazioni parallelamente. Quando si chiama un metodo asincrono Entity Framework, usare sempre la parola chiave `await`.
* Se si vogliono sfruttare i vantaggi del codice asincrono in termini di prestazioni, verificare che i pacchetti della libreria impiegati, ad esempio per il paging, usino la modalità asincrona per chiamare i metodi di Entity Framework che generano query da inviare al database.

Per altre informazioni sulla programmazione asincrona in .NET, vedere [Panoramica della programmazione asincrona](/dotnet/articles/standard/async).

## <a name="next-steps"></a>Passaggi successivi

Passare all'esercitazione successiva per informazioni su come eseguire operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) di base.

> [!div class="nextstepaction"]
> [Implementare funzionalità CRUD di base](crud.md)

::: moniker-end
