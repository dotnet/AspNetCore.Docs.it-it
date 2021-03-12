---
title: "Esercitazione: Creare un'API Web con ASP.NET Core"
author: rick-anderson
description: Informazioni su come creare un'API Web con ASP.NET Core.
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 02/04/2021
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
- Models
uid: tutorials/first-web-api
ms.openlocfilehash: 789cd1a867bc8c17401bbac5c02951b4bd2999b6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587658"
---
# <a name="tutorial-create-a-web-api-with-aspnet-core"></a>Esercitazione: Creare un'API Web con ASP.NET Core

Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Kirk Larkin](https://twitter.com/serpent5)e [Mike Wasson](https://github.com/mikewasson)

Questa esercitazione illustra le nozioni di base della creazione di un'API Web con ASP.NET Core.

::: moniker range=">= aspnetcore-5.0"

In questa esercitazione verranno illustrate le procedure per:

> [!div class="checklist"]
> * Creare un progetto di API Web.
> * Aggiungere una classe modello e un contesto di database.
> * Eseguire lo scaffolding di un controller con i metodi CRUD.
> * Configurare il routing, i percorsi URL e i valori restituiti.
> * Chiamare l'API Web con Postman.

Al termine si avrà un'API Web che può gestire gli elementi di tipo "attività" archiviati in un database.

## <a name="overview"></a>Panoramica

Questa esercitazione consente di creare l'API seguente:

|API | Descrizione | Testo della richiesta | Corpo della risposta |
|--- | ---- | ---- | ---- |
|`GET /api/TodoItems` | Ottiene tutti gli elementi attività | nessuno | Matrice di elementi attività|
|`GET /api/TodoItems/{id}` | Ottiene un elemento in base all'ID | nessuno | Elemento attività|
|`POST /api/TodoItems` | Aggiunge un nuovo elemento | Elemento attività | Elemento attività |
|`PUT /api/TodoItems/{id}` | Aggiorna un elemento esistente &nbsp; | Elemento attività | nessuno |
|`DELETE /api/TodoItems/{id}` &nbsp; &nbsp; | Elimina un elemento &nbsp;&nbsp; | nessuno | nessuno|

Il diagramma seguente visualizza la struttura dell'app.

![Il client è rappresentato da una casella a sinistra. Invia una richiesta e riceve una risposta dall'applicazione, una casella disegnata a destra. Nella riquadro dell'applicazione tre caselle rappresentano il controller, il modello e il livello di accesso ai dati. La richiesta viene ricevuta dal controller dell'applicazione e vengono eseguite operazioni di lettura/scrittura tra il controller e il livello di accesso ai dati. Il modello viene serializzato e restituito al client nella risposta.](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-5.0.md)]

---

## <a name="create-a-web-project"></a>Creare un progetto Web

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Scegliere **nuovo** progetto dal menu **file** > .
* Selezionare il modello **Applicazione Web ASP.NET Core** e fare clic su **Avanti**.
* Assegnare al progetto il nome *TodoApi* e fare clic su **Crea**.
* Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.net Core** e **ASP.NET Core 5,0** . Selezionare il modello **API** e fare clic su **Crea**.

![Finestra di dialogo Nuovo progetto di Visual Studio](first-web-api/_static/5/vs.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Cambiare directory (`cd`) e passare alla cartella che conterrà la cartella del progetto.
* Eseguire i comandi seguenti:

   ```dotnetcli
   dotnet new webapi -o TodoApi
   cd TodoApi
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   code -r ../TodoApi
   ```

* Quando una finestra di dialogo chiede se si vuole aggiungere gli asset necessari al progetto, selezionare **Sì**.

  I comandi precedenti:

  * Crea un nuovo progetto API Web e lo apre in Visual Studio Code.
  * Aggiunge i pacchetti NuGet necessari nella sezione successiva.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Selezionare **File** > **Nuova soluzione**.

  ![Nuova soluzione macOS](first-web-api-mac/_static/sln.png)

* In Visual Studio per Mac precedente alla versione 8,6 Selezionare API **app .NET Core**  >    >    >  **Avanti**. Nella versione 8,6 o successive selezionare API **app Web e console**  >    >    >  **Avanti**.

  ![Selezione modello API macOS](first-web-api-mac/_static/api_template.png)

* Nella finestra di dialogo **Configura nuova ASP.NET Core API Web** selezionare il **Framework di destinazione**.NET Core 5. x più recente. Selezionare **Avanti**.

* Immettere *TodoApi* in **Nome progetto**, quindi selezionare **Crea**.

  ![Finestra di dialogo di configurazione](first-web-api-mac/_static/2.png)

[!INCLUDE[](~/includes/mac-terminal-access.md)]

Aprire un terminale di comando nella cartella del progetto ed eseguire il comando seguente:

   ```dotnetcli
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   ```

---

### <a name="test-the-project"></a>Testare il progetto

Il modello di progetto crea un' `WeatherForecast` API con supporto per [spavalderia](xref:tutorials/web-api-help-pages-using-swagger).

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Premere CTRL+F5 per l'esecuzione senza il debugger.

[!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio viene avviato:

* Server Web IIS Express.
* Il browser predefinito e passa a `https://localhost:<port>/swagger/index.html` , dove `<port>` è un numero di porta scelto in modo casuale.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/trustCertVSC.md)]

Premere CTRL+F5 per eseguire l'app. In un browser passare all'URL seguente: [https://localhost:5001/swagger](https://localhost:5001/swagger)

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Selezionare **Esegui**  >  **Avvia debug** per avviare l'app. Visual Studio per Mac apre un browser e naviga all'indirizzo `https://localhost:<port>`, dove `<port>` è un numero di porta selezionato a caso. Viene restituito un errore HTTP 404 (Non trovato). Accodare `/swagger` all'URL (impostare l'URL su `https://localhost:<port>/swagger`).

---

Viene visualizzata la pagina di spavalderia `/swagger/index.html` . Selezionare **Get**  >  **try it out**  >  **Execute (Esegui**). Viene visualizzata la pagina:

* Comando [curl](https://curl.haxx.se/) per testare l'API weatherforecast.
* URL per testare l'API WeatherForecast.
* Il codice di risposta, il corpo e le intestazioni.
* Casella di riepilogo a discesa con i tipi di supporto e il valore e lo schema di esempio.

<!-- Review: Do we care the IE generates several errors. It shows the data, but with  Unrecognized response type; displaying content as text.
-->
Spavalderia viene usato per generare la documentazione e le pagine della guida utili per le API Web. Questa esercitazione è incentrata sulla creazione di un'API Web. Per ulteriori informazioni su spavalderia, vedere <xref:tutorials/web-api-help-pages-using-swagger> .

Copiare e incollare l' **URL della richiesta** nel browser:  `https://localhost:<port>/WeatherForecast`

Viene restituito un codice JSON simile al seguente:

```json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
```

### <a name="update-the-launchurl"></a>Aggiornare launchUrl

In *Properties\launchSettings.json* aggiornare `launchUrl` da `"swagger"` a `"api/TodoItems"` :

```json
"launchUrl": "api/TodoItems",
```

Poiché spavalderia è stato rimosso, il markup precedente modifica l'URL che viene avviato al metodo GET del controller aggiunto nelle sezioni riportate di seguito.

## <a name="add-a-model-class"></a>Aggiungere una classe del modello

Un *modello* è un set di classi che rappresentano i dati gestiti dall'app. Il modello per questa app è una classe `TodoItem` singola.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto. Selezionare **Aggiungi**  >  **nuova cartella**. Assegnare un nome alla cartella *Models* .

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**. Assegnare alla classe il nome *TodoItem* e selezionare **Aggiungi**.

* Sostituire il codice del modello con il codice seguente:

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aggiungere una cartella denominata *Models* .

* Aggiungere una `TodoItem` classe alla *Models* cartella con il codice seguente:

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Fare clic con il pulsante destro del mouse sul progetto. Selezionare **Aggiungi**  >  **nuova cartella**. Assegnare un nome alla cartella *Models* .

  ![Nuova cartella](first-web-api-mac/_static/folder.png)

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi** > **nuovo file** > **generale** > **vuota classe**.

* Assegnare alla classe il nome *TodoItem* e fare clic su **Nuovo**.

* Sostituire il codice del modello con il codice seguente:

---

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Models/TodoItem.cs?name=snippet)]

La proprietà `Id` funziona come chiave univoca in un database relazionale.

Le classi di modelli possono essere inserite in qualsiasi punto del progetto, ma la *Models* cartella viene utilizzata per convenzione.

## <a name="add-a-database-context"></a>Aggiungere un contesto di database

Il *contesto di database* è la classe principale che coordina le funzionalità di Entity Framework per un modello di dati. Questa classe viene creata mediante derivazione dalla classe <xref:Microsoft.EntityFrameworkCore.DbContext?displayProperty=fullName>.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="add-nuget-packages"></a>Aggiungere pacchetti NuGet

* Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Gestisci pacchetti NuGet per la soluzione**.
* Selezionare la scheda **Sfoglia** e quindi immettere `Microsoft.EntityFrameworkCore.InMemory` nella casella di ricerca.
* Selezionare `Microsoft.EntityFrameworkCore.InMemory` nel riquadro sinistro.
* Selezionare la casella di controllo **Progetto** nel riquadro di destra e quindi selezionare **Installa**.

![Gestione pacchetti NuGet](first-web-api/_static/5/vsNuGet.png)

## <a name="add-the-todocontext-database-context"></a>Aggiungere il contesto del database TodoContext

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**. Assegnare alla classe il nome *TodoContext* e fare clic su **Aggiungi**.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

* Aggiungere una `TodoContext` classe alla *Models* cartella.

---

* Immettere il codice seguente:

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a>Registrare il contesto del database

In ASP.NET Core i servizi come il contesto del database devono essere registrati con il [contenitore di inserimento delle dipendenze](xref:fundamentals/dependency-injection). Il contenitore rende disponibile il servizio ai controller.

Aggiornare *Startup.cs* con il codice seguente:

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Startup.cs?highlight=7-8,23-24&name=snippet_all)]

