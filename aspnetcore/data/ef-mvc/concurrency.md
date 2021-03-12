---
title: 'Esercitazione: gestire la concorrenza ASP.NET MVC con EF Core'
description: Questa esercitazione descrive la gestione dei conflitti quando più utenti aggiornano la stessa entità contemporaneamente.
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
uid: data/ef-mvc/concurrency
ms.openlocfilehash: cc93069899953d04d1df4e79a282c349a163b06e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586774"
---
# <a name="tutorial-handle-concurrency---aspnet-mvc-with-ef-core"></a>Esercitazione: gestire la concorrenza ASP.NET MVC con EF Core

Nelle esercitazioni precedenti è stato descritto come aggiornare i dati. Questa esercitazione descrive la gestione dei conflitti quando più utenti aggiornano la stessa entità contemporaneamente.

Si creano pagine Web che funzionano con l'entità Department (Reparto) e si gestiscono gli errori di concorrenza. Le illustrazioni seguenti visualizzano le pagine Edit (Modifica) e Delete (Elimina) e includono alcuni messaggi che vengono visualizzati se si verifica un conflitto di concorrenza.

![Pagina Department Edit (Modifica - Reparto)](concurrency/_static/edit-error.png)

![Pagina Department Delete (Elimina - Reparto)](concurrency/_static/delete-error.png)

In questa esercitazione:

> [!div class="checklist"]
> * Scoprire di più sui conflitti di concorrenza
> * Aggiungere una proprietà di rilevamento modifiche
> * Creare un controller e visualizzazioni Departments
> * Aggiornare la visualizzazione Index
> * Aggiornare i metodi Edit
> * Aggiornare la visualizzazione Edit
> * Testare i conflitti di concorrenza
> * Aggiornare la pagina Delete (Elimina)
> * Aggiornare le visualizzazioni Details (Dettagli) e Create (Crea)

## <a name="prerequisites"></a>Prerequisiti

* [Aggiornare dati correlati](update-related-data.md)

## <a name="concurrency-conflicts"></a>Conflitti di concorrenza

Un conflitto di concorrenza si verifica quando un utente visualizza i dati di un'entità per modificarli mentre un altro utente aggiorna i dati della stessa entità prima che la modifica del primo utente venga scritta nel database. Se non si abilita il rilevamento di questi conflitti, l'ultimo utente che aggiorna il database sovrascrive le modifiche apportate dall'altro utente. In molte applicazioni questo rischio è accettabile: se il numero di utenti è ridotto o se l'eventuale sovrascrittura di alcune modifiche non è un aspetto critico, i costi della programmazione per la concorrenza possono superare i vantaggi. In tal caso non è necessario configurare l'applicazione per la gestione dei conflitti di concorrenza.

### <a name="pessimistic-concurrency-locking"></a>Concorrenza pessimistica (blocco)

Se è importante che l'applicazione eviti la perdita accidentale di dati in scenari di concorrenza, un metodo per garantire che ciò accada è l'uso dei blocchi di database. Questo approccio è denominato concorrenza pessimistica. Ad esempio, prima di leggere una riga da un database si richiede un blocco per l'accesso di sola lettura o per l'accesso in modalità aggiornamento. Se si blocca una riga per l'accesso di aggiornamento, nessun altro utente può bloccare la riga per l'accesso di sola lettura o di aggiornamento, perché riceverebbe una copia di dati dei quali è in corso la modifica. Se si blocca una riga per l'accesso in sola lettura, anche altri utenti possono bloccarla per l'accesso in sola lettura, ma non per l'aggiornamento.

La gestione dei blocchi presenta svantaggi. La sua programmazione può risultare complicata. Richiede molte risorse di gestione del database e può causare problemi di prestazioni quando il numero di utenti di un'applicazione aumenta. Per questi motivi non tutti i sistemi di gestione di database supportano la concorrenza pessimistica. Entity Framework Core non offre supporto predefinito per questa modalità e la presente esercitazione non indica come implementarla.

### <a name="optimistic-concurrency"></a>Concorrenza ottimistica

