---
title: 'Esercitazione: leggere dati correlati-MVC ASP.NET con EF Core'
description: In questa esercitazione verranno letti e visualizzati dati correlati, ovvero dati che Entity Framework carica all'interno delle proprietà di navigazione.
author: rick-anderson
ms.author: riande
ms.date: 09/28/2019
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
uid: data/ef-mvc/read-related-data
ms.openlocfilehash: 9ea3bd7aaa075ae4601af51c7cc90f28a884a096
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587892"
---
# <a name="tutorial-read-related-data---aspnet-mvc-with-ef-core"></a>Esercitazione: leggere dati correlati-MVC ASP.NET con EF Core

Nell'esercitazione precedente è stato completato il modello di dati School. In questa esercitazione verranno letti e visualizzati dati correlati, ovvero dati che Entity Framework carica all'interno delle proprietà di navigazione.

Le figure seguenti illustrano le pagine che verranno usate.

![Pagina di indice dei corsi](read-related-data/_static/courses-index.png)

![Pagina di indice degli insegnanti](read-related-data/_static/instructors-index.png)

In questa esercitazione:

> [!div class="checklist"]
> * Scoprire come caricare i dati correlati
> * Creare una pagina Courses
> * Creare una pagina Instructors
> * Ottenere informazioni sul caricamento esplicito

## <a name="prerequisites"></a>Prerequisiti

* [Creare un modello di dati complesso](complex-data-model.md)

## <a name="learn-how-to-load-related-data"></a>Scoprire come caricare i dati correlati

Il software ORM (Object-Relational Mapping), ad esempio Entity Framework, può caricare dati correlati nelle proprietà di navigazione di un'entità in diversi modi:

* Caricamento eager. Quando l'entità viene letta, i dati correlati corrispondenti vengono recuperati insieme ad essa. Ciò in genere ha come risultato una query join singola che recupera tutti i dati necessari. Per specificare il caricamento eager in Entity Framework Core si usano i metodi `Include` e `ThenInclude`.

  ![Esempio di caricamento eager](read-related-data/_static/eager-loading.png)

  È possibile recuperare alcuni dati tramite query separate. In questo caso EF "corregge" le proprietà di navigazione.  In altre parole, EF aggiunge automaticamente le entità recuperate separatamente nelle proprietà di navigazione corrispondenti delle entità recuperate in precedenza. Per la query che recupera i dati correlati, è possibile usare il metodo `Load` anziché un metodo che restituisce un elenco o un oggetto, ad esempio `ToList` o `Single`.

  ![Esempio di query separate](read-related-data/_static/separate-queries.png)

* Caricamento esplicito. Quando un'entità viene letta per la prima volta, i dati correlati non vengono recuperati. Il codice del caricamento consente di recuperare i dati correlati se sono necessari. Come nel caso del caricamento eager con query separate, il caricamento esplicito ha come risultato l'invio di più query al database. La differenza è che con il caricamento esplicito il codice specifica le proprietà di navigazione da caricare. In Entity Framework Core 1.1, per eseguire il caricamento esplicito è possibile usare il metodo `Load`. Ad esempio:

  ![Esempio di caricamento esplicito](read-related-data/_static/explicit-loading.png)

* Caricamento lazy. Quando un'entità viene letta per la prima volta, i dati correlati non vengono recuperati. La prima volta che si tenta di accedere a una proprietà di navigazione, tuttavia, i dati necessari per quest'ultima vengono recuperati automaticamente. Una query viene inviata al database ogni volta che si tenta di ottenere dati da una proprietà di navigazione per la prima volta. Entity Framework Core 1.0 non supporta il caricamento lazy.

### <a name="performance-considerations"></a>Considerazioni sulle prestazioni

