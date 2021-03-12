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
# <a name="tutorial-create-a-complex-data-model---aspnet-mvc-with-ef-core"></a><span data-ttu-id="01faf-103">Esercitazione: creare un modello di dati complesso-ASP.NET MVC con EF Core</span><span class="sxs-lookup"><span data-stu-id="01faf-103">Tutorial: Create a complex data model - ASP.NET MVC with EF Core</span></span>

<span data-ttu-id="01faf-104">Nelle esercitazioni precedenti è stato usato un modello di dati semplice costituito da tre entità.</span><span class="sxs-lookup"><span data-stu-id="01faf-104">In the previous tutorials, you worked with a simple data model that was composed of three entities.</span></span> <span data-ttu-id="01faf-105">In questa esercitazione si aggiungeranno altre entità e relazioni e si personalizzerà il modello di dati specificando regole di formattazione, convalida e mapping del database.</span><span class="sxs-lookup"><span data-stu-id="01faf-105">In this tutorial, you'll add more entities and relationships and you'll customize the data model by specifying formatting, validation, and database mapping rules.</span></span>

<span data-ttu-id="01faf-106">Al termine dell'operazione le classi di entità verranno incluse nel modello di dati completato, illustrato nella figura seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-106">When you're finished, the entity classes will make up the completed data model that's shown in the following illustration:</span></span>

![Diagramma dell'entità](complex-data-model/_static/diagram.png)

<span data-ttu-id="01faf-108">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="01faf-108">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="01faf-109">Personalizzare il modello di dati</span><span class="sxs-lookup"><span data-stu-id="01faf-109">Customize the Data model</span></span>
> * <span data-ttu-id="01faf-110">Apportare modifiche all'entità Student</span><span class="sxs-lookup"><span data-stu-id="01faf-110">Make changes to Student entity</span></span>
> * <span data-ttu-id="01faf-111">Creare l'entità Instructor</span><span class="sxs-lookup"><span data-stu-id="01faf-111">Create Instructor entity</span></span>
> * <span data-ttu-id="01faf-112">Creare l'entità OfficeAssignment</span><span class="sxs-lookup"><span data-stu-id="01faf-112">Create OfficeAssignment entity</span></span>
> * <span data-ttu-id="01faf-113">Modificare l'entità Course</span><span class="sxs-lookup"><span data-stu-id="01faf-113">Modify Course entity</span></span>
> * <span data-ttu-id="01faf-114">Creare l'entità Department</span><span class="sxs-lookup"><span data-stu-id="01faf-114">Create Department entity</span></span>
> * <span data-ttu-id="01faf-115">Modificare l'entità Enrollment</span><span class="sxs-lookup"><span data-stu-id="01faf-115">Modify Enrollment entity</span></span>
> * <span data-ttu-id="01faf-116">Aggiornare il contesto di database</span><span class="sxs-lookup"><span data-stu-id="01faf-116">Update the database context</span></span>
> * <span data-ttu-id="01faf-117">Eseguire il seeding del database con dati di test</span><span class="sxs-lookup"><span data-stu-id="01faf-117">Seed database with test data</span></span>
> * <span data-ttu-id="01faf-118">Aggiungere una migrazione</span><span class="sxs-lookup"><span data-stu-id="01faf-118">Add a migration</span></span>
> * <span data-ttu-id="01faf-119">Modificare la stringa di connessione</span><span class="sxs-lookup"><span data-stu-id="01faf-119">Change the connection string</span></span>
> * <span data-ttu-id="01faf-120">Aggiornare il database</span><span class="sxs-lookup"><span data-stu-id="01faf-120">Update the database</span></span>

## <a name="prerequisites"></a><span data-ttu-id="01faf-121">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="01faf-121">Prerequisites</span></span>

* [<span data-ttu-id="01faf-122">Uso di migrazioni EF Core</span><span class="sxs-lookup"><span data-stu-id="01faf-122">Using EF Core migrations</span></span>](migrations.md)

## <a name="customize-the-data-model"></a><span data-ttu-id="01faf-123">Personalizzare il modello di dati</span><span class="sxs-lookup"><span data-stu-id="01faf-123">Customize the Data model</span></span>

<span data-ttu-id="01faf-124">In questa sezione si apprenderà come personalizzare il modello di dati usando attributi che specificano regole di formattazione, convalida e mapping del database.</span><span class="sxs-lookup"><span data-stu-id="01faf-124">In this section you'll see how to customize the data model by using attributes that specify formatting, validation, and database mapping rules.</span></span> <span data-ttu-id="01faf-125">Quindi nelle sezioni seguenti si creerà il modello di dati School (Istituto scolastico) completo, aggiungendo attributi alle classi già create e creando nuove classi per i tipi di entità rimanenti nel modello.</span><span class="sxs-lookup"><span data-stu-id="01faf-125">Then in several of the following sections you'll create the complete School data model by adding attributes to the classes you already created and creating new classes for the remaining entity types in the model.</span></span>

### <a name="the-datatype-attribute"></a><span data-ttu-id="01faf-126">Attributo DataType</span><span class="sxs-lookup"><span data-stu-id="01faf-126">The DataType attribute</span></span>

<span data-ttu-id="01faf-127">Per le date di iscrizione degli studenti, tutte le pagine Web attualmente visualizzano l'ora oltre alla data, anche se l'unico elemento rilevante di questo campo è la data.</span><span class="sxs-lookup"><span data-stu-id="01faf-127">For student enrollment dates, all of the web pages currently display the time along with the date, although all you care about for this field is the date.</span></span> <span data-ttu-id="01faf-128">Mediante gli attributi di annotazione dei dati è possibile modificare il codice per correggere il formato di visualizzazione in tutte le visualizzazioni che visualizzano i dati.</span><span class="sxs-lookup"><span data-stu-id="01faf-128">By using data annotation attributes, you can make one code change that will fix the display format in every view that shows the data.</span></span> <span data-ttu-id="01faf-129">Per un esempio di come eseguire questa operazione si aggiunge un attributo alla proprietà `EnrollmentDate` nella classe `Student`.</span><span class="sxs-lookup"><span data-stu-id="01faf-129">To see an example of how to do that, you'll add an attribute to the `EnrollmentDate` property in the `Student` class.</span></span>

<span data-ttu-id="01faf-130">In *Models/Student.cs* aggiungere un'istruzione `using` per lo spazio dei nomi `System.ComponentModel.DataAnnotations` e aggiungere gli attributi `DataType` e `DisplayFormat` alla proprietà `EnrollmentDate` come indicato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-130">In *Models/Student.cs*, add a `using` statement for the `System.ComponentModel.DataAnnotations` namespace and add `DataType` and `DisplayFormat` attributes to the `EnrollmentDate` property, as shown in the following example:</span></span>

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_DataType&highlight=3,12-13)]

<span data-ttu-id="01faf-131">L'attributo `DataType` viene usato per specificare un tipo di dati che è più specifico del tipo intrinseco del database.</span><span class="sxs-lookup"><span data-stu-id="01faf-131">The `DataType` attribute is used to specify a data type that's more specific than the database intrinsic type.</span></span> <span data-ttu-id="01faf-132">In questo caso si vuole tenere traccia solo della data e non di data e ora.</span><span class="sxs-lookup"><span data-stu-id="01faf-132">In this case we only want to keep track of the date, not the date and time.</span></span> <span data-ttu-id="01faf-133">L'enumerazione `DataType` include molti tipi di dati, ad esempio Date, Time, PhoneNumber, Currency, EmailAddress e così via.</span><span class="sxs-lookup"><span data-stu-id="01faf-133">The  `DataType` Enumeration provides for many data types, such as Date, Time, PhoneNumber, Currency, EmailAddress, and more.</span></span> <span data-ttu-id="01faf-134">L'attributo `DataType` può anche consentire all'applicazione di fornire automaticamente le funzionalità specifiche del tipo.</span><span class="sxs-lookup"><span data-stu-id="01faf-134">The `DataType` attribute can also enable the application to automatically provide type-specific features.</span></span> <span data-ttu-id="01faf-135">Ad esempio, è possibile creare un collegamento `mailto:` per `DataType.EmailAddress` e fornire un selettore data per `DataType.Date` nei browser che supportano HTML5.</span><span class="sxs-lookup"><span data-stu-id="01faf-135">For example, a `mailto:` link can be created for `DataType.EmailAddress`, and a date selector can be provided for `DataType.Date` in browsers that support HTML5.</span></span> <span data-ttu-id="01faf-136">L'attributo `DataType` produce attributi HTML5 `data-` supportati dai browser HTML5.</span><span class="sxs-lookup"><span data-stu-id="01faf-136">The `DataType` attribute emits HTML 5 `data-` (pronounced data dash) attributes that HTML 5 browsers can understand.</span></span> <span data-ttu-id="01faf-137">Gli attributi `DataType` non garantiscono alcuna convalida.</span><span class="sxs-lookup"><span data-stu-id="01faf-137">The `DataType` attributes don't provide any validation.</span></span>

<span data-ttu-id="01faf-138">`DataType.Date` non specifica il formato della data visualizzata.</span><span class="sxs-lookup"><span data-stu-id="01faf-138">`DataType.Date` doesn't specify the format of the date that's displayed.</span></span> <span data-ttu-id="01faf-139">Per impostazione predefinita il campo dati viene visualizzato in base ai formati predefiniti derivanti dal valore CultureInfo del server.</span><span class="sxs-lookup"><span data-stu-id="01faf-139">By default, the data field is displayed according to the default formats based on the server's CultureInfo.</span></span>

<span data-ttu-id="01faf-140">L'attributo `DisplayFormat` viene usato per specificare in modo esplicito il formato della data:</span><span class="sxs-lookup"><span data-stu-id="01faf-140">The `DisplayFormat` attribute is used to explicitly specify the date format:</span></span>

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

<span data-ttu-id="01faf-141">L'impostazione `ApplyFormatInEditMode` specifica che la formattazione deve essere applicata anche quando il valore viene visualizzato in una casella di testo per la modifica.</span><span class="sxs-lookup"><span data-stu-id="01faf-141">The `ApplyFormatInEditMode` setting specifies that the formatting should also be applied when the value is displayed in a text box for editing.</span></span> <span data-ttu-id="01faf-142">Per determinati campi questa scelta non è consigliabile. Ad esempio per i valori di valuta può non risultare opportuno che il simbolo di valuta sia incluso nella casella di testo per la modifica.</span><span class="sxs-lookup"><span data-stu-id="01faf-142">(You might not want that for some fields -- for example, for currency values, you might not want the currency symbol in the text box for editing.)</span></span>