L'alternativa alla concorrenza pessimistica è la concorrenza ottimistica. Nella concorrenza ottimistica si consente che i conflitti di concorrenza si verifichino, quindi si reagisce con le modalità appropriate. Ad esempio Jane visita la pagina Department Edit (Modifica - Reparto) e cambia l'importo di Budget per il reparto English (Inglese) da $ 350.000,00 a $ 0,00.

![Impostazione del budget su 0](concurrency/_static/change-budget.png)

Prima che Jane faccia clic su **Salva** John visita la stessa pagina e cambia il valore del campo Start Date (Data inizio) da 9/1/2007 a 9/1/2013.

![Impostazione della data di inizio su 2013](concurrency/_static/change-date.png)

Jane fa clic su **Salva** per prima e vede la sua modifica quando il browser torna alla pagina di indice.

![Budget impostato su zero](concurrency/_static/budget-zero.png)

Quindi John fa clic su **Salva** in una pagina Edit (Modifica) che visualizza ancora un budget pari a $ 350.000,00. Le operazioni successive dipendono da come si decide di gestire i conflitti di concorrenza.

Di seguito sono elencate alcune opzioni:

* È possibile tenere traccia della proprietà che un utente ha modificato e aggiornare solo le colonne corrispondenti nel database.

     Nello scenario dell'esempio non si perde nessun dato, perché i due utenti hanno aggiornato proprietà diverse. Quando un utente torna a visualizzare il reparto English (Inglese), visualizza sia le modifiche di Jane sia quelle di John: la data di inizio 9/1/2013 e un budget di zero dollari. Questo metodo di aggiornamento può ridurre il numero di conflitti che causano la perdita di dati, ma non può evitare la perdita di dati se vengono apportate modifiche in competizione tra loro alla stessa proprietà di un'entità. Questo funzionamento di Entity Framework dipende dalla modalità di implementazione del codice di aggiornamento. In molti casi in un'app Web questo approccio risulta poco pratico, perché richiede la gestione di grandi quantità di codice statico per tenere traccia di tutti i valori di proprietà originali per un'entità, nonché dei nuovi valori. La gestione di grandi quantità di stato può influire sulle prestazioni dell'applicazione perché richiede risorse del server o deve essere inclusa nella pagina Web, ad esempio in campi nascosti, o in un oggetto cookie .

* È possibile consentire che la modifica di John sovrascriva la modifica di Jane.

     Quando un utente torna a visualizzare il reparto English (Inglese), visualizza 9/1/2013 e il valore $ 350.000,00 ripristinato. Questo scenario è detto *Priorità client* o *Last in Wins* (Priorità ultimo accesso). Tutti i valori del client hanno la precedenza sugli elementi presenti nell'archivio dati. Come indicato nell'introduzione a questa sezione, se non si esegue alcuna codifica per la gestione della concorrenza, questa operazione viene eseguita automaticamente.

* È possibile impedire l'aggiornamento del database con la modifica di John.

     In genere viene visualizzato un messaggio di errore e lo stato corrente dei dati e si consente all'utente di riapplicare le modifiche se lo desidera. Questo scenario è detto *Store Wins* (Priorità archivio). I valori dell'archivio dati hanno la precedenza sui valori inviati dal client. Verrà implementato lo scenario Store WINS in questa esercitazione. Questo metodo garantisce che nessuna modifica venga sovrascritta senza che un utente riceva un avviso.

### <a name="detecting-concurrency-conflicts"></a>Rilevamento dei conflitti di concorrenza

È possibile risolvere i conflitti gestendo le eccezioni `DbConcurrencyException` generate da Entity Framework. Per determinare quando generare queste eccezioni, Entity Framework deve essere in grado di rilevare i conflitti. Pertanto è necessario configurare il database e il modello di dati in modo appropriato. Di seguito sono elencate alcune opzioni per abilitare il rilevamento dei conflitti:

* Nella tabella del database, includere una colonna di rilevamento che può essere usata per determinare quando è stata modificata una riga. Quindi è possibile configurare Entity Framework per includere tale colonna nella clausola Where dei comandi SQL Update o Delete.

     Il tipo di dati della colonna di rilevamento è in genere `rowversion`. Il valore `rowversion` è un numero sequenziale che viene incrementato ogni volta che la riga viene aggiornata. In un comando Update o Delete, la clausola Where include il valore originale della colonna di rilevamento (la versione originale della riga). Se la riga in corso di aggiornamento è stata modificata da un altro utente, il valore della colonna `rowversion` è diverso dal valore originale, pertanto l'istruzione Update o Delete non trova la riga da aggiornare a causa della clausola Where. Quando Entity Framework rileva che nessuna riga è stata aggiornata dal comando Update o Delete (ovvero quando il numero di righe interessate è pari a zero), interpreta questo fatto come un conflitto di concorrenza.

* Configurare Entity Framework in modo che includa i valori originali di ogni colonna della tabella nella clausola Where dei comandi Update e Delete.

     Come nella prima opzione, se è stata apportata qualsiasi modifica dopo la lettura iniziale della riga, la clausola Where non restituisce una riga a Update ed Entity Framework interpreta questo fatto come un conflitto di concorrenza. Per le tabelle di database con molte colonne, questo approccio può risultare in clausole Where molto grandi e richiedere grandi quantità di stato. Come notato in precedenza, la gestione di grandi quantità di codice statico può ridurre le prestazioni dell'applicazione. Pertanto questo approccio è in genere sconsigliato e non è il metodo usato in questa esercitazione.

     Se si vuole comunque implementare questo approccio alla concorrenza, è necessario contrassegnare con l'aggiunta dell'attributo `ConcurrencyCheck` tutte le proprietà dell'entità che non sono chiavi primarie e per le quali si vuole tenere traccia della concorrenza. Grazie a questa modifica, Entity Framework include tutte le colonne nella clausola SQL Where delle istruzioni Update e Delete.

Nella parte restante di questa esercitazione si aggiunge una proprietà di rilevamento `rowversion` all'entità Department (Reparto), si crea un controller e delle visualizzazioni e si esegue un test per verificare che tutto funzioni correttamente.

## <a name="add-a-tracking-property"></a>Aggiungere una proprietà di rilevamento modifiche

In *Models/Department.cs* aggiungere una proprietà di rilevamento denominata RowVersion:

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Final&highlight=26,27)]

L'attributo `Timestamp` specifica che questa colonna viene inclusa nella clausola Where dei comandi Update e Delete inviati al database. L'attributo viene chiamato `Timestamp` perché le versioni precedenti di SQL Server usavano un tipo di dati SQL `timestamp` prima che questo fosse sostituito dalla notazione SQL `rowversion`. Il tipo .NET per `rowversion` è una matrice di byte.

Se si preferisce usare l'API Fluent, è possibile usare il metodo `IsConcurrencyToken` (in *Data/SchoolContext.cs*) per specificare la proprietà di rilevamento, come illustrato nell'esempio seguente:

```csharp
modelBuilder.Entity<Department>()
    .Property(p => p.RowVersion).IsConcurrencyToken();
```

In seguito all'aggiunta di una proprietà il modello di database è stato modificato, pertanto è necessario eseguire una nuova migrazione.

Salvare le modifiche e compilare il progetto, quindi immettere i comandi seguenti nella finestra di comando:

```dotnetcli
dotnet ef migrations add RowVersion
```

```dotnetcli
dotnet ef database update
```

## <a name="create-departments-controller-and-views"></a>Creare un controller e visualizzazioni Departments

Eseguire lo scaffolding di un controller e di visualizzazioni Departments come già fatto in precedenza per Students, Courses e Instructors.

![Scaffolding di Department](concurrency/_static/add-departments-controller.png)

Nel file *DepartmentsController.cs*, convertire tutte e quattro le ricorrenze di "FirstMidName" in "FullName" in modo che gli elenchi a discesa degli amministratori di reparto contengano il nome completo dell'insegnante anziché solo il cognome.

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_Dropdown)]

## <a name="update-index-view"></a>Aggiornare la visualizzazione Index

