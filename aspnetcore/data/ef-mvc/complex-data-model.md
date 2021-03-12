---
title: 'Esercitazione: creare un modello di dati complesso-ASP.NET MVC con EF Core'
description: In questa esercitazione si aggiungono altre entità e relazioni e si personalizza il modello di dati specificando regole di formattazione, convalida e mapping.
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 03/27/2019
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
uid: data/ef-mvc/complex-data-model
ms.openlocfilehash: a881ff28d41a272ade559c60efbd884f2a3c4e3e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587788"
---
# <a name="tutorial-create-a-complex-data-model---aspnet-mvc-with-ef-core"></a>Esercitazione: creare un modello di dati complesso-ASP.NET MVC con EF Core

Nelle esercitazioni precedenti è stato usato un modello di dati semplice costituito da tre entità. In questa esercitazione si aggiungeranno altre entità e relazioni e si personalizzerà il modello di dati specificando regole di formattazione, convalida e mapping del database.

Al termine dell'operazione le classi di entità verranno incluse nel modello di dati completato, illustrato nella figura seguente:

![Diagramma dell'entità](complex-data-model/_static/diagram.png)

In questa esercitazione:

> [!div class="checklist"]
> * Personalizzare il modello di dati
> * Apportare modifiche all'entità Student
> * Creare l'entità Instructor
> * Creare l'entità OfficeAssignment
> * Modificare l'entità Course
> * Creare l'entità Department
> * Modificare l'entità Enrollment
> * Aggiornare il contesto di database
> * Eseguire il seeding del database con dati di test
> * Aggiungere una migrazione
> * Modificare la stringa di connessione
> * Aggiornare il database

## <a name="prerequisites"></a>Prerequisiti

* [Uso di migrazioni EF Core](migrations.md)

## <a name="customize-the-data-model"></a>Personalizzare il modello di dati

In questa sezione si apprenderà come personalizzare il modello di dati usando attributi che specificano regole di formattazione, convalida e mapping del database. Quindi nelle sezioni seguenti si creerà il modello di dati School (Istituto scolastico) completo, aggiungendo attributi alle classi già create e creando nuove classi per i tipi di entità rimanenti nel modello.

### <a name="the-datatype-attribute"></a>Attributo DataType

Per le date di iscrizione degli studenti, tutte le pagine Web attualmente visualizzano l'ora oltre alla data, anche se l'unico elemento rilevante di questo campo è la data. Mediante gli attributi di annotazione dei dati è possibile modificare il codice per correggere il formato di visualizzazione in tutte le visualizzazioni che visualizzano i dati. Per un esempio di come eseguire questa operazione si aggiunge un attributo alla proprietà `EnrollmentDate` nella classe `Student`.

In *Models/Student.cs* aggiungere un'istruzione `using` per lo spazio dei nomi `System.ComponentModel.DataAnnotations` e aggiungere gli attributi `DataType` e `DisplayFormat` alla proprietà `EnrollmentDate` come indicato nell'esempio seguente:

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_DataType&highlight=3,12-13)]

L'attributo `DataType` viene usato per specificare un tipo di dati che è più specifico del tipo intrinseco del database. In questo caso si vuole tenere traccia solo della data e non di data e ora. L'enumerazione `DataType` include molti tipi di dati, ad esempio Date, Time, PhoneNumber, Currency, EmailAddress e così via. L'attributo `DataType` può anche consentire all'applicazione di fornire automaticamente le funzionalità specifiche del tipo. Ad esempio, è possibile creare un collegamento `mailto:` per `DataType.EmailAddress` e fornire un selettore data per `DataType.Date` nei browser che supportano HTML5. L'attributo `DataType` produce attributi HTML5 `data-` supportati dai browser HTML5. Gli attributi `DataType` non garantiscono alcuna convalida.

`DataType.Date` non specifica il formato della data visualizzata. Per impostazione predefinita il campo dati viene visualizzato in base ai formati predefiniti derivanti dal valore CultureInfo del server.

L'attributo `DisplayFormat` viene usato per specificare in modo esplicito il formato della data:

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

L'impostazione `ApplyFormatInEditMode` specifica che la formattazione deve essere applicata anche quando il valore viene visualizzato in una casella di testo per la modifica. Per determinati campi questa scelta non è consigliabile. Ad esempio per i valori di valuta può non risultare opportuno che il simbolo di valuta sia incluso nella casella di testo per la modifica.

