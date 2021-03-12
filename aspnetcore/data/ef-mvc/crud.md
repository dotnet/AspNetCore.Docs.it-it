---
title: 'Esercitazione: implementare la funzionalità CRUD-ASP.NET MVC con EF Core'
description: In questa esercitazione verrà esaminato e personalizzato il codice CRUD (Create, Read, Update, Delete) che lo scaffolding di MVC crea automaticamente nei controller e nelle visualizzazioni.
author: rick-anderson
ms.author: riande
ms.custom: mvc
ms.date: 02/04/2019
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
uid: data/ef-mvc/crud
ms.openlocfilehash: c3cfee107541a20a089809193d4bf7fa08730bd2
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586397"
---
# <a name="tutorial-implement-crud-functionality---aspnet-mvc-with-ef-core"></a>Esercitazione: implementare la funzionalità CRUD-ASP.NET MVC con EF Core

Nell'esercitazione precedente è stata creata un'applicazione MVC che memorizza e visualizza i dati usando Entity Framework e SQL Server LocalDB. In questa esercitazione verrà esaminato e personalizzato il codice CRUD (Create, Read, Update, Delete) che lo scaffolding di MVC crea automaticamente nei controller e nelle visualizzazioni.

> [!NOTE]
> È pratica comune implementare il modello di repository per creare un livello di astrazione tra il controller e il livello di accesso ai dati. Per mantenere le esercitazioni semplici e incentrate sulla descrizione dell'uso di Entity Framework, non vengono usati i repository. Per informazioni sui repository con EF, vedere l'[ultima esercitazione della serie](advanced.md).

In questa esercitazione:

> [!div class="checklist"]
> * Personalizzare la pagina Details
> * Aggiornare la pagina Create
> * Aggiornare la pagina Edit (Modifica)
> * Aggiornare la pagina Delete (Elimina)
> * Chiudere le connessioni di database

## <a name="prerequisites"></a>Prerequisiti

* [Introduzione a EF Core e ASP.NET Core MVC](intro.md)

## <a name="customize-the-details-page"></a>Personalizzare la pagina Details

Il codice con scaffolding della pagina Students Index ha escluso la proprietà `Enrollments` poiché la proprietà contiene una raccolta. Nella pagina **Details** verranno visualizzati i contenuti della raccolta in una tabella HTML.

In *Controllers/StudentsController.cs* il metodo di azione per la visualizzazione Details usa il metodo `SingleOrDefaultAsync` per recuperare una singola entità `Student`. Aggiungere un codice che chiama i metodi `Include`, `ThenInclude` e `AsNoTracking`, come illustrato nel codice evidenziato seguente.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Details&highlight=8-12)]

I metodi `Include` e `ThenInclude` fanno in modo che il contesto carichi la proprietà di navigazione `Student.Enrollments` e la proprietà di navigazione `Enrollment.Course` all'interno di ogni registrazione.  Per altre informazioni su questi metodi, vedere l'esercitazione [Leggere dati correlati](read-related-data.md).

Il metodo `AsNoTracking` migliora le prestazioni negli scenari in cui le entità restituite non vengono aggiornate nel contesto corrente. Altre informazioni su `AsNoTracking` sono disponibili alla fine di questa esercitazione.

### <a name="route-data"></a>Indirizzare i dati

Il valore della chiave passato al metodo `Details` deriva dai *dati della route*. I dati della route sono i dati trovati dallo strumento di associazione di modelli in un segmento dell'URL. Ad esempio, la route predefinita specifica i segmenti del controller, dell'azione e dell'ID:

[!code-csharp[](intro/samples/cu/Startup.cs?name=snippet_Route&highlight=5)]

Nell'URL seguente la route predefinita mappa Instructor come controller, Index come azione e 1 come ID: questi sono i valori dei dati della route.

```
http://localhost:1230/Instructor/Index/1?courseID=2021
```

L'ultima parte dell'URL ("?courseID=2021") è un valore di stringa di query. Lo strumento di associazione di modelli passa anche il valore ID al parametro `id` del metodo `Index` se viene passato come un valore di stringa di query:

```
http://localhost:1230/Instructor/Index?id=1&CourseID=2021
```

Nella pagina index gli URL dei collegamenti ipertestuali vengono creati dalle istruzioni di helper tag nella Razor visualizzazione. Nel codice seguente Razor il `id` parametro corrisponde alla route predefinita, quindi `id` viene aggiunto ai dati della route.

```html
<a asp-action="Edit" asp-route-id="@item.ID">Edit</a>
```

Quando `item.ID` è 6, viene generato il codice HTML seguente:

```html
<a href="/Students/Edit/6">Edit</a>
```

Nel codice seguente Razor , `studentID` non corrisponde a un parametro nella route predefinita, quindi viene aggiunto come stringa di query.

```html
<a asp-action="Edit" asp-route-studentID="@item.ID">Edit</a>
```

Quando `item.ID` è 6, viene generato il codice HTML seguente:

```html
<a href="/Students/Edit?studentID=6">Edit</a>
```

Per altre informazioni sugli helper tag, vedere <xref:mvc/views/tag-helpers/intro>.

### <a name="add-enrollments-to-the-details-view"></a>Aggiungere le registrazioni alla visualizzazione Details

Aprire *Views/Students/Details.cshtml*. Ogni campo viene visualizzato usando gli helper `DisplayNameFor` e `DisplayFor`, come illustrato nell'esempio seguente:

[!code-cshtml[](intro/samples/cu/Views/Students/Details.cshtml?range=13-18&highlight=2,5)]

Dopo l'ultimo campo e immediatamente prima del tag `</dl>` di chiusura, aggiungere il codice seguente per visualizzare un elenco delle registrazioni:

[!code-cshtml[](intro/samples/cu/Views/Students/Details.cshtml?range=31-52)]

Se dopo aver incollato il codice il rientro è errato, premere CTRL-K-D per correggerlo.

Il codice esegue il ciclo nelle entità nella proprietà di navigazione `Enrollments`. Per ogni registrazione, il codice visualizza il titolo del corso e il voto. Il titolo del corso viene recuperato dall'entità Course memorizzata nella proprietà di navigazione `Course` dell'entità Enrollments.

Eseguire l'app, selezionare la scheda **Students** e fare clic sul collegamento **Details** relativo a uno studente. Viene visualizzato l'elenco dei corsi e dei voti dello studente selezionato:

![Pagina Details (Student)](crud/_static/student-details.png)

## <a name="update-the-create-page"></a>Aggiornare la pagina Create

In *StudentsController.cs* modificare il metodo HttpPost `Create` aggiungendo un blocco Try-Catch e rimuovendo l'ID dall'attributo `Bind`.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Create&highlight=4,6-7,14-21)]

Questo codice aggiunge l'entità Student creata dallo strumento di associazione di modelli di ASP.NET Core MVC al set di entità Students e salva le modifiche nel database. Lo strumento di associazione di modelli corrisponde alla funzionalità di ASP.NET Core MVC che rende più semplice l'uso di dati inviati da un modulo; lo strumento di associazione di modelli converte i valori di modulo inviati in tipi CLR e li passa al metodo di azione nei parametri. In questo caso lo strumento di associazione di modelli crea automaticamente un'istanza di un'entità Student usando i valori di proprietà della raccolta Form).

`ID` è stato rimosso dall'attributo `Bind` poiché l'ID è il valore di chiave primaria che SQL Server imposta automaticamente quando viene inserita la riga. L'input dell'utente non imposta il valore ID.

