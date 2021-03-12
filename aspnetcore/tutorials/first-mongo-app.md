---
title: Creare un'API Web con ASP.NET Core e MongoDB
author: prkhandelwal
description: Questa esercitazione illustra come creare un'API Web ASP.NET Core con un database NoSQL MongoDB.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc, seodec18
ms.date: 08/17/2019
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
uid: tutorials/first-mongo-app
ms.openlocfilehash: 489a15f2011e44ffd9f9bfe7d5410b7d79a10632
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587606"
---
# <a name="create-a-web-api-with-aspnet-core-and-mongodb"></a>Creare un'API Web con ASP.NET Core e MongoDB

Di [Pratik Khandelwal](https://twitter.com/K2Prk) e [Scott Addie](https://twitter.com/Scott_Addie)

::: moniker range=">= aspnetcore-3.0"

Questa esercitazione crea un'API Web che esegue operazioni di creazione, lettura, aggiornamento ed eliminazione (CRUD) su un database NoSQL [MongoDB](https://www.mongodb.com/what-is-mongodb).

In questa esercitazione verranno illustrate le procedure per:

> [!div class="checklist"]
> * Configurare MongoDB
> * Creare un database MongoDB
> * Definire una raccolta e uno schema MongoDB
> * Eseguire operazioni CRUD di MongoDB da un'API Web
> * Personalizzare la serializzazione JSON

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/first-mongo-app/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* [.NET Core SDK 3.0 o versione successiva](https://dotnet.microsoft.com/download/dotnet-core)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**
* [MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [.NET Core SDK 3.0 o versione successiva](https://dotnet.microsoft.com/download/dotnet-core)
* [Visual Studio Code](https://code.visualstudio.com/download)
* [C# per Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [MongoDB](https://docs.mongodb.com/manual/administration/install-community/)

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* [.NET Core SDK 3.0 o versione successiva](https://dotnet.microsoft.com/download/dotnet-core)
* [Visual Studio per Mac versione 7.7 o successiva](https://visualstudio.microsoft.com/downloads/)
* [MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)

---

## <a name="configure-mongodb"></a>Configurare MongoDB

Se si usa Windows, MongoDB viene installato in *C: \\ programmi \\ MongoDB* per impostazione predefinita. Aggiungere *C: \\ Program Files \\ MongoDB \\ server \\ \<version_number> \\ bin* alla `Path` variabile di ambiente. Questa modifica consente l'accesso MongoDB da qualsiasi posizione nel computer di sviluppo.

Usare la shell mongo nelle procedure seguenti per creare un database, creare le raccolte e archiviare i documenti. Per altre informazioni sui comandi della shell mongo, vedere [Working with the mongo Shell](https://docs.mongodb.com/manual/mongo/#working-with-the-mongo-shell) (Utilizzo della shell mongo).

1. Scegliere una directory nel computer di sviluppo per archiviare i dati. Ad esempio, *C: \\ BooksData* in Windows. Creare la directory se non esiste. La shell mongo non consente di creare nuove directory.
1. Aprire una shell dei comandi. Eseguire il comando seguente per connettersi a MongoDB sulla porta predefinita 27017. Ricordare di sostituire `<data_directory_path>` con la directory scelta nel passaggio precedente.

   ```console
   mongod --dbpath <data_directory_path>
   ```

1. Aprire un'altra istanza della shell dei comandi. Connettersi al database di test predefinito eseguendo il comando seguente:

   ```console
   mongo
   ```

1. Eseguire il comando seguente in una shell dei comandi:

   ```console
   use BookstoreDb
   ```

   Se non esiste già, viene creato un database denominato *BookstoreDb*. Se il database esiste, la connessione viene aperta per le transazioni.

1. Creare una raccolta `Books` tramite il comando seguente:

   ```console
   db.createCollection('Books')
   ```

   Viene visualizzato il risultato seguente:

   ```console
   { "ok" : 1 }
   ```

1. Definire uno schema per la raccolta `Books` e inserire due documenti usando il comando seguente:

   ```console
   db.Books.insertMany([{'Name':'Design Patterns','Price':54.93,'Category':'Computers','Author':'Ralph Johnson'}, {'Name':'Clean Code','Price':43.15,'Category':'Computers','Author':'Robert C. Martin'}])
   ```

   Viene visualizzato il risultato seguente:

   ```console
   {
     "acknowledged" : true,
     "insertedIds" : [
       ObjectId("5bfd996f7b8e48dc15ff215d"),
       ObjectId("5bfd996f7b8e48dc15ff215e")
     ]
   }
   ```
  
   > [!NOTE]
   > L'ID illustrato in questo articolo non corrisponde agli ID quando si esegue questo campione.

1. Visualizzare i documenti nel database usando il comando seguente:

   ```console
   db.Books.find({}).pretty()
   ```

   Viene visualizzato il risultato seguente:

   ```console
   {
     "_id" : ObjectId("5bfd996f7b8e48dc15ff215d"),
     "Name" : "Design Patterns",
     "Price" : 54.93,
     "Category" : "Computers",
     "Author" : "Ralph Johnson"
   }
   {
     "_id" : ObjectId("5bfd996f7b8e48dc15ff215e"),
     "Name" : "Clean Code",
     "Price" : 43.15,
     "Category" : "Computers",
     "Author" : "Robert C. Martin"
   }
   ```

   Lo schema aggiunge una proprietà `_id` generata automaticamente di tipo `ObjectId` per ogni documento.

Il database è pronto. È possibile iniziare a creare l'API Web ASP.NET Core.

## <a name="create-the-aspnet-core-web-api-project"></a>Creare il progetto per l'API Web ASP.NET Core

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Passare a **file** > **nuovo** > **progetto**.
1. Selezionare il tipo di progetto **Applicazione Web ASP.NET Core** e selezionare **Avanti**.
1. Assegnare al progetto il nome *BooksApi* e selezionare **Crea**.
1. Selezionare il framework di destinazione **.NET Core** e **ASP.NET Core 3.0**. Selezionare il modello di progetto **API** e scegliere **Crea**.
1. Visitare la [raccolta NuGet: MongoDB. driver](https://www.nuget.org/packages/MongoDB.Driver/) per determinare la versione stabile più recente del driver .NET per MongoDB. Nella finestra **Console di Gestione pacchetti** passare alla radice del progetto. Eseguire il comando seguente per installare il driver .NET per MongoDB:

   ```powershell
   Install-Package MongoDB.Driver -Version {VERSION}
   ```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. Eseguire i comandi seguenti in una shell dei comandi:

   ```dotnetcli
   dotnet new webapi -o BooksApi
   code BooksApi
   ```

   Un nuovo progetto API Web ASP.NET Core destinato a .NET Core viene generato e aperto in Visual Studio Code.

1. Dopo che l'icona di OmniSharp Flame della barra di stato è verde, una finestra di dialogo richiede che le **risorse necessarie per la compilazione e il debug siano mancanti da' BooksApi '. Aggiungerli?** Selezionare **Sì**.
1. Visitare la [raccolta NuGet: MongoDB. driver](https://www.nuget.org/packages/MongoDB.Driver/) per determinare la versione stabile più recente del driver .NET per MongoDB. Aprire **Terminale integrato** e passare alla radice del progetto. Eseguire il comando seguente per installare il driver .NET per MongoDB:

   ```dotnetcli
   dotnet add BooksApi.csproj package MongoDB.Driver -v {VERSION}
   ```

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. In Visual Studio per Mac precedente alla versione 8,6, selezionare **file**  >  **nuova soluzione**  >  **.NET Core**  >  **app** dall'intestazione laterale. Nella versione 8,6 o successive selezionare **file**  >  **nuova soluzione**  >  **Web e**  >  **app** console dalla barra laterale.
1. Selezionare il  > modello di progetto **API** ASP.NET Core C# e fare clic su **Avanti**.
1. Selezionare **.NET Core 3,1** dall'elenco a discesa **Framework di destinazione** e quindi fare clic su **Avanti**.
1. Immettere *BooksApi* per **Nome progetto** e selezionare **Crea**.
1. Nel riquadro **Soluzione** fare clic con il pulsante destro del mouse sul nodo **Dipendenze** del progetto e scegliere **Aggiungi pacchetti**.
1. Immettere *MongoDB.Driver* nella casella di ricerca, selezionare il pacchetto *MongoDB.Driver* e quindi **Aggiungi pacchetto**.
1. Selezionare il pulsante **Accetta** nella finestra di dialogo **Accettazione della licenza**.

---

## <a name="add-an-entity-model"></a>Aggiungere un modello di entità

1. Aggiungere una directory *Models* alla radice del progetto.
1. Aggiungere una classe `Book` alla directory *Models* con il codice seguente:

   ```csharp
   using MongoDB.Bson;
   using MongoDB.Bson.Serialization.Attributes;

   namespace BooksApi.Models
   {
       public class Book
       {
           [BsonId]
           [BsonRepresentation(BsonType.ObjectId)]
           public string Id { get; set; }

           [BsonElement("Name")]
           public string BookName { get; set; }

           public decimal Price { get; set; }

           public string Category { get; set; }

           public string Author { get; set; }
       }
   }
   ```

   Nella classe precedente, la proprietà `Id`:

   * È obbligatoria per il mapping tra l'oggetto CLR (Common Language Runtime) e la raccolta MongoDB.
   * Viene annotato con [`[BsonId]`](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_Serialization_Attributes_BsonIdAttribute.htm) per indicare questa proprietà come chiave primaria del documento.
   * Viene annotato con [`[BsonRepresentation(BsonType.ObjectId)]`](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_Serialization_Attributes_BsonRepresentationAttribute.htm) per consentire il passaggio del parametro come tipo `string` anziché come una struttura [ObjectID](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_ObjectId.htm) . Mongo gestisce la conversione da `string` a `ObjectId`.

   La `BookName` proprietà è annotata con l' [`[BsonElement]`](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_Serialization_Attributes_BsonElementAttribute.htm) attributo. Il valore dell'attributo `Name` rappresenta il nome della proprietà nella raccolta MongoDB.

## <a name="add-a-configuration-model"></a>Aggiungere un modello di configurazione

1. Aggiungere i valori di configurazione del database seguenti a *appsettings.json* :

   [!code-json[](first-mongo-app/samples/3.x/SampleApp/appsettings.json?highlight=2-6)]

1. Aggiungere un file *BookstoreDatabaseSettings.cs* alla directory *Models* con il codice seguente:

   [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Models/BookstoreDatabaseSettings.cs)]

   La `BookstoreDatabaseSettings` classe precedente viene utilizzata per archiviare i *appsettings.json* valori delle `BookstoreDatabaseSettings` proprietà del file. I nomi delle proprietà JSON e C# sono identici per semplificare il processo di mapping.

1. Aggiungere il codice evidenziato seguente a `Startup.ConfigureServices`:

   [!code-csharp[](first-mongo-app/samples_snapshot/3.x/SampleApp/Startup.ConfigureServices.AddDbSettings.cs?highlight=3-8)]

   Nel codice precedente:

   * L'istanza di configurazione a cui *appsettings.json* `BookstoreDatabaseSettings` viene associata la sezione del file è registrata nel contenitore di inserimento delle dipendenze. Ad esempio, la `BookstoreDatabaseSettings` proprietà di un oggetto `ConnectionString` viene popolata con la `BookstoreDatabaseSettings:ConnectionString` Proprietà in *appsettings.json* .
   * L'interfaccia `IBookstoreDatabaseSettings` è registrata nell'inserimento di dipendenze con una [durata del servizio](xref:fundamentals/dependency-injection#service-lifetimes) singleton. Quando avviene l'inserimento, l'istanza dell'interfaccia restituisce un oggetto `BookstoreDatabaseSettings`.

1. Aggiungere il codice seguente all'inizio del file *Startup.cs* per risolvere i riferimenti a `BookstoreDatabaseSettings` e `IBookstoreDatabaseSettings`:

   [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Startup.cs?name=snippet_UsingBooksApiModels)]

## <a name="add-a-crud-operations-service"></a>Aggiungere un servizio di operazioni CRUD

1. Aggiungere una directory *Services* alla radice del progetto.
1. Aggiungere una classe `BookService` alla directory *Services* con il codice seguente:

   [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Services/BookService.cs?name=snippet_BookServiceClass)]

   Nel codice precedente un'istanza di `IBookstoreDatabaseSettings` viene recuperata dall'inserimento di dipendenze tramite l'inserimento del costruttore. Questa tecnica consente di accedere ai *appsettings.json* valori di configurazione aggiunti nella sezione [aggiungere un modello di configurazione](#add-a-configuration-model) .

1. Aggiungere il codice evidenziato seguente a `Startup.ConfigureServices`:

   [!code-csharp[](first-mongo-app/samples_snapshot/3.x/SampleApp/Startup.ConfigureServices.AddSingletonService.cs?highlight=9)]

   Nel codice precedente la classe `BookService` è registrata con l'inserimento di dipendenze per supportare l'inserimento del costruttore nelle classi che la utilizzano. La durata del servizio singleton è più appropriata perché `BookService` assume una dipendenza diretta a `MongoClient`. Le [linee guida per il riutilizzo dei client mongo](https://mongodb.github.io/mongo-csharp-driver/2.8/reference/driver/connecting/#re-use)ufficiali `MongoClient` devono essere registrate in di con una durata del servizio singleton.

1. Aggiungere il codice seguente all'inizio del file *Startup.cs* per risolvere il riferimento a `BookService`:

   [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Startup.cs?name=snippet_UsingBooksApiServices)]

La classe `BookService` usa i membri `MongoDB.Driver` seguenti per eseguire operazioni CRUD sul database:

* [MongoClient](https://api.mongodb.com/csharp/current/html/T_MongoDB_Driver_MongoClient.htm): legge l'istanza del server per l'esecuzione di operazioni di database. Al costruttore di questa classe viene passata la stringa di connessione MongoDB:

  [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Services/BookService.cs?name=snippet_BookServiceConstructor&highlight=3)]

* [IMongoDatabase](https://api.mongodb.com/csharp/current/html/T_MongoDB_Driver_IMongoDatabase.htm): rappresenta il database Mongo per l'esecuzione di operazioni. Questa esercitazione usa il metodo [GetCollection\<TDocument>(collection)](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoDatabase_GetCollection__1.htm) generico sull'interfaccia per ottenere l'accesso ai dati in una raccolta specifica. Eseguire le operazioni CRUD sulla raccolta dopo la chiamata a questo metodo. Nella chiamata del metodo `GetCollection<TDocument>(collection)`:

  * `collection` rappresenta il nome della raccolta.
  * `TDocument` rappresenta il tipo di oggetto CLR archiviato nella raccolta.

`GetCollection<TDocument>(collection)` restituisce un oggetto [MongoCollection](https://api.mongodb.com/csharp/current/html/T_MongoDB_Driver_MongoCollection.htm) che rappresenta la raccolta. In questa esercitazione, vengono richiamati i metodi seguenti sulla raccolta:

* [DeleteOne](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollection_1_DeleteOne.htm): Elimina un singolo documento corrispondente ai criteri di ricerca specificati.
* [Find \<TDocument> ](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollectionExtensions_Find__1_1.htm): restituisce tutti i documenti nella raccolta che corrispondono ai criteri di ricerca specificati.
* [InsertOne](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollection_1_InsertOne.htm): inserisce l'oggetto fornito come nuovo documento nella raccolta.
* [ReplaceOne](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollection_1_ReplaceOne.htm): sostituisce il singolo documento corrispondente ai criteri di ricerca specificati con l'oggetto fornito.

## <a name="add-a-controller"></a>Aggiunta di un controller

Aggiungere una classe `BooksController` alla directory *Controllers* con il codice seguente:

[!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Controllers/BooksController.cs)]

Il controller dell'API Web precedente:

* Usa la classe `BookService` per eseguire operazioni CRUD.
* Contiene metodi di azione per supportare le richieste HTTP GET, POST, PUT e DELETE.
* Chiama <xref:System.Web.Http.ApiController.CreatedAtRoute*> nel metodo dell'azione `Create` per restituire una risposta [HTTP 201](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html). Il codice di stato 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server. `Location` aggiunge anche un'intestazione `CreatedAtRoute` alla risposta. L'intestazione `Location` specifica l'URI del libro appena creato.

## <a name="test-the-web-api"></a>Testare l'API Web

1. Compilare ed eseguire l'app.

1. Passare a `http://localhost:<port>/api/books` per testare il metodo dell'azione `Get` senza parametri del controller. Viene visualizzata la risposta JSON seguente:

   ```json
   [
     {
       "id":"5bfd996f7b8e48dc15ff215d",
       "bookName":"Design Patterns",
       "price":54.93,
       "category":"Computers",
       "author":"Ralph Johnson"
     },
     {
       "id":"5bfd996f7b8e48dc15ff215e",
       "bookName":"Clean Code",
       "price":43.15,
       "category":"Computers",
       "author":"Robert C. Martin"
     }
   ]
   ```

1. Passare a `http://localhost:<port>/api/books/{id here}` per testare il metodo dell'azione `Get` in overload del controller. Viene visualizzata la risposta JSON seguente:

   ```json
   {
     "id":"{ID}",
     "bookName":"Clean Code",
     "price":43.15,
     "category":"Computers",
     "author":"Robert C. Martin"
   }
   ```

## <a name="configure-json-serialization-options"></a>Configurare le opzioni di serializzazione JSON

Esistono due dettagli da modificare per le risposte JSON restituite nella sezione [Testare l'API Web](#test-the-web-api):

* La notazione a cammello predefinita per i nomi di proprietà deve essere modificata in modo da adottare la convenzione Pascal dei nomi di proprietà dell'oggetto CLR.
* La proprietà `bookName` deve essere restituita come `Name`.

Per soddisfare i requisiti precedenti, apportare le modifiche seguenti:

1. JSON.NET è stato rimosso dal framework condiviso di ASP.NET. Aggiungere un riferimento al pacchetto a [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson) .

1. In `Startup.ConfigureServices` concatenare il codice evidenziato seguente alla chiamata del metodo `AddControllers`:

   [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Startup.cs?name=snippet_ConfigureServices&highlight=12)]

   Con la modifica precedente, i nomi delle proprietà nella risposta JSON serializzata dell'API Web corrispondono ai nomi di proprietà corrispondenti nel tipo di oggetto CLR. Ad esempio, la proprietà `Author` della classe `Book` viene serializzata come `Author`.

1. In *models/book. cs* annotare la `BookName` proprietà con l' [`[JsonProperty]`](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_JsonPropertyAttribute.htm) attributo seguente:

   [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Models/Book.cs?name=snippet_BookNameProperty&highlight=2)]

   Il valore `Name` dell'attributo `[JsonProperty]` rappresenta il nome della proprietà nella risposta JSON serializzata dell'API Web.

1. Aggiungere il codice seguente all'inizio di *Models/Book.cs* per risolvere il riferimento all'attributo `[JsonProperty]`:

   [!code-csharp[](first-mongo-app/samples/3.x/SampleApp/Models/Book.cs?name=snippet_NewtonsoftJsonImport)]

1. Ripetere i passaggi definiti nella sezione [Testare l'API Web](#test-the-web-api). Si noti la differenza nei nomi di proprietà JSON.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Questa esercitazione crea un'API Web che esegue operazioni di creazione, lettura, aggiornamento ed eliminazione (CRUD) su un database NoSQL [MongoDB](https://www.mongodb.com/what-is-mongodb).

In questa esercitazione verranno illustrate le procedure per:

> [!div class="checklist"]
> * Configurare MongoDB
> * Creare un database MongoDB
> * Definire una raccolta e uno schema MongoDB
> * Eseguire operazioni CRUD di MongoDB da un'API Web
> * Personalizzare la serializzazione JSON

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/first-mongo-app/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* [.NET Core SDK 2.2](https://dotnet.microsoft.com/download/dotnet-core)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**
* [MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [.NET Core SDK 2.2](https://dotnet.microsoft.com/download/dotnet-core)
* [Visual Studio Code](https://code.visualstudio.com/download)
* [C# per Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [MongoDB](https://docs.mongodb.com/manual/administration/install-community/)

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* [.NET Core SDK 2.2](https://dotnet.microsoft.com/download/dotnet-core)
* [Visual Studio per Mac versione 7.7 o successiva](https://visualstudio.microsoft.com/downloads/)
* [MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)

---

## <a name="configure-mongodb"></a>Configurare MongoDB

Se si usa Windows, MongoDB viene installato in *C: \\ programmi \\ MongoDB* per impostazione predefinita. Aggiungere *C: \\ Program Files \\ MongoDB \\ server \\ \<version_number> \\ bin* alla `Path` variabile di ambiente. Questa modifica consente l'accesso MongoDB da qualsiasi posizione nel computer di sviluppo.

Usare la shell mongo nelle procedure seguenti per creare un database, creare le raccolte e archiviare i documenti. Per altre informazioni sui comandi della shell mongo, vedere [Working with the mongo Shell](https://docs.mongodb.com/manual/mongo/#working-with-the-mongo-shell) (Utilizzo della shell mongo).

1. Scegliere una directory nel computer di sviluppo per archiviare i dati. Ad esempio, *C: \\ BooksData* in Windows. Creare la directory se non esiste. La shell mongo non consente di creare nuove directory.
1. Aprire una shell dei comandi. Eseguire il comando seguente per connettersi a MongoDB sulla porta predefinita 27017. Ricordare di sostituire `<data_directory_path>` con la directory scelta nel passaggio precedente.

   ```console
   mongod --dbpath <data_directory_path>
   ```

1. Aprire un'altra istanza della shell dei comandi. Connettersi al database di test predefinito eseguendo il comando seguente:

   ```console
   mongo
   ```

1. Eseguire il comando seguente in una shell dei comandi:

   ```console
   use BookstoreDb
   ```

   Se non esiste già, viene creato un database denominato *BookstoreDb*. Se il database esiste, la connessione viene aperta per le transazioni.

1. Creare una raccolta `Books` tramite il comando seguente:

   ```console
   db.createCollection('Books')
   ```

   Viene visualizzato il risultato seguente:

   ```console
   { "ok" : 1 }
   ```

1. Definire uno schema per la raccolta `Books` e inserire due documenti usando il comando seguente:

   ```console
   db.Books.insertMany([{'Name':'Design Patterns','Price':54.93,'Category':'Computers','Author':'Ralph Johnson'}, {'Name':'Clean Code','Price':43.15,'Category':'Computers','Author':'Robert C. Martin'}])
   ```

   Viene visualizzato il risultato seguente:

   ```console
   {
     "acknowledged" : true,
     "insertedIds" : [
       ObjectId("5bfd996f7b8e48dc15ff215d"),
       ObjectId("5bfd996f7b8e48dc15ff215e")
     ]
   }
   ```
  
   > [!NOTE]
   > L'ID illustrato in questo articolo non corrisponde agli ID quando si esegue questo campione.

1. Visualizzare i documenti nel database usando il comando seguente:

   ```console
   db.Books.find({}).pretty()
   ```

   Viene visualizzato il risultato seguente:

   ```console
   {
     "_id" : ObjectId("5bfd996f7b8e48dc15ff215d"),
     "Name" : "Design Patterns",
     "Price" : 54.93,
     "Category" : "Computers",
     "Author" : "Ralph Johnson"
   }
   {
     "_id" : ObjectId("5bfd996f7b8e48dc15ff215e"),
     "Name" : "Clean Code",
     "Price" : 43.15,
     "Category" : "Computers",
     "Author" : "Robert C. Martin"
   }
   ```

   Lo schema aggiunge una proprietà `_id` generata automaticamente di tipo `ObjectId` per ogni documento.

Il database è pronto. È possibile iniziare a creare l'API Web ASP.NET Core.

## <a name="create-the-aspnet-core-web-api-project"></a>Creare il progetto per l'API Web ASP.NET Core

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Passare a **file** > **nuovo** > **progetto**.
1. Selezionare il tipo di progetto **Applicazione Web ASP.NET Core** e selezionare **Avanti**.
1. Assegnare al progetto il nome *BooksApi* e selezionare **Crea**.
1. Selezionare il framework di destinazione **.NET Core** e **ASP.NET Core 2.2**. Selezionare il modello di progetto **API** e scegliere **Crea**.
1. Visitare la [raccolta NuGet: MongoDB. driver](https://www.nuget.org/packages/MongoDB.Driver/) per determinare la versione stabile più recente del driver .NET per MongoDB. Nella finestra **Console di Gestione pacchetti** passare alla radice del progetto. Eseguire il comando seguente per installare il driver .NET per MongoDB:

   ```powershell
   Install-Package MongoDB.Driver -Version {VERSION}
   ```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. Eseguire i comandi seguenti in una shell dei comandi:

   ```dotnetcli
   dotnet new webapi -o BooksApi
   code BooksApi
   ```

   Un nuovo progetto API Web ASP.NET Core destinato a .NET Core viene generato e aperto in Visual Studio Code.

1. Dopo che l'icona di OmniSharp Flame della barra di stato è verde, una finestra di dialogo richiede che le **risorse necessarie per la compilazione e il debug siano mancanti da' BooksApi '. Aggiungerli?** Selezionare **Sì**.
1. Visitare la [raccolta NuGet: MongoDB. driver](https://www.nuget.org/packages/MongoDB.Driver/) per determinare la versione stabile più recente del driver .NET per MongoDB. Aprire **Terminale integrato** e passare alla radice del progetto. Eseguire il comando seguente per installare il driver .NET per MongoDB:

   ```dotnetcli
   dotnet add BooksApi.csproj package MongoDB.Driver -v {VERSION}
   ```

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. In Visual Studio per Mac precedente alla versione 8,6, selezionare **file**  >  **nuova soluzione**  >  **.NET Core**  >  **app** dall'intestazione laterale. Nella versione 8,6 o successive selezionare **file**  >  **nuova soluzione**  >  **Web e**  >  **app** console dalla barra laterale.
1. Selezionare il modello di progetto C# **API Web ASP.NET Core** e selezionare **Avanti**.
1. Selezionare **.NET Core 2.2** nell'elenco a discesa **Framework di destinazione** e selezionare **Avanti**.
1. Immettere *BooksApi* per **Nome progetto** e selezionare **Crea**.
1. Nel riquadro **Soluzione** fare clic con il pulsante destro del mouse sul nodo **Dipendenze** del progetto e scegliere **Aggiungi pacchetti**.
1. Immettere *MongoDB.Driver* nella casella di ricerca, selezionare il pacchetto *MongoDB.Driver* e quindi **Aggiungi pacchetto**.
1. Selezionare il pulsante **Accetta** nella finestra di dialogo **Accettazione della licenza**.

---

## <a name="add-an-entity-model"></a>Aggiungere un modello di entità

1. Aggiungere una directory *Models* alla radice del progetto.
1. Aggiungere una classe `Book` alla directory *Models* con il codice seguente:

   ```csharp
   using MongoDB.Bson;
   using MongoDB.Bson.Serialization.Attributes;

   namespace BooksApi.Models
   {
       public class Book
       {
           [BsonId]
           [BsonRepresentation(BsonType.ObjectId)]
           public string Id { get; set; }

           [BsonElement("Name")]
           public string BookName { get; set; }

           public decimal Price { get; set; }

           public string Category { get; set; }

           public string Author { get; set; }
       }
   }
   ```

   Nella classe precedente, la proprietà `Id`:

   * È obbligatoria per il mapping tra l'oggetto CLR (Common Language Runtime) e la raccolta MongoDB.
   * Viene annotato con [`[BsonId]`](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_Serialization_Attributes_BsonIdAttribute.htm) per indicare questa proprietà come chiave primaria del documento.
   * Viene annotato con [`[BsonRepresentation(BsonType.ObjectId)]`](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_Serialization_Attributes_BsonRepresentationAttribute.htm) per consentire il passaggio del parametro come tipo `string` anziché come una struttura [ObjectID](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_ObjectId.htm) . Mongo gestisce la conversione da `string` a `ObjectId`.

   La `BookName` proprietà è annotata con l' [`[BsonElement]`](https://api.mongodb.com/csharp/current/html/T_MongoDB_Bson_Serialization_Attributes_BsonElementAttribute.htm) attributo. Il valore dell'attributo `Name` rappresenta il nome della proprietà nella raccolta MongoDB.

## <a name="add-a-configuration-model"></a>Aggiungere un modello di configurazione

1. Aggiungere i valori di configurazione del database seguenti a *appsettings.json* :

   [!code-json[](first-mongo-app/samples/2.x/SampleApp/appsettings.json?highlight=2-6)]

1. Aggiungere un file *BookstoreDatabaseSettings.cs* alla directory *Models* con il codice seguente:

   [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Models/BookstoreDatabaseSettings.cs)]

   La `BookstoreDatabaseSettings` classe precedente viene utilizzata per archiviare i *appsettings.json* valori delle `BookstoreDatabaseSettings` proprietà del file. I nomi delle proprietà JSON e C# sono identici per semplificare il processo di mapping.

1. Aggiungere il codice evidenziato seguente a `Startup.ConfigureServices`:

   [!code-csharp[](first-mongo-app/samples_snapshot/2.x/SampleApp/Startup.ConfigureServices.AddDbSettings.cs?highlight=3-7)]

   Nel codice precedente:

   * L'istanza di configurazione a cui *appsettings.json* `BookstoreDatabaseSettings` viene associata la sezione del file è registrata nel contenitore di inserimento delle dipendenze. Ad esempio, la `BookstoreDatabaseSettings` proprietà di un oggetto `ConnectionString` viene popolata con la `BookstoreDatabaseSettings:ConnectionString` Proprietà in *appsettings.json* .
   * L'interfaccia `IBookstoreDatabaseSettings` è registrata nell'inserimento di dipendenze con una [durata del servizio](xref:fundamentals/dependency-injection#service-lifetimes) singleton. Quando avviene l'inserimento, l'istanza dell'interfaccia restituisce un oggetto `BookstoreDatabaseSettings`.

1. Aggiungere il codice seguente all'inizio del file *Startup.cs* per risolvere i riferimenti a `BookstoreDatabaseSettings` e `IBookstoreDatabaseSettings`:

   [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Startup.cs?name=snippet_UsingBooksApiModels)]

## <a name="add-a-crud-operations-service"></a>Aggiungere un servizio di operazioni CRUD

1. Aggiungere una directory *Services* alla radice del progetto.
1. Aggiungere una classe `BookService` alla directory *Services* con il codice seguente:

   [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Services/BookService.cs?name=snippet_BookServiceClass)]

   Nel codice precedente un'istanza di `IBookstoreDatabaseSettings` viene recuperata dall'inserimento di dipendenze tramite l'inserimento del costruttore. Questa tecnica consente di accedere ai *appsettings.json* valori di configurazione aggiunti nella sezione [aggiungere un modello di configurazione](#add-a-configuration-model) .

1. Aggiungere il codice evidenziato seguente a `Startup.ConfigureServices`:

   [!code-csharp[](first-mongo-app/samples_snapshot/2.x/SampleApp/Startup.ConfigureServices.AddSingletonService.cs?highlight=9)]

   Nel codice precedente la classe `BookService` è registrata con l'inserimento di dipendenze per supportare l'inserimento del costruttore nelle classi che la utilizzano. La durata del servizio singleton è più appropriata perché `BookService` assume una dipendenza diretta a `MongoClient`. Le [linee guida per il riutilizzo dei client mongo](https://mongodb.github.io/mongo-csharp-driver/2.8/reference/driver/connecting/#re-use)ufficiali `MongoClient` devono essere registrate in di con una durata del servizio singleton.

1. Aggiungere il codice seguente all'inizio del file *Startup.cs* per risolvere il riferimento a `BookService`:

   [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Startup.cs?name=snippet_UsingBooksApiServices)]

La classe `BookService` usa i membri `MongoDB.Driver` seguenti per eseguire operazioni CRUD sul database:

* [MongoClient](https://api.mongodb.com/csharp/current/html/T_MongoDB_Driver_MongoClient.htm): legge l'istanza del server per l'esecuzione di operazioni di database. Al costruttore di questa classe viene passata la stringa di connessione MongoDB:

  [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Services/BookService.cs?name=snippet_BookServiceConstructor&highlight=3)]

* [IMongoDatabase](https://api.mongodb.com/csharp/current/html/T_MongoDB_Driver_IMongoDatabase.htm): rappresenta il database Mongo per l'esecuzione di operazioni. Questa esercitazione usa il metodo [GetCollection\<TDocument>(collection)](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoDatabase_GetCollection__1.htm) generico sull'interfaccia per ottenere l'accesso ai dati in una raccolta specifica. Eseguire le operazioni CRUD sulla raccolta dopo la chiamata a questo metodo. Nella chiamata del metodo `GetCollection<TDocument>(collection)`:

  * `collection` rappresenta il nome della raccolta.
  * `TDocument` rappresenta il tipo di oggetto CLR archiviato nella raccolta.

`GetCollection<TDocument>(collection)` restituisce un oggetto [MongoCollection](https://api.mongodb.com/csharp/current/html/T_MongoDB_Driver_MongoCollection.htm) che rappresenta la raccolta. In questa esercitazione, vengono richiamati i metodi seguenti sulla raccolta:

* [DeleteOne](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollection_1_DeleteOne.htm): Elimina un singolo documento corrispondente ai criteri di ricerca specificati.
* [Find \<TDocument> ](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollectionExtensions_Find__1_1.htm): restituisce tutti i documenti nella raccolta che corrispondono ai criteri di ricerca specificati.
* [InsertOne](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollection_1_InsertOne.htm): inserisce l'oggetto fornito come nuovo documento nella raccolta.
* [ReplaceOne](https://api.mongodb.com/csharp/current/html/M_MongoDB_Driver_IMongoCollection_1_ReplaceOne.htm): sostituisce il singolo documento corrispondente ai criteri di ricerca specificati con l'oggetto fornito.

## <a name="add-a-controller"></a>Aggiunta di un controller

Aggiungere una classe `BooksController` alla directory *Controllers* con il codice seguente:

[!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Controllers/BooksController.cs)]

Il controller dell'API Web precedente:

* Usa la classe `BookService` per eseguire operazioni CRUD.
* Contiene metodi di azione per supportare le richieste HTTP GET, POST, PUT e DELETE.
* Chiama <xref:System.Web.Http.ApiController.CreatedAtRoute*> nel metodo dell'azione `Create` per restituire una risposta [HTTP 201](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html). Il codice di stato 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server. `Location` aggiunge anche un'intestazione `CreatedAtRoute` alla risposta. L'intestazione `Location` specifica l'URI del libro appena creato.

## <a name="test-the-web-api"></a>Testare l'API Web

1. Compilare ed eseguire l'app.

1. Passare a `http://localhost:<port>/api/books` per testare il metodo dell'azione `Get` senza parametri del controller. Viene visualizzata la risposta JSON seguente:

   ```json
   [
     {
       "id":"5bfd996f7b8e48dc15ff215d",
       "bookName":"Design Patterns",
       "price":54.93,
       "category":"Computers",
       "author":"Ralph Johnson"
     },
     {
       "id":"5bfd996f7b8e48dc15ff215e",
       "bookName":"Clean Code",
       "price":43.15,
       "category":"Computers",
       "author":"Robert C. Martin"
     }
   ]
   ```

1. Passare a `http://localhost:<port>/api/books/{id here}` per testare il metodo dell'azione `Get` in overload del controller. Viene visualizzata la risposta JSON seguente:

   ```json
   {
     "id":"{ID}",
     "bookName":"Clean Code",
     "price":43.15,
     "category":"Computers",
     "author":"Robert C. Martin"
   }
   ```

## <a name="configure-json-serialization-options"></a>Configurare le opzioni di serializzazione JSON

Esistono due dettagli da modificare per le risposte JSON restituite nella sezione [Testare l'API Web](#test-the-web-api):

* La notazione a cammello predefinita per i nomi di proprietà deve essere modificata in modo da adottare la convenzione Pascal dei nomi di proprietà dell'oggetto CLR.
* La proprietà `bookName` deve essere restituita come `Name`.

Per soddisfare i requisiti precedenti, apportare le modifiche seguenti:

1. In `Startup.ConfigureServices` concatenare il codice evidenziato seguente alla chiamata del metodo `AddMvc`:

   [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Startup.cs?name=snippet_ConfigureServices&highlight=12)]

   Con la modifica precedente, i nomi delle proprietà nella risposta JSON serializzata dell'API Web corrispondono ai nomi di proprietà corrispondenti nel tipo di oggetto CLR. Ad esempio, la proprietà `Author` della classe `Book` viene serializzata come `Author`.

1. In *models/book. cs* annotare la `BookName` proprietà con l' [`[JsonProperty]`](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_JsonPropertyAttribute.htm) attributo seguente:

   [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Models/Book.cs?name=snippet_BookNameProperty&highlight=2)]

   Il valore `Name` dell'attributo `[JsonProperty]` rappresenta il nome della proprietà nella risposta JSON serializzata dell'API Web.

1. Aggiungere il codice seguente all'inizio di *Models/Book.cs* per risolvere il riferimento all'attributo `[JsonProperty]`:

   [!code-csharp[](first-mongo-app/samples/2.x/SampleApp/Models/Book.cs?name=snippet_NewtonsoftJsonImport)]

1. Ripetere i passaggi definiti nella sezione [Testare l'API Web](#test-the-web-api). Si noti la differenza nei nomi di proprietà JSON.

::: moniker-end

## <a name="add-authentication-support-to-a-web-api"></a>Aggiungere il supporto per l'autenticazione a un'API Web

[!INCLUDE[](~/includes/IdentityServer4.md)]

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sulla creazione di API Web ASP.NET Core, vedere le risorse seguenti:

* [Versione di YouTube di questo articolo](https://www.youtube.com/watch?v=7uJt_sOenyo&feature=youtu.be)
* <xref:web-api/index>
* <xref:web-api/action-return-types>
* [Microsoft Learn: creare un'API Web con ASP.NET Core](/learn/modules/build-web-api-aspnet-core/)
