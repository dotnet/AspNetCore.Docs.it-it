---
title: Introduzione alle Razor pagine ASP.NET Core
author: Rick-Anderson
description: Spiega in che modo Razor le pagine in ASP.NET Core rendono più semplici e più produttivi gli scenari incentrati sulla pagina del codice rispetto all'uso di MVC
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.date: 02/12/2020
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
uid: razor-pages/index
ms.openlocfilehash: 78b192cb2240046d16b1b766954ed4ca5229d888
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586709"
---
# <a name="introduction-to-razor-pages-in-aspnet-core"></a>Introduzione alle Razor pagine ASP.NET Core


Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Ryan Nowak](https://github.com/rynowak)

Razor Le pagine possono rendere più semplici e più produttivi gli scenari incentrati sulla pagina di codifica rispetto all'utilizzo di controller e visualizzazioni.

Se si sta cercando un'esercitazione in cui si usa l'approccio Model-View-Controller, vedere [Introduzione ad ASP.NET Core MVC](xref:tutorials/first-mvc-app/start-mvc).

Questo documento fornisce un'introduzione alle Razor pagine. Non è un'esercitazione dettagliata. Se alcune delle sezioni sono troppo avanzate, vedere [Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start). Per una panoramica di ASP.NET Core, vedere [Introduzione a ASP.NET Core](xref:index).

## <a name="prerequisites"></a>Prerequisiti

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-5.0.md)]

---

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

<a name="rpvs17"></a>

## <a name="create-a-razor-pages-project"></a>Creazione di un Razor progetto di pagine

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Per istruzioni dettagliate su come creare un progetto di pagine, vedere [Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) Razor .

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Eseguire `dotnet new webapp` dalla riga di comando.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Per istruzioni dettagliate su come creare un progetto di pagine, vedere [Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) Razor .

---

## <a name="razor-pages"></a>Razor Pagine

Razor Le pagine sono abilitate in *Startup.cs*:

[!code-csharp[](index/3.0sample/RazorPagesIntro/Startup.cs?name=snippet_Startup&highlight=12,36)]

Si consideri una pagina di base: <a name="OnGet"></a>

[!code-cshtml[](index/3.0sample/RazorPagesIntro/Pages/Index.cshtml?highlight=1)]

Il codice precedente ha un aspetto molto simile a quello di un [ Razor file di visualizzazione](xref:tutorials/first-mvc-app/adding-view) usato in un'app ASP.NET Core con i controller e le visualizzazioni. Ciò che lo rende diverso è la [`@page`](xref:mvc/views/razor#page) direttiva. `@page` trasforma il file in un'azione MVC, ovvero gestisce le richieste direttamente, senza passare attraverso un controller. `@page` deve essere la prima Razor direttiva in una pagina. `@page` influiscono sul comportamento di altri [Razor](xref:mvc/views/razor) costrutti. Razor I nomi file delle pagine hanno un suffisso *. cshtml* .

Nei due file seguenti viene visualizzata una pagina simile che usa una classe `PageModel`. Il file *Pages/Index2.cshtml*:

[!code-cshtml[](index/3.0sample/RazorPagesIntro/Pages/Index2.cshtml)]

Il modello di pagina *Pages/Index2.cshtml.cs*:

[!code-csharp[](index/3.0sample/RazorPagesIntro/Pages/Index2.cshtml.cs)]

Per convenzione, il `PageModel` file di classe ha lo stesso nome del Razor file di paging con l'aggiunta di *. cs* . La pagina precedente, ad esempio, Razor è *pages/index2. cshtml*. Il file che contiene la classe `PageModel` è denominato *Pages/Index2.cshtml.cs*.

Le associazioni dei percorsi URL alle pagine sono determinate dalla posizione della pagina nel file system. La tabella seguente mostra un Razor percorso di pagina e l'URL corrispondente:

| Percorso e nome file               | URL corrispondente |
| ----------------- | ------------ |
| */Pages/Index.cshtml* | `/` o `/Index` |
| */Pages/Contact.cshtml* | `/Contact` |
| */Pages/Store/Contact.cshtml* | `/Store/Contact` |
| */Pages/Store/Index.cshtml* | `/Store` o `/Store/Index` |

Note:

* Per Razor impostazione predefinita, il runtime cerca i file di pagine nella cartella *pages* .
* `Index` è la pagina predefinita quando un URL non include una pagina.

## <a name="write-a-basic-form"></a>Scrivere un form di base

Razor Pagine è progettato per semplificare l'implementazione dei modelli comuni usati con i Web browser quando si compila un'app. L' [associazione di modelli](xref:mvc/models/model-binding), gli [Helper Tag](xref:mvc/views/tag-helpers/intro)e gli helper HTML *funzionano solo* con le proprietà definite in una Razor classe di pagine. Si consideri una pagina che implementa un form "contact us" di base per il modello `Contact`:

Per gli esempi riportati in questo documento, `DbContext` viene inizializzato nel file [Startup.cs](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/razor-pages/index/3.0sample/RazorPagesContacts/Startup.cs#L23-L24).

Il database in memoria richiede il `Microsoft.EntityFrameworkCore.InMemory` pacchetto NuGet.

[!code-csharp[](index/3.0sample/RazorPagesContacts/Startup.cs?name=snippet)]

Il modello di dati:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Models/Customer.cs)]

Il contesto del database:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Data/CustomerDbContext.cs)]

Il file di visualizzazione *Pages/Create.cshtml*:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml)]

Il modello di pagina *Pages/Create.cshtml.cs*:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_ALL)]

Per convenzione, la classe `PageModel` è denominata `<PageName>Model` e si trova nello stesso spazio dei nomi della pagina.

La classe `PageModel` consente la separazione della logica di una pagina dalla relativa presentazione. Definisce i gestori di pagina per le richieste inviate alla pagina e i dati usati per il rendering della pagina. Questa separazione consente:

* Gestione delle dipendenze di pagina tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).
* [Unit test](xref:test/razor-pages-tests)

La pagina contiene un oggetto , ovvero un `OnPostAsync` *metodo gestore* che viene eseguito per le richieste `POST`, quando un utente invia il form. È possibile aggiungere i metodi del gestore per qualsiasi verbo HTTP. I gestori più comuni sono i seguenti:

* `OnGet` per inizializzare lo stato necessario per la pagina. Nel codice precedente, il `OnGet` Metodo Visualizza la pagina *CreateModel. cshtml* Razor .
* `OnPost` per gestire gli invii di form.

Il suffisso `Async` nel nome è facoltativo, ma viene spesso usato per convenzione per le funzioni asincrone. Il codice precedente è tipico per le Razor pagine.

Se si ha familiarità con le app ASP.NET che usano i controller e le visualizzazioni:

* Il `OnPostAsync` codice nell'esempio precedente ha un aspetto simile al codice del controller tipico.
* La maggior parte delle primitive MVC come l' [associazione di modelli](xref:mvc/models/model-binding), la [convalida](xref:mvc/models/validation)e i risultati dell'azione funzionano allo stesso modo con i controller e le Razor pagine. 

Il metodo `OnPostAsync` precedente:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_OnPostAsync)]

Il flusso di base di `OnPostAsync`:

Verificare se sono presenti errori di convalida.

* Se non sono presenti errori, salvare i dati e reindirizzare.
* Se sono presenti errori, visualizzare di nuovo la pagina con i messaggi di convalida. In molti casi, gli errori di convalida vengono rilevati nel client e non vengono mai inviati al server.

Il file di visualizzazione *Pages/Create.cshtml*:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml)]

Il codice HTML sottoposto a rendering da *pages/create. cshtml*:

[!code-html[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create4.html)]

Nel codice precedente, inserendo il modulo:

* Con dati validi:

  * Il `OnPostAsync` metodo del gestore chiama il <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.RedirectToPage*> metodo helper. `RedirectToPage` restituisce un'istanza di <xref:Microsoft.AspNetCore.Mvc.RedirectToPageResult>. `RedirectToPage`:

    * Risultato dell'azione.
    * È simile a `RedirectToAction` o `RedirectToRoute` (usato nei controller e nelle viste).
    * È personalizzato per le pagine. Nell'esempio precedente viene reindirizzato alla pagina di indice radice (`/Index`). `RedirectToPage` è descritto in dettaglio nella sezione [generazione URL per pagine](#url_gen) .

* Con gli errori di convalida passati al server:

  * Il `OnPostAsync` metodo del gestore chiama il <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageBase.Page*> metodo helper. `Page` restituisce un'istanza di <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageResult>. La restituzione di `Page` è simile al modo in cui le azioni nel controller restituiscono `View`. `PageResult` è il tipo restituito predefinito per un metodo del gestore. Un metodo gestore che restituisce `void` esegue il rendering della pagina.
  * Nell'esempio precedente, se si pubblica il form senza alcun valore, in [ModelState. IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid) viene restituito false. In questo esempio non vengono visualizzati errori di convalida nel client. Il controllo degli errori di convalida viene trattato più avanti in questo documento.

  [!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=3-6)]

* Con errori di convalida rilevati dalla convalida lato client:

  * I dati **non** vengono inviati al server.
  * La convalida lato client è illustrata più avanti in questo documento.

La `Customer` Proprietà usa l' [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attributo per acconsentire esplicitamente all'associazione di modelli:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_PageModel&highlight=15-16)]

`[BindProperty]`**non** deve essere utilizzato nei modelli contenenti proprietà che non devono essere modificate dal client. Per ulteriori informazioni, vedere [overposting](xref:data/ef-rp/crud#overposting).

Razor Le pagine, per impostazione predefinita, associano proprietà solo con non `GET` verbi. L'associazione alle proprietà Elimina la necessità di scrivere codice per convertire i dati HTTP nel tipo di modello. Riduce il codice usando la stessa proprietà per il eseguire il rendering dei campi del form (`<input asp-for="Customer.Name">`) e accettare l'input.

[!INCLUDE[](~/includes/bind-get.md)]

Esaminando il file di visualizzazione *pages/create. cshtml* :

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml?highlight=3,9)]

* Nel codice precedente, l' [Helper Tag](xref:mvc/views/working-with-forms#the-input-tag-helper) `<input asp-for="Customer.Name" />` di input associa l' `<input>` elemento HTML all'espressione del `Customer.Name` modello.
* [`@addTagHelper`](xref:mvc/views/tag-helpers/intro#addtaghelper-makes-tag-helpers-available) rende disponibili gli helper tag.

### <a name="the-home-page"></a>La home page

*Index. cshtml* è il Home page:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml)]

La classe `PageModel` associata (*Index.cshtml.cs*):

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml.cs?name=snippet)]

Il file *index. cshtml* contiene il markup seguente:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml?range=21)]

L' `<a /a>` [Helper tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) ha usato l' `asp-route-{value}` attributo per generare un collegamento alla pagina di modifica. Il collegamento contiene i dati della route con l'ID contatto. Ad esempio: `https://localhost:5001/Edit/1`. Gli [Helper Tag](xref:mvc/views/tag-helpers/intro) consentono al codice lato server di partecipare alla creazione e al rendering di elementi HTML nei Razor file.

Il file *index. cshtml* contiene il markup per la creazione di un pulsante Delete per ogni contatto del cliente:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml?range=22-23)]

HTML sottoposto a rendering:

```html
<button type="submit" formaction="/Customers?id=1&amp;handler=delete">delete</button>
```

Quando il pulsante Elimina viene sottoposto a rendering in HTML, il relativo [formaction](https://developer.mozilla.org/docs/Web/HTML/Element/button#attr-formaction) include i parametri per:

* ID contatto del cliente, specificato dall' `asp-route-id` attributo.
* Oggetto `handler` , specificato dall' `asp-page-handler` attributo.

Quando il pulsante è selezionato, viene inviata una richiesta `POST` di modulo al server. Per convenzione, il nome del metodo del gestore viene selezionato in base al valore del parametro `handler` in base allo schema `OnPost[handler]Async`.

Poiché in questo esempio l'`handler` è `delete`, il metodo `OnPostDeleteAsync` viene usato per elaborare la richiesta `POST`. Se `asp-page-handler` viene impostato su un valore diverso, ad esempio `remove`, viene selezionato un metodo gestore con il nome `OnPostRemoveAsync`.

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml.cs?name=snippet2)]

Il metodo `OnPostDeleteAsync`:

* Ottiene `id` dalla stringa di query.
* Interroga il database in merito al contatto del cliente con `FindAsync`.
* Se il contatto del cliente viene trovato, viene rimosso e il database viene aggiornato.
* Chiama <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.RedirectToPage*> per reindirizzare alla pagina di indice radice (`/Index`).

### <a name="the-editcshtml-file"></a>File Edit. cshtml

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Edit.cshtml?highlight=1)]

La prima riga contiene la direttiva `@page "{id:int}"`. Il vincolo di routing `"{id:int}"` indica alla pagina di accettare le richieste inviate alla pagina che contengono i dati della route `int`. Se una richiesta inviata alla pagina non contiene dati della route che possono essere convertiti in un oggetto `int`, il runtime restituisce un errore HTTP 404 (non trovato). Per rendere l'ID facoltativo, aggiungere `?` al vincolo di route:

 ```cshtml
@page "{id:int?}"
```

Il file *Edit.cshtml.cs* :

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Edit.cshtml.cs?name=snippet)]

## <a name="validation"></a>Convalida

Regole di convalida:

* Sono specificati in modo dichiarativo nella classe del modello.
* Vengono applicati ovunque nell'app.

Lo <xref:System.ComponentModel.DataAnnotations> spazio dei nomi fornisce un set di attributi di convalida predefiniti che vengono applicati in modo dichiarativo a una classe o a una proprietà. DataAnnotations contiene anche attributi di formattazione come [`[DataType]`](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) questa guida alla formattazione e non forniscono alcuna convalida.

Si consideri il `Customer` modello:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Models/Customer.cs)]

Utilizzando il seguente file di visualizzazione *create. cshtml* :

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create3.cshtml?highlight=3,8-9,15-99)]

Il codice precedente:

* Include gli script di convalida jQuery e jQuery.
* USA gli `<div />` `<span />` [Helper Tag](xref:mvc/views/tag-helpers/intro) e per abilitare:

  * Convalida lato client.
  * Rendering degli errori di convalida.

* Viene generato il codice HTML seguente:

  [!code-html[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create5.html)]

Se si pubblica il modulo di creazione senza un valore di nome, viene visualizzato il messaggio di errore "il campo nome è obbligatorio". nel form. Se JavaScript è abilitato nel client, il browser Visualizza l'errore senza inviare al server.

L' `[StringLength(10)]` attributo genera `data-val-length-max="10"` sull'HTML sottoposto a rendering. `data-val-length-max` impedisce ai browser di immettere più della lunghezza massima specificata. Se viene usato uno strumento come [Fiddler](https://www.telerik.com/fiddler) per modificare e riprodurre il post:

* Con il nome più lungo di 10.
* Il messaggio di errore "il nome del campo deve essere una stringa con una lunghezza massima di 10". un errore imprevisto".

Si consideri il `Movie` modello seguente:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDA.cs?name=snippet1)]