A differenza dell'attributo `Bind`, il blocco Try-Catch rappresenta l'unica modifica apportata al codice con scaffolding. Se viene rilevata un'eccezione che deriva da `DbUpdateException` durante il salvataggio delle modifiche, viene visualizzato un messaggio di errore generico. Poiché le eccezioni `DbUpdateException` sono a volte causate da elementi esterni all'applicazione e non da un errore di programmazione, viene consigliato all'utente di riprovare. Sebbene non sia implementata in questo esempio, un'applicazione di controllo della qualità di produzione potrebbe registrare l'eccezione. Per altre informazioni, vedere la sezione **Log for insight** (Registrare informazioni dettagliate) in [Monitoring and Telemetry (Building Real-World Cloud Apps with Azure)](/aspnet/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry) (Monitoraggio e telemetria (creazione di app cloud realistiche con Azure)).

L'attributo `ValidateAntiForgeryToken` è utile per prevenire attacchi tramite richieste intersito false (CSRF). Il token viene inserito automaticamente nella visualizzazione da [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) e viene incluso quando il modulo viene inviato dall'utente. Il token è convalidato dall'attributo `ValidateAntiForgeryToken`. Per altre informazioni, vedere <xref:security/anti-request-forgery>.

<a id="overpost"></a>

### <a name="security-note-about-overposting"></a>Nota sulla sicurezza relativa all'overposting

L'attributo `Bind` che il codice con scaffolding include nel metodo `Create` consente di impedire l'overposting negli scenari di creazione. Si supponga ad esempio che l'entità Student includa una proprietà `Secret` che la pagina Web non deve impostare.

```csharp
public class Student
{
    public int ID { get; set; }
    public string LastName { get; set; }
    public string FirstMidName { get; set; }
    public DateTime EnrollmentDate { get; set; }
    public string Secret { get; set; }
}
```

Anche se non è presente un campo `Secret` nella pagina Web, un hacker potrebbe usare uno strumento come Fiddler oppure scrivere codice JavaScript per inviare un valore di modulo `Secret`. Senza l'attributo `Bind` che limita i campi usati dallo strumento di associazione di modelli durante la creazione di un'istanza di Student, lo strumento individuerebbe il valore di modulo `Secret` e lo userebbe per creare l'istanza dell'entità Student. Di conseguenza, qualsiasi valore specificato dall'hacker per il campo di modulo `Secret` verrebbe aggiornato nel database. L'immagine seguente illustra lo strumento Fiddler che aggiunge il campo `Secret` (con il valore "OverPost") ai valori di modulo inviati.

![Fiddler aggiunge il campo Secret](crud/_static/fiddler.png)

Il valore "OverPost" verrebbe quindi aggiunto alla proprietà `Secret` della riga inserita, sebbene non sia prevista in alcun modo l'impostazione della proprietà da parte della pagina Web.

È possibile impedire l'overposting negli scenari di modifica leggendo prima l'entità dal database e quindi chiamando `TryUpdateModel`, passando un elenco delle proprietà consentite esplicito. Questo metodo viene usato in queste esercitazioni.

In alternativa, per impedire l'overposting numerosi sviluppatori usano i modelli di visualizzazione anziché le classi di entità con l'associazione di modelli. Includere solo le proprietà da aggiornare nel modello di visualizzazione. Al termine delle operazioni eseguite dallo strumento di associazione di modelli di MVC, copiare le proprietà del modello di visualizzazione nell'istanza dell'entità usando facoltativamente uno strumento come AutoMapper. Usare `_context.Entry` nell'istanza dell'entità per impostarne lo stato su `Unchanged` e quindi impostare `Property("PropertyName").IsModified` su true in ogni proprietà dell'entità inclusa nel modello di visualizzazione. Questo metodo funziona negli scenari di modifica e negli scenari di creazione.

### <a name="test-the-create-page"></a>Testare la pagina Create

Il codice in *views/students/create. cshtml* USA `label` , `input` e `span` (per i messaggi di convalida) gli helper tag per ogni campo.

Eseguire l'app, selezionare la scheda **Students** e fare clic su **Crea nuovo**.

Immettere i nomi e una data. Provare a immettere una data non valida se il browser lo consente. Alcuni browser forzano l'uso di una selezione data. Fare quindi clic su **Crea** per visualizzare il messaggio di errore.

![Errore di convalida della data](crud/_static/date-error.png)

