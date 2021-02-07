---
title: "Parte 3: aggiungere una visualizzazione a un'app MVC ASP.NET Core"
author: rick-anderson
description: Parte 3 della serie di esercitazioni su ASP.NET Core MVC.
ms.author: riande
ms.date: 01/28/2021
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
uid: tutorials/first-mvc-app/adding-view
ms.custom: contperf-fy21q3
ms.openlocfilehash: 1542e44fcc6d0ae22fb1a759ea3a3ed1d866cbc7
ms.sourcegitcommit: 19a004ff2be73876a9ef0f1ac44d0331849ad159
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/07/2021
ms.locfileid: "99804569"
---
# <a name="part-3-add-a-view-to-an-aspnet-core-mvc-app"></a>Parte 3: aggiungere una visualizzazione a un'app MVC ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

In questa sezione viene modificata la `HelloWorldController` classe per l'utilizzo dei [Razor](xref:mvc/views/razor) file di visualizzazione. Questa operazione Incapsula in modo semplice il processo di generazione di risposte HTML a un client.

I modelli di visualizzazione vengono creati utilizzando Razor . Razormodelli di vista basati su:

* Avere un'estensione di file *. cshtml* .
* Consente di creare in modo elegante l'output HTML con C#.

Attualmente il `Index` metodo restituisce una stringa con un messaggio nella classe controller. Nella classe `HelloWorldController` sostituire il metodo `Index` con il codice seguente:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_4)]

Il codice precedente:

* Chiama il metodo del controller <xref:Microsoft.AspNetCore.Mvc.Controller.View*> .
* Usa un modello di visualizzazione per generare una risposta HTML.

Metodi controller:

* Sono denominati *metodi di azione*.  Ad esempio, il `Index` metodo di azione del codice precedente.
* Generalmente restituisce un oggetto <xref:Microsoft.AspNetCore.Mvc.IActionResult> o una classe derivata da <xref:Microsoft.AspNetCore.Mvc.ActionResult> , non un tipo come `string` .

## <a name="add-a-view"></a>Aggiungere una vista

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Fare clic con il pulsante destro del mouse sulla cartella *views* , quindi **aggiungere > nuova cartella** e denominare la cartella *HelloWorld*.

Fare clic con il pulsante destro del mouse sulla cartella *views/HelloWorld* , quindi **aggiungere > nuovo elemento**.

Nella finestra di dialogo **Aggiungi nuovo elemento-MvcMovie** :

* Nella casella di ricerca in alto a destra immettere *view* (vista)
* Selezione **Razor visualizzazione**
* Mantenere il valore della casella **Nome**, *Index.cshtml*.
* Selezionare **Aggiungi**

![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view50.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Aggiungere una `Index` vista per `HelloWorldController` :

* Aggiungere una nuova cartella denominata *Views/HelloWorld*.
* Aggiungere un nuovo file al nome di cartella *Views/HelloWorld**Index.cshtml*.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Control: fare clic sulla cartella *views* , quindi **aggiungere > nuova cartella** e denominare la cartella *HelloWorld*.

Control-fare clic sulla cartella *views/HelloWorld* , quindi **aggiungere > nuovo file**.

Nel finestra di dialogo **Nuovo file**:

* Selezionare **ASP.NET Core** nel riquadro sinistro.
* Selezionare **Razor Visualizza-vuoto** nel riquadro centrale.
* Digitare *index* nella casella **nome** .
* Selezionare **Nuovo**.

  ![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view_macVSM8.9.png)

---

Sostituire il contenuto del file di visualizzazione *views/HelloWorld/index. cshtml* Razor con il codice seguente:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/HelloWorld/Index1.cshtml?highlight=7)]

Passa a `https://localhost:{PORT}/HelloWorld` :

* Il `Index` metodo in ha `HelloWorldController` eseguito l'istruzione `return View();` , che ha specificato che il metodo deve usare un file di modello di visualizzazione per eseguire il rendering di una risposta nel browser.
* Il nome del file di modello di visualizzazione non è stato specificato, quindi MVC ha utilizzato il file di visualizzazione predefinito. Quando il nome del file di visualizzazione non è specificato, viene restituita la visualizzazione predefinita. In questo esempio, la visualizzazione predefinita ha lo stesso nome del metodo di azione `Index` . Viene utilizzato il modello di visualizzazione */views/HelloWorld/index.cshtml* .
* La figura seguente mostra la stringa "Hello from the view template!" hardcoded nella visualizzazione:

  ![Finestra del browser](~/tutorials/first-mvc-app/adding-view/_static/hell_template.png)