È possibile usare l'attributo `DisplayFormat` da solo, ma in genere è consigliabile usare anche l'attributo `DataType`. L'attributo `DataType` esprime la semantica dei dati anziché la modalità di rendering dei dati in una schermata e offre i seguenti vantaggi che non si ottengono con `DisplayFormat`:

* Il browser può abilitare le funzionalità HTML5, ad esempio per visualizzare un controllo di calendario, il simbolo della valuta appropriato per le impostazioni locali, i collegamenti alla posta elettronica, alcune istanze di convalida lato client e così via.

* Per impostazione predefinita, il browser eseguirà il rendering dei dati usando il formato corretto in base alle impostazioni locali del sistema.

Per ulteriori informazioni, vedere la [ \<input> documentazione dell'helper Tag](../../mvc/views/working-with-forms.md#the-input-tag-helper).

Eseguire l'app, passare alla pagina Students Index (Indice studenti) e verificare che l'ora non viene più visualizzata nelle date di iscrizione. Lo stesso vale per tutte le viste che usano il modello Student (Studente).

![Pagina Students Index (Indice studenti) con date e senza ore](complex-data-model/_static/dates-no-times.png)

### <a name="the-stringlength-attribute"></a>Attributo StringLength

È anche possibile specificare regole di convalida dei dati e messaggi di errore di convalida mediante gli attributi. L'attributo `StringLength` imposta la lunghezza massima nel database e offre la convalida lato client e lato server per ASP.NET Core MVC. È anche possibile specificare la lunghezza minima della stringa in questo attributo, ma il valore minimo non ha alcun effetto sullo schema del database.

Ad esempio si supponga di voler limitare a 50 il numero massimo di caratteri che gli utenti possono immettere per un nome. Per aggiungere questa limitazione aggiungere attributi `StringLength` alle proprietà `LastName` e `FirstMidName`, come illustrato nell'esempio seguente:

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_StringLength&highlight=10,12)]

L'attributo `StringLength` non impedisce a un utente di immettere spazi vuoti per un nome. È possibile usare l'attributo `RegularExpression` per applicare restrizioni all'input. Ad esempio il codice seguente richiede che il primo carattere sia maiuscolo e i caratteri rimanenti siano caratteri alfabetici:

```csharp
[RegularExpression(@"^[A-Z]+[a-zA-Z]*$")]
```

L'attributo `MaxLength` offre funzionalità simili a quelle dell'attributo `StringLength` ma non offre la convalida lato client.

La modifica apportata al modello di database ora richiede un aggiornamento dello schema del database. Si userà migrations per aggiornare lo schema senza perdere i dati eventualmente aggiunti al database tramite l'interfaccia utente dell'applicazione.

Salvare le modifiche e compilare il progetto. Quindi aprire una finestra dei comandi nella cartella di progetto ed eseguire i comandi seguenti:

```dotnetcli
dotnet ef migrations add MaxLengthOnNames
```

```dotnetcli
dotnet ef database update
```

Il comando `migrations add` segnala che può verificarsi la perdita di dati, perché la modifica riduce la lunghezza massima per due colonne.  Migrations crea un file denominato *\<timeStamp> _MaxLengthOnNames. cs*. Il metodo `Up` di questo file contiene codice che aggiorna il database per adattarlo al modello di dati corrente. Il comando `database update` ha eseguito tale codice.

Il timestamp che precede il nome del file delle migrazioni viene usato da Entity Framework per ordinare le migrazioni. È possibile creare più migrazioni prima di eseguire il comando update-database, dopodiché tutte le migrazioni vengono applicate nell'ordine in cui sono state create.

Eseguire l'app, selezionare la scheda **Students** (Studenti), fare clic su **Crea nuovo** e provare a immettere un nome con più di 50 caratteri. L'applicazione dovrebbe impedire questa operazione. 

### <a name="the-column-attribute"></a>Attributo Column

È possibile usare gli attributi anche per controllare il mapping delle classi e delle proprietà nel database. Si supponga di aver usato il nome `FirstMidName` per il campo first-name (Nome) perché il campo potrebbe contenere anche un secondo nome. Tuttavia si vuole che la colonna di database sia denominata `FirstName`, perché gli utenti che scrivono query ad hoc per il database sono abituati a tale nome. Per eseguire questo mapping è possibile usare l'attributo `Column`.