Il codice precedente:

* Rimuove le chiamate di spavalderia.
* Rimuove le dichiarazioni `using` inutilizzate.
* Aggiunge il contesto del database al contenitore di inserimento delle dipendenze.
* Specifica che il contesto del database userà un database in memoria.

## <a name="scaffold-a-controller"></a>Eseguire lo scaffolding di un controller

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Fare clic con il pulsante destro del mouse sulla cartella *Controllers*.
* Selezionare **Aggiungi** > **Nuovo elemento di scaffolding**.
* Selezionare **Controller API con azioni, che usa Entity Framework** e quindi selezionare **Aggiungi**.
* Nella finestra di dialogo **Add API Controller with actions, using Entity Framework** (Aggiungi controller API con azioni, che usa Entity Framework):

  * Selezionare **TodoItem (TodoApi. Models )** nella **classe del modello**.
  * Selezionare **TodoContext (TodoApi. Models )** nella **classe del contesto dati**.
  * Selezionare **Aggiungi**.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Eseguire i comandi seguenti:

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

I comandi precedenti:

* Aggiungere i pacchetti NuGet necessari per lo scaffolding.
* Installa il motore di scaffolding (`dotnet-aspnet-codegenerator`).
* Esegue lo scaffolding di `TodoItemsController`.

---

Il codice generato:

* Contrassegna la classe con l' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo. L'attributo indica che il controller risponde alle richieste di API Web. Per informazioni sui comportamenti specifici consentiti dall'attributo, vedere <xref:web-api/index>.
* Usa l'inserimento delle dipendenze per inserire il contesto del database (`TodoContext`) nel controller. Il contesto di database viene usato in ognuno dei metodi [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) nel controller.

I modelli di ASP.NET Core per:

* I controller con le visualizzazioni includono `[action]` nel modello di route.
* I controller API non includono `[action]` nel modello di route.

Quando il `[action]` token non è incluso nel modello di route, il nome dell' [azione](xref:mvc/controllers/routing#action) viene escluso dalla route. Ovvero, il nome del metodo associato all'azione non viene usato nella route corrispondente.

## <a name="update-the-posttodoitem-create-method"></a>Aggiornare il metodo di creazione PostTodoItem

Aggiornare l'istruzione return in `PostTodoItem` per usare l'operatore [NameOf](/dotnet/csharp/language-reference/operators/nameof) :

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Create)]

Il codice precedente è un metodo HTTP POST, come indicato dall' [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attributo. Il metodo ottiene il valore dell'elemento attività dal corpo della richiesta HTTP.

Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).

Il metodo <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>:

* Restituisce un [codice di stato HTTP 201](https://developer.mozilla.org/docs/Web/HTTP/Status/201) in caso di esito positivo. HTTP 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server.
* Aggiunge un'intestazione [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) alla risposta. L'intestazione `Location` specifica l'[URI](https://developer.mozilla.org/docs/Glossary/URI) dell'elemento attività appena creato. Per altre informazioni, vedere [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
* Fa riferimento all'azione `GetTodoItem` per creare l'URI dell'intestazione `Location`. La parola chiave `nameof` C# viene usata per evitare di impostare il nome dell'azione come hardcoded nella chiamata a `CreatedAtAction`.

### <a name="install-postman"></a>Installare Postman

Questa esercitazione usa Postman per testare l'API Web.

* Installa il [post](https://www.getpostman.com/downloads/)
* Avviare l'app Web.
* Avviare Postman.
* Disattivare **SSL certificate verification** (Verifica certificato SSL)
  * Da **File** > **Settings** (Impostazioni) (scheda **General** (Generale)) disabilitare **SSL certificate verification** (Verifica certificato SSL).
    > [!WARNING]
    > Riattivare la verifica dei certificati SSL al termine del test del controller.

<a name="post"></a>

### <a name="test-posttodoitem-with-postman"></a>Testare PostTodoItem con Postman

* Creare una nuova richiesta.
* Impostare il metodo HTTP su `POST`.
* Impostare l'URI su `https://localhost:<port>/api/TodoItems` . Ad esempio: `https://localhost:5001/api/TodoItems`.
* Selezionare la scheda **Corpo**.
* Selezionare il pulsante di opzione **raw** (non elaborato).
* Impostare il tipo su **JSON (application/json)**.
* Nel corpo della richiesta immettere codice JSON per un elemento attività:

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* Selezionare **Invia**.

  ![Postman con richiesta di creazione](first-web-api/_static/3/create.png)

### <a name="test-the-location-header-uri"></a>Testare l'URI dell'intestazione della posizione

L'URI dell'intestazione Location può essere testato nel browser. Copiare e incollare l'URI dell'intestazione location nel browser.

Per eseguire il test in un post:

* Selezionare la scheda **Headers** (Intestazioni) nel riquadro **Response** (Risposta).
* Copiare il valore dell'intestazione **Location** (Posizione):

  ![Scheda Headers (Intestazioni) della console Postman](first-web-api/_static/3/create.png)

* Impostare il metodo HTTP su `GET`.
* Impostare l'URI su `https://localhost:<port>/api/TodoItems/1` . Ad esempio: `https://localhost:5001/api/TodoItems/1`.
* Selezionare **Invia**.

## <a name="examine-the-get-methods"></a>Esaminare i metodi GET

Vengono implementati due endpoint GET:

* `GET /api/TodoItems`
* `GET /api/TodoItems/{id}`

Testare l'app chiamando i due endpoint da un browser o da Postman. Ad esempio:

* `https://localhost:5001/api/TodoItems`
* `https://localhost:5001/api/TodoItems/1`

Una risposta simile alla seguente viene generata dalla chiamata a `GetTodoItems`:

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

### <a name="test-get-with-postman"></a>Testare Get con Postman

* Creare una nuova richiesta.
* Impostare il metodo HTTP su **GET**.
* Impostare l'URI della richiesta su `https://localhost:<port>/api/TodoItems` . Ad esempio: `https://localhost:5001/api/TodoItems`.
* Impostare **Two pane view** (Visualizzazione in due riquadri) in Postman.
* Selezionare **Invia**.

Questa app usa un database in memoria. Se l'app viene arrestata e avviata, la richiesta GET precedente non restituirà alcun dato. Se non vengono restituiti dati, eseguire [POST](#post) per pubblicare i dati nell'app.

## <a name="routing-and-url-paths"></a>Routing e percorsi di URL

L' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo indica un metodo che risponde a una richiesta HTTP Get. Il percorso dell'URL per ogni metodo viene costruito nel modo seguente:

* Iniziare con la stringa di modello nell'attributo `Route` del controller:

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=TodoController&highlight=1)]

* Sostituire `[controller]` con il nome del controller, ovvero, per convenzione, il nome della classe controller meno il suffisso "Controller". In questo esempio il nome della classe controller è **TodoItems** Controller, quindi il nome del controller è "TodoItems". Il [routing](xref:mvc/controllers/routing) ASP.NET Core non fa distinzione tra maiuscole e minuscole.
* Se l'attributo `[HttpGet]` ha un modello di route, ad esempio `[HttpGet("products")]`, aggiungerlo al percorso. In questo esempio non si usa un modello. Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).

Nel metodo `GetTodoItem` seguente, `"{id}"` è una variabile segnaposto per l'identificatore univoco dell'elemento attività. Quando `GetTodoItem` viene richiamato, il valore di `"{id}"` nell'URL viene fornito al metodo nel `id` parametro.

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a>Valori restituiti

Il tipo restituito dei `GetTodoItems` metodi e `GetTodoItem` è [ActionResult. \<T> ](xref:web-api/action-return-types#actionresultt-type) ASP.NET Core serializza automaticamente l'oggetto su [JSON](https://www.json.org/) e scrive il codice JSON nel corpo del messaggio di risposta. Il codice di risposta per questo tipo restituito è [200 OK](https://developer.mozilla.org/docs/Web/HTTP/Status/200), supponendo che non esistano eccezioni non gestite. Le eccezioni non gestite vengono convertite in errori 5xx.

I tipi restituiti `ActionResult` possono rappresentare un ampio intervallo di codici di stato HTTP. Ad esempio, `GetTodoItem` può restituire due valori di stato diversi:

* Se nessun elemento corrisponde all'ID richiesto, il metodo restituisce un codice di errore di [stato 404](https://developer.mozilla.org/docs/Web/HTTP/Status/404) <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> .
* In caso contrario, il metodo restituisce il codice 200 con un corpo della risposta JSON. La restituzione di `item` risulta in una risposta HTTP 200.

## <a name="the-puttodoitem-method"></a>Metodo PutTodoItem

Esaminare il metodo `PutTodoItem`:

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Update)]