## <a name="change-views-and-layout-pages"></a>Modificare le viste e le pagine di layout

Selezionare il menu collegamenti **MvcMovie**, **Home** e **privacy**. Ogni pagina mostra lo stesso layout di menu. Il layout di menu viene implementato nel file *Views/Shared/_Layout.cshtml*.

Aprire il file *Views/Shared/_Layout.cshtml*.

I modelli di [layout](xref:mvc/views/layout) consentono:

* Specifica del layout del contenitore HTML di un sito in un'unica posizione.
* Applicazione del layout del contenitore HTML in più pagine del sito.

Trovare la riga `@RenderBody()`. `RenderBody` è un segnaposto dove tutte le pagine specifiche della vista vengono presentate, *incapsulate* nella pagina di layout. Ad esempio, se si seleziona il collegamento **Privacy**, il rendering della vista **Views/Home/Privacy.cshtml** viene eseguito all'interno del metodo `RenderBody`.

## <a name="change-the-title-footer-and-menu-link-in-the-layout-file"></a>Modificare il titolo, il piè di pagina e il collegamento di menu nel file di layout

Sostituire il contenuto del file *Views/Shared/_Layout. cshtml* con il markup seguente. Le modifiche sono evidenziate:
::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Views/Shared/_Layout.cshtml?highlight=6,14,40)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie5/Views/Shared/_Layout.cshtml?highlight=6,14,40)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

Il markup precedente ha apportato le modifiche seguenti:

* Tre occorrenze di `MvcMovie` a `Movie App` .
* L'elemento di ancoraggio `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` è stato sostituito con `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>`.

Nel markup precedente, l' `asp-area=""` [attributo dell'helper tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) e il valore dell'attributo sono stati omessi perché l'app non usa le [aree](xref:mvc/controllers/areas).

**Nota**: il `Movies` controller non è stato implementato. A questo punto, il `Movie App` collegamento non funziona.

Salvare le modifiche e selezionare il collegamento per la **privacy** . Si noti come il titolo sulla scheda del browser visualizzi **Privacy Policy - Movie App** anziché **Privacy Policy - Mvc Movie**:

![Scheda Privacy](~/tutorials/first-mvc-app/adding-view/_static/privacy50.png)

Selezionare il collegamento **Home**.

Si noti che il titolo e il testo di ancoraggio visualizzano l' **app Movie**. Le modifiche sono state apportate una sola volta nel modello di layout e tutte le pagine del sito riflettono il nuovo testo del collegamento e il nuovo titolo.

Esaminare il file *Views/_ViewStart.cshtml*:

```cshtml
@{
    Layout = "_Layout";
}
```

Il file *Views/_ViewStart.cshtml* attiva il file *Views/Shared/_Layout.cshtml* per ogni vista. È possibile usare la proprietà `Layout` per impostare una vista di layout differente oppure impostarla su `null` e quindi non verrà usato alcun file di layout.

Aprire il file di visualizzazione *views/HelloWorld/index. cshtml* .

Modificare il titolo e l' `<h2>` elemento come evidenziato nell'esempio seguente:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Index2.cshtml?highlight=2,5)]

Il titolo e l' `<h2>` elemento sono leggermente diversi, quindi è chiaro quale parte del codice cambia la visualizzazione.

`ViewData["Title"] = "Movie List";` nel codice precedente imposta la proprietà `Title` del dizionario `ViewData` su "Movie List". La proprietà `Title` viene usata nell'elemento HTML `<title>` nella pagina di layout:

```cshtml
<title>@ViewData["Title"] - Movie App</title>
```

Salvare la modifica e passare a `https://localhost:{PORT}/HelloWorld`.

Si noti che sono stati modificati i seguenti elementi:

* Titolo del browser.
* Intestazione primaria.
* Intestazioni secondarie.