<span data-ttu-id="01faf-143">È possibile usare l'attributo `DisplayFormat` da solo, ma in genere è consigliabile usare anche l'attributo `DataType`.</span><span class="sxs-lookup"><span data-stu-id="01faf-143">You can use the `DisplayFormat` attribute by itself, but it's generally a good idea to use the `DataType` attribute also.</span></span> <span data-ttu-id="01faf-144">L'attributo `DataType` esprime la semantica dei dati anziché la modalità di rendering dei dati in una schermata e offre i seguenti vantaggi che non si ottengono con `DisplayFormat`:</span><span class="sxs-lookup"><span data-stu-id="01faf-144">The `DataType` attribute conveys the semantics of the data as opposed to how to render it on a screen, and provides the following benefits that you don't get with `DisplayFormat`:</span></span>

* <span data-ttu-id="01faf-145">Il browser può abilitare le funzionalità HTML5, ad esempio per visualizzare un controllo di calendario, il simbolo della valuta appropriato per le impostazioni locali, i collegamenti alla posta elettronica, alcune istanze di convalida lato client e così via.</span><span class="sxs-lookup"><span data-stu-id="01faf-145">The browser can enable HTML5 features (for example to show a calendar control, the locale-appropriate currency symbol, email links, some client-side input validation, etc.).</span></span>

* <span data-ttu-id="01faf-146">Per impostazione predefinita, il browser eseguirà il rendering dei dati usando il formato corretto in base alle impostazioni locali del sistema.</span><span class="sxs-lookup"><span data-stu-id="01faf-146">By default, the browser will render data using the correct format based on your locale.</span></span>

<span data-ttu-id="01faf-147">Per ulteriori informazioni, vedere la [ \<input> documentazione dell'helper Tag](../../mvc/views/working-with-forms.md#the-input-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="01faf-147">For more information, see the [\<input> tag helper documentation](../../mvc/views/working-with-forms.md#the-input-tag-helper).</span></span>

<span data-ttu-id="01faf-148">Eseguire l'app, passare alla pagina Students Index (Indice studenti) e verificare che l'ora non viene più visualizzata nelle date di iscrizione.</span><span class="sxs-lookup"><span data-stu-id="01faf-148">Run the app, go to the Students Index page and notice that times are no longer displayed for the enrollment dates.</span></span> <span data-ttu-id="01faf-149">Lo stesso vale per tutte le viste che usano il modello Student (Studente).</span><span class="sxs-lookup"><span data-stu-id="01faf-149">The same will be true for any view that uses the Student model.</span></span>

![Pagina Students Index (Indice studenti) con date e senza ore](complex-data-model/_static/dates-no-times.png)

### <a name="the-stringlength-attribute"></a><span data-ttu-id="01faf-151">Attributo StringLength</span><span class="sxs-lookup"><span data-stu-id="01faf-151">The StringLength attribute</span></span>

<span data-ttu-id="01faf-152">È anche possibile specificare regole di convalida dei dati e messaggi di errore di convalida mediante gli attributi.</span><span class="sxs-lookup"><span data-stu-id="01faf-152">You can also specify data validation rules and validation error messages using attributes.</span></span> <span data-ttu-id="01faf-153">L'attributo `StringLength` imposta la lunghezza massima nel database e offre la convalida lato client e lato server per ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="01faf-153">The `StringLength` attribute sets the maximum length  in the database and provides client side and server side validation for ASP.NET Core MVC.</span></span> <span data-ttu-id="01faf-154">È anche possibile specificare la lunghezza minima della stringa in questo attributo, ma il valore minimo non ha alcun effetto sullo schema del database.</span><span class="sxs-lookup"><span data-stu-id="01faf-154">You can also specify the minimum string length in this attribute, but the minimum value has no impact on the database schema.</span></span>

<span data-ttu-id="01faf-155">Ad esempio si supponga di voler limitare a 50 il numero massimo di caratteri che gli utenti possono immettere per un nome.</span><span class="sxs-lookup"><span data-stu-id="01faf-155">Suppose you want to ensure that users don't enter more than 50 characters for a name.</span></span> <span data-ttu-id="01faf-156">Per aggiungere questa limitazione aggiungere attributi `StringLength` alle proprietà `LastName` e `FirstMidName`, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-156">To add this limitation, add `StringLength` attributes to the `LastName` and `FirstMidName` properties, as shown in the following example:</span></span>

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_StringLength&highlight=10,12)]

<span data-ttu-id="01faf-157">L'attributo `StringLength` non impedisce a un utente di immettere spazi vuoti per un nome.</span><span class="sxs-lookup"><span data-stu-id="01faf-157">The `StringLength` attribute won't prevent a user from entering white space for a name.</span></span> <span data-ttu-id="01faf-158">È possibile usare l'attributo `RegularExpression` per applicare restrizioni all'input.</span><span class="sxs-lookup"><span data-stu-id="01faf-158">You can use the `RegularExpression` attribute to apply restrictions to the input.</span></span> <span data-ttu-id="01faf-159">Ad esempio il codice seguente richiede che il primo carattere sia maiuscolo e i caratteri rimanenti siano caratteri alfabetici:</span><span class="sxs-lookup"><span data-stu-id="01faf-159">For example, the following code requires the first character to be upper case and the remaining characters to be alphabetical:</span></span>

```csharp
[RegularExpression(@"^[A-Z]+[a-zA-Z]*$")]
```

<span data-ttu-id="01faf-160">L'attributo `MaxLength` offre funzionalità simili a quelle dell'attributo `StringLength` ma non offre la convalida lato client.</span><span class="sxs-lookup"><span data-stu-id="01faf-160">The `MaxLength` attribute provides functionality similar to the `StringLength` attribute but doesn't provide client side validation.</span></span>

<span data-ttu-id="01faf-161">La modifica apportata al modello di database ora richiede un aggiornamento dello schema del database.</span><span class="sxs-lookup"><span data-stu-id="01faf-161">The database model has now changed in a way that requires a change in the database schema.</span></span> <span data-ttu-id="01faf-162">Si userà migrations per aggiornare lo schema senza perdere i dati eventualmente aggiunti al database tramite l'interfaccia utente dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-162">You'll use migrations to update the schema without losing any data that you may have added to the database by using the application UI.</span></span>

<span data-ttu-id="01faf-163">Salvare le modifiche e compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="01faf-163">Save your changes and build the project.</span></span> <span data-ttu-id="01faf-164">Quindi aprire una finestra dei comandi nella cartella di progetto ed eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="01faf-164">Then open the command window in the project folder and enter the following commands:</span></span>

```dotnetcli
dotnet ef migrations add MaxLengthOnNames
```

```dotnetcli
dotnet ef database update
```

<span data-ttu-id="01faf-165">Il comando `migrations add` segnala che può verificarsi la perdita di dati, perché la modifica riduce la lunghezza massima per due colonne.</span><span class="sxs-lookup"><span data-stu-id="01faf-165">The `migrations add` command warns that data loss may occur, because the change makes the maximum length shorter for two columns.</span></span>  <span data-ttu-id="01faf-166">Migrations crea un file denominato *\<timeStamp> _MaxLengthOnNames. cs*.</span><span class="sxs-lookup"><span data-stu-id="01faf-166">Migrations creates a file named *\<timeStamp>_MaxLengthOnNames.cs*.</span></span> <span data-ttu-id="01faf-167">Il metodo `Up` di questo file contiene codice che aggiorna il database per adattarlo al modello di dati corrente.</span><span class="sxs-lookup"><span data-stu-id="01faf-167">This file contains code in the `Up` method that will update the database to match the current data model.</span></span> <span data-ttu-id="01faf-168">Il comando `database update` ha eseguito tale codice.</span><span class="sxs-lookup"><span data-stu-id="01faf-168">The `database update` command ran that code.</span></span>

<span data-ttu-id="01faf-169">Il timestamp che precede il nome del file delle migrazioni viene usato da Entity Framework per ordinare le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="01faf-169">The timestamp prefixed to the migrations file name is used by Entity Framework to order the migrations.</span></span> <span data-ttu-id="01faf-170">È possibile creare più migrazioni prima di eseguire il comando update-database, dopodiché tutte le migrazioni vengono applicate nell'ordine in cui sono state create.</span><span class="sxs-lookup"><span data-stu-id="01faf-170">You can create multiple migrations before running the update-database command, and then all of the migrations are applied in the order in which they were created.</span></span>

<span data-ttu-id="01faf-171">Eseguire l'app, selezionare la scheda **Students** (Studenti), fare clic su **Crea nuovo** e provare a immettere un nome con più di 50 caratteri.</span><span class="sxs-lookup"><span data-stu-id="01faf-171">Run the app, select the **Students** tab, click **Create New**, and try to enter either name longer than 50 characters.</span></span> <span data-ttu-id="01faf-172">L'applicazione dovrebbe impedire questa operazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-172">The application should prevent you from doing this.</span></span> 

### <a name="the-column-attribute"></a><span data-ttu-id="01faf-173">Attributo Column</span><span class="sxs-lookup"><span data-stu-id="01faf-173">The Column attribute</span></span>

<span data-ttu-id="01faf-174">È possibile usare gli attributi anche per controllare il mapping delle classi e delle proprietà nel database.</span><span class="sxs-lookup"><span data-stu-id="01faf-174">You can also use attributes to control how your classes and properties are mapped to the database.</span></span> <span data-ttu-id="01faf-175">Si supponga di aver usato il nome `FirstMidName` per il campo first-name (Nome) perché il campo potrebbe contenere anche un secondo nome.</span><span class="sxs-lookup"><span data-stu-id="01faf-175">Suppose you had used the name `FirstMidName` for the first-name field because the field might also contain a middle name.</span></span> <span data-ttu-id="01faf-176">Tuttavia si vuole che la colonna di database sia denominata `FirstName`, perché gli utenti che scrivono query ad hoc per il database sono abituati a tale nome.</span><span class="sxs-lookup"><span data-stu-id="01faf-176">But you want the database column to be named `FirstName`, because users who will be writing ad-hoc queries against the database are accustomed to that name.</span></span> <span data-ttu-id="01faf-177">Per eseguire questo mapping è possibile usare l'attributo `Column`.</span><span class="sxs-lookup"><span data-stu-id="01faf-177">To make this mapping, you can use the `Column` attribute.</span></span>

