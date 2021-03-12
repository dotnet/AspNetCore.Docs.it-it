---
title: 'Esercitazione: aggiornare i dati correlati-ASP.NET MVC con EF Core'
description: In questa esercitazione verrà effettuato l'aggiornamento di dati correlati tramite l'aggiornamento di campi di chiave esterna e proprietà di navigazione.
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
uid: data/ef-mvc/update-related-data
ms.openlocfilehash: dadfceca2431ace558700ae06417f01c99751419
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588282"
---
# <a name="tutorial-update-related-data---aspnet-mvc-with-ef-core"></a>Esercitazione: aggiornare i dati correlati-ASP.NET MVC con EF Core

Nell'esercitazione precedente sono stati visualizzati dati correlati. In questa esercitazione i dati correlati verranno aggiornati tramite l'aggiornamento di campi di chiave esterna e proprietà di navigazione.

Le figure seguenti illustrano alcune delle pagine che verranno usate.

![Pagina di modifica del corso](update-related-data/_static/course-edit.png)

![Pagina modifica insegnante](update-related-data/_static/instructor-edit-courses.png)

In questa esercitazione:

> [!div class="checklist"]
> * Personalizzare le pagine dei corsi
> * Aggiungere la pagina Edit per gli insegnanti
> * Aggiungere corsi alla pagina Edit
> * Aggiornare la pagina Delete
> * Aggiungere posizione dell'ufficio e corsi alla pagina Create

## <a name="prerequisites"></a>Prerequisiti

* [Leggere dati correlati](read-related-data.md)

## <a name="customize-courses-pages"></a>Personalizzare le pagine dei corsi

Quando viene creata, una nuova entità corso deve essere in relazione con un dipartimento esistente. Per semplificare il raggiungimento di questo obiettivo, il codice con scaffolding include i metodi del controller e le visualizzazioni di creazione e modifica includono un elenco a discesa per la selezione del dipartimento. L'elenco a discesa imposta la proprietà di chiave esterna `Course.DepartmentID`. Questo è tutto ciò che serve a Entity Framework per caricare la proprietà di navigazione `Department` con l'entità Department appropriata. Verrà usato il codice con scaffolding, che però verrà modificato leggermente per aggiungere la gestione degli errori e l'ordinamento dell'elenco a discesa.

In *CoursesController.cs* eliminare i quattro metodi Create ed Edit e sostituirli con il codice seguente:

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreateGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreatePost)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditPost)]

Dopo il metodo `Edit` HttpPost, creare un nuovo metodo che carichi le informazioni di dipartimento per l'elenco a discesa.

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_Departments)]

Il metodo `PopulateDepartmentsDropDownList` ottiene un elenco di tutti i dipartimenti ordinato per nome, crea una raccolta `SelectList` per un elenco a discesa e passa tale raccolta alla visualizzazione in `ViewBag`. Il metodo accetta il parametro facoltativo `selectedDepartment`, che consente al codice chiamante di specificare l'elemento che deve essere selezionato quando viene eseguito il rendering dell'elenco a discesa. La visualizzazione passerà il nome "DepartmentID" all'helper tag `<select>`, che quindi saprà di dover cercare nell'oggetto `ViewBag` una raccolta `SelectList` denominata "DepartmentID".

Il metodo `Create` HttpGet chiama il metodo `PopulateDepartmentsDropDownList` senza impostare l'elemento selezionato, perché per un nuovo corso il dipartimento non è ancora stabilito:

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=3&name=snippet_CreateGet)]

Il metodo `Edit` HttpGet imposta l'elemento selezionato, in base all'ID del dipartimento già assegnato al corso in fase di modifica:

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=15&name=snippet_EditGet)]

I metodi HttpPost sia per `Create` che per `Edit` includono anche codice che imposta l'elemento selezionato quando la pagina viene visualizzata di nuovo dopo un errore. Ciò garantisce che, quando la pagina viene visualizzata di nuovo per mostrare il messaggio di errore, il dipartimento selezionato rimane selezionato.

### <a name="add-asnotracking-to-details-and-delete-methods"></a>Aggiungere .AsNoTracking ai metodi Details e Delete

Per ottimizzare le prestazioni delle pagine dei dettagli e di eliminazione del corso, aggiungere chiamate a `AsNoTracking` nei metodi `Details` e `Delete` HttpGet.

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_Details)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_DeleteGet)]

### <a name="modify-the-course-views"></a>Modificare le visualizzazioni dei corsi