Gli attributi di convalida specificano il comportamento da applicare alle proprietà del modello a cui sono applicati:

* Gli `Required` `MinimumLength` attributi e indicano che una proprietà deve avere un valore, ma nulla impedisce a un utente di immettere spazi vuoti per soddisfare la convalida.
* L'attributo `RegularExpression` viene usato per limitare i caratteri che possono essere inseriti. Nel codice precedente "Genre":

  * Deve includere solo lettere.
  * La prima lettera deve essere maiuscola. Gli spazi, i numeri e i caratteri speciali non sono consentiti.

* `RegularExpression` "Rating":

  * Richiede che il primo carattere sia una lettera maiuscola.
  * Consente i caratteri speciali e i numeri negli spazi successivi. "PG-13" è valido per una classificazione, ma non per "Genre".

* L'attributo `Range` vincola un valore all'interno di un intervallo specificato.
* L' `StringLength` attributo imposta la lunghezza massima di una proprietà di stringa e, facoltativamente, la lunghezza minima.
* I tipi di valore, ad esempio `decimal`, `int`, `float` e `DateTime`, sono intrinsecamente necessari e non richiedono l'attributo `[Required]`.

Nella pagina Crea per il `Movie` modello vengono visualizzati gli errori con valori non validi:

![Il modulo di vista del film con più errori di convalida del lato client jQuery](~/tutorials/razor-pages/validation/_static/val.png)

Per altre informazioni, vedere:

* [Aggiungere la convalida all'app Movie](xref:tutorials/razor-pages/validation)
* [Convalida del modello in ASP.NET Core](xref:mvc/models/validation).

## <a name="handle-head-requests-with-an-onget-handler-fallback"></a>Gestire le richieste HEAD con un fallback del gestore OnGet

`HEAD` le richieste consentono di recuperare le intestazioni per una risorsa specifica. A differenza delle richieste `GET`, le richieste `HEAD` non restituiscono un corpo della risposta.

In genere, per le richieste `HEAD` viene creato e chiamato un gestore `OnHead`:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Privacy.cshtml.cs?name=snippet)]

Razor Le pagine eseguono il fallback alla chiamata del `OnGet` gestore se non `OnHead` è stato definito alcun gestore.

<a name="xsrf"></a>

## <a name="xsrfcsrf-and-razor-pages"></a>XSRF/CSRF e Razor pagine

Razor Le pagine sono protette dalla [convalida antifalsificazione](xref:security/anti-request-forgery). [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) inserisce i token antifalsificazione negli elementi del form HTML.

<a name="layout"></a>

## <a name="using-layouts-partials-templates-and-tag-helpers-with-razor-pages"></a>Uso di layout, parti parziali, modelli e helper tag con Razor pagine

Le pagine funzionano con tutte le funzionalità del Razor motore di visualizzazione. Layout, parti parziali, modelli, helper tag, *_ViewStart. cshtml* e *_ViewImports. cshtml* funzionano allo stesso modo per le Razor visualizzazioni convenzionali.

La pagina verrà ora riorganizzata in modo da usufruire di alcune di queste funzionalità.

Aggiungere una [pagina di layout](xref:mvc/views/layout) a *Pages/Shared/_Layout.cshtml*:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Shared/_Layout2.cshtml?hightlight=12)]

Il [layout](xref:mvc/views/layout):

* Controlla il layout di ogni pagina, a meno che la pagina non accetti il layout.
* Importa le strutture HTML, ad esempio JavaScript e i fogli di stile.
* Viene eseguito il rendering del contenuto della Razor pagina dove `@RenderBody()` viene chiamato il metodo.

Per ulteriori informazioni, vedere [pagina layout](xref:mvc/views/layout).

La proprietà [Layout](xref:mvc/views/layout#specifying-a-layout) proprietà viene impostata in *Pages/_ViewStart.cshtml*:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewStart.cshtml)]

Il layout è nella cartella *Pages/Shared*. Le pagine cercano altre visualizzazioni (layout, modelli, righe parzialmente eseguite) in modo gerarchico, partendo dalla stessa cartella della pagina corrente. Un layout nella cartella pages */Shared* può essere utilizzato da qualsiasi Razor pagina nella cartella *pages* .

Il file di layout dovrebbe andare nella cartella *Pages/Shared*.

Si consiglia di **non** inserire il file di layout nella cartella *Views/Shared*. *Views/Shared* è un modello destinato alle visualizzazioni MVC. Razor Le pagine hanno lo scopo di basarsi sulla gerarchia di cartelle, non sulle convenzioni di percorso.

Visualizza la ricerca da una Razor pagina include la cartella *pagine* . I layout, i modelli e i parziali utilizzati con i controller MVC e le Razor visualizzazioni convenzionali *funzionano semplicemente*.

Aggiungere un file *Pages/_ViewImports.cshtml*:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml)]

`@namespace` viene spiegato in seguito nell'esercitazione. La direttiva `@addTagHelper` inserisce gli [helper tag predefiniti](xref:mvc/views/tag-helpers/builtin-th/Index) in tutte le pagine presenti nella cartella *Pages*.

<a name="namespace"></a>

La `@namespace` direttiva impostata in una pagina:

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Customers/Namespace2.cshtml?highlight=2)]

La `@namespace` direttiva imposta lo spazio dei nomi per la pagina. La direttiva `@model` non deve necessariamente includere lo spazio dei nomi.

Se la direttiva `@namespace` è contenuta in *_ViewImports.cshtml*, lo spazio dei nomi specificato indica il prefisso per lo spazio dei nomi generato nella pagina che importa la direttiva `@namespace`. Il resto dello spazio dei nomi generato (la parte del suffisso) è il percorso relativo separato dal punto tra la cartella contenente *_Viewimports.cshtml* e la cartella contenente la pagina.

Ad esempio, la classe `PageModel`*Pages/Customers/Edit.cshtml.cs* imposta in modo esplicito lo spazio dei nomi:

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/Edit.cshtml.cs?name=snippet_namespace)]

Il file *Pages/_ViewImports.cshtml* file imposta il seguente spazio dei nomi:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml?highlight=1)]

Lo spazio dei nomi generato per la pagina *pages/Customers/Edit. cshtml* Razor è uguale a quello della `PageModel` classe.

`@namespace`*funziona anche con le Razor visualizzazioni convenzionali.*

Si consideri il file di visualizzazione *pages/create. cshtml* :

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create3.cshtml?highlight=2-3)]

Il file di visualizzazione *pages/create. cshtml* aggiornato con *_ViewImports. cshtml* e il file di layout precedente:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create4.cshtml?highlight=2)]

Nel codice precedente, il *_ViewImports. cshtml* ha importato lo spazio dei nomi e gli helper tag. Il file di layout ha importato i file JavaScript.

Il [ Razor progetto di avvio pagine](#rpvs17) contiene le *pagine/_ValidationScriptsPartial. cshtml*, che consente di associare la convalida lato client.

Per altre informazioni sulle visualizzazioni parziali, vedere <xref:mvc/views/partial>.

<a name="url_gen"></a>

## <a name="url-generation-for-pages"></a>Generazione di URL per le pagine

La pagina `Create`, riportata in precedenza, usa `RedirectToPage`:

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_PageModel&highlight=28)]

L'applicazione ha la struttura di file o cartella seguente:

* */Pages*

  * *Index.cshtml*
  * *Privacy. cshtml*
  * */Customers*

    * *Create.cshtml*
    * *Edit.cshtml*
    * *Index.cshtml*

