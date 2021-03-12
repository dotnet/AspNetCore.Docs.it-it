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
# <a name="tutorial-handle-concurrency---aspnet-mvc-with-ef-core"></a><span data-ttu-id="11522-103">Esercitazione: gestire la concorrenza ASP.NET MVC con EF Core</span><span class="sxs-lookup"><span data-stu-id="11522-103">Tutorial: Handle concurrency - ASP.NET MVC with EF Core</span></span>

<span data-ttu-id="11522-104">Nelle esercitazioni precedenti è stato descritto come aggiornare i dati.</span><span class="sxs-lookup"><span data-stu-id="11522-104">In earlier tutorials, you learned how to update data.</span></span> <span data-ttu-id="11522-105">Questa esercitazione descrive la gestione dei conflitti quando più utenti aggiornano la stessa entità contemporaneamente.</span><span class="sxs-lookup"><span data-stu-id="11522-105">This tutorial shows how to handle conflicts when multiple users update the same entity at the same time.</span></span>

<span data-ttu-id="11522-106">Si creano pagine Web che funzionano con l'entità Department (Reparto) e si gestiscono gli errori di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-106">You'll create web pages that work with the Department entity and handle concurrency errors.</span></span> <span data-ttu-id="11522-107">Le illustrazioni seguenti visualizzano le pagine Edit (Modifica) e Delete (Elimina) e includono alcuni messaggi che vengono visualizzati se si verifica un conflitto di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-107">The following illustrations show the Edit and Delete pages, including some messages that are displayed if a concurrency conflict occurs.</span></span>

![Pagina Department Edit (Modifica - Reparto)](concurrency/_static/edit-error.png)

![Pagina Department Delete (Elimina - Reparto)](concurrency/_static/delete-error.png)

<span data-ttu-id="11522-110">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="11522-110">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="11522-111">Scoprire di più sui conflitti di concorrenza</span><span class="sxs-lookup"><span data-stu-id="11522-111">Learn about concurrency conflicts</span></span>
> * <span data-ttu-id="11522-112">Aggiungere una proprietà di rilevamento modifiche</span><span class="sxs-lookup"><span data-stu-id="11522-112">Add a tracking property</span></span>
> * <span data-ttu-id="11522-113">Creare un controller e visualizzazioni Departments</span><span class="sxs-lookup"><span data-stu-id="11522-113">Create Departments controller and views</span></span>
> * <span data-ttu-id="11522-114">Aggiornare la visualizzazione Index</span><span class="sxs-lookup"><span data-stu-id="11522-114">Update Index view</span></span>
> * <span data-ttu-id="11522-115">Aggiornare i metodi Edit</span><span class="sxs-lookup"><span data-stu-id="11522-115">Update Edit methods</span></span>
> * <span data-ttu-id="11522-116">Aggiornare la visualizzazione Edit</span><span class="sxs-lookup"><span data-stu-id="11522-116">Update Edit view</span></span>
> * <span data-ttu-id="11522-117">Testare i conflitti di concorrenza</span><span class="sxs-lookup"><span data-stu-id="11522-117">Test concurrency conflicts</span></span>
> * <span data-ttu-id="11522-118">Aggiornare la pagina Delete (Elimina)</span><span class="sxs-lookup"><span data-stu-id="11522-118">Update the Delete page</span></span>
> * <span data-ttu-id="11522-119">Aggiornare le visualizzazioni Details (Dettagli) e Create (Crea)</span><span class="sxs-lookup"><span data-stu-id="11522-119">Update Details and Create views</span></span>

## <a name="prerequisites"></a><span data-ttu-id="11522-120">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="11522-120">Prerequisites</span></span>

* [<span data-ttu-id="11522-121">Aggiornare dati correlati</span><span class="sxs-lookup"><span data-stu-id="11522-121">Update related data</span></span>](update-related-data.md)

## <a name="concurrency-conflicts"></a><span data-ttu-id="11522-122">Conflitti di concorrenza</span><span class="sxs-lookup"><span data-stu-id="11522-122">Concurrency conflicts</span></span>

<span data-ttu-id="11522-123">Un conflitto di concorrenza si verifica quando un utente visualizza i dati di un'entità per modificarli mentre un altro utente aggiorna i dati della stessa entità prima che la modifica del primo utente venga scritta nel database.</span><span class="sxs-lookup"><span data-stu-id="11522-123">A concurrency conflict occurs when one user displays an entity's data in order to edit it, and then another user updates the same entity's data before the first user's change is written to the database.</span></span> <span data-ttu-id="11522-124">Se non si abilita il rilevamento di questi conflitti, l'ultimo utente che aggiorna il database sovrascrive le modifiche apportate dall'altro utente.</span><span class="sxs-lookup"><span data-stu-id="11522-124">If you don't enable the detection of such conflicts, whoever updates the database last overwrites the other user's changes.</span></span> <span data-ttu-id="11522-125">In molte applicazioni questo rischio è accettabile: se il numero di utenti è ridotto o se l'eventuale sovrascrittura di alcune modifiche non è un aspetto critico, i costi della programmazione per la concorrenza possono superare i vantaggi.</span><span class="sxs-lookup"><span data-stu-id="11522-125">In many applications, this risk is acceptable: if there are few users, or few updates, or if isn't really critical if some changes are overwritten, the cost of programming for concurrency might outweigh the benefit.</span></span> <span data-ttu-id="11522-126">In tal caso non è necessario configurare l'applicazione per la gestione dei conflitti di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-126">In that case, you don't have to configure the application to handle concurrency conflicts.</span></span>

### <a name="pessimistic-concurrency-locking"></a><span data-ttu-id="11522-127">Concorrenza pessimistica (blocco)</span><span class="sxs-lookup"><span data-stu-id="11522-127">Pessimistic concurrency (locking)</span></span>

<span data-ttu-id="11522-128">Se è importante che l'applicazione eviti la perdita accidentale di dati in scenari di concorrenza, un metodo per garantire che ciò accada è l'uso dei blocchi di database.</span><span class="sxs-lookup"><span data-stu-id="11522-128">If your application does need to prevent accidental data loss in concurrency scenarios, one way to do that is to use database locks.</span></span> <span data-ttu-id="11522-129">Questo approccio è denominato concorrenza pessimistica.</span><span class="sxs-lookup"><span data-stu-id="11522-129">This is called pessimistic concurrency.</span></span> <span data-ttu-id="11522-130">Ad esempio, prima di leggere una riga da un database si richiede un blocco per l'accesso di sola lettura o per l'accesso in modalità aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="11522-130">For example, before you read a row from a database, you request a lock for read-only or for update access.</span></span> <span data-ttu-id="11522-131">Se si blocca una riga per l'accesso di aggiornamento, nessun altro utente può bloccare la riga per l'accesso di sola lettura o di aggiornamento, perché riceverebbe una copia di dati dei quali è in corso la modifica.</span><span class="sxs-lookup"><span data-stu-id="11522-131">If you lock a row for update access, no other users are allowed to lock the row either for read-only or update access, because they would get a copy of data that's in the process of being changed.</span></span> <span data-ttu-id="11522-132">Se si blocca una riga per l'accesso in sola lettura, anche altri utenti possono bloccarla per l'accesso in sola lettura, ma non per l'aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="11522-132">If you lock a row for read-only access, others can also lock it for read-only access but not for update.</span></span>