Il motore di scaffolding crea una colonna RowVersion nella vista Index, ma questo campo non deve essere visualizzato.

Sostituire il codice in *Views/Departments/Index.cshtml* con il codice seguente:

[!code-cshtml[](intro/samples/cu/Views/Departments/Index.cshtml?highlight=4,7,44)]

Questo codice imposta come intestazione "Departments" (Reparti), elimina la colonna RowVersion e visualizza il nome completo anziché solo il cognome dell'amministratore.

## <a name="update-edit-methods"></a>Aggiornare i metodi Edit

Sia nel metodo HttpGet `Edit` che nel metodo `Details`, aggiungere `AsNoTracking`. Nel metodo HttpGet `Edit` aggiungere il caricamento eager per Administrator.

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EagerLoading)]

Sostituire il codice esistente del metodo HttpPost `Edit` con il codice seguente.

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EditPost)]

Il codice inizia con la lettura del reparto da aggiornare. Se il metodo `FirstOrDefaultAsync` restituisce null, il reparto è stato eliminato da un altro utente. In questo caso il codice usa i valori del modulo registrato per creare un'entità reparto, in modo che la pagina di modifica possa essere visualizzata nuovamente con un messaggio di errore. In alternativa, non è necessario creare nuovamente l'entità del reparto se si visualizza solo un messaggio di errore senza visualizzare di nuovo i campi del reparto.

La visualizzazione archivia il valore `RowVersion` originale in un campo nascosto e questo metodo riceve il valore nel parametro `rowVersion`. Prima della chiamata di `SaveChanges` è necessario inserire il valore originale della proprietà `RowVersion` nella raccolta `OriginalValues` dell'entità.

```csharp
_context.Entry(departmentToUpdate).Property("RowVersion").OriginalValue = rowVersion;
```

Quando in seguito Entity Framework crea un comando SQL UPDATE, il comando includerà una clausola WHERE che cerca una riga con il valore `RowVersion` originale. Se nessuna riga è interessata dal comando UPDATE (nessuna riga presenta il valore `RowVersion` originale), Entity Framework genera un'eccezione `DbUpdateConcurrencyException`.

Il codice del blocco catch di tale eccezione ottiene l'entità Department interessata con i valori aggiornati dalla proprietà `Entries` nell'oggetto eccezione.

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=164)]

La raccolta `Entries` ha un solo oggetto `EntityEntry`.  È possibile usare tale oggetto per ottenere i nuovi valori immessi dall'utente e i valori del database corrente.

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=165-166)]

Il codice aggiunge un messaggio di errore personalizzato per ogni colonna che ha valori di database diversi da ciò che l'utente ha immesso nella pagina Edit (per brevità qui viene riportato un solo campo).

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=174-178)]

Infine il codice imposta il valore `RowVersion` di `departmentToUpdate` sul nuovo valore recuperato dal database. Questo nuovo valore `RowVersion` viene archiviato nel campo nascosto quando viene visualizzata nuovamente la pagina Edit (Modifica). Quando l'utente torna a fare clic su **Salva** vengono rilevati solo gli errori di concorrenza che si verificano dopo la nuova visualizzazione della pagina Edit (Modifica).

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=199-200)]

L'istruzione `ModelState.Remove` è necessaria perché `ModelState` presenta il valore obsoleto `RowVersion`. Nella visualizzazione, il valore `ModelState` di un campo ha la precedenza sui valori di proprietà del modello quando entrambi gli elementi sono presenti.

## <a name="update-edit-view"></a>Aggiornare la visualizzazione Edit

In *Views/Departments/Edit.cshtml* apportare le modifiche seguenti:

* Aggiungere un campo nascosto per salvare il valore della proprietà `RowVersion` subito dopo il campo nascosto della proprietà `DepartmentID`.

* Aggiungere un'opzione "Select Administrator" (Seleziona amministratore) all'elenco a discesa.

[!code-cshtml[](intro/samples/cu/Views/Departments/Edit.cshtml?highlight=16,34-36)]