Questa è la convalida lato server che si ottiene per impostazione predefinita; in un'esercitazione successiva viene descritto come aggiungere gli attributi che generano codice anche per la convalida lato client. Il codice evidenziato seguente illustra il controllo di convalida del modello nel metodo `Create`.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Create&highlight=8)]

Modificare la data impostando un valore valido e fare clic su **Crea** per visualizzare il nuovo studente nella pagina **Index**.

## <a name="update-the-edit-page"></a>Aggiornare la pagina Edit (Modifica)

In *StudentController.cs* il metodo HttpGet `Edit`, ovvero il metodo senza l'attributo `HttpPost`, usa il metodo `SingleOrDefaultAsync` per recuperare l'entità Student selezionata, come nel metodo `Details`. Non è necessario modificare questo metodo.

### <a name="recommended-httppost-edit-code-read-and-update"></a>Codice di modifica HttpPost consigliato: lettura e aggiornamento

Sostituire il metodo di azione HttpPost Edit con il codice seguente.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ReadFirst)]

Queste modifiche implementano una procedura di sicurezza consigliata per impedire l'overposting. Lo scaffolder ha generato un attributo `Bind` e aggiunto l'entità creata dallo strumento di associazione di modelli all'entità impostata con un flag `Modified`. L'uso di un codice simile non è consigliabile in molti scenari poiché l'attributo `Bind` cancella tutti i dati esistenti nei campi non elencati nel parametro `Include`.

Il nuovo codice legge l'entità esistente e chiama `TryUpdateModel` per aggiornare i campi nell'entità recuperata [in base all'input dell'utente nei dati del modulo inviati](xref:mvc/models/model-binding). Il rilevamento modifiche automatico di Entity Framework imposta il flag `Modified` nei campi modificati dall'input del modulo. Quando viene chiamato il metodo `SaveChanges`, Entity Framework crea le istruzioni SQL per aggiornare la riga del database. I conflitti di concorrenza vengono ignorati e vengono aggiornate solo le colonne della tabella che sono state aggiornate dall'utente nel database. (Un'esercitazione successiva illustra come gestire i conflitti di concorrenza).

Come procedura consigliata per evitare l'overposting, i campi che si desidera aggiornare dalla pagina di **modifica** vengono dichiarati nei `TryUpdateModel` parametri. La stringa vuota che precede l'elenco dei campi nell'elenco di parametri è relativa a un prefisso da usare con i nomi dei campi del modulo. Attualmente non sono presenti campi aggiuntivi da proteggere, ma elencando i campi che si desidera associare allo strumento di associazione di modelli si garantisce che se si aggiungono i campi al modello di dati in futuro, questi ultimi vengono protetti automaticamente finché non vengono aggiunti in modo esplicito.

In seguito a queste modifiche, la firma del metodo HttpPost `Edit` corrisponde al metodo HttpGet `Edit`. Di conseguenza, il metodo `EditPost` è stato rinominato.

### <a name="alternative-httppost-edit-code-create-and-attach"></a>Codice di modifica HttpPost alternativo: creazione e collegamento

Il codice di modifica HttpPost garantisce che vengano aggiornate soltanto le colonne modificate e mantiene i dati nelle proprietà che non si vuole includere per l'associazione di modelli. Tuttavia, l'approccio con lettura iniziale richiede una lettura del database aggiuntiva e può generare un codice più complesso per la gestione dei conflitti di concorrenza. In alternativa, è possibile collegare un'entità creata dallo strumento di associazione di modelli al contesto EF e contrassegnarla come modificata. (Non aggiornare il progetto con questo codice. Il codice ha il solo scopo di descrivere un approccio facoltativo).

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_CreateAndAttach)]

È possibile usare questo approccio quando l'interfaccia utente della pagina Web include tutti i campi dell'entità e può aggiornare qualsiasi campo.