<span data-ttu-id="11522-133">La gestione dei blocchi presenta svantaggi.</span><span class="sxs-lookup"><span data-stu-id="11522-133">Managing locks has disadvantages.</span></span> <span data-ttu-id="11522-134">La sua programmazione può risultare complicata.</span><span class="sxs-lookup"><span data-stu-id="11522-134">It can be complex to program.</span></span> <span data-ttu-id="11522-135">Richiede molte risorse di gestione del database e può causare problemi di prestazioni quando il numero di utenti di un'applicazione aumenta.</span><span class="sxs-lookup"><span data-stu-id="11522-135">It requires significant database management resources, and it can cause performance problems as the number of users of an application increases.</span></span> <span data-ttu-id="11522-136">Per questi motivi non tutti i sistemi di gestione di database supportano la concorrenza pessimistica.</span><span class="sxs-lookup"><span data-stu-id="11522-136">For these reasons, not all database management systems support pessimistic concurrency.</span></span> <span data-ttu-id="11522-137">Entity Framework Core non offre supporto predefinito per questa modalità e la presente esercitazione non indica come implementarla.</span><span class="sxs-lookup"><span data-stu-id="11522-137">Entity Framework Core provides no built-in support for it, and this tutorial doesn't show you how to implement it.</span></span>

### <a name="optimistic-concurrency"></a><span data-ttu-id="11522-138">Concorrenza ottimistica</span><span class="sxs-lookup"><span data-stu-id="11522-138">Optimistic Concurrency</span></span>

<span data-ttu-id="11522-139">L'alternativa alla concorrenza pessimistica è la concorrenza ottimistica.</span><span class="sxs-lookup"><span data-stu-id="11522-139">The alternative to pessimistic concurrency is optimistic concurrency.</span></span> <span data-ttu-id="11522-140">Nella concorrenza ottimistica si consente che i conflitti di concorrenza si verifichino, quindi si reagisce con le modalità appropriate.</span><span class="sxs-lookup"><span data-stu-id="11522-140">Optimistic concurrency means allowing concurrency conflicts to happen, and then reacting appropriately if they do.</span></span> <span data-ttu-id="11522-141">Ad esempio Jane visita la pagina Department Edit (Modifica - Reparto) e cambia l'importo di Budget per il reparto English (Inglese) da $ 350.000,00 a $ 0,00.</span><span class="sxs-lookup"><span data-stu-id="11522-141">For example, Jane visits the Department Edit page and changes the Budget amount for the English department from $350,000.00 to $0.00.</span></span>

![Impostazione del budget su 0](concurrency/_static/change-budget.png)

<span data-ttu-id="11522-143">Prima che Jane faccia clic su **Salva** John visita la stessa pagina e cambia il valore del campo Start Date (Data inizio) da 9/1/2007 a 9/1/2013.</span><span class="sxs-lookup"><span data-stu-id="11522-143">Before Jane clicks **Save**, John visits the same page and changes the Start Date field from 9/1/2007 to 9/1/2013.</span></span>

![Impostazione della data di inizio su 2013](concurrency/_static/change-date.png)

<span data-ttu-id="11522-145">Jane fa clic su **Salva** per prima e vede la sua modifica quando il browser torna alla pagina di indice.</span><span class="sxs-lookup"><span data-stu-id="11522-145">Jane clicks **Save** first and sees her change when the browser returns to the Index page.</span></span>

![Budget impostato su zero](concurrency/_static/budget-zero.png)

<span data-ttu-id="11522-147">Quindi John fa clic su **Salva** in una pagina Edit (Modifica) che visualizza ancora un budget pari a $ 350.000,00.</span><span class="sxs-lookup"><span data-stu-id="11522-147">Then John clicks **Save** on an Edit page that still shows a budget of $350,000.00.</span></span> <span data-ttu-id="11522-148">Le operazioni successive dipendono da come si decide di gestire i conflitti di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-148">What happens next is determined by how you handle concurrency conflicts.</span></span>

<span data-ttu-id="11522-149">Di seguito sono elencate alcune opzioni:</span><span class="sxs-lookup"><span data-stu-id="11522-149">Some of the options include the following:</span></span>

* <span data-ttu-id="11522-150">È possibile tenere traccia della proprietà che un utente ha modificato e aggiornare solo le colonne corrispondenti nel database.</span><span class="sxs-lookup"><span data-stu-id="11522-150">You can keep track of which property a user has modified and update only the corresponding columns in the database.</span></span>

     <span data-ttu-id="11522-151">Nello scenario dell'esempio non si perde nessun dato, perché i due utenti hanno aggiornato proprietà diverse.</span><span class="sxs-lookup"><span data-stu-id="11522-151">In the example scenario, no data would be lost, because different properties were updated by the two users.</span></span> <span data-ttu-id="11522-152">Quando un utente torna a visualizzare il reparto English (Inglese), visualizza sia le modifiche di Jane sia quelle di John: la data di inizio 9/1/2013 e un budget di zero dollari.</span><span class="sxs-lookup"><span data-stu-id="11522-152">The next time someone browses the English department, they will see both Jane's and John's changes -- a start date of 9/1/2013 and a budget of zero dollars.</span></span> <span data-ttu-id="11522-153">Questo metodo di aggiornamento può ridurre il numero di conflitti che causano la perdita di dati, ma non può evitare la perdita di dati se vengono apportate modifiche in competizione tra loro alla stessa proprietà di un'entità.</span><span class="sxs-lookup"><span data-stu-id="11522-153">This method of updating can reduce the number of conflicts that could result in data loss, but it can't avoid data loss if competing changes are made to the same property of an entity.</span></span> <span data-ttu-id="11522-154">Questo funzionamento di Entity Framework dipende dalla modalità di implementazione del codice di aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="11522-154">Whether the Entity Framework works this way depends on how you implement your update code.</span></span> <span data-ttu-id="11522-155">In molti casi in un'app Web questo approccio risulta poco pratico, perché richiede la gestione di grandi quantità di codice statico per tenere traccia di tutti i valori di proprietà originali per un'entità, nonché dei nuovi valori.</span><span class="sxs-lookup"><span data-stu-id="11522-155">It's often not practical in a web application, because it can require that you maintain large amounts of state in order to keep track of all original property values for an entity as well as new values.</span></span> <span data-ttu-id="11522-156">La gestione di grandi quantità di stato può influire sulle prestazioni dell'applicazione perché richiede risorse del server o deve essere inclusa nella pagina Web, ad esempio in campi nascosti, o in un oggetto cookie .</span><span class="sxs-lookup"><span data-stu-id="11522-156">Maintaining large amounts of state can affect application performance because it either requires server resources or must be included in the web page itself (for example, in hidden fields) or in a cookie.</span></span>