`PutTodoItem` è simile a `PostTodoItem` ma usa la richiesta HTTP PUT. La risposta è [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). In base alla specifica HTTP, una richiesta PUT richiede che il client invii l'intera entità aggiornata e non solo le modifiche. Per supportare gli aggiornamenti parziali, usare [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).

Se si riceve un errore chiamando `PutTodoItem`, chiamare `GET` per verificare che il database includa un elemento.

### <a name="test-the-puttodoitem-method"></a>Testare il metodo PutTodoItem

Questo esempio usa un database in memoria che deve essere inizializzato ogni volta che l'app viene avviata. Deve esistere un elemento nel database prima di eseguire una chiamata PUT. Chiamare GET per assicurarsi che sia presente un elemento nel database prima di effettuare una chiamata PUT.

Aggiornare l'elemento to-do con ID = 1 e impostarne il nome su `"feed fish"` :

```json
  {
    "Id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

L'immagine seguente visualizza l'aggiornamento di Postman:

![Console Postman con risposta 204 (No Content)](first-web-api/_static/3/pmcput.png)

## <a name="the-deletetodoitem-method"></a>Metodo DeleteTodoItem

Esaminare il metodo `DeleteTodoItem`:

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Delete)]

### <a name="test-the-deletetodoitem-method"></a>Testare il metodo DeleteTodoItem

Usare Postman per eliminare un elemento attività:

* Impostare il metodo su `DELETE`.
* Impostare l'URI dell'oggetto da eliminare (ad esempio `https://localhost:5001/api/TodoItems/1` ).
* Selezionare **Invia**.

<a name="over-post-v5"></a>

## <a name="prevent-over-posting"></a>Impedisci overposting

Attualmente l'app di esempio espone l'intero `TodoItem` oggetto. Le app di produzione limitano in genere i dati di input e restituiti usando un subset del modello. Esistono diversi motivi alla base di questo e la sicurezza è una delle principali. Il subset di un modello è in genere indicato come oggetto Trasferimento dati (DTO), modello di input o modello di visualizzazione. **Dto** viene usato in questo articolo.

Un DTO può essere usato per:

* Impedisci l'overposting.
* Nascondere le proprietà che i client non dovrebbero visualizzare.
* Omettere alcune proprietà per ridurre le dimensioni del payload.
* Rendere flat gli oggetti grafici che contengono oggetti annidati. I grafici a oggetti flat possono essere più pratici per i client.

Per illustrare l'approccio DTO, aggiornare la `TodoItem` classe in modo da includere un campo Secret:

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Models/TodoItem.cs?name=snippet&highlight=8)]

Il campo segreto deve essere nascosto da questa app, ma un'app amministrativa può scegliere di esporla.

Verificare che sia possibile pubblicare e ottenere il campo secret.

Creare un modello DTO:

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Models/TodoItemDTO.cs?name=snippet)]

Aggiornare `TodoItemsController` da usare `TodoItemDTO` :

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Controllers/TodoItemsController.cs?name=snippet)]

Verificare che non sia possibile inserire o ottenere il campo secret.

## <a name="call-the-web-api-with-javascript"></a>Chiamare l'API Web con JavaScript