L'attributo `Column` specifica che quando viene creato il database, la colonna della tabella `Student` mappata sulla proprietà `FirstMidName` verrà denominata `FirstName`. In altri termini, quando il codice fa riferimento a `Student.FirstMidName` i dati provengono dalla colonna `FirstName` della tabella `Student` o vengono aggiornati in tale colonna. Se non si specificano nomi di colonna, le colonne avranno lo stesso nome della proprietà.

Nel file *Student.cs* aggiungere un'istruzione `using` per `System.ComponentModel.DataAnnotations.Schema` e aggiungere l'attributo del nome colonna alla proprietà `FirstMidName`, come illustrato nel codice evidenziato seguente:

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Column&highlight=4,14)]

L'aggiunta dell'attributo `Column` modifica il modello che supporta `SchoolContext` e che pertanto non corrisponderà al database.

Salvare le modifiche e compilare il progetto. Quindi aprire una finestra dei comandi nella cartella di progetto ed eseguire i comandi seguenti per creare un'altra migrazione:

```dotnetcli
dotnet ef migrations add ColumnFirstName
```

```dotnetcli
dotnet ef database update
```

In **Esplora oggetti di SQL Server** aprire il designer della tabella **Student** (Studente) facendo doppio clic sulla tabella.

![Tabella Students (Studenti) in SSOX dopo le migrazioni](complex-data-model/_static/ssox-after-migration.png)

Prima dell'applicazione delle prime due migrazioni, le colonne del nome erano di tipo nvarchar(MAX). Ora sono di tipo nvarchar(50) e il nome colonna è cambiato da FirstMidName a FirstName.

> [!Note]
> Se si prova a compilare prima di aver creato tutte le classi di entità delle sezioni seguenti, possono verificarsi errori di compilazione.

## <a name="changes-to-student-entity"></a>Modifiche all'entità Student

![Entità Student](complex-data-model/_static/student-entity.png)

In *Models/Student.cs* sostituire il codice aggiunto in precedenza con il codice seguente. Le modifiche sono evidenziate.

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_BeforeInheritance&highlight=11,13,15,18,22,24-31)]

### <a name="the-required-attribute"></a>Attributo Required

L'attributo `Required` rende obbligatori i campi delle proprietà del nome. L'attributo `Required` non è necessario per i tipi non nullable quali i tipi del valore (DateTime, int, double, float e così via). I tipi che non possono essere null vengono considerati automaticamente come campi obbligatori.

L'attributo `Required` deve essere usato con `MinimumLength` per l'applicazione di `MinimumLength`.

```csharp
[Display(Name = "Last Name")]
[Required]
[StringLength(50, MinimumLength=2)]
public string LastName { get; set; }
```

### <a name="the-display-attribute"></a>Attributo Display

L'attributo `Display` specifica che la didascalia delle caselle di testo deve essere "First Name" (Nome), "Last Name" (Cognome), "Full Name" (Nome e cognome) ed "Enrollment Date" (Data di iscrizione) anziché il nome della proprietà (senza spazi tra le parole) in ogni istanza.

### <a name="the-fullname-calculated-property"></a>Proprietà calcolata FullName

`FullName` è una proprietà calcolata che restituisce un valore creato concatenando altre due proprietà. Di conseguenza ha una sola funzione di accesso get e nel database non viene generata nessuna colonna `FullName`.

## <a name="create-instructor-entity"></a>Creare l'entità Instructor

![Entità Instructor](complex-data-model/_static/instructor-entity.png)

Creare *Models/Instructor.cs* sostituendo il codice del modello con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Instructor.cs?name=snippet_BeforeInheritance)]

Si noti che molte proprietà sono uguali nelle entità Student e Instructor. Nell'esercitazione [Implementing Inheritance](inheritance.md) (Implementazione dell'ereditarietà) più avanti in questa serie si effettuerà il refactoring di questo codice per eliminare la ridondanza.

È possibile posizionare più attributi su una riga, pertanto è anche possibile scrivere gli attributi `HireDate` come indicato di seguito:

```csharp
[DataType(DataType.Date),Display(Name = "Hire Date"),DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

### <a name="the-courseassignments-and-officeassignment-navigation-properties"></a>Proprietà di navigazione CourseAssignments e OfficeAssignment

Le proprietà `CourseAssignments` e `OfficeAssignment` sono proprietà di navigazione.

Un insegnante può tenere un numero qualsiasi di corsi, pertanto `CourseAssignments` è definita come raccolta.

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

Se una proprietà di navigazione può contenere più entità, il tipo della proprietà deve essere un elenco in cui le voci possono essere aggiunte, eliminate e aggiornate.  È possibile specificare `ICollection<T>` o un tipo come `List<T>` o `HashSet<T>`. Se si specifica `ICollection<T>`, per impostazione predefinita EF crea una raccolta `HashSet<T>`.

Il motivo per cui queste sono entità `CourseAssignment` viene illustrato più avanti nella sezione relativa alle relazioni molti-a-molti.

Le regole business di Contoso University specificano che un insegnante non può avere più di un ufficio, pertanto la proprietà `OfficeAssignment` contiene un'unica entità OfficeAssignment (che può essere null se non è assegnato alcun ufficio).

```csharp
public OfficeAssignment OfficeAssignment { get; set; }
```

## <a name="create-officeassignment-entity"></a>Creare l'entità OfficeAssignment

![Entità OfficeAssignment](complex-data-model/_static/officeassignment-entity.png)

Creare *Models/OfficeAssignment.cs* con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/OfficeAssignment.cs)]

### <a name="the-key-attribute"></a>Attributo Key

Esiste una relazione uno-a-zero-o-uno tra le entità Instructor e OfficeAssignment. L'assegnazione dell'ufficio esiste solo in relazione all'insegnante al quale viene assegnato l'ufficio, pertanto la chiave primaria dell'assegnazione è anche la chiave esterna per l'entità Instructor. Tuttavia Entity Framework non riconosce automaticamente InstructorID come chiave primaria di questa entità, perché il nome non rispetta la convenzione di denominazione ID o classnameID. Per identificare l'entità come chiave viene usato l'attributo `Key`:

```csharp
[Key]
public int InstructorID { get; set; }
```

È anche possibile usare l'attributo `Key` se l'entità dispone di una chiave primaria propria ma si vuole assegnare alla proprietà un nome diverso da classnameID o ID.

Per impostazione predefinita, EF considera la chiave come non generata dal database, perché la colonna è destinata a una relazione di identificazione.

### <a name="the-instructor-navigation-property"></a>Proprietà di navigazione Instructor

L'entità Instructor dispone di una proprietà di navigazione `OfficeAssignment` nullable (perché un docente potrebbe non avere un ufficio assegnato) e l'entità OfficeAssignment dispone di una proprietà di navigazione `Instructor` non nullable (perché un'assegnazione di ufficio non può esistere senza un insegnante: `InstructorID` è non nullable). Quando un'entità Instructor dispone di un'entità OfficeAssignment correlata, ogni entità include un riferimento all'altra entità nella relativa proprietà di navigazione.

È possibile inserire un attributo `[Required]` nella proprietà di navigazione Instructor per specificare che deve essere presente un insegnante correlato, ma questa operazione non è necessaria perché la chiave esterna `InstructorID` (che è anche la chiave per questa tabella) è di tipo non nullable.

## <a name="modify-course-entity"></a>Modificare l'entità Course

![Entità Course](complex-data-model/_static/course-entity.png)

In *Models/Course.cs* sostituire il codice aggiunto in precedenza con il codice seguente. Le modifiche sono evidenziate.

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Final&highlight=2,10,13,16,19,21,23)]

L'entità Course ha una proprietà di chiave esterna `DepartmentID` che fa riferimento all'entità Department correlata e include una proprietà di navigazione `Department`.

In Entity Framework non è necessario aggiungere una proprietà di chiave esterna al modello di dati se è disponibile una proprietà di navigazione per un'entità correlata.  EF crea automaticamente chiavi esterne nel database ogni volta che sono necessarie e crea [proprietà nascoste](/ef/core/modeling/shadow-properties) per tali chiavi. Tuttavia il fatto di avere la chiave esterna nel modello di dati può rendere più semplici ed efficienti gli aggiornamenti. Quando ad esempio si recupera un'entità Course per la modifica, l'entità Department è null se non viene caricata, pertanto prima di aggiornare l'entità Course è necessario recuperare l'entità Department. Quando la proprietà di chiave esterna `DepartmentID` viene inclusa nel modello di dati, non è necessario recuperare l'entità Department prima di eseguire l'aggiornamento.

### <a name="the-databasegenerated-attribute"></a>Attributo DatabaseGenerated

L'attributo `DatabaseGenerated` con il parametro `None` per la proprietà `CourseID` indica che i valori di chiave primaria vengono specificati dall'utente anziché essere generati dal database.

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.None)]
[Display(Name = "Number")]
public int CourseID { get; set; }
```

