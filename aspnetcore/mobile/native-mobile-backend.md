---
title: Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core
author: ardalis
description: Informazioni su come creare servizi back-end mediante ASP.NET Core MVC per il supporto di app per dispositivi mobili native.
ms.author: riande
ms.date: 12/05/2019
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
uid: mobile/native-mobile-backend
ms.openlocfilehash: 4e86241771e884ba9079bcdf9a09eebc6acd62c8
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530216"
---
# <a name="create-backend-services-for-native-mobile-apps-with-aspnet-core"></a>Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core

Di [Steve Smith](https://ardalis.com/)

Le app per dispositivi mobili possono comunicare con i servizi back-end di ASP.NET Core. Per istruzioni sulla connessione a servizi Web locali da simulatori iOS ed emulatori Android, vedere [Connettersi a servizi Web locali da simulatori iOS ed emulatori Android](/xamarin/cross-platform/deploy-test/connect-to-local-web-services).

[Visualizzare o scaricare codice di esempio di servizi back-end](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mobile/native-mobile-backend/sample)

## <a name="the-sample-native-mobile-app"></a>App per dispositivi mobili nativa di esempio

Questa esercitazione illustra come creare servizi back-end mediante ASP.NET Core MVC per il supporto di app per dispositivi mobili native. Usa come client nativo l'[app ToDoRest di Xamarin.Forms](/xamarin/xamarin-forms/data-cloud/consuming/rest), che include client nativi separati per i dispositivi Android, iOS, Windows Universal e Windows Phone. È possibile eseguire l'esercitazione collegata per creare l'app nativa (e installare gli strumenti Xamarin gratuiti necessari), nonché scaricare la soluzione di esempio di Xamarin. L'esempio di Xamarin include un progetto di servizi ASP.NET Web API 2, che viene sostituito dall'app ASP.NET Core di questo articolo (senza richiedere nessuna modifica al client).

![Applicazione ToDoRest in esecuzione su uno smartphone Android](native-mobile-backend/_static/todo-android.png)

### <a name="features"></a>Funzionalità

L'app ToDoRest supporta l'aggiunta, l'eliminazione, l'aggiornamento e l'aggiunta a elenco di elementi attività. Ogni elemento include un ID, un nome, delle note e una proprietà che indica se è stato eseguito.

La visualizzazione principale degli elementi, che appare nell'immagine in alto, contiene un elenco con il nome di ogni elemento. Un segno di spunta indica se l'elemento è stato eseguito.

Se si tocca l'icona `+` viene visualizzata una finestra di dialogo per l'aggiunta di elementi:

![Finestra di dialogo per l'aggiunta di elementi](native-mobile-backend/_static/todo-android-new-item.png)

Se si tocca un elemento sulla schermata dell'elenco principale viene visualizzata una finestra di dialogo di modifica in cui è possibile modificare le opzioni Name (Nome), Notes (Note) e Done (Fatto) dell'elemento oppure eliminare l'elemento stesso:

![Finestra di dialogo di modifica dell'elemento](native-mobile-backend/_static/todo-android-edit-item.png)

Questo esempio è configurato per impostazione predefinita per l'uso dei servizi back-end ospitati in developer.xamarin.com, che consentono operazioni di sola lettura. Per eseguire direttamente il test nel proprio computer con l'app ASP.NET Core creata nella sezione successiva, è necessario modificare la costante `RestUrl` dell'app. Passare al progetto `ToDoREST` e aprire il file *Constants.cs*. Sostituire `RestUrl` con un URL che include l'indirizzo IP del computer (non localhost o 127.0.0.1, poiché questo indirizzo viene usato dall'emulatore di dispositivo e non dal computer in uso). Includere anche il numero di porta (5000). Per verificare che i servizi funzionino con un dispositivo, assicurarsi che l'accesso a questa porta non sia bloccato da un firewall attivo.

```csharp
// URL of REST service (Xamarin ReadOnly Service)
//public static string RestUrl = "http://developer.xamarin.com:8081/api/todoitems{0}";

// use your machine's IP address
public static string RestUrl = "http://192.168.1.207:5000/api/todoitems/{0}";
```

## <a name="creating-the-aspnet-core-project"></a>Creazione del progetto ASP.NET Core

Creare una nuova applicazione Web ASP.NET Core in Visual Studio. Scegliere il modello API Web e Nessuna autenticazione. Denominare il progetto *ToDoApi*.

![Finestra di dialogo Nuova applicazione Web ASP.NET Core con il modello di progetto API Web selezionato](native-mobile-backend/_static/web-api-template.png)

L'applicazione deve rispondere a tutte le richieste effettuate sulla porta 5000. A tale scopo aggiornare *Program.cs* in modo che includa `.UseUrls("http://*:5000")`:

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Program.cs?range=10-16&highlight=3)]

> [!NOTE]
> Assicurarsi di eseguire l'applicazione direttamente anziché dietro IIS Express, che per impostazione predefinita ignora le richieste non locali. Eseguire [dotnet run](/dotnet/core/tools/dotnet-run) al prompt dei comandi oppure scegliere il profilo del nome dell'applicazione nell'elenco a discesa Destinazione di debug sulla barra degli strumenti di Visual Studio.

Aggiungere una classe modello che rappresenti gli elementi attività. Contrassegnare i campi obbligatori con l' `[Required]` attributo:

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Models/ToDoItem.cs)]

