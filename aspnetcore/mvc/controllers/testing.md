---
title: Test della logica dei controller in ASP.NET Core
author: ardalis
description: Informazioni sul test della logica dei controller in ASP.NET Core con Moq e xUnit.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 7/22/2020
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
uid: mvc/controllers/testing
ms.openlocfilehash: 0f4fcceefdef946a5f29633c069cbb64ec345a3d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586956"
---
# <a name="unit-test-controller-logic-in-aspnet-core"></a>Logica unità test controller in ASP.NET Core

Di [Steve Smith](https://ardalis.com/)

::: moniker range=">= aspnetcore-3.0"

Gli [unit test](/dotnet/articles/core/testing/unit-testing-with-dotnet-test) implicano l'esecuzione di test su una parte di un'app isolandola dall'infrastruttura e dalle dipendenze. Quando si sottopone a unit test la logica del controller, si verificano solo i contenuti di una singola azione e non il comportamento delle relative dipendenze o del framework.

## <a name="unit-testing-controllers"></a>Controller di unit test

Configurare unit test delle azioni del controller per concentrare l'attenzione sul comportamento del controller. Uno unit test del controller evita scenari come [filtri](xref:mvc/controllers/filters), [routing](xref:fundamentals/routing) e [associazione di modelli](xref:mvc/models/model-binding). I test che verificano le interazioni tra i componenti che collettivamente rispondono a una richiesta vengono gestiti dai *test di integrazione*. Per altre informazioni sui test di integrazione, vedere <xref:test/integration-tests>.

Se si scrivono route e filtri personalizzati, sottoporli a unit test in isolamento e non durante i test relativi a una determinata azione del controller.

Per una dimostrazione degli unit test del controller, esaminare il controller seguente nell'app di esempio. 

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/testing/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

Il controller Home visualizza un elenco di sessioni di brainstorming e consente di creare nuove sessioni con una richiesta POST:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/HomeController.cs?name=snippet_HomeController&highlight=1,5,10,31-32)]

Il controller precedente:

* Segue il [principio delle dipendenze esplicite](/dotnet/architecture/modern-web-apps-azure/architectural-principles#explicit-dependencies).
* Si aspetta che l'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) fornisca un'istanza di `IBrainstormSessionRepository`.
* Può essere testato con un servizio `IBrainstormSessionRepository` fittizio tramite il framework di un oggetto fittizio, ad esempio [Moq](https://www.nuget.org/packages/Moq/). Un *oggetto fittizio* è un oggetto creato con un set predeterminato di comportamenti di proprietà e metodi usati per il testing. Per altre informazioni, vedere [Introduction to integration tests](xref:test/integration-tests#introduction-to-integration-tests) (Introduzione ai test di integrazione).

Il metodo `HTTP GET Index` non dispone di cicli o rami e chiama un solo metodo. Lo unit test per questa azione:

* Simula il servizio `IBrainstormSessionRepository` usando il metodo `GetTestSessions`. `GetTestSessions` crea due sessioni di brainstorming fittizie con date e nomi di sessione.
* Esegue il metodo `Index`.
* Crea asserzioni sul risultato restituito dal metodo:
  * Viene restituito un <xref:Microsoft.AspNetCore.Mvc.ViewResult>.
  * [ViewDataDictionary.Model](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary.Model*) è di tipo `StormSessionViewModel`.
  * Nel `ViewDataDictionary.Model` sono archiviate due sessioni di brainstorming.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_Index_ReturnsAViewResult_WithAListOfBrainstormSessions&highlight=14-17)]

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_GetTestSessions)]

Il metodo `HTTP POST Index` del controller Home verifica che:

* Quando [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid*) è `false`, il metodo di azione restituisca un elemento *di tipo 400 - Richiesta non valida* <xref:Microsoft.AspNetCore.Mvc.ViewResult> con i dati appropriati.
* Quando `ModelState.IsValid` è `true`:
  * Venga chiamato il metodo `Add` sul repository.
  * Venga restituito un <xref:Microsoft.AspNetCore.Mvc.RedirectToActionResult> con gli argomenti corretti.