Se si sa di aver bisogno di dati correlati per tutte le entità recuperate, il caricamento eager spesso garantisce le prestazioni migliori, perché l'invio di un'unica query al database è in genere più efficiente dell'invio di query separate per ogni entità recuperata. Si supponga ad esempio che ogni dipartimento abbia dieci corsi correlati. Il caricamento eager di tutti i dati correlati comporta un'unica query (join) e un unico round trip al database. Una query separata per i corsi di ogni dipartimento comporta undici round trip al database. I round trip aggiuntivi al database influiscono in modo particolarmente negativo sulle prestazioni quando la latenza è elevata.

D'altra parte, in alcuni scenari query separate sono più efficienti. Il caricamento eager di tutti i dati correlati in una sola query può causare la generazione di un join molto complesso, che SQL Server non è in grado di elaborare in modo efficiente. Oppure se è necessario accedere alle proprietà di navigazione di un'entità solo per un subset di un set di entità in corso di elaborazione, query separate potrebbero offrire prestazioni migliori perché il caricamento immediato di tutti i dati recupererebbe più dati di quelli necessari. Se le prestazioni rappresentano un aspetto essenziale, per avere la certezza di scegliere il metodo più efficiente è consigliabile testare le prestazioni di entrambi i tipi di caricamento.

## <a name="create-a-courses-page"></a>Creare una pagina Courses

L'entità Course include una proprietà di navigazione contenente l'entità Department del corso assegnato al dipartimento. Per visualizzare il nome del dipartimento assegnato in un elenco di corsi, è necessario ottenere la proprietà Name dell'entità Department nella proprietà di navigazione `Course.Department`.

Creare un controller denominato CoursesController per il tipo di entità Course con le stesse opzioni del **controller MVC con visualizzazioni, tramite lo scaffolder Entity Framework** usato in precedenza per il controller Students, come illustrato nella figura seguente:

![Aggiungere il controller per i corsi](read-related-data/_static/add-courses-controller.png)

Aprire *CoursesController.cs* ed esaminare il metodo `Index`. Lo scaffolding automatico ha specificato il caricamento eager per la proprietà di navigazione `Department` tramite il metodo `Include`.

Sostituire il metodo `Index` con il codice seguente, che usa un nome più appropriato per l'`IQueryable` che restituisce entità Course (`courses` anziché `schoolContext`):

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_RevisedIndexMethod)]

Aprire *Views/Courses/Index.cshtml* e sostituire il codice del modello con il codice seguente. Le modifiche sono evidenziate:

[!code-cshtml[](intro/samples/cu/Views/Courses/Index.cshtml?highlight=4,7,15-17,34-36,44)]

Al codice con scaffolding sono state apportate le modifiche seguenti:

* Il titolo è stato modificato da Index (Indice) a Courses (Corsi).

* È stata aggiunta la colonna **Number** (Numero) con il valore della proprietà `CourseID`. Per impostazione predefinita, non viene eseguito lo scaffolding delle chiavi primarie, perché in genere non sono significative per gli utenti finali. In questo caso, tuttavia, la chiave primaria è significativa ed è necessario visualizzarla.

* Modificare la colonna **Department** (Dipartimento) per visualizzare il nome del dipartimento. Il codice visualizza la proprietà `Name` dell'entità Department che viene caricata nella proprietà di navigazione `Department`:

  ```html
  @Html.DisplayFor(modelItem => item.Department.Name)
  ```

Eseguire l'app e selezionare la scheda **Courses** (Corsi) per visualizzare l'elenco con i nomi dei dipartimenti.

![Pagina di indice dei corsi](read-related-data/_static/courses-index.png)

## <a name="create-an-instructors-page"></a>Creare una pagina Instructors

In questa sezione verranno creati un controller e una visualizzazione per l'entità Instructor allo scopo di visualizzare la pagina Instructors:

![Pagina di indice degli insegnanti](read-related-data/_static/instructors-index.png)

Questa pagina legge e visualizza dati correlati nei modi seguenti:

* L'elenco di insegnanti visualizza dati correlati provenienti dall'entità OfficeAssignment. Tra le entità Instructor e OfficeAssignment esiste una relazione uno-a-zero-o-uno. Per le entità OfficeAssignment verrà usato il caricamento eager. Come spiegato in precedenza, il caricamento eager è in genere più efficiente quando sono necessari i dati correlati per tutte le righe recuperate della tabella primaria. In questo caso, si vogliono visualizzare le assegnazioni di ufficio per tutti gli insegnanti visualizzati.

* Quando l'utente seleziona un insegnante, vengono visualizzate le entità Course correlate. Tra le entità Instructor e Course esiste una relazione molti-a-molti. Si userà il caricamento eager per le entità Course e le entità Department correlate. In questo caso, query separate potrebbero essere più efficienti, perché i corsi sono necessari solo per l'insegnante selezionato. Questo esempio, tuttavia, illustra come usare il caricamento eager per le proprietà di navigazione con entità esse stesse all'interno di proprietà di navigazione.

* Quando l'utente seleziona un corso, vengono visualizzati i dati correlati dal set di entità Enrollments. Tra le entità Course ed Enrollment esiste una relazione uno-a-molti. Per le entità Enrollment e le entità Student correlate si useranno query separate.

### <a name="create-a-view-model-for-the-instructor-index-view"></a>Creare un modello per la visualizzazione dell'indice degli insegnanti

La pagina Instructors (Insegnanti) mostra i dati di tre tabelle diverse. Verrà quindi creato un modello di visualizzazione che includa tre proprietà, ognuna contenente i dati di una delle tabelle.

Nella cartella *SchoolViewModels* creare *InstructorIndexData.cs* e sostituire il codice esistente con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/InstructorIndexData.cs)]

### <a name="create-the-instructor-controller-and-views"></a>Creare il controller e le visualizzazioni per gli insegnanti

Creare un controller per gli insegnanti con azioni di lettura/scrittura EF come illustrato nella figura seguente:

![Aggiungere il controller per gli insegnanti](read-related-data/_static/add-instructors-controller.png)

Aprire *InstructorsController.cs* e aggiungere un'istruzione using tramite lo spazio dei nomi ViewModel:

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Using)]

Sostituire il metodo Index con il codice seguente per eseguire il caricamento eager di dati correlati e inserirlo nel modello di visualizzazione.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EagerLoading)]

Il metodo accetta dati di route facoltativi (`id`) e un parametro di stringa di query (`courseID`) che forniscono i valori relativi all'ID dell'insegnante e del corso selezionati. I parametri sono forniti dai collegamenti ipertestuali **Select** (Seleziona) nella pagina.

Il codice inizia creando un'istanza del modello di visualizzazione e inserendola nell'elenco degli insegnanti. Il codice specifica il caricamento eager delle proprietà di navigazione `Instructor.OfficeAssignment` e `Instructor.CourseAssignments`. All'interno della proprietà `CourseAssignments` viene caricata la proprietà `Course` e all'interno di questa vengono caricate le proprietà `Enrollments` e `Department`, e all'interno di ogni entità `Enrollment` viene caricata la proprietà `Student`.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude)]

Dato che la visualizzazione richiede sempre l'entità OfficeAssignment, è più efficiente recuperare quest'ultima nella stessa query. Le entità Course sono necessarie quando viene selezionato un insegnante nella pagina Web. Un'unica query, quindi, è più efficiente di più query solo se nella maggior parte dei casi la pagina viene visualizzata con un corso selezionato.

Il codice ripete `CourseAssignments` e `Course` perché da `Course` sono necessarie due proprietà. La prima stringa delle chiamate `ThenInclude` ottiene `CourseAssignment.Course`, `Course.Enrollments` e `Enrollment.Student`.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=3-6)]

A questo punto nel codice, un'altra chiamata `ThenInclude` riguarda le proprietà di navigazione di `Student`, che non sono necessarie. Ma chiamando `Include` il codice ricomincia con le proprietà di `Instructor`. È quindi necessario ripetere la sequenza, questa volta specificando `Course.Department` anziché `Course.Enrollments`.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=7-9)]