## <a name="test-concurrency-conflicts"></a>Testare i conflitti di concorrenza

Eseguire l'app e passare alla pagina Departments Index (Indice reparti). Fare clic con il pulsante destro del mouse sul collegamento ipertestuale **Edit** (Modifica) per il reparto English (Inglese) e selezionare **Apri link in nuova scheda**, quindi fare clic sul collegamento ipertestuale **Edit** (Modifica) per il reparto English (Inglese). Le due schede del browser ora visualizzano le stesse informazioni.

Modificare un campo nella prima scheda del browser e fare clic su **Salva**.

![Pagina Department Edit (Modifica - Reparto) 1 dopo la modifica](concurrency/_static/edit-after-change-1.png)

Il browser visualizza la pagina Index con il valore modificato.

Modificare un campo nella seconda scheda del browser.

![Pagina Department Edit (Modifica - Reparto) 2 dopo la modifica](concurrency/_static/edit-after-change-2.png)

Fare clic su **Salva**. Viene visualizzato un messaggio di errore:

![Messaggio di errore della pagina Department Edit (Modifica - Reparto)](concurrency/_static/edit-error.png)

Fare clic su **Salva**. Il valore immesso nella seconda scheda del browser viene salvato. I valori salvati vengono visualizzati nella pagina Index.

## <a name="update-the-delete-page"></a>Aggiornare la pagina Delete (Elimina)

Per la pagina Delete (Elimina), Entity Framework rileva conflitti di concorrenza causati da un altro utente che ha modificato il reparto con modalità simili. Quando il metodo HttpGet `Delete` visualizza la conferma, la visualizzazione include il valore `RowVersion` originale in un campo nascosto. Questo valore viene quindi reso disponibile al metodo HttpPost `Delete` che viene chiamato quando l'utente conferma l'eliminazione. Quando Entity Framework crea il comando SQL DELETE, include una clausola WHERE con il valore `RowVersion` originale. Se il comando non ha effetto su nessuna riga (ovvero se la riga è stata modificata dopo la visualizzazione della pagina di conferma Delete) viene attivata un'eccezione di concorrenza e viene chiamato il metodo HttpGet `Delete` con un flag di errore impostato su true, per tornare a visualizzare la pagina di conferma con un messaggio di errore. È anche possibile che il comando non abbia effetto su nessuna riga perché la riga è stata eliminata da un altro utente. In questo caso non viene visualizzato nessun messaggio di errore.

### <a name="update-the-delete-methods-in-the-departments-controller"></a>Aggiornare i metodi Delete nel controller Departments

In *DepartmentsController.cs*, sostituire il metodo HttpGet `Delete` con il codice seguente:

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeleteGet&highlight=1,10,14-17,21-29)]

Il metodo accetta un parametro facoltativo che indica se la pagina viene nuovamente visualizzata dopo un errore di concorrenza. Se questo flag è true e il reparto specificato non esiste più significa che è stato eliminato da un altro utente. In questo caso, il codice esegue il reindirizzamento alla pagina Index.  Se questo flag è true e il reparto esiste, è stato modificato da un altro utente. In questo caso il codice invia un messaggio di errore alla vista usando `ViewData`.

Sostituire il codice nel metodo HttpPost `Delete` (denominato `DeleteConfirmed`) con il codice seguente.

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeletePost&highlight=1,3,5-8,11-18)]

Nel codice sottoposto a scaffolding appena sostituito, questo metodo accettava solo un ID record:

```csharp
public async Task<IActionResult> DeleteConfirmed(int id)
```

Questo parametro è stato convertito in un'istanza di entità Department creata dallo strumento di associazione di modelli. Ciò consente a EF di accedere al valore della proprietà RowVersion oltre che alla chiave del record.

```csharp
public async Task<IActionResult> Delete(Department department)
```

Anche il nome del metodo di azione è stato modificato da `DeleteConfirmed` a `Delete`. Il codice sottoposto a scaffolding usava il nome `DeleteConfirmed` per offrire al metodo HttpPost una firma unica. (CLR richiede che i metodi di overload abbiano parametri di metodo diversi). Ora che le firme sono univoche, è possibile attenersi alla convenzione MVC e usare lo stesso nome per i metodi di eliminazione HttpPost e HttpGet.