In *Views/Courses/Create.cshtml* aggiungere l'opzione "Select Department" (Selezionare il dipartimento) all'elenco a discesa **Department** (Dipartimento), modificare la didascalia da **DepartmentID** (ID dipartimento) a **Department** (Dipartimento) e aggiungere un messaggio di convalida.

[!code-cshtml[](intro/samples/cu/Views/Courses/Create.cshtml?highlight=2-6&range=29-35)]

In *Views/Courses/Edit.cshtml* apportare al campo Department (Dipartimento) la stessa modifica effettuata in *Create.cshtml*.

In *Views/Courses/Edit.cshtml* aggiungere anche il campo per il numero di corso prima del campo **Title** (Titolo). Poiché il numero di corso è la chiave primaria, viene visualizzato, ma non può essere modificato.

[!code-cshtml[](intro/samples/cu/Views/Courses/Edit.cshtml?range=15-18)]

Per il numero di corso è già presente un campo nascosto (`<input type="hidden">`) nella visualizzazione di modifica. L'aggiunta di un helper tag `<label>` non elimina la necessità del campo nascosto, poiché senza di questo il numero di corso non viene incluso nei dati inviati quando l'utente fa clic su **Save** (Salva) nella pagina **Edit** (Modifica).

In *Views/Courses/Delete.cshtml* aggiungere il campo del numero di corso nella parte superiore e modificare l'ID del dipartimento nel nome del dipartimento.

[!code-cshtml[](intro/samples/cu/Views/Courses/Delete.cshtml?highlight=14-19,36)]

In *Views/Courses/Details.cshtml* apportare la stessa modifica appena effettuata in *Delete.cshtml*.

### <a name="test-the-course-pages"></a>Testare le pagine del corso

Eseguire l'app, selezionare la scheda **Courses** (Corsi), fare clic su **Create New** (Crea nuovo) e immettere i dati per un nuovo corso:

![Pagina di creazione del corso](update-related-data/_static/course-create.png)

Fare clic su **Crea**. La pagina di indice dei corsi viene visualizzata con il nuovo corso aggiunto all'elenco. Il nome del dipartimento nell'elenco della pagina di indice deriva dalla proprietà di navigazione, che mostra che la relazione è stata stabilita correttamente.

Fare clic su **Edit** (Modifica) per un corso nella pagina di indice dei corsi.

![Pagina di modifica del corso](update-related-data/_static/course-edit.png)

Modificare i dati nella pagina e fare clic su **Save** (Salva). La pagina di indice dei corsi verrà visualizzata con i dati del corso aggiornati.

## <a name="add-instructors-edit-page"></a>Aggiungere la pagina Edit per gli insegnanti

Quando si modifica il record di un insegnante, è necessario essere in grado di aggiornare l'assegnazione dell'ufficio. L'entità Instructor ha una relazione uno-a-zero-o-uno con l'entità OfficeAssignment. Ciò significa che il codice deve gestire le situazioni seguenti:

* Se l'utente cancella l'assegnazione di un ufficio e questa originariamente rappresentava l'unico valore, eliminare l'entità OfficeAssignment.

* Se l'utente immette l'assegnazione di un ufficio e prima non c'era alcun valore, creare una nuova entità OfficeAssignment.

* Se l'utente modifica il valore di un'assegnazione di ufficio, modificare il valore in un'entità OfficeAssignment esistente.

### <a name="update-the-instructors-controller"></a>Aggiornare il controller Instructors

In *InstructorsController.cs* modificare il codice nel metodo `Edit` HttpGet in modo che carichi la proprietà di navigazione `OfficeAssignment` dell'entità Instructor e chiami `AsNoTracking`:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=8-11&name=snippet_EditGetOA)]

Sostituire il metodo `Edit` HttpPost con il codice seguente per gestire gli aggiornamenti delle assegnazioni di ufficio:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EditPostOA)]

Il codice esegue le seguenti attività:

* Modifica il nome del metodo in `EditPost` perché ora la firma è la stessa del metodo `Edit` HttpGet (l'attributo `ActionName` specifica che l'URL `/Edit/` viene ancora usato).

* Ottiene l'entità Instructor corrente dal database tramite il caricamento eager per la proprietà di navigazione `OfficeAssignment`. Ciò corrisponde a quanto effettuato nel metodo `Edit` HttpGet.

* Aggiorna l'entità Instructor recuperata con valori dallo strumento di associazione di modelli. L' `TryUpdateModel` Overload consente di dichiarare le proprietà che si desidera includere. In questo modo è possibile evitare l'overposting, come illustrato nella [seconda esercitazione](crud.md).

    <!-- Snippets don't play well with <ul> [!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=241-244)] -->

    ```csharp
    if (await TryUpdateModelAsync<Instructor>(
        instructorToUpdate,
        "",
        i => i.FirstMidName, i => i.LastName, i => i.HireDate, i => i.OfficeAssignment))
    ```