Per impostazione predefinita, Entity Framework presuppone che i valori di chiave primaria vengano generati dal database. Questa è la condizione ottimale nella maggior parte degli scenari. Tuttavia per le entità Course si userà un numero di corso specificato dall'utente, ad esempio il numero di serie 1000 per un reparto, il numero di serie 2000 per un altro reparto e così via.

Anche l'attributo `DatabaseGenerated` può essere usato anche per generare valori predefiniti, come nel caso delle colonne di database usate per registrare la data di creazione o aggiornamento di una riga.  Per altre informazioni, vedere [Generated Properties](/ef/core/modeling/generated-properties) (Proprietà generate).

### <a name="foreign-key-and-navigation-properties"></a>Proprietà chiave esterna e di navigazione

La proprietà di chiave esterna e le proprietà di navigazione nell'entità Course riflettono le relazioni seguenti:

Un corso viene assegnato a un solo reparto, pertanto sono presenti una chiave esterna `DepartmentID` e una proprietà di navigazione `Department` per i motivi indicati in precedenza.

```csharp
public int DepartmentID { get; set; }
public Department Department { get; set; }
```

Un corso può avere un numero qualsiasi di studenti iscritti, pertanto la proprietà di navigazione `Enrollments` è una raccolta:

```csharp
public ICollection<Enrollment> Enrollments { get; set; }
```

Un corso può essere impartito da più insegnanti, pertanto la proprietà di navigazione `CourseAssignments` è una raccolta (il tipo `CourseAssignment` è illustrato [più avanti](#many-to-many-relationships)):

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

## <a name="create-department-entity"></a>Creare l'entità Department

![Entità reparto](complex-data-model/_static/department-entity.png)

Creare *Models/Department.cs* con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Begin)]

### <a name="the-column-attribute"></a>Attributo Column

In precedenza l'attributo `Column` è stato usato per modificare il mapping del nome di colonna. Nel codice dell'entità Department l'attributo `Column` viene usato per modificare il mapping del tipo di dati SQL e per far sì che la colonna venga definita usando il tipo SQL Server money nel database:

```csharp
[Column(TypeName="money")]
public decimal Budget { get; set; }
```

In genere il mapping della colonna non è necessario, perché Entity Framework sceglie il tipo di dati SQL Server appropriato in base al tipo CLR definito per la proprietà. Il tipo CLR `decimal` esegue il mapping a un tipo SQL Server `decimal`. In questo caso tuttavia si sa che la colonna includerà importi in valuta, pertanto il tipo di dati money è più appropriato.

### <a name="foreign-key-and-navigation-properties"></a>Proprietà chiave esterna e di navigazione

Le proprietà di chiave esterna e le proprietà di navigazione riflettono le relazioni seguenti:

Un reparto può avere o meno un amministratore e un amministratore è sempre un insegnante. Di conseguenza la proprietà `InstructorID` è inclusa come chiave esterna per l'entità Instructor e dopo l'indicazione del tipo `int` viene aggiunto un punto interrogativo, per contrassegnare la proprietà come nullable. La proprietà di navigazione è denominata `Administrator` ma contiene un'entità Instructor:

```csharp
public int? InstructorID { get; set; }
public Instructor Administrator { get; set; }
```

Un reparto può avere molti corsi, pertanto è disponibile una proprietà di navigazione Courses:

```csharp
public ICollection<Course> Courses { get; set; }
```

> [!NOTE]
> Per convenzione, Entity Framework consente l'eliminazione a catena per le chiavi esterne non nullable e per le relazioni molti-a-molti. Ciò può determinare regole di eliminazione a catena circolari, che generano un'eccezione quando si prova ad aggiungere una migrazione. Ad esempio, se la proprietà Department.InstructorID non è stata definita come nullable, EF configura una regola di eliminazione a catena per eliminare il reparto quando si elimina l'insegnante, che non è il risultato desiderato. Se le regole business richiedono che la proprietà `InstructorID` sia non nullable, è necessario usare la seguente istruzione API Fluent per disattivare l'eliminazione a catena nella relazione:
>
> ```csharp
> modelBuilder.Entity<Department>()
>    .HasOne(d => d.Administrator)
>    .WithMany()
>    .OnDelete(DeleteBehavior.Restrict)
> ```

## <a name="modify-enrollment-entity"></a>Modificare l'entità Enrollment

![Entità Enrollment](complex-data-model/_static/enrollment-entity.png)

