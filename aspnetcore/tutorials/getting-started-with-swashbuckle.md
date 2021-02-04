---
title: Introduzione a Swashbuckle e ad ASP.NET Core
author: zuckerthoben
description: Informazioni su come aggiungere Swashbuckle al progetto dell'API Web ASP.NET Core per integrare l'interfaccia utente di Swagger.
ms.author: scaddie
ms.custom: mvc
ms.date: 06/26/2020
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
uid: tutorials/get-started-with-swashbuckle
ms.openlocfilehash: 68601333cbd49a1d667337021de246a73f536600
ms.sourcegitcommit: c1839f2992b003c92cd958244a2e0771ae928786
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/04/2021
ms.locfileid: "99551650"
---
# <a name="get-started-with-swashbuckle-and-aspnet-core"></a>Introduzione a Swashbuckle e ad ASP.NET Core

Di [Shayne Boyer](https://twitter.com/spboyer) e [Scott Addie](https://twitter.com/Scott_Addie)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/web-api-help-pages-using-swagger/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

Esistono tre componenti principali di Swashbuckle:

* [Swashbuckle.AspNetCore.Swagger](https://www.nuget.org/packages/Swashbuckle.AspNetCore.Swagger/): un modello a oggetti Swagger e il middleware per esporre oggetti `SwaggerDocument` come endpoint JSON.

* [Swashbuckle.AspNetCore.SwaggerGen](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerGen/): un generatore Swagger che compila oggetti `SwaggerDocument` direttamente da route, controller e modelli. È usato in genere con il middleware di endpoint di Swagger per esporre automaticamente i file JSON di Swagger.

* [Swashbuckle.AspNetCore.SwaggerUI](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerUI/): una versione incorporata dello strumento interfaccia utente di Swagger, Interpreta i file JSON di Swagger per creare un'esperienza avanzata e personalizzabile per descrivere le funzionalità delle API Web. Include test harness predefiniti per i metodi pubblici.

## <a name="package-installation"></a>Installazione del pacchetto

È possibile aggiungere Swashbuckle con gli approcci seguenti:

### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dalla finestra **Console di Gestione pacchetti**:
  * Passare a **Visualizza**  >  **altre**  >  **console di gestione pacchetti** di Windows
  * Passare alla directory che contiene il file *TodoApi.csproj*
  * Eseguire il comando seguente:

    ```powershell
    Install-Package Swashbuckle.AspNetCore -Version 5.6.3
    ```

* Dalla finestra di dialogo **Gestisci pacchetti NuGet**:
  * Fare clic con il pulsante destro del mouse sul progetto in **Esplora soluzioni**  >  **Gestisci pacchetti NuGet**
  * Impostare **Origine pacchetto** su "nuget.org"
  * Verificare che l'opzione "Includi versione preliminare" sia abilitata
  * Immettere "Swashbuckle.AspNetCore" nella casella di ricerca
  * Selezionare il pacchetto "Swashbuckle.AspNetCore" più recente nella scheda **Sfoglia** e fare clic su **Installa**

### <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Fare clic su con il pulsante destro del mouse sulla cartella *Pacchetti* in **Solution Pad** (Riquadro della soluzione) > **Aggiungi pacchetti**
* Impostare l'elenco a discesa **Aggiungi pacchetti** della finestra **Origine** su "nuget.org"
* Verificare che l'opzione "Mostra pacchetti di versioni non definitive" sia abilitata
* Immettere "Swashbuckle.AspNetCore" nella casella di ricerca
* Selezionare il pacchetto "Swashbuckle.AspNetCore" più recente nel riquadro dei risultati e fare clic su **Aggiungi pacchetto**

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Eseguire il comando seguente da **Terminale integrato**:

```dotnetcli
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore -v 5.6.3
```

### <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Eseguire il comando seguente:

```dotnetcli
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore -v 5.6.3
```

---

## <a name="add-and-configure-swagger-middleware"></a>Aggiungere e configurare il middleware di Swagger

Aggiungere il generatore di Swagger alla raccolta di servizi nel metodo `Startup.ConfigureServices`:

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=8)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=9)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=8)]

::: moniker-end

Nel metodo `Startup.Configure` abilitare il middleware per la gestione del documento JSON generato e l'interfaccia utente di Swagger:

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_Configure&highlight=4,8-11)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_Configure&highlight=4,8-11)]

::: moniker-end