<span data-ttu-id="01faf-178">L'attributo `Column` specifica che quando viene creato il database, la colonna della tabella `Student` mappata sulla proprietà `FirstMidName` verrà denominata `FirstName`.</span><span class="sxs-lookup"><span data-stu-id="01faf-178">The `Column` attribute specifies that when the database is created, the column of the `Student` table that maps to the `FirstMidName` property will be named `FirstName`.</span></span> <span data-ttu-id="01faf-179">In altri termini, quando il codice fa riferimento a `Student.FirstMidName` i dati provengono dalla colonna `FirstName` della tabella `Student` o vengono aggiornati in tale colonna.</span><span class="sxs-lookup"><span data-stu-id="01faf-179">In other words, when your code refers to `Student.FirstMidName`, the data will come from or be updated in the `FirstName` column of the `Student` table.</span></span> <span data-ttu-id="01faf-180">Se non si specificano nomi di colonna, le colonne avranno lo stesso nome della proprietà.</span><span class="sxs-lookup"><span data-stu-id="01faf-180">If you don't specify column names, they're given the same name as the property name.</span></span>

<span data-ttu-id="01faf-181">Nel file *Student.cs* aggiungere un'istruzione `using` per `System.ComponentModel.DataAnnotations.Schema` e aggiungere l'attributo del nome colonna alla proprietà `FirstMidName`, come illustrato nel codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-181">In the *Student.cs* file, add a `using` statement for `System.ComponentModel.DataAnnotations.Schema` and add the column name attribute to the `FirstMidName` property, as shown in the following highlighted code:</span></span>

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_Column&highlight=4,14)]

<span data-ttu-id="01faf-182">L'aggiunta dell'attributo `Column` modifica il modello che supporta `SchoolContext` e che pertanto non corrisponderà al database.</span><span class="sxs-lookup"><span data-stu-id="01faf-182">The addition of the `Column` attribute changes the model backing the `SchoolContext`, so it won't match the database.</span></span>

<span data-ttu-id="01faf-183">Salvare le modifiche e compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="01faf-183">Save your changes and build the project.</span></span> <span data-ttu-id="01faf-184">Quindi aprire una finestra dei comandi nella cartella di progetto ed eseguire i comandi seguenti per creare un'altra migrazione:</span><span class="sxs-lookup"><span data-stu-id="01faf-184">Then open the command window in the project folder and enter the following commands to create another migration:</span></span>

```dotnetcli
dotnet ef migrations add ColumnFirstName
```

```dotnetcli
dotnet ef database update
```

<span data-ttu-id="01faf-185">In **Esplora oggetti di SQL Server** aprire il designer della tabella **Student** (Studente) facendo doppio clic sulla tabella.</span><span class="sxs-lookup"><span data-stu-id="01faf-185">In **SQL Server Object Explorer**, open the Student table designer by double-clicking the **Student** table.</span></span>

![Tabella Students (Studenti) in SSOX dopo le migrazioni](complex-data-model/_static/ssox-after-migration.png)

<span data-ttu-id="01faf-187">Prima dell'applicazione delle prime due migrazioni, le colonne del nome erano di tipo nvarchar(MAX).</span><span class="sxs-lookup"><span data-stu-id="01faf-187">Before you applied the first two migrations, the name columns were of type nvarchar(MAX).</span></span> <span data-ttu-id="01faf-188">Ora sono di tipo nvarchar(50) e il nome colonna è cambiato da FirstMidName a FirstName.</span><span class="sxs-lookup"><span data-stu-id="01faf-188">They're now nvarchar(50) and the column name has changed from FirstMidName to FirstName.</span></span>

> [!Note]
> <span data-ttu-id="01faf-189">Se si prova a compilare prima di aver creato tutte le classi di entità delle sezioni seguenti, possono verificarsi errori di compilazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-189">If you try to compile before you finish creating all of the entity classes in the following sections, you might get compiler errors.</span></span>

## <a name="changes-to-student-entity"></a><span data-ttu-id="01faf-190">Modifiche all'entità Student</span><span class="sxs-lookup"><span data-stu-id="01faf-190">Changes to Student entity</span></span>

![Entità Student](complex-data-model/_static/student-entity.png)

<span data-ttu-id="01faf-192">In *Models/Student.cs* sostituire il codice aggiunto in precedenza con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="01faf-192">In *Models/Student.cs*, replace the code you added earlier with the following code.</span></span> <span data-ttu-id="01faf-193">Le modifiche sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="01faf-193">The changes are highlighted.</span></span>

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_BeforeInheritance&highlight=11,13,15,18,22,24-31)]

### <a name="the-required-attribute"></a><span data-ttu-id="01faf-194">Attributo Required</span><span class="sxs-lookup"><span data-stu-id="01faf-194">The Required attribute</span></span>

<span data-ttu-id="01faf-195">L'attributo `Required` rende obbligatori i campi delle proprietà del nome.</span><span class="sxs-lookup"><span data-stu-id="01faf-195">The `Required` attribute makes the name properties required fields.</span></span> <span data-ttu-id="01faf-196">L'attributo `Required` non è necessario per i tipi non nullable quali i tipi del valore (DateTime, int, double, float e così via).</span><span class="sxs-lookup"><span data-stu-id="01faf-196">The `Required` attribute isn't needed for non-nullable types such as value types (DateTime, int, double, float, etc.).</span></span> <span data-ttu-id="01faf-197">I tipi che non possono essere null vengono considerati automaticamente come campi obbligatori.</span><span class="sxs-lookup"><span data-stu-id="01faf-197">Types that can't be null are automatically treated as required fields.</span></span>

<span data-ttu-id="01faf-198">L'attributo `Required` deve essere usato con `MinimumLength` per l'applicazione di `MinimumLength`.</span><span class="sxs-lookup"><span data-stu-id="01faf-198">The `Required` attribute must be used with `MinimumLength` for the `MinimumLength` to be enforced.</span></span>

```csharp
[Display(Name = "Last Name")]
[Required]
[StringLength(50, MinimumLength=2)]
public string LastName { get; set; }
```

### <a name="the-display-attribute"></a><span data-ttu-id="01faf-199">Attributo Display</span><span class="sxs-lookup"><span data-stu-id="01faf-199">The Display attribute</span></span>

<span data-ttu-id="01faf-200">L'attributo `Display` specifica che la didascalia delle caselle di testo deve essere "First Name" (Nome), "Last Name" (Cognome), "Full Name" (Nome e cognome) ed "Enrollment Date" (Data di iscrizione) anziché il nome della proprietà (senza spazi tra le parole) in ogni istanza.</span><span class="sxs-lookup"><span data-stu-id="01faf-200">The `Display` attribute specifies that the caption for the text boxes should be "First Name", "Last Name", "Full Name", and "Enrollment Date" instead of the property name in each instance (which has no space dividing the words).</span></span>

### <a name="the-fullname-calculated-property"></a><span data-ttu-id="01faf-201">Proprietà calcolata FullName</span><span class="sxs-lookup"><span data-stu-id="01faf-201">The FullName calculated property</span></span>

<span data-ttu-id="01faf-202">`FullName` è una proprietà calcolata che restituisce un valore creato concatenando altre due proprietà.</span><span class="sxs-lookup"><span data-stu-id="01faf-202">`FullName` is a calculated property that returns a value that's created by concatenating two other properties.</span></span> <span data-ttu-id="01faf-203">Di conseguenza ha una sola funzione di accesso get e nel database non viene generata nessuna colonna `FullName`.</span><span class="sxs-lookup"><span data-stu-id="01faf-203">Therefore it has only a get accessor, and no `FullName` column will be generated in the database.</span></span>

## <a name="create-instructor-entity"></a><span data-ttu-id="01faf-204">Creare l'entità Instructor</span><span class="sxs-lookup"><span data-stu-id="01faf-204">Create Instructor entity</span></span>

![Entità Instructor](complex-data-model/_static/instructor-entity.png)

<span data-ttu-id="01faf-206">Creare *Models/Instructor.cs* sostituendo il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-206">Create *Models/Instructor.cs*, replacing the template code with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/Instructor.cs?name=snippet_BeforeInheritance)]

<span data-ttu-id="01faf-207">Si noti che molte proprietà sono uguali nelle entità Student e Instructor.</span><span class="sxs-lookup"><span data-stu-id="01faf-207">Notice that several properties are the same in the Student and Instructor entities.</span></span> <span data-ttu-id="01faf-208">Nell'esercitazione [Implementing Inheritance](inheritance.md) (Implementazione dell'ereditarietà) più avanti in questa serie si effettuerà il refactoring di questo codice per eliminare la ridondanza.</span><span class="sxs-lookup"><span data-stu-id="01faf-208">In the [Implementing Inheritance](inheritance.md) tutorial later in this series, you'll refactor this code to eliminate the redundancy.</span></span>

<span data-ttu-id="01faf-209">È possibile posizionare più attributi su una riga, pertanto è anche possibile scrivere gli attributi `HireDate` come indicato di seguito:</span><span class="sxs-lookup"><span data-stu-id="01faf-209">You can put multiple attributes on one line, so you could also write the `HireDate` attributes as follows:</span></span>