* <span data-ttu-id="11522-157">È possibile consentire che la modifica di John sovrascriva la modifica di Jane.</span><span class="sxs-lookup"><span data-stu-id="11522-157">You can let John's change overwrite Jane's change.</span></span>

     <span data-ttu-id="11522-158">Quando un utente torna a visualizzare il reparto English (Inglese), visualizza 9/1/2013 e il valore $ 350.000,00 ripristinato.</span><span class="sxs-lookup"><span data-stu-id="11522-158">The next time someone browses the English department, they will see 9/1/2013 and the restored $350,000.00 value.</span></span> <span data-ttu-id="11522-159">Questo scenario è detto *Priorità client* o *Last in Wins* (Priorità ultimo accesso).</span><span class="sxs-lookup"><span data-stu-id="11522-159">This is called a *Client Wins* or *Last in Wins* scenario.</span></span> <span data-ttu-id="11522-160">Tutti i valori del client hanno la precedenza sugli elementi presenti nell'archivio dati. Come indicato nell'introduzione a questa sezione, se non si esegue alcuna codifica per la gestione della concorrenza, questa operazione viene eseguita automaticamente.</span><span class="sxs-lookup"><span data-stu-id="11522-160">(All values from the client take precedence over what's in the data store.) As noted in the introduction to this section, if you don't do any coding for concurrency handling, this will happen automatically.</span></span>

* <span data-ttu-id="11522-161">È possibile impedire l'aggiornamento del database con la modifica di John.</span><span class="sxs-lookup"><span data-stu-id="11522-161">You can prevent John's change from being updated in the database.</span></span>

     <span data-ttu-id="11522-162">In genere viene visualizzato un messaggio di errore e lo stato corrente dei dati e si consente all'utente di riapplicare le modifiche se lo desidera.</span><span class="sxs-lookup"><span data-stu-id="11522-162">Typically, you would display an error message, show him the current state of the data, and allow him to reapply his changes if he still wants to make them.</span></span> <span data-ttu-id="11522-163">Questo scenario è detto *Store Wins* (Priorità archivio).</span><span class="sxs-lookup"><span data-stu-id="11522-163">This is called a *Store Wins* scenario.</span></span> <span data-ttu-id="11522-164">I valori dell'archivio dati hanno la precedenza sui valori inviati dal client. Verrà implementato lo scenario Store WINS in questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="11522-164">(The data-store values take precedence over the values submitted by the client.) You'll implement the Store Wins scenario in this tutorial.</span></span> <span data-ttu-id="11522-165">Questo metodo garantisce che nessuna modifica venga sovrascritta senza che un utente riceva un avviso.</span><span class="sxs-lookup"><span data-stu-id="11522-165">This method ensures that no changes are overwritten without a user being alerted to what's happening.</span></span>

### <a name="detecting-concurrency-conflicts"></a><span data-ttu-id="11522-166">Rilevamento dei conflitti di concorrenza</span><span class="sxs-lookup"><span data-stu-id="11522-166">Detecting concurrency conflicts</span></span>

<span data-ttu-id="11522-167">È possibile risolvere i conflitti gestendo le eccezioni `DbConcurrencyException` generate da Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="11522-167">You can resolve conflicts by handling `DbConcurrencyException` exceptions that the Entity Framework throws.</span></span> <span data-ttu-id="11522-168">Per determinare quando generare queste eccezioni, Entity Framework deve essere in grado di rilevare i conflitti.</span><span class="sxs-lookup"><span data-stu-id="11522-168">In order to know when to throw these exceptions, the Entity Framework must be able to detect conflicts.</span></span> <span data-ttu-id="11522-169">Pertanto è necessario configurare il database e il modello di dati in modo appropriato.</span><span class="sxs-lookup"><span data-stu-id="11522-169">Therefore, you must configure the database and the data model appropriately.</span></span> <span data-ttu-id="11522-170">Di seguito sono elencate alcune opzioni per abilitare il rilevamento dei conflitti:</span><span class="sxs-lookup"><span data-stu-id="11522-170">Some options for enabling conflict detection include the following:</span></span>

* <span data-ttu-id="11522-171">Nella tabella del database, includere una colonna di rilevamento che può essere usata per determinare quando è stata modificata una riga.</span><span class="sxs-lookup"><span data-stu-id="11522-171">In the database table, include a tracking column that can be used to determine when a row has been changed.</span></span> <span data-ttu-id="11522-172">Quindi è possibile configurare Entity Framework per includere tale colonna nella clausola Where dei comandi SQL Update o Delete.</span><span class="sxs-lookup"><span data-stu-id="11522-172">You can then configure the Entity Framework to include that column in the Where clause of SQL Update or Delete commands.</span></span>

     <span data-ttu-id="11522-173">Il tipo di dati della colonna di rilevamento è in genere `rowversion`.</span><span class="sxs-lookup"><span data-stu-id="11522-173">The data type of the tracking column is typically `rowversion`.</span></span> <span data-ttu-id="11522-174">Il valore `rowversion` è un numero sequenziale che viene incrementato ogni volta che la riga viene aggiornata.</span><span class="sxs-lookup"><span data-stu-id="11522-174">The `rowversion` value is a sequential number that's incremented each time the row is updated.</span></span> <span data-ttu-id="11522-175">In un comando Update o Delete, la clausola Where include il valore originale della colonna di rilevamento (la versione originale della riga).</span><span class="sxs-lookup"><span data-stu-id="11522-175">In an Update or Delete command, the Where clause includes the original value of the tracking column (the original row version) .</span></span> <span data-ttu-id="11522-176">Se la riga in corso di aggiornamento è stata modificata da un altro utente, il valore della colonna `rowversion` è diverso dal valore originale, pertanto l'istruzione Update o Delete non trova la riga da aggiornare a causa della clausola Where.</span><span class="sxs-lookup"><span data-stu-id="11522-176">If the row being updated has been changed by another user, the value in the `rowversion` column is different than the original value, so the Update or Delete statement can't find the row to update because of the Where clause.</span></span> <span data-ttu-id="11522-177">Quando Entity Framework rileva che nessuna riga è stata aggiornata dal comando Update o Delete (ovvero quando il numero di righe interessate è pari a zero), interpreta questo fatto come un conflitto di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-177">When the Entity Framework finds that no rows have been updated by the Update or Delete command (that is, when the number of affected rows is zero), it interprets that as a concurrency conflict.</span></span>

* <span data-ttu-id="11522-178">Configurare Entity Framework in modo che includa i valori originali di ogni colonna della tabella nella clausola Where dei comandi Update e Delete.</span><span class="sxs-lookup"><span data-stu-id="11522-178">Configure the Entity Framework to include the original values of every column in the table in the Where clause of Update and Delete commands.</span></span>

     <span data-ttu-id="11522-179">Come nella prima opzione, se è stata apportata qualsiasi modifica dopo la lettura iniziale della riga, la clausola Where non restituisce una riga a Update ed Entity Framework interpreta questo fatto come un conflitto di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-179">As in the first option, if anything in the row has changed since the row was first read, the Where clause won't return a row to update, which the Entity Framework interprets as a concurrency conflict.</span></span> <span data-ttu-id="11522-180">Per le tabelle di database con molte colonne, questo approccio può risultare in clausole Where molto grandi e richiedere grandi quantità di stato.</span><span class="sxs-lookup"><span data-stu-id="11522-180">For database tables that have many columns, this approach can result in very large Where clauses, and can require that you maintain large amounts of state.</span></span> <span data-ttu-id="11522-181">Come notato in precedenza, la gestione di grandi quantità di codice statico può ridurre le prestazioni dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="11522-181">As noted earlier, maintaining large amounts of state can affect application performance.</span></span> <span data-ttu-id="11522-182">Pertanto questo approccio è in genere sconsigliato e non è il metodo usato in questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="11522-182">Therefore this approach is generally not recommended, and it isn't the method used in this tutorial.</span></span>

     <span data-ttu-id="11522-183">Se si vuole comunque implementare questo approccio alla concorrenza, è necessario contrassegnare con l'aggiunta dell'attributo `ConcurrencyCheck` tutte le proprietà dell'entità che non sono chiavi primarie e per le quali si vuole tenere traccia della concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-183">If you do want to implement this approach to concurrency, you have to mark all non-primary-key properties in the entity you want to track concurrency for by adding the `ConcurrencyCheck` attribute to them.</span></span> <span data-ttu-id="11522-184">Grazie a questa modifica, Entity Framework include tutte le colonne nella clausola SQL Where delle istruzioni Update e Delete.</span><span class="sxs-lookup"><span data-stu-id="11522-184">That change enables the Entity Framework to include all columns in the SQL Where clause of Update and Delete statements.</span></span>

<span data-ttu-id="11522-185">Nella parte restante di questa esercitazione si aggiunge una proprietà di rilevamento `rowversion` all'entità Department (Reparto), si crea un controller e delle visualizzazioni e si esegue un test per verificare che tutto funzioni correttamente.</span><span class="sxs-lookup"><span data-stu-id="11522-185">In the remainder of this tutorial you'll add a `rowversion` tracking property to the Department entity, create a controller and views, and test to verify that everything works correctly.</span></span>

## <a name="add-a-tracking-property"></a><span data-ttu-id="11522-186">Aggiungere una proprietà di rilevamento modifiche</span><span class="sxs-lookup"><span data-stu-id="11522-186">Add a tracking property</span></span>

<span data-ttu-id="11522-187">In *Models/Department.cs* aggiungere una proprietà di rilevamento denominata RowVersion:</span><span class="sxs-lookup"><span data-stu-id="11522-187">In *Models/Department.cs*, add a tracking property named RowVersion:</span></span>

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Final&highlight=26,27)]