È possibile eseguire il test di uno stato del modello non valido aggiungendo gli errori con <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> come illustrato nel primo test riportato di seguito:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_ModelState_ValidOrInvalid&highlight=9,16-17,38-41)]

Quando [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) non è valido, viene restituito lo stesso `ViewResult` come per una richiesta GET. Il test non prova a passare un modello non valido. Il passaggio di un modello non valido non è un approccio corretto, in quanto l'associazione di modelli non è in esecuzione (benché un [test di integrazione](xref:test/integration-tests) usi invece l'associazione di modelli). In questo caso l'associazione di modelli non viene testata. Questi unit test testano solo il codice del metodo di azione.

Il secondo test verifica che quando `ModelState` è valido:

* Venga aggiunto un nuovo elemento `BrainstormSession` (tramite il repository).
* Il metodo restituisca un `RedirectToActionResult` con le proprietà previste.

Le chiamate fittizie che non vengono eseguite sono in genere ignorate, ma la chiamata di `Verifiable` al termine della chiamata setup consente la convalida fittizia nel test. Questa operazione viene eseguita con la chiamata a `mockRepo.Verify`, che non supera il test se non è stato chiamato il metodo previsto.

> [!NOTE]
> La libreria Moq usata in questo esempio consente la combinazione di simulazioni verificabili o "rigide" con simulazioni non verificabili (dette anche simulazioni "generiche" o stub "generici"). Altre informazioni sulla [personalizzazione del comportamento di simulazione con Moq](https://github.com/Moq/moq4/wiki/Quickstart#customizing-mock-behavior).

[SessionController](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/controllers/testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs) nell'app di esempio visualizza informazioni correlate a una sessione di brainstorming specifica. Il controller include la logica per gestire i valori `id` non validi (nell'esempio seguente sono riportati due scenari `return` in proposito). L'istruzione finale `return` restituisce un nuovo elemento `StormSessionViewModel` alla vista (*Controllers/SessionController.cs*):

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs?name=snippet_SessionController&highlight=12-16,18-22,31)]

Gli unit test includono un test per ogni scenario `return` nell'azione `Index` del controller Session:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/SessionControllerTests.cs?name=snippet_SessionControllerTests&highlight=2,11-14,18,31-32,36,50-55)]

Passando al controller Ideas, l'app espone la funzionalità come API Web sulla route `api/ideas`:

* Il metodo `ForSession` restituisce un elenco di idee (`IdeaDTO`) associate a una sessione di brainstorming.
* Il metodo `Create` aggiunge nuove idee a una sessione.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionAndCreate&highlight=1-2,21-22)]

Evitare di restituire direttamente le entità di dominio aziendali tramite chiamate API. Le entità di dominio infatti:

* Spesso includono più dati di quelli richiesti dal client.
* Associano senza necessità il modello di dominio interno dell'app all'API esposta pubblicamente.

Il mapping tra le entità di dominio e i tipi restituiti al client può essere eseguito:

* Manualmente con un'operazione LINQ `Select`, come avviene nell'app di esempio. Per altre informazioni, vedere [LINQ (Language-Integrated Query)](/dotnet/standard/using-linq).
* Automaticamente con una libreria, come [AutoMapper](https://github.com/AutoMapper/AutoMapper).

L'app di esempio esegue quindi una dimostrazione degli unit test per i metodi API `Create` e `ForSession` del controller Ideas.

L' app di esempio contiene due test `ForSession`. Il primo determina se `ForSession` restituisce un <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult> (HTTP non trovato) per una sessione non valida:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests4&highlight=5,7-8,15-16)]

Il secondo test `ForSession` determina se `ForSession` restituisce un elenco di idee (`<List<IdeaDTO>>`) per una sessione valida. I controlli esaminano anche la prima idea per verificare che la relativa proprietà `Name` sia corretta:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests5&highlight=5,7-8,15-18)]

Per testare il comportamento del metodo `Create` quando `ModelState` non è valido, l'app di esempio aggiunge un errore del modello al controller come parte del test. Non provare a testare la convalida o l'associazione di modelli negli unit test. Testare solo il comportamento del metodo di azione rispetto a un `ModelState` non valido:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests1&highlight=7,13)]