* Se la posizione dell'ufficio è vuota, impostare la proprietà Instructor.OfficeAssignment su Null, in modo che la riga correlata nella tabella OfficeAssignment venga eliminata.

    <!-- Snippets don't play well with <ul>  "intro/samples/cu/Controllers/InstructorsController.cs"} -->

    ```csharp
    if (String.IsNullOrWhiteSpace(instructorToUpdate.OfficeAssignment?.Location))
    {
        instructorToUpdate.OfficeAssignment = null;
    }
    ```

* Salva le modifiche nel database.

### <a name="update-the-instructor-edit-view"></a>Aggiornare la visualizzazione di modifica dell'insegnante

In *Views/Instructors/Edit.cshtml* aggiungere un nuovo campo per la modifica della posizione dell'ufficio alla fine, prima del pulsante **Save** (Salva):

[!code-cshtml[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=30-34)]

Eseguire l'app, selezionare la scheda **Instructors** (Insegnanti) e quindi fare clic su **Edit** (Modifica) per un insegnante. Modificare **Office Location** (Posizione ufficio) e fare clic su **Save** (Salva).

![Pagina di modifica dell'insegnante](update-related-data/_static/instructor-edit-office.png)

## <a name="add-courses-to-edit-page"></a>Aggiungere corsi alla pagina Edit

Gli insegnanti possono tenere un numero qualsiasi di corsi. A questo punto la pagina di modifica dell'insegnante verrà migliorata aggiungendo la possibilità di modificare le assegnazioni di corso tramite un gruppo di caselle di controllo, come illustrato nello screenshot seguente:

![Pagina di modifica dell'insegnante con corsi](update-related-data/_static/instructor-edit-courses.png)

Tra le entità Course e Instructor c'è una relazione molti-a-molti. Per aggiungere e rimuovere relazioni, aggiungere e rimuovere entità dal set di entità di join CourseAssignments.

L'interfaccia utente che consente di modificare i corsi assegnati a un insegnante è costituita da un gruppo di caselle di controllo. È visualizzata una casella di controllo per ogni corso nel database e le caselle corrispondenti ai corsi attualmente assegnati all'insegnante sono selezionate. L'utente può selezionare e deselezionare le caselle di controllo per modificare le assegnazioni dei corsi. Se il numero di corsi fosse molto superiore, sarebbe probabilmente consigliabile usare un altro metodo di presentazione dei dati nella visualizzazione, ma si userebbe lo stesso metodo di modifica di un'entità di join per creare o eliminare relazioni.

### <a name="update-the-instructors-controller"></a>Aggiornare il controller Instructors

Per fornire i dati alla visualizzazione dell'elenco di caselle di controllo, si userà una classe modello di visualizzazione.

Creare *AssignedCourseData.cs* nella cartella *SchoolViewModels* e sostituire il codice esistente con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/AssignedCourseData.cs)]

In *InstructorsController.cs* sostituire il metodo `Edit` HttpGet con il codice seguente. Le modifiche sono evidenziate.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=10,17,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&name=snippet_EditGetCourses)]

Il codice aggiunge il caricamento eager per la proprietà di navigazione `Courses` e chiama il nuovo metodo `PopulateAssignedCourseData` per fornire informazioni per la matrice di caselle di controllo tramite la classe modello di visualizzazione `AssignedCourseData`.

Il codice nel metodo `PopulateAssignedCourseData` legge tutte le entità Course per caricare un elenco di corsi tramite la classe modello di visualizzazione. Per ogni corso, il codice verifica se è presente nella proprietà di navigazione `Courses` dell'insegnante. Per creare un ricerca efficiente per la verifica dell'assegnazione di un corso all'insegnante, i corsi assegnati all'insegnante vengono inseriti in una raccolta `HashSet`. La proprietà `Assigned` è impostata su true per i corsi assegnati all'insegnante. La visualizzazione usa questa proprietà per determinare quali caselle di controllo devono essere visualizzate come selezionate. L'elenco, infine, viene passato alla visualizzazione in `ViewData`.

Aggiungere quindi il codice che viene eseguito quando l'utente fa clic su **Save** (Salva). Sostituire il metodo `EditPost` con il codice seguente e aggiungere un nuovo metodo che aggiorni la proprietà di navigazione `Courses` dell'entità Instructor.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=1,3,12,13,25,39-40&name=snippet_EditPostCourses)]

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=1-31)]