<span data-ttu-id="11522-188">L'attributo `Timestamp` specifica che questa colonna viene inclusa nella clausola Where dei comandi Update e Delete inviati al database.</span><span class="sxs-lookup"><span data-stu-id="11522-188">The `Timestamp` attribute specifies that this column will be included in the Where clause of Update and Delete commands sent to the database.</span></span> <span data-ttu-id="11522-189">L'attributo viene chiamato `Timestamp` perché le versioni precedenti di SQL Server usavano un tipo di dati SQL `timestamp` prima che questo fosse sostituito dalla notazione SQL `rowversion`.</span><span class="sxs-lookup"><span data-stu-id="11522-189">The attribute is called `Timestamp` because previous versions of SQL Server used a SQL `timestamp` data type before the SQL `rowversion` replaced it.</span></span> <span data-ttu-id="11522-190">Il tipo .NET per `rowversion` è una matrice di byte.</span><span class="sxs-lookup"><span data-stu-id="11522-190">The .NET type for `rowversion` is a byte array.</span></span>

<span data-ttu-id="11522-191">Se si preferisce usare l'API Fluent, è possibile usare il metodo `IsConcurrencyToken` (in *Data/SchoolContext.cs*) per specificare la proprietà di rilevamento, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="11522-191">If you prefer to use the fluent API, you can use the `IsConcurrencyToken` method (in *Data/SchoolContext.cs*) to specify the tracking property, as shown in the following example:</span></span>

```csharp
modelBuilder.Entity<Department>()
    .Property(p => p.RowVersion).IsConcurrencyToken();
```

<span data-ttu-id="11522-192">In seguito all'aggiunta di una proprietà il modello di database è stato modificato, pertanto è necessario eseguire una nuova migrazione.</span><span class="sxs-lookup"><span data-stu-id="11522-192">By adding a property you changed the database model, so you need to do another migration.</span></span>

<span data-ttu-id="11522-193">Salvare le modifiche e compilare il progetto, quindi immettere i comandi seguenti nella finestra di comando:</span><span class="sxs-lookup"><span data-stu-id="11522-193">Save your changes and build the project, and then enter the following commands in the command window:</span></span>

```dotnetcli
dotnet ef migrations add RowVersion
```

```dotnetcli
dotnet ef database update
```

## <a name="create-departments-controller-and-views"></a><span data-ttu-id="11522-194">Creare un controller e visualizzazioni Departments</span><span class="sxs-lookup"><span data-stu-id="11522-194">Create Departments controller and views</span></span>

<span data-ttu-id="11522-195">Eseguire lo scaffolding di un controller e di visualizzazioni Departments come già fatto in precedenza per Students, Courses e Instructors.</span><span class="sxs-lookup"><span data-stu-id="11522-195">Scaffold a Departments controller and views as you did earlier for Students, Courses, and Instructors.</span></span>

![Scaffolding di Department](concurrency/_static/add-departments-controller.png)

<span data-ttu-id="11522-197">Nel file *DepartmentsController.cs*, convertire tutte e quattro le ricorrenze di "FirstMidName" in "FullName" in modo che gli elenchi a discesa degli amministratori di reparto contengano il nome completo dell'insegnante anziché solo il cognome.</span><span class="sxs-lookup"><span data-stu-id="11522-197">In the *DepartmentsController.cs* file, change all four occurrences of "FirstMidName" to "FullName" so that the department administrator drop-down lists will contain the full name of the instructor rather than just the last name.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_Dropdown)]

## <a name="update-index-view"></a><span data-ttu-id="11522-198">Aggiornare la visualizzazione Index</span><span class="sxs-lookup"><span data-stu-id="11522-198">Update Index view</span></span>

<span data-ttu-id="11522-199">Il motore di scaffolding crea una colonna RowVersion nella vista Index, ma questo campo non deve essere visualizzato.</span><span class="sxs-lookup"><span data-stu-id="11522-199">The scaffolding engine created a RowVersion column in the Index view, but that field shouldn't be displayed.</span></span>

<span data-ttu-id="11522-200">Sostituire il codice in *Views/Departments/Index.cshtml* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="11522-200">Replace the code in *Views/Departments/Index.cshtml* with the following code.</span></span>