Se il reparto è già stato eliminato, il metodo `AnyAsync` restituisce false e l'applicazione torna al metodo Index.

Se viene rilevato un errore di concorrenza, il codice visualizza nuovamente la pagina di conferma Delete (Elimina) e visualizza un flag indicante che è necessario visualizzare un messaggio di errore di concorrenza.

### <a name="update-the-delete-view"></a>Aggiornare la pagina Delete

In *Views/Departments/Delete.cshtml*, sostituire il codice sottoposto a scaffolding con il codice seguente, che aggiunge un campo messaggio di errore e campi nascosti per le proprietà DepartmentID e RowVersion. Le modifiche sono evidenziate.

[!code-cshtml[](intro/samples/cu/Views/Departments/Delete.cshtml?highlight=9,38,44,45,48)]

Questa impostazione determina le modifiche seguenti:

* Aggiunge un messaggio di errore tra le intestazioni `h2` e `h3`.

* Sostituisce FirstMidName con FullName nel campo **Administrator** (Amministratore).

* Rimuove il campo RowVersion.

* Aggiunge un campo nascosto per la proprietà `RowVersion`.

Eseguire l'app e passare alla pagina Departments Index (Indice reparti). Fare clic con il pulsante destro del mouse sul collegamento ipertestuale **Edit** (Modifica) per il reparto English (Inglese) e selezionare **Apri link in nuova scheda**, quindi fare clic sul collegamento ipertestuale **Edit** per il reparto English.

Nella prima finestra modificare uno dei valori e fare clic su **Salva**:

![Pagina Department Edit (Modifica - Reparto) dopo la modifica e prima dell'eliminazione](concurrency/_static/edit-after-change-for-delete.png)

Nella seconda scheda fare clic su **Delete** (Elimina). Viene visualizzato il messaggio di errore di concorrenza e i valori di Department (Reparto) vengono aggiornati con i dati attualmente presenti nel database.

![Pagina di conferma Department Delete (Elimina - Reparto) con errore di concorrenza](concurrency/_static/delete-error.png)

Se si fa di nuovo clic su **Delete** (Elimina) viene visualizzata la pagina Index che indica che il reparto è stato eliminato.

## <a name="update-details-and-create-views"></a>Aggiornare le visualizzazioni Details (Dettagli) e Create (Crea)

Facoltativamente è possibile pulire il codice di scaffolding nelle visualizzazioni Details (Dettagli) e Create (Crea).

Sostituire il codice in *Views/Departments/Details.cshtml* per eliminare la colonna RowVersion e visualizzare il nome completo di Administrator (Amministratore).

[!code-cshtml[](intro/samples/cu/Views/Departments/Details.cshtml?highlight=35)]

Sostituire il codice in *Views/Departments/Create.cshtml* per aggiungere all'elenco a discesa un'opzione Select (Seleziona).

[!code-cshtml[](intro/samples/cu/Views/Departments/Create.cshtml?highlight=32-34)]

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a>Risorse aggiuntive

 Per altre informazioni su come gestire i conflitti di concorrenza in EF Core, vedere [Conflitti di concorrenza](/ef/core/saving/concurrency).

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione:

> [!div class="checklist"]
> * Scoprire di più sui conflitti di concorrenza
> * Aggiungere una proprietà di rilevamento modifiche
> * Creare un controller e visualizzazioni Departments
> * Aggiornare la visualizzazione Index
> * Aggiornare i metodi Edit
> * Aggiornare la visualizzazione Edit
> * Testare i conflitti di concorrenza
> * Aggiornamento della pagina Delete
> * Aggiornare le visualizzazioni Details e Create

Passare all'esercitazione successiva per apprendere come implementare l'ereditarietà tabella per gerarchia per le entità Instructor e Student.

> [!div class="nextstepaction"]
> [Passaggio successivo: implementare l'ereditarietà tabella per gerarchia](inheritance.md)