Sebbene il codice con scaffolding usi l'approccio che prevede la creazione e il collegamento, il codice rileva solo le eccezioni `DbUpdateConcurrencyException` e restituisce i codici di errore 404.  L'esempio illustrato rileva qualsiasi eccezione di aggiornamento del database e visualizza un messaggio di errore.

### <a name="entity-states"></a>Stati di entità

Il contesto del database rileva se le entità in memoria sono sincronizzate con le righe corrispondenti nel database e queste informazioni determinano le operazioni eseguite quando viene chiamato il metodo `SaveChanges`. Ad esempio, quando una nuova entità viene passata al metodo `Add`, lo stato dell'entità viene impostato su `Added`. Quando in seguito viene chiamato il metodo `SaveChanges`, il contesto del database esegue un comando SQL INSERT.

Le entità possono essere in uno dei seguenti stati:

* `Added`. L'entità non esiste ancora nel database. Il metodo `SaveChanges` genera un'istruzione INSERT.

* `Unchanged`. Il metodo `SaveChanges` non deve eseguire alcuna operazione con l'entità. Quando un'entità viene letta dal database, l'entità ha inizialmente questo stato.

* `Modified`. Sono stati modificati alcuni o tutti i valori di proprietà dell'entità. Il metodo `SaveChanges` genera un'istruzione UPDATE.

* `Deleted`. L'entità è stata contrassegnata per l'eliminazione. Il metodo `SaveChanges` genera un'istruzione DELETE.

* `Detached`. L'entità non viene registrata dal contesto del database.

In un'applicazione desktop le modifiche dello stato vengono in genere impostate automaticamente. Viene letta un'entità e vengono apportate modifiche ad alcuni valori delle proprietà. In questo modo lo stato dell'entità viene modificato automaticamente in `Modified`. Quando in seguito viene chiamato `SaveChanges`, Entity Framework genera un'istruzione SQL UPDATE che aggiorna solo le proprietà modificate.

In un'app Web il `DbContext` che inizialmente legge un'entità e ne visualizza i dati da modificare viene eliminato dopo il rendering di una pagina. Quando viene chiamato il metodo di azione HttpPost `Edit`, viene effettuata una nuova richiesta Web con una nuova istanza di `DbContext`. La rilettura dell'entità nel nuovo contesto simula l'elaborazione desktop.

Tuttavia, se non si vuole eseguire un'operazione di lettura aggiuntiva, è necessario usare l'oggetto dell'entità creato dallo strumento di associazione di modelli.  Il modo più semplice per eseguire questa operazione consiste nell'impostare lo stato dell'entità su Modified, come avviene nel codice di modifica HttpPost alternativo illustrato in precedenza. Quando in seguito viene chiamato `SaveChanges`, Entity Framework aggiorna tutte le colonne della riga di database poiché il contesto non ha la possibilità di individuare le proprietà modificate.

Se si vuole evitare l'approccio con lettura iniziale e si vuole anche che l'istruzione SQL UPDATE aggiorni solo i campi modificati dall'utente, il codice è più complesso. È necessario salvare i valori originali, usando ad esempio campi nascosti, in modo che siano disponibili quando viene chiamato il metodo HttpPost `Edit`. Creare quindi un'entità Student usando i valori originali, chiamare il metodo `Attach` con la versione originale dell'entità, aggiornare i valori dell'entità con i nuovi valori e quindi chiamare `SaveChanges`.

### <a name="test-the-edit-page"></a>Testare la pagina Edit

Eseguire l'app, selezionare la scheda **Students** e quindi fare clic su un collegamento ipertestuale **Modifica**.

![Pagina Edit (Students)](crud/_static/student-edit.png)

Modificare alcuni dati e fare clic su **Salva**. Viene visualizzata la pagina **Index** con i dati modificati.

## <a name="update-the-delete-page"></a>Aggiornare la pagina Delete (Elimina)

In *StudentController.cs* il codice di modello per il metodo HttpGet `Delete` usa il metodo `SingleOrDefaultAsync` per recuperare l'entità Student selezionata, come descritto per i metodi Details ed Edit. Tuttavia, per implementare un messaggio di errore personalizzato quando la chiamata di `SaveChanges` ha esito negativo, verrà aggiunta una funzionalità al metodo e alla visualizzazione corrispondente.