Il codice seguente viene eseguito quando è stato selezionato un insegnante. L'insegnante selezionato viene recuperato dall'elenco di insegnanti nel modello di visualizzazione. La proprietà `Courses` del modello di visualizzazione viene quindi caricata con le entità Course dalla proprietà di navigazione `CourseAssignments` di tale insegnante.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=56-62)]

Il metodo `Where` restituisce una raccolta, ma in questo caso i criteri passati a tale metodo hanno come risultato la restituzione di una sola entità Instructor. Il metodo `Single` converte la raccolta in un'entità Instructor singola, che consente l'accesso alla proprietà `CourseAssignments` di tale entità. La proprietà `CourseAssignments` contiene entità `CourseAssignment`, di cui si vogliono solo le entità `Course` correlate.

Usare il metodo `Single` per una raccolta quando si sa che la raccolta ha un solo elemento. Il metodo Single genera un'eccezione se la raccolta passata è vuota o se contiene più elementi. In alternativa, è possibile usare `SingleOrDefault`, che restituisce un valore predefinito (Null in questo caso) se la raccolta è vuota. In questo caso, tuttavia, ciò avrebbe comunque come risultato un'eccezione (a causa del tentativo di cercare una proprietà `Courses` in un riferimento Null) e il messaggio di eccezione indicherebbe meno chiaramente la causa del problema. Quando si chiama il metodo `Single`, è anche possibile passare la condizione Where anziché chiamare il metodo `Where` separatamente:

```csharp
.Single(i => i.ID == id.Value)
```

Anziché:

```csharp
.Where(i => i.ID == id.Value).Single()
```

Se è stato selezionato un corso, questo viene quindi recuperato dall'elenco dei corsi nel modello di visualizzazione. La proprietà `Enrollments` del modello di visualizzazione viene quindi caricata con le entità Enrollment dalla proprietà di navigazione `Enrollments` di tale corso.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=64-69)]

### <a name="modify-the-instructor-index-view"></a>Modificare la visualizzazione dell'indice degli insegnanti

In *Views/Instructors/Index.cshtml* sostituire il codice del modello con il codice seguente. Le modifiche sono evidenziate.

::: moniker range=">= aspnetcore-2.2"

[!code-cshtml[](intro/samples/5cu-snap/Views/Instructors/Index.cshtml?range=1-62&highlight=1,3-7,15-19,24,26-31,41-52,54)]

::: moniker-end

::: moniker range="<= aspnetcore-2.1"

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=1-62&highlight=1,3-7,15-19,24,26-31,41-52,54)]

::: moniker-end

Al codice esistente sono state apportate le modifiche seguenti:

* La classe del modello è stata modificata in `InstructorIndexData`.

* Il titolo pagina è stato modificato da **Index** (Indice) a **Instructors** (Insegnanti).

* È stata aggiunta la colonna **Office** (Ufficio) che visualizza `item.OfficeAssignment.Location` solo se `item.OfficeAssignment` non è Null. Poiché questa è una relazione uno-a-zero-o-uno, potrebbe non esserci un'entità OfficeAssignment correlata.

  ```cshtml
  @if (item.OfficeAssignment != null)
  {
      @item.OfficeAssignment.Location
  }
  ```