[!code-cshtml[](intro/samples/cu/Views/Departments/Index.cshtml?highlight=4,7,44)]

<span data-ttu-id="11522-201">Questo codice imposta come intestazione "Departments" (Reparti), elimina la colonna RowVersion e visualizza il nome completo anziché solo il cognome dell'amministratore.</span><span class="sxs-lookup"><span data-stu-id="11522-201">This changes the heading to "Departments", deletes the RowVersion column, and shows full name instead of first name for the administrator.</span></span>

## <a name="update-edit-methods"></a><span data-ttu-id="11522-202">Aggiornare i metodi Edit</span><span class="sxs-lookup"><span data-stu-id="11522-202">Update Edit methods</span></span>

<span data-ttu-id="11522-203">Sia nel metodo HttpGet `Edit` che nel metodo `Details`, aggiungere `AsNoTracking`.</span><span class="sxs-lookup"><span data-stu-id="11522-203">In both the HttpGet `Edit` method and the `Details` method, add `AsNoTracking`.</span></span> <span data-ttu-id="11522-204">Nel metodo HttpGet `Edit` aggiungere il caricamento eager per Administrator.</span><span class="sxs-lookup"><span data-stu-id="11522-204">In the HttpGet `Edit` method, add eager loading for the Administrator.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EagerLoading)]

<span data-ttu-id="11522-205">Sostituire il codice esistente del metodo HttpPost `Edit` con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="11522-205">Replace the existing code for the HttpPost `Edit` method with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EditPost)]

<span data-ttu-id="11522-206">Il codice inizia con la lettura del reparto da aggiornare.</span><span class="sxs-lookup"><span data-stu-id="11522-206">The code begins by trying to read the department to be updated.</span></span> <span data-ttu-id="11522-207">Se il metodo `FirstOrDefaultAsync` restituisce null, il reparto è stato eliminato da un altro utente.</span><span class="sxs-lookup"><span data-stu-id="11522-207">If the `FirstOrDefaultAsync` method returns null, the department was deleted by another user.</span></span> <span data-ttu-id="11522-208">In questo caso il codice usa i valori del modulo registrato per creare un'entità reparto, in modo che la pagina di modifica possa essere visualizzata nuovamente con un messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="11522-208">In that case the code uses the posted form values to create a department entity so that the Edit page can be redisplayed with an error message.</span></span> <span data-ttu-id="11522-209">In alternativa, non è necessario creare nuovamente l'entità del reparto se si visualizza solo un messaggio di errore senza visualizzare di nuovo i campi del reparto.</span><span class="sxs-lookup"><span data-stu-id="11522-209">As an alternative, you wouldn't have to re-create the department entity if you display only an error message without redisplaying the department fields.</span></span>

<span data-ttu-id="11522-210">La visualizzazione archivia il valore `RowVersion` originale in un campo nascosto e questo metodo riceve il valore nel parametro `rowVersion`.</span><span class="sxs-lookup"><span data-stu-id="11522-210">The view stores the original `RowVersion` value in a hidden field, and this method receives that value in the `rowVersion` parameter.</span></span> <span data-ttu-id="11522-211">Prima della chiamata di `SaveChanges` è necessario inserire il valore originale della proprietà `RowVersion` nella raccolta `OriginalValues` dell'entità.</span><span class="sxs-lookup"><span data-stu-id="11522-211">Before you call `SaveChanges`, you have to put that original `RowVersion` property value in the `OriginalValues` collection for the entity.</span></span>

```csharp
_context.Entry(departmentToUpdate).Property("RowVersion").OriginalValue = rowVersion;
```

<span data-ttu-id="11522-212">Quando in seguito Entity Framework crea un comando SQL UPDATE, il comando includerà una clausola WHERE che cerca una riga con il valore `RowVersion` originale.</span><span class="sxs-lookup"><span data-stu-id="11522-212">Then when the Entity Framework creates a SQL UPDATE command, that command will include a WHERE clause that looks for a row that has the original `RowVersion` value.</span></span> <span data-ttu-id="11522-213">Se nessuna riga è interessata dal comando UPDATE (nessuna riga presenta il valore `RowVersion` originale), Entity Framework genera un'eccezione `DbUpdateConcurrencyException`.</span><span class="sxs-lookup"><span data-stu-id="11522-213">If no rows are affected by the UPDATE command (no rows have the original `RowVersion` value),  the Entity Framework throws a `DbUpdateConcurrencyException` exception.</span></span>

<span data-ttu-id="11522-214">Il codice del blocco catch di tale eccezione ottiene l'entità Department interessata con i valori aggiornati dalla proprietà `Entries` nell'oggetto eccezione.</span><span class="sxs-lookup"><span data-stu-id="11522-214">The code in the catch block for that exception gets the affected Department entity that has the updated values from the `Entries` property on the exception object.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=164)]

<span data-ttu-id="11522-215">La raccolta `Entries` ha un solo oggetto `EntityEntry`.</span><span class="sxs-lookup"><span data-stu-id="11522-215">The `Entries` collection will have just one `EntityEntry` object.</span></span>  <span data-ttu-id="11522-216">È possibile usare tale oggetto per ottenere i nuovi valori immessi dall'utente e i valori del database corrente.</span><span class="sxs-lookup"><span data-stu-id="11522-216">You can use that object to get the new values entered by the user and the current database values.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=165-166)]

<span data-ttu-id="11522-217">Il codice aggiunge un messaggio di errore personalizzato per ogni colonna che ha valori di database diversi da ciò che l'utente ha immesso nella pagina Edit (per brevità qui viene riportato un solo campo).</span><span class="sxs-lookup"><span data-stu-id="11522-217">The code adds a custom error message for each column that has database values different from what the user entered on the Edit page (only one field is shown here for brevity).</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=174-178)]

<span data-ttu-id="11522-218">Infine il codice imposta il valore `RowVersion` di `departmentToUpdate` sul nuovo valore recuperato dal database.</span><span class="sxs-lookup"><span data-stu-id="11522-218">Finally, the code sets the `RowVersion` value of the `departmentToUpdate` to the new value retrieved from the database.</span></span> <span data-ttu-id="11522-219">Questo nuovo valore `RowVersion` viene archiviato nel campo nascosto quando viene visualizzata nuovamente la pagina Edit (Modifica). Quando l'utente torna a fare clic su **Salva** vengono rilevati solo gli errori di concorrenza che si verificano dopo la nuova visualizzazione della pagina Edit (Modifica).</span><span class="sxs-lookup"><span data-stu-id="11522-219">This new `RowVersion` value will be stored in the hidden field when the Edit page is redisplayed, and the next time the user clicks **Save**, only concurrency errors that happen since the redisplay of the Edit page will be caught.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=199-200)]