```csharp
[DataType(DataType.Date),Display(Name = "Hire Date"),DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

### <a name="the-courseassignments-and-officeassignment-navigation-properties"></a><span data-ttu-id="01faf-210">Proprietà di navigazione CourseAssignments e OfficeAssignment</span><span class="sxs-lookup"><span data-stu-id="01faf-210">The CourseAssignments and OfficeAssignment navigation properties</span></span>

<span data-ttu-id="01faf-211">Le proprietà `CourseAssignments` e `OfficeAssignment` sono proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-211">The `CourseAssignments` and `OfficeAssignment` properties are navigation properties.</span></span>

<span data-ttu-id="01faf-212">Un insegnante può tenere un numero qualsiasi di corsi, pertanto `CourseAssignments` è definita come raccolta.</span><span class="sxs-lookup"><span data-stu-id="01faf-212">An instructor can teach any number of courses, so `CourseAssignments` is defined as a collection.</span></span>

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

<span data-ttu-id="01faf-213">Se una proprietà di navigazione può contenere più entità, il tipo della proprietà deve essere un elenco in cui le voci possono essere aggiunte, eliminate e aggiornate.</span><span class="sxs-lookup"><span data-stu-id="01faf-213">If a navigation property can hold multiple entities, its type must be a list in which entries can be added, deleted, and updated.</span></span>  <span data-ttu-id="01faf-214">È possibile specificare `ICollection<T>` o un tipo come `List<T>` o `HashSet<T>`.</span><span class="sxs-lookup"><span data-stu-id="01faf-214">You can specify `ICollection<T>` or a type such as `List<T>` or `HashSet<T>`.</span></span> <span data-ttu-id="01faf-215">Se si specifica `ICollection<T>`, per impostazione predefinita EF crea una raccolta `HashSet<T>`.</span><span class="sxs-lookup"><span data-stu-id="01faf-215">If you specify `ICollection<T>`, EF creates a `HashSet<T>` collection by default.</span></span>

<span data-ttu-id="01faf-216">Il motivo per cui queste sono entità `CourseAssignment` viene illustrato più avanti nella sezione relativa alle relazioni molti-a-molti.</span><span class="sxs-lookup"><span data-stu-id="01faf-216">The reason why these are `CourseAssignment` entities is explained below in the section about many-to-many relationships.</span></span>

<span data-ttu-id="01faf-217">Le regole business di Contoso University specificano che un insegnante non può avere più di un ufficio, pertanto la proprietà `OfficeAssignment` contiene un'unica entità OfficeAssignment (che può essere null se non è assegnato alcun ufficio).</span><span class="sxs-lookup"><span data-stu-id="01faf-217">Contoso University business rules state that an instructor can only have at most one office, so the `OfficeAssignment` property holds a single OfficeAssignment entity (which may be null if no office is assigned).</span></span>

```csharp
public OfficeAssignment OfficeAssignment { get; set; }
```

## <a name="create-officeassignment-entity"></a><span data-ttu-id="01faf-218">Creare l'entità OfficeAssignment</span><span class="sxs-lookup"><span data-stu-id="01faf-218">Create OfficeAssignment entity</span></span>

![Entità OfficeAssignment](complex-data-model/_static/officeassignment-entity.png)

<span data-ttu-id="01faf-220">Creare *Models/OfficeAssignment.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-220">Create *Models/OfficeAssignment.cs* with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/OfficeAssignment.cs)]

### <a name="the-key-attribute"></a><span data-ttu-id="01faf-221">Attributo Key</span><span class="sxs-lookup"><span data-stu-id="01faf-221">The Key attribute</span></span>

<span data-ttu-id="01faf-222">Esiste una relazione uno-a-zero-o-uno tra le entità Instructor e OfficeAssignment.</span><span class="sxs-lookup"><span data-stu-id="01faf-222">There's a one-to-zero-or-one relationship  between the Instructor and the OfficeAssignment entities.</span></span> <span data-ttu-id="01faf-223">L'assegnazione dell'ufficio esiste solo in relazione all'insegnante al quale viene assegnato l'ufficio, pertanto la chiave primaria dell'assegnazione è anche la chiave esterna per l'entità Instructor.</span><span class="sxs-lookup"><span data-stu-id="01faf-223">An office assignment only exists in relation to the instructor it's assigned to, and therefore its primary key is also its foreign key to the Instructor entity.</span></span> <span data-ttu-id="01faf-224">Tuttavia Entity Framework non riconosce automaticamente InstructorID come chiave primaria di questa entità, perché il nome non rispetta la convenzione di denominazione ID o classnameID.</span><span class="sxs-lookup"><span data-stu-id="01faf-224">But the Entity Framework can't automatically recognize InstructorID as the primary key of this entity because its name doesn't follow the ID or classnameID naming convention.</span></span> <span data-ttu-id="01faf-225">Per identificare l'entità come chiave viene usato l'attributo `Key`:</span><span class="sxs-lookup"><span data-stu-id="01faf-225">Therefore, the `Key` attribute is used to identify it as the key:</span></span>

```csharp
[Key]
public int InstructorID { get; set; }
```

<span data-ttu-id="01faf-226">È anche possibile usare l'attributo `Key` se l'entità dispone di una chiave primaria propria ma si vuole assegnare alla proprietà un nome diverso da classnameID o ID.</span><span class="sxs-lookup"><span data-stu-id="01faf-226">You can also use the `Key` attribute if the entity does have its own primary key but you want to name the property something other than classnameID or ID.</span></span>

<span data-ttu-id="01faf-227">Per impostazione predefinita, EF considera la chiave come non generata dal database, perché la colonna è destinata a una relazione di identificazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-227">By default, EF treats the key as non-database-generated because the column is for an identifying relationship.</span></span>

### <a name="the-instructor-navigation-property"></a><span data-ttu-id="01faf-228">Proprietà di navigazione Instructor</span><span class="sxs-lookup"><span data-stu-id="01faf-228">The Instructor navigation property</span></span>

<span data-ttu-id="01faf-229">L'entità Instructor dispone di una proprietà di navigazione `OfficeAssignment` nullable (perché un docente potrebbe non avere un ufficio assegnato) e l'entità OfficeAssignment dispone di una proprietà di navigazione `Instructor` non nullable (perché un'assegnazione di ufficio non può esistere senza un insegnante: `InstructorID` è non nullable).</span><span class="sxs-lookup"><span data-stu-id="01faf-229">The Instructor entity has a nullable `OfficeAssignment` navigation property (because an instructor might not have an office assignment), and the OfficeAssignment entity has a non-nullable `Instructor` navigation property (because an office assignment can't exist without an instructor -- `InstructorID` is non-nullable).</span></span> <span data-ttu-id="01faf-230">Quando un'entità Instructor dispone di un'entità OfficeAssignment correlata, ogni entità include un riferimento all'altra entità nella relativa proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-230">When an Instructor entity has a related OfficeAssignment entity, each entity will have a reference to the other one in its navigation property.</span></span>

<span data-ttu-id="01faf-231">È possibile inserire un attributo `[Required]` nella proprietà di navigazione Instructor per specificare che deve essere presente un insegnante correlato, ma questa operazione non è necessaria perché la chiave esterna `InstructorID` (che è anche la chiave per questa tabella) è di tipo non nullable.</span><span class="sxs-lookup"><span data-stu-id="01faf-231">You could put a `[Required]` attribute on the Instructor navigation property to specify that there must be a related instructor, but you don't have to do that because the `InstructorID` foreign key (which is also the key to this table) is non-nullable.</span></span>

## <a name="modify-course-entity"></a><span data-ttu-id="01faf-232">Modificare l'entità Course</span><span class="sxs-lookup"><span data-stu-id="01faf-232">Modify Course entity</span></span>

![Entità Course](complex-data-model/_static/course-entity.png)

<span data-ttu-id="01faf-234">In *Models/Course.cs* sostituire il codice aggiunto in precedenza con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="01faf-234">In *Models/Course.cs*, replace the code you added earlier with the following code.</span></span> <span data-ttu-id="01faf-235">Le modifiche sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="01faf-235">The changes are highlighted.</span></span>

[!code-csharp[](intro/samples/cu/Models/Course.cs?name=snippet_Final&highlight=2,10,13,16,19,21,23)]

<span data-ttu-id="01faf-236">L'entità Course ha una proprietà di chiave esterna `DepartmentID` che fa riferimento all'entità Department correlata e include una proprietà di navigazione `Department`.</span><span class="sxs-lookup"><span data-stu-id="01faf-236">The course entity has a foreign key property `DepartmentID` which points to the related Department entity and it has a `Department` navigation property.</span></span>

<span data-ttu-id="01faf-237">In Entity Framework non è necessario aggiungere una proprietà di chiave esterna al modello di dati se è disponibile una proprietà di navigazione per un'entità correlata.</span><span class="sxs-lookup"><span data-stu-id="01faf-237">The Entity Framework doesn't require you to add a foreign key property to your data model when you have a navigation property for a related entity.</span></span>  <span data-ttu-id="01faf-238">EF crea automaticamente chiavi esterne nel database ogni volta che sono necessarie e crea [proprietà nascoste](/ef/core/modeling/shadow-properties) per tali chiavi.</span><span class="sxs-lookup"><span data-stu-id="01faf-238">EF automatically creates foreign keys in the database wherever they're needed and creates [shadow properties](/ef/core/modeling/shadow-properties) for them.</span></span> <span data-ttu-id="01faf-239">Tuttavia il fatto di avere la chiave esterna nel modello di dati può rendere più semplici ed efficienti gli aggiornamenti.</span><span class="sxs-lookup"><span data-stu-id="01faf-239">But having the foreign key in the data model can make updates simpler and more efficient.</span></span> <span data-ttu-id="01faf-240">Quando ad esempio si recupera un'entità Course per la modifica, l'entità Department è null se non viene caricata, pertanto prima di aggiornare l'entità Course è necessario recuperare l'entità Department.</span><span class="sxs-lookup"><span data-stu-id="01faf-240">For example, when you fetch a course entity to edit, the  Department entity is null if you don't load it, so when you update the course entity, you would have to first fetch the Department entity.</span></span> <span data-ttu-id="01faf-241">Quando la proprietà di chiave esterna `DepartmentID` viene inclusa nel modello di dati, non è necessario recuperare l'entità Department prima di eseguire l'aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="01faf-241">When the foreign key property `DepartmentID` is included in the data model, you don't need to fetch the Department entity before you update.</span></span>

### <a name="the-databasegenerated-attribute"></a><span data-ttu-id="01faf-242">Attributo DatabaseGenerated</span><span class="sxs-lookup"><span data-stu-id="01faf-242">The DatabaseGenerated attribute</span></span>

<span data-ttu-id="01faf-243">L'attributo `DatabaseGenerated` con il parametro `None` per la proprietà `CourseID` indica che i valori di chiave primaria vengono specificati dall'utente anziché essere generati dal database.</span><span class="sxs-lookup"><span data-stu-id="01faf-243">The `DatabaseGenerated` attribute with the `None` parameter on the `CourseID` property specifies that primary key values are provided by the user rather than generated by the database.</span></span>

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.None)]
[Display(Name = "Number")]
public int CourseID { get; set; }
```

<span data-ttu-id="01faf-244">Per impostazione predefinita, Entity Framework presuppone che i valori di chiave primaria vengano generati dal database.</span><span class="sxs-lookup"><span data-stu-id="01faf-244">By default, Entity Framework assumes that primary key values are generated by the database.</span></span> <span data-ttu-id="01faf-245">Questa è la condizione ottimale nella maggior parte degli scenari.</span><span class="sxs-lookup"><span data-stu-id="01faf-245">That's what you want in most scenarios.</span></span> <span data-ttu-id="01faf-246">Tuttavia per le entità Course si userà un numero di corso specificato dall'utente, ad esempio il numero di serie 1000 per un reparto, il numero di serie 2000 per un altro reparto e così via.</span><span class="sxs-lookup"><span data-stu-id="01faf-246">However, for Course entities, you'll use a user-specified course number such as a 1000 series for one department, a 2000 series for another department, and so on.</span></span>

