---
title: Componenti di visualizzazione in ASP.NET Core
author: rick-anderson
description: Informazioni sull'uso dei componenti di visualizzazione in ASP.NET Core e su come aggiungere tali componenti alle app.
ms.author: riande
ms.custom: mvc
ms.date: 12/18/2019
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
uid: mvc/views/view-components
ms.openlocfilehash: 1d0e0da3a5d047d7457e7ca7587c81029e33cb69
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586007"
---
# <a name="view-components-in-aspnet-core"></a>Componenti di visualizzazione in ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/view-components/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="view-components"></a>Componenti di visualizzazione

I componenti di visualizzazione hanno aspetti comuni con le visualizzazioni parziali, ma sono molto più efficienti. I componenti di visualizzazione non usano l'associazione di modelli. Dipendono soltanto dai dati specificati in fase di chiamata. Questo articolo è stato scritto usando i controller e le visualizzazioni, ma i componenti di visualizzazione funzionano anche con le Razor pagine.

Un componente di visualizzazione:

* Esegue il rendering di un blocco anziché di un'intera risposta.
* Include la stessa separazione dei concetti e gli stessi vantaggi per i test individuati nel controller e nella visualizzazione.
* Può contenere parametri e logica di business.
* In genere viene richiamato da una pagina di layout.

I componenti di visualizzazione possono essere impiegati in un punto qualsiasi della logica di rendering riutilizzabile che risulta troppo complessa per una visualizzazione parziale, ad esempio:

* Menu di spostamento dinamici
* Tag cloud (nel quale viene eseguita una query del database)
* Pannello di accesso
* Carrello acquisti
* Articoli recentemente pubblicati
* Contenuto dell'intestazione laterale in un blog tradizionale
* Pannello di accesso che viene eseguito in ogni pagina e che visualizza il collegamento di accesso o di disconnessione, a seconda dello stato di accesso dell'utente

Un componente di visualizzazione è costituito da due parti: la classe (in genere derivata da [ViewComponent](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponent)) e il risultato restituito (in genere una visualizzazione). Come per i controller, un componente di visualizzazione può essere un oggetto POCO. Molti sviluppatori preferiscono tuttavia sfruttare i metodi e le proprietà disponibili derivando da `ViewComponent`.

Quando si considera che i componenti di visualizzazione soddisfino le specifiche di un'app, è consigliabile usare i Razor componenti. Razor i componenti combinano anche il markup con il codice C# per produrre unità di interfaccia utente riutilizzabili. Razor i componenti sono progettati per la produttività degli sviluppatori quando forniscono la logica e la composizione dell'interfaccia utente lato client. Per altre informazioni, vedere <xref:blazor/components/index>. Per informazioni su come incorporare i Razor componenti in un'app MVC o Razor pagine, vedere <xref:blazor/components/prerendering-and-integration?pivots=server> .

## <a name="creating-a-view-component"></a>Creazione di un componente di visualizzazione

Questa sezione contiene i requisiti principali per creare un componente di visualizzazione. Più avanti in questo articolo, vengono esaminati nel dettaglio tutti i passaggi e viene creato un componente di visualizzazione.

### <a name="the-view-component-class"></a>Classe del componente di visualizzazione

È possibile creare una classe del componente di visualizzazione in uno dei modi seguenti:

* Derivando da *ViewComponent*
* Assegnando a una classe con l'attributo `[ViewComponent]` o derivando da una classe con l'attributo `[ViewComponent]`
* Creando una classe in cui il nome termina con il suffisso *ViewComponent*

Come i controller, i componenti di visualizzazione devono essere classi pubbliche, non devono essere classi annidate e astratte. Il nome del componente di visualizzazione corrisponde al nome della classe privato del suffisso "ViewComponent". È anche possibile specificarlo in modo esplicito usando la proprietà `ViewComponentAttribute.Name`.

Una classe del componente di visualizzazione:

* Supporta pienamente l'[inserimento delle dipendenze ](../../fundamentals/dependency-injection.md) del costruttore

* Non partecipa al ciclo di vita del controller, non è quindi possibile usare i [filtri](../controllers/filters.md) in un componente di visualizzazione

### <a name="view-component-methods"></a>Metodi del componente di visualizzazione