La firma del metodo è ora diversa dal metodo `Edit` HttpGet, quindi il nome del metodo cambia di nuovo da `EditPost` a `Edit`.

Poiché la visualizzazione non ha una raccolta di entità Course, lo strumento di associazione di modelli non può aggiornare automaticamente la proprietà di navigazione `CourseAssignments`. Anziché usare lo strumento di associazione di modelli per aggiornare la proprietà di navigazione `CourseAssignments`, questa operazione viene eseguita nel nuovo metodo `UpdateInstructorCourses`. È pertanto necessario escludere la `CourseAssignments` proprietà dall'associazione di modelli. Questa operazione non richiede alcuna modifica al codice che chiama `TryUpdateModel` perché si sta usando l'overload che richiede l'approvazione esplicita e `CourseAssignments` non è incluso nell'elenco di inclusione.

Se nessuna casella di controllo è selezionata, il codice in `UpdateInstructorCourses` inizializza la proprietà di navigazione `CourseAssignments` con una raccolta vuota e torna:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=3-7)]

Il codice quindi esegue il ciclo di tutti i corsi nel database e controlla ogni corso a fronte di quelli assegnati all'insegnante rispetto a quelli selezionati nella visualizzazione. Per facilitare l'esecuzione di ricerche efficienti, le ultime due raccolte sono archiviate all'interno di oggetti `HashSet`.

Se la casella di controllo di un corso è selezionata ma il corso non è presente nella proprietà di navigazione `Instructor.CourseAssignments`, il corso viene aggiunto alla raccolta nella proprietà di navigazione.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=14-20&name=snippet_UpdateCourses)]

Se la casella di controllo di un corso non è selezionata ma il corso è presente nella proprietà di navigazione `Instructor.CourseAssignments`, il corso viene rimosso dalla proprietà di navigazione.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=21-29&name=snippet_UpdateCourses)]

### <a name="update-the-instructor-views"></a>Aggiornare le visualizzazioni dell'insegnante

In *Views/Instructors/Edit.cshtml* aggiungere un campo **Courses** (Corsi) con una matrice di caselle di controllo aggiungendo il codice seguente immediatamente dopo gli elementi `div` per il campo **Office** (Ufficio) e prima dell'elemento `div` per il pulsante **Save** (Salva).

<a id="notepad"></a>
> [!NOTE]
> Quando si incolla il codice in Visual Studio, è possibile che le interruzioni di riga vengano modificate in un modo che danneggia il codice. Se il codice ha un aspetto diverso dopo aver incollato, premere CTRL+Z una volta per annullare la formattazione automatica. Ciò corregge le interruzioni di riga, che vengono visualizzate come illustrato qui. Il rientro non deve necessariamente essere perfetto, ma le righe `@:</tr><tr>`, `@:<td>`, `@:</td>` e `@:</tr>` devono trovarsi in una sola riga, come illustrato. In caso contrario, viene visualizzato un errore di runtime. Dopo aver selezionato il blocco di nuovo codice, premere Tab tre volte per allineare il nuovo codice con il codice esistente. Questo problema è stato risolto in Visual Studio 2019.

[!code-cshtml[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=35-61)]

Questo codice crea una tabella HTML con tre colonne. In ogni colonna è presente una casella di controllo seguita da una didascalia costituita dal numero di corso e dal titolo. Le caselle di controllo hanno tutte lo stesso nome ("selectedCourses"). Questo informa lo strumento di associazione di modelli che devono essere considerate come un gruppo. L'attributo value di ogni casella di controllo è impostato sul valore di `CourseID`. Quando la pagina viene pubblicata, lo strumento di associazione di modelli passa al controller una matrice costituita dai valori `CourseID` delle sole caselle di controllo selezionate.

Quando viene eseguito il rendering iniziale delle caselle di controllo, quelle corrispondenti ai corsi assegnati all'insegnante hanno attributi checked, che le rendono selezionate (visualizzate come selezionate).

Eseguire l'app, selezionare la scheda **Instructors** (Insegnanti) e fare clic su **Edit** (Modifica) per un insegnante per visualizzare la pagina **Edit** (Modifica).