Le pagine *pages/Customers/create. cshtml* e *pages/Customers/Edit. cshtml* reindirizza a *pages/Customers/index. cshtml* dopo l'esito positivo. La stringa `./Index` è un nome di pagina relativo usato per accedere alla pagina precedente. Viene usato per generare gli URL nella pagina *pages/Customers/index. cshtml* . Ad esempio:

* `Url.Page("./Index", ...)`
* `<a asp-page="./Index">Customers Index Page</a>`
* `RedirectToPage("./Index")`

Il nome della pagina assoluta `/Index` viene usato per generare gli URL nella pagina *pages/index. cshtml* . Ad esempio:

* `Url.Page("/Index", ...)`
* `<a asp-page="/Index">Home Index Page</a>`
* `RedirectToPage("/Index")`

Il nome della pagina è il percorso alla pagina dalla cartella radice */Pages*, inclusa una barra iniziale `/`, ad esempio `/Index`. Gli esempi di generazione di URL precedenti offrono opzioni avanzate e funzionalità funzionali rispetto a un URL a livello di codice. La generazione di URL usa il [routing](xref:mvc/controllers/routing) ed è in grado di generare e codificare i parametri in base al modo in cui la route è definita nel percorso di destinazione.

La generazione di URL per le pagine supporta i nomi relativi. Nella tabella seguente viene illustrata la pagina di indice selezionata utilizzando `RedirectToPage` parametri diversi in *pages/Customers/create. cshtml*.

| RedirectToPage(x)| Pagina |
| ----------------- | ------------ |
| RedirectToPage("/Index") | *Pages/Index* |
| RedirectToPage("./Index"); | *Pages/Customers/Index* |
| RedirectToPage("../Index") | *Pages/Index* |
| RedirectToPage("Index")  | *Pages/Customers/Index* |

<!-- Test via ~/razor-pages/index/3.0sample/RazorPagesContacts/Pages/Customers/Details.cshtml.cs -->

`RedirectToPage("Index")`, `RedirectToPage("./Index")` e `RedirectToPage("../Index")` sono *nomi relativi*. Il parametro `RedirectToPage` è *combinato* con il percorso della pagina corrente per calcolare il nome della pagina di destinazione.

Il collegamento dei nomi relativi è utile quando si compilano siti con una struttura complessa. Quando vengono usati nomi relativi per il collegamento tra le pagine in una cartella:

* La ridenominazione di una cartella non interrompe i collegamenti relativi.
* I collegamenti non vengono interrotti perché non includono il nome della cartella.

Per reindirizzare la pagina a un'[Area](xref:mvc/controllers/areas) diversa, specificare l'area:

```csharp
RedirectToPage("/Index", new { area = "Services" });
```

Per altre informazioni, vedere <xref:mvc/controllers/areas> e <xref:razor-pages/razor-pages-conventions>.

## <a name="viewdata-attribute"></a>Attributo viewData

I dati possono essere passati a una pagina con <xref:Microsoft.AspNetCore.Mvc.ViewDataAttribute> . Le proprietà con l' `[ViewData]` attributo hanno i valori archiviati e caricati da <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary> .

Nell'esempio seguente, `AboutModel` applica l' `[ViewData]` attributo alla `Title` proprietà:

```csharp
public class AboutModel : PageModel
{
    [ViewData]
    public string Title { get; } = "About";

    public void OnGet()
    {
    }
}
```

Nella pagina About (Informazioni) accedere alla proprietà `Title` come proprietà del modello:

```cshtml
<h1>@Model.Title</h1>
```

Nel layout il titolo viene letto dal dizionario ViewData:

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

## <a name="tempdata"></a>TempData

ASP.NET Core espone <xref:Microsoft.AspNetCore.Mvc.Controller.TempData> . Questa proprietà archivia i dati finché non viene letta. I metodi <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.TempDataDictionary.Keep*> e <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.TempDataDictionary.Peek*> possono essere usati per esaminare i dati senza eliminazione. `TempData` è utile per il reindirizzamento, quando i dati sono necessari per più di una singola richiesta.

Il codice seguente imposta il valore di `Message` usando `TempData`:

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateDot.cshtml.cs?highlight=10-11,25&name=snippet_Temp)]

Il markup seguente nel file *Pages/Customers/Index.cshtml* visualizza il valore di `Message` usando `TempData`.

```cshtml
<h3>Msg: @Model.Message</h3>
```

Il modello di pagina *Pages/Customers/Index.cshtml.cs* applica l'attributo `[TempData]` alla proprietà `Message`.

```csharp
[TempData]
public string Message { get; set; }
```

Per ulteriori informazioni, vedere [TempData](xref:fundamentals/app-state#tempdata).

<a name="mhpp"></a>

## <a name="multiple-handlers-per-page"></a>Più gestori per pagina

La pagina seguente genera markup per due gestori usando l'helper tag `asp-page-handler`:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?highlight=12-13)]

Il form nell'esempio precedente contiene due pulsanti di invio, ognuno dei quali usa `FormActionTagHelper` per l'invio a un URL diverso. L'attributo `asp-page-handler` è correlato a `asp-page`. `asp-page-handler` genera gli URL che indirizzano a ognuno dei metodi gestore definiti da una pagina. `asp-page` non è specificato poiché l'esempio è collegato alla pagina corrente.

Il modello di pagina:

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml.cs?highlight=20,32)]

Il codice precedente usa *metodi gestore denominati*. I metodi gestore denominati vengono creati usando il testo che nel nome segue `On<HTTP Verb>` e precede `Async` (se presente). Nell'esempio precedente i metodi della pagina sono OnPost **JoinList** Async e OnPost **JoinListUC** Async. Rimuovendo *OnPost* e *Async*, i nomi dei gestori sono `JoinList` e `JoinListUC`.

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?range=12-13)]

Usando il codice precedente, il percorso URL che indirizza a `OnPostJoinListAsync` è `https://localhost:5001/Customers/CreateFATH?handler=JoinList`. Il percorso URL che indirizza a `OnPostJoinListUCAsync` è `https://localhost:5001/Customers/CreateFATH?handler=JoinListUC`.

## <a name="custom-routes"></a>Route personalizzate

Usare la direttiva `@page` per:

* Specificare una route personalizzata a una pagina. Ad esempio, è possibile impostare la route alla pagina About (Informazioni) su `/Some/Other/Path` con `@page "/Some/Other/Path"`.
* Aggiungere segmenti alla route predefinita di una pagina. Ad esempio, è possibile aggiungere un segmento "item" alla route predefinita di una pagina con `@page "item"`.
* Aggiungere parametri alla route predefinita di una pagina. Ad esempio, un parametro ID, `id`, può essere necessario per una pagina con `@page "{id}"`.

Un percorso relativo alla directory radice designato da una tilde (`~`) all'inizio del percorso è supportato. Ad esempio, `@page "~/Some/Other/Path"` è identico a `@page "/Some/Other/Path"`.

Se non si preferisce la stringa di query `?handler=JoinList` nell'URL, modificare la route per inserire il nome del gestore nella parte relativa al percorso dell'URL. La route può essere personalizzata aggiungendo un modello di route racchiuso tra virgolette doppie dopo la `@page` direttiva.

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateRoute.cshtml?highlight=1)]

Usando il codice precedente, il percorso URL che indirizza a `OnPostJoinListAsync` è `https://localhost:5001/Customers/CreateFATH/JoinList`. Il percorso URL che indirizza a `OnPostJoinListUCAsync` è `https://localhost:5001/Customers/CreateFATH/JoinListUC`.