<span data-ttu-id="01faf-247">Anche l'attributo `DatabaseGenerated` può essere usato anche per generare valori predefiniti, come nel caso delle colonne di database usate per registrare la data di creazione o aggiornamento di una riga.</span><span class="sxs-lookup"><span data-stu-id="01faf-247">The `DatabaseGenerated` attribute can also be used to generate default values, as in the case of database columns used to record the date a row was created or updated.</span></span>  <span data-ttu-id="01faf-248">Per altre informazioni, vedere [Generated Properties](/ef/core/modeling/generated-properties) (Proprietà generate).</span><span class="sxs-lookup"><span data-stu-id="01faf-248">For more information, see [Generated Properties](/ef/core/modeling/generated-properties).</span></span>

### <a name="foreign-key-and-navigation-properties"></a><span data-ttu-id="01faf-249">Proprietà chiave esterna e di navigazione</span><span class="sxs-lookup"><span data-stu-id="01faf-249">Foreign key and navigation properties</span></span>

<span data-ttu-id="01faf-250">La proprietà di chiave esterna e le proprietà di navigazione nell'entità Course riflettono le relazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="01faf-250">The foreign key properties and navigation properties in the Course entity reflect the following relationships:</span></span>

<span data-ttu-id="01faf-251">Un corso viene assegnato a un solo reparto, pertanto sono presenti una chiave esterna `DepartmentID` e una proprietà di navigazione `Department` per i motivi indicati in precedenza.</span><span class="sxs-lookup"><span data-stu-id="01faf-251">A course is assigned to one department, so there's a `DepartmentID` foreign key and a `Department` navigation property for the reasons mentioned above.</span></span>

```csharp
public int DepartmentID { get; set; }
public Department Department { get; set; }
```

<span data-ttu-id="01faf-252">Un corso può avere un numero qualsiasi di studenti iscritti, pertanto la proprietà di navigazione `Enrollments` è una raccolta:</span><span class="sxs-lookup"><span data-stu-id="01faf-252">A course can have any number of students enrolled in it, so the `Enrollments` navigation property is a collection:</span></span>

```csharp
public ICollection<Enrollment> Enrollments { get; set; }
```