> [!NOTE]
> Swashbuckle si basa su MVC <xref:Microsoft.AspNetCore.Mvc.ApiExplorer> per individuare le route e gli endpoint. Se il progetto chiama <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc%2A> , le route e gli endpoint vengono individuati automaticamente. Quando <xref:Microsoft.Extensions.DependencyInjection.MvcCoreServiceCollectionExtensions.AddMvcCore%2A> si chiama, il <xref:Microsoft.Extensions.DependencyInjection.MvcApiExplorerMvcCoreBuilderExtensions.AddApiExplorer%2A> metodo deve essere chiamato in modo esplicito. Per ulteriori informazioni, vedere [Swashbuckle, ApiExplorer e routing](https://github.com/domaindrivendev/Swashbuckle.AspNetCore#swashbuckle-apiexplorer-and-routing).

La chiamata del metodo `UseSwaggerUI` precedente abilita il [middleware dei file statici](xref:fundamentals/static-files). Se la destinazione è .NET Framework o .NET Core 1. x, aggiungere il pacchetto NuGet [Microsoft. AspNetCore. staticfiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles/) al progetto.

Avviare l'app e passare a `http://localhost:<port>/swagger/v1/swagger.json`. Il documento generato che descrive gli endpoint viene visualizzato come illustrato nella [specifica openapi (openapi.json)](xref:tutorials/web-api-help-pages-using-swagger#openapi-specification-openapijson).

L'interfaccia utente di Swagger è disponibile in `http://localhost:<port>/swagger`. Esplorare l'API tramite l'interfaccia utente di Swagger e incorporarla in altri programmi.

> [!TIP]
> Per gestire l'interfaccia utente di Swagger nella radice dell'app (`http://localhost:<port>/`), impostare la proprietà `RoutePrefix` su una stringa vuota:
>
> [!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup3.cs?name=snippet_UseSwaggerUI&highlight=4)]

Se si usano le directory con ISS o il proxy inverso, impostare l'endpoint Swagger su un percorso relativo tramite il prefisso `./`. Ad esempio: `./swagger/v1/swagger.json`. L'uso di `/swagger/v1/swagger.json` indica all'app di cercare il file JSON nella vera radice dell'URL (con il prefisso della route, se usato). Usare, ad esempio, `http://localhost:<port>/<route_prefix>/swagger/v1/swagger.json` invece di `http://localhost:<port>/<virtual_directory>/<route_prefix>/swagger/v1/swagger.json`.

> [!NOTE]
> Per impostazione predefinita, Swashbuckle genera ed espone il codice JSON di spavalderia nella versione 3,0 della specifica, &mdash; denominata ufficialmente openapi Specification. Per supportare la compatibilità con le versioni precedenti, è invece possibile scegliere di esporre JSON in formato 2,0. Questo formato 2,0 è importante per le integrazioni, ad esempio Microsoft Power Apps e Microsoft Flow che attualmente supportano OpenAPI versione 2,0. Per acconsentire esplicitamente al formato 2,0, impostare la `SerializeAsV2` Proprietà in `Startup.Configure` :
>
> [!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup3.cs?name=snippet_Configure&highlight=4-7)]

## <a name="customize-and-extend"></a>Personalizzare ed estendere

Swagger offre opzioni per documentare il modello a oggetti e personalizzare l'interfaccia utente in modo che corrisponda al tema.

Nella classe `Startup` aggiungere gli spazi dei nomi seguenti:

```csharp
using System;
using System.Reflection;
using System.IO;
```

### <a name="api-info-and-description"></a>Informazioni e descrizione API

L'azione di configurazione passata al metodo `AddSwaggerGen` aggiunge informazioni come ad esempio autore, licenza e descrizione:

Nella classe `Startup` importare gli spazi dei nomi seguenti per usare la classe `OpenApiInfo`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_InfoClassNamespace)]

Utilizzando la `OpenApiInfo` classe, modificare le informazioni visualizzate nell'interfaccia utente:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup4.cs?name=snippet_AddSwaggerGen)]

L'interfaccia utente di Swagger visualizza le informazioni sulla versione:

![Interfaccia utente di Swagger con informazioni sulla versione: descrizione, autore e altri collegamenti](web-api-help-pages-using-swagger/_static/custom-info.png)

### <a name="xml-comments"></a>Commenti in XML

I commenti XML possono essere abilitati con gli approcci seguenti:

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-2.0"

* Fare clic con il pulsante destro del mouse sul progetto in **Esplora soluzioni** e scegliere **Modifica <nome_progetto>.csproj**.
* Aggiungere manualmente le righe evidenziate al file con estensione *csproj*:

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* Fare clic con il pulsante destro del mouse sul progetto in **Esplora soluzioni** e scegliere **Proprietà**.
* Controllare la casella **file di documentazione XML** nella sezione **output** della scheda **compilazione** .

::: moniker-end

#### <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

::: moniker range=">= aspnetcore-2.0"

* Dal *riquadro della soluzione* premere **controllo** e fare clic sul nome del progetto. Passare a **strumenti**  >  **modifica file**.
* Aggiungere manualmente le righe evidenziate al file con estensione *csproj*:

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* Aprire la finestra di dialogo **Opzioni progetto** > **Compila** > **Compilatore**.
* Selezionare la casella **Genera documentazione XML** nella sezione **Opzioni generali**