Se non sono presenti modifiche nel browser, è possibile che venga memorizzato nella cache il contenuto visualizzato. Premere CTRL + F5 nel browser per forzare il caricamento della risposta dal server. Il titolo del browser viene creato con il valore `ViewData["Title"]` impostato nel modello di vista *Index.cshtml* e la stringa "- Movie App" aggiuntiva viene aggiunta nel file di layout.

Il contenuto del modello di vista *Index.cshtml* viene unito al modello di vista *Views/Shared/_Layout.cshtml*. Viene inviata una singola risposta HTML al browser. I modelli di layout rendono semplice apportare modifiche che si applicano a tutte le pagine di un'app. Per altre informazioni, vedere [layout](xref:mvc/views/layout).

![Vista dell'elenco di film](~/tutorials/first-mvc-app/adding-view/_static/hell50.png)

Il piccolo bit di "dati", il "Hello from the view template!" Message, è tuttavia hardcoded. L'applicazione MVC contiene una "V" (visualizzazione), una "C" (controller), ma non ancora "M" (modello).

## <a name="passing-data-from-the-controller-to-the-view"></a>Passaggio di dati dal controller alla vista

Le azioni del controller vengono richiamate in risposta a una richiesta URL in ingresso. Una classe controller è il punto in cui viene scritto il codice che gestisce le richieste in ingresso del browser. Il controller recupera i dati da un'origine dati e determina il tipo di risposta da inviare al browser. I modelli di vista possono essere usati da un controller per generare e formattare una risposta HTML al browser.

Il compito dei controller è fornire i dati necessari affinché un modello di vista possa eseguire il rendering di una risposta.

I modelli di visualizzazione **non** devono:

* Logica di business
* Interagire direttamente con un database.

Un modello di visualizzazione dovrebbe funzionare solo con i dati forniti dal controller. Il mantenimento di questa "separazione dei problemi" consente di mantenere il codice:

* Pulito.
* Verificabile.
* Gestibile.

Attualmente il metodo `Welcome` nella classe `HelloWorldController` accetta un parametro `name` e `ID` e quindi genera i valori direttamente al browser.

Invece di ottenere il rendering di questa risposta come stringa, cambiare il controller in modo che usi un modello di vista. Il modello di visualizzazione genera una risposta dinamica, il che significa che i dati appropriati devono essere passati dal controller alla visualizzazione per generare la risposta. A tale scopo, fare in modo che il controller inserisca i dati dinamici (parametri) necessari per il modello di visualizzazione in un `ViewData` dizionario. Il modello di visualizzazione può quindi accedere ai dati dinamici.

In *HelloWorldController.cs*, modificare il metodo `Welcome` in modo da aggiungere un valore `Message` e `NumTimes` al dizionario `ViewData`.

Il `ViewData` dizionario è un oggetto dinamico, ovvero è possibile usare qualsiasi tipo. L' `ViewData` oggetto non dispone di proprietà definite fino a quando non viene aggiunto alcun elemento. Il [sistema di associazione di modelli MVC](xref:mvc/models/model-binding) esegue automaticamente il mapping dei parametri denominati `name` e `numTimes` dalla stringa di query ai parametri nel metodo. Il completamento `HelloWorldController` :

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_5&highlight=13-19)]

L'oggetto di dizionario `ViewData` contiene i dati che verranno passati alla vista.

Creare un modello di vista Welcome denominato *Views/HelloWorld/Welcome.cshtml*.

Si creerà un ciclo nel modello di vista *Welcome.cshtml* che visualizza la stringa "Hello" `NumTimes`. Sostituire il contenuto di *Views/HelloWorld/Welcome.cshtml* con quanto segue:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Welcome.cshtml)]

Salvare le modifiche e passare all'URL seguente:

`https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

I dati vengono ricavati dall'URL e passati al controller usando lo strumento di [associazione di modelli MVC](xref:mvc/models/model-binding). Il controller crea un pacchetto di dati in un dizionario `ViewData` e passa tale oggetto alla vista. La vista esegue quindi il rendering dei dati in formato HTML al browser.

![Vista Privacy con un'etichetta di benvenuto e la frase Hello Rick riportata quattro volte](~/tutorials/first-mvc-app/adding-view/_static/rick2_50.png)

Nell'esempio precedente, il `ViewData` dizionario è stato usato per passare i dati dal controller a una visualizzazione. Più avanti nell'esercitazione si userà un modello di vista per passare i dati da un controller a una vista. L'approccio al modello di visualizzazione per il passaggio dei dati è preferibile rispetto all' `ViewData` approccio del dizionario.

Nella prossima esercitazione, viene creato un database di film.

> [!div class="step-by-step"]
> [Precedente: aggiungere un controller](adding-controller.md) 
>  Passaggio [successivo: aggiungere un modello](adding-model.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

In questa sezione, la `HelloWorldController` classe viene modificata per usare i [Razor](xref:mvc/views/razor) file di visualizzazione per incapsulare in modo semplice il processo di generazione di risposte HTML a un client.

Si crea un file di modello di visualizzazione utilizzando Razor . Razori modelli di vista basati su hanno un'estensione di file *. cshtml* . Consentono di creare l'output HTML in modo accurato con C#.

Attualmente il metodo `Index` restituisce una stringa con un messaggio hardcoded nella classe controller. Nella classe `HelloWorldController` sostituire il metodo `Index` con il codice seguente:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_4)]

Il codice precedente chiama il metodo <xref:Microsoft.AspNetCore.Mvc.Controller.View*> del controller. Usa un modello di visualizzazione per generare una risposta HTML. I metodi del controller, noti anche come *metodi di azione*, ad esempio il metodo `Index` descritto in precedenza, restituiscono in genere un <xref:Microsoft.AspNetCore.Mvc.IActionResult> (o una classe derivata da <xref:Microsoft.AspNetCore.Mvc.ActionResult>) e non un tipo come `string`.

## <a name="add-a-view"></a>Aggiungere una vista

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Fare clic con il pulsante destro del mouse sulla cartella *views* , quindi **aggiungere > nuova cartella** e denominare la cartella *HelloWorld*.

* Fare clic con il pulsante destro del mouse sulla cartella *views/HelloWorld* , quindi **aggiungere > nuovo elemento**.

* Nella finestra di dialogo **Aggiungi nuovo elemento - MvcMovie**

  * Nella casella di ricerca in alto a destra immettere *view* (vista)

  * Selezione **Razor visualizzazione**

  * Mantenere il valore della casella **Nome**, *Index.cshtml*.

  * Selezionare **Aggiungi**

![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Aggiungere una vista `Index` per `HelloWorldController`.

* Aggiungere una nuova cartella denominata *Views/HelloWorld*.
* Aggiungere un nuovo file al nome di cartella *Views/HelloWorld**Index.cshtml*.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Fare clic con il pulsante destro del mouse sulla cartella *Views*, quindi su **Aggiungi > Nuova cartella** e denominare la cartella *HelloWorld*.
* Fare clic con il pulsante destro del mouse sulla cartella *Views/HelloWorld*, quindi su **Aggiungi > Nuovo file**.
* Nel finestra di dialogo **Nuovo file**:

  * Selezionare **Web** nel riquadro a sinistra.
  * Selezionare **File HTML vuoto** nel riquadro centrale.
  * Digitare *Index.cshtml* nella casella **Nome**.
  * Selezionare **Nuovo**.

![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view_mac.png)

---

Sostituire il contenuto del file di visualizzazione *views/HelloWorld/index. cshtml* Razor con il codice seguente:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/HelloWorld/Index1.cshtml?highlight=7)]

Accedere a `https://localhost:{PORT}/HelloWorld`. Il metodo `Index` in `HelloWorldController` non ha eseguito molte operazioni; ha eseguito l'istruzione `return View();` che ha specificato che il metodo deve usare un file di modello della vista per eseguire il rendering di una risposta al browser. Poiché non è stato specificato un nome di file di modello di visualizzazione, per impostazione predefinita MVC usa il file di visualizzazione predefinito. Il file di visualizzazione predefinito ha lo stesso nome del metodo (`Index`), quindi viene usato */Views/HelloWorld/Index.cshtml*. L'immagine seguente mostra la stringa "Hello from our View Template!" hardcoded nella vista.

![Finestra del browser](~/tutorials/first-mvc-app/adding-view/_static/hell_template.png)

## <a name="change-views-and-layout-pages"></a>Modificare le viste e le pagine di layout