<span data-ttu-id="01faf-253">Un corso può essere impartito da più insegnanti, pertanto la proprietà di navigazione `CourseAssignments` è una raccolta (il tipo `CourseAssignment` è illustrato [più avanti](#many-to-many-relationships)):</span><span class="sxs-lookup"><span data-stu-id="01faf-253">A course may be taught by multiple instructors, so the `CourseAssignments` navigation property is a collection (the type `CourseAssignment` is explained [later](#many-to-many-relationships)):</span></span>

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

## <a name="create-department-entity"></a><span data-ttu-id="01faf-254">Creare l'entità Department</span><span class="sxs-lookup"><span data-stu-id="01faf-254">Create Department entity</span></span>

![Entità reparto](complex-data-model/_static/department-entity.png)

<span data-ttu-id="01faf-256">Creare *Models/Department.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-256">Create *Models/Department.cs* with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Begin)]

### <a name="the-column-attribute"></a><span data-ttu-id="01faf-257">Attributo Column</span><span class="sxs-lookup"><span data-stu-id="01faf-257">The Column attribute</span></span>

<span data-ttu-id="01faf-258">In precedenza l'attributo `Column` è stato usato per modificare il mapping del nome di colonna.</span><span class="sxs-lookup"><span data-stu-id="01faf-258">Earlier you used the `Column` attribute to change column name mapping.</span></span> <span data-ttu-id="01faf-259">Nel codice dell'entità Department l'attributo `Column` viene usato per modificare il mapping del tipo di dati SQL e per far sì che la colonna venga definita usando il tipo SQL Server money nel database:</span><span class="sxs-lookup"><span data-stu-id="01faf-259">In the code for the Department entity, the `Column` attribute is being used to change SQL data type mapping so that the column will be defined using the SQL Server money type in the database:</span></span>

```csharp
[Column(TypeName="money")]
public decimal Budget { get; set; }
```

<span data-ttu-id="01faf-260">In genere il mapping della colonna non è necessario, perché Entity Framework sceglie il tipo di dati SQL Server appropriato in base al tipo CLR definito per la proprietà.</span><span class="sxs-lookup"><span data-stu-id="01faf-260">Column mapping is generally not required, because the Entity Framework chooses the appropriate SQL Server data type based on the CLR type that you define for the property.</span></span> <span data-ttu-id="01faf-261">Il tipo CLR `decimal` esegue il mapping a un tipo SQL Server `decimal`.</span><span class="sxs-lookup"><span data-stu-id="01faf-261">The CLR `decimal` type maps to a SQL Server `decimal` type.</span></span> <span data-ttu-id="01faf-262">In questo caso tuttavia si sa che la colonna includerà importi in valuta, pertanto il tipo di dati money è più appropriato.</span><span class="sxs-lookup"><span data-stu-id="01faf-262">But in this case you know that the column will be holding currency amounts, and the money data type is more appropriate for that.</span></span>

### <a name="foreign-key-and-navigation-properties"></a><span data-ttu-id="01faf-263">Proprietà chiave esterna e di navigazione</span><span class="sxs-lookup"><span data-stu-id="01faf-263">Foreign key and navigation properties</span></span>

<span data-ttu-id="01faf-264">Le proprietà di chiave esterna e le proprietà di navigazione riflettono le relazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="01faf-264">The foreign key and navigation properties reflect the following relationships:</span></span>

<span data-ttu-id="01faf-265">Un reparto può avere o meno un amministratore e un amministratore è sempre un insegnante.</span><span class="sxs-lookup"><span data-stu-id="01faf-265">A department may or may not have an administrator, and an administrator is always an instructor.</span></span> <span data-ttu-id="01faf-266">Di conseguenza la proprietà `InstructorID` è inclusa come chiave esterna per l'entità Instructor e dopo l'indicazione del tipo `int` viene aggiunto un punto interrogativo, per contrassegnare la proprietà come nullable.</span><span class="sxs-lookup"><span data-stu-id="01faf-266">Therefore the `InstructorID` property is included as the foreign key to the Instructor entity, and a question mark is added after the `int` type designation to mark the property as nullable.</span></span> <span data-ttu-id="01faf-267">La proprietà di navigazione è denominata `Administrator` ma contiene un'entità Instructor:</span><span class="sxs-lookup"><span data-stu-id="01faf-267">The navigation property is named `Administrator` but holds an Instructor entity:</span></span>

```csharp
public int? InstructorID { get; set; }
public Instructor Administrator { get; set; }
```

<span data-ttu-id="01faf-268">Un reparto può avere molti corsi, pertanto è disponibile una proprietà di navigazione Courses:</span><span class="sxs-lookup"><span data-stu-id="01faf-268">A department may have many courses, so there's a Courses navigation property:</span></span>

```csharp
public ICollection<Course> Courses { get; set; }
```

> [!NOTE]
> <span data-ttu-id="01faf-269">Per convenzione, Entity Framework consente l'eliminazione a catena per le chiavi esterne non nullable e per le relazioni molti-a-molti.</span><span class="sxs-lookup"><span data-stu-id="01faf-269">By convention, the Entity Framework enables cascade delete for non-nullable foreign keys and for many-to-many relationships.</span></span> <span data-ttu-id="01faf-270">Ciò può determinare regole di eliminazione a catena circolari, che generano un'eccezione quando si prova ad aggiungere una migrazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-270">This can result in circular cascade delete rules, which will cause an exception when you try to add a migration.</span></span> <span data-ttu-id="01faf-271">Ad esempio, se la proprietà Department.InstructorID non è stata definita come nullable, EF configura una regola di eliminazione a catena per eliminare il reparto quando si elimina l'insegnante, che non è il risultato desiderato.</span><span class="sxs-lookup"><span data-stu-id="01faf-271">For example, if you didn't define the Department.InstructorID property as nullable, EF would configure a cascade delete rule to delete the department when you delete the instructor, which isn't what you want to have happen.</span></span> <span data-ttu-id="01faf-272">Se le regole business richiedono che la proprietà `InstructorID` sia non nullable, è necessario usare la seguente istruzione API Fluent per disattivare l'eliminazione a catena nella relazione:</span><span class="sxs-lookup"><span data-stu-id="01faf-272">If your business rules required the `InstructorID` property to be non-nullable, you would have to use the following fluent API statement to disable cascade delete on the relationship:</span></span>
>
> ```csharp
> modelBuilder.Entity<Department>()
>    .HasOne(d => d.Administrator)
>    .WithMany()
>    .OnDelete(DeleteBehavior.Restrict)
> ```

## <a name="modify-enrollment-entity"></a><span data-ttu-id="01faf-273">Modificare l'entità Enrollment</span><span class="sxs-lookup"><span data-stu-id="01faf-273">Modify Enrollment entity</span></span>

![Entità Enrollment](complex-data-model/_static/enrollment-entity.png)

<span data-ttu-id="01faf-275">In *Models/Enrollment.cs* sostituire il codice aggiunto in precedenza con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-275">In *Models/Enrollment.cs*, replace the code you added earlier with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/Enrollment.cs?name=snippet_Final&highlight=1-2,16)]

### <a name="foreign-key-and-navigation-properties"></a><span data-ttu-id="01faf-276">Proprietà chiave esterna e di navigazione</span><span class="sxs-lookup"><span data-stu-id="01faf-276">Foreign key and navigation properties</span></span>

<span data-ttu-id="01faf-277">Le proprietà di chiave esterna e le proprietà di navigazione riflettono le relazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="01faf-277">The foreign key properties and navigation properties reflect the following relationships:</span></span>

<span data-ttu-id="01faf-278">Un record di iscrizione è relativo a un singolo corso, pertanto sono presenti una proprietà di chiave esterna `CourseID` e una proprietà di navigazione `Course`:</span><span class="sxs-lookup"><span data-stu-id="01faf-278">An enrollment record is for a single course, so there's a `CourseID` foreign key property and a `Course` navigation property:</span></span>

```csharp
public int CourseID { get; set; }
public Course Course { get; set; }
```

<span data-ttu-id="01faf-279">Un record di iscrizione è relativo a un singolo studente, pertanto sono presenti una proprietà di chiave esterna `StudentID` e una proprietà di navigazione `Student`:</span><span class="sxs-lookup"><span data-stu-id="01faf-279">An enrollment record is for a single student, so there's a `StudentID` foreign key property and a `Student` navigation property:</span></span>

```csharp
public int StudentID { get; set; }
public Student Student { get; set; }
```

## <a name="many-to-many-relationships"></a><span data-ttu-id="01faf-280">Relazioni molti-a-molti</span><span class="sxs-lookup"><span data-stu-id="01faf-280">Many-to-Many relationships</span></span>

<span data-ttu-id="01faf-281">È presente una relazione molti-a-molti tra le entità Student e Course e l'entità Enrollment funziona come una tabella di join molti-a-molti *con payload* nel database.</span><span class="sxs-lookup"><span data-stu-id="01faf-281">There's a many-to-many relationship between the Student and Course entities, and the Enrollment entity functions as a many-to-many join table *with payload* in the database.</span></span> <span data-ttu-id="01faf-282">"Con payload" significa che la tabella Enrollment contiene dati aggiuntivi oltre alle chiavi esterne delle tabelle di join (in questo caso una chiave primaria e una proprietà Grade).</span><span class="sxs-lookup"><span data-stu-id="01faf-282">"With payload" means that the Enrollment table contains additional data besides foreign keys for the joined tables (in this case, a primary key and a Grade property).</span></span>

<span data-ttu-id="01faf-283">La figura seguente illustra l'aspetto di queste relazioni in un diagramma di entità.</span><span class="sxs-lookup"><span data-stu-id="01faf-283">The following illustration shows what these relationships look like in an entity diagram.</span></span> <span data-ttu-id="01faf-284">Il diagramma è stato generato con Entity Framework Power Tools per EF 6.x. La creazione del diagramma non fa parte dell'esercitazione e il diagramma viene visualizzato solo come esempio.</span><span class="sxs-lookup"><span data-stu-id="01faf-284">(This diagram was generated using the Entity Framework Power Tools for EF 6.x; creating the diagram isn't part of the tutorial, it's just being used here as an illustration.)</span></span>

![Relazione molti-a-molti Student-Course (Studente-Corso)](complex-data-model/_static/student-course.png)

<span data-ttu-id="01faf-286">Ogni riga della relazione inizia con un 1 e termina con un asterisco (\*), per indicare una relazione uno-a-molti.</span><span class="sxs-lookup"><span data-stu-id="01faf-286">Each relationship line has a 1 at one end and an asterisk (\*) at the other, indicating a one-to-many relationship.</span></span>

<span data-ttu-id="01faf-287">Se la tabella Enrollment (Iscrizione) non include informazioni sul livello è sufficiente che contenga le due chiavi esterne CourseID e StudentID.</span><span class="sxs-lookup"><span data-stu-id="01faf-287">If the Enrollment table didn't include grade information, it would only need to contain the two foreign keys CourseID and StudentID.</span></span> <span data-ttu-id="01faf-288">In questo caso sarebbe una tabella di join molti-a-molti senza payload (o tabella di join pura) del database.</span><span class="sxs-lookup"><span data-stu-id="01faf-288">In that case, it would be a many-to-many join table without payload (or a pure join table) in the database.</span></span> <span data-ttu-id="01faf-289">Le entità Instructor e Course dispongono di questo tipo di relazione molti-a-molti e il passaggio successivo consiste nel creare una classe di entità che faccia da tabella di join senza payload.</span><span class="sxs-lookup"><span data-stu-id="01faf-289">The Instructor and Course entities have that kind of many-to-many relationship, and your next step is to create an entity class to function as a join table without payload.</span></span>

<span data-ttu-id="01faf-290">Le tabelle di join implicite per le relazioni molti-a-molti sono supportate in EF 6.x, ma non in EF Core.</span><span class="sxs-lookup"><span data-stu-id="01faf-290">(EF 6.x supports implicit join tables for many-to-many relationships, but EF Core doesn't.</span></span> <span data-ttu-id="01faf-291">Per altre informazioni, vedere l'[approfondimento nel repository EF Core di GitHub repository](https://github.com/aspnet/EntityFramework/issues/1368).</span><span class="sxs-lookup"><span data-stu-id="01faf-291">For more information, see the [discussion in the EF Core GitHub repository](https://github.com/aspnet/EntityFramework/issues/1368).)</span></span>

## <a name="the-courseassignment-entity"></a><span data-ttu-id="01faf-292">Entità CourseAssignment</span><span class="sxs-lookup"><span data-stu-id="01faf-292">The CourseAssignment entity</span></span>

![Entità CourseAssignment](complex-data-model/_static/courseassignment-entity.png)

<span data-ttu-id="01faf-294">Creare *Models/CourseAssignment.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="01faf-294">Create *Models/CourseAssignment.cs* with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/CourseAssignment.cs)]

### <a name="join-entity-names"></a><span data-ttu-id="01faf-295">Nomi delle entità di join</span><span class="sxs-lookup"><span data-stu-id="01faf-295">Join entity names</span></span>

<span data-ttu-id="01faf-296">È necessaria una tabella di join del database per la relazione molti-a-molti Instructor-Courses e tale relazione deve essere rappresentata da un set di entità.</span><span class="sxs-lookup"><span data-stu-id="01faf-296">A join table is required in the database for the Instructor-to-Courses many-to-many relationship, and it has to be represented by an entity set.</span></span> <span data-ttu-id="01faf-297">È pratica comune assegnare a un'entità di join il nome `EntityName1EntityName2`, che in questo caso sarebbe `CourseInstructor`.</span><span class="sxs-lookup"><span data-stu-id="01faf-297">It's common to name a join entity `EntityName1EntityName2`, which in this case would be `CourseInstructor`.</span></span> <span data-ttu-id="01faf-298">È tuttavia consigliabile scegliere un nome che descrive la relazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-298">However, we recommend that you choose a name that describes the relationship.</span></span> <span data-ttu-id="01faf-299">I modelli di dati sono inizialmente semplici, quindi crescono e diventano più complessi. In molti casi ai join senza payload vengono assegnati payload in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="01faf-299">Data models start out simple and grow, with no-payload joins frequently getting payloads later.</span></span> <span data-ttu-id="01faf-300">Se si inizia con un nome di entità descrittivo, non sarà necessario modificarlo successivamente.</span><span class="sxs-lookup"><span data-stu-id="01faf-300">If you start with a descriptive entity name, you won't have to change the name later.</span></span> <span data-ttu-id="01faf-301">Idealmente l'entità di join dovrebbe avere il proprio nome naturale (se possibile composto da un'unica parola) nel dominio di business.</span><span class="sxs-lookup"><span data-stu-id="01faf-301">Ideally, the join entity would have its own natural (possibly single word) name in the business domain.</span></span> <span data-ttu-id="01faf-302">Ad esempio Books (Documentazione) e Customers (Clienti) potrebbero essere collegati mediante Ratings (Valutazioni).</span><span class="sxs-lookup"><span data-stu-id="01faf-302">For example, Books and Customers could be linked through Ratings.</span></span> <span data-ttu-id="01faf-303">Per questa relazione `CourseAssignment` è una soluzione migliore rispetto a `CourseInstructor`.</span><span class="sxs-lookup"><span data-stu-id="01faf-303">For this relationship, `CourseAssignment` is a better choice than `CourseInstructor`.</span></span>

### <a name="composite-key"></a><span data-ttu-id="01faf-304">Chiave composta</span><span class="sxs-lookup"><span data-stu-id="01faf-304">Composite key</span></span>

<span data-ttu-id="01faf-305">Dato che le chiavi esterne non sono nullable e insieme identificano in modo univoco ogni riga della tabella, non è necessario avere una chiave primaria separata.</span><span class="sxs-lookup"><span data-stu-id="01faf-305">Since the foreign keys are not nullable and together uniquely identify each row of the table, there's no need for a separate primary key.</span></span> <span data-ttu-id="01faf-306">Le proprietà *InstructorID* e *CourseID* funzionano come una chiave primaria composta.</span><span class="sxs-lookup"><span data-stu-id="01faf-306">The *InstructorID* and *CourseID* properties should function as a composite primary key.</span></span> <span data-ttu-id="01faf-307">L'unico modo per identificare le chiavi primarie composte in Entity Framework è l'uso di *API Fluent* (l'operazione non può essere eseguita con gli attributi).</span><span class="sxs-lookup"><span data-stu-id="01faf-307">The only way to identify composite primary keys to EF is by using the *fluent API* (it can't be done by using attributes).</span></span> <span data-ttu-id="01faf-308">Nella sezione successiva si vedrà come configurare la chiave primaria composta.</span><span class="sxs-lookup"><span data-stu-id="01faf-308">You'll see how to configure the composite primary key in the next section.</span></span>

<span data-ttu-id="01faf-309">La chiave composta garantisce che anche se è possibile avere più righe per un corso e più righe per un insegnante, non è possibile avere più righe per lo stesso insegnante e lo stesso corso.</span><span class="sxs-lookup"><span data-stu-id="01faf-309">The composite key ensures that while you can have multiple rows for one course, and multiple rows for one instructor, you can't have multiple rows for the same instructor and course.</span></span> <span data-ttu-id="01faf-310">L'entità di join `Enrollment` definisce la propria chiave primaria, pertanto sono possibili i duplicati di questo tipo.</span><span class="sxs-lookup"><span data-stu-id="01faf-310">The `Enrollment` join entity defines its own primary key, so duplicates of this sort are possible.</span></span> <span data-ttu-id="01faf-311">Per evitare tali duplicati è possibile aggiungere un indice univoco ai campi chiave esterna o configurare `Enrollment` con una chiave primaria composta simile a `CourseAssignment`.</span><span class="sxs-lookup"><span data-stu-id="01faf-311">To prevent such duplicates, you could add a unique index on the foreign key fields, or configure `Enrollment` with a primary composite key similar to `CourseAssignment`.</span></span> <span data-ttu-id="01faf-312">Per altre informazioni, vedere [Indexes](/ef/core/modeling/indexes) (Indici).</span><span class="sxs-lookup"><span data-stu-id="01faf-312">For more information, see [Indexes](/ef/core/modeling/indexes).</span></span>

## <a name="update-the-database-context"></a><span data-ttu-id="01faf-313">Aggiornare il contesto di database</span><span class="sxs-lookup"><span data-stu-id="01faf-313">Update the database context</span></span>

<span data-ttu-id="01faf-314">Aggiungere il codice evidenziato di seguito al file *Data/SchoolContext.cs*:</span><span class="sxs-lookup"><span data-stu-id="01faf-314">Add the following highlighted code to the *Data/SchoolContext.cs* file:</span></span>

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_BeforeInheritance&highlight=15-18,25-31)]