Vedere [esercitazione: chiamare un'API web ASP.NET Core con JavaScript](xref:tutorials/web-api-javascript).

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

In questa esercitazione verranno illustrate le procedure per:

> [!div class="checklist"]
> * Creare un progetto di API Web.
> * Aggiungere una classe modello e un contesto di database.
> * Eseguire lo scaffolding di un controller con i metodi CRUD.
> * Configurare il routing, i percorsi URL e i valori restituiti.
> * Chiamare l'API Web con Postman.

Al termine si avrà un'API Web che può gestire gli elementi di tipo "attività" archiviati in un database.

## <a name="overview"></a>Panoramica

Questa esercitazione consente di creare l'API seguente:

|API | Descrizione | Testo della richiesta | Corpo della risposta |
|--- | ---- | ---- | ---- |
|`GET /api/TodoItems` | Ottiene tutti gli elementi attività | nessuno | Matrice di elementi attività|
|`GET /api/TodoItems/{id}` | Ottiene un elemento in base all'ID | nessuno | Elemento attività|
|`POST /api/TodoItems` | Aggiunge un nuovo elemento | Elemento attività | Elemento attività |
|`PUT /api/TodoItems/{id}` | Aggiorna un elemento esistente &nbsp; | Elemento attività | nessuno |
|`DELETE /api/TodoItems/{id}` &nbsp; &nbsp; | Elimina un elemento &nbsp;&nbsp; | nessuno | nessuno|

Il diagramma seguente visualizza la struttura dell'app.

![Il client è rappresentato da una casella a sinistra. Invia una richiesta e riceve una risposta dall'applicazione, una casella disegnata a destra. Nella riquadro dell'applicazione tre caselle rappresentano il controller, il modello e il livello di accesso ai dati. La richiesta viene ricevuta dal controller dell'applicazione e vengono eseguite operazioni di lettura/scrittura tra il controller e il livello di accesso ai dati. Il modello viene serializzato e restituito al client nella risposta.](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-web-project"></a>Creare un progetto Web

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Scegliere **nuovo** progetto dal menu **file** > .
* Selezionare il modello **Applicazione Web ASP.NET Core** e fare clic su **Avanti**.
* Assegnare al progetto il nome *TodoApi* e fare clic su **Crea**.
* Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.net Core** e **ASP.NET Core 3,1** . Selezionare il modello **API** e fare clic su **Crea**.

![Finestra di dialogo Nuovo progetto di Visual Studio](first-web-api/_static/vs3.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Cambiare directory (`cd`) e passare alla cartella che conterrà la cartella del progetto.
* Eseguire i comandi seguenti:

   ```dotnetcli
   dotnet new webapi -o TodoApi
   cd TodoApi
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   code -r ../TodoApi
   ```

* Quando una finestra di dialogo chiede se si vuole aggiungere gli asset necessari al progetto, selezionare **Sì**.

  I comandi precedenti:

  * Crea un nuovo progetto API Web e lo apre in Visual Studio Code.
  * Aggiunge i pacchetti NuGet necessari nella sezione successiva.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Selezionare **File** > **Nuova soluzione**.

  ![Nuova soluzione macOS](first-web-api-mac/_static/sln.png)

* In Visual Studio per Mac precedente alla versione 8,6 Selezionare API **app .NET Core**  >    >    >  **Avanti**. Nella versione 8,6 o successive selezionare API **app Web e console**  >    >    >  **Avanti**.

  ![Selezione modello API macOS](first-web-api-mac/_static/api_template.png)

* Nella finestra di dialogo **Configura nuova ASP.NET Core API Web** selezionare il **Framework di destinazione**.NET Core 3. x più recente. Selezionare **Avanti**.

* Immettere *TodoApi* in **Nome progetto**, quindi selezionare **Crea**.

  ![Finestra di dialogo di configurazione](first-web-api-mac/_static/2.png)

[!INCLUDE[](~/includes/mac-terminal-access.md)]

Aprire un terminale di comando nella cartella del progetto ed eseguire il comando seguente:

   ```dotnetcli
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   ```

---

### <a name="test-the-api"></a>Testare l'API

Il modello di progetto crea un'API `WeatherForecast`. Chiamare il metodo `Get` da un browser per eseguire il test dell'app.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Premere CTRL+F5 per eseguire l'app. Visual Studio apre un browser e naviga all'indirizzo `https://localhost:<port>/WeatherForecast`, dove `<port>` è un numero di porta selezionato a caso.

Se in una finestra di dialogo viene chiesto se considerare attendibile il certificato IIS Express, selezionare **Sì**. Nella finestra di dialogo **Avviso di sicurezza** successiva selezionare **Sì**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Premere CTRL+F5 per eseguire l'app. In un browser passare all'URL seguente: `https://localhost:5001/WeatherForecast`.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Selezionare **Esegui**  >  **Avvia debug** per avviare l'app. Visual Studio per Mac apre un browser e naviga all'indirizzo `https://localhost:<port>`, dove `<port>` è un numero di porta selezionato a caso. Viene restituito un errore HTTP 404 (Non trovato). Accodare `/WeatherForecast` all'URL (impostare l'URL su `https://localhost:<port>/WeatherForecast`).

---

Viene restituito un codice JSON simile al seguente:

```json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
```

## <a name="add-a-model-class"></a>Aggiungere una classe del modello

Un *modello* è un set di classi che rappresentano i dati gestiti dall'app. Il modello per questa app è una classe `TodoItem` singola.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto. Selezionare **Aggiungi**  >  **nuova cartella**. Assegnare un nome alla cartella *Models* .

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**. Assegnare alla classe il nome *TodoItem* e selezionare **Aggiungi**.

* Sostituire il codice del modello con il codice seguente:

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aggiungere una cartella denominata *Models* .

* Aggiungere una `TodoItem` classe alla *Models* cartella con il codice seguente:

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Fare clic con il pulsante destro del mouse sul progetto. Selezionare **Aggiungi**  >  **nuova cartella**. Assegnare un nome alla cartella *Models* .

  ![Nuova cartella](first-web-api-mac/_static/folder.png)

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi** > **nuovo file** > **generale** > **vuota classe**.

* Assegnare alla classe il nome *TodoItem* e fare clic su **Nuovo**.

* Sostituire il codice del modello con il codice seguente:

---

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoItem.cs?name=snippet)]

La proprietà `Id` funziona come chiave univoca in un database relazionale.

Le classi di modelli possono essere inserite in qualsiasi punto del progetto, ma la *Models* cartella viene utilizzata per convenzione.

## <a name="add-a-database-context"></a>Aggiungere un contesto di database

Il *contesto di database* è la classe principale che coordina le funzionalità di Entity Framework per un modello di dati. Questa classe viene creata mediante derivazione dalla classe `Microsoft.EntityFrameworkCore.DbContext`.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

### <a name="add-nuget-packages"></a>Aggiungere pacchetti NuGet

* Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Gestisci pacchetti NuGet per la soluzione**.
* Selezionare la scheda **Sfoglia** e quindi immettere **Microsoft. EntityFrameworkCore. InMemory** nella casella di ricerca.
* Selezionare **Microsoft. EntityFrameworkCore. InMemory** nel riquadro sinistro.
* Selezionare la casella di controllo **Progetto** nel riquadro di destra e quindi selezionare **Installa**.

![Gestione pacchetti NuGet](first-web-api/_static/vs3NuGet.png)

## <a name="add-the-todocontext-database-context"></a>Aggiungere il contesto del database TodoContext

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**. Assegnare alla classe il nome *TodoContext* e fare clic su **Aggiungi**.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

* Aggiungere una `TodoContext` classe alla *Models* cartella.

---

* Immettere il codice seguente:

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a>Registrare il contesto del database

In ASP.NET Core i servizi come il contesto del database devono essere registrati con il [contenitore di inserimento delle dipendenze](xref:fundamentals/dependency-injection). Il contenitore rende disponibile il servizio ai controller.

Aggiornare *Startup.cs* con il codice evidenziato seguente:

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Startup.cs?highlight=7-8,23-24&name=snippet_all)]

Il codice precedente:

* Rimuove le dichiarazioni `using` inutilizzate.
* Aggiunge il contesto del database al contenitore di inserimento delle dipendenze.
* Specifica che il contesto del database userà un database in memoria.

## <a name="scaffold-a-controller"></a>Eseguire lo scaffolding di un controller

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Fare clic con il pulsante destro del mouse sulla cartella *Controllers*.
* Selezionare **Aggiungi** > **Nuovo elemento di scaffolding**.
* Selezionare **Controller API con azioni, che usa Entity Framework** e quindi selezionare **Aggiungi**.
* Nella finestra di dialogo **Add API Controller with actions, using Entity Framework** (Aggiungi controller API con azioni, che usa Entity Framework):

  * Selezionare **TodoItem (TodoApi. Models )** nella **classe del modello**.
  * Selezionare **TodoContext (TodoApi. Models )** nella **classe del contesto dati**.
  * Selezionare **Aggiungi**.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Eseguire i comandi seguenti:

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet tool update -g Dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

I comandi precedenti:

* Aggiungere i pacchetti NuGet necessari per lo scaffolding.
* Installa il motore di scaffolding (`dotnet-aspnet-codegenerator`).
* Esegue lo scaffolding di `TodoItemsController`.

---

Il codice generato:

* Contrassegna la classe con l' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo. L'attributo indica che il controller risponde alle richieste di API Web. Per informazioni sui comportamenti specifici consentiti dall'attributo, vedere <xref:web-api/index>.
* Usa l'inserimento delle dipendenze per inserire il contesto del database (`TodoContext`) nel controller. Il contesto di database viene usato in ognuno dei metodi [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) nel controller.