Selezionare i collegamenti di menu: **MvcMovie**, **Home** e **Privacy**. Ogni pagina mostra lo stesso layout di menu. Il layout di menu viene implementato nel file *Views/Shared/_Layout.cshtml*. Aprire il file *Views/Shared/_Layout.cshtml*.

I modelli di [layout](xref:mvc/views/layout) consentono di specificare il layout del contenitore HTML del sito in un'unica posizione e quindi di applicarlo in più pagine del sito. Trovare la riga `@RenderBody()`. `RenderBody` è un segnaposto dove tutte le pagine specifiche della vista vengono presentate, *incapsulate* nella pagina di layout. Ad esempio, se si seleziona il collegamento **Privacy**, il rendering della vista **Views/Home/Privacy.cshtml** viene eseguito all'interno del metodo `RenderBody`.

## <a name="change-the-title-footer-and-menu-link-in-the-layout-file"></a>Modificare il titolo, il piè di pagina e il collegamento di menu nel file di layout

* Negli elementi del titolo e del piè di pagina modificare `MvcMovie` in `Movie App`.
* Modificare l'elemento ancora `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` in `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>`.

Il markup seguente visualizza le modifiche evidenziate:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Shared/_Layout.cshtml?highlight=6,24,51)]

Nel markup precedente, l'attributo `asp-area` [Helper Tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) è stato omesso perché questa app non usa le [Aree](xref:mvc/controllers/areas).

<!-- Routing has changed in 2.2, it's going to the last route.
>[!WARNING]
> We haven't implemented the `Movies` controller yet, so if you click the `Movie App` link, you get a 404 (Not found) error.
-->

**Nota**: il `Movies` controller non è stato implementato. A questo punto, il collegamento `Movie App` non è funzionale.

Salvare le modifiche e selezionare il collegamento **Privacy**. Si noti come il titolo sulla scheda del browser visualizzi **Privacy Policy - Movie App** anziché **Privacy Policy - Mvc Movie**:

![Scheda Privacy](~/tutorials/first-mvc-app/adding-view/_static/privacy50.png)

Toccare il collegamento **Home** e notare che anche il titolo e il testo di ancoraggio visualizzano **Movie App**. La modifica è stata apportata una volta nel modello di layout e tutte le pagine nel sito riflettono il nuovo testo del collegamento e il nuovo titolo.

Esaminare il file *Views/_ViewStart.cshtml*:

```cshtml
@{
    Layout = "_Layout";
}
```

Il file *Views/_ViewStart.cshtml* attiva il file *Views/Shared/_Layout.cshtml* per ogni vista. È possibile usare la proprietà `Layout` per impostare una vista di layout differente oppure impostarla su `null` e quindi non verrà usato alcun file di layout.

Modificare il titolo e l'elemento `<h2>` del file *Views/HelloWorld/Index.cshtml* di visualizzazione:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Index2.cshtml?highlight=2,5)]

Il titolo e l'elemento `<h2>`sono leggermente diversi in modo da poter esaminare la parte specifica di codice che modifica la visualizzazione.

`ViewData["Title"] = "Movie List";` nel codice precedente imposta la proprietà `Title` del dizionario `ViewData` su "Movie List". La proprietà `Title` viene usata nell'elemento HTML `<title>` nella pagina di layout:

```cshtml
<title>@ViewData["Title"] - Movie App</title>
```

Salvare la modifica e passare a `https://localhost:{PORT}/HelloWorld`. Si noti che il titolo del browser, l'intestazione primaria e le intestazioni secondarie sono cambiati. Se le modifiche non sono visibili nel browser, è possibile che si stia visualizzando il contenuto memorizzato nella cache. Premere CTRL + F5 nel browser per forzare il caricamento della risposta dal server. Il titolo del browser viene creato con l' `ViewData["Title"]` impostazione nel modello di vista *index. cshtml* e l'aggiunta di "-Movie app" nel file di layout.

Si noti anche come il contenuto del modello di vista *Index.cshtml* sia stato unito con il modello di vista *Views/Shared/_Layout.cshtml* e sia stata inviata una singola risposta HTML al browser. I modelli di layout rendono molto semplice apportare modifiche che si applicano a tutte le pagine dell'applicazione. Per altre informazioni, vedere [Layout](xref:mvc/views/layout).