<span data-ttu-id="01faf-315">Questo codice aggiunge le nuove entità e configura la chiave primaria composta dell'entità CourseAssignment.</span><span class="sxs-lookup"><span data-stu-id="01faf-315">This code adds the new entities and configures the CourseAssignment entity's composite primary key.</span></span>

## <a name="about-a-fluent-api-alternative"></a><span data-ttu-id="01faf-316">Informazioni su un'alternativa API Fluent</span><span class="sxs-lookup"><span data-stu-id="01faf-316">About a fluent API alternative</span></span>

<span data-ttu-id="01faf-317">Il codice nel metodo `OnModelCreating` della classe `DbContext` usa l'*API Fluent* per configurare il comportamento di Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="01faf-317">The code in the `OnModelCreating` method of the `DbContext` class uses the *fluent API* to configure EF behavior.</span></span> <span data-ttu-id="01faf-318">L'API è denominata "API Fluent" perché viene spesso usata unendo una serie di chiamate di metodi in un'unica istruzione, come in questo esempio tratto dalla [documentazione di EF Core](/ef/core/modeling/#use-fluent-api-to-configure-a-model):</span><span class="sxs-lookup"><span data-stu-id="01faf-318">The API is called "fluent" because it's often used by stringing a series of method calls together into a single statement, as in this example from the [EF Core documentation](/ef/core/modeling/#use-fluent-api-to-configure-a-model):</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

<span data-ttu-id="01faf-319">In questa esercitazione si usa l'API Fluent solo per operazioni di mapping del database non eseguibili con gli attributi.</span><span class="sxs-lookup"><span data-stu-id="01faf-319">In this tutorial, you're using the fluent API only for database mapping that you can't do with attributes.</span></span> <span data-ttu-id="01faf-320">È tuttavia possibile usare l'API Fluent anche per specificare la maggior parte delle regole di formattazione, convalida e mapping specificabili tramite gli attributi.</span><span class="sxs-lookup"><span data-stu-id="01faf-320">However, you can also use the fluent API to specify most of the formatting, validation, and mapping rules that you can do by using attributes.</span></span> <span data-ttu-id="01faf-321">Alcuni attributi quali `MinimumLength` non possono essere applicati con l'API Fluent.</span><span class="sxs-lookup"><span data-stu-id="01faf-321">Some attributes such as `MinimumLength` can't be applied with the fluent API.</span></span> <span data-ttu-id="01faf-322">Come accennato in precedenza, `MinimumLength` non modifica lo schema, ma si limita ad applicare una regola di convalida lato client e lato server.</span><span class="sxs-lookup"><span data-stu-id="01faf-322">As mentioned previously, `MinimumLength` doesn't change the schema, it only applies a client and server side validation rule.</span></span>

<span data-ttu-id="01faf-323">Alcuni sviluppatori preferiscono usare solo l'API Fluent, per dare un aspetto "ordinato" alle classi di entità.</span><span class="sxs-lookup"><span data-stu-id="01faf-323">Some developers prefer to use the fluent API exclusively so that they can keep their entity classes "clean."</span></span> <span data-ttu-id="01faf-324">È possibile usare un misto di attributi e API Fluent e alcune personalizzazioni sono eseguibili solo mediante l'API Fluent, ma in generale è consigliabile scegliere uno dei due approcci e usarlo in modo il più possibile coerente.</span><span class="sxs-lookup"><span data-stu-id="01faf-324">You can mix attributes and fluent API if you want, and there are a few customizations that can only be done by using fluent API, but in general the recommended practice is to choose one of these two approaches and use that consistently as much as possible.</span></span> <span data-ttu-id="01faf-325">Se si usano entrambi gli approcci, tenere presente che ogni volta che si verifica un conflitto l'API Fluent esegue l'override degli attributi.</span><span class="sxs-lookup"><span data-stu-id="01faf-325">If you do use both, note that wherever there's a conflict, Fluent API overrides attributes.</span></span>

<span data-ttu-id="01faf-326">Per altre informazioni sul confronto tra attributi e API Fluent, vedere [Metodi di configurazione](/ef/core/modeling/).</span><span class="sxs-lookup"><span data-stu-id="01faf-326">For more information about attributes vs. fluent API, see [Methods of configuration](/ef/core/modeling/).</span></span>

## <a name="entity-diagram-showing-relationships"></a><span data-ttu-id="01faf-327">Diagramma dell'entità che visualizza le relazioni</span><span class="sxs-lookup"><span data-stu-id="01faf-327">Entity Diagram Showing Relationships</span></span>

<span data-ttu-id="01faf-328">La figura seguente visualizza il diagramma creato da Entity Framework Power Tools per il modello School completato.</span><span class="sxs-lookup"><span data-stu-id="01faf-328">The following illustration shows the diagram that the Entity Framework Power Tools create for the completed School model.</span></span>

![Diagramma dell'entità](complex-data-model/_static/diagram.png)

<span data-ttu-id="01faf-330">Oltre alle linee delle relazioni uno-a-molti (da 1 a \*) è possibile visualizzare la linea della relazione uno-a-zero-o-uno (da 1 a 0..1) tra le entità Instructor e OfficeAssignment e la linea della relazione zero-o-uno-a-molti (da 0..1 a \*) tra le entità Instructor e Department.</span><span class="sxs-lookup"><span data-stu-id="01faf-330">Besides the one-to-many relationship lines (1 to \*), you can see here the one-to-zero-or-one relationship line (1 to 0..1) between the Instructor and OfficeAssignment entities and the zero-or-one-to-many relationship line (0..1 to \*) between the Instructor and Department entities.</span></span>

## <a name="seed-database-with-test-data"></a><span data-ttu-id="01faf-331">Eseguire il seeding del database con dati di test</span><span class="sxs-lookup"><span data-stu-id="01faf-331">Seed database with test data</span></span>

<span data-ttu-id="01faf-332">Sostituire il codice nel file *Data/DbInitializer.cs* con il codice seguente per includere dati di inizializzazione per le nuove entità create.</span><span class="sxs-lookup"><span data-stu-id="01faf-332">Replace the code in the *Data/DbInitializer.cs* file with the following code in order to provide seed data for the new entities you've created.</span></span>

[!code-csharp[](intro/samples/cu/Data/DbInitializer.cs?name=snippet_Final)]

<span data-ttu-id="01faf-333">Come si è visto nella prima esercitazione, la maggior parte di questo codice crea semplicemente nuovi oggetti entità e carica i dati di esempio nelle proprietà in base alle esigenze di test.</span><span class="sxs-lookup"><span data-stu-id="01faf-333">As you saw in the first tutorial, most of this code simply creates new entity objects and loads sample data into properties as required for testing.</span></span> <span data-ttu-id="01faf-334">Osservare la modalità di gestione delle relazioni molti-a-molti: il codice crea relazioni tramite la creazione di entità nei set di entità di join `Enrollments` e `CourseAssignment`.</span><span class="sxs-lookup"><span data-stu-id="01faf-334">Notice how the many-to-many relationships are handled: the code creates relationships by creating entities in the `Enrollments` and `CourseAssignment` join entity sets.</span></span>

## <a name="add-a-migration"></a><span data-ttu-id="01faf-335">Aggiungere una migrazione</span><span class="sxs-lookup"><span data-stu-id="01faf-335">Add a migration</span></span>

<span data-ttu-id="01faf-336">Salvare le modifiche e compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="01faf-336">Save your changes and build the project.</span></span> <span data-ttu-id="01faf-337">Quindi aprire la finestra di comando nella cartella del progetto e immettere il comando `migrations add` (non eseguire ancora il comando update-database):</span><span class="sxs-lookup"><span data-stu-id="01faf-337">Then open the command window in the project folder and enter the `migrations add` command (don't do the update-database command yet):</span></span>

```dotnetcli
dotnet ef migrations add ComplexDataModel
```

<span data-ttu-id="01faf-338">Viene visualizzato un avviso sulla possibile perdita di dati.</span><span class="sxs-lookup"><span data-stu-id="01faf-338">You get a warning about possible data loss.</span></span>

```text
An operation was scaffolded that may result in the loss of data. Please review the migration for accuracy.
Done. To undo this action, use 'ef migrations remove'
```

<span data-ttu-id="01faf-339">Se si prova a eseguire il comando `database update` in questa fase (evitare di farlo), si ottiene il seguente errore:</span><span class="sxs-lookup"><span data-stu-id="01faf-339">If you tried to run the `database update` command at this point (don't do it yet), you would get the following error:</span></span>

> <span data-ttu-id="01faf-340">The ALTER TABLE statement conflicted with the FOREIGN KEY constraint "FK_dbo.Course_dbo.Department_DepartmentID".</span><span class="sxs-lookup"><span data-stu-id="01faf-340">The ALTER TABLE statement conflicted with the FOREIGN KEY constraint "FK_dbo.Course_dbo.Department_DepartmentID".</span></span> <span data-ttu-id="01faf-341">The conflict occurred in database "ContosoUniversity", table "dbo.Department", column 'DepartmentID' (L'istruzione ALTER TABLE è in conflitto con il vincolo FOREIGN KEY "FK_dbo.Course_dbo.Department_DepartmentID". Il conflitto si è verificato nella colonna 'DepartmentID' della tabella "dbo.Department"del database "ContosoUniversity").</span><span class="sxs-lookup"><span data-stu-id="01faf-341">The conflict occurred in database "ContosoUniversity", table "dbo.Department", column 'DepartmentID'.</span></span>

<span data-ttu-id="01faf-342">In determinati casi, quando si eseguono migrazioni con dati esistenti è necessario inserire dati stub nel database per soddisfare i vincoli di chiave esterna.</span><span class="sxs-lookup"><span data-stu-id="01faf-342">Sometimes when you execute migrations with existing data, you need to insert stub data into the database to satisfy foreign key constraints.</span></span> <span data-ttu-id="01faf-343">Il codice generato nel metodo `Up` aggiunge una chiave esterna DepartmentID alla tabella Course.</span><span class="sxs-lookup"><span data-stu-id="01faf-343">The generated code in the `Up` method adds a non-nullable DepartmentID foreign key to the Course table.</span></span> <span data-ttu-id="01faf-344">Se sono già presenti righe nella tabella Course quando viene eseguito il codice, l'operazione `AddColumn` non riesce perché SQL Server non determina il valore da inserire nella colonna, che non può essere null.</span><span class="sxs-lookup"><span data-stu-id="01faf-344">If there are already rows in the Course table when the code runs, the `AddColumn` operation fails because SQL Server doesn't know what value to put in the column that can't be null.</span></span> <span data-ttu-id="01faf-345">In questa esercitazione si esegue la migrazione in un nuovo database, ma in un'applicazione di produzione è necessario che la migrazione gestisca i dati esistenti. Le istruzioni che seguono visualizzano un esempio di esecuzione di questa operazione.</span><span class="sxs-lookup"><span data-stu-id="01faf-345">For this tutorial you'll run the migration on a new database, but in a production application you'd have to make the migration handle existing data, so the following directions show an example of how to do that.</span></span>

<span data-ttu-id="01faf-346">Per fare in modo che questa migrazione funzioni con i dati esistenti, è necessario modificare il codice per dare alla nuova colonna un valore predefinito e creare un reparto stub denominato "Temp" che svolga la funzione di reparto predefinito.</span><span class="sxs-lookup"><span data-stu-id="01faf-346">To make this migration work with existing data you have to change the code to give the new column a default value, and create a stub department named "Temp" to act as the default department.</span></span> <span data-ttu-id="01faf-347">Di conseguenza, dopo l'esecuzione del metodo `Up` tutte le righe Course esistenti saranno correlate al reparto "Temp".</span><span class="sxs-lookup"><span data-stu-id="01faf-347">As a result, existing Course rows will all be related to the "Temp" department after the `Up` method runs.</span></span>

* <span data-ttu-id="01faf-348">Aprire il file *{timestamp}_ComplexDataModel.cs*.</span><span class="sxs-lookup"><span data-stu-id="01faf-348">Open the *{timestamp}_ComplexDataModel.cs* file.</span></span>

* <span data-ttu-id="01faf-349">Impostare come commento la riga di codice che aggiunge la colonna DepartmentID alla tabella Course.</span><span class="sxs-lookup"><span data-stu-id="01faf-349">Comment out the line of code that adds the DepartmentID column to the Course table.</span></span>

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CommentOut&highlight=9-13)]

* <span data-ttu-id="01faf-350">Aggiungere il codice evidenziato seguente dopo il codice che crea la tabella Department:</span><span class="sxs-lookup"><span data-stu-id="01faf-350">Add the following highlighted code after the code that creates the Department table:</span></span>

  [!code-csharp[](intro/samples/cu/Migrations/20170215234014_ComplexDataModel.cs?name=snippet_CreateDefaultValue&highlight=22-32)]

<span data-ttu-id="01faf-351">In un'applicazione di produzione è necessario creare codice o script per aggiungere righe Department e associare le righe Course alle nuove righe Department.</span><span class="sxs-lookup"><span data-stu-id="01faf-351">In a production application, you would write code or scripts to add Department rows and relate Course rows to the new Department rows.</span></span> <span data-ttu-id="01faf-352">In tal modo il reparto "Temp" o il valore predefinito nella colonna Course.DepartmentID non saranno più necessari.</span><span class="sxs-lookup"><span data-stu-id="01faf-352">You would then no longer need the "Temp" department or the default value on the Course.DepartmentID column.</span></span>

<span data-ttu-id="01faf-353">Salvare le modifiche e compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="01faf-353">Save your changes and build the project.</span></span>

## <a name="change-the-connection-string"></a><span data-ttu-id="01faf-354">Modificare la stringa di connessione</span><span class="sxs-lookup"><span data-stu-id="01faf-354">Change the connection string</span></span>

<span data-ttu-id="01faf-355">Ora la classe `DbInitializer` include nuovo codice che aggiunge dati di inizializzazione per le nuove entità a un database vuoto.</span><span class="sxs-lookup"><span data-stu-id="01faf-355">You now have new code in the `DbInitializer` class that adds seed data for the new entities to an empty database.</span></span> <span data-ttu-id="01faf-356">Per fare in modo che EF crei un nuovo database vuoto, modificare il nome del database nella stringa di connessione in *appsettings.json* ContosoUniversity3 o un altro nome che non è stato usato nel computer che si sta usando.</span><span class="sxs-lookup"><span data-stu-id="01faf-356">To make EF create a new empty database, change the name of the database in the connection string in *appsettings.json* to ContosoUniversity3 or some other name that you haven't used on the computer you're using.</span></span>

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity3;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
```

<span data-ttu-id="01faf-357">Salvare la modifica in *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="01faf-357">Save your change to *appsettings.json*.</span></span>

> [!NOTE]
> <span data-ttu-id="01faf-358">In alternativa alla modifica del nome del database, è possibile eliminare il database.</span><span class="sxs-lookup"><span data-stu-id="01faf-358">As an alternative to changing the database name, you can delete the database.</span></span> <span data-ttu-id="01faf-359">Usare **Esplora oggetti di SQL Server** (SSOX) o il comando CLI `database drop`:</span><span class="sxs-lookup"><span data-stu-id="01faf-359">Use **SQL Server Object Explorer** (SSOX) or the `database drop` CLI command:</span></span>
>
> ```dotnetcli
> dotnet ef database drop
> ```

## <a name="update-the-database"></a><span data-ttu-id="01faf-360">Aggiornare il database</span><span class="sxs-lookup"><span data-stu-id="01faf-360">Update the database</span></span>

<span data-ttu-id="01faf-361">Dopo aver modificato il nome del database o eliminato il database, eseguire il comando `database update` nella finestra di comando per eseguire le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="01faf-361">After you have changed the database name or deleted the database, run the `database update` command in the command window to execute the migrations.</span></span>

```dotnetcli
dotnet ef database update
```

<span data-ttu-id="01faf-362">Eseguire l'app per far sì che il metodo `DbInitializer.Initialize` venga eseguito e popoli il nuovo database.</span><span class="sxs-lookup"><span data-stu-id="01faf-362">Run the app to cause the `DbInitializer.Initialize` method to run and populate the new database.</span></span>

<span data-ttu-id="01faf-363">Aprire il database in SSOX come in precedenza, quindi espandere il nodo **Tabelle** per visualizzare tutte le tabelle che sono state create.</span><span class="sxs-lookup"><span data-stu-id="01faf-363">Open the database in SSOX as you did earlier, and expand the **Tables** node to see that all of the tables have been created.</span></span> <span data-ttu-id="01faf-364">Se SSOX è ancora aperto dall'operazione precedente, fare clic sul pulsante **Aggiorna**.</span><span class="sxs-lookup"><span data-stu-id="01faf-364">(If you still have SSOX open from the earlier time, click the **Refresh** button.)</span></span>

![Tabelle in SSOX](complex-data-model/_static/ssox-tables.png)

<span data-ttu-id="01faf-366">Eseguire l'app per attivare il codice inizializzatore che esegue l'inizializzazione del database.</span><span class="sxs-lookup"><span data-stu-id="01faf-366">Run the app to trigger the initializer code that seeds the database.</span></span>

<span data-ttu-id="01faf-367">Fare clic con il pulsante destro del mouse sulla tabella **CourseAssignment** e selezionare **Visualizza dati** per verificare che la tabella contenga dati.</span><span class="sxs-lookup"><span data-stu-id="01faf-367">Right-click the **CourseAssignment** table and select **View Data** to verify that it has data in it.</span></span>

![Dati CourseAssignment in SSOX](complex-data-model/_static/ssox-ci-data.png)

## <a name="get-the-code"></a><span data-ttu-id="01faf-369">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="01faf-369">Get the code</span></span>

[<span data-ttu-id="01faf-370">Scaricare o visualizzare l'applicazione completata.</span><span class="sxs-lookup"><span data-stu-id="01faf-370">Download or view the completed application.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a><span data-ttu-id="01faf-371">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="01faf-371">Next steps</span></span>

<span data-ttu-id="01faf-372">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="01faf-372">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="01faf-373">Personalizzazione del modello di dati</span><span class="sxs-lookup"><span data-stu-id="01faf-373">Customized the Data model</span></span>
> * <span data-ttu-id="01faf-374">Modifica dell'entità Student</span><span class="sxs-lookup"><span data-stu-id="01faf-374">Made changes to Student entity</span></span>
> * <span data-ttu-id="01faf-375">Creazione dell'entità Instructor</span><span class="sxs-lookup"><span data-stu-id="01faf-375">Created Instructor entity</span></span>
> * <span data-ttu-id="01faf-376">Creazione dell'entità OfficeAssignment</span><span class="sxs-lookup"><span data-stu-id="01faf-376">Created OfficeAssignment entity</span></span>
> * <span data-ttu-id="01faf-377">Modifica dell'entità Course</span><span class="sxs-lookup"><span data-stu-id="01faf-377">Modified Course entity</span></span>
> * <span data-ttu-id="01faf-378">Creazione dell'entità Department</span><span class="sxs-lookup"><span data-stu-id="01faf-378">Created Department entity</span></span>
> * <span data-ttu-id="01faf-379">Modifica dell'entità Enrollment</span><span class="sxs-lookup"><span data-stu-id="01faf-379">Modified Enrollment entity</span></span>
> * <span data-ttu-id="01faf-380">Aggiornamento del contesto di database</span><span class="sxs-lookup"><span data-stu-id="01faf-380">Updated the database context</span></span>
> * <span data-ttu-id="01faf-381">Seeding del database con dati di test</span><span class="sxs-lookup"><span data-stu-id="01faf-381">Seeded database with test data</span></span>
> * <span data-ttu-id="01faf-382">Aggiunta di una migrazione</span><span class="sxs-lookup"><span data-stu-id="01faf-382">Added a migration</span></span>
> * <span data-ttu-id="01faf-383">Modifica della stringa di connessione</span><span class="sxs-lookup"><span data-stu-id="01faf-383">Changed the connection string</span></span>
> * <span data-ttu-id="01faf-384">Aggiornamento del database</span><span class="sxs-lookup"><span data-stu-id="01faf-384">Updated the database</span></span>

<span data-ttu-id="01faf-385">Passare all'esercitazione successiva per altre informazioni su come accedere ai dati correlati.</span><span class="sxs-lookup"><span data-stu-id="01faf-385">Advance to the next tutorial to learn more about how to access related data.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="01faf-386">Passaggio successivo: accedere ai dati correlati</span><span class="sxs-lookup"><span data-stu-id="01faf-386">Next: Access related data</span></span>](read-related-data.md)