::: moniker-end

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Aggiungere manualmente le righe evidenziate al file con estensione *csproj*:

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

#### <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Aggiungere manualmente le righe evidenziate al file con estensione *csproj*:

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

---

Abilitando i commenti XML, viene eseguito il debug delle informazioni per membri e tipi pubblici non documentati. I membri e tipi non documentati sono indicati nel messaggio di avviso. Ad esempio, il messaggio seguente indica una violazione del codice di avviso 1591:

```text
warning CS1591: Missing XML comment for publicly visible type or member 'TodoController.GetAll()'
```

Per eliminare gli avvisi per l'intero progetto, definire un elenco delimitato da punto e virgola di codici di avviso da ignorare nel file di progetto. L'aggiunta di codici di avviso a `$(NoWarn);` applica anche i [valori predefiniti di C#](https://github.com/dotnet/sdk/blob/2eb6c546931b5bcb92cd3128b93932a980553ea1/src/Tasks/Microsoft.NET.Build.Tasks/targets/Microsoft.NET.Sdk.CSharp.props#L16).

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=3)]

::: moniker-end

Per eliminare gli avvisi solo per membri specifici, racchiudere il codice in direttive del preprocessore [#pragma warning](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-pragma-warning). Questo approccio è utile per il codice che non deve essere esposto tramite la documentazione API. Nell'esempio seguente il codice di avviso CS1591 viene ignorato per l'intera `Program` classe. L'imposizione del codice di avviso viene ripristinata alla chiusura della definizione della classe. Specificare più codici di avviso con un elenco delimitato da virgole.

```csharp
namespace TodoApi
{
#pragma warning disable CS1591
    public class Program
    {
        public static void Main(string[] args) =>
            BuildWebHost(args).Run();

        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .Build();
    }
#pragma warning restore CS1591
}
```

Configurare Swagger per usare il file XML che viene generato con le istruzioni precedenti. Per Linux o sistemi operativi diversi da Windows, i percorsi e i nomi di file possono fare distinzione tra maiuscole e minuscole. Ad esempio, un file *ToDoApi.XML* è valido in Windows, ma non in CentOS.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=31-33)]

::: moniker-end

::: moniker range="= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup1x.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

Nel codice precedente, la [Reflection](/dotnet/csharp/programming-guide/concepts/reflection) viene usata per compilare un nome file XML corrispondente a quello del progetto API Web. La proprietà [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory*) viene usata per costruire un percorso del file XML. Alcune funzionalità di Swagger (ad esempio, schemi di parametri di input o i metodi HTTP e i codici di risposta dai rispettivi attributi) funzionano senza l'uso di un file di documentazione XML. Per la maggior parte delle funzionalità, vale a dire i riepiloghi dei metodi e le descrizioni dei parametri e dei codici di risposta, l'uso di un file XML è obbligatorio.

Se si aggiungono commenti con barra tripla a un'azione si migliora Swagger UI aggiungendo la descrizione all'intestazione della sezione. Aggiungere un [\<summary>](/dotnet/csharp/programming-guide/xmldoc/summary) elemento sopra l' `Delete` azione:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Delete&highlight=1-3)]

L'interfaccia utente di Swagger visualizza il testo interno dell'elemento `<summary>` del codice precedente:

![Interfaccia utente di Swagger che visualizza il commento XML "Deletes a specific TodoItem." per il metodo DELETE](web-api-help-pages-using-swagger/_static/triple-slash-comments.png)

L'interfaccia utente è determinata dallo schema JSON generato:

```json
"delete": {
    "tags": [
        "Todo"
    ],
    "summary": "Deletes a specific TodoItem.",
    "operationId": "ApiTodoByIdDelete",
    "consumes": [],
    "produces": [],
    "parameters": [
        {
            "name": "id",
            "in": "path",
            "description": "",
            "required": true,
            "type": "integer",
            "format": "int64"
        }
    ],
    "responses": {
        "200": {
            "description": "Success"
        }
    }
}
```

Aggiungere un [\<remarks>](/dotnet/csharp/programming-guide/xmldoc/remarks) elemento alla `Create` documentazione del metodo di azione. In questo modo vengono integrate le informazioni specificate nell'elemento `<summary>` e l'interfaccia utente di Swagger risulta più affidabile. Il contenuto dell'elemento `<remarks>` può essere costituito da testo, JSON o XML.

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

Si notino i miglioramenti dell'interfaccia utente con questi commenti aggiuntivi:

![Interfaccia utente di Swagger con commenti aggiuntivi visualizzati](web-api-help-pages-using-swagger/_static/xml-comments-extended.png)