`?` che segue `handler` indica che il parametro di route è facoltativo.

## <a name="advanced-configuration-and-settings"></a>Impostazioni e configurazione avanzate

La configurazione e le impostazioni nelle sezioni seguenti non sono richieste dalla maggior parte delle app.

Per configurare le opzioni avanzate, usare l' <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages%2A> Overload che configura <xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions> :

[!code-csharp[](index/3.0sample/RazorPagesContacts/StartupRPoptions.cs?name=snippet)]

Utilizzare <xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions> per impostare la directory radice per le pagine o aggiungere le convenzioni del modello applicativo per le pagine. Per ulteriori informazioni sulle convenzioni, vedere la pagina relativa alle [ Razor convenzioni di autorizzazione](xref:security/authorization/razor-pages-authorization).

Per eseguire la precompilazione delle visualizzazioni, vedere [ Razor visualizzare la compilazione](xref:mvc/views/view-compilation).

### <a name="specify-that-razor-pages-are-at-the-content-root"></a>Consente di specificare che Razor le pagine si trovano nella radice del contenuto

Per impostazione predefinita, Razor le pagine hanno una radice nella directory */pages* Aggiungere <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.WithRazorPagesAtContentRoot*> per specificare che le Razor pagine si trovano nella [radice del contenuto](xref:fundamentals/index#content-root) ( <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootPath> ) dell'app:

[!code-csharp[](index/3.0sample/RazorPagesContacts/StartupWithRazorPagesAtContentRoot.cs?name=snippet)]

### <a name="specify-that-razor-pages-are-at-a-custom-root-directory"></a>Consente di specificare che le Razor pagine si trovano in una directory radice personalizzata

Aggiungere <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcCoreBuilderExtensions.WithRazorPagesRoot*> per specificare che Razor le pagine si trovano in una directory radice personalizzata nell'app (fornire un percorso relativo):

[!code-csharp[](index/3.0sample/RazorPagesContacts/StartupWithRazorPagesRoot.cs?name=snippet)]

## <a name="additional-resources"></a>Risorse aggiuntive

* Vedere Introduzione [alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start), che si basa su questa introduzione.
* [Autorizzare gli attributi e le Razor pagine](xref:security/authorization/simple#aarp)
* [Scaricare o visualizzare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/index/3.0sample)
* <xref:index>
* [Razor informazioni di riferimento sulla sintassi per ASP.NET Core](xref:mvc/views/razor)
* <xref:mvc/controllers/areas>
* <xref:tutorials/razor-pages/razor-pages-start>
* <xref:security/authorization/razor-pages-authorization>
* <xref:razor-pages/razor-pages-conventions>
* <xref:test/razor-pages-tests>
* <xref:mvc/views/partial>
* <xref:blazor/components/prerendering-and-integration>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

<a name="rpvs17"></a>

## <a name="create-a-razor-pages-project"></a>Creazione di un Razor progetto di pagine

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Per istruzioni dettagliate su come creare un progetto di pagine, vedere [Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) Razor .

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Eseguire `dotnet new webapp` dalla riga di comando.

Aprire il file *CSPROJ* generato da Visual Studio per Mac.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Eseguire `dotnet new webapp` dalla riga di comando.

---

## <a name="razor-pages"></a>Razor Pagine

Razor Le pagine sono abilitate in *Startup.cs*:

[!code-csharp[](index/sample/RazorPagesIntro/Startup.cs?name=snippet_Startup)]

Si consideri una pagina di base: <a name="OnGet"></a>

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Index.cshtml)]

Il codice precedente ha un aspetto molto simile a quello di un [ Razor file di visualizzazione](xref:tutorials/first-mvc-app/adding-view) usato in un'app ASP.NET Core con i controller e le visualizzazioni. Ciò che lo differenzia è la direttiva `@page`. `@page` trasforma il file in un'azione MVC, ovvero gestisce le richieste direttamente, senza passare attraverso un controller. `@page` deve essere la prima Razor direttiva in una pagina. `@page` influiscono sul comportamento di altri Razor costrutti.

Nei due file seguenti viene visualizzata una pagina simile che usa una classe `PageModel`. Il file *Pages/Index2.cshtml*:

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Index2.cshtml)]

Il modello di pagina *Pages/Index2.cshtml.cs*:

[!code-csharp[](index/sample/RazorPagesIntro/Pages/Index2.cshtml.cs)]

Per convenzione, il `PageModel` file di classe ha lo stesso nome del Razor file di paging con l'aggiunta di *. cs* . La pagina precedente, ad esempio, Razor è *pages/index2. cshtml*. Il file che contiene la classe `PageModel` è denominato *Pages/Index2.cshtml.cs*.

Le associazioni dei percorsi URL alle pagine sono determinate dalla posizione della pagina nel file system. La tabella seguente mostra un Razor percorso di pagina e l'URL corrispondente:

| Percorso e nome file               | URL corrispondente |
| ----------------- | ------------ |
| */Pages/Index.cshtml* | `/` o `/Index` |
| */Pages/Contact.cshtml* | `/Contact` |
| */Pages/Store/Contact.cshtml* | `/Store/Contact` |
| */Pages/Store/Index.cshtml* | `/Store` o `/Store/Index` |

Note:

* Per Razor impostazione predefinita, il runtime cerca i file di pagine nella cartella *pages* .
* `Index` è la pagina predefinita quando un URL non include una pagina.

## <a name="write-a-basic-form"></a>Scrivere un form di base

Razor Pagine è progettato per semplificare l'implementazione dei modelli comuni usati con i Web browser quando si compila un'app. L' [associazione di modelli](xref:mvc/models/model-binding), gli [Helper Tag](xref:mvc/views/tag-helpers/intro)e gli helper HTML *funzionano solo* con le proprietà definite in una Razor classe di pagine. Si consideri una pagina che implementa un form "contact us" di base per il modello `Contact`:

Per gli esempi riportati in questo documento, `DbContext` viene inizializzato nel file [Startup.cs](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/razor-pages/index/sample/RazorPagesContacts/Startup.cs#L15-L16).

[!code-csharp[](index/sample/RazorPagesContacts/Startup.cs?highlight=15-16)]

Il modello di dati:

[!code-csharp[](index/sample/RazorPagesContacts/Data/Customer.cs)]

Il contesto del database:

[!code-csharp[](index/sample/RazorPagesContacts/Data/AppDbContext.cs)]

Il file di visualizzazione *Pages/Create.cshtml*:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Create.cshtml)]

Il modello di pagina *Pages/Create.cshtml.cs*:

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_ALL)]

Per convenzione, la classe `PageModel` è denominata `<PageName>Model` e si trova nello stesso spazio dei nomi della pagina.

La classe `PageModel` consente la separazione della logica di una pagina dalla relativa presentazione. Definisce i gestori di pagina per le richieste inviate alla pagina e i dati usati per il rendering della pagina. Questa separazione consente:

* Gestione delle dipendenze di pagina tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).
* [Testing unità](xref:test/razor-pages-tests) delle pagine.

La pagina contiene un oggetto , ovvero un `OnPostAsync` *metodo gestore* che viene eseguito per le richieste `POST`, quando un utente invia il form. È possibile aggiungere metodi del gestore per qualsiasi verbo HTTP. I gestori più comuni sono i seguenti:

* `OnGet` per inizializzare lo stato necessario per la pagina. Esempio di [OnGet](#OnGet).
* `OnPost` per gestire gli invii di form.

Il suffisso `Async` nel nome è facoltativo, ma viene spesso usato per convenzione per le funzioni asincrone. Il codice precedente è tipico per le Razor pagine.

Se si ha familiarità con le app ASP.NET che usano i controller e le visualizzazioni:

* Il `OnPostAsync` codice nell'esempio precedente ha un aspetto simile al codice del controller tipico.
* La maggior parte delle primitive MVC, ad esempio l' [associazione di modelli](xref:mvc/models/model-binding), la [convalida](xref:mvc/models/validation), la [convalida](xref:mvc/models/validation)e i risultati dell'azione, sono condivisi.

Il metodo `OnPostAsync` precedente:

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync)]

Il flusso di base di `OnPostAsync`:

Verificare se sono presenti errori di convalida.

* Se non sono presenti errori, salvare i dati e reindirizzare.
* Se sono presenti errori, visualizzare di nuovo la pagina con i messaggi di convalida. La convalida lato client è identica alle applicazioni ASP.NET Core MVC tradizionali. In molti casi, gli errori di convalida vengono rilevati nel client e non vengono mai inviati al server.

Quando i dati vengono immessi correttamente, il metodo gestore `OnPostAsync` chiama il metodo helper `RedirectToPage` per restituire un'istanza di `RedirectToPageResult`. `RedirectToPage` è un nuovo risultato dell'azione, simile a `RedirectToAction` o `RedirectToRoute`, ma personalizzato per le pagine. Nell'esempio precedente viene reindirizzato alla pagina di indice radice (`/Index`). `RedirectToPage` è descritto in dettaglio nella sezione [generazione URL per pagine](#url_gen) .

Quando il form inviato contiene errori di convalida (che vengono passati al server), il metodo gestore `OnPostAsync` chiama il metodo helper `Page`. `Page` restituisce un'istanza di `PageResult`. La restituzione di `Page` è simile al modo in cui le azioni nel controller restituiscono `View`. `PageResult` è il tipo restituito predefinito per un metodo del gestore. Un metodo gestore che restituisce `void` esegue il rendering della pagina.

La proprietà `Customer` usa l'attributo `[BindProperty]` optare per consentire l'associazione di modelli.

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_PageModel&highlight=10-11)]

Razor Le pagine, per impostazione predefinita, associano proprietà solo con non `GET` verbi. L'associazione alle proprietà può ridurre la quantità di codice da scrivere. Riduce il codice usando la stessa proprietà per il eseguire il rendering dei campi del form (`<input asp-for="Customer.Name">`) e accettare l'input.

[!INCLUDE[](~/includes/bind-get.md)]

La home page (*Index.cshtml*):

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml)]

La classe `PageModel` associata (*Index.cshtml.cs*):

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs)]

Il file *Index.cshtml* contiene il markup seguente per creare un collegamento di modifica per ogni contatto:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=21)]

L' `<a asp-page="./Edit" asp-route-id="@contact.Id">Edit</a>` [Helper tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) ha usato l' `asp-route-{value}` attributo per generare un collegamento alla pagina di modifica. Il collegamento contiene i dati della route con l'ID contatto. Ad esempio: `https://localhost:5001/Edit/1`. Gli [Helper Tag](xref:mvc/views/tag-helpers/intro) consentono al codice lato server di partecipare alla creazione e al rendering di elementi HTML nei Razor file. Gli helper tag sono abilitati da `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers`

Il file *Pages/Edit.cshtml*:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Edit.cshtml?highlight=1)]

La prima riga contiene la direttiva `@page "{id:int}"`. Il vincolo di routing `"{id:int}"` indica alla pagina di accettare le richieste inviate alla pagina che contengono i dati della route `int`. Se una richiesta inviata alla pagina non contiene dati della route che possono essere convertiti in un oggetto `int`, il runtime restituisce un errore HTTP 404 (non trovato). Per rendere l'ID facoltativo, aggiungere `?` al vincolo di route:

 ```cshtml
@page "{id:int?}"
```

Il file *Pages/Edit.cshtml.cs*:

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Edit.cshtml.cs)]

Il file *Index.cshtml* contiene anche il markup per creare un pulsante di eliminazione per ogni contatto cliente:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=22-23)]

Quando viene eseguito il rendering del pulsante di eliminazione in HTML, il relativo `formaction` include i parametri per:

* L'ID di contatto cliente dall'attributo `asp-route-id`.
* L'`handler` specificato dall'attributo `asp-page-handler`.

Di seguito è riportato un esempio di pulsante di eliminazione di cui è stato eseguito il rendering con un ID di contatto cliente `1`:

```html
<button type="submit" formaction="/?id=1&amp;handler=delete">delete</button>
```

Quando il pulsante è selezionato, viene inviata una richiesta `POST` di modulo al server. Per convenzione, il nome del metodo del gestore viene selezionato in base al valore del parametro `handler` in base allo schema `OnPost[handler]Async`.

Poiché in questo esempio l'`handler` è `delete`, il metodo `OnPostDeleteAsync` viene usato per elaborare la richiesta `POST`. Se `asp-page-handler` viene impostato su un valore diverso, ad esempio `remove`, viene selezionato un metodo gestore con il nome `OnPostRemoveAsync`. Il codice seguente illustra il `OnPostDeleteAsync` gestore:

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs?range=26-37)]

Il metodo `OnPostDeleteAsync`:

* Accetta l'`id` dalla stringa di query. Se la direttiva della pagina *index. cshtml* conteneva il vincolo `"{id:int?}"` di routing, `id` verrebbe derivare dai dati della route. I dati della route per `id` sono specificati nell'URI, ad esempio `https://localhost:5001/Customers/2` .
* Interroga il database in merito al contatto del cliente con `FindAsync`.
* Se viene trovato il contatto cliente, viene rimosso dall'elenco dei contatti del cliente. Il database viene aggiornato.
* Chiama `RedirectToPage` per reindirizzare alla pagina di indice radice (`/Index`).

## <a name="mark-page-properties-as-required"></a>Contrassegnare le proprietà della pagina in base alle esigenze

Le proprietà di un oggetto `PageModel` possono essere contrassegnate con l'attributo [obbligatorio](/dotnet/api/system.componentmodel.dataannotations.requiredattribute) :

[!code-csharp[](index/sample/Create.cshtml.cs?highlight=3,15-16)]

Per altre informazioni, vedere [Convalida del modello](xref:mvc/models/validation).

## <a name="handle-head-requests-with-an-onget-handler-fallback"></a>Gestire le richieste HEAD con un fallback del gestore OnGet

Le richieste `HEAD` consentono di recuperare le intestazioni di una risorsa specifica. A differenza delle richieste `GET`, le richieste `HEAD` non restituiscono un corpo della risposta.

In genere, per le richieste `HEAD` viene creato e chiamato un gestore `OnHead`: 

```csharp
public void OnHead()
{
    HttpContext.Response.Headers.Add("HandledBy", "Handled by OnHead!");
}
```

In ASP.NET Core 2,1 o versioni successive, Razor le pagine eseguono il fallback alla chiamata del `OnGet` gestore se non `OnHead` è stato definito alcun gestore. Questo comportamento è abilitato tramite la chiamata a [SetCompatibilityVersion](xref:mvc/compatibility-version) in `Startup.ConfigureServices`:

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
```

I modelli predefiniti generano la chiamata `SetCompatibilityVersion` in ASP.NET Core 2.1 e 2.2. `SetCompatibilityVersion` imposta effettivamente l' Razor opzione Pages `AllowMappingHeadRequestsToGetHandler` su `true` .

Anziché acconsentire esplicitamente a tutti i comportamenti con `SetCompatibilityVersion`, è possibile acconsentire a comportamenti *specifici*. Il codice seguente acconsente esplicitamente al mapping delle richieste `HEAD` al gestore `OnGet`:

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.AllowMappingHeadRequestsToGetHandler = true;
    });
```