Il secondo test di `Create` dipende dal fatto che il repository restituisca `null`, pertanto il repository fittizio è configurato per restituire `null`. Non è necessario creare un database di test (in memoria o con un altro approccio) e creare una query che restituisca questo risultato. Il test può essere eseguito in una singola istruzione, come illustrato dal codice di esempio:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests2&highlight=7-8,15)]

Il terzo test di `Create`, `Create_ReturnsNewlyCreatedIdeaForSession`, verifica che venga chiamato il metodo `UpdateAsync` del repository. La simulazione viene chiamata con `Verifiable` e quindi viene chiamato il metodo `Verify` del repository fittizio per confermare che il metodo verificabile sia stato eseguito. Non spetta allo unit test verificare che il metodo `UpdateAsync` abbia salvato i dati. Questa operazione può essere eseguita con un test di integrazione.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests3&highlight=20-22,28-33)]

## <a name="test-actionresultt"></a>Test ActionResult\<T>

In ASP.NET Core 2,1 o versioni successive [, \<T> ActionResult](xref:web-api/action-return-types#actionresultt-type) ( <xref:Microsoft.AspNetCore.Mvc.ActionResult%601> ) consente di restituire un tipo che deriva da `ActionResult` o restituisce un tipo specifico.

L'app di esempio include un metodo che restituisce un `List<IdeaDTO>` per un determinato `id` di sessione. Se l'`id` di sessione non esiste, il controller restituisce <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>:

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionActionResult&highlight=10,21)]

In `ApiIdeasControllerTests` sono inclusi due test del controller `ForSessionActionResult`.

Il primo test verifica che il controller restituisca un `ActionResult` ma non un elenco non esistente di idee per un `id` di sessione inesistente:

* Il tipo di `ActionResult` è `ActionResult<List<IdeaDTO>>`.
* <xref:Microsoft.AspNetCore.Mvc.ActionResult`1.Result*> è di tipo <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=7,10,13-14)]

Per un `id` di sessione valido, il secondo test verifica che il metodo restituisca:

* Il metodo restituisca un `ActionResult` con un tipo `List<IdeaDTO>`.
* [ActionResult \<T> . Il valore](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*) è un `List<IdeaDTO>` tipo.
* Il primo elemento nell'elenco sia un'idea valida corrispondente all'idea archiviata nella sessione fittizia (ottenuta chiamando `GetTestSession`).

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsIdeasForSession&highlight=7-8,15-18)]

L'app di esempio include anche un metodo per creare una nuova `Idea` per una determinata sessione. Il controller restituisce:

* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> per un modello non valido.
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> se la sessione non esiste.
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> quando la sessione viene aggiornata con la nuova idea.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_CreateActionResult&highlight=9,16,29)]

In `ApiIdeasControllerTests` sono inclusi tre test di `CreateActionResult`.

Il primo test verifica che per un modello non valido venga restituito un elemento <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsBadRequest_GivenInvalidModel&highlight=7,13-14)]

Il secondo test verifica che venga restituito un elemento <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> se la sessione non esiste.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=5,15,22-23)]

Per un `id` di sessione valido, il test finale verifica che:

* Il metodo restituisca un `ActionResult` di tipo `BrainstormSession`.
* [ActionResult \<T> . Il risultato](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Result*) è <xref:Microsoft.AspNetCore.Mvc.CreatedAtActionResult> . `CreatedAtActionResult` è analogo a una risposta *201 - Creato* con un'intestazione `Location`.
* [ActionResult \<T> . Il valore](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*) è un `BrainstormSession` tipo.
* La chiamata fittizia per aggiornare la sessione, `UpdateAsync(testSession)`, sia stata eseguita. La chiamata del metodo `Verifiable` viene controllata eseguendo `mockRepo.Verify()` nelle asserzioni.
* Siano stati restituiti due oggetti `Idea` per la sessione.
* L'ultimo elemento (`Idea` aggiunta dalla chiamata fittizia a `UpdateAsync`) corrisponda alla `newIdea` aggiunta alla sessione nel test.

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNewlyCreatedIdeaForSession&highlight=20-22,28-34)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

I [controller](xref:mvc/controllers/actions) hanno un ruolo centrale in qualsiasi app ASP.NET Core MVC. Di conseguenza, è importante essere certi che funzionino nel modo previsto. I test automatizzati possono rilevare eventuali errori prima che l'app venga distribuita in un ambiente di produzione.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/testing/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="unit-tests-of-controller-logic"></a>Unit test della logica dei controller

Gli [unit test](/dotnet/articles/core/testing/unit-testing-with-dotnet-test) implicano l'esecuzione di test su una parte di un'app isolandola dall'infrastruttura e dalle dipendenze. Quando si sottopone a unit test la logica del controller, si verificano solo i contenuti di una singola azione e non il comportamento delle relative dipendenze o del framework.

Configurare unit test delle azioni del controller per concentrare l'attenzione sul comportamento del controller. Uno unit test del controller evita scenari come [filtri](xref:mvc/controllers/filters), [routing](xref:fundamentals/routing) e [associazione di modelli](xref:mvc/models/model-binding). I test che verificano le interazioni tra i componenti che collettivamente rispondono a una richiesta vengono gestiti dai *test di integrazione*. Per altre informazioni sui test di integrazione, vedere <xref:test/integration-tests>.

Se si scrivono route e filtri personalizzati, sottoporli a unit test in isolamento e non durante i test relativi a una determinata azione del controller.

Per una dimostrazione degli unit test del controller, esaminare il controller seguente nell'app di esempio. Il controller Home visualizza un elenco di sessioni di brainstorming e consente di creare nuove sessioni con una richiesta POST:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/HomeController.cs?name=snippet_HomeController&highlight=1,5,10,31-32)]

Il controller precedente:

* Segue il [principio delle dipendenze esplicite](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).
* Si aspetta che l'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) fornisca un'istanza di `IBrainstormSessionRepository`.
* Può essere testato con un servizio `IBrainstormSessionRepository` fittizio tramite il framework di un oggetto fittizio, ad esempio [Moq](https://www.nuget.org/packages/Moq/). Un *oggetto fittizio* è un oggetto creato con un set predeterminato di comportamenti di proprietà e metodi usati per il testing. Per altre informazioni, vedere [Introduction to integration tests](xref:test/integration-tests#introduction-to-integration-tests) (Introduzione ai test di integrazione).

Il metodo `HTTP GET Index` non dispone di cicli o rami e chiama un solo metodo. Lo unit test per questa azione:

* Simula il servizio `IBrainstormSessionRepository` usando il metodo `GetTestSessions`. `GetTestSessions` crea due sessioni di brainstorming fittizie con date e nomi di sessione.
* Esegue il metodo `Index`.
* Crea asserzioni sul risultato restituito dal metodo:
  * Viene restituito un <xref:Microsoft.AspNetCore.Mvc.ViewResult>.
  * [ViewDataDictionary.Model](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary.Model*) è di tipo `StormSessionViewModel`.
  * Nel `ViewDataDictionary.Model` sono archiviate due sessioni di brainstorming.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_Index_ReturnsAViewResult_WithAListOfBrainstormSessions&highlight=14-17)]

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_GetTestSessions)]

Il metodo `HTTP POST Index` del controller Home verifica che:

* Quando [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid*) è `false`, il metodo di azione restituisca un elemento *di tipo 400 - Richiesta non valida* <xref:Microsoft.AspNetCore.Mvc.ViewResult> con i dati appropriati.
* Quando `ModelState.IsValid` è `true`:
  * Venga chiamato il metodo `Add` sul repository.
  * Venga restituito un <xref:Microsoft.AspNetCore.Mvc.RedirectToActionResult> con gli argomenti corretti.

È possibile eseguire il test di uno stato del modello non valido aggiungendo gli errori con <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> come illustrato nel primo test riportato di seguito:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_ModelState_ValidOrInvalid&highlight=9,16-17,38-41)]

Quando [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) non è valido, viene restituito lo stesso `ViewResult` come per una richiesta GET. Il test non prova a passare un modello non valido. Il passaggio di un modello non valido non è un approccio corretto, in quanto l'associazione di modelli non è in esecuzione (benché un [test di integrazione](xref:test/integration-tests) usi invece l'associazione di modelli). In questo caso l'associazione di modelli non viene testata. Questi unit test testano solo il codice del metodo di azione.

Il secondo test verifica che quando `ModelState` è valido:

* Venga aggiunto un nuovo elemento `BrainstormSession` (tramite il repository).
* Il metodo restituisca un `RedirectToActionResult` con le proprietà previste.

Le chiamate fittizie che non vengono eseguite sono in genere ignorate, ma la chiamata di `Verifiable` al termine della chiamata setup consente la convalida fittizia nel test. Questa operazione viene eseguita con la chiamata a `mockRepo.Verify`, che non supera il test se non è stato chiamato il metodo previsto.

> [!NOTE]
> La libreria Moq usata in questo esempio consente la combinazione di simulazioni verificabili o "rigide" con simulazioni non verificabili (dette anche simulazioni "generiche" o stub "generici"). Altre informazioni sulla [personalizzazione del comportamento di simulazione con Moq](https://github.com/Moq/moq4/wiki/Quickstart#customizing-mock-behavior).

[SessionController](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/controllers/testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs) nell'app di esempio visualizza informazioni correlate a una sessione di brainstorming specifica. Il controller include la logica per gestire i valori `id` non validi (nell'esempio seguente sono riportati due scenari `return` in proposito). L'istruzione finale `return` restituisce un nuovo elemento `StormSessionViewModel` alla vista (*Controllers/SessionController.cs*):

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs?name=snippet_SessionController&highlight=12-16,18-22,31)]

Gli unit test includono un test per ogni scenario `return` nell'azione `Index` del controller Session:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/SessionControllerTests.cs?name=snippet_SessionControllerTests&highlight=2,11-14,18,31-32,36,50-55)]

Passando al controller Ideas, l'app espone la funzionalità come API Web sulla route `api/ideas`:

* Il metodo `ForSession` restituisce un elenco di idee (`IdeaDTO`) associate a una sessione di brainstorming.
* Il metodo `Create` aggiunge nuove idee a una sessione.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionAndCreate&highlight=1-2,21-22)]

Evitare di restituire direttamente le entità di dominio aziendali tramite chiamate API. Le entità di dominio infatti:

* Spesso includono più dati di quelli richiesti dal client.
* Associano senza necessità il modello di dominio interno dell'app all'API esposta pubblicamente.

Il mapping tra le entità di dominio e i tipi restituiti al client può essere eseguito:

* Manualmente con un'operazione LINQ `Select`, come avviene nell'app di esempio. Per altre informazioni, vedere [LINQ (Language-Integrated Query)](/dotnet/standard/using-linq).
* Automaticamente con una libreria, come [AutoMapper](https://github.com/AutoMapper/AutoMapper).

L'app di esempio esegue quindi una dimostrazione degli unit test per i metodi API `Create` e `ForSession` del controller Ideas.

L' app di esempio contiene due test `ForSession`. Il primo determina se `ForSession` restituisce un <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult> (HTTP non trovato) per una sessione non valida:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests4&highlight=5,7-8,15-16)]

Il secondo test `ForSession` determina se `ForSession` restituisce un elenco di idee (`<List<IdeaDTO>>`) per una sessione valida. I controlli esaminano anche la prima idea per verificare che la relativa proprietà `Name` sia corretta:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests5&highlight=5,7-8,15-18)]

Per testare il comportamento del metodo `Create` quando `ModelState` non è valido, l'app di esempio aggiunge un errore del modello al controller come parte del test. Non provare a testare la convalida o l'associazione di modelli negli unit test. Testare solo il comportamento del metodo di azione rispetto a un `ModelState` non valido:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests1&highlight=7,13)]

Il secondo test di `Create` dipende dal fatto che il repository restituisca `null`, pertanto il repository fittizio è configurato per restituire `null`. Non è necessario creare un database di test (in memoria o con un altro approccio) e creare una query che restituisca questo risultato. Il test può essere eseguito in una singola istruzione, come illustrato dal codice di esempio:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests2&highlight=7-8,15)]

Il terzo test di `Create`, `Create_ReturnsNewlyCreatedIdeaForSession`, verifica che venga chiamato il metodo `UpdateAsync` del repository. La simulazione viene chiamata con `Verifiable` e quindi viene chiamato il metodo `Verify` del repository fittizio per confermare che il metodo verificabile sia stato eseguito. Non spetta allo unit test verificare che il metodo `UpdateAsync` abbia salvato i dati. Questa operazione può essere eseguita con un test di integrazione.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests3&highlight=20-22,28-33)]

## <a name="test-actionresultt"></a>Test ActionResult\<T>

In ASP.NET Core 2,1 o versioni successive [, \<T> ActionResult](xref:web-api/action-return-types#actionresultt-type) ( <xref:Microsoft.AspNetCore.Mvc.ActionResult%601> ) consente di restituire un tipo che deriva da `ActionResult` o restituisce un tipo specifico.

L'app di esempio include un metodo che restituisce un `List<IdeaDTO>` per un determinato `id` di sessione. Se l'`id` di sessione non esiste, il controller restituisce <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>:

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionActionResult&highlight=10,21)]

In `ApiIdeasControllerTests` sono inclusi due test del controller `ForSessionActionResult`.

Il primo test verifica che il controller restituisca un `ActionResult` ma non un elenco non esistente di idee per un `id` di sessione inesistente:

* Il tipo di `ActionResult` è `ActionResult<List<IdeaDTO>>`.
* <xref:Microsoft.AspNetCore.Mvc.ActionResult`1.Result*> è di tipo <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=7,10,13-14)]

