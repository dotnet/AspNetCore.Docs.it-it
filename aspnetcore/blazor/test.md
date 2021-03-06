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
# <a name="test-components-in-aspnet-core-blazor"></a><span data-ttu-id="2152f-103">Componenti di test in ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="2152f-103">Test components in ASP.NET Core Blazor</span></span>

<span data-ttu-id="2152f-104">Da [](https://egilhansen.com/)</span><span class="sxs-lookup"><span data-stu-id="2152f-104">By: [Egil Hansen](https://egilhansen.com/)</span></span>

<span data-ttu-id="2152f-105">I test sono un aspetto importante della creazione di software stabile e gestibile.</span><span class="sxs-lookup"><span data-stu-id="2152f-105">Testing is an important aspect of building stable and maintainable software.</span></span>

<span data-ttu-id="2152f-106">Per testare un Blazor componente, il *componente* sottoposto a test (taglia) è:</span><span class="sxs-lookup"><span data-stu-id="2152f-106">To test a Blazor component, the *Component Under Test* (CUT) is:</span></span>

* <span data-ttu-id="2152f-107">Sottoposto a rendering con l'input pertinente per il test.</span><span class="sxs-lookup"><span data-stu-id="2152f-107">Rendered with relevant input for the test.</span></span>
* <span data-ttu-id="2152f-108">A seconda del tipo di test eseguito, probabilmente soggetto a interazioni o modifiche.</span><span class="sxs-lookup"><span data-stu-id="2152f-108">Depending on the type of test performed, possibly subject to interaction or modification.</span></span> <span data-ttu-id="2152f-109">È ad esempio possibile attivare i gestori eventi, ad esempio un `onclick` evento per un pulsante.</span><span class="sxs-lookup"><span data-stu-id="2152f-109">For example, event handlers can be triggered, such as an `onclick` event for a button.</span></span>
* <span data-ttu-id="2152f-110">Controllati per i valori previsti.</span><span class="sxs-lookup"><span data-stu-id="2152f-110">Inspected for expected values.</span></span>

## <a name="test-approaches"></a><span data-ttu-id="2152f-111">Approcci di test</span><span class="sxs-lookup"><span data-stu-id="2152f-111">Test approaches</span></span>

<span data-ttu-id="2152f-112">Due approcci comuni per i test dei Blazor componenti sono i test end-to-end (E2E) e gli unit test:</span><span class="sxs-lookup"><span data-stu-id="2152f-112">Two common approaches for testing Blazor components are end-to-end (E2E) testing and unit testing:</span></span>

* <span data-ttu-id="2152f-113">**Unit test**: gli [unit test](/dotnet/core/testing/) vengono scritti con una libreria di unit test che fornisce:</span><span class="sxs-lookup"><span data-stu-id="2152f-113">**Unit testing**: [Unit tests](/dotnet/core/testing/) are written with a unit testing library that provides:</span></span>
  * <span data-ttu-id="2152f-114">Rendering di componenti.</span><span class="sxs-lookup"><span data-stu-id="2152f-114">Component rendering.</span></span>
  * <span data-ttu-id="2152f-115">Ispezione dell'output e dello stato dei componenti.</span><span class="sxs-lookup"><span data-stu-id="2152f-115">Inspection of component output and state.</span></span>
  * <span data-ttu-id="2152f-116">Attivazione di gestori eventi e metodi del ciclo di vita.</span><span class="sxs-lookup"><span data-stu-id="2152f-116">Triggering of event handlers and life cycle methods.</span></span>
  * <span data-ttu-id="2152f-117">Asserzioni che il comportamento del componente è corretto.</span><span class="sxs-lookup"><span data-stu-id="2152f-117">Assertions that component behavior is correct.</span></span>

  <span data-ttu-id="2152f-118">[bUnit](https://github.com/egil/bUnit) è un esempio di libreria che consente di eseguire Razor unit test dei componenti.</span><span class="sxs-lookup"><span data-stu-id="2152f-118">[bUnit](https://github.com/egil/bUnit) is an example of a library that enables Razor component unit testing.</span></span>

* <span data-ttu-id="2152f-119">**Test di E2E**: un test runner esegue un' Blazor app che contiene il taglia e automatizza un'istanza del browser.</span><span class="sxs-lookup"><span data-stu-id="2152f-119">**E2E testing**: A test runner runs a Blazor app containing the CUT and automates a browser instance.</span></span> <span data-ttu-id="2152f-120">Lo strumento di test esamina e interagisce con il taglio tramite il browser.</span><span class="sxs-lookup"><span data-stu-id="2152f-120">The testing tool inspects and interacts with the CUT through the browser.</span></span> <span data-ttu-id="2152f-121">[Selenium](https://github.com/SeleniumHQ/selenium) è un esempio di Framework di test E2E che può essere usato con le Blazor app.</span><span class="sxs-lookup"><span data-stu-id="2152f-121">[Selenium](https://github.com/SeleniumHQ/selenium) is an example of an E2E testing framework that can be used with Blazor apps.</span></span>

<span data-ttu-id="2152f-122">Negli unit test Blazor viene richiesto solo il componente ( Razor /c #).</span><span class="sxs-lookup"><span data-stu-id="2152f-122">In unit testing, only the Blazor component (Razor/C#) is involved.</span></span> <span data-ttu-id="2152f-123">Le dipendenze esterne, ad esempio servizi e interoperabilità JS, devono essere simulate.</span><span class="sxs-lookup"><span data-stu-id="2152f-123">External dependencies, such as services and JS interop, must be mocked.</span></span> <span data-ttu-id="2152f-124">Nel test di E2E, il Blazor componente e l'intera infrastruttura ausiliaria fanno parte del test, tra cui CSS, js e le API DOM e browser.</span><span class="sxs-lookup"><span data-stu-id="2152f-124">In E2E testing, the Blazor component and all of it's auxiliary infrastructure are part of the test, including CSS, JS, and the DOM and browser APIs.</span></span>

<span data-ttu-id="2152f-125">L' *ambito del test* descrive l'ampiezza dei test.</span><span class="sxs-lookup"><span data-stu-id="2152f-125">*Test scope* describes how extensive the tests are.</span></span> <span data-ttu-id="2152f-126">L'ambito di test ha in genere un impatto sulla velocità dei test.</span><span class="sxs-lookup"><span data-stu-id="2152f-126">Test scope typically has an influence on the speed of the tests.</span></span> <span data-ttu-id="2152f-127">Gli unit test vengono eseguiti in un subset dei sottosistemi dell'applicazione e in genere vengono eseguiti in millisecondi.</span><span class="sxs-lookup"><span data-stu-id="2152f-127">Unit tests run on a subset of the app's subsystems and usually execute in milliseconds.</span></span> <span data-ttu-id="2152f-128">I test di E2E, che testano un ampio gruppo di sottosistemi dell'app, possono richiedere alcuni secondi per il completamento.</span><span class="sxs-lookup"><span data-stu-id="2152f-128">E2E tests, which test a broad group of the app's subsystems, can take several seconds to complete.</span></span> 

<span data-ttu-id="2152f-129">Il testing unità consente inoltre di accedere all'istanza del taglio, consentendo l'ispezione e la verifica dello stato interno del componente.</span><span class="sxs-lookup"><span data-stu-id="2152f-129">Unit testing also provides access to the instance of the CUT, allowing for inspection and verification of the component's internal state.</span></span> <span data-ttu-id="2152f-130">Questo in genere non è possibile nei test di E2E.</span><span class="sxs-lookup"><span data-stu-id="2152f-130">This normally isn't possible in E2E testing.</span></span>

<span data-ttu-id="2152f-131">Per quanto riguarda l'ambiente del componente, i test di E2E devono verificare che sia stato raggiunto lo stato di ambiente previsto prima dell'avvio della verifica.</span><span class="sxs-lookup"><span data-stu-id="2152f-131">With regard to the component's environment, E2E tests must make sure that the expected environmental state has been reached before verification starts.</span></span> <span data-ttu-id="2152f-132">In caso contrario, il risultato è imprevedibile.</span><span class="sxs-lookup"><span data-stu-id="2152f-132">Otherwise, the result is unpredictable.</span></span> <span data-ttu-id="2152f-133">Negli unit test, il rendering del taglio e il ciclo di vita del test sono più integrati, migliorando la stabilità dei test.</span><span class="sxs-lookup"><span data-stu-id="2152f-133">In unit testing, the rendering of the CUT and the life cycle of the test are more integrated, which improves test stability.</span></span>

<span data-ttu-id="2152f-134">Il test di E2E comporta l'avvio di più processi, l'I/O su rete e su disco e altre attività di sottosistema che spesso comportano una scarsa affidabilità dei test.</span><span class="sxs-lookup"><span data-stu-id="2152f-134">E2E testing involves launching multiple processes, network and disk I/O, and other subsystem activity that often lead to poor test reliability.</span></span> <span data-ttu-id="2152f-135">Gli unit test sono in genere isolati da questi tipi di problemi.</span><span class="sxs-lookup"><span data-stu-id="2152f-135">Unit tests are typically insulated from these sorts of issues.</span></span>

<span data-ttu-id="2152f-136">Nella tabella seguente sono riepilogate le differenze tra i due approcci di testing.</span><span class="sxs-lookup"><span data-stu-id="2152f-136">The following table summarizes the difference between the two testing approaches.</span></span>

| <span data-ttu-id="2152f-137">Funzionalità</span><span class="sxs-lookup"><span data-stu-id="2152f-137">Capability</span></span>                       | <span data-ttu-id="2152f-138">Unit test</span><span class="sxs-lookup"><span data-stu-id="2152f-138">Unit testing</span></span>                     | <span data-ttu-id="2152f-139">Test di E2E</span><span class="sxs-lookup"><span data-stu-id="2152f-139">E2E testing</span></span>                             |
| -------------------------------- | -------------------------------- | --------------------------------------- |
| <span data-ttu-id="2152f-140">Ambito del test</span><span class="sxs-lookup"><span data-stu-id="2152f-140">Test scope</span></span>                       | <span data-ttu-id="2152f-141">Blazor solo componente ( Razor /c #)</span><span class="sxs-lookup"><span data-stu-id="2152f-141">Blazor component (Razor/C#) only</span></span> | <span data-ttu-id="2152f-142">Blazor componente ( Razor /c #) con CSS/JS</span><span class="sxs-lookup"><span data-stu-id="2152f-142">Blazor component (Razor/C#) with CSS/JS</span></span> |
| <span data-ttu-id="2152f-143">Tempo di esecuzione del test</span><span class="sxs-lookup"><span data-stu-id="2152f-143">Test execution time</span></span>              | <span data-ttu-id="2152f-144">Millisecondi</span><span class="sxs-lookup"><span data-stu-id="2152f-144">Milliseconds</span></span>                     | <span data-ttu-id="2152f-145">Secondi</span><span class="sxs-lookup"><span data-stu-id="2152f-145">Seconds</span></span>                                 |
| <span data-ttu-id="2152f-146">Accesso all'istanza del componente</span><span class="sxs-lookup"><span data-stu-id="2152f-146">Access to the component instance</span></span> | <span data-ttu-id="2152f-147">Sì</span><span class="sxs-lookup"><span data-stu-id="2152f-147">Yes</span></span>                              | <span data-ttu-id="2152f-148">No</span><span class="sxs-lookup"><span data-stu-id="2152f-148">No</span></span>                                      |
| <span data-ttu-id="2152f-149">Sensibile all'ambiente</span><span class="sxs-lookup"><span data-stu-id="2152f-149">Sensitive to the environment</span></span>     | <span data-ttu-id="2152f-150">No</span><span class="sxs-lookup"><span data-stu-id="2152f-150">No</span></span>                               | <span data-ttu-id="2152f-151">Sì</span><span class="sxs-lookup"><span data-stu-id="2152f-151">Yes</span></span>                                     |
| <span data-ttu-id="2152f-152">Affidabilità</span><span class="sxs-lookup"><span data-stu-id="2152f-152">Reliability</span></span>                      | <span data-ttu-id="2152f-153">Più affidabile</span><span class="sxs-lookup"><span data-stu-id="2152f-153">More reliable</span></span>                    | <span data-ttu-id="2152f-154">Meno affidabile</span><span class="sxs-lookup"><span data-stu-id="2152f-154">Less reliable</span></span>                           |

## <a name="choose-the-most-appropriate-test-approach"></a><span data-ttu-id="2152f-155">Scegliere l'approccio di test più appropriato</span><span class="sxs-lookup"><span data-stu-id="2152f-155">Choose the most appropriate test approach</span></span>

<span data-ttu-id="2152f-156">Si consideri lo scenario in cui si sceglie il tipo di test da eseguire.</span><span class="sxs-lookup"><span data-stu-id="2152f-156">Consider the scenario when choosing the type of testing to perform.</span></span> <span data-ttu-id="2152f-157">Alcune considerazioni sono descritte nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="2152f-157">Some considerations are described in the following table.</span></span>

| <span data-ttu-id="2152f-158">Scenario</span><span class="sxs-lookup"><span data-stu-id="2152f-158">Scenario</span></span> | <span data-ttu-id="2152f-159">Approccio consigliato</span><span class="sxs-lookup"><span data-stu-id="2152f-159">Suggested approach</span></span> | <span data-ttu-id="2152f-160">Commenti</span><span class="sxs-lookup"><span data-stu-id="2152f-160">Remarks</span></span> |
| -------- | ------------------ | ------- |
| <span data-ttu-id="2152f-161">Componente senza logica di interoperabilità JS</span><span class="sxs-lookup"><span data-stu-id="2152f-161">Component without JS interop logic</span></span> | <span data-ttu-id="2152f-162">Unit test</span><span class="sxs-lookup"><span data-stu-id="2152f-162">Unit testing</span></span> | <span data-ttu-id="2152f-163">Quando non esiste alcuna dipendenza dall'interoperabilità JS in un Blazor componente, il componente può essere testato senza accedere a js o all'API DOM.</span><span class="sxs-lookup"><span data-stu-id="2152f-163">When there's no dependency on JS interop in a Blazor component, the component can be tested without access to JS or the DOM API.</span></span> <span data-ttu-id="2152f-164">In questo scenario, non vi sono svantaggi nella scelta del testing unità.</span><span class="sxs-lookup"><span data-stu-id="2152f-164">In this scenario, there are no disadvantages to choosing unit testing.</span></span> |
| <span data-ttu-id="2152f-165">Componente con logica di interoperabilità JS semplice</span><span class="sxs-lookup"><span data-stu-id="2152f-165">Component with simple JS interop logic</span></span> | <span data-ttu-id="2152f-166">Unit test</span><span class="sxs-lookup"><span data-stu-id="2152f-166">Unit testing</span></span> | <span data-ttu-id="2152f-167">È frequente che i componenti eseguono una query sul DOM o attivano le animazioni tramite l'interoperabilità JS.</span><span class="sxs-lookup"><span data-stu-id="2152f-167">It's common for components to query the DOM or trigger animations through JS interop.</span></span> <span data-ttu-id="2152f-168">Il testing unità è in genere preferibile in questo scenario, poiché è semplice simulare l'interazione JS tramite l' <xref:Microsoft.JSInterop.IJSRuntime> interfaccia.</span><span class="sxs-lookup"><span data-stu-id="2152f-168">Unit testing is usually preferred in this scenario, since it's straightforward to mock the JS interaction through the <xref:Microsoft.JSInterop.IJSRuntime> interface.</span></span> |
| <span data-ttu-id="2152f-169">Componente che dipende dal codice JS complesso</span><span class="sxs-lookup"><span data-stu-id="2152f-169">Component that depends on complex JS code</span></span> | <span data-ttu-id="2152f-170">Unit test e test JS distinti</span><span class="sxs-lookup"><span data-stu-id="2152f-170">Unit testing and separate JS testing</span></span> | <span data-ttu-id="2152f-171">Se un componente usa l'interoperabilità JS per chiamare una libreria JS grande o complessa, ma l'interazione tra il Blazor componente e la libreria JS è semplice, l'approccio migliore è probabilmente trattare il componente e la libreria JS o il codice come due parti separate e testarle singolarmente.</span><span class="sxs-lookup"><span data-stu-id="2152f-171">If a component uses JS interop to call a large or complex JS library but the interaction between the Blazor component and JS library is simple, then the best approach is likely to treat the component and JS library or code as two separate parts and test each individually.</span></span> <span data-ttu-id="2152f-172">Testare il Blazor componente con una libreria di unit test e testare js con una libreria di test js.</span><span class="sxs-lookup"><span data-stu-id="2152f-172">Test the Blazor component with a unit testing library, and test the JS with a JS testing library.</span></span> |
| <span data-ttu-id="2152f-173">Componente con logica che dipende dalla manipolazione JS del DOM del browser</span><span class="sxs-lookup"><span data-stu-id="2152f-173">Component with logic that depends on JS manipulation of the browser DOM</span></span> | <span data-ttu-id="2152f-174">Test di E2E</span><span class="sxs-lookup"><span data-stu-id="2152f-174">E2E testing</span></span> | <span data-ttu-id="2152f-175">Quando la funzionalità di un componente dipende da JS e dalla relativa manipolazione del DOM, verificare sia il codice JS sia il Blazor codice insieme in un test di E2E.</span><span class="sxs-lookup"><span data-stu-id="2152f-175">When a component's functionality is dependent on JS and its manipulation of the DOM, verify both the JS and Blazor code together in an E2E test.</span></span> <span data-ttu-id="2152f-176">Si tratta dell'approccio Blazor adottato dagli sviluppatori del Framework con la Blazor logica di rendering del browser, che include codice C# e JS strettamente accoppiato.</span><span class="sxs-lookup"><span data-stu-id="2152f-176">This is the approach that the Blazor framework developers have taken with Blazor's browser rendering logic, which has tightly-coupled C# and JS code.</span></span> <span data-ttu-id="2152f-177">Il codice C# e JS deve collaborare per eseguire correttamente il rendering dei Blazor componenti in un browser.</span><span class="sxs-lookup"><span data-stu-id="2152f-177">The C# and JS code must work together to correctly render Blazor components in a browser.</span></span>
| <span data-ttu-id="2152f-178">Componente che dipende dalla libreria di componenti di terze parti con dipendenze difficili da simulare</span><span class="sxs-lookup"><span data-stu-id="2152f-178">Component that depends on 3rd party component library with hard-to-mock dependencies</span></span> | <span data-ttu-id="2152f-179">Test di E2E</span><span class="sxs-lookup"><span data-stu-id="2152f-179">E2E testing</span></span> | <span data-ttu-id="2152f-180">Quando la funzionalità di un componente dipende da una libreria di componenti di terze parti con dipendenze difficili da simulare, ad esempio l'interoperabilità JS, il test di E2E potrebbe essere l'unica opzione per testare il componente.</span><span class="sxs-lookup"><span data-stu-id="2152f-180">When a component's functionality is dependent on a 3rd party component library that has hard-to-mock dependencies, such as JS interop, E2E testing might be the only option to test the component.</span></span> |

## <a name="test-components-with-bunit"></a><span data-ttu-id="2152f-181">Testare i componenti con bUnit</span><span class="sxs-lookup"><span data-stu-id="2152f-181">Test components with bUnit</span></span>

<span data-ttu-id="2152f-182">Non esiste un Framework ufficiale di test Microsoft per Blazor , ma il progetto gestito dalla community [bUnit](https://github.com/egil/bUnit) offre un modo pratico per unit test Blazor componenti.</span><span class="sxs-lookup"><span data-stu-id="2152f-182">There's no official Microsoft testing framework for Blazor, but the community-driven project [bUnit](https://github.com/egil/bUnit) provides a convenient way to unit test Blazor components.</span></span>

> [!NOTE]
> <span data-ttu-id="2152f-183">bUnit è una libreria di test di terze parti e non è supportata né gestita da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="2152f-183">bUnit is a third-party testing library and isn't supported or maintained by Microsoft.</span></span>

<span data-ttu-id="2152f-184">bUnit funziona con Framework di test di uso generico, ad esempio [MSTest](/dotnet/core/testing/unit-testing-with-mstest), [NUnit](https://nunit.org/)e [xUnit](https://xunit.github.io/).</span><span class="sxs-lookup"><span data-stu-id="2152f-184">bUnit works with general-purpose testing frameworks, such as [MSTest](/dotnet/core/testing/unit-testing-with-mstest), [NUnit](https://nunit.org/), and [xUnit](https://xunit.github.io/).</span></span> <span data-ttu-id="2152f-185">Questi framework di test fanno in modo che i test di bUnit abbiano un aspetto analogo agli unit test normali.</span><span class="sxs-lookup"><span data-stu-id="2152f-185">These testing frameworks make bUnit tests look and feel like regular unit tests.</span></span> <span data-ttu-id="2152f-186">i test bUnit integrati con un Framework di test per utilizzo generico vengono in genere eseguiti con:</span><span class="sxs-lookup"><span data-stu-id="2152f-186">bUnit tests integrated with a general-purpose testing framework are ordinarily executed with:</span></span>

* <span data-ttu-id="2152f-187">[Esplora test di Visual Studio](/visualstudio/test/run-unit-tests-with-test-explorer).</span><span class="sxs-lookup"><span data-stu-id="2152f-187">[Visual Studio's Test Explorer](/visualstudio/test/run-unit-tests-with-test-explorer).</span></span>
* <span data-ttu-id="2152f-188">[`dotnet test`](/dotnet/core/tools/dotnet-test) Comando CLI in una shell comandi.</span><span class="sxs-lookup"><span data-stu-id="2152f-188">[`dotnet test`](/dotnet/core/tools/dotnet-test) CLI command in a command shell.</span></span>
* <span data-ttu-id="2152f-189">Una pipeline di test automatizzato di DevOps.</span><span class="sxs-lookup"><span data-stu-id="2152f-189">An automated DevOps testing pipeline.</span></span>

> [!NOTE]
> <span data-ttu-id="2152f-190">I concetti di test e le implementazioni di test in diversi framework di test sono simili, ma non identici.</span><span class="sxs-lookup"><span data-stu-id="2152f-190">Test concepts and test implementations across different test frameworks are similar but not identical.</span></span> <span data-ttu-id="2152f-191">Per informazioni aggiuntive, vedere la documentazione del Framework di test.</span><span class="sxs-lookup"><span data-stu-id="2152f-191">Refer to the test framework's documentation for guidance.</span></span>

<span data-ttu-id="2152f-192">Di seguito viene illustrata la struttura di un test bUnit sul `Counter` componente in un'app in base a un [ Blazor modello di progetto](xref:blazor/project-structure).</span><span class="sxs-lookup"><span data-stu-id="2152f-192">The following demonstrates the structure of a bUnit test on the `Counter` component in an app based on a [Blazor project template](xref:blazor/project-structure).</span></span> <span data-ttu-id="2152f-193">Il `Counter` componente Visualizza e incrementa un contatore in base all'utente che seleziona un pulsante nella pagina:</span><span class="sxs-lookup"><span data-stu-id="2152f-193">The `Counter` component displays and increments a counter based on the user selecting a button in the page:</span></span>

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

<span data-ttu-id="2152f-194">Il test bUnit seguente verifica che il contatore del taglio venga incrementato correttamente quando si seleziona il pulsante:</span><span class="sxs-lookup"><span data-stu-id="2152f-194">The following bUnit test verifies that the CUT's counter is incremented correctly when the button is selected:</span></span>

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

<span data-ttu-id="2152f-195">Le azioni seguenti si verificano a ogni passaggio del test:</span><span class="sxs-lookup"><span data-stu-id="2152f-195">The following actions take place at each step of the test:</span></span>

* <span data-ttu-id="2152f-196">*Arrange*: il `Counter` componente viene sottoposto a rendering usando bUnit `TestContext` .</span><span class="sxs-lookup"><span data-stu-id="2152f-196">*Arrange*: The `Counter` component is rendered using bUnit's `TestContext`.</span></span> <span data-ttu-id="2152f-197">L'elemento Paragraph del taglio ( `<p>` ) viene trovato e assegnato a `paraElm` .</span><span class="sxs-lookup"><span data-stu-id="2152f-197">The CUT's paragraph element (`<p>`) is found and assigned to `paraElm`.</span></span>

* <span data-ttu-id="2152f-198">*Act*: l'elemento del pulsante ( `<button>` ) viene individuato e quindi selezionato chiamando `Click` , che deve incrementare il contatore e aggiornare il contenuto del tag di paragrafo ( `<p>` ).</span><span class="sxs-lookup"><span data-stu-id="2152f-198">*Act*: The button's element (`<button>`) is located and then selected by calling `Click`, which should increment the counter and update the content of the paragraph tag (`<p>`).</span></span> <span data-ttu-id="2152f-199">Il contenuto di testo dell'elemento Paragraph viene ottenuto chiamando `TextContent` .</span><span class="sxs-lookup"><span data-stu-id="2152f-199">The paragraph element text content is obtained by calling `TextContent`.</span></span>

* <span data-ttu-id="2152f-200">*Assert*: `MarkupMatches` viene chiamato sul contenuto di testo per verificare che corrisponda alla stringa prevista, ovvero `Current count: 1` .</span><span class="sxs-lookup"><span data-stu-id="2152f-200">*Assert*: `MarkupMatches` is called on the text content to verify that it matches the expected string, which is `Current count: 1`.</span></span>

> [!NOTE]
> <span data-ttu-id="2152f-201">Il `MarkupMatches` Metodo Assert è diverso da un'asserzione di confronto di stringhe normale (ad esempio, `Assert.Equal("Current count: 1", paraElmText);` ) `MarkupMatches` esegue un confronto semantico tra l'input e il markup HTML previsto.</span><span class="sxs-lookup"><span data-stu-id="2152f-201">The `MarkupMatches` assert method differs from a regular string comparison assertion (for example, `Assert.Equal("Current count: 1", paraElmText);`) `MarkupMatches` performs a semantic comparison of the input and expected HTML markup.</span></span> <span data-ttu-id="2152f-202">Un confronto semantico è consapevole della semantica HTML, il che significa che gli spazi vuoti non significativi vengono ignorati.</span><span class="sxs-lookup"><span data-stu-id="2152f-202">A semantic comparison is aware of HTML semantics, meaning things like insignificant whitespace is ignored.</span></span> <span data-ttu-id="2152f-203">Ciò comporta test più stabili.</span><span class="sxs-lookup"><span data-stu-id="2152f-203">This results in more stable tests.</span></span> <span data-ttu-id="2152f-204">Per altre informazioni, vedere [personalizzazione del confronto tra HTML semantico](https://bunit.egilhansen.com/docs/verification/semantic-html-comparison).</span><span class="sxs-lookup"><span data-stu-id="2152f-204">For more information, see [Customizing the Semantic HTML Comparison](https://bunit.egilhansen.com/docs/verification/semantic-html-comparison).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="2152f-205">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="2152f-205">Additional resources</span></span>

* <span data-ttu-id="2152f-206">[Introduzione con bUnit](https://bunit.egilhansen.com/docs/getting-started/): le istruzioni bUnit includono indicazioni sulla creazione di un progetto di test, sul riferimento ai pacchetti del Framework di test e sulla compilazione e sull'esecuzione di test.</span><span class="sxs-lookup"><span data-stu-id="2152f-206">[Getting Started with bUnit](https://bunit.egilhansen.com/docs/getting-started/): bUnit instructions include guidance on creating a test project, referencing testing framework packages, and building and running tests.</span></span>