* È stata aggiunta la colonna **Courses** (Corsi) che visualizza i corsi tenuti da ogni insegnante. Per ulteriori informazioni, vedere la sezione relativa alla [transizione di riga esplicita](xref:mvc/views/razor#explicit-line-transition) dell' Razor articolo della sintassi.

* Aggiunto codice che aggiunge in modo condizionale una classe CSS bootstrap all' `tr` elemento dell'insegnante selezionato. Questa classe imposta un colore di sfondo per la riga selezionata.

* È stato aggiunto un nuovo collegamento ipertestuale con etichetta **Select** (Seleziona) immediatamente prima degli altri collegamenti in ogni riga.Ciò comporta l'invio dell'ID dell'insegnante selezionato al metodo `Index`.

  ```cshtml
  <a asp-action="Index" asp-route-id="@item.ID">Select</a> |
  ```

Eseguire l'app e selezionare la scheda **Instructors (insegnanti** ). La pagina Visualizza la proprietà location delle entità OfficeAssignment correlate e una cella della tabella vuota quando non esiste alcuna entità OfficeAssignment correlata.

![Pagina di indice degli insegnanti con nessuna selezione](read-related-data/_static/instructors-index-no-selection.png)

Nel file *Views/Instructors/Index.cshtml*, dopo l'elemento di chiusura della tabella (alla fine del file), aggiungere il codice seguente. Quando è selezionato un insegnante, Questo codice visualizza un elenco dei corsi correlati all'insegnante stesso.

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=63-98)]

Questo codice legge la proprietà `Courses` del modello di visualizzazione per visualizzare l'elenco dei corsi. Fornisce anche il collegamento ipertestuale **Select** (Seleziona) che invia l'ID del corso selezionato al metodo di azione `Index`.

Aggiornare la pagina e selezionare un insegnante. È ora possibile vedere una griglia con i corsi assegnati all'insegnante selezionato. Per ogni corso è possibile vedere il nome del dipartimento assegnato.

![Pagina di indice degli insegnanti con un insegnante selezionato](read-related-data/_static/instructors-index-instructor-selected.png)

Dopo il blocco di codice appena aggiunto, aggiungere il codice seguente. Quando è selezionato un corso, questo codice visualizza l'elenco degli studenti iscritti al corso selezionato.

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=103-125)]

Il codice legge la proprietà Enrollments del modello di visualizzazione per visualizzare l'elenco degli studenti iscritti al corso.

Aggiornare di nuovo la pagina e selezionare un insegnante. Selezionare quindi un corso per visualizzare l'elenco degli studenti iscritti e i voti corrispondenti.

![Pagina di indice degli insegnanti con un corso selezionato](read-related-data/_static/instructors-index.png)

## <a name="about-explicit-loading"></a>Informazioni sul caricamento esplicito

Quando è stato recuperato l'elenco degli insegnanti in *InstructorsController.cs*, per la proprietà di navigazione `CourseAssignments` è stato specificato il caricamento eager.

Si supponga che gli utenti vogliano visualizzare solo raramente le iscrizioni per un corso e un insegnante selezionati. In tal caso, è consigliabile caricare i dati delle iscrizioni solo se richiesti. Per vedere un esempio di come eseguire il caricamento esplicito, sostituire il metodo `Index` con il codice seguente, che rimuove il caricamento eager per Enrollments e carica questa proprietà in modo esplicito. Le modifiche al codice sono evidenziate.

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ExplicitLoading&highlight=23-29)]

Il nuovo codice rilascia le chiamate al metodo *ThenInclude* per i dati delle iscrizioni dal codice che recupera entità Instructor. Elimina anche `AsNoTracking`.  Se sono selezionati un insegnante e un corso, il codice evidenziato recupera le entità Enrollment per il corso selezionato e le entità Student per ogni Enrollment.

Eseguire l'app e passare alla pagina di indice degli insegnanti. Non si noterà alcuna differenza in ciò che viene visualizzato nella pagina, anche se è stata modificata la modalità di recupero dei dati.

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione:

> [!div class="checklist"]
> * Come caricare i dati correlati
> * Creazione di una pagina Courses
> * Creazione di una pagina Instructors
> * Raccolta di informazioni sul caricamento esplicito

Passare all'esercitazione successiva per informazioni su come aggiornare i dati correlati.

> [!div class="nextstepaction"]
> [Aggiornare dati correlati](update-related-data.md)