I metodi dell'API richiedono un approccio per l'uso dei dati. Usare la stessa interfaccia `IToDoRepository` usata nell'esempio originale di Xamarin:

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Interfaces/IToDoRepository.cs)]

Per questo esempio, l'implementazione usa solo una raccolta privata di elementi:

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Services/ToDoRepository.cs)]

Configurare l'implementazione in *Startup.cs*:

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Startup.cs?highlight=6&range=29-35)]

A questo punto si è pronti per creare *ToDoItemsController*.

> [!TIP]
> Per altre informazioni sulla creazione di API Web, vedere [Sviluppare la prima API Web con ASP.NET Core MVC e Visual Studio](../tutorials/first-web-api.md).

## <a name="creating-the-controller"></a>Creazione del controller

Aggiungere al progetto un nuovo controller, *ToDoItemsController*. Il controller deve ereditare da Microsoft.AspNetCore.Mvc.Controller. Aggiungere un attributo `Route` per indicare che il controller gestirà le richieste inoltrate ai percorsi che iniziano con `api/todoitems`. Il token `[controller]` nella route viene sostituito dal nome del controller (omettendo il suffisso `Controller`) ed è particolarmente utile per le route globali. Altre informazioni sul [routing](../fundamentals/routing.md).

Per funzionare, il controller richiede un `IToDoRepository`. Richiedere un'istanza di questo tipo tramite il costruttore del controller. In fase di esecuzione, questa istanza viene resa disponibile tramite il supporto dell'[inserimento di dipendenze](../fundamentals/dependency-injection.md) specificato dal framework.

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=1-17&highlight=9,14)]

Questa API supporta quattro verbi HTTP diversi per eseguire operazioni CRUD (Create, Read, Update, Delete - Creazione, Lettura, Aggiornamento, Eliminazione) nell'origine dati. L'operazione più semplice tra queste è Read, che corrisponde a una richiesta HTTP GET.

### <a name="reading-items"></a>Lettura degli elementi

La richiesta di un elenco di elementi viene eseguita con una richiesta GET al metodo `List`. L'attributo `[HttpGet]` del metodo `List` indica che questa azione deve gestire solo le richieste GET. La route di questa azione è la route specificata nel controller. Non è necessario usare il nome dell'azione come parte della route. È sufficiente garantire che ogni azione disponga di una route univoca e non ambigua. Gli attributi di routing possono essere applicati sia a livello di controller che di metodo per definire route specifiche.

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=19-23)]

Il metodo `List` restituisce un codice di risposta 200 (OK) e tutti gli elementi di attività (ToDo), serializzati con formato JSON.

È possibile eseguire il test del nuovo metodo API usando un'ampia gamma di strumenti, ad esempio [Postman](https://www.getpostman.com/docs/), visualizzato di seguito:

![Console di Postman con una richiesta GET per elementi di attività (todoitems) e il corpo della risposta con JSON per tre elementi restituiti](native-mobile-backend/_static/postman-get.png)