In *Models/Enrollment.cs* sostituire il codice aggiunto in precedenza con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Final&highlight=1-2,16)]

### <a name="foreign-key-and-navigation-properties"></a>Proprietà chiave esterna e di navigazione

Le proprietà di chiave esterna e le proprietà di navigazione riflettono le relazioni seguenti:

Un record di iscrizione è relativo a un singolo corso, pertanto sono presenti una proprietà di chiave esterna `CourseID` e una proprietà di navigazione `Course`:

```csharp
public int CourseID { get; set; }
public Course Course { get; set; }
```

Un record di iscrizione è relativo a un singolo studente, pertanto sono presenti una proprietà di chiave esterna `StudentID` e una proprietà di navigazione `Student`:

```csharp
public int StudentID { get; set; }
public Student Student { get; set; }
```

## <a name="many-to-many-relationships"></a>Relazioni molti-a-molti

È presente una relazione molti-a-molti tra le entità Student e Course e l'entità Enrollment funziona come una tabella di join molti-a-molti *con payload* nel database. "Con payload" significa che la tabella Enrollment contiene dati aggiuntivi oltre alle chiavi esterne delle tabelle di join (in questo caso una chiave primaria e una proprietà Grade).

La figura seguente illustra l'aspetto di queste relazioni in un diagramma di entità. Il diagramma è stato generato con Entity Framework Power Tools per EF 6.x. La creazione del diagramma non fa parte dell'esercitazione e il diagramma viene visualizzato solo come esempio.

![Relazione molti-a-molti Student-Course (Studente-Corso)](complex-data-model/_static/student-course.png)

Ogni riga della relazione inizia con un 1 e termina con un asterisco (*), per indicare una relazione uno-a-molti.

Se la tabella Enrollment (Iscrizione) non include informazioni sul livello è sufficiente che contenga le due chiavi esterne CourseID e StudentID. In questo caso sarebbe una tabella di join molti-a-molti senza payload (o tabella di join pura) del database. Le entità Instructor e Course dispongono di questo tipo di relazione molti-a-molti e il passaggio successivo consiste nel creare una classe di entità che faccia da tabella di join senza payload.

Le tabelle di join implicite per le relazioni molti-a-molti sono supportate in EF 6.x, ma non in EF Core. Per altre informazioni, vedere l'[approfondimento nel repository EF Core di GitHub repository](https://github.com/aspnet/EntityFramework/issues/1368).

## <a name="the-courseassignment-entity"></a>Entità CourseAssignment

![Entità CourseAssignment](complex-data-model/_static/courseassignment-entity.png)

Creare *Models/CourseAssignment.cs* con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/CourseAssignment.cs)]

### <a name="join-entity-names"></a>Nomi delle entità di join

È necessaria una tabella di join del database per la relazione molti-a-molti Instructor-Courses e tale relazione deve essere rappresentata da un set di entità. È pratica comune assegnare a un'entità di join il nome `EntityName1EntityName2`, che in questo caso sarebbe `CourseInstructor`. È tuttavia consigliabile scegliere un nome che descrive la relazione. I modelli di dati sono inizialmente semplici, quindi crescono e diventano più complessi. In molti casi ai join senza payload vengono assegnati payload in un secondo momento. Se si inizia con un nome di entità descrittivo, non sarà necessario modificarlo successivamente. Idealmente l'entità di join dovrebbe avere il proprio nome naturale (se possibile composto da un'unica parola) nel dominio di business. Ad esempio Books (Documentazione) e Customers (Clienti) potrebbero essere collegati mediante Ratings (Valutazioni). Per questa relazione `CourseAssignment` è una soluzione migliore rispetto a `CourseInstructor`.

### <a name="composite-key"></a>Chiave composta

Dato che le chiavi esterne non sono nullable e insieme identificano in modo univoco ogni riga della tabella, non è necessario avere una chiave primaria separata. Le proprietà *InstructorID* e *CourseID* funzionano come una chiave primaria composta. L'unico modo per identificare le chiavi primarie composte in Entity Framework è l'uso di *API Fluent* (l'operazione non può essere eseguita con gli attributi). Nella sezione successiva si vedrà come configurare la chiave primaria composta.

La chiave composta garantisce che anche se è possibile avere più righe per un corso e più righe per un insegnante, non è possibile avere più righe per lo stesso insegnante e lo stesso corso. L'entità di join `Enrollment` definisce la propria chiave primaria, pertanto sono possibili i duplicati di questo tipo. Per evitare tali duplicati è possibile aggiungere un indice univoco ai campi chiave esterna o configurare `Enrollment` con una chiave primaria composta simile a `CourseAssignment`. Per altre informazioni, vedere [Indexes](/ef/core/modeling/indexes) (Indici).

## <a name="update-the-database-context"></a>Aggiornare il contesto di database

Aggiungere il codice evidenziato di seguito al file *Data/SchoolContext.cs*:

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_BeforeInheritance&highlight=15-18,25-31)]