<span data-ttu-id="11522-220">L'istruzione `ModelState.Remove` è necessaria perché `ModelState` presenta il valore obsoleto `RowVersion`.</span><span class="sxs-lookup"><span data-stu-id="11522-220">The `ModelState.Remove` statement is required because `ModelState` has the old `RowVersion` value.</span></span> <span data-ttu-id="11522-221">Nella visualizzazione, il valore `ModelState` di un campo ha la precedenza sui valori di proprietà del modello quando entrambi gli elementi sono presenti.</span><span class="sxs-lookup"><span data-stu-id="11522-221">In the view, the `ModelState` value for a field takes precedence over the model property values when both are present.</span></span>

## <a name="update-edit-view"></a><span data-ttu-id="11522-222">Aggiornare la visualizzazione Edit</span><span class="sxs-lookup"><span data-stu-id="11522-222">Update Edit view</span></span>

<span data-ttu-id="11522-223">In *Views/Departments/Edit.cshtml* apportare le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="11522-223">In *Views/Departments/Edit.cshtml*, make the following changes:</span></span>

* <span data-ttu-id="11522-224">Aggiungere un campo nascosto per salvare il valore della proprietà `RowVersion` subito dopo il campo nascosto della proprietà `DepartmentID`.</span><span class="sxs-lookup"><span data-stu-id="11522-224">Add a hidden field to save the `RowVersion` property value, immediately following the hidden field for the `DepartmentID` property.</span></span>

* <span data-ttu-id="11522-225">Aggiungere un'opzione "Select Administrator" (Seleziona amministratore) all'elenco a discesa.</span><span class="sxs-lookup"><span data-stu-id="11522-225">Add a "Select Administrator" option to the drop-down list.</span></span>

[!code-cshtml[](intro/samples/cu/Views/Departments/Edit.cshtml?highlight=16,34-36)]

## <a name="test-concurrency-conflicts"></a><span data-ttu-id="11522-226">Testare i conflitti di concorrenza</span><span class="sxs-lookup"><span data-stu-id="11522-226">Test concurrency conflicts</span></span>

<span data-ttu-id="11522-227">Eseguire l'app e passare alla pagina Departments Index (Indice reparti).</span><span class="sxs-lookup"><span data-stu-id="11522-227">Run the app and go to the Departments Index page.</span></span> <span data-ttu-id="11522-228">Fare clic con il pulsante destro del mouse sul collegamento ipertestuale **Edit** (Modifica) per il reparto English (Inglese) e selezionare **Apri link in nuova scheda**, quindi fare clic sul collegamento ipertestuale **Edit** (Modifica) per il reparto English (Inglese).</span><span class="sxs-lookup"><span data-stu-id="11522-228">Right-click the **Edit** hyperlink for the English department and select **Open in new tab**, then click the **Edit** hyperlink for the English department.</span></span> <span data-ttu-id="11522-229">Le due schede del browser ora visualizzano le stesse informazioni.</span><span class="sxs-lookup"><span data-stu-id="11522-229">The two browser tabs now display the same information.</span></span>

<span data-ttu-id="11522-230">Modificare un campo nella prima scheda del browser e fare clic su **Salva**.</span><span class="sxs-lookup"><span data-stu-id="11522-230">Change a field in the first browser tab and click **Save**.</span></span>

![Pagina Department Edit (Modifica - Reparto) 1 dopo la modifica](concurrency/_static/edit-after-change-1.png)

<span data-ttu-id="11522-232">Il browser visualizza la pagina Index con il valore modificato.</span><span class="sxs-lookup"><span data-stu-id="11522-232">The browser shows the Index page with the changed value.</span></span>

<span data-ttu-id="11522-233">Modificare un campo nella seconda scheda del browser.</span><span class="sxs-lookup"><span data-stu-id="11522-233">Change a field in the second browser tab.</span></span>

![Pagina Department Edit (Modifica - Reparto) 2 dopo la modifica](concurrency/_static/edit-after-change-2.png)

<span data-ttu-id="11522-235">Fare clic su **Salva**.</span><span class="sxs-lookup"><span data-stu-id="11522-235">Click **Save**.</span></span> <span data-ttu-id="11522-236">Viene visualizzato un messaggio di errore:</span><span class="sxs-lookup"><span data-stu-id="11522-236">You see an error message:</span></span>

![Messaggio di errore della pagina Department Edit (Modifica - Reparto)](concurrency/_static/edit-error.png)

<span data-ttu-id="11522-238">Fare clic su **Salva**.</span><span class="sxs-lookup"><span data-stu-id="11522-238">Click **Save** again.</span></span> <span data-ttu-id="11522-239">Il valore immesso nella seconda scheda del browser viene salvato.</span><span class="sxs-lookup"><span data-stu-id="11522-239">The value you entered in the second browser tab is saved.</span></span> <span data-ttu-id="11522-240">I valori salvati vengono visualizzati nella pagina Index.</span><span class="sxs-lookup"><span data-stu-id="11522-240">You see the saved values when the Index page appears.</span></span>

## <a name="update-the-delete-page"></a><span data-ttu-id="11522-241">Aggiornare la pagina Delete (Elimina)</span><span class="sxs-lookup"><span data-stu-id="11522-241">Update the Delete page</span></span>

<span data-ttu-id="11522-242">Per la pagina Delete (Elimina), Entity Framework rileva conflitti di concorrenza causati da un altro utente che ha modificato il reparto con modalità simili.</span><span class="sxs-lookup"><span data-stu-id="11522-242">For the Delete page, the Entity Framework detects concurrency conflicts caused by someone else editing the department in a similar manner.</span></span> <span data-ttu-id="11522-243">Quando il metodo HttpGet `Delete` visualizza la conferma, la visualizzazione include il valore `RowVersion` originale in un campo nascosto.</span><span class="sxs-lookup"><span data-stu-id="11522-243">When the HttpGet `Delete` method displays the confirmation view, the view includes the original `RowVersion` value in a hidden field.</span></span> <span data-ttu-id="11522-244">Questo valore viene quindi reso disponibile al metodo HttpPost `Delete` che viene chiamato quando l'utente conferma l'eliminazione.</span><span class="sxs-lookup"><span data-stu-id="11522-244">That value is then available to the HttpPost `Delete` method that's called when the user confirms the deletion.</span></span> <span data-ttu-id="11522-245">Quando Entity Framework crea il comando SQL DELETE, include una clausola WHERE con il valore `RowVersion` originale.</span><span class="sxs-lookup"><span data-stu-id="11522-245">When the Entity Framework creates the SQL DELETE command, it includes a WHERE clause with the original `RowVersion` value.</span></span> <span data-ttu-id="11522-246">Se il comando non ha effetto su nessuna riga (ovvero se la riga è stata modificata dopo la visualizzazione della pagina di conferma Delete) viene attivata un'eccezione di concorrenza e viene chiamato il metodo HttpGet `Delete` con un flag di errore impostato su true, per tornare a visualizzare la pagina di conferma con un messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="11522-246">If the command results in zero rows affected (meaning the row was changed after the Delete confirmation page was displayed), a concurrency exception is thrown, and the HttpGet `Delete` method is called with an error flag set to true in order to redisplay the confirmation page with an error message.</span></span> <span data-ttu-id="11522-247">È anche possibile che il comando non abbia effetto su nessuna riga perché la riga è stata eliminata da un altro utente. In questo caso non viene visualizzato nessun messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="11522-247">It's also possible that zero rows were affected because the row was deleted by another user, so in that case no error message is displayed.</span></span>