I modelli di ASP.NET Core per:

* I controller con le visualizzazioni includono `[action]` nel modello di route.
* I controller API non includono `[action]` nel modello di route.

Quando il `[action]` token non è incluso nel modello di route, il nome dell' [azione](xref:mvc/controllers/routing#action) viene escluso dalla route. Ovvero, il nome del metodo associato all'azione non viene usato nella route corrispondente.

## <a name="examine-the-posttodoitem-create-method"></a>Esaminare il metodo di creazione PostTodoItem

Sostituire l'istruzione return in `PostTodoItem` per usare l'operatore [nameof](/dotnet/csharp/language-reference/operators/nameof):

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Create)]

Il codice precedente è un metodo HTTP POST, come indicato dall' [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attributo. Il metodo ottiene il valore dell'elemento attività dal corpo della richiesta HTTP.

Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).

Il metodo <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>:

* Restituisce un codice di stato HTTP 201 in caso di esito positivo. HTTP 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server.
* Aggiunge un'intestazione [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) alla risposta. L'intestazione `Location` specifica l'[URI](https://developer.mozilla.org/docs/Glossary/URI) dell'elemento attività appena creato. Per altre informazioni, vedere [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
* Fa riferimento all'azione `GetTodoItem` per creare l'URI dell'intestazione `Location`. La parola chiave `nameof` C# viene usata per evitare di impostare il nome dell'azione come hardcoded nella chiamata a `CreatedAtAction`.

### <a name="install-postman"></a>Installare Postman

Questa esercitazione usa Postman per testare l'API Web.

* Installa il [post](https://www.getpostman.com/downloads/)
* Avviare l'app Web.
* Avviare Postman.
* Disattivare **SSL certificate verification** (Verifica certificato SSL)
  * Da **File** > **Settings** (Impostazioni) (scheda **General** (Generale)) disabilitare **SSL certificate verification** (Verifica certificato SSL).
    > [!WARNING]
    > Riattivare la verifica dei certificati SSL al termine del test del controller.

<a name="post"></a>

### <a name="test-posttodoitem-with-postman"></a>Testare PostTodoItem con Postman

* Creare una nuova richiesta.
* Impostare il metodo HTTP su `POST`.
* Impostare l'URI su `https://localhost:<port>/api/TodoItems` . Ad esempio: `https://localhost:5001/api/TodoItems`.
* Selezionare la scheda **Corpo**.
* Selezionare il pulsante di opzione **raw** (non elaborato).
* Impostare il tipo su **JSON (application/json)**.
* Nel corpo della richiesta immettere codice JSON per un elemento attività:

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* Selezionare **Invia**.

  ![Postman con richiesta di creazione](first-web-api/_static/3/create.png)

### <a name="test-the-location-header-uri-with-postman"></a>Testare l'URI dell'intestazione Location con post

* Selezionare la scheda **Headers** (Intestazioni) nel riquadro **Response** (Risposta).
* Copiare il valore dell'intestazione **Location** (Posizione):

  ![Scheda Headers (Intestazioni) della console Postman](first-web-api/_static/3/create.png)

* Impostare il metodo HTTP su `GET`.
* Impostare l'URI su `https://localhost:<port>/api/TodoItems/1` . Ad esempio: `https://localhost:5001/api/TodoItems/1`.
* Selezionare **Invia**.

## <a name="examine-the-get-methods"></a>Esaminare i metodi GET

Questi metodi implementano due metodi GET:

* `GET /api/TodoItems`
* `GET /api/TodoItems/{id}`

Testare l'app chiamando i due endpoint da un browser o da Postman. Ad esempio:

* `https://localhost:5001/api/TodoItems`
* `https://localhost:5001/api/TodoItems/1`

Una risposta simile alla seguente viene generata dalla chiamata a `GetTodoItems`:

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

### <a name="test-get-with-postman"></a>Testare Get con Postman

* Creare una nuova richiesta.
* Impostare il metodo HTTP su **GET**.
* Impostare l'URI della richiesta su `https://localhost:<port>/api/TodoItems` . Ad esempio: `https://localhost:5001/api/TodoItems`.
* Impostare **Two pane view** (Visualizzazione in due riquadri) in Postman.
* Selezionare **Invia**.

Questa app usa un database in memoria. Se l'app viene arrestata e avviata, la richiesta GET precedente non restituirà alcun dato. Se non vengono restituiti dati, eseguire [POST](#post) per pubblicare i dati nell'app.

## <a name="routing-and-url-paths"></a>Routing e percorsi di URL

L' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo indica un metodo che risponde a una richiesta HTTP Get. Il percorso dell'URL per ogni metodo viene costruito nel modo seguente:

* Iniziare con la stringa di modello nell'attributo `Route` del controller:

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=TodoController&highlight=1)]

* Sostituire `[controller]` con il nome del controller, ovvero, per convenzione, il nome della classe controller meno il suffisso "Controller". In questo esempio il nome della classe controller è **TodoItems** Controller, quindi il nome del controller è "TodoItems". Il [routing](xref:mvc/controllers/routing) ASP.NET Core non fa distinzione tra maiuscole e minuscole.
* Se l'attributo `[HttpGet]` ha un modello di route, ad esempio `[HttpGet("products")]`, aggiungerlo al percorso. In questo esempio non si usa un modello. Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).

Nel metodo `GetTodoItem` seguente, `"{id}"` è una variabile segnaposto per l'identificatore univoco dell'elemento attività. Quando `GetTodoItem` viene richiamato, il valore di `"{id}"` nell'URL viene fornito al metodo nel `id` parametro.

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a>Valori restituiti 

Il tipo restituito dei `GetTodoItems` metodi e `GetTodoItem` è [ActionResult. \<T> ](xref:web-api/action-return-types#actionresultt-type) ASP.NET Core serializza automaticamente l'oggetto su [JSON](https://www.json.org/) e scrive il codice JSON nel corpo del messaggio di risposta. Il codice di risposta per questo tipo restituito è 200, presupponendo che non esistano eccezioni non gestite. Le eccezioni non gestite vengono convertite in errori 5xx.

I tipi restituiti `ActionResult` possono rappresentare un ampio intervallo di codici di stato HTTP. Ad esempio, `GetTodoItem` può restituire due valori di stato diversi:

* Se nessun elemento corrisponde all'ID richiesto, il metodo restituisce un <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> codice di errore 404.
* In caso contrario, il metodo restituisce il codice 200 con un corpo della risposta JSON. La restituzione di `item` risulta in una risposta HTTP 200.

## <a name="the-puttodoitem-method"></a>Metodo PutTodoItem

Esaminare il metodo `PutTodoItem`:

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Update)]

`PutTodoItem` è simile a `PostTodoItem` ma usa la richiesta HTTP PUT. La risposta è [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). In base alla specifica HTTP, una richiesta PUT richiede che il client invii l'intera entità aggiornata e non solo le modifiche. Per supportare gli aggiornamenti parziali, usare [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).

Se si riceve un errore chiamando `PutTodoItem`, chiamare `GET` per verificare che il database includa un elemento.

### <a name="test-the-puttodoitem-method"></a>Testare il metodo PutTodoItem

Questo esempio usa un database in memoria che deve essere inizializzato ogni volta che l'app viene avviata. Deve esistere un elemento nel database prima di eseguire una chiamata PUT. Chiamare GET per assicurarsi che sia presente un elemento nel database prima di effettuare una chiamata PUT.

Aggiornare l'attività con ID = 1 e impostarne il nome su "feed Fish":