Un componente di visualizzazione definisce la propria logica in un metodo `InvokeAsync` che restituisce `Task<IViewComponentResult>` o in un metodo asincrono `Invoke` che restituisce `IViewComponentResult`. I parametri vengono rilevati direttamente dalla chiamata del componente di visualizzazione e non dall'associazione di modelli. Un componente di visualizzazione non gestisce mai direttamente una richiesta. In genere, inizializza un modello e lo passa a una visualizzazione chiamando il metodo `View`. Riepilogando, i metodi del componente di visualizzazione:

* Definiscono un metodo `InvokeAsync` che restituisce `Task<IViewComponentResult>` o un metodo sincrono `Invoke` che restituisce `IViewComponentResult`.
* In genere, Inizializza un modello e lo passa a una visualizzazione chiamando il `ViewComponent` `View` metodo.
* I parametri vengono rilevati dal metodo di chiamata, non da HTTP, e non vi è alcuna associazione di modelli.
* Non sono raggiungibili direttamente come un endpoint HTTP. Vengono richiamati dal codice (in genere in una vista). Un componente di visualizzazione non gestisce mai una richiesta.
* Sono sottoposti a overload sulla firma e non sui dettagli dalla richiesta HHTP corrente.

### <a name="view-search-path"></a>Percorso di ricerca della visualizzazione

Il runtime esegue la ricerca della visualizzazione nei percorsi seguenti:

* /Views/{Nome controller}/Components/{Nome componente visualizzazione}/{Nome visualizzazione}
* /Views/Shared/Components/{Nome componente visualizzazione}/{Nome visualizzazione}
* /Pages/Shared/Components/{Nome componente visualizzazione}/{Nome visualizzazione}

Il percorso di ricerca si applica ai progetti che usano i controller e le visualizzazioni e le Razor pagine.

Il nome di visualizzazione predefinito per un componente di visualizzazione è *Default*, quindi il file della visualizzazione viene solitamente denominato *Default.cshtml*. È possibile specificare un nome di visualizzazione diverso quando si crea il risultato del componente di visualizzazione o quando si chiama il metodo `View`.

Si consiglia di denominare il file della visualizzazione *Default.cshtml* e usare il percorso *Views/Shared/Components/{Nome componente visualizzazione}/{Nome visualizzazione}*. Il componente di visualizzazione `PriorityList` in questo esempio usa *Views/Shared/Components/PriorityList/Default.cshtml* per la visualizzazione del componente di visualizzazione.

### <a name="customize-the-view-search-path"></a>Personalizzare il percorso di ricerca della visualizzazione

Per personalizzare il percorso di ricerca della visualizzazione, modificare la Razor <xref:Microsoft.AspNetCore.Mvc.Razor.RazorViewEngineOptions.ViewLocationFormats> raccolta. Per cercare, ad esempio, le visualizzazioni all'interno del percorso "/Components/{View Component Name}/{View Name}", aggiungere un nuovo elemento alla raccolta:

[!code-csharp[](view-components/samples_snapshot/2.x/Startup.cs?name=snippet_ViewLocationFormats&highlight=4)]

Nel codice precedente, il segnaposto " {0} " rappresenta il percorso "Components/{View Component Name}/{View Name}".

## <a name="invoking-a-view-component"></a>Chiamata di un componente di visualizzazione

Per usare il componente di visualizzazione, chiamare il codice seguente all'interno di una visualizzazione:

```cshtml
@await Component.InvokeAsync("Name of view component", {Anonymous Type Containing Parameters})
```

I parametri saranno passati al metodo `InvokeAsync`. Il `PriorityList` componente di visualizzazione sviluppato nell'articolo viene richiamato dal file di visualizzazione *views/todo/index. cshtml* . Nell'esempio seguente il metodo `InvokeAsync` viene chiamato con due parametri:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

::: moniker range=">= aspnetcore-1.1"

## <a name="invoking-a-view-component-as-a-tag-helper"></a>Chiamata di un componente di visualizzazione come helper tag

Per ASP.NET Core 1.1 e versioni successive, è possibile richiamare un componente di visualizzazione come [helper tag](xref:mvc/views/tag-helpers/intro):

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexTagHelper.cshtml?range=37-38)]

