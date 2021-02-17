---
title: Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core
author: rick-anderson
description: Informazioni su come creare servizi back-end mediante ASP.NET Core MVC per il supporto di app per dispositivi mobili native.
ms.author: riande
ms.date: 2/12/2021
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
ms.openlocfilehash: e496b7811cc534b6f0f6dfdb857f6e462b38049e
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564024"
---
# <a name="create-backend-services-for-native-mobile-apps-with-aspnet-core"></a>Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core

Di [James Montemagno su](https://twitter.com/JamesMontemagno)

Le app per dispositivi mobili possono comunicare con i servizi back-end di ASP.NET Core. Per istruzioni sulla connessione a servizi Web locali da simulatori iOS ed emulatori Android, vedere [Connettersi a servizi Web locali da simulatori iOS ed emulatori Android](/xamarin/cross-platform/deploy-test/connect-to-local-web-services).

[Visualizzare o scaricare codice di esempio di servizi back-end](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST)

## <a name="the-sample-native-mobile-app"></a>App per dispositivi mobili nativa di esempio

Questa esercitazione illustra come creare servizi back-end usando ASP.NET Core per supportare le app per dispositivi mobili native. Usa l' [app Novell. Forms Todorer](/xamarin/xamarin-forms/data-cloud/consuming/rest) come client nativo, che include client nativi distinti per Android, iOS e Windows. È possibile eseguire l'esercitazione collegata per creare l'app nativa (e installare gli strumenti Xamarin gratuiti necessari), nonché scaricare la soluzione di esempio di Xamarin. L'esempio Novell include un progetto di servizi API Web ASP.NET Core, che viene sostituito dall'app ASP.NET Core di questo articolo, senza alcuna modifica richiesta dal client.

![Applicazione ToDoRest in esecuzione su uno smartphone Android](native-mobile-backend/_static/todo-android.png)

### <a name="features"></a>Funzionalità

L' [app Todorer](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST) supporta l'inserimento, l'aggiunta, l'eliminazione e l'aggiornamento di To-Do elementi. Ogni elemento include un ID, un nome, delle note e una proprietà che indica se è stato eseguito.

La visualizzazione principale degli elementi, che appare nell'immagine in alto, contiene un elenco con il nome di ogni elemento. Un segno di spunta indica se l'elemento è stato eseguito.

Se si tocca l'icona `+` viene visualizzata una finestra di dialogo per l'aggiunta di elementi:

![Finestra di dialogo per l'aggiunta di elementi](native-mobile-backend/_static/todo-android-new-item.png)

Se si tocca un elemento sulla schermata dell'elenco principale viene visualizzata una finestra di dialogo di modifica in cui è possibile modificare le opzioni Name (Nome), Notes (Note) e Done (Fatto) dell'elemento oppure eliminare l'elemento stesso:

![Finestra di dialogo di modifica dell'elemento](native-mobile-backend/_static/todo-android-edit-item.png)

Per testarlo in base all'app ASP.NET Core creata nella sezione successiva in esecuzione nel computer, aggiornare la costante dell'app [`RestUrl`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs#L13) .

Gli emulatori Android non vengono eseguiti nel computer locale e usano un IP di loopback (10.0.2.2) per comunicare con il computer locale. Usare [Novell. Essentials DeviceInfo](/xamarin/essentials/device-information/) per rilevare le operazioni eseguite dal sistema per usare l'URL corretto.

Passare al [`TodoREST`](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoREST) progetto e aprire il [`Constants.cs`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs) file. Il file *Constants.cs* contiene la configurazione seguente.

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoREST/Constants.cs" highlight="13":::

Facoltativamente, è possibile distribuire il servizio Web in un servizio cloud, ad esempio Azure, e aggiornare il `RestUrl` .

## <a name="creating-the-aspnet-core-project"></a>Creazione del progetto ASP.NET Core

Creare una nuova applicazione Web ASP.NET Core in Visual Studio. Scegliere il modello API Web. Denominare il progetto *TodoAPI*.

![Finestra di dialogo Nuova applicazione Web ASP.NET Core con il modello di progetto API Web selezionato](native-mobile-backend/_static/web-api-template.png)

L'app deve rispondere a tutte le richieste effettuate alla porta 5000, incluso il traffico http Clear-Text per il client mobile. Aggiornare *Startup.cs* <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> , quindi non viene eseguito in fase di sviluppo:

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Startup.cs" id="snippet" highlight="7-11":::

> [!NOTE]
> Eseguire l'app direttamente, anziché dietro IIS Express. Per impostazione predefinita, IIS Express ignora le richieste non locali. Eseguire [DotNet Run](/dotnet/core/tools/dotnet-run) da un prompt dei comandi o scegliere il profilo nome app dall'elenco a discesa destinazione di debug nella barra degli strumenti di Visual Studio.

Aggiungere una classe modello che rappresenti gli elementi attività. Contrassegnare i campi obbligatori con l' `[Required]` attributo:

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Models/TodoItem.cs":::

I metodi dell'API richiedono un approccio per l'uso dei dati. Usare la stessa interfaccia `ITodoRepository` usata nell'esempio originale di Xamarin:

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Interfaces/ITodoRepository.cs":::

Per questo esempio, l'implementazione usa solo una raccolta privata di elementi:

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Services/TodoRepository.cs":::

Configurare l'implementazione in *Startup.cs*:

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Startup.cs" id="snippet2" highlight="3":::

## <a name="creating-the-controller"></a>Creazione del controller

Aggiungere un nuovo controller al progetto [TodoItemsController](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs). Deve ereditare da <xref:Microsoft.AspNetCore.Mvc.ControllerBase> . Aggiungere un attributo `Route` per indicare che il controller gestirà le richieste inoltrate ai percorsi che iniziano con `api/todoitems`. Il token `[controller]` nella route viene sostituito dal nome del controller (omettendo il suffisso `Controller`) ed è particolarmente utile per le route globali. Altre informazioni sul [routing](../fundamentals/routing.md).

Per funzionare, il controller richiede un `ITodoRepository`. Richiedere un'istanza di questo tipo tramite il costruttore del controller. In fase di esecuzione, questa istanza viene resa disponibile tramite il supporto dell'[inserimento di dipendenze](../fundamentals/dependency-injection.md) specificato dal framework.

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetDI":::

Questa API supporta quattro verbi HTTP diversi per eseguire operazioni CRUD (Create, Read, Update, Delete - Creazione, Lettura, Aggiornamento, Eliminazione) nell'origine dati. L'operazione più semplice tra queste è Read, che corrisponde a una richiesta HTTP GET.

### <a name="reading-items"></a>Lettura degli elementi

La richiesta di un elenco di elementi viene eseguita con una richiesta GET al metodo `List`. L'attributo `[HttpGet]` del metodo `List` indica che questa azione deve gestire solo le richieste GET. La route di questa azione è la route specificata nel controller. Non è necessario usare il nome dell'azione come parte della route. È sufficiente garantire che ogni azione disponga di una route univoca e non ambigua. Gli attributi di routing possono essere applicati sia a livello di controller che di metodo per definire route specifiche.

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippet":::

Il `List` metodo restituisce un codice di risposta 200 OK e tutti gli elementi todo, serializzati come JSON.

È possibile eseguire il test del nuovo metodo API usando un'ampia gamma di strumenti, ad esempio [Postman](https://www.getpostman.com/docs/), visualizzato di seguito:

![Console di Postman con una richiesta GET per elementi di attività (todoitems) e il corpo della risposta con JSON per tre elementi restituiti](native-mobile-backend/_static/postman-get.png)

### <a name="creating-items"></a>Creazione di elementi

Per convenzione, la creazione di nuovi elementi di dati viene associata al verbo HTTP POST. Al `Create` metodo è `[HttpPost]` applicato un attributo e viene accettata un' `TodoItem` istanza. Poiché l' `item` argomento viene passato nel corpo del post, questo parametro specifica l' `[FromBody]` attributo.

All'interno del metodo, l'elemento viene verificato per validità ed esistenza precedente nell'archivio dati. Se non si verificano problemi, l'elemento viene aggiunto tramite il repository. La verifica `ModelState.IsValid` esegue la [convalida del modello](../mvc/models/validation.md) e deve essere eseguita in ogni metodo dell'API che accetta input utente.

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetCreate":::

Nell'esempio viene utilizzato un oggetto `enum` contenente i codici di errore passati al client mobile:

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetErrorCode":::

Eseguire il test dell'aggiunta di nuovi elementi con Postman scegliendo il verbo POST che rende disponibile il nuovo oggetto in formato JSON nel corpo della richiesta. È anche necessario aggiungere un'intestazione di richiesta specificando come `Content-Type` la voce `application/json`.

![Console di Postman che visualizzano un POST e una risposta](native-mobile-backend/_static/postman-post.png)

Il metodo restituisce l'elemento appena creato nella risposta.

### <a name="updating-items"></a>Aggiornamento degli elementi

La modifica dei record viene eseguita mediante richieste PUT HTTP. Fatta eccezione per questa modifica, il metodo `Edit` è quasi identico a `Create`. Si noti che se il record non viene trovato, l'azione `Edit` restituisce una risposta `NotFound` (404).

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetEdit":::

Per eseguire test con Postman, impostare il verbo su PUT. Specificare i dati dell'oggetto aggiornati nel Corpo della richiesta.

![Console di Postman con un elemento PUT e una risposta](native-mobile-backend/_static/postman-put.png)

Se va a buon fine, questo metodo restituisce una risposta `NoContent` (204) per coerenza con l'API preesistente.

### <a name="deleting-items"></a>Eliminazione di elementi

L'eliminazione di record si ottiene inviando richieste DELETE al servizio e passando l'ID dell'elemento da eliminare. Come con gli aggiornamenti, le richieste relative a elementi che non esistono ricevono risposte `NotFound`. Una richiesta con esito positivo riceve una risposta `NoContent` (204).

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetDelete":::

Si noti che durante il test della funzionalità di eliminazione non è necessario alcun elemento nella sezione Corpo della richiesta.

![Console di Postman con un'operazione DELETE e una risposta](native-mobile-backend/_static/postman-delete.png)

## <a name="prevent-over-posting"></a>Impedisci overposting

Attualmente l'app di esempio espone l'intero `TodoItem` oggetto. Le app di produzione limitano in genere i dati di input e restituiti usando un subset del modello. Esistono diversi motivi alla base di questo e la sicurezza è una delle principali. Il subset di un modello è in genere indicato come oggetto Trasferimento dati (DTO), modello di input o modello di visualizzazione. **Dto** viene usato in questo articolo.

Un DTO può essere usato per:

* Impedisci l'overposting.
* Nascondere le proprietà che i client non dovrebbero visualizzare.
* Omettere alcune proprietà per ridurre le dimensioni del payload.
* Rendere flat gli oggetti grafici che contengono oggetti annidati. I grafici a oggetti flat possono essere più pratici per i client.

Per illustrare l'approccio DTO, vedere [impedire l'overposting](xref:tutorials/first-web-api#prevent-over-posting)

## <a name="common-web-api-conventions"></a>Convenzioni comuni dell'API Web

Durante lo sviluppo dei servizi back-end per l'app è necessario creare un set di convenzioni o criteri coerenti per la gestione dei problemi di montaggio incrociato. Ad esempio nel servizio precedente le richieste di record specifici che non sono stati trovati ricevono una risposta `NotFound` anziché una risposta `BadRequest`. Analogamente, i comandi inoltrati a questo servizio che trasmettono tipi associati al modello verificano sempre `ModelState.IsValid` e restituiscono `BadRequest` per i tipi di modello non validi.

Dopo aver identificato criteri comuni per le API è in genere possibile incapsulare tali criteri in un [filtro](../mvc/controllers/filters.md). Altre informazioni su [come incapsulare criteri comuni per le API nelle applicazioni ASP.NET Core MVC](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters).

## <a name="additional-resources"></a>Risorse aggiuntive

- [Novell. Forms: autenticazione del servizio Web](/xamarin/xamarin-forms/data-cloud/authentication/)
- [Novell. Forms: utilizzo di un servizio Web RESTful](/xamarin/xamarin-forms/data-cloud/web-services/rest)
- [Microsoft Learn: utilizzo di servizi Web REST nelle app Novell](/learn/modules/consume-rest-services/)
- [Microsoft Learn: creare un'API Web con ASP.NET Core](/learn/modules/build-web-api-aspnet-core/)