Per un `id` di sessione valido, il secondo test verifica che il metodo restituisca:

* Il metodo restituisca un `ActionResult` con un tipo `List<IdeaDTO>`.
* [ActionResult \<T> . Il valore](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*) è un `List<IdeaDTO>` tipo.
* Il primo elemento nell'elenco sia un'idea valida corrispondente all'idea archiviata nella sessione fittizia (ottenuta chiamando `GetTestSession`).

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsIdeasForSession&highlight=7-8,15-18)]

L'app di esempio include anche un metodo per creare una nuova `Idea` per una determinata sessione. Il controller restituisce:

* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> per un modello non valido.
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> se la sessione non esiste.
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> quando la sessione viene aggiornata con la nuova idea.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_CreateActionResult&highlight=9,16,29)]

In `ApiIdeasControllerTests` sono inclusi tre test di `CreateActionResult`.

Il primo test verifica che per un modello non valido venga restituito un elemento <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsBadRequest_GivenInvalidModel&highlight=7,13-14)]

Il secondo test verifica che venga restituito un elemento <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> se la sessione non esiste.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=5,15,22-23)]

Per un `id` di sessione valido, il test finale verifica che:

* Il metodo restituisca un `ActionResult` di tipo `BrainstormSession`.
* [ActionResult \<T> . Il risultato](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Result*) è <xref:Microsoft.AspNetCore.Mvc.CreatedAtActionResult> . `CreatedAtActionResult` è analogo a una risposta *201 - Creato* con un'intestazione `Location`.
* [ActionResult \<T> . Il valore](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*) è un `BrainstormSession` tipo.
* La chiamata fittizia per aggiornare la sessione, `UpdateAsync(testSession)`, sia stata eseguita. La chiamata del metodo `Verifiable` viene controllata eseguendo `mockRepo.Verify()` nelle asserzioni.
* Siano stati restituiti due oggetti `Idea` per la sessione.
* L'ultimo elemento (`Idea` aggiunta dalla chiamata fittizia a `UpdateAsync`) corrisponda alla `newIdea` aggiunta alla sessione nel test.

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNewlyCreatedIdeaForSession&highlight=20-22,28-34)]

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/integration-tests>
* [Creazione ed esecuzione di unit test con Visual Studio](/visualstudio/test/unit-test-your-code)
* [Tested. AspNetCore. Mvc-libreria di test Fluent per ASP.NET Core MVC](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc): libreria di unit test fortemente tipizzata, che fornisce un'interfaccia Fluent per il test di app per le API Web e MVC. (*Non mantenuto o supportato da Microsoft).*
* [JustMockLite](https://github.com/telerik/JustMockLite): Framework fittizio per sviluppatori .NET. (*Non mantenuto o supportato da Microsoft).*