### <a name="data-annotations"></a>Annotazioni dei dati

Contrassegnare il modello con attributi, disponibile nello spazio dei nomi [System. ComponentModel. DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) , per guidare i componenti dell'interfaccia utente di spavalderia.

Aggiungere l'attributo `[Required]` alla proprietà `Name` della classe `TodoItem`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Models/TodoItem.cs?highlight=10)]

La presenza di questo attributo modifica il comportamento dell'interfaccia utente e lo schema JSON sottostante:

```json
"definitions": {
    "TodoItem": {
        "required": [
            "name"
        ],
        "type": "object",
        "properties": {
            "id": {
                "format": "int64",
                "type": "integer"
            },
            "name": {
                "type": "string"
            },
            "isComplete": {
                "default": false,
                "type": "boolean"
            }
        }
    }
},
```

Aggiungere l'attributo `[Produces("application/json")]` al controller API. Il suo scopo consiste nel dichiarare che le azioni del controller supportano un tipo di contenuto della risposta di *application/json*:

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

L'elenco a discesa **tipo di contenuto della risposta** seleziona questo tipo di contenuto come impostazione predefinita per le azioni Get del controller:

![Interfaccia utente di Swagger con tipo di contenuto di risposta predefinito](web-api-help-pages-using-swagger/_static/json-response-content-type.png)

Con l'aumento dell'uso delle annotazioni dei dati nell'API Web, le pagine della Guida dell'API e dell'interfaccia utente diventano più descrittive e utili.

### <a name="describe-response-types"></a>Descrivere i tipi di risposta

La principale preoccupazione degli sviluppatori che utilizzano un'API Web è ciò che viene restituito, in particolare i tipi di risposta e i codici di errore, se diversi da quelli standard. I tipi di risposta e i codici di errore vengono indicati nei commenti XML e nelle annotazioni dei dati.

L'azione `Create` restituisce un codice di stato HTTP 201 in caso di esito positivo. Quando il corpo della richiesta inviata è Null, viene restituito un codice di stato HTTP 400. Senza la documentazione appropriata nell'interfaccia utente di Swagger, il consumer non riconosce tali risultati previsti. Risolvere il problema aggiungendo le righe evidenziate nell'esempio seguente:

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

L'interfaccia utente di Swagger ora documenta chiaramente i codici di risposta HTTP previsti:

![Interfaccia utente di Swagger che visualizza la descrizione della classe risposta POST "Returns the newly created Todo item" e "400 - If the item is null" per il codice di stato e il motivo nei messaggi di risposta](web-api-help-pages-using-swagger/_static/data-annotations-response-types.png)

::: moniker range=">= aspnetcore-2.2"

In ASP.NET Core 2.2 o versioni successive, possono essere usate convenzioni in alternativa alla decorazione esplicita di azioni singole con `[ProducesResponseType]`. Per altre informazioni, vedere <xref:web-api/advanced/conventions>.

Per supportare la `[ProducesResponseType]` decorazione, il pacchetto [Swashbuckle. AspNetCore. Annotations](https://github.com/domaindrivendev/Swashbuckle.AspNetCore/blob/master/README.md#swashbuckleaspnetcoreannotations) offre estensioni per abilitare e migliorare la risposta, lo schema e i metadati dei parametri.

::: moniker-end

### <a name="customize-the-ui"></a>Personalizzare l'interfaccia utente

L'interfaccia utente predefinita è sia funzionale che rappresentabile. ma è necessario che le pagine della documentazione API rappresentino il proprio marchio o tema. Per personalizzare i componenti Swashbuckle è necessario aggiungere le risorse per gestire i file statici e quindi compilare la struttura di cartelle per ospitare i file.

Se si usa .NET Framework o .NET Core 1.x, aggiungere il pacchetto NuGet [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles) al progetto:

```xml
<PackageReference Include="Microsoft.AspNetCore.StaticFiles" Version="2.0.0" />
```

Il pacchetto NuGet precedente è già installato se si usano .NET Core 2.x e il [metapackage](xref:fundamentals/metapackage).

Abilitare il middleware dei file statici:

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup.cs?name=snippet_Configure&highlight=3)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_Configure&highlight=3)]

::: moniker-end

Per inserire fogli di stile CSS aggiuntivi, aggiungerli alla cartella *wwwroot* del progetto e specificare il percorso relativo nelle opzioni del middleware:

```csharp
app.UseSwaggerUI(c =>
{
     c.InjectStylesheet("/swagger-ui/custom.css");
}
```

## <a name="additional-resources"></a>Risorse aggiuntive

* [Microsoft Learn: migliorare l'esperienza di sviluppo di un'API con la documentazione di spavalderia](/learn/modules/improve-api-developer-experience-with-swagger/)