Questo codice aggiunge le nuove entità e configura la chiave primaria composta dell'entità CourseAssignment.

## <a name="about-a-fluent-api-alternative"></a>Informazioni su un'alternativa API Fluent

Il codice nel metodo `OnModelCreating` della classe `DbContext` usa l'*API Fluent* per configurare il comportamento di Entity Framework. L'API è denominata "API Fluent" perché viene spesso usata unendo una serie di chiamate di metodi in un'unica istruzione, come in questo esempio tratto dalla [documentazione di EF Core](/ef/core/modeling/#use-fluent-api-to-configure-a-model):

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

In questa esercitazione si usa l'API Fluent solo per operazioni di mapping del database non eseguibili con gli attributi. È tuttavia possibile usare l'API Fluent anche per specificare la maggior parte delle regole di formattazione, convalida e mapping specificabili tramite gli attributi. Alcuni attributi quali `MinimumLength` non possono essere applicati con l'API Fluent. Come accennato in precedenza, `MinimumLength` non modifica lo schema, ma si limita ad applicare una regola di convalida lato client e lato server.

Alcuni sviluppatori preferiscono usare solo l'API Fluent, per dare un aspetto "ordinato" alle classi di entità. È possibile usare un misto di attributi e API Fluent e alcune personalizzazioni sono eseguibili solo mediante l'API Fluent, ma in generale è consigliabile scegliere uno dei due approcci e usarlo in modo il più possibile coerente. Se si usano entrambi gli approcci, tenere presente che ogni volta che si verifica un conflitto l'API Fluent esegue l'override degli attributi.

Per altre informazioni sul confronto tra attributi e API Fluent, vedere [Metodi di configurazione](/ef/core/modeling/).

## <a name="entity-diagram-showing-relationships"></a>Diagramma dell'entità che visualizza le relazioni

La figura seguente visualizza il diagramma creato da Entity Framework Power Tools per il modello School completato.

![Diagramma dell'entità](complex-data-model/_static/diagram.png)

Oltre alle linee delle relazioni uno-a-molti (da 1 a \*) è possibile visualizzare la linea della relazione uno-a-zero-o-uno (da 1 a 0..1) tra le entità Instructor e OfficeAssignment e la linea della relazione zero-o-uno-a-molti (da 0..1 a *) tra le entità Instructor e Department.

## <a name="seed-database-with-test-data"></a>Eseguire il seeding del database con dati di test

Sostituire il codice nel file *Data/DbInitializer.cs* con il codice seguente per includere dati di inizializzazione per le nuove entità create.

[!code-csharp[](intro/samples/cu/Data/DbInitializer.cs?name=snippet_Final)]

Come si è visto nella prima esercitazione, la maggior parte di questo codice crea semplicemente nuovi oggetti entità e carica i dati di esempio nelle proprietà in base alle esigenze di test. Osservare la modalità di gestione delle relazioni molti-a-molti: il codice crea relazioni tramite la creazione di entità nei set di entità di join `Enrollments` e `CourseAssignment`.

## <a name="add-a-migration"></a>Aggiungere una migrazione

Salvare le modifiche e compilare il progetto. Quindi aprire la finestra di comando nella cartella del progetto e immettere il comando `migrations add` (non eseguire ancora il comando update-database):

```dotnetcli
dotnet ef migrations add ComplexDataModel
```

Viene visualizzato un avviso sulla possibile perdita di dati.

```text
An operation was scaffolded that may result in the loss of data. Please review the migration for accuracy.
Done. To undo this action, use 'ef migrations remove'
```

Se si prova a eseguire il comando `database update` in questa fase (evitare di farlo), si ottiene il seguente errore:

> The ALTER TABLE statement conflicted with the FOREIGN KEY constraint "FK_dbo.Course_dbo.Department_DepartmentID". The conflict occurred in database "ContosoUniversity", table "dbo.Department", column 'DepartmentID' (L'istruzione ALTER TABLE è in conflitto con il vincolo FOREIGN KEY "FK_dbo.Course_dbo.Department_DepartmentID". Il conflitto si è verificato nella colonna 'DepartmentID' della tabella "dbo.Department"del database "ContosoUniversity").

In determinati casi, quando si eseguono migrazioni con dati esistenti è necessario inserire dati stub nel database per soddisfare i vincoli di chiave esterna. Il codice generato nel metodo `Up` aggiunge una chiave esterna DepartmentID alla tabella Course. Se sono già presenti righe nella tabella Course quando viene eseguito il codice, l'operazione `AddColumn` non riesce perché SQL Server non determina il valore da inserire nella colonna, che non può essere null. In questa esercitazione si esegue la migrazione in un nuovo database, ma in un'applicazione di produzione è necessario che la migrazione gestisca i dati esistenti. Le istruzioni che seguono visualizzano un esempio di esecuzione di questa operazione.

Per fare in modo che questa migrazione funzioni con i dati esistenti, è necessario modificare il codice per dare alla nuova colonna un valore predefinito e creare un reparto stub denominato "Temp" che svolga la funzione di reparto predefinito. Di conseguenza, dopo l'esecuzione del metodo `Up` tutte le righe Course esistenti saranno correlate al reparto "Temp".

* Aprire il file *{timestamp}_ComplexDataModel.cs*.

* Impostare come commento la riga di codice che aggiunge la colonna DepartmentID alla tabella Course.

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CommentOut&highlight=9-13)]