![Pagina di modifica dell'insegnante con corsi](update-related-data/_static/instructor-edit-courses.png)

Modificare alcune assegnazioni di corsi e fare clic su Save (Salva). Le modifiche effettuate si riflettono nella pagina di indice.

> [!NOTE]
> L'approccio qui adottato per la modifica dei dati dei corsi degli insegnanti funziona bene quando è presente un numero limitato di corsi. Per raccolte molto più grandi, sarebbero necessari un'interfaccia utente diversa e un altro metodo di aggiornamento.

## <a name="update-delete-page"></a>Aggiornare la pagina Delete

In *InstructorsController.cs* eliminare il metodo `DeleteConfirmed` e sostituirlo con il codice seguente.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=5-7,9-12&name=snippet_DeleteConfirmed)]

Questo codice apporta le modifiche seguenti:

* Esegue il caricamento eager per la proprietà di navigazione `CourseAssignments`. È necessario includere questa proprietà o EF non sarà a conoscenza delle entità `CourseAssignment` correlate e non le eliminerà. Per evitare la necessità di leggerle qui, è possibile configurare l'eliminazione a catena nel database.

* Se l'insegnante da eliminare è assegnato come responsabile di un dipartimento, tale assegnazione viene rimossa dal dipartimento.

## <a name="add-office-location-and-courses-to-create-page"></a>Aggiungere posizione dell'ufficio e corsi alla pagina Create

In *InstructorsController.cs*, eliminare i metodi `Create` HttpGet e HttpPost e quindi sostituirli con il codice seguente:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Create&highlight=3-5,12,14-22,29)]

Questo codice è simile a quello dei metodi `Edit`, ad eccezione del fatto che inizialmente non è selezionato alcun corso. Il metodo `Create` HttpGet chiama il metodo `PopulateAssignedCourseData` non perché possono essere presenti corsi selezionati, ma per fornire una raccolta vuota per il ciclo `foreach` nella visualizzazioni (in caso contrario il codice di visualizzazione genera un'eccezione di riferimento Null).

Il metodo `Create` HttpPost aggiunge i corsi selezionati alla proprietà di navigazione `CourseAssignments` prima di controllare la presenza di errori di convalida e aggiungere il nuovo insegnante al database. I corsi vengono aggiunti anche in caso di errori di modello. Quindi se si verificano errori di questo tipo (ad esempio se l'utente digita una data non valida) e la pagina viene visualizzata di nuovo con un messaggio di errore, tutte le selezioni relative ai corsi effettuate vengono ripristinate automaticamente.

Si noti che, perché sia possibile aggiungere corsi alla proprietà di navigazione `CourseAssignments`, è necessario inizializzare la proprietà come raccolta vuota:

```csharp
instructor.CourseAssignments = new List<CourseAssignment>();
```

Anziché all'interno di codice di controllo, è possibile eseguire questa operazione nel modello Instructor modificando il getter della proprietà in modo da creare automaticamente la raccolta, se questa non esiste, come illustrato nell'esempio seguente:

```csharp
private ICollection<CourseAssignment> _courseAssignments;
public ICollection<CourseAssignment> CourseAssignments
{
    get
    {
        return _courseAssignments ?? (_courseAssignments = new List<CourseAssignment>());
    }
    set
    {
        _courseAssignments = value;
    }
}
```

Se si modifica la proprietà `CourseAssignments` in questo modo, è possibile rimuovere il codice di inizializzazione esplicita della proprietà nel controller.

In *Views/Instructor/Create.cshtml* aggiungere una casella di testo per la posizione dell'ufficio per i corsi prima del pulsante Submit (Invia). Come nel caso della pagina Edit (Modifica), [correggere la formattazione se Visual Studio riformatta il codice quando lo si incolla](#notepad).

[!code-cshtml[](intro/samples/cu/Views/Instructors/Create.cshtml?range=29-61)]

Eseguire il test eseguendo l'app e creando un insegnante.

## <a name="handling-transactions"></a>Gestione delle transazioni

Come spiegato nell' [esercitazione su CRUD](crud.md), per impostazione predefinita Entity Framework implementa in modo implicito le transazioni. Per gli scenari in cui è necessario un maggior controllo, ad esempio per includere le operazioni eseguite all'esterno di Entity Framework in una transazione, vedere [Transazioni](/ef/core/saving/transactions).

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione:

> [!div class="checklist"]
> * Personalizzare le pagine dei corsi
> * Aggiungere la pagina Edit per gli insegnanti
> * Aggiungere corsi alla pagina Edit
> * Aggiornare la pagina Delete
> * Aggiungere posizione dell'ufficio e corsi alla pagina Create

Passare all'esercitazione successiva per informazioni su come gestire i conflitti di concorrenza.

> [!div class="nextstepaction"]
> [Gestire i conflitti di concorrenza](concurrency.md)