Analogamente alle operazioni di aggiornamento e creazione, le operazioni di eliminazione richiedono due metodi di azione. Il metodo chiamato in risposta a una richiesta GET apre una visualizzazione che offre all'utente la possibilità di approvare o annullare l'operazione di eliminazione. Se l'utente approva, viene creata una richiesta POST. In questo caso, viene chiamato il metodo HttpPost `Delete` che esegue l'operazione di eliminazione.

Si aggiungerà un blocco Try-Catch al metodo HttpPost `Delete` per gestire eventuali errori che possono verificarsi quando viene aggiornato il database. Se si verifica un errore, il metodo HttpPost Delete chiama il metodo HttpGet Delete passando un parametro che indica che si è verificato un errore. Il metodo HttpGet Delete visualizza nuovamente la pagina di conferma con il messaggio di errore offrendo all'utente la possibilità di annullare o ripetere l'operazione.

Sostituire il metodo di azione HttpGet `Delete` con il codice seguente che gestisce la segnalazione degli errori.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteGet&highlight=1,9,16-21)]

Questo codice accetta un parametro facoltativo che indica se il metodo è stato chiamato dopo un errore di salvataggio delle modifiche. Il parametro ha valore false quando il metodo HttpGet `Delete` viene chiamato senza essere preceduto da un errore. Quando viene chiamato dal metodo HttpPost `Delete` in risposta a un errore di aggiornamento del database, il parametro ha valore true e viene passato un messaggio di errore alla visualizzazione.

### <a name="the-read-first-approach-to-httppost-delete"></a>Approccio con lettura iniziale per HttpPost Delete

Sostituire il metodo di azione HttpPost `Delete` (denominato `DeleteConfirmed`) con il codice seguente che esegue l'operazione di eliminazione e rileva eventuali errori di aggiornamento del database.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteWithReadFirst&highlight=6-9,11-12,16-21)]

Questo codice recupera l'entità selezionata, quindi chiama il metodo `Remove` per impostare lo stato dell'entità su `Deleted`. Quando viene chiamato `SaveChanges`, viene generato un comando SQL DELETE.

### <a name="the-create-and-attach-approach-to-httppost-delete"></a>Approccio con creazione e collegamento per HttpPost Delete

Se il miglioramento delle prestazioni in un'applicazione a volume elevato è una priorità, è possibile evitare una query SQL non necessaria creando un'istanza di un'entità Student usando solo il valore di chiave primaria e impostando lo stato dell'entità su `Deleted`. Questo è tutto ciò di cui Entity Framework necessita per eliminare l'entità. (Non inserire questo codice nel progetto. Il codice ha il solo scopo di descrivere un approccio alternativo).

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteWithoutReadFirst&highlight=7-8)]

Se l'entità include anche dati correlati che devono essere eliminati, assicurarsi che sia configurata nel database l'eliminazione a catena. Con questo approccio per l'eliminazione di entità, EF potrebbe non rilevare le entità correlate da eliminare.

### <a name="update-the-delete-view"></a>Aggiornare la pagina Delete

In *Views/Student/Delete.cshtml* aggiungere un messaggio di errore tra l'intestazione h2 e l'intestazione h3, come illustrato nell'esempio seguente:

[!code-cshtml[](intro/samples/cu/Views/Students/Delete.cshtml?range=7-9&highlight=2)]

Eseguire l'app, selezionare la scheda **Students** e fare clic sul collegamento ipertestuale **Elimina**:

![Pagina di conferma dell'eliminazione](crud/_static/student-delete.png)

Fare clic su **Elimina**. Viene visualizzata la pagina Index senza lo studente eliminato. (L'esercitazione sulla concorrenza include un esempio di codice per la gestione degli errori).

## <a name="close-database-connections"></a>Chiudere le connessioni di database

Per liberare le risorse di una connessione di database, è necessario eliminare l'istanza del contesto il prima possibile dopo averla usata. L'[inserimento delle dipendenze](../../fundamentals/dependency-injection.md) incorporato di ASP.NET Core esegue automaticamente questa attività.

In *Startup.cs* chiamare il [metodo di estensione AddDbContext](https://github.com/aspnet/EntityFrameworkCore/blob/03bcb5122e3f577a84498545fcf130ba79a3d987/src/Microsoft.EntityFrameworkCore/EntityFrameworkServiceCollectionExtensions.cs) per effettuare il provisioning della classe `DbContext` nel contenitore di inserimento delle dipendenze di ASP.NET Core. Il metodo imposta la durata del servizio su `Scoped` per impostazione predefinita. `Scoped` indica che la durata dell'oggetto del contesto corrisponde alla durata della richiesta Web e il metodo `Dispose` viene chiamato automaticamente alla fine della richiesta Web.

## <a name="handle-transactions"></a>Gestire le transazioni

Per impostazione predefinita Entity Framework implementa in modo implicito le transazioni. Negli scenari in cui vengono apportate modifiche a più righe o tabelle e viene quindi chiamato `SaveChanges`, Entity Framework verifica automaticamente che tutte le modifiche siano state apportate o che tutte le modifiche abbiano avuto esito negativo. Se sono state apportate alcune modifiche e successivamente si verifica un errore, viene automaticamente eseguito il rollback di tali verifiche. Per gli scenari in cui è necessario un maggior controllo, ad esempio per includere le operazioni eseguite all'esterno di Entity Framework in una transazione, vedere [Transazioni](/ef/core/saving/transactions).

## <a name="no-tracking-queries"></a>Query senza registrazione

Quando un contesto di database recupera righe di tabella e crea oggetti entità che le rappresentano, per impostazione predefinita rileva se le entità in memoria sono sincronizzate con ciò che è presente nel database. I dati in memoria svolgono la funzione di una cache e vengono usati per l'aggiornamento di un'entità. Questa memorizzazione nella cache spesso non è necessaria in un'applicazione Web poiché le istanze del contesto hanno spesso una durata breve (viene creata ed eliminata una nuova istanza per ogni richiesta) e il contesto che legge un'entità viene in genere eliminato prima che l'entità venga riutilizzata.

È possibile disabilitare la registrazione degli oggetti entità in memoria chiamando il metodo `AsNoTracking`. Gli scenari tipici in cui viene disabilitata la registrazione includono i seguenti:

* Durante la durata del contesto non è necessario eseguire l'aggiornamento di alcuna entità e non è necessario che EF [carichi automaticamente le proprietà di navigazione con entità recuperate da query separate](read-related-data.md). Queste condizioni si verificano in genere nei metodi di azione HttpGet del controller.

* Si esegue una query che recupera una grande quantità di dati e viene aggiornata solo una piccola quantità dei dati restituiti. Può essere utile disattivare la registrazione per la query ed eseguire successivamente una query per le poche entità che devono essere aggiornate.

* Si vuole collegare un'entità per aggiornarla, ma la stessa entità è stata recuperata in precedenza per uno scopo diverso. Poiché l'entità viene già registrata dal contesto di database, non è possibile collegare l'entità che si vuole modificare. Un modo per gestire questa situazione consiste nel chiamare `AsNoTracking` nella query precedente.

Per altre informazioni, vedere [Tracking vs. no-tracking](/ef/core/querying/tracking).

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione:

> [!div class="checklist"]
> * Personalizzazione della pagina Details
> * Aggiornamento della pagina Create
> * Aggiornamento della pagina Edit
> * Aggiornamento della pagina Delete
> * Chiusura delle connessioni di database

Passare all'esercitazione successiva per informazioni su come estendere la funzionalità della pagina **Index** aggiungendo ordinamento, filtro e suddivisione in pagine.

> [!div class="nextstepaction"]
> [Passaggio successivo: ordinamento, filtro e paging](sort-filter-page.md)