### <a name="update-the-delete-methods-in-the-departments-controller"></a><span data-ttu-id="11522-248">Aggiornare i metodi Delete nel controller Departments</span><span class="sxs-lookup"><span data-stu-id="11522-248">Update the Delete methods in the Departments controller</span></span>

<span data-ttu-id="11522-249">In *DepartmentsController.cs*, sostituire il metodo HttpGet `Delete` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="11522-249">In *DepartmentsController.cs*, replace the HttpGet `Delete` method with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeleteGet&highlight=1,10,14-17,21-29)]

<span data-ttu-id="11522-250">Il metodo accetta un parametro facoltativo che indica se la pagina viene nuovamente visualizzata dopo un errore di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-250">The method accepts an optional parameter that indicates whether the page is being redisplayed after a concurrency error.</span></span> <span data-ttu-id="11522-251">Se questo flag è true e il reparto specificato non esiste più significa che è stato eliminato da un altro utente.</span><span class="sxs-lookup"><span data-stu-id="11522-251">If this flag is true and the department specified no longer exists, it was deleted by another user.</span></span> <span data-ttu-id="11522-252">In questo caso, il codice esegue il reindirizzamento alla pagina Index.</span><span class="sxs-lookup"><span data-stu-id="11522-252">In that case, the code redirects to the Index page.</span></span>  <span data-ttu-id="11522-253">Se questo flag è true e il reparto esiste, è stato modificato da un altro utente.</span><span class="sxs-lookup"><span data-stu-id="11522-253">If this flag is true and the Department does exist, it was changed by another user.</span></span> <span data-ttu-id="11522-254">In questo caso il codice invia un messaggio di errore alla vista usando `ViewData`.</span><span class="sxs-lookup"><span data-stu-id="11522-254">In that case, the code sends an error message to the view using `ViewData`.</span></span>

<span data-ttu-id="11522-255">Sostituire il codice nel metodo HttpPost `Delete` (denominato `DeleteConfirmed`) con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="11522-255">Replace the code in the HttpPost `Delete` method (named `DeleteConfirmed`) with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeletePost&highlight=1,3,5-8,11-18)]

<span data-ttu-id="11522-256">Nel codice sottoposto a scaffolding appena sostituito, questo metodo accettava solo un ID record:</span><span class="sxs-lookup"><span data-stu-id="11522-256">In the scaffolded code that you just replaced, this method accepted only a record ID:</span></span>

```csharp
public async Task<IActionResult> DeleteConfirmed(int id)
```

<span data-ttu-id="11522-257">Questo parametro è stato convertito in un'istanza di entità Department creata dallo strumento di associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="11522-257">You've changed this parameter to a Department entity instance created by the model binder.</span></span> <span data-ttu-id="11522-258">Ciò consente a EF di accedere al valore della proprietà RowVersion oltre che alla chiave del record.</span><span class="sxs-lookup"><span data-stu-id="11522-258">This gives EF access to the RowVersion property value in addition to the record key.</span></span>

```csharp
public async Task<IActionResult> Delete(Department department)
```

<span data-ttu-id="11522-259">Anche il nome del metodo di azione è stato modificato da `DeleteConfirmed` a `Delete`.</span><span class="sxs-lookup"><span data-stu-id="11522-259">You have also changed the action method name from `DeleteConfirmed` to `Delete`.</span></span> <span data-ttu-id="11522-260">Il codice sottoposto a scaffolding usava il nome `DeleteConfirmed` per offrire al metodo HttpPost una firma unica.</span><span class="sxs-lookup"><span data-stu-id="11522-260">The scaffolded code used the name `DeleteConfirmed` to give the HttpPost method a unique signature.</span></span> <span data-ttu-id="11522-261">(CLR richiede che i metodi di overload abbiano parametri di metodo diversi). Ora che le firme sono univoche, è possibile attenersi alla convenzione MVC e usare lo stesso nome per i metodi di eliminazione HttpPost e HttpGet.</span><span class="sxs-lookup"><span data-stu-id="11522-261">(The CLR requires overloaded methods to have different method parameters.) Now that the signatures are unique, you can stick with the MVC convention and use the same name for the HttpPost and HttpGet delete methods.</span></span>

<span data-ttu-id="11522-262">Se il reparto è già stato eliminato, il metodo `AnyAsync` restituisce false e l'applicazione torna al metodo Index.</span><span class="sxs-lookup"><span data-stu-id="11522-262">If the department is already deleted, the `AnyAsync` method returns false and the application just goes back to the Index method.</span></span>

<span data-ttu-id="11522-263">Se viene rilevato un errore di concorrenza, il codice visualizza nuovamente la pagina di conferma Delete (Elimina) e visualizza un flag indicante che è necessario visualizzare un messaggio di errore di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="11522-263">If a concurrency error is caught, the code redisplays the Delete confirmation page and provides a flag that indicates it should display a concurrency error message.</span></span>

### <a name="update-the-delete-view"></a><span data-ttu-id="11522-264">Aggiornare la pagina Delete</span><span class="sxs-lookup"><span data-stu-id="11522-264">Update the Delete view</span></span>

<span data-ttu-id="11522-265">In *Views/Departments/Delete.cshtml*, sostituire il codice sottoposto a scaffolding con il codice seguente, che aggiunge un campo messaggio di errore e campi nascosti per le proprietà DepartmentID e RowVersion.</span><span class="sxs-lookup"><span data-stu-id="11522-265">In *Views/Departments/Delete.cshtml*, replace the scaffolded code with the following code that adds an error message field and hidden fields for the DepartmentID and RowVersion properties.</span></span> <span data-ttu-id="11522-266">Le modifiche sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="11522-266">The changes are highlighted.</span></span>

[!code-cshtml[](intro/samples/cu/Views/Departments/Delete.cshtml?highlight=9,38,44,45,48)]

<span data-ttu-id="11522-267">Questa impostazione determina le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="11522-267">This makes the following changes:</span></span>

* <span data-ttu-id="11522-268">Aggiunge un messaggio di errore tra le intestazioni `h2` e `h3`.</span><span class="sxs-lookup"><span data-stu-id="11522-268">Adds an error message between the `h2` and `h3` headings.</span></span>

* <span data-ttu-id="11522-269">Sostituisce FirstMidName con FullName nel campo **Administrator** (Amministratore).</span><span class="sxs-lookup"><span data-stu-id="11522-269">Replaces FirstMidName with FullName in the **Administrator** field.</span></span>

* <span data-ttu-id="11522-270">Rimuove il campo RowVersion.</span><span class="sxs-lookup"><span data-stu-id="11522-270">Removes the RowVersion field.</span></span>

