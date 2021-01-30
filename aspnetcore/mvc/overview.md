---
title: Panoramica di ASP.NET MVC
author: ardalis
description: Informazioni sul framework avanzato di ASP.NET Core MVC per la creazione di app Web e API tramite lo schema progettuale MVC (Model-View-Controller).
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
uid: mvc/overview
ms.openlocfilehash: 642a7b26a3ba0a79a35437a5c19ee0be63a367af
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057317"
---
# <a name="overview-of-aspnet-core-mvc"></a>Panoramica di ASP.NET MVC

Di [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC è un framework avanzato per la creazione di app Web e API tramite lo schema progettuale MVC (Model-View-Controller).

## <a name="what-is-the-mvc-pattern"></a>Cos'è lo schema MVC?

Nello schema architetturale MVC (Model-View-Controller) l'applicazione viene suddivisa in tre gruppi principali di componenti: modelli, visualizzazioni e controller. Questo schema consente di realizzare la [separazione delle competenze](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns). Grazie all'uso di questo schema, le richieste dell'utente vengono indirizzate a un controller responsabile di interagire con il modello per eseguire le azioni dell'utente e/o recuperare i risultati delle query. Il controller sceglie la visualizzazione da mostrare all'utente e le fornisce i dati del modello necessari.

Il diagramma seguente illustra i tre componenti principali e quali fanno riferimento agli altri:

![Schema MVC](overview/_static/mvc.png)

Questa definizione delle responsabilità consente di ridimensionare l'applicazione in termini di complessità, perché è più facile scrivere codice, eseguire il debug e testare qualcosa (modello, visualizzazione o controller) che presenta un unico processo. È più difficile aggiornare, testare ed eseguire il debug di codice con dipendenze distribuite in due o più di queste tre aree. La logica dell'interfaccia utente, ad esempio, tende a cambiare più frequentemente rispetto alla logica di business. Se il codice di presentazione e la logica di business sono combinati in un singolo oggetto, l'oggetto che contiene la logica di business deve essere modificato ogni volta che l'interfaccia utente viene modificata. Ciò spesso introduce errori e rende necessaria la ripetizione dei test della logica di business dopo ogni minima modifica dell'interfaccia utente.

> [!NOTE]
> La visualizzazione e il controller dipendono dal modello. Il modello, tuttavia, non dipende né dalla visualizzazione né dal controller. Questo è uno dei principali vantaggi della separazione. La separazione consente di compilare e testare il modello in modo indipendente dalla presentazione visiva.

### <a name="model-responsibilities"></a>Responsabilità del modello

In un'applicazione MVC il modello rappresenta lo stato dell'applicazione e la logica di business o le operazioni che essa deve eseguire. La logica di business deve essere incapsulata nel modello insieme alla logica di implementazione per rendere persistente lo stato dell'applicazione. Le visualizzazioni fortemente tipizzate usano in genere tipi ViewModel progettati per contenere i dati da visualizzare in una particolare visualizzazione. Il controller crea e popola queste istanze di ViewModel dal modello.

### <a name="view-responsibilities"></a>Responsabilità della visualizzazione

Le visualizzazioni sono responsabili della presentazione del contenuto tramite l'interfaccia utente. Usano il [ Razor motore di visualizzazione](#razor-view-engine) per incorporare il codice .NET nel markup HTML. Nelle visualizzazioni la quantità di logica deve essere minima e la logica in esse contenuta deve essere relativa alla presentazione del contenuto. Se è necessario eseguire una grande quantità di logica nei file di visualizzazione per visualizzare dati da un modello complesso, valutare l'uso di un [componente di visualizzazione](views/view-components.md), un ViewModel o un modello di visualizzazione per semplificare la visualizzazione.

### <a name="controller-responsibilities"></a>Responsabilità del controller

I controller sono i componenti che gestiscono l'interazione dell'utente, interagiscono con il modello e selezionano in definitiva la visualizzazione di cui verrà eseguito il rendering. In un'applicazione MVC la visualizzazione presenta solo le informazioni, mentre il controller gestisce e risponde all'input e all'interazione dell'utente. Nello schema MVC il controller è il punto di ingresso iniziale ed è responsabile della selezione dei tipi di modello con cui interagire e della visualizzazione di cui eseguire il rendering. Come suggerito dal nome, questo componente controlla il modo in cui l'app risponde a una determinata richiesta.

> [!NOTE]
> È consigliabile non sovraccaricare eccessivamente i controller con troppe responsabilità. Per evitare che la logica del controller diventi troppo complessa, escludere la logica di business dal controller e includerla nel modello di dominio.

>[!TIP]
> Se si ritiene che le azioni del controller eseguano frequentemente gli stessi tipi di azioni, spostare queste azioni comuni nei [filtri](#filters).

## <a name="what-is-aspnet-core-mvc"></a>Cos'è ASP.NET Core MVC

Il framework ASP.NET Core MVC è un framework di presentazione leggero, open source e altamente testabile ottimizzato per l'uso con ASP.NET Core.

ASP.NET Core MVC offre un sistema basato su schemi per la creazione di siti Web dinamici che consente una netta separazione delle competenze. Offre il controllo completo sul markup, supporta lo sviluppo basato su test e usa gli standard Web più recenti.

## <a name="features"></a>Funzionalità

ASP.NET Core MVC include:

* [Routing](#routing)
* [Associazione di modelli](#model-binding)
* [Convalida modello](#model-validation)
* [Inserimento delle dipendenze](../fundamentals/dependency-injection.md)
* [Filtri](#filters)
* [Aree](#areas)
* [API Web](#web-apis)
* [Testabilità](#testability)
* [Razor motore di visualizzazione](#razor-view-engine)
* [Visualizzazioni fortemente tipizzate](#strongly-typed-views)
* [Helper per i tag](#tag-helpers)
* [Visualizza componenti](#view-components)

### <a name="routing"></a>Routing

ASP.NET Core MVC si basa sul [routing di ASP.NET Core](../fundamentals/routing.md), un potente componente per il mapping di URL che consente di compilare applicazioni con URL comprensibili che supportano la ricerca. Ciò consente di definire criteri di denominazione dell'URL dell'applicazione che funzionano perfettamente per l'ottimizzazione dei motori di ricerca (SEO) e la generazione di collegamenti, indipendentemente da come sono organizzati i file nel server Web. È possibile definire le route usando una pratica sintassi del modello di route che supporta i vincoli di valore delle route, i valori predefiniti e quelli facoltativi.

Il *routing basato su convenzioni* consente di definire a livello globale i formati URL accettati dall'applicazione e il modo in cui ognuno di questi formati viene mappato a un metodo di azione specifico su un determinato controller. Alla ricezione di una richiesta in ingresso, il motore di routing analizza l'URL e lo confronta con uno dei formati di URL definiti, quindi chiama il metodo di azione del controller associato.

```csharp
routes.MapRoute(name: "Default", template: "{controller=Home}/{action=Index}/{id?}");
```

Il *routing di attributi* consente di specificare informazioni di routing assegnando ai controller e alle azioni attributi che definiscono le route dell'applicazione. Ciò significa che le definizioni delle route vengono posizionate accanto al controller e all'azione a cui sono associate.

```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
      ...
    }
}
```

### <a name="model-binding"></a>Associazione di modelli

L'[associazione di modelli](models/model-binding.md) di ASP.NET Core MVC converte i dati delle richieste client, ad esempio valori del modulo, dati della route, parametri della stringa di query, intestazioni HTTP, in oggetti che il controller è in grado di gestire. Di conseguenza, non è necessario che la logica del controller risolva i dati della richiesta in ingresso poiché li avrà semplicemente come parametri associati ai propri metodi di azione.

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null) { ... }
```

### <a name="model-validation"></a>Convalida modello

ASP.NET Core MVC supporta la [convalida](models/validation.md) assegnando all'oggetto modello attributi di convalida di annotazione dei dati. Gli attributi di convalida vengono controllati sul lato client prima che i valori siano inviati al server, nonché nel server prima che sia chiamata l'azione del controller.

```csharp
using System.ComponentModel.DataAnnotations;
public class LoginViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Display(Name = "Remember me?")]
    public bool RememberMe { get; set; }
}
```

Un'azione del controller:

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
{
    if (ModelState.IsValid)
    {
      // work with the model
    }
    // At this point, something failed, redisplay form
    return View(model);
}
```

Il framework gestisce la convalida dei dati della richiesta nel client e nel server. La logica di convalida specificata nei tipi di modello viene aggiunta alle visualizzazioni sottoposte a rendering come annotazioni discrete e viene applicata al browser con [jQuery Validation](https://jqueryvalidation.org/).

### <a name="dependency-injection"></a>Inserimento di dipendenze

ASP.NET Core include il supporto predefinito per l'[inserimento di dipendenze](../fundamentals/dependency-injection.md). In ASP.NET Core MVC i [controller](controllers/dependency-injection.md) possono richiedere i servizi necessari attraverso i propri costruttori. Ciò consente loro di seguire il [principio delle dipendenze esplicite](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).

L'app può inoltre usare l'[inserimento di dipendenze nei file di visualizzazione](views/dependency-injection.md) tramite la direttiva `@inject`:

```cshtml
@inject SomeService ServiceName

<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ServiceName.GetTitle</title>
</head>
<body>
    <h1>@ServiceName.GetTitle</h1>
</body>
</html>
```

### <a name="filters"></a>Filtri

I [filtri](controllers/filters.md) consentono agli sviluppatori di incapsulare questioni trasversali quali la gestione delle eccezioni o l'autorizzazione. I filtri consentono l'esecuzione di logica pre-elaborazione e post-elaborazione personalizzata per i metodi di azione e possono essere configurati per l'esecuzione in determinate fasi della pipeline di esecuzione per una richiesta specifica. È possibile applicare i filtri ai controller o alle azioni come attributi o eseguirli a livello globale. Il framework include diversi filtri, ad esempio, `Authorize`. `[Authorize]` è l'attributo usato per creare filtri di autorizzazione MVC.

```csharp
[Authorize]
public class AccountController : Controller
```

### <a name="areas"></a>Aree

Le [aree](controllers/areas.md) consentono di partizionare un'app Web ASP.NET Core MVC di grandi dimensioni in raggruppamenti funzionali più piccoli. Un'area è una struttura MVC all'interno di un'applicazione. In un progetto MVC i componenti logici come modello, controller e visualizzazione si trovano in cartelle diverse e MVC usa le convenzioni di denominazione per creare la relazione tra questi componenti. Per un'app di grandi dimensioni può risultare utile suddividere l'app in aree di funzionalità di alto livello distinte. Ad esempio, un'app di e-commerce con più business unit, ad esempio checkout, fatturazione e ricerca e così via. Ognuna di queste unità dispone di visualizzazioni, controller e modelli di componenti logici.

### <a name="web-apis"></a>API Web

Oltre a essere una piattaforma ideale per la creazione di siti Web, ASP.NET Core MVC include un ottimo supporto per la creazione di API Web. È possibile creare servizi destinati a un'ampia gamma di client, tra cui browser e dispositivi mobili.

Il framework include il supporto per la negoziazione del contenuto HTTP con il supporto predefinito per la [formattazione dei dati](xref:web-api/advanced/formatting) come JSON o XML. È possibile scrivere [formattatori personalizzati](xref:web-api/advanced/custom-formatters) per aggiungere il supporto per i propri formati.

Usare la generazione di collegamenti per abilitare il supporto per l'ipermedia. È possibile abilitare facilmente il supporto per la [Condivisione di risorse tra le origini (CORS)](https://www.w3.org/TR/cors/) in modo da poter condividere le proprie API Web tra più applicazioni Web.

### <a name="testability"></a>Testabilità

L'uso delle interfacce e dell'inserimento di dipendenze rende il framework adatto al testing unità. Il framework include inoltre funzionalità come TestHost e il provider InMemory per Entity Framework grazie alle quali i [test di integrazione](xref:test/integration-tests) risultano semplici e veloci. Per altre informazioni, vedere [Test della logica dei controller](controllers/testing.md).

### <a name="no-locrazor-view-engine"></a>Razor motore di visualizzazione

[ASP.NET Core visualizzazioni MVC](views/overview.md) usano il [ Razor motore di visualizzazione](views/razor.md) per eseguire il rendering delle visualizzazioni. Razor è un linguaggio di markup modello compatto, espressivo e fluido per la definizione di visualizzazioni con codice C# incorporato. Razor viene utilizzato per generare dinamicamente contenuto Web nel server. È possibile combinare correttamente il codice server con il contenuto e il codice sul lato client.

```cshtml
<ul>
    @for (int i = 0; i < 5; i++) {
        <li>List item @i</li>
    }
</ul>
```

Utilizzando il Razor motore di visualizzazione è possibile definire [layout](views/layout.md), [visualizzazioni parziali](views/partial.md) e sezioni sostituibili.

### <a name="strongly-typed-views"></a>Visualizzazioni fortemente tipizzate

Razor le visualizzazioni in MVC possono essere fortemente tipizzate in base al modello. I controller sono in grado di passare un modello fortemente tipizzato alle visualizzazioni abilitando per queste ultime il controllo del tipo e il supporto IntelliSense.

La visualizzazione seguente, ad esempio, esegue il rendering di un modello di tipo `IEnumerable<Product>`:

```cshtml
@model IEnumerable<Product>
<ul>
    @foreach (Product p in Model)
    {
        <li>@p.Name</li>
    }
</ul>
```

### <a name="tag-helpers"></a>Helper tag

Gli [Helper Tag](views/tag-helpers/intro.md) consentono al codice lato server di partecipare alla creazione e al rendering di elementi HTML nei Razor file. È possibile usare gli helper tag per definire tag personalizzati, ad esempio `<environment>` o per modificare il comportamento di tag esistenti, ad esempio `<label>`. Gli helper tag vengono associati a elementi specifici in base al nome dell'elemento e ai relativi attributi. Offrono i vantaggi del rendering lato server mantenendo al tempo stesso un'esperienza di modifica HTML.

Esistono molti helper tag predefiniti per le attività comuni, ad esempio la creazione di moduli e collegamenti, il caricamento di asset e così via, e altri ancora sono disponibili nei repository GitHub pubblici e come pacchetti NuGet. Gli helper tag vengono creati in C# e hanno come destinazione gli elementi HTML in base al nome di elemento, nome di attributo o tag padre. L'helper tag LinkTagHelper predefinito, ad esempio, può essere usato per creare un collegamento all'azione `Login` di `AccountsController`:

```cshtml
<p>
    Thank you for confirming your email.
    Please <a asp-controller="Account" asp-action="Login">Click here to Log in</a>.
</p>
```

`EnvironmentTagHelper` può essere usato per includere diversi script nelle visualizzazioni (ad esempio, non elaborate o minimizzate) in base all'ambiente di runtime, ad esempio ambiente sviluppo, di gestione temporanea o di produzione:

```cshtml
<environment names="Development">
    <script src="~/lib/jquery/dist/jquery.js"></script>
</environment>
<environment names="Staging,Production">
    <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.1.4.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
</environment>
```

Gli helper tag forniscono un'esperienza di sviluppo intuitiva e un ambiente IntelliSense avanzato per la creazione di HTML e Razor markup. La maggior parte degli helper tag predefiniti hanno come destinazione elementi HTML esistenti e forniscono attributi sul lato server per l'elemento.

### <a name="view-components"></a>Componenti di visualizzazione

I [componenti di visualizzazione](views/view-components.md) consentono di creare pacchetti di logica di rendering e di riusarla nell'applicazione. Sono simili alle [visualizzazioni parziali](views/partial.md), ma con la logica associata.

## <a name="compatibility-version"></a>Versione di compatibilità

Il metodo <xref:Microsoft.Extensions.DependencyInjection.MvcCoreMvcBuilderExtensions.SetCompatibilityVersion*> consente a un'app di acconsentire o rifiutare esplicitamente modifiche potenzialmente importanti del comportamento introdotte in ASP.NET Core MVC 2.1 o versioni successive.

Per altre informazioni, vedere <xref:mvc/compatibility-version>.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Tested. AspNetCore. Mvc-libreria di test Fluent per ASP.NET Core MVC](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc): libreria di unit test fortemente tipizzata, che fornisce un'interfaccia Fluent per il test di app per le API Web e MVC. (*Non mantenuto o supportato da Microsoft).*
* <xref:blazor/components/prerendering-and-integration>