![Vista dell'elenco di film](~/tutorials/first-mvc-app/adding-view/_static/hell3.png)

Il breve testo di "dati", in questo caso il messaggio "Hello from our View Template!", è tuttavia hardcoded. L'applicazione MVC ha un elemento "V" (vista) ed è stato ottenuto un elemento "C" (controller), ma non ancora un elemento "M" (modello).

## <a name="passing-data-from-the-controller-to-the-view"></a>Passaggio di dati dal controller alla vista

Le azioni del controller vengono richiamate in risposta a una richiesta URL in ingresso. Una classe controller è il punto in cui viene scritto il codice che gestisce le richieste in ingresso del browser. Il controller recupera i dati da un'origine dati e determina il tipo di risposta da inviare al browser. I modelli di vista possono essere usati da un controller per generare e formattare una risposta HTML al browser.

Il compito dei controller è fornire i dati necessari affinché un modello di vista possa eseguire il rendering di una risposta. Procedura consigliata: i modelli di vista **non** devono eseguire la logica di business o interagire direttamente con un database. Un modello di vista deve invece usare solo i dati forniti dal controller. Questa "separazione delle problematiche" consente di mantenere il codice pulito e semplifica la gestione e i test.

Attualmente il metodo `Welcome` nella classe `HelloWorldController` accetta un parametro `name` e `ID` e quindi genera i valori direttamente al browser. Invece di ottenere il rendering di questa risposta come stringa, cambiare il controller in modo che usi un modello di vista. Il modello di vista genera una risposta dinamica, il che significa che è necessario passare i bit di dati appropriati dal controller alla vista per generare la risposta. A tale scopo, fare in modo che il controller inserisca i dati dinamici (parametri) di cui il modello di vista necessita in un dizionario `ViewData` a cui il modello di vista può accedere.

In *HelloWorldController.cs*, modificare il metodo `Welcome` in modo da aggiungere un valore `Message` e `NumTimes` al dizionario `ViewData`. Il dizionario `ViewData` è un oggetto dinamico, ovvero è possibile usare qualunque tipo; l'oggetto `ViewData` non dispone di proprietà definite fino a quando non si inserisce un elemento al suo interno. Il sistema di [associazione di modelli MVC](xref:mvc/models/model-binding) esegue il mapping dei parametri denominati (`name` e `numTimes`) dalla stringa di query nella barra degli indirizzi ai parametri nel metodo. Il file *HelloWorldController.cs* completo avrà un aspetto simile al seguente:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_5)]

L'oggetto di dizionario `ViewData` contiene i dati che verranno passati alla vista.

Creare un modello di vista Welcome denominato *Views/HelloWorld/Welcome.cshtml*.

Si creerà un ciclo nel modello di vista *Welcome.cshtml* che visualizza la stringa "Hello" `NumTimes`. Sostituire il contenuto di *Views/HelloWorld/Welcome.cshtml* con quanto segue:

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Welcome.cshtml)]

Salvare le modifiche e passare all'URL seguente:

`https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

I dati vengono prelevati dall'URL e passati al controller usando lo [strumento di associazione di modelli MVC](xref:mvc/models/model-binding). Il controller crea un pacchetto di dati in un dizionario `ViewData` e passa tale oggetto alla vista. La vista esegue quindi il rendering dei dati in formato HTML al browser.

![Vista Privacy con un'etichetta di benvenuto e la frase Hello Rick riportata quattro volte](~/tutorials/first-mvc-app/adding-view/_static/rick2.png)

Nell'esempio precedente è stato usato il dizionario `ViewData` per passare i dati dal controller a una vista. Più avanti nell'esercitazione si userà un modello di vista per passare i dati da un controller a una vista. In genere l'approccio basato sul modello di vista per passare i dati è preferito rispetto all'approccio basato sul dizionario `ViewData`. Per altre informazioni, vedere [quando usare ViewBag, ViewData o TempData](https://www.rachelappel.com/when-to-use-viewbag-viewdata-or-tempdata-in-asp-net-mvc-3-applications/) .

Nella prossima esercitazione, viene creato un database di film.

> [!div class="step-by-step"]
> [Precedente](adding-controller.md) 
>  [Avanti](adding-model.md)

::: moniker-end