```json
  {
    "id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

L'immagine seguente visualizza l'aggiornamento di Postman:

![Console Postman con risposta 204 (No Content)](first-web-api/_static/3/pmcput.png)

## <a name="the-deletetodoitem-method"></a>Metodo DeleteTodoItem

Esaminare il metodo `DeleteTodoItem`:

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Delete)]

### <a name="test-the-deletetodoitem-method"></a>Testare il metodo DeleteTodoItem

Usare Postman per eliminare un elemento attività:

* Impostare il metodo su `DELETE`.
* Impostare l'URI dell'oggetto da eliminare (ad esempio `https://localhost:5001/api/TodoItems/1` ).
* Selezionare **Invia**.

<a name="over-post"></a>
<a name="over-post-v3"></a>

## <a name="prevent-over-posting"></a>Impedisci overposting

Attualmente l'app di esempio espone l'intero `TodoItem` oggetto. Le app di produzione limitano in genere i dati di input e restituiti usando un subset del modello. Esistono diversi motivi alla base di questo e la sicurezza è una delle principali. Il subset di un modello è in genere indicato come oggetto Trasferimento dati (DTO), modello di input o modello di visualizzazione. **Dto** viene usato in questo articolo.

Un DTO può essere usato per:

* Impedisci l'overposting.
* Nascondere le proprietà che i client non dovrebbero visualizzare.
* Omettere alcune proprietà per ridurre le dimensioni del payload.
* Rendere flat gli oggetti grafici che contengono oggetti annidati. I grafici a oggetti flat possono essere più pratici per i client.

Per illustrare l'approccio DTO, aggiornare la `TodoItem` classe in modo da includere un campo Secret:

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Models/TodoItem.cs?name=snippet&highlight=6)]

Il campo segreto deve essere nascosto da questa app, ma un'app amministrativa può scegliere di esporla.

Verificare che sia possibile pubblicare e ottenere il campo secret.

Creare un modello DTO:

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Models/TodoItemDTO.cs?name=snippet)]

Aggiornare `TodoItemsController` da usare `TodoItemDTO` :

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Controllers/TodoItemsController.cs?name=snippet)]

Verificare che non sia possibile inserire o ottenere il campo secret.

## <a name="call-the-web-api-with-javascript"></a>Chiamare l'API Web con JavaScript

Vedere [esercitazione: chiamare un'API web ASP.NET Core con JavaScript](xref:tutorials/web-api-javascript).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

In questa esercitazione verranno illustrate le procedure per:

> [!div class="checklist"]
> * Creare un progetto di API Web.
> * Aggiungere una classe modello e un contesto di database.
> * Aggiungere un controller.
> * Aggiungere metodi CRUD.
> * Configurare routing e percorsi URL.
> * Specificare valori restituiti.
> * Chiamare l'API Web con Postman.
> * Chiamare l'API Web con JavaScript.

Al termine si dispone di un'API web che può gestire gli elementi di tipo "attività" archiviati in un database relazionale.

## <a name="overview-21"></a>Panoramica 2,1

Questa esercitazione consente di creare l'API seguente:

|API | Descrizione | Testo della richiesta | Corpo della risposta |
|--- | ---- | ---- | ---- |
|GET /api/TodoItems | Ottiene tutti gli elementi attività | nessuno | Matrice di elementi attività|
|GET /api/TodoItems/{id} | Ottiene un elemento in base all'ID | nessuno | Elemento attività|
|POST /api/TodoItems | Aggiunge un nuovo elemento | Elemento attività | Elemento attività |
|PUT /api/TodoItems/{id} | Aggiorna un elemento esistente &nbsp; | Elemento attività | nessuno |
|Elimina/api/TodoItems/{id} &nbsp;&nbsp; | Elimina un elemento &nbsp;&nbsp; | nessuno | nessuno|

Il diagramma seguente visualizza la struttura dell'app.

![Il client è rappresentato da una casella a sinistra. Invia una richiesta e riceve una risposta dall'applicazione, una casella disegnata a destra. Nella riquadro dell'applicazione tre caselle rappresentano il controller, il modello e il livello di accesso ai dati. La richiesta viene ricevuta dal controller dell'applicazione e vengono eseguite operazioni di lettura/scrittura tra il controller e il livello di accesso ai dati. Il modello viene serializzato e restituito al client nella risposta.](first-web-api/_static/architecture.png)

## <a name="prerequisites-21"></a>Prerequisiti 2,1

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

## <a name="create-a-web-project-21"></a>Creare un progetto Web 2,1

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Scegliere **nuovo** progetto dal menu **file** > .
* Selezionare il modello **Applicazione Web ASP.NET Core** e fare clic su **Avanti**.
* Assegnare al progetto il nome *TodoApi* e fare clic su **Crea**.
* Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.NET Core** e **ASP.NET Core 2.2**. Selezionare il modello **API** e fare clic su **Crea**. **Non** selezionare **Abilita supporto Docker**.

![Finestra di dialogo Nuovo progetto di Visual Studio](first-web-api/_static/vs.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Cambiare directory (`cd`) e passare alla cartella che conterrà la cartella del progetto.
* Eseguire i comandi seguenti:

   ```dotnetcli
   dotnet new webapi -o TodoApi
   code -r TodoApi
   ```

  Questi comandi creano un nuovo progetto API Web e aprono una nuova istanza di Visual Studio Code nella nuova cartella di progetto.

* Quando una finestra di dialogo chiede se si vuole aggiungere gli asset necessari al progetto, selezionare **Sì**.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Selezionare **File** > **Nuova soluzione**.

  ![Nuova soluzione macOS](first-web-api-mac/_static/sln.png)

* In Visual Studio per Mac precedente alla versione 8,6 Selezionare API **app .NET Core**  >    >    >  **Avanti**. Nella versione 8,6 o successive selezionare API **app Web e console**  >    >    >  **Avanti**.
  
* Nella finestra di dialogo **Configura nuova ASP.NET Core API Web** selezionare il **Framework di destinazione**.NET Core 2. x più recente. Selezionare **Avanti**.

* Immettere *TodoApi* in **Nome progetto**, quindi selezionare **Crea**.

  ![Finestra di dialogo di configurazione](first-web-api-mac/_static/2.png)

---

### <a name="test-the-api-21"></a>Testare l'API 2,1

Il modello di progetto crea un'API `values`. Chiamare il metodo `Get` da un browser per eseguire il test dell'app.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Premere CTRL+F5 per eseguire l'app. Visual Studio apre un browser e naviga all'indirizzo `https://localhost:<port>/api/values`, dove `<port>` è un numero di porta selezionato a caso.

Se in una finestra di dialogo viene chiesto se considerare attendibile il certificato IIS Express, selezionare **Sì**. Nella finestra di dialogo **Avviso di sicurezza** successiva selezionare **Sì**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Premere CTRL+F5 per eseguire l'app. In un browser passare all'URL seguente: `https://localhost:5001/api/values`.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Selezionare **Esegui**  >  **Avvia debug** per avviare l'app. Visual Studio per Mac apre un browser e naviga all'indirizzo `https://localhost:<port>`, dove `<port>` è un numero di porta selezionato a caso. Viene restituito un errore HTTP 404 (Non trovato). Accodare `/api/values` all'URL (impostare l'URL su `https://localhost:<port>/api/values`).

---

Viene restituito il codice JSON seguente:

```json
["value1","value2"]
```

## <a name="add-a-model-class-21"></a>Aggiungere una classe modello 2,1

Un *modello* è un set di classi che rappresentano i dati gestiti dall'app. Il modello per questa app è una classe `TodoItem` singola.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto. Selezionare **Aggiungi**  >  **nuova cartella**. Assegnare un nome alla cartella *Models* .

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**. Assegnare alla classe il nome *TodoItem* e selezionare **Aggiungi**.

* Sostituire il codice del modello con il codice seguente:

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aggiungere una cartella denominata *Models* .

* Aggiungere una `TodoItem` classe alla *Models* cartella con il codice seguente:

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Fare clic con il pulsante destro del mouse sul progetto. Selezionare **Aggiungi**  >  **nuova cartella**. Assegnare un nome alla cartella *Models* .

  ![Nuova cartella](first-web-api-mac/_static/folder.png)

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi** > **nuovo file** > **generale** > **vuota classe**.

* Assegnare alla classe il nome *TodoItem* e fare clic su **Nuovo**.

* Sostituire il codice del modello con il codice seguente:

---

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoItem.cs)]