### <a name="creating-items"></a>Creazione di elementi

Per convenzione, la creazione di nuovi elementi di dati viene associata al verbo HTTP POST. Al `Create` metodo è `[HttpPost]` applicato un attributo e viene accettata un' `ToDoItem` istanza. Poiché l' `item` argomento viene passato nel corpo del post, questo parametro specifica l' `[FromBody]` attributo.

All'interno del metodo, l'elemento viene verificato per validità ed esistenza precedente nell'archivio dati. Se non si verificano problemi, l'elemento viene aggiunto tramite il repository. La verifica `ModelState.IsValid` esegue la [convalida del modello](../mvc/models/validation.md) e deve essere eseguita in ogni metodo dell'API che accetta input utente.

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=25-46)]

L'esempio usa un'enumerazione contenente codici di errore che vengono passati al client per dispositivi mobili:

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=91-99)]

Eseguire il test dell'aggiunta di nuovi elementi con Postman scegliendo il verbo POST che rende disponibile il nuovo oggetto in formato JSON nel corpo della richiesta. È anche necessario aggiungere un'intestazione di richiesta specificando come `Content-Type` la voce `application/json`.

![Console di Postman che visualizzano un POST e una risposta](native-mobile-backend/_static/postman-post.png)

Il metodo restituisce l'elemento appena creato nella risposta.

### <a name="updating-items"></a>Aggiornamento degli elementi

La modifica dei record viene eseguita mediante richieste PUT HTTP. Fatta eccezione per questa modifica, il metodo `Edit` è quasi identico a `Create`. Si noti che se il record non viene trovato, l'azione `Edit` restituisce una risposta `NotFound` (404).

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=48-69)]

Per eseguire test con Postman, impostare il verbo su PUT. Specificare i dati dell'oggetto aggiornati nel Corpo della richiesta.

![Console di Postman con un elemento PUT e una risposta](native-mobile-backend/_static/postman-put.png)

Se va a buon fine, questo metodo restituisce una risposta `NoContent` (204) per coerenza con l'API preesistente.

### <a name="deleting-items"></a>Eliminazione di elementi

L'eliminazione di record si ottiene inviando richieste DELETE al servizio e passando l'ID dell'elemento da eliminare. Come con gli aggiornamenti, le richieste relative a elementi che non esistono ricevono risposte `NotFound`. Una richiesta con esito positivo riceve una risposta `NoContent` (204).

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=71-88)]

Si noti che durante il test della funzionalità di eliminazione non è necessario alcun elemento nella sezione Corpo della richiesta.

![Console di Postman con un'operazione DELETE e una risposta](native-mobile-backend/_static/postman-delete.png)

## <a name="common-web-api-conventions"></a>Convenzioni comuni dell'API Web

Durante lo sviluppo dei servizi back-end per l'app è necessario creare un set di convenzioni o criteri coerenti per la gestione dei problemi di montaggio incrociato. Ad esempio nel servizio precedente le richieste di record specifici che non sono stati trovati ricevono una risposta `NotFound` anziché una risposta `BadRequest`. Analogamente, i comandi inoltrati a questo servizio che trasmettono tipi associati al modello verificano sempre `ModelState.IsValid` e restituiscono `BadRequest` per i tipi di modello non validi.

Dopo aver identificato criteri comuni per le API è in genere possibile incapsulare tali criteri in un [filtro](../mvc/controllers/filters.md). Altre informazioni su [come incapsulare criteri comuni per le API nelle applicazioni ASP.NET Core MVC](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters).

## <a name="additional-resources"></a>Risorse aggiuntive

- [Novell. Forms: autenticazione del servizio Web](/xamarin/xamarin-forms/data-cloud/authentication/)
- [Novell. Forms: utilizzo di un servizio Web RESTful](/xamarin/xamarin-forms/data-cloud/web-services/rest)
- [Microsoft Learn: utilizzo di servizi Web REST nelle app Novell](/learn/modules/consume-rest-services/)
- [Microsoft Learn: creare un'API Web con ASP.NET Core](/learn/modules/build-web-api-aspnet-core/)