* <span data-ttu-id="11522-271">Aggiunge un campo nascosto per la proprietà `RowVersion`.</span><span class="sxs-lookup"><span data-stu-id="11522-271">Adds a hidden field for the `RowVersion` property.</span></span>

<span data-ttu-id="11522-272">Eseguire l'app e passare alla pagina Departments Index (Indice reparti).</span><span class="sxs-lookup"><span data-stu-id="11522-272">Run the app and go to the Departments Index page.</span></span> <span data-ttu-id="11522-273">Fare clic con il pulsante destro del mouse sul collegamento ipertestuale **Edit** (Modifica) per il reparto English (Inglese) e selezionare **Apri link in nuova scheda**, quindi fare clic sul collegamento ipertestuale **Edit** per il reparto English.</span><span class="sxs-lookup"><span data-stu-id="11522-273">Right-click the **Delete** hyperlink for the English department and select **Open in new tab**, then in the first tab click the **Edit** hyperlink for the English department.</span></span>

<span data-ttu-id="11522-274">Nella prima finestra modificare uno dei valori e fare clic su **Salva**:</span><span class="sxs-lookup"><span data-stu-id="11522-274">In the first window, change one of the values, and click **Save**:</span></span>

![Pagina Department Edit (Modifica - Reparto) dopo la modifica e prima dell'eliminazione](concurrency/_static/edit-after-change-for-delete.png)

<span data-ttu-id="11522-276">Nella seconda scheda fare clic su **Delete** (Elimina).</span><span class="sxs-lookup"><span data-stu-id="11522-276">In the second tab, click **Delete**.</span></span> <span data-ttu-id="11522-277">Viene visualizzato il messaggio di errore di concorrenza e i valori di Department (Reparto) vengono aggiornati con i dati attualmente presenti nel database.</span><span class="sxs-lookup"><span data-stu-id="11522-277">You see the concurrency error message, and the Department values are refreshed with what's currently in the database.</span></span>

![Pagina di conferma Department Delete (Elimina - Reparto) con errore di concorrenza](concurrency/_static/delete-error.png)

<span data-ttu-id="11522-279">Se si fa di nuovo clic su **Delete** (Elimina) viene visualizzata la pagina Index che indica che il reparto è stato eliminato.</span><span class="sxs-lookup"><span data-stu-id="11522-279">If you click **Delete** again, you're redirected to the Index page, which shows that the department has been deleted.</span></span>

## <a name="update-details-and-create-views"></a><span data-ttu-id="11522-280">Aggiornare le visualizzazioni Details (Dettagli) e Create (Crea)</span><span class="sxs-lookup"><span data-stu-id="11522-280">Update Details and Create views</span></span>

<span data-ttu-id="11522-281">Facoltativamente è possibile pulire il codice di scaffolding nelle visualizzazioni Details (Dettagli) e Create (Crea).</span><span class="sxs-lookup"><span data-stu-id="11522-281">You can optionally clean up scaffolded code in the Details and Create views.</span></span>

<span data-ttu-id="11522-282">Sostituire il codice in *Views/Departments/Details.cshtml* per eliminare la colonna RowVersion e visualizzare il nome completo di Administrator (Amministratore).</span><span class="sxs-lookup"><span data-stu-id="11522-282">Replace the code in *Views/Departments/Details.cshtml* to delete the RowVersion column and show the full name of the Administrator.</span></span>

[!code-cshtml[](intro/samples/cu/Views/Departments/Details.cshtml?highlight=35)]

<span data-ttu-id="11522-283">Sostituire il codice in *Views/Departments/Create.cshtml* per aggiungere all'elenco a discesa un'opzione Select (Seleziona).</span><span class="sxs-lookup"><span data-stu-id="11522-283">Replace the code in *Views/Departments/Create.cshtml* to add a Select option to the drop-down list.</span></span>

[!code-cshtml[](intro/samples/cu/Views/Departments/Create.cshtml?highlight=32-34)]

## <a name="get-the-code"></a><span data-ttu-id="11522-284">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="11522-284">Get the code</span></span>

[<span data-ttu-id="11522-285">Scaricare o visualizzare l'applicazione completata.</span><span class="sxs-lookup"><span data-stu-id="11522-285">Download or view the completed application.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a><span data-ttu-id="11522-286">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="11522-286">Additional resources</span></span>

 <span data-ttu-id="11522-287">Per altre informazioni su come gestire i conflitti di concorrenza in EF Core, vedere [Conflitti di concorrenza](/ef/core/saving/concurrency).</span><span class="sxs-lookup"><span data-stu-id="11522-287">For more information about how to handle concurrency in EF Core, see [Concurrency conflicts](/ef/core/saving/concurrency).</span></span>

## <a name="next-steps"></a><span data-ttu-id="11522-288">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="11522-288">Next steps</span></span>

<span data-ttu-id="11522-289">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="11522-289">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="11522-290">Scoprire di più sui conflitti di concorrenza</span><span class="sxs-lookup"><span data-stu-id="11522-290">Learned about concurrency conflicts</span></span>
> * <span data-ttu-id="11522-291">Aggiungere una proprietà di rilevamento modifiche</span><span class="sxs-lookup"><span data-stu-id="11522-291">Added a tracking property</span></span>
> * <span data-ttu-id="11522-292">Creare un controller e visualizzazioni Departments</span><span class="sxs-lookup"><span data-stu-id="11522-292">Created Departments controller and views</span></span>
> * <span data-ttu-id="11522-293">Aggiornare la visualizzazione Index</span><span class="sxs-lookup"><span data-stu-id="11522-293">Updated Index view</span></span>
> * <span data-ttu-id="11522-294">Aggiornare i metodi Edit</span><span class="sxs-lookup"><span data-stu-id="11522-294">Updated Edit methods</span></span>
> * <span data-ttu-id="11522-295">Aggiornare la visualizzazione Edit</span><span class="sxs-lookup"><span data-stu-id="11522-295">Updated Edit view</span></span>
> * <span data-ttu-id="11522-296">Testare i conflitti di concorrenza</span><span class="sxs-lookup"><span data-stu-id="11522-296">Tested concurrency conflicts</span></span>
> * <span data-ttu-id="11522-297">Aggiornamento della pagina Delete</span><span class="sxs-lookup"><span data-stu-id="11522-297">Updated the Delete page</span></span>
> * <span data-ttu-id="11522-298">Aggiornare le visualizzazioni Details e Create</span><span class="sxs-lookup"><span data-stu-id="11522-298">Updated Details and Create views</span></span>

<span data-ttu-id="11522-299">Passare all'esercitazione successiva per apprendere come implementare l'ereditarietà tabella per gerarchia per le entità Instructor e Student.</span><span class="sxs-lookup"><span data-stu-id="11522-299">Advance to the next tutorial to learn how to implement table-per-hierarchy inheritance for the Instructor and Student entities.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="11522-300">Passaggio successivo: implementare l'ereditarietà tabella per gerarchia</span><span class="sxs-lookup"><span data-stu-id="11522-300">Next: Implement table-per-hierarchy inheritance</span></span>](inheritance.md)