<a name="xsrf"></a>

## <a name="xsrfcsrf-and-razor-pages"></a>XSRF/CSRF e Razor pagine

Non è necessario scrivere codice per la [convalida antifalsificazione](xref:security/anti-request-forgery). La generazione e la convalida di token antifalsificazione sono incluse automaticamente nelle Razor pagine.

<a name="layout"></a>

## <a name="using-layouts-partials-templates-and-tag-helpers-with-razor-pages"></a>Uso di layout, parti parziali, modelli e helper tag con Razor pagine

Le pagine funzionano con tutte le funzionalità del Razor motore di visualizzazione. Layout, parti parziali, modelli, helper tag, *_ViewStart. cshtml*, *_ViewImports. cshtml* funzionano esattamente come per le Razor visualizzazioni convenzionali.

La pagina verrà ora riorganizzata in modo da usufruire di alcune di queste funzionalità.

Aggiungere una [pagina di layout](xref:mvc/views/layout) a *Pages/Shared/_Layout.cshtml*:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_LayoutSimple.cshtml)]

Il [layout](xref:mvc/views/layout):

* Controlla il layout di ogni pagina, a meno che la pagina non accetti il layout.
* Importa le strutture HTML, ad esempio JavaScript e i fogli di stile.

Vedere l'articolo sulla [pagina Layout](xref:mvc/views/layout) per altre informazioni.

La proprietà [Layout](xref:mvc/views/layout#specifying-a-layout) proprietà viene impostata in *Pages/_ViewStart.cshtml*:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewStart.cshtml)]

Il layout è nella cartella *Pages/Shared*. Le pagine cercano altre visualizzazioni (layout, modelli, righe parzialmente eseguite) in modo gerarchico, partendo dalla stessa cartella della pagina corrente. Un layout nella cartella pages */Shared* può essere utilizzato da qualsiasi Razor pagina nella cartella *pages* .

Il file di layout dovrebbe andare nella cartella *Pages/Shared*.

Si consiglia di **non** inserire il file di layout nella cartella *Views/Shared*. *Views/Shared* è un modello destinato alle visualizzazioni MVC. Razor Le pagine hanno lo scopo di basarsi sulla gerarchia di cartelle, non sulle convenzioni di percorso.

Visualizza la ricerca da una Razor pagina include la cartella *pagine* . I layout, i modelli e i parziali utilizzati con i controller MVC e le Razor visualizzazioni convenzionali *funzionano semplicemente*.

Aggiungere un file *Pages/_ViewImports.cshtml*:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml)]

`@namespace` viene spiegato in seguito nell'esercitazione. La direttiva `@addTagHelper` inserisce gli [helper tag predefiniti](xref:mvc/views/tag-helpers/builtin-th/Index) in tutte le pagine presenti nella cartella *Pages*.

<a name="namespace"></a>

Quando la direttiva `@namespace` viene usata in modo esplicito in una pagina:

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Customers/Namespace2.cshtml?highlight=2)]

La direttiva imposta lo spazio dei nomi per la pagina. La direttiva `@model` non deve necessariamente includere lo spazio dei nomi.

Se la direttiva `@namespace` è contenuta in *_ViewImports.cshtml*, lo spazio dei nomi specificato indica il prefisso per lo spazio dei nomi generato nella pagina che importa la direttiva `@namespace`. Il resto dello spazio dei nomi generato (la parte del suffisso) è il percorso relativo separato dal punto tra la cartella contenente *_Viewimports.cshtml* e la cartella contenente la pagina.

Ad esempio, la classe `PageModel`*Pages/Customers/Edit.cshtml.cs* imposta in modo esplicito lo spazio dei nomi:

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/Edit.cshtml.cs?name=snippet_namespace)]

Il file *Pages/_ViewImports.cshtml* file imposta il seguente spazio dei nomi:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml?highlight=1)]

Lo spazio dei nomi generato per la pagina *pages/Customers/Edit. cshtml* Razor è uguale a quello della `PageModel` classe.

`@namespace`*funziona anche con le Razor visualizzazioni convenzionali.*

Il file di visualizzazione *Pages/Create.cshtml* originale:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Create.cshtml?highlight=2)]

Il file di visualizzazione *Pages/Create.cshtml* aggiornato:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/Create.cshtml?highlight=2)]

Il [ Razor progetto di avvio pagine](#rpvs17) contiene le *pagine/_ValidationScriptsPartial. cshtml*, che consente di associare la convalida lato client.

Per altre informazioni sulle visualizzazioni parziali, vedere <xref:mvc/views/partial>.

<a name="url_gen"></a>

## <a name="url-generation-for-pages"></a>Generazione di URL per le pagine

La pagina `Create`, riportata in precedenza, usa `RedirectToPage`:

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=10)]

L'applicazione ha la struttura di file o cartella seguente:

* */Pages*

  * *Index.cshtml*
  * */Customers*

    * *Create.cshtml*
    * *Edit.cshtml*
    * *Index.cshtml*

Le pagine *Pages/Customers/Create.cshtml* e *Pages/Customers/Edit.cshtml* reindirizzano a *Pages/Index.cshtml* dopo l'esecuzione. La stringa `/Index` fa parte dell'URI di accesso alla pagina precedente. La stringa `/Index` può essere usata per generare gli URI alla pagina *Pages/Index.cshtml*. Ad esempio:

* `Url.Page("/Index", ...)`
* `<a asp-page="/Index">My Index Page</a>`
* `RedirectToPage("/Index")`

Il nome della pagina è il percorso alla pagina dalla cartella radice */Pages*, inclusa una barra iniziale `/`, ad esempio `/Index`. Gli esempi di generazione di URL precedenti offrono opzioni e caratteristiche funzionali avanzate rispetto agli URL hardcoded. La generazione di URL usa il [routing](xref:mvc/controllers/routing) ed è in grado di generare e codificare i parametri in base al modo in cui la route è definita nel percorso di destinazione.

La generazione di URL per le pagine supporta i nomi relativi. La tabella seguente indica quale pagina di indice viene selezionata con diversi parametri `RedirectToPage` da *Pages/Customers/Create.cshtml*:

| RedirectToPage(x)| Pagina |
| ----------------- | ------------ |
| RedirectToPage("/Index") | *Pages/Index* |
| RedirectToPage("./Index"); | *Pages/Customers/Index* |
| RedirectToPage("../Index") | *Pages/Index* |
| RedirectToPage("Index")  | *Pages/Customers/Index* |

`RedirectToPage("Index")`, `RedirectToPage("./Index")` e `RedirectToPage("../Index")`  sono *nomi relativi*. Il parametro `RedirectToPage` è *combinato* con il percorso della pagina corrente per calcolare il nome della pagina di destinazione.  <!-- Review: Original had The provided string is combined with the page name of the current page to compute the name of the destination page.  page name, not page path -->

Il collegamento dei nomi relativi è utile quando si compilano siti con una struttura complessa. Se si usano i nomi relativi per il collegamento tra le pagine in una cartella, è possibile rinominare tale cartella. Tutti i collegamenti continuano a funzionare perché non includono il nome della cartella.

Per reindirizzare la pagina a un'[Area](xref:mvc/controllers/areas) diversa, specificare l'area:

```csharp
RedirectToPage("/Index", new { area = "Services" });
```

Per altre informazioni, vedere <xref:mvc/controllers/areas>.

