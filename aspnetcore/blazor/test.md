---
title: Componenti di test in ASP.NET Core Blazor
author: guardrex
description: Informazioni su come testare componments nelle Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/10/2020
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
uid: blazor/test
ms.openlocfilehash: 1a7b1114934f4fe7006d60bdbd0f06792d2c6935
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394551"
---
# <a name="test-components-in-aspnet-core-blazor"></a>Componenti di test in ASP.NET Core Blazor

Da [](https://egilhansen.com/)

I test sono un aspetto importante della creazione di software stabile e gestibile.

Per testare un Blazor componente, il *componente* sottoposto a test (taglia) è:

* Sottoposto a rendering con l'input pertinente per il test.
* A seconda del tipo di test eseguito, probabilmente soggetto a interazioni o modifiche. È ad esempio possibile attivare i gestori eventi, ad esempio un `onclick` evento per un pulsante.
* Controllati per i valori previsti.

## <a name="test-approaches"></a>Approcci di test

Due approcci comuni per i test dei Blazor componenti sono i test end-to-end (E2E) e gli unit test:

* **Unit test**: gli [unit test](/dotnet/core/testing/) vengono scritti con una libreria di unit test che fornisce:
  * Rendering di componenti.
  * Ispezione dell'output e dello stato dei componenti.
  * Attivazione di gestori eventi e metodi del ciclo di vita.
  * Asserzioni che il comportamento del componente è corretto.

  [bUnit](https://github.com/egil/bUnit) è un esempio di libreria che consente di eseguire Razor unit test dei componenti.

* **Test di E2E**: un test runner esegue un' Blazor app che contiene il taglia e automatizza un'istanza del browser. Lo strumento di test esamina e interagisce con il taglio tramite il browser. [Selenium](https://github.com/SeleniumHQ/selenium) è un esempio di Framework di test E2E che può essere usato con le Blazor app.

Negli unit test Blazor viene richiesto solo il componente ( Razor /c #). Le dipendenze esterne, ad esempio servizi e interoperabilità JS, devono essere simulate. Nel test di E2E, il Blazor componente e l'intera infrastruttura ausiliaria fanno parte del test, tra cui CSS, js e le API DOM e browser.

L' *ambito del test* descrive l'ampiezza dei test. L'ambito di test ha in genere un impatto sulla velocità dei test. Gli unit test vengono eseguiti in un subset dei sottosistemi dell'applicazione e in genere vengono eseguiti in millisecondi. I test di E2E, che testano un ampio gruppo di sottosistemi dell'app, possono richiedere alcuni secondi per il completamento. 

Il testing unità consente inoltre di accedere all'istanza del taglio, consentendo l'ispezione e la verifica dello stato interno del componente. Questo in genere non è possibile nei test di E2E.

Per quanto riguarda l'ambiente del componente, i test di E2E devono verificare che sia stato raggiunto lo stato di ambiente previsto prima dell'avvio della verifica. In caso contrario, il risultato è imprevedibile. Negli unit test, il rendering del taglio e il ciclo di vita del test sono più integrati, migliorando la stabilità dei test.

Il test di E2E comporta l'avvio di più processi, l'I/O su rete e su disco e altre attività di sottosistema che spesso comportano una scarsa affidabilità dei test. Gli unit test sono in genere isolati da questi tipi di problemi.

Nella tabella seguente sono riepilogate le differenze tra i due approcci di testing.

| Funzionalità                       | Unit test                     | Test di E2E                             |
| -------------------------------- | -------------------------------- | --------------------------------------- |
| Ambito del test                       | Blazor solo componente ( Razor /c #) | Blazor componente ( Razor /c #) con CSS/JS |
| Tempo di esecuzione del test              | Millisecondi                     | Secondi                                 |
| Accesso all'istanza del componente | Sì                              | No                                      |
| Sensibile all'ambiente     | No                               | Sì                                     |
| Affidabilità                      | Più affidabile                    | Meno affidabile                           |

## <a name="choose-the-most-appropriate-test-approach"></a>Scegliere l'approccio di test più appropriato

Si consideri lo scenario in cui si sceglie il tipo di test da eseguire. Alcune considerazioni sono descritte nella tabella seguente.

| Scenario | Approccio consigliato | Commenti |
| -------- | ------------------ | ------- |
| Componente senza logica di interoperabilità JS | Unit test | Quando non esiste alcuna dipendenza dall'interoperabilità JS in un Blazor componente, il componente può essere testato senza accedere a js o all'API DOM. In questo scenario, non vi sono svantaggi nella scelta del testing unità. |
| Componente con logica di interoperabilità JS semplice | Unit test | È frequente che i componenti eseguono una query sul DOM o attivano le animazioni tramite l'interoperabilità JS. Il testing unità è in genere preferibile in questo scenario, poiché è semplice simulare l'interazione JS tramite l' <xref:Microsoft.JSInterop.IJSRuntime> interfaccia. |
| Componente che dipende dal codice JS complesso | Unit test e test JS distinti | Se un componente usa l'interoperabilità JS per chiamare una libreria JS grande o complessa, ma l'interazione tra il Blazor componente e la libreria JS è semplice, l'approccio migliore è probabilmente trattare il componente e la libreria JS o il codice come due parti separate e testarle singolarmente. Testare il Blazor componente con una libreria di unit test e testare js con una libreria di test js. |
| Componente con logica che dipende dalla manipolazione JS del DOM del browser | Test di E2E | Quando la funzionalità di un componente dipende da JS e dalla relativa manipolazione del DOM, verificare sia il codice JS sia il Blazor codice insieme in un test di E2E. Si tratta dell'approccio Blazor adottato dagli sviluppatori del Framework con la Blazor logica di rendering del browser, che include codice C# e JS strettamente accoppiato. Il codice C# e JS deve collaborare per eseguire correttamente il rendering dei Blazor componenti in un browser.
| Componente che dipende dalla libreria di componenti di terze parti con dipendenze difficili da simulare | Test di E2E | Quando la funzionalità di un componente dipende da una libreria di componenti di terze parti con dipendenze difficili da simulare, ad esempio l'interoperabilità JS, il test di E2E potrebbe essere l'unica opzione per testare il componente. |

## <a name="test-components-with-bunit"></a>Testare i componenti con bUnit

Non esiste un Framework ufficiale di test Microsoft per Blazor , ma il progetto gestito dalla community [bUnit](https://github.com/egil/bUnit) offre un modo pratico per unit test Blazor componenti.

> [!NOTE]
> bUnit è una libreria di test di terze parti e non è supportata né gestita da Microsoft.

bUnit funziona con Framework di test di uso generico, ad esempio [MSTest](/dotnet/core/testing/unit-testing-with-mstest), [NUnit](https://nunit.org/)e [xUnit](https://xunit.github.io/). Questi framework di test fanno in modo che i test di bUnit abbiano un aspetto analogo agli unit test normali. i test bUnit integrati con un Framework di test per utilizzo generico vengono in genere eseguiti con:

* [Esplora test di Visual Studio](/visualstudio/test/run-unit-tests-with-test-explorer).
* [`dotnet test`](/dotnet/core/tools/dotnet-test) Comando CLI in una shell comandi.
* Una pipeline di test automatizzato di DevOps.

> [!NOTE]
> I concetti di test e le implementazioni di test in diversi framework di test sono simili, ma non identici. Per informazioni aggiuntive, vedere la documentazione del Framework di test.

Di seguito viene illustrata la struttura di un test bUnit sul `Counter` componente in un'app in base a un [ Blazor modello di progetto](xref:blazor/project-structure). Il `Counter` componente Visualizza e incrementa un contatore in base all'utente che seleziona un pulsante nella pagina:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

Il test bUnit seguente verifica che il contatore del taglio venga incrementato correttamente quando si seleziona il pulsante:

```csharp
[Fact]
public void CounterShouldIncrementWhenSelected()
{
    // Arrange
    using var ctx = new TestContext();
    var cut = ctx.RenderComponent<Counter>();
    var paraElm = cut.Find("p");

    // Act
    cut.Find("button").Click();
    var paraElmText = paraElm.TextContent;

    // Assert
    paraElmText.MarkupMatches("Current count: 1");
}
```

Le azioni seguenti si verificano a ogni passaggio del test:

* *Arrange*: il `Counter` componente viene sottoposto a rendering usando bUnit `TestContext` . L'elemento Paragraph del taglio ( `<p>` ) viene trovato e assegnato a `paraElm` .

* *Act*: l'elemento del pulsante ( `<button>` ) viene individuato e quindi selezionato chiamando `Click` , che deve incrementare il contatore e aggiornare il contenuto del tag di paragrafo ( `<p>` ). Il contenuto di testo dell'elemento Paragraph viene ottenuto chiamando `TextContent` .

* *Assert*: `MarkupMatches` viene chiamato sul contenuto di testo per verificare che corrisponda alla stringa prevista, ovvero `Current count: 1` .

> [!NOTE]
> Il `MarkupMatches` Metodo Assert è diverso da un'asserzione di confronto di stringhe normale (ad esempio, `Assert.Equal("Current count: 1", paraElmText);` ) `MarkupMatches` esegue un confronto semantico tra l'input e il markup HTML previsto. Un confronto semantico è consapevole della semantica HTML, il che significa che gli spazi vuoti non significativi vengono ignorati. Ciò comporta test più stabili. Per altre informazioni, vedere [personalizzazione del confronto tra HTML semantico](https://bunit.egilhansen.com/docs/verification/semantic-html-comparison).

## <a name="additional-resources"></a>Risorse aggiuntive

* [Introduzione con bUnit](https://bunit.egilhansen.com/docs/getting-started/): le istruzioni bUnit includono indicazioni sulla creazione di un progetto di test, sul riferimento ai pacchetti del Framework di test e sulla compilazione e sull'esecuzione di test.