La proprietà `Id` funziona come chiave univoca in un database relazionale.

Le classi di modelli possono essere inserite in qualsiasi punto del progetto, ma la *Models* cartella viene utilizzata per convenzione.

## <a name="add-a-database-context-21"></a>Aggiungere un contesto di database 2,1

Il *contesto di database* è la classe principale che coordina le funzionalità di Entity Framework per un modello di dati. Questa classe viene creata mediante derivazione dalla classe `Microsoft.EntityFrameworkCore.DbContext`.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**. Assegnare alla classe il nome *TodoContext* e fare clic su **Aggiungi**.

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

* Aggiungere una `TodoContext` classe alla *Models* cartella.

---

* Sostituire il codice del modello con il codice seguente:

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context-21"></a>Registrare il contesto del database 2,1

In ASP.NET Core i servizi come il contesto del database devono essere registrati con il [contenitore di inserimento delle dipendenze](xref:fundamentals/dependency-injection). Il contenitore rende disponibile il servizio ai controller.

Aggiornare *Startup.cs* con il codice evidenziato seguente:

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup1.cs?highlight=5,8,25-26&name=snippet_all)]

Il codice precedente:

* Rimuove le dichiarazioni `using` inutilizzate.
* Aggiunge il contesto del database al contenitore di inserimento delle dipendenze.
* Specifica che il contesto del database userà un database in memoria.

## <a name="add-a-controller-21"></a>Aggiungere un controller 2,1

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Fare clic con il pulsante destro del mouse sulla cartella *Controllers*.
* Selezionare **Aggiungi** > **nuovo elemento**.
* Nella finestra di dialogo **Aggiungi nuovo elemento** selezionare il modello **API Controller Class** (Classe controller API).
* Assegnare alla classe il nome *TodoController* e selezionare **Aggiungi**.

  ![Finestra di dialogo Aggiungi nuovo elemento con controller nella casella di ricerca e controller API Web selezionato](first-web-api/_static/new_controller.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

* Nella cartella *Controllers* creare una classe denominata `TodoController`.

---

* Sostituire il codice del modello con il codice seguente:

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController2.cs?name=snippet_todo1)]

Il codice precedente:

* Definisce una classe controller API senza metodi.
* Contrassegna la classe con l' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo. L'attributo indica che il controller risponde alle richieste di API Web. Per informazioni sui comportamenti specifici consentiti dall'attributo, vedere <xref:web-api/index>.
* Usa l'inserimento delle dipendenze per inserire il contesto del database (`TodoContext`) nel controller. Il contesto di database viene usato in ognuno dei metodi [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) nel controller.
* Aggiunge un elemento denominato `Item1` al database se il database è vuoto. Questo codice si trova nel costruttore, quindi viene eseguito ogni volta che è presente una nuova richiesta HTTP. Se si eliminano tutti gli elementi, il costruttore crea di nuovo `Item1` quando viene nuovamente chiamato un metodo API. Potrebbe pertanto sembrare che l'eliminazione non abbia funzionato, mentre di fatto ha funzionato.

## <a name="add-get-methods-21"></a>Aggiungi metodi Get 2,1

Per specificare un'API che recupera elementi attività, aggiungere i metodi seguenti alla classe `TodoController`:

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetAll)]

Questi metodi implementano due metodi GET:

* `GET /api/todo`
* `GET /api/todo/{id}`

Arrestare l'app se è ancora in esecuzione. Quindi eseguirla di nuovo per includere le modifiche più recenti.

Testare l'app chiamando i due endpoint da un browser. Ad esempio:

* `https://localhost:<port>/api/todo`
* `https://localhost:<port>/api/todo/1`

La chiamata a `GetTodoItems` genera la risposta HTTP seguente:

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

## <a name="routing-and-url-paths-21"></a>Percorsi URL e routing 2,1

L' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo indica un metodo che risponde a una richiesta HTTP Get. Il percorso dell'URL per ogni metodo viene costruito nel modo seguente:

* Iniziare con la stringa di modello nell'attributo `Route` del controller:

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=TodoController&highlight=3)]

* Sostituire `[controller]` con il nome del controller, ovvero, per convenzione, il nome della classe controller meno il suffisso "Controller". In questo esempio il nome della classe controller è **Todo** Controller, pertanto il nome del controller è "todo". Il [routing](xref:mvc/controllers/routing) ASP.NET Core non fa distinzione tra maiuscole e minuscole.
* Se l'attributo `[HttpGet]` ha un modello di route, ad esempio `[HttpGet("products")]`, aggiungerlo al percorso. In questo esempio non si usa un modello. Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).

Nel metodo `GetTodoItem` seguente, `"{id}"` è una variabile segnaposto per l'identificatore univoco dell'elemento attività. Quando viene chiamato `GetTodoItem`, il valore di `"{id}"` viene specificato per il metodo nel rispettivo parametro `id`.

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values-21"></a>Valori restituiti 2,1

Il tipo restituito dei `GetTodoItems` metodi e `GetTodoItem` è [ActionResult. \<T> ](xref:web-api/action-return-types#actionresultt-type) ASP.NET Core serializza automaticamente l'oggetto su [JSON](https://www.json.org/) e scrive il codice JSON nel corpo del messaggio di risposta. Il codice di risposta per questo tipo restituito è 200, presupponendo che non esistano eccezioni non gestite. Le eccezioni non gestite vengono convertite in errori 5xx.

I tipi restituiti `ActionResult` possono rappresentare un ampio intervallo di codici di stato HTTP. Ad esempio, `GetTodoItem` può restituire due valori di stato diversi:

* Se nessun elemento corrisponde all'ID richiesto, il metodo restituisce un <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> codice di errore 404.
* In caso contrario, il metodo restituisce il codice 200 con un corpo della risposta JSON. La restituzione di `item` risulta in una risposta HTTP 200.

## <a name="test-the-gettodoitems-method-21"></a>Testare il metodo GetTodoItems 2,1

Questa esercitazione usa Postman per testare l'API Web.

* Installare [Postman](https://www.getpostman.com/downloads/).
* Avviare l'app Web.
* Avviare Postman.
* Disabilitare la **Verifica del certificato SSL**.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Da **File** > **Settings** (Impostazioni) (scheda **General** (Generale)) disabilitare **SSL certificate verification** (Verifica certificato SSL).

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

* Dalle preferenze del **poster**  >   (scheda **generale** ) disabilitare la **Verifica del certificato SSL**. In alternativa, selezionare la chiave inglese e selezionare **Settings** (Impostazioni), quindi disabilitare la verifica del certificato SSL.

---
  
> [!WARNING]
> Riattivare la verifica dei certificati SSL al termine del test del controller.

* Creare una nuova richiesta.
  * Impostare il metodo HTTP su **GET**.
  * Impostare l'URI della richiesta su `https://localhost:<port>/api/todo` . Ad esempio: `https://localhost:5001/api/todo`.
* Impostare **Two pane view** (Visualizzazione in due riquadri) in Postman.
* Selezionare **Invia**.

![Postman con richiesta GET](first-web-api/_static/2pv.png)

## <a name="add-a-create-method-21"></a>Aggiungere un metodo Create 2,1

Aggiungere il metodo `PostTodoItem` seguente in *Controllers/TodoController.cs*: 

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Create)]

Il codice precedente è un metodo HTTP POST, come indicato dall' [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attributo. Il metodo ottiene il valore dell'elemento attività dal corpo della richiesta HTTP.

Il metodo `CreatedAtAction`:

* Restituisce un codice di stato HTTP 201 in caso di esito positivo. HTTP 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server.
* Aggiunge un'intestazione `Location` alla risposta. L'intestazione `Location` specifica l'URI dell'elemento attività appena creato. Per altre informazioni, vedere [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
* Fa riferimento all'azione `GetTodoItem` per creare l'URI dell'intestazione `Location`. La parola chiave `nameof` C# viene usata per evitare di impostare il nome dell'azione come hardcoded nella chiamata a `CreatedAtAction`.

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

### <a name="test-the-posttodoitem-method-21"></a>Testare il metodo PostTodoItem 2,1

* Compilare il progetto.
* In Postman impostare il metodo HTTP su `POST`.
* Impostare l'URI su `https://localhost:<port>/api/Todo` . Ad esempio: `https://localhost:5001/api/Todo`.
* Selezionare la scheda **Corpo**.
* Selezionare il pulsante di opzione **raw** (non elaborato).
* Impostare il tipo su **JSON (application/json)**.
* Nel corpo della richiesta immettere codice JSON per un elemento attività:

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* Selezionare **Invia**.

  ![Postman con richiesta di creazione](first-web-api/_static/create.png)

  Se viene visualizzato un errore HTTP 405 - Metodo non consentito, è probabile che il progetto non sia stato compilato dopo l'aggiunta del metodo `PostTodoItem`.

### <a name="test-the-location-header-uri-21"></a>Testare l'URI dell'intestazione Location 2,1

* Selezionare la scheda **Headers** (Intestazioni) nel riquadro **Response** (Risposta).
* Copiare il valore dell'intestazione **Location** (Posizione):

  ![Scheda Headers (Intestazioni) della console Postman](first-web-api/_static/pmc2.png)

* Impostare il metodo su GET.
* Impostare l'URI su `https://localhost:<port>/api/TodoItems/2` . Ad esempio: `https://localhost:5001/api/TodoItems/2`.
* Selezionare **Invia**.

## <a name="add-a-puttodoitem-method-21"></a>Aggiungere un metodo PutTodoItem 2,1

Aggiungere il metodo `PutTodoItem` seguente:

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Update)]

`PutTodoItem` è simile a `PostTodoItem` ma usa la richiesta HTTP PUT. La risposta è [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). In base alla specifica HTTP, una richiesta PUT richiede che il client invii l'intera entità aggiornata e non solo le modifiche. Per supportare gli aggiornamenti parziali, usare [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).

Se si riceve un errore chiamando `PutTodoItem`, chiamare `GET` per verificare che il database includa un elemento.

### <a name="test-the-puttodoitem-method-21"></a>Testare il metodo PutTodoItem 2,1

Questo esempio usa un database in memoria che deve essere inizializzato ogni volta che l'app viene avviata. Deve esistere un elemento nel database prima di eseguire una chiamata PUT. Chiamare GET per assicurarsi che sia presente un elemento nel database prima di effettuare una chiamata PUT.

Aggiornare l'attività con ID = 1 e impostarne il nome su "feed Fish":

```json
  {
    "id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

L'immagine seguente visualizza l'aggiornamento di Postman:

![Console Postman con risposta 204 (No Content)](first-web-api/_static/pmcput.png)

## <a name="add-a-deletetodoitem-method-21"></a>Aggiungere un metodo DeleteTodoItem 2,1

Aggiungere il metodo `DeleteTodoItem` seguente:

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Delete)]

La `DeleteTodoItem` risposta è [204 (nessun contenuto)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).

### <a name="test-the-deletetodoitem-method-21"></a>Testare il metodo DeleteTodoItem 2,1

Usare Postman per eliminare un elemento attività:

* Impostare il metodo su `DELETE`.
* Impostare l'URI dell'oggetto da eliminare (ad esempio, `https://localhost:5001/api/todo/1` ).
* Selezionare **Invia**.

L'app di esempio consente di eliminare tutti gli elementi. Quando viene eliminato l'ultimo elemento, tuttavia, ne viene creato uno nuovo dal costruttore della classe modello alla successiva chiamata dell'API.

## <a name="call-the-web-api-with-javascript-21"></a>Chiamare l'API Web con JavaScript 2,1

In questa sezione viene aggiunta una pagina HTML che usa JavaScript per chiamare l'API Web. jQuery avvia la richiesta. JavaScript aggiorna la pagina con i dettagli della risposta dell'API Web.

Configurare l'app per [servire file statici](xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A) e [abilitare il mapping del file predefinito](xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A) aggiornando *Startup.cs* con il codice evidenziato seguente:

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup.cs?highlight=14-15&name=snippet_configure)]

Creare una cartella *wwwroot* nella directory del progetto.

Aggiungere un file HTML denominato *index.html* alla directory *wwwroot*. Sostituire il contenuto con il markup seguente:

[!code-html[](first-web-api/samples/2.2/TodoApi/wwwroot/index.html)]

Aggiungere un file JavaScript con nome *site.js* alla directory *wwwroot*. Sostituirne il contenuto con il codice seguente:

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_SiteJs)]

Può essere necessario modificare le impostazioni di avvio del progetto ASP.NET Core per il test della pagina HTML in locale:

* Aprire *Properties\launchSettings.json*.
* Rimuovere la `launchUrl` proprietà per forzare l'apertura dell'app a *index.html* &mdash; file predefinito del progetto.

In questo esempio vengono chiamati tutti i metodi CRUD dell'API Web. Di seguito sono incluse le spiegazioni delle chiamate all'API.

### <a name="get-a-list-of-to-do-items-21"></a>Ottenere un elenco di elementi attività 2,1

jQuery invia una richiesta HTTP GET all'API Web, che restituisce JSON che rappresenta una matrice di elementi attività. La funzione di callback `success` viene chiamata se la richiesta ha esito positivo. Nel callback il modello DOM viene aggiornato con le informazioni dell'elemento attività.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_GetData)]

### <a name="add-a-to-do-item-21"></a>Aggiungere un'attività 2,1

jQuery invia una richiesta HTTP POST con l'elemento attività nel corpo della richiesta. Le opzioni `accepts` e `contentType` sono impostate su `application/json` per specificare il tipo di supporto ricevuto e inviato. L'elemento attività viene convertito in JSON usando [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify). Quando l'API restituisce un codice di stato di esecuzione riuscita, viene chiamata la funzione `getData` per aggiornare la tabella HTML.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AddItem)]

### <a name="update-a-to-do-item-21"></a>Aggiornare un'attività 2,1

L'aggiornamento di un elemento attività è simile all'aggiunta di un elemento di questo tipo. L'`url` cambia per includere l'identificatore univoco dell'elemento e `type` è `PUT`.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxPut)]

### <a name="delete-a-to-do-item-21"></a>Eliminare un'attività 2,1

È possibile eliminare un elemento attività impostando `type` su `DELETE` nella chiamata AJAX e specificando l'identificatore univoco dell'elemento nell'URL.

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxDelete)]

::: moniker-end

<a name="auth"></a>

## <a name="add-authentication-support-to-a-web-api-21"></a>Aggiungere il supporto per l'autenticazione a un'API Web 2,1

[!INCLUDE[](~/includes/IdentityServer4.md)]

## <a name="additional-resources-21"></a>Risorse aggiuntive 2,1

[Visualizzare o scaricare il codice di esempio per questa esercitazione](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/first-web-api/samples). Vedere [come scaricare un esempio](xref:index#how-to-download-a-sample).

Per altre informazioni, vedere le risorse seguenti:

* <xref:web-api/index>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:data/ef-rp/intro>
* <xref:mvc/controllers/routing>
* <xref:web-api/action-return-types>
* <xref:host-and-deploy/azure-apps/index>
* <xref:host-and-deploy/index>
* [Versione YouTube dell'esercitazione](https://www.youtube.com/watch?v=TTkhEyGBfAk)
* [Microsoft Learn: creare un'API Web con ASP.NET Core](/learn/modules/build-web-api-aspnet-core/)