## <a name="viewdata-attribute"></a>Attributo viewData

È possibile passare i dati a una pagina tramite [ViewDataAttribute](/dotnet/api/microsoft.aspnetcore.mvc.viewdataattribute). I valori delle proprietà nei controller o nei Razor modelli di pagina con l' `[ViewData]` attributo sono archiviati e caricati da [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary).

Nell'esempio seguente, `AboutModel` contiene una `Title` proprietà contrassegnata con `[ViewData]` . La proprietà `Title` è impostata sul titolo della pagina About (Informazioni):

```csharp
public class AboutModel : PageModel
{
    [ViewData]
    public string Title { get; } = "About";

    public void OnGet()
    {
    }
}
```

Nella pagina About (Informazioni) accedere alla proprietà `Title` come proprietà del modello:

```cshtml
<h1>@Model.Title</h1>
```

Nel layout il titolo viene letto dal dizionario ViewData:

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

## <a name="tempdata"></a>TempData

ASP.NET Core espone la proprietà [TempData](/dotnet/api/microsoft.aspnetcore.mvc.controller.tempdata#Microsoft_AspNetCore_Mvc_Controller_TempData) per un [controller](/dotnet/api/microsoft.aspnetcore.mvc.controller). Questa proprietà archivia i dati finché non viene letta. I metodi `Keep` e `Peek` possono essere usati per esaminare i dati senza eliminazione. `TempData` è utile per il reindirizzamento quando i dati sono necessari per più di una singola richiesta.

Il codice seguente imposta il valore di `Message` usando `TempData`:

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateDot.cshtml.cs?highlight=10-11,25&name=snippet_Temp)]

Il markup seguente nel file *Pages/Customers/Index.cshtml* visualizza il valore di `Message` usando `TempData`.

```cshtml
<h3>Msg: @Model.Message</h3>
```

Il modello di pagina *Pages/Customers/Index.cshtml.cs* applica l'attributo `[TempData]` alla proprietà `Message`.

```csharp
[TempData]
public string Message { get; set; }
```

Per altre informazioni, vedere [TempData](xref:fundamentals/app-state#tempdata).

<a name="mhpp"></a>

## <a name="multiple-handlers-per-page"></a>Più gestori per pagina

La pagina seguente genera markup per due gestori usando l'helper tag `asp-page-handler`:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?highlight=12-13)]

<!-- Review: the FormActionTagHelper applies to all <form /> elements on a Razor page, even when there's no `asp-` attribute   -->

Il form nell'esempio precedente contiene due pulsanti di invio, ognuno dei quali usa `FormActionTagHelper` per l'invio a un URL diverso. L'attributo `asp-page-handler` è correlato a `asp-page`. `asp-page-handler` genera gli URL che indirizzano a ognuno dei metodi gestore definiti da una pagina. `asp-page` non è specificato poiché l'esempio è collegato alla pagina corrente.

Il modello di pagina:

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml.cs?highlight=20,32)]

Il codice precedente usa *metodi gestore denominati*. I metodi gestore denominati vengono creati usando il testo che nel nome segue `On<HTTP Verb>` e precede `Async` (se presente). Nell'esempio precedente i metodi della pagina sono OnPost **JoinList** Async e OnPost **JoinListUC** Async. Rimuovendo *OnPost* e *Async*, i nomi dei gestori sono `JoinList` e `JoinListUC`.

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?range=12-13)]

Usando il codice precedente, il percorso URL che indirizza a `OnPostJoinListAsync` è `https://localhost:5001/Customers/CreateFATH?handler=JoinList`. Il percorso URL che indirizza a `OnPostJoinListUCAsync` è `https://localhost:5001/Customers/CreateFATH?handler=JoinListUC`.

## <a name="custom-routes"></a>Route personalizzate

Usare la direttiva `@page` per:

* Specificare una route personalizzata a una pagina. Ad esempio, è possibile impostare la route alla pagina About (Informazioni) su `/Some/Other/Path` con `@page "/Some/Other/Path"`.
* Aggiungere segmenti alla route predefinita di una pagina. Ad esempio, è possibile aggiungere un segmento "item" alla route predefinita di una pagina con `@page "item"`.
* Aggiungere parametri alla route predefinita di una pagina. Ad esempio, un parametro ID, `id`, può essere necessario per una pagina con `@page "{id}"`.

Un percorso relativo alla directory radice designato da una tilde (`~`) all'inizio del percorso è supportato. Ad esempio, `@page "~/Some/Other/Path"` è identico a `@page "/Some/Other/Path"`.

Se non si preferisce la stringa di query `?handler=JoinList` nell'URL, modificare la route per inserire il nome del gestore nella parte relativa al percorso dell'URL. La route può essere personalizzata aggiungendo un modello di route racchiuso tra virgolette doppie dopo la `@page` direttiva.

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateRoute.cshtml?highlight=1)]

Usando il codice precedente, il percorso URL che indirizza a `OnPostJoinListAsync` è `https://localhost:5001/Customers/CreateFATH/JoinList`. Il percorso URL che indirizza a `OnPostJoinListUCAsync` è `https://localhost:5001/Customers/CreateFATH/JoinListUC`.

`?` che segue `handler` indica che il parametro di route è facoltativo.

## <a name="configuration-and-settings"></a>Configurazione e impostazioni

Per configurare le opzioni avanzate, usare il metodo di estensione `AddRazorPagesOptions` nel generatore MVC:

[!code-csharp[](index/sample/RazorPagesContacts/StartupAdvanced.cs?name=snippet_1)]

Attualmente è possibile usare `RazorPagesOptions` per impostare la directory radice per le pagine o aggiungere convenzioni di modello applicativo per le pagine. In questo modo si potrà garantire una maggiore estendibilità in futuro.

Per eseguire la precompilazione delle visualizzazioni, vedere [ Razor visualizzare la compilazione](xref:mvc/views/view-compilation) .

[Scaricare o visualizzare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/index/sample).

Vedere Introduzione [alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start), che si basa su questa introduzione.

### <a name="specify-that-razor-pages-are-at-the-content-root"></a>Consente di specificare che Razor le pagine si trovano nella radice del contenuto

Per impostazione predefinita, Razor le pagine hanno una radice nella directory */pages* Aggiungere [con Razor PagesAtContentRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvcbuilderextensions.withrazorpagesatcontentroot) a [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) per specificare che le Razor pagine si trovano nella [radice del contenuto](xref:fundamentals/index#content-root) ([ContentRootPath](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.contentrootpath)) dell'app:

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesAtContentRoot();
```

### <a name="specify-that-razor-pages-are-at-a-custom-root-directory"></a>Consente di specificare che le Razor pagine si trovano in una directory radice personalizzata

Aggiungere [con Razor PagesRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvccorebuilderextensions.withrazorpagesroot) a [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) per specificare che le Razor pagine si trovano in una directory radice personalizzata nell'app (fornire un percorso relativo):

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesRoot("/path/to/razor/pages");
```

## <a name="additional-resources"></a>Risorse aggiuntive

* [Autorizzare gli attributi e le Razor pagine](xref:security/authorization/simple#aarp)
* <xref:index>
* [Razor informazioni di riferimento sulla sintassi per ASP.NET Core](xref:mvc/views/razor)
* <xref:mvc/controllers/areas>
* <xref:tutorials/razor-pages/razor-pages-start>
* <xref:security/authorization/razor-pages-authorization>
* <xref:razor-pages/razor-pages-conventions>
* <xref:test/razor-pages-tests>
* <xref:mvc/views/partial>

::: moniker-end