* Aggiungere il codice evidenziato seguente dopo il codice che crea la tabella Department:

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CreateDefaultValue&highlight=22-32)]

In un'applicazione di produzione è necessario creare codice o script per aggiungere righe Department e associare le righe Course alle nuove righe Department. In tal modo il reparto "Temp" o il valore predefinito nella colonna Course.DepartmentID non saranno più necessari.

Salvare le modifiche e compilare il progetto.

## <a name="change-the-connection-string"></a>Modificare la stringa di connessione

Ora la classe `DbInitializer` include nuovo codice che aggiunge dati di inizializzazione per le nuove entità a un database vuoto. Per fare in modo che EF crei un nuovo database vuoto, modificare il nome del database nella stringa di connessione in *appsettings.json* ContosoUniversity3 o un altro nome che non è stato usato nel computer che si sta usando.

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity3;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
```

Salvare la modifica in *appsettings.json* .

> [!NOTE]
> In alternativa alla modifica del nome del database, è possibile eliminare il database. Usare **Esplora oggetti di SQL Server** (SSOX) o il comando CLI `database drop`:
>
> ```dotnetcli
> dotnet ef database drop
> ```

## <a name="update-the-database"></a>Aggiornare il database

Dopo aver modificato il nome del database o eliminato il database, eseguire il comando `database update` nella finestra di comando per eseguire le migrazioni.

```dotnetcli
dotnet ef database update
```

Eseguire l'app per far sì che il metodo `DbInitializer.Initialize` venga eseguito e popoli il nuovo database.

Aprire il database in SSOX come in precedenza, quindi espandere il nodo **Tabelle** per visualizzare tutte le tabelle che sono state create. Se SSOX è ancora aperto dall'operazione precedente, fare clic sul pulsante **Aggiorna**.

![Tabelle in SSOX](complex-data-model/_static/ssox-tables.png)

Eseguire l'app per attivare il codice inizializzatore che esegue l'inizializzazione del database.

Fare clic con il pulsante destro del mouse sulla tabella **CourseAssignment** e selezionare **Visualizza dati** per verificare che la tabella contenga dati.

![Dati CourseAssignment in SSOX](complex-data-model/_static/ssox-ci-data.png)

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione:

> [!div class="checklist"]
> * Personalizzazione del modello di dati
> * Modifica dell'entità Student
> * Creazione dell'entità Instructor
> * Creazione dell'entità OfficeAssignment
> * Modifica dell'entità Course
> * Creazione dell'entità Department
> * Modifica dell'entità Enrollment
> * Aggiornamento del contesto di database
> * Seeding del database con dati di test
> * Aggiunta di una migrazione
> * Modifica della stringa di connessione
> * Aggiornamento del database

Passare all'esercitazione successiva per altre informazioni su come accedere ai dati correlati.

> [!div class="nextstepaction"]
> [Passaggio successivo: accedere ai dati correlati](read-related-data.md)