La classe scritta usando la convenzione Pascal e i parametri del metodo per gli helper tag vengono convertiti nel formato corrispondente [kebab case](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101). Per richiamare un componente di visualizzazione, l'helper tag usa l'elemento `<vc></vc>`. Il componente di visualizzazione viene specificato nel modo seguente:

```cshtml
<vc:[view-component-name]
  parameter1="parameter1 value"
  parameter2="parameter2 value">
</vc:[view-component-name]>
```

Per usare un componente di visualizzazione come helper tag, registrare l'assembly contenente il componente di visualizzazione usando la direttiva `@addTagHelper`. Se il componente di visualizzazione si trova in un assembly denominato `MyWebApp`, aggiungere la direttiva seguente al file *_ViewImports.cshtml*:

```cshtml
@addTagHelper *, MyWebApp
```

È possibile registrare un componente di visualizzazione come helper tag per qualsiasi file che fa riferimento al componente di visualizzazione. Vedere [Gestione dell'ambito dell'helper tag](xref:mvc/views/tag-helpers/intro#managing-tag-helper-scope) per altre informazioni su come registrare gli helper tag.

Il metodo `InvokeAsync` usato in questa esercitazione:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

Nel markup dell'helper tag:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexTagHelper.cshtml?range=37-38)]

Nell'esempio precedente il componente di visualizzazione `PriorityList` diventa `priority-list`. I parametri per il componente di visualizzazione vengono passati come attributi nel formato kebab case.

::: moniker-end

### <a name="invoking-a-view-component-directly-from-a-controller"></a>Richiamo di un componente di visualizzazione direttamente da un controller

I componenti di visualizzazione sono solitamente richiamati da una visualizzazione, ma possono essere richiamati direttamente da un metodo del controller. A differenza dei controller i componenti di visualizzazione non definiscono endpoint. È tuttavia possibile implementare semplicemente un'azione del controller in modo che venga restituito il contenuto di un oggetto `ViewComponentResult`.

In questo esempio il componente di visualizzazione viene chiamato direttamente dal controller:

[!code-csharp[](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

## <a name="walkthrough-creating-a-simple-view-component"></a>Procedura dettagliata: creazione di un componente di visualizzazione semplice

[Scaricare](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/view-components/sample), compilare e testare il codice di avvio. Si tratta di un progetto semplice con un controller `ToDo` che visualizza un elenco di elementi *ToDo*.

![Elenco di elementi ToDo](view-components/_static/2dos.png)

### <a name="add-a-viewcomponent-class"></a>Aggiungere una classe ViewComponent

Creare una cartella *ViewComponents* e aggiungere la classe `PriorityListViewComponent` seguente:

[!code-csharp[](view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponent1.cs?name=snippet1)]

Note riguardanti il codice:

* Le classi del componente di visualizzazione possono essere contenute in **qualsiasi** cartella del progetto.
* Poiché il nome della classe PriorityList **ViewComponent** termina con il suffisso **ViewComponent**, il runtime usa la stringa "PriorityList" per fare riferimento al componente della classe da una visualizzazione. Questo aspetto viene poi spiegato in modo più dettagliato.
* L'attributo `[ViewComponent]` può modificare il nome usato per fare riferimento a un componente di visualizzazione. Ad esempio, sarebbe stato possibile denominare la classe `XYZ` e applicare l'attributo `ViewComponent`:

  ```csharp
  [ViewComponent(Name = "PriorityList")]
     public class XYZ : ViewComponent
     ```

* L'attributo `[ViewComponent]` precedente indica al selettore del componente di visualizzazione di usare il nome `PriorityList` per eseguire la ricerca delle visualizzazioni associate al componente e di usare la stringa "PriorityList" per fare riferimento al componente della classe da una visualizzazione. Questo aspetto viene poi spiegato in modo più dettagliato.
* Il componente usa l'[inserimento delle dipendenze](../../fundamentals/dependency-injection.md) per rendere disponibile il contesto dei dati.
* `InvokeAsync` espone un metodo che può essere chiamato da una visualizzazione e può accettare un numero arbitrario di argomenti.
* Il metodo `InvokeAsync` restituisce il set di elementi `ToDo` che soddisfano i parametri `isDone` e `maxPriority`.

### <a name="create-the-view-component-razor-view"></a>Creare la visualizzazione del componente di visualizzazione Razor

* Creare la cartella *Views/Shared/Components*. Il nome di questa cartella **deve** essere *Components*.

* Creare la cartella *Views/Shared/Components/PriorityList*. Il nome di questa cartella deve corrispondere al nome della classe del componente di visualizzazione oppure al nome della classe privato del suffisso (se è stata adottata la convenzione ed è stato usato il suffisso *ViewComponent* nel nome della classe). Se è stato usato l'attributo `ViewComponent`, il nome della classe dovrà corrispondere alla designazione dell'attributo.

* Creare una vista *Views/Shared/Components/PriorityName/default. cshtml* Razor :


  [!code-cshtml[](view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/Default1.cshtml)]

   La Razor visualizzazione accetta un elenco di `TodoItem` e le Visualizza. Se il metodo `InvokeAsync` del componente di visualizzazione non passa il nome della visualizzazione (come in questo esempio), per convenzione viene usato *Default* come nome della visualizzazione. Più avanti nell'esercitazione viene illustrato come passare il nome della visualizzazione. Per sostituire lo stile predefinito per un controller specifico, aggiungere una visualizzazione alla cartella di visualizzazione specifica del controller, ad esempio *Views/ToDo/Components/PriorityList/Default.cshtml*.

    Se il componente di visualizzazione è specifico del controller, è possibile aggiungerlo alla cartella specifica del controller (*views/todo/Components/PriorityName/default. cshtml*).

* Aggiungere un oggetto `div` contenente una chiamata al componente dell'elenco priorità alla fine del file *Views/ToDo/index.cshtml*:

    [!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFirst.cshtml?range=34-38)]

Il markup `@await Component.InvokeAsync` illustra la sintassi per chiamare i componenti di visualizzazione. Il primo argomento corrisponde al nome del componente che si vuole richiamare o chiamare. I parametri successivi vengono passati al componente. `InvokeAsync` può accettare un numero arbitrario di argomenti.

Testare l'app. La figura seguente illustra l'elenco ToDo e gli elementi con priorità:

![elenco todo ed elementi con priorità](view-components/_static/pi.png)

È anche possibile chiamare il componente di visualizzazione direttamente dal controller:

[!code-csharp[](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

![elementi con priorità dall'azione IndexVC](view-components/_static/indexvc.png)

### <a name="specifying-a-view-name"></a>Impostazione di un nome di visualizzazione

Per un componente di visualizzazione, in alcune condizioni è possibile dover specificare una visualizzazione non predefinita. Il codice seguente illustra come specificare la visualizzazione "PVC" dal metodo `InvokeAsync`. Aggiornare il metodo `InvokeAsync` nella classe `PriorityListViewComponent`.

[!code-csharp[](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponentFinal.cs?highlight=4,5,6,7,8,9&range=28-39)]

Copia il file *Views/Shared/Components/PriorityList/Default.cshtml* in una visualizzazione denominata *Views/Shared/Components/PriorityList/PVC.cshtml*. Aggiungere un'intestazione per indicare che viene usata una visualizzazione PVC.

[!code-cshtml[](../../mvc/views/view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/PVC.cshtml?highlight=3)]

Aggiornare *Views/ToDo/Index.cshtml*:

<!-- Views/ToDo/Index.cshtml is never imported, so change to test tutorial -->

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

Eseguire l'app e verificare la visualizzazione PVC.

![Componente di visualizzazione con priorità](view-components/_static/pvc.png)

Se non viene eseguito il rendering della visualizzazione PVC, verificare che si stia chiamando il componente di visualizzazione con priorità pari a 4 o superiore.

### <a name="examine-the-view-path"></a>Esaminare il percorso di visualizzazione

* Modificare il parametro relativo alla priorità impostandolo su tre o priorità inferiore perché la visualizzazione con priorità non venga restituita.
* Rinominare temporaneamente il *valore views/todo/Components/PriorityName/default. cshtml* in *1Default. cshtml*.
* Testare l'app. Verrà visualizzato il messaggio seguente:

   ```
   An unhandled exception occurred while processing the request.
   InvalidOperationException: The view 'Components/PriorityList/Default' wasn't found. The following locations were searched:
   /Views/ToDo/Components/PriorityList/Default.cshtml
   /Views/Shared/Components/PriorityList/Default.cshtml
   EnsureSuccessful
   ```

* Copiare *Views/ToDo/Components/PriorityList/1Default.cshtml* in *Views/Shared/Components/PriorityList/Default.cshtml*.
* Aggiungere un markup alla visualizzazione del componente di visualizzazione todo *condiviso* per indicare che la visualizzazione è dalla cartella *condivisa* .
* Testare la visualizzazione del componente **Shared**.

![Output di ToDo con visualizzazione del componente Shared](view-components/_static/shared.png)

### <a name="avoiding-hard-coded-strings"></a>Evitare stringhe hardcoded

Per garantire la sicurezza in fase di compilazione, è possibile sostituire il nome del componente di compilazione hardcoded con il nome della classe. Creare il componente di visualizzazione senza il suffisso "ViewComponent":

[!code-csharp[](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityList.cs?highlight=10&range=5-35)]

Aggiungere un' `using` istruzione al Razor file di visualizzazione e usare l' `nameof` operatore:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexNameof.cshtml?range=1-6,35-)]

## <a name="perform-synchronous-work"></a>Eseguire operazioni sincrone

Il framework gestisce la chiamata di un metodo `Invoke` sincrono se non è necessario eseguire operazioni asincrone. Il metodo seguente crea un componente di visualizzazione `Invoke` sincrono:

```csharp
public class PriorityList : ViewComponent
{
    public IViewComponentResult Invoke(int maxPriority, bool isDone)
    {
        var items = new List<string> { $"maxPriority: {maxPriority}", $"isDone: {isDone}" };
        return View(items);
    }
}
```

Il file del componente di visualizzazione Razor elenca le stringhe passate al `Invoke` Metodo (*views/Home/Components/PriorityName/default. cshtml*):

```cshtml
@model List<string>

<h3>Priority Items</h3>
<ul>
    @foreach (var item in Model)
    {
        <li>@item</li>
    }
</ul>
```

::: moniker range=">= aspnetcore-1.1"

Il componente di visualizzazione viene richiamato in un Razor file, ad esempio *views/Home/index. cshtml*, usando uno degli approcci seguenti:

* <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper>
* [Helper tag](xref:mvc/views/tag-helpers/intro)

Per usare l'approccio <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper>, chiamare `Component.InvokeAsync`:

::: moniker-end

::: moniker range="< aspnetcore-1.1"

Il componente di visualizzazione viene richiamato in un Razor file, ad esempio *views/Home/index. cshtml*, con <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper> .

Chiamare `Component.InvokeAsync`:

::: moniker-end

```cshtml
@await Component.InvokeAsync(nameof(PriorityList), new { maxPriority = 4, isDone = true })
```

::: moniker range=">= aspnetcore-1.1"

Per usare l'helper tag, registrare l'assembly contenente il componente di visualizzazione usando la direttiva `@addTagHelper` (il componente di visualizzazione è in un assembly denominato `MyWebApp`):

```cshtml
@addTagHelper *, MyWebApp
```

Usare l'helper tag dei componenti di visualizzazione nel Razor file di markup:

```cshtml
<vc:priority-list max-priority="999" is-done="false">
</vc:priority-list>
```

::: moniker-end

La firma del metodo di `PriorityList.Invoke` è sincrona, ma Razor trova e chiama il metodo con `Component.InvokeAsync` nel file di markup.

## <a name="all-view-component-parameters-are-required"></a>Tutti i parametri del componente di visualizzazione sono obbligatori

Ogni parametro in un componente di visualizzazione è un attributo obbligatorio. Vedere [il problema in GitHub](https://github.com/dotnet/AspNetCore/issues/5011). Se un parametro viene omesso:

* La firma del metodo `InvokeAsync` non corrisponde, quindi il metodo non verrà eseguito.
* ViewComponent non esegue il rendering dei markup.
* Non vengono generati errori.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Inserimento di dipendenze in visualizzazioni](xref:mvc/views/dependency-injection)
