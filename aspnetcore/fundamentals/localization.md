---
title: Globalizzazione e localizzazione in ASP.NET Core
author: rick-anderson
description: Informazioni su come ASP.NET Core offre servizi e middleware per la localizzazione di contenuti in diverse lingue e culture.
ms.author: riande
ms.date: 11/30/2019
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
uid: fundamentals/localization
ms.openlocfilehash: 8212025bbf778334045b8fbe7a5dae312524b426
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588022"
---
# <a name="globalization-and-localization-in-aspnet-core"></a>Globalizzazione e localizzazione in ASP.NET Core

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

[Rick Anderson](https://twitter.com/RickAndMSFT), [Damien Bowden](https://twitter.com/damien_bod), [Bart Calixto](https://twitter.com/bartmax), [Nadeem Afana](https://afana.me/) e [Hisham Bin Ateya](https://twitter.com/hishambinateya)

Un sito Web multilingue consente al sito di raggiungere un pubblico più ampio. AP.NET Core offre servizi e middleware per la localizzazione in diverse lingue e culture.

L'internazionalizzazione implica la [globalizzazione](/dotnet/api/system.globalization) e la [localizzazione](/dotnet/standard/globalization-localization/localization). La globalizzazione è il processo di progettazione di app che supportano impostazioni cultura diverse. La globalizzazione aggiunge supporto per l'input, la visualizzazione e l'output di una specifica serie di script della lingua legati a determinate aree geografiche.

La localizzazione è il processo di adattamento di un'app globalizzata, già elaborata per la localizzazione, a particolari impostazioni cultura e impostazioni locali. Per altre informazioni, vedere **Termini relativi alla globalizzazione e alla localizzazione** nella parte finale di questo documento.

La localizzazione dell'app comporta quanto segue:

1. Rendere localizzabile il contenuto dell'app
1. Fornire le risorse localizzate per le lingue e le impostazioni cultura supportate
1. Implementare una strategia per la selezione della lingua o delle impostazioni cultura per ogni richiesta

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/localization/sample/3.x/Localization) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="make-the-apps-content-localizable"></a>Rendere localizzabile il contenuto dell'app

<xref:Microsoft.Extensions.Localization.IStringLocalizer> e <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> sono stati progettati per migliorare la produttività durante lo sviluppo di app localizzate. `IStringLocalizer` USA <xref:System.Resources.ResourceManager> e <xref:System.Resources.ResourceReader> per fornire risorse specifiche delle impostazioni cultura in fase di esecuzione. L'interfaccia ha un indicizzatore e un oggetto `IEnumerable` per la restituzione di stringhe localizzate. `IStringLocalizer` non richiede l'archiviazione delle stringhe di lingua predefinite in un file di risorse. È possibile sviluppare un'app per la localizzazione senza creare file di risorse nelle prime fasi di sviluppo. Il codice seguente illustra come eseguire il wrapping della stringa "About Title" per la localizzazione.

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

Nel codice precedente, l' `IStringLocalizer<T>` implementazione deriva dall' [inserimento delle dipendenze](dependency-injection.md). Se non viene trovato il valore localizzato di "About Title", viene restituita la chiave dell'indicizzatore, ovvero la stringa "About Title". È possibile lasciare le stringhe letterali della lingua predefinita nell'app ed eseguirne il wrapping nel localizzatore per potersi concentrare sullo sviluppo dell'app. Sviluppare l'app con la lingua predefinita e prepararla per la fase di localizzazione senza prima creare un file di risorse predefinito. In alternativa, è possibile usare l'approccio tradizionale e fornire una chiave per recuperare la stringa della lingua predefinita. Per molti sviluppatori il nuovo flusso di lavoro che prevede di non avere un file con estensione *resx* della lingua predefinita e di eseguire semplicemente il wrapping dei valori letterali di stringa consente di ridurre il sovraccarico della localizzazione di un'app. Altri sviluppatori preferiscono il flusso di lavoro tradizionale poiché semplifica l'uso di valori letterali di stringa più lunghi e l'aggiornamento delle stringhe localizzate.

Usare l'implementazione `IHtmlLocalizer<T>` per le risorse che contengono HTML. L'HTML di `IHtmlLocalizer` codifica gli argomenti formattati nella stringa di risorsa, ma non codifica in HTML la stringa di risorsa. Nell'esempio evidenziato di seguito, solo il valore del parametro `name` è codificato in HTML.

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> In genere, solo localizzare testo, non HTML.

Al livello più basso, è possibile ottenere `IStringLocalizerFactory` dall'[inserimento delle dipendenze](dependency-injection.md):

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

Il codice precedente illustra ognuno dei due metodi di creazione della factory.

È possibile suddividere le stringhe localizzate per controller o area oppure usare un unico contenitore. Nell'app di esempio, viene usata una classe fittizia denominata `SharedResource` per le risorse condivise.

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

Alcuni sviluppatori usano la classe `Startup` per contenere le stringhe globali o condivise. Nell'esempio seguente vengono usati i localizzatori `InfoController` e `SharedResource`:

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>Localizzazione delle visualizzazioni

Il servizio `IViewLocalizer` fornisce le stringhe localizzate per una [visualizzazione](xref:mvc/views/overview). La classe `ViewLocalizer` implementa questa interfaccia e individua la posizione della risorsa dal percorso del file della visualizzazione. Il codice seguente illustra come usare l'implementazione predefinita di `IViewLocalizer`:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

L'implementazione predefinita di `IViewLocalizer` individua il file di risorse in base al nome file della visualizzazione. Non è disponibile alcuna opzione per l'uso di un file di risorse condivise globali. `ViewLocalizer` implementa il localizzatore usando `IHtmlLocalizer` , quindi Razor non codifica la stringa localizzata in HTML. È possibile parametrizzare le stringhe di risorsa e `IViewLocalizer` codificherà in HTML i parametri ma non la stringa di risorsa. Si consideri il Razor markup seguente:

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

Un file di risorse francese può contenere quanto segue:

| Chiave | Valore |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

La visualizzazione di cui è stato eseguito il rendering conterrà il markup HTML del file di risorse.

> [!NOTE]
> In genere, solo localizzare testo, non HTML.

Per usare un file di risorse condivise in una visualizzazione, inserire `IHtmlLocalizer<T>`:

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>Localizzazione di DataAnnotations

I messaggi di errore DataAnnotations vengono localizzati con `IStringLocalizer<T>`. Usando l'opzione `ResourcesPath = "Resources"` è possibile memorizzare i messaggi di errore in `RegisterViewModel` in uno dei percorsi seguenti:

* *Risorse/ViewModels. account. RegisterViewModel. fr. resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

In ASP.NET Core MVC 1.1.0 e versioni successive, gli attributi non di convalida vengono localizzati. ASP.NET Core MVC 1.0 **non** ricerca le stringhe localizzate per gli attributi non di convalida.

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>Uso di un'unica stringa di risorsa per più classi

Il codice seguente illustra come usare una sola stringa di risorsa per gli attributi di convalida con più classi:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

Nel codice precedente `SharedResource` è la classe corrispondente al file con estensione resx in cui sono memorizzati i messaggi di convalida. Con questo approccio, DataAnnotations userà solo `SharedResource` anziché la risorsa per ogni classe.

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>Fornire le risorse localizzate per le lingue e le impostazioni cultura supportate

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures e SupportedUICultures

ASP.NET Core consente di specificare due valori di impostazioni cultura, <xref:Microsoft.AspNetCore.Builder.RequestLocalizationOptions.SupportedCultures> e <xref:Microsoft.AspNetCore.Builder.RequestLocalizationOptions.SupportedUICultures>. L' <xref:System.Globalization.CultureInfo> oggetto per `SupportedCultures` determina i risultati delle funzioni dipendenti dalle impostazioni cultura, ad esempio la data, l'ora, il numero e la formattazione della valuta. `SupportedCultures` determina anche l'ordinamento del testo, le convenzioni di maiuscole e minuscole e i confronti di stringhe. Per ulteriori informazioni sul modo in cui il server ottiene le impostazioni cultura, vedere <xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=nameWithType> e <xref:System.Globalization.CultureInfo.CurrentUICulture?displayProperty=nameWithType> . `SupportedUICultures`Determina quali stringhe tradotte (da `.resx` file) vengono cercate da <xref:System.Resources.ResourceManager> . Il `ResourceManager` Cerca stringhe specifiche delle impostazioni cultura determinate da `CurrentUICulture` . Ogni thread in .NET include oggetti `CurrentCulture` e `CurrentUICulture`. Il Framework controlla questi valori durante il rendering delle funzioni dipendenti dalle impostazioni cultura. Se le impostazioni cultura del thread corrente sono impostate su `en-US` (inglese, Stati Uniti), `DateTime.Now.ToLongDateString()` viene visualizzato, `Thursday, February 18, 2016` ma se `CurrentCulture` è impostato su `es-ES` (Spagnolo, Spagna), l'output è `jueves, 18 de febrero de 2016` .

## <a name="resource-files"></a>File di risorse

Un file di risorse è un meccanismo utile per la separazione delle stringhe localizzabili dal codice. Le stringhe tradotte per la lingua non predefinita sono isolate nei file di risorse *resx* . È possibile ad esempio che si voglia creare un file di risorse spagnolo denominato *Welcome.es.resx* contenente le stringhe tradotte. "es" è il codice di lingua per lo spagnolo. Per creare questo file di risorse in Visual Studio:

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella che conterrà il file di risorse > **Aggiungi** > **Nuovo elemento**.

   ![Menu contestuale annidato: in Esplora soluzioni viene visualizzato un menu contestuale per le risorse. Un secondo menu contestuale viene visualizzato per Aggiungi con il comando Nuovo elemento evidenziato.](localization/_static/newi.png)

1. Nella casella **Cerca modelli installati** immettere "risorse" e assegnare un nome al file.

   ![Finestra di dialogo Aggiungi nuovo elemento](localization/_static/res.png)

1. Immettere il valore della chiave (stringa nativa) nella colonna **Nome** e la stringa tradotta nella colonna **Valore**.

   ![File Welcome.es.resx (file di risorse Welcome per lo spagnolo) con la parola Hello nella colonna Nome e la parola Hola nella colonna Valore](localization/_static/hola.png)

   Visual Studio visualizza il file *Welcome.es.resx*.

   ![Esplora soluzioni con il file di risorse Welcome per lo spagnolo (es)](localization/_static/se.png)

## <a name="resource-file-naming"></a>Denominazione dei file di risorse

Le risorse sono denominate con il nome completo del tipo della relativa classe meno il nome dell'assembly. Ad esempio, una risorsa francese di un progetto il cui assembly principale è `LocalizationWebsite.Web.dll` per la classe `LocalizationWebsite.Web.Startup` viene denominata *Startup.fr.resx*. Una risorsa per la classe `LocalizationWebsite.Web.Controllers.HomeController` viene denominata *Controllers.HomeController.fr.resx*. Se lo spazio dei nomi della classe di destinazione non corrisponde al nome dell'assembly è necessario specificare il nome completo del tipo. Ad esempio, nel progetto di esempio una risorsa per il tipo `ExtraNamespace.Tools` verrebbe denominata *ExtraNamespace.Tools.fr.resx*.

Poiché nel progetto di esempio il metodo `ConfigureServices` imposta `ResourcesPath` su "Resources", il percorso relativo del progetto per il file di risorse francese del controller principale è *Resources/Controllers.HomeController.fr.resx*. In alternativa, è possibile usare le cartelle per organizzare i file di risorse. Per il controller principale, il percorso sarà *Resources/Controllers/HomeController.fr.resx*. Se non si usa l'opzione `ResourcesPath`, il file con estensione *resx* viene memorizzato nella directory di base del progetto. Il file di risorse per `HomeController` sarà denominato *Controllers.HomeController.fr.resx*. La scelta di usare la convenzione di denominazione con il punto o il percorso dipende da come si vuole organizzare i file di risorse.

| Nome risorsa | Denominazione con il punto o il percorso |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | Punto  |
| Resources/Controllers/HomeController.fr.resx  | Percorso |

I file di risorse che usano `@inject IViewLocalizer` nelle Razor viste seguono un modello simile. Il file di risorse per una visualizzazione può essere denominato usando la denominazione con il punto o con il percorso. Razor Visualizza file di risorse riproduce il percorso del file di visualizzazione associato. Se `ResourcesPath` viene impostato su "Resources", il file di risorse francese associato alla visualizzazione *Views/Home/About.cshtml* sarà uno dei seguenti:

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

Se non si usa l'opzione `ResourcesPath`, il file con estensione *resx* per una visualizzazione viene inserito nella stessa cartella della visualizzazione.

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

L' <xref:Microsoft.Extensions.Localization.RootNamespaceAttribute> attributo fornisce lo spazio dei nomi radice di un assembly quando lo spazio dei nomi radice di un assembly è diverso dal nome dell'assembly. 

> [!WARNING]
> Questo problema può verificarsi quando il nome di un progetto non è un identificatore .NET valido. Ad esempio `my-project-name.csproj` , utilizzerà lo spazio dei nomi radice `my_project_name` e il nome dell'assembly `my-project-name` che genera questo errore. 

Se lo spazio dei nomi radice di un assembly è diverso dal nome dell'assembly:

* Per impostazione predefinita, la localizzazione non funziona.
* Le risorse vengono cercate all'interno dell'assembly in un modo che impedisce la localizzazione. `RootNamespace` è un valore necessario in fase di compilazione che non è disponibile per il processo in esecuzione. 

Se l'attributo `RootNamespace` è diverso da `AssemblyName`, includere quanto segue in *AssemblyInfo.cs*, sostituendo i valori dei parametri con quelli effettivi:

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

Il codice precedente consente di risolvere correttamente i file resx.

## <a name="culture-fallback-behavior"></a>Comportamento di fallback delle impostazioni cultura

Durante la ricerca di una risorsa, la localizzazione esegue un "fallback delle impostazioni cultura". A partire dalle impostazioni cultura richieste, se queste non vengono trovate, il codice adotta le impostazioni cultura padre di quelle richieste. Tenere presente che la proprietà [CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) rappresenta le impostazioni cultura padre. Ciò comporta in genere (ma non sempre) la rimozione della notazione nazionale dal codice ISO. Ad esempio la versione della lingua spagnola parlata in Messico è "es-MX". L'elemento padre è "es", ovvero spagnolo non specifico per nessun paese.

Si supponga che il sito riceva una richiesta di una risorsa "Welcome" che usa le impostazioni cultura "fr-CA". Il sistema di localizzazione cerca le risorse seguenti nell'ordine elencato e seleziona la prima corrispondenza:

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx* (se `NeutralResourcesLanguage` è "fr-CA")

Ad esempio, se si rimuove l'indicatore delle impostazioni cultura ".fr" e le impostazioni cultura sono impostate sul francese, viene letto il file di risorse predefinito e vengono localizzate le stringhe. Lo strumento di gestione delle risorse imposta una risorsa predefinita o di fallback per i casi in cui nessun elemento soddisfa le impostazioni cultura richieste. Per restituire solo la chiave in caso di mancanza di una risorsa per le impostazioni cultura richieste è necessario che non sia specificato un file di risorse predefinito.

### <a name="generate-resource-files-with-visual-studio"></a>Generare file di risorse con Visual Studio

Se si crea un file di risorse in Visual Studio senza le impostazioni cultura nel nome del file (ad esempio, *Welcome.resx*), Visual Studio crea una classe C# con una proprietà per ogni stringa. In genere questo non è il risultato desiderato quando si usa ASP.NET Core. Solitamente non è presente un file di risorse con estensione *resx* predefinito (un file con estensione *resx* senza nome delle impostazioni cultura). Si consiglia di creare il file con estensione *resx* con un nome delle impostazioni cultura, ad esempio *Welcome.fr.resx*. Quando si crea un file con estensione *resx* con un nome delle impostazioni cultura, Visual Studio non genera il file di classe.

### <a name="add-other-cultures"></a>Aggiungere altre impostazioni cultura

Ogni combinazione di lingua e impostazioni cultura (diversa dalla lingua predefinita) richiede un file di risorse univoco. Creare file di risorse per impostazioni cultura e locali diverse creando nuovi file di risorse in cui i codici di lingua ISO sono inclusi nel nome file, ad esempio **en-us**, **fr-ca** e **en-gb**. I codici ISO vengono inseriti tra il nome file e l'estensione *resx* come in *Welcome.es-MX.resx* (spagnolo/Messico).

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>Implementare una strategia per la selezione della lingua o delle impostazioni cultura per ogni richiesta

### <a name="configure-localization"></a>Configurare la localizzazione

La localizzazione è configurata nel metodo `Startup.ConfigureServices`:

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` aggiunge i servizi di localizzazione al contenitore dei servizi. Il codice precedente imposta anche il percorso delle risorse su "Resources".

* `AddViewLocalization` aggiunge il supporto per i file di visualizzazione localizzati. Nell'esempio la localizzazione delle visualizzazioni è basata sul suffisso dei file di visualizzazione. Ad esempio "fr" nel file *Index.fr.cshtml*.

* `AddDataAnnotationsLocalization` aggiunge il supporto per `DataAnnotations` i messaggi di convalida localizzati tramite `IStringLocalizer` astrazioni.

### <a name="localization-middleware"></a>Middleware di localizzazione

Le impostazioni cultura correnti in un richiesta sono impostate nel [middleware](xref:fundamentals/middleware/index) di localizzazione. Il middleware di localizzazione viene abilitato nel metodo `Startup.Configure`. Il middleware di localizzazione deve essere configurato prima di qualsiasi middleware che potrebbe controllare le impostazioni cultura della richiesta (ad esempio, `app.UseMvcWithDefaultRoute()`).

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` inizializza un oggetto `RequestLocalizationOptions`. In ogni richiesta l'elenco di `RequestCultureProvider` in `RequestLocalizationOptions` è enumerato e viene usato il primo provider in grado di determinare le impostazioni cultura della richiesta. I provider predefiniti provengono dalla classe `RequestLocalizationOptions`:

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

L'elenco predefinito passa dal più specifico al meno specifico. Di seguito in questo articolo viene descritto come modificare l'ordine e aggiungere un provider delle impostazioni cultura personalizzato. Se nessuno dei provider è in grado di determinare le impostazioni cultura della richiesta, viene usato `DefaultRequestCulture`.

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

Alcune app utilizzeranno una stringa di query per impostare <https://docs.microsoft.com/dotnet/api/system.globalization.cultureinfo?view=netcore-3.1> . Per le app che usano l' cookie approccio dell'intestazione o Accept-Language, l'aggiunta di una stringa di query all'URL è utile per il debug e il test del codice. Per impostazione predefinita, `QueryStringRequestCultureProvider` viene registrato come primo provider di localizzazione nell'elenco `RequestCultureProvider`. Passare i parametri della stringa di query `culture` e `ui-culture`. L'esempio seguente specifica le impostazioni cultura specifiche (lingua e area) per spagnolo/Messico:

   `http://localhost:5000/?culture=es-MX&ui-culture=es-MX`

Se si passa uno solo dei due parametri (`culture` o `ui-culture`), il provider della stringa di query imposterà entrambi i valori usando il parametro passato. Ad esempio, se si specificano solo le impostazioni cultura verranno impostati entrambi i parametri `Culture` e `UICulture`:

```
http://localhost:5000/?culture=es-MX
```

### <a name="cookierequestcultureprovider"></a>CookieRequestCultureProvider

Le app di produzione forniranno spesso un meccanismo per impostare le impostazioni cultura con le impostazioni cultura ASP.NET Core cookie . Usare il `MakeCookieValue` metodo per creare un oggetto cookie .

`CookieRequestCultureProvider` `DefaultCookieName` Restituisce il nome predefinito cookie utilizzato per tenere traccia delle informazioni sulle impostazioni cultura preferite dell'utente. Il cookie nome predefinito è `.AspNetCore.Culture` .

Il cookie formato è `c=%LANGCODE%|uic=%LANGCODE%` , dove `c` è `Culture` e `uic` è `UICulture` , ad esempio:

```
c=en-UK|uic=en-US
```

Se si specificano solo le impostazioni cultura o le impostazioni cultura dell'interfaccia utente, le impostazioni specificate verranno usate per entrambi i tipi di impostazioni.

### <a name="the-accept-language-http-header"></a>Intestazione HTTP Accept-Language

L'[intestazione Accept-Language](https://www.w3.org/International/questions/qa-accept-lang-locales) può essere impostata nella maggior parte dei browser ed è stata originariamente progettata per specificare la lingua dell'utente. Questa impostazione indica la lingua impostata per l'invio da parte del browser o che il browser ha ereditato dal sistema operativo sottostante. L'intestazione HTTP Accept-Language di una richiesta del browser non è un metodo infallibile per individuare la lingua preferita dell'utente (vedere [Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php) (Impostazione delle preferenze di lingua in un browser)). Un'app di produzione deve consentire all'utente di personalizzare le impostazioni cultura.

### <a name="set-the-accept-language-http-header-in-ie"></a>Impostare l'intestazione HTTP Accept-Language in IE

1. Dall'icona a forma di ingranaggio, toccare **Opzioni Internet**.

1. Toccare **Lingue**.

   ![Opzioni Internet](localization/_static/lang.png)

1. Toccare **Imposta preferenze lingua**.

1. Toccare **Aggiungi una lingua**.

1. Aggiungere la lingua.

1. Toccare la lingua e quindi toccare **Sposta su**.

### <a name="use-a-custom-provider"></a>Usare un provider personalizzato

Si supponga di voler consentire agli utenti di memorizzare lingua e impostazioni cultura nei database. È possibile creare un provider per la ricerca di questi valori per l'utente. Il codice seguente illustra come aggiungere un provider personalizzato:

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

Usare `RequestLocalizationOptions` per aggiungere o rimuovere i provider di localizzazione.

### <a name="set-the-culture-programmatically"></a>Specificare le impostazioni cultura a livello di codice

Questo progetto di esempio **Localization.StarterWeb** in [GitHub](https://github.com/aspnet/entropy) contiene l'interfaccia utente per impostare `Culture`. Il file *Views/Shared/_SelectLanguagePartial.cshtml* consente di selezionare le impostazioni cultura dall'elenco delle impostazioni cultura supportate:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

Il file *Views/Shared/_SelectLanguagePartial.cshtml* viene aggiunto alla sezione `footer` del file di layout in modo che sia disponibile in tutte le visualizzazioni:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

Il `SetLanguage` metodo imposta le impostazioni cultura cookie .

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

Non è possibile collegare *_SelectLanguagePartial.cshtml* al codice di esempio per questo progetto. Il progetto **Localization. StarterWeb** in [GitHub](https://github.com/aspnet/entropy) dispone di codice per il flusso `RequestLocalizationOptions` di a un oggetto Razor parziale tramite il contenitore di [inserimento delle dipendenze](dependency-injection.md) .

## <a name="model-binding-route-data-and-query-strings"></a>Associazione di modelli dati della route e stringhe di query

Vedere [comportamento di globalizzazione delle stringhe di query e dati della route di associazione di modelli](xref:mvc/models/model-binding#glob).

## <a name="globalization-and-localization-terms"></a>Termini relativi alla globalizzazione e alla localizzazione

Il processo di localizzazione dell'app richiede anche una conoscenza di base dei set di caratteri comunemente usati nello sviluppo del software moderno e dei problemi associati a essi. Sebbene tutti i computer memorizzano il testo come numeri (codici), sistemi diversi memorizzano lo stesso testo usando numeri diversi. Il processo di localizzazione si riferisce alla traduzione dell'interfaccia utente dell'app per impostazioni cultura o locali specifiche.

La [localizzabilità](/dotnet/standard/globalization-localization/localizability-review) è un processo intermedio che verifica che un'app globalizzata sia pronta per la localizzazione.

Il formato [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) per il nome delle impostazioni cultura è `<languagecode2>-<country/regioncode2>`, dove `<languagecode2>` è il codice della lingua e `<country/regioncode2>` è il codice della cultura secondaria. Ad esempio, `es-CL` per spagnolo (Cile), `en-US` per inglese (Stati Uniti) e `en-AU` per inglese (Australia). [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) è la combinazione di un codice di cultura in lettere minuscole di due lettere ISO 639 associato a una lingua e un codice di cultura secondaria in lettere maiuscole di due lettere ISO 3166 associato a un paese o un'area. Vedere <https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>.

L'internazionalizzazione è spesso abbreviata con "I18N". L'abbreviazione include la prima e l'ultima lettera e il numero di lettere che intercorrono tra di loro nel termine inglese "Internationalization". 18 è il numero di lettere tra la prima "I" e l'ultima "N". Lo stesso vale per globalizzazione (G11N) e localizzazione (L10N).

Termini:

* Globalizzazione (G11N): processo che consente a un'app di supportare lingue e aree diverse.
* Localizzazione (L10N): processo di personalizzazione di un'app per una determinata lingua e area.
* Internazionalizzazione (I18N): descrive la globalizzazione e la localizzazione.
* Impostazioni cultura: si tratta di una lingua e, facoltativamente, di un'area.
* Impostazioni cultura neutre: cultura con una lingua specificata ma senza area. (ad esempio "en", "es")
* Impostazioni cultura specifiche: cultura con una lingua e un'area specificate. (ad esempio "en-US", "en-GB", "es-CL")
* Impostazioni padre: impostazioni cultura neutre con impostazioni cultura specifiche. (ad esempio, "en" rappresenta le impostazioni cultura padre di "en-US" e "en-GB")
* Impostazioni locali: le impostazioni locali corrispondono alle impostazioni cultura.

[!INCLUDE[](~/includes/localization/currency.md)]

[!INCLUDE[](~/includes/localization/unsupported-culture-log-level.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* [Progetto Localization.StarterWeb](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb) usato nell'articolo.
* [Globalizzazione e localizzazione di applicazioni .NET](/dotnet/standard/globalization-localization/index)
* [Risorse nei file con estensione resx](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft Multilingual App Toolkit](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [Localizzazione e generics](http://hishambinateya.com/localization-and-generics)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[Rick Anderson](https://twitter.com/RickAndMSFT), [Damien Bowden](https://twitter.com/damien_bod), [Bart Calixto](https://twitter.com/bartmax), [Nadeem Afana](https://afana.me/) e [Hisham Bin Ateya](https://twitter.com/hishambinateya)

Un sito Web multilingue consente al sito di raggiungere un pubblico più ampio. AP.NET Core offre servizi e middleware per la localizzazione in diverse lingue e culture.

L'internazionalizzazione implica la [globalizzazione](/dotnet/api/system.globalization) e la [localizzazione](/dotnet/standard/globalization-localization/localization). La globalizzazione è il processo di progettazione di app che supportano impostazioni cultura diverse. La globalizzazione aggiunge supporto per l'input, la visualizzazione e l'output di una specifica serie di script della lingua legati a determinate aree geografiche.

La localizzazione è il processo di adattamento di un'app globalizzata, già elaborata per la localizzazione, a particolari impostazioni cultura e impostazioni locali. Per altre informazioni, vedere **Termini relativi alla globalizzazione e alla localizzazione** nella parte finale di questo documento.

La localizzazione dell'app comporta quanto segue:

1. Rendere localizzabile il contenuto dell'app
1. Fornire le risorse localizzate per le lingue e le impostazioni cultura supportate
1. Implementare una strategia per la selezione della lingua o delle impostazioni cultura per ogni richiesta

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/localization/sample/Localization) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="make-the-apps-content-localizable"></a>Rendere localizzabile il contenuto dell'app

<xref:Microsoft.Extensions.Localization.IStringLocalizer> e <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> sono stati progettati per migliorare la produttività durante lo sviluppo di app localizzate. `IStringLocalizer` USA <xref:System.Resources.ResourceManager> e <xref:System.Resources.ResourceReader> per fornire risorse specifiche delle impostazioni cultura in fase di esecuzione. L'interfaccia ha un indicizzatore e un oggetto `IEnumerable` per la restituzione di stringhe localizzate. `IStringLocalizer` non richiede l'archiviazione delle stringhe di lingua predefinite in un file di risorse. È possibile sviluppare un'app per la localizzazione senza creare file di risorse nelle prime fasi di sviluppo. Il codice seguente illustra come eseguire il wrapping della stringa "About Title" per la localizzazione.

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

Nel codice precedente, l' `IStringLocalizer<T>` implementazione deriva dall' [inserimento delle dipendenze](dependency-injection.md). Se non viene trovato il valore localizzato di "About Title", viene restituita la chiave dell'indicizzatore, ovvero la stringa "About Title". È possibile lasciare le stringhe letterali della lingua predefinita nell'app ed eseguirne il wrapping nel localizzatore per potersi concentrare sullo sviluppo dell'app. Sviluppare l'app con la lingua predefinita e prepararla per la fase di localizzazione senza prima creare un file di risorse predefinito. In alternativa, è possibile usare l'approccio tradizionale e fornire una chiave per recuperare la stringa della lingua predefinita. Per molti sviluppatori il nuovo flusso di lavoro che prevede di non avere un file con estensione *resx* della lingua predefinita e di eseguire semplicemente il wrapping dei valori letterali di stringa consente di ridurre il sovraccarico della localizzazione di un'app. Altri sviluppatori preferiscono il flusso di lavoro tradizionale poiché semplifica l'uso di valori letterali di stringa più lunghi e l'aggiornamento delle stringhe localizzate.

Usare l'implementazione `IHtmlLocalizer<T>` per le risorse che contengono HTML. L'HTML di `IHtmlLocalizer` codifica gli argomenti formattati nella stringa di risorsa, ma non codifica in HTML la stringa di risorsa. Nell'esempio evidenziato di seguito, solo il valore del parametro `name` è codificato in HTML.

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> In genere, solo localizzare testo, non HTML.

Al livello più basso, è possibile ottenere `IStringLocalizerFactory` dall'[inserimento delle dipendenze](dependency-injection.md):

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

Il codice precedente illustra ognuno dei due metodi di creazione della factory.

È possibile suddividere le stringhe localizzate per controller o area oppure usare un unico contenitore. Nell'app di esempio, viene usata una classe fittizia denominata `SharedResource` per le risorse condivise.

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

Alcuni sviluppatori usano la classe `Startup` per contenere le stringhe globali o condivise. Nell'esempio seguente vengono usati i localizzatori `InfoController` e `SharedResource`:

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>Localizzazione delle visualizzazioni

Il servizio `IViewLocalizer` fornisce le stringhe localizzate per una [visualizzazione](xref:mvc/views/overview). La classe `ViewLocalizer` implementa questa interfaccia e individua la posizione della risorsa dal percorso del file della visualizzazione. Il codice seguente illustra come usare l'implementazione predefinita di `IViewLocalizer`:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

L'implementazione predefinita di `IViewLocalizer` individua il file di risorse in base al nome file della visualizzazione. Non è disponibile alcuna opzione per l'uso di un file di risorse condivise globali. `ViewLocalizer` implementa il localizzatore usando `IHtmlLocalizer` , quindi Razor non codifica la stringa localizzata in HTML. È possibile parametrizzare le stringhe di risorsa e `IViewLocalizer` codificherà in HTML i parametri ma non la stringa di risorsa. Si consideri il Razor markup seguente:

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

Un file di risorse francese può contenere quanto segue:

| Chiave | Valore |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

La visualizzazione di cui è stato eseguito il rendering conterrà il markup HTML del file di risorse.

> [!NOTE]
> In genere, solo localizzare testo, non HTML.

Per usare un file di risorse condivise in una visualizzazione, inserire `IHtmlLocalizer<T>`:

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>Localizzazione di DataAnnotations

I messaggi di errore DataAnnotations vengono localizzati con `IStringLocalizer<T>`. Usando l'opzione `ResourcesPath = "Resources"` è possibile memorizzare i messaggi di errore in `RegisterViewModel` in uno dei percorsi seguenti:

* *Risorse/ViewModels. account. RegisterViewModel. fr. resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

In ASP.NET Core MVC 1.1.0 e versioni successive, gli attributi non di convalida vengono localizzati. ASP.NET Core MVC 1.0 **non** ricerca le stringhe localizzate per gli attributi non di convalida.

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>Uso di un'unica stringa di risorsa per più classi

Il codice seguente illustra come usare una sola stringa di risorsa per gli attributi di convalida con più classi:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

Nel codice precedente `SharedResource` è la classe corrispondente al file con estensione resx in cui sono memorizzati i messaggi di convalida. Con questo approccio, DataAnnotations userà solo `SharedResource` anziché la risorsa per ogni classe.

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>Fornire le risorse localizzate per le lingue e le impostazioni cultura supportate

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures e SupportedUICultures

ASP.NET Core consente di specificare due valori di impostazioni cultura, `SupportedCultures` e `SupportedUICultures`. L'oggetto [CultureInfo](/dotnet/api/system.globalization.cultureinfo) per `SupportedCultures` determina i risultati delle funzioni dipendenti dalle impostazioni cultura, ad esempio date, ore, numeri e formattazione delle valute. `SupportedCultures` determina anche l'ordinamento del testo, le convenzioni di maiuscole e minuscole e i confronti di stringhe. Per altre informazioni su come il server ottiene le impostazioni cultura, vedere [CultureInfo.CurrentCulture](/dotnet/api/system.stringcomparer.currentculture#System_StringComparer_CurrentCulture). `SupportedUICultures`Determina quali stringhe tradotte (dai file *resx* ) vengono cercate da [ResourceManager](/dotnet/api/system.resources.resourcemanager). `ResourceManager` esegue la ricerca nelle stringhe specifiche delle impostazioni cultura determinate da `CurrentUICulture`. Ogni thread in .NET include oggetti `CurrentCulture` e `CurrentUICulture`. ASP.NET Core controlla questi valori quando viene eseguito il rendering delle funzioni dipendenti dalle impostazioni cultura. Ad esempio, se le impostazioni cultura del thread corrente sono impostate su "en-US" (inglese, Stati Uniti), `DateTime.Now.ToLongDateString()` visualizza "Thursday, February 18, 2016". Se invece `CurrentCulture` è impostato su "es-ES" (spagnolo, Spagna) l'output sarà "jueves, 18 de febrero de 2016".

## <a name="resource-files"></a>File di risorse

Un file di risorse è un meccanismo utile per la separazione delle stringhe localizzabili dal codice. Le stringhe tradotte per la lingua non predefinita sono isolate nei file di risorse *resx* . È possibile ad esempio che si voglia creare un file di risorse spagnolo denominato *Welcome.es.resx* contenente le stringhe tradotte. "es" è il codice di lingua per lo spagnolo. Per creare questo file di risorse in Visual Studio:

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella che conterrà il file di risorse > **Aggiungi** > **Nuovo elemento**.

   ![Menu contestuale annidato: in Esplora soluzioni viene visualizzato un menu contestuale per le risorse. Un secondo menu contestuale viene visualizzato per Aggiungi con il comando Nuovo elemento evidenziato.](localization/_static/newi.png)

1. Nella casella **Cerca modelli installati** immettere "risorse" e assegnare un nome al file.

   ![Finestra di dialogo Aggiungi nuovo elemento](localization/_static/res.png)

1. Immettere il valore della chiave (stringa nativa) nella colonna **Nome** e la stringa tradotta nella colonna **Valore**.

   ![File Welcome.es.resx (file di risorse Welcome per lo spagnolo) con la parola Hello nella colonna Nome e la parola Hola nella colonna Valore](localization/_static/hola.png)

   Visual Studio visualizza il file *Welcome.es.resx*.

   ![Esplora soluzioni con il file di risorse Welcome per lo spagnolo (es)](localization/_static/se.png)

## <a name="resource-file-naming"></a>Denominazione dei file di risorse

Le risorse sono denominate con il nome completo del tipo della relativa classe meno il nome dell'assembly. Ad esempio, una risorsa francese di un progetto il cui assembly principale è `LocalizationWebsite.Web.dll` per la classe `LocalizationWebsite.Web.Startup` viene denominata *Startup.fr.resx*. Una risorsa per la classe `LocalizationWebsite.Web.Controllers.HomeController` viene denominata *Controllers.HomeController.fr.resx*. Se lo spazio dei nomi della classe di destinazione non corrisponde al nome dell'assembly è necessario specificare il nome completo del tipo. Ad esempio, nel progetto di esempio una risorsa per il tipo `ExtraNamespace.Tools` verrebbe denominata *ExtraNamespace.Tools.fr.resx*.

Poiché nel progetto di esempio il metodo `ConfigureServices` imposta `ResourcesPath` su "Resources", il percorso relativo del progetto per il file di risorse francese del controller principale è *Resources/Controllers.HomeController.fr.resx*. In alternativa, è possibile usare le cartelle per organizzare i file di risorse. Per il controller principale, il percorso sarà *Resources/Controllers/HomeController.fr.resx*. Se non si usa l'opzione `ResourcesPath`, il file con estensione *resx* viene memorizzato nella directory di base del progetto. Il file di risorse per `HomeController` sarà denominato *Controllers.HomeController.fr.resx*. La scelta di usare la convenzione di denominazione con il punto o il percorso dipende da come si vuole organizzare i file di risorse.

| Nome risorsa | Denominazione con il punto o il percorso |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | Punto  |
| Resources/Controllers/HomeController.fr.resx  | Percorso |

I file di risorse che usano `@inject IViewLocalizer` nelle Razor viste seguono un modello simile. Il file di risorse per una visualizzazione può essere denominato usando la denominazione con il punto o con il percorso. Razor Visualizza file di risorse riproduce il percorso del file di visualizzazione associato. Se `ResourcesPath` viene impostato su "Resources", il file di risorse francese associato alla visualizzazione *Views/Home/About.cshtml* sarà uno dei seguenti:

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

Se non si usa l'opzione `ResourcesPath`, il file con estensione *resx* per una visualizzazione viene inserito nella stessa cartella della visualizzazione.

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

L' <xref:Microsoft.Extensions.Localization.RootNamespaceAttribute> attributo fornisce lo spazio dei nomi radice di un assembly quando lo spazio dei nomi radice di un assembly è diverso dal nome dell'assembly. 

> [!WARNING]
> Questo problema può verificarsi quando il nome di un progetto non è un identificatore .NET valido. Ad esempio `my-project-name.csproj` , utilizzerà lo spazio dei nomi radice `my_project_name` e il nome dell'assembly `my-project-name` che genera questo errore. 

Se lo spazio dei nomi radice di un assembly è diverso dal nome dell'assembly:

* Per impostazione predefinita, la localizzazione non funziona.
* Le risorse vengono cercate all'interno dell'assembly in un modo che impedisce la localizzazione. `RootNamespace` è un valore necessario in fase di compilazione che non è disponibile per il processo in esecuzione. 

Se l'attributo `RootNamespace` è diverso da `AssemblyName`, includere quanto segue in *AssemblyInfo.cs*, sostituendo i valori dei parametri con quelli effettivi:

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

Il codice precedente consente di risolvere correttamente i file resx.

## <a name="culture-fallback-behavior"></a>Comportamento di fallback delle impostazioni cultura

Durante la ricerca di una risorsa, la localizzazione esegue un "fallback delle impostazioni cultura". A partire dalle impostazioni cultura richieste, se queste non vengono trovate, il codice adotta le impostazioni cultura padre di quelle richieste. Tenere presente che la proprietà [CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) rappresenta le impostazioni cultura padre. Ciò comporta in genere (ma non sempre) la rimozione della notazione nazionale dal codice ISO. Ad esempio la versione della lingua spagnola parlata in Messico è "es-MX". L'elemento padre è "es", ovvero spagnolo non specifico per nessun paese.

Si supponga che il sito riceva una richiesta di una risorsa "Welcome" che usa le impostazioni cultura "fr-CA". Il sistema di localizzazione cerca le risorse seguenti nell'ordine elencato e seleziona la prima corrispondenza:

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx* (se `NeutralResourcesLanguage` è "fr-CA")

Ad esempio, se si rimuove l'indicatore delle impostazioni cultura ".fr" e le impostazioni cultura sono impostate sul francese, viene letto il file di risorse predefinito e vengono localizzate le stringhe. Lo strumento di gestione delle risorse imposta una risorsa predefinita o di fallback per i casi in cui nessun elemento soddisfa le impostazioni cultura richieste. Per restituire solo la chiave in caso di mancanza di una risorsa per le impostazioni cultura richieste è necessario che non sia specificato un file di risorse predefinito.

### <a name="generate-resource-files-with-visual-studio"></a>Generare file di risorse con Visual Studio

Se si crea un file di risorse in Visual Studio senza le impostazioni cultura nel nome del file (ad esempio, *Welcome.resx*), Visual Studio crea una classe C# con una proprietà per ogni stringa. In genere questo non è il risultato desiderato quando si usa ASP.NET Core. Solitamente non è presente un file di risorse con estensione *resx* predefinito (un file con estensione *resx* senza nome delle impostazioni cultura). Si consiglia di creare il file con estensione *resx* con un nome delle impostazioni cultura, ad esempio *Welcome.fr.resx*. Quando si crea un file con estensione *resx* con un nome delle impostazioni cultura, Visual Studio non genera il file di classe.

### <a name="add-other-cultures"></a>Aggiungere altre impostazioni cultura

Ogni combinazione di lingua e impostazioni cultura (diversa dalla lingua predefinita) richiede un file di risorse univoco. Creare file di risorse per impostazioni cultura e locali diverse creando nuovi file di risorse in cui i codici di lingua ISO sono inclusi nel nome file, ad esempio **en-us**, **fr-ca** e **en-gb**. I codici ISO vengono inseriti tra il nome file e l'estensione *resx* come in *Welcome.es-MX.resx* (spagnolo/Messico).

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>Implementare una strategia per la selezione della lingua o delle impostazioni cultura per ogni richiesta

### <a name="configure-localization"></a>Configurare la localizzazione

La localizzazione è configurata nel metodo `Startup.ConfigureServices`:

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` aggiunge i servizi di localizzazione al contenitore dei servizi. Il codice precedente imposta anche il percorso delle risorse su "Resources".

* `AddViewLocalization` aggiunge il supporto per i file di visualizzazione localizzati. Nell'esempio la localizzazione delle visualizzazioni è basata sul suffisso dei file di visualizzazione. Ad esempio "fr" nel file *Index.fr.cshtml*.

* `AddDataAnnotationsLocalization` aggiunge il supporto per `DataAnnotations` i messaggi di convalida localizzati tramite `IStringLocalizer` astrazioni.

### <a name="localization-middleware"></a>Middleware di localizzazione

Le impostazioni cultura correnti in un richiesta sono impostate nel [middleware](xref:fundamentals/middleware/index) di localizzazione. Il middleware di localizzazione viene abilitato nel metodo `Startup.Configure`. Il middleware di localizzazione deve essere configurato prima di qualsiasi middleware che potrebbe controllare le impostazioni cultura della richiesta (ad esempio, `app.UseMvcWithDefaultRoute()`).

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` inizializza un oggetto `RequestLocalizationOptions`. In ogni richiesta l'elenco di `RequestCultureProvider` in `RequestLocalizationOptions` è enumerato e viene usato il primo provider in grado di determinare le impostazioni cultura della richiesta. I provider predefiniti provengono dalla classe `RequestLocalizationOptions`:

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

L'elenco predefinito passa dal più specifico al meno specifico. Di seguito in questo articolo viene descritto come modificare l'ordine e aggiungere un provider delle impostazioni cultura personalizzato. Se nessuno dei provider è in grado di determinare le impostazioni cultura della richiesta, viene usato `DefaultRequestCulture`.

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

Alcune app utilizzeranno una stringa di query per impostare <https://docs.microsoft.com/dotnet/api/system.globalization.cultureinfo?view=netcore-3.1> . Per le app che usano l' cookie approccio dell'intestazione o Accept-Language, l'aggiunta di una stringa di query all'URL è utile per il debug e il test del codice. Per impostazione predefinita, `QueryStringRequestCultureProvider` viene registrato come primo provider di localizzazione nell'elenco `RequestCultureProvider`. Passare i parametri della stringa di query `culture` e `ui-culture`. L'esempio seguente specifica le impostazioni cultura specifiche (lingua e area) per spagnolo/Messico:

```
http://localhost:5000/?culture=es-MX&ui-culture=es-MX
```

Se si passa uno solo dei due parametri (`culture` o `ui-culture`), il provider della stringa di query imposterà entrambi i valori usando il parametro passato. Ad esempio, se si specificano solo le impostazioni cultura verranno impostati entrambi i parametri `Culture` e `UICulture`:

```
http://localhost:5000/?culture=es-MX
```

### <a name="cookierequestcultureprovider"></a>CookieRequestCultureProvider

Le app di produzione forniranno spesso un meccanismo per impostare le impostazioni cultura con le impostazioni cultura ASP.NET Core cookie . Usare il `MakeCookieValue` metodo per creare un oggetto cookie .

`CookieRequestCultureProvider` `DefaultCookieName` Restituisce il nome predefinito cookie utilizzato per tenere traccia delle informazioni sulle impostazioni cultura preferite dell'utente. Il cookie nome predefinito è `.AspNetCore.Culture` .

Il cookie formato è `c=%LANGCODE%|uic=%LANGCODE%` , dove `c` è `Culture` e `uic` è `UICulture` , ad esempio:

```
c=en-UK|uic=en-US
```

Se si specificano solo le impostazioni cultura o le impostazioni cultura dell'interfaccia utente, le impostazioni specificate verranno usate per entrambi i tipi di impostazioni.

### <a name="the-accept-language-http-header"></a>Intestazione HTTP Accept-Language

L'[intestazione Accept-Language](https://www.w3.org/International/questions/qa-accept-lang-locales) può essere impostata nella maggior parte dei browser ed è stata originariamente progettata per specificare la lingua dell'utente. Questa impostazione indica la lingua impostata per l'invio da parte del browser o che il browser ha ereditato dal sistema operativo sottostante. L'intestazione HTTP Accept-Language di una richiesta del browser non è un metodo infallibile per individuare la lingua preferita dell'utente (vedere [Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php) (Impostazione delle preferenze di lingua in un browser)). Un'app di produzione deve consentire all'utente di personalizzare le impostazioni cultura.

### <a name="set-the-accept-language-http-header-in-ie"></a>Impostare l'intestazione HTTP Accept-Language in IE

1. Dall'icona a forma di ingranaggio, toccare **Opzioni Internet**.

1. Toccare **Lingue**.

   ![Opzioni Internet](localization/_static/lang.png)

1. Toccare **Imposta preferenze lingua**.

1. Toccare **Aggiungi una lingua**.

1. Aggiungere la lingua.

1. Toccare la lingua e quindi toccare **Sposta su**.

### <a name="use-a-custom-provider"></a>Usare un provider personalizzato

Si supponga di voler consentire agli utenti di memorizzare lingua e impostazioni cultura nei database. È possibile creare un provider per la ricerca di questi valori per l'utente. Il codice seguente illustra come aggiungere un provider personalizzato:

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

Usare `RequestLocalizationOptions` per aggiungere o rimuovere i provider di localizzazione.

### <a name="set-the-culture-programmatically"></a>Specificare le impostazioni cultura a livello di codice

Questo progetto di esempio **Localization.StarterWeb** in [GitHub](https://github.com/aspnet/entropy) contiene l'interfaccia utente per impostare `Culture`. Il file *Views/Shared/_SelectLanguagePartial.cshtml* consente di selezionare le impostazioni cultura dall'elenco delle impostazioni cultura supportate:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

Il file *Views/Shared/_SelectLanguagePartial.cshtml* viene aggiunto alla sezione `footer` del file di layout in modo che sia disponibile in tutte le visualizzazioni:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

Il `SetLanguage` metodo imposta le impostazioni cultura cookie .

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

Non è possibile collegare *_SelectLanguagePartial.cshtml* al codice di esempio per questo progetto. Il progetto **Localization. StarterWeb** in [GitHub](https://github.com/aspnet/entropy) dispone di codice per il flusso `RequestLocalizationOptions` di a un oggetto Razor parziale tramite il contenitore di [inserimento delle dipendenze](dependency-injection.md) .

## <a name="model-binding-route-data-and-query-strings"></a>Associazione di modelli dati della route e stringhe di query

Vedere [comportamento di globalizzazione delle stringhe di query e dati della route di associazione di modelli](xref:mvc/models/model-binding#glob).

## <a name="globalization-and-localization-terms"></a>Termini relativi alla globalizzazione e alla localizzazione

Il processo di localizzazione dell'app richiede anche una conoscenza di base dei set di caratteri comunemente usati nello sviluppo del software moderno e dei problemi associati a essi. Sebbene tutti i computer memorizzano il testo come numeri (codici), sistemi diversi memorizzano lo stesso testo usando numeri diversi. Il processo di localizzazione si riferisce alla traduzione dell'interfaccia utente dell'app per impostazioni cultura o locali specifiche.

La [localizzabilità](/dotnet/standard/globalization-localization/localizability-review) è un processo intermedio che verifica che un'app globalizzata sia pronta per la localizzazione.

Il formato [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) per il nome delle impostazioni cultura è `<languagecode2>-<country/regioncode2>`, dove `<languagecode2>` è il codice della lingua e `<country/regioncode2>` è il codice della cultura secondaria. Ad esempio, `es-CL` per spagnolo (Cile), `en-US` per inglese (Stati Uniti) e `en-AU` per inglese (Australia). [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) è la combinazione di un codice di cultura in lettere minuscole di due lettere ISO 639 associato a una lingua e un codice di cultura secondaria in lettere maiuscole di due lettere ISO 3166 associato a un paese o un'area. Vedere <https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>.

L'internazionalizzazione è spesso abbreviata con "I18N". L'abbreviazione include la prima e l'ultima lettera e il numero di lettere che intercorrono tra di loro nel termine inglese "Internationalization". 18 è il numero di lettere tra la prima "I" e l'ultima "N". Lo stesso vale per globalizzazione (G11N) e localizzazione (L10N).

Termini:

* Globalizzazione (G11N): processo che consente a un'app di supportare lingue e aree diverse.
* Localizzazione (L10N): processo di personalizzazione di un'app per una determinata lingua e area.
* Internazionalizzazione (I18N): descrive la globalizzazione e la localizzazione.
* Impostazioni cultura: si tratta di una lingua e, facoltativamente, di un'area.
* Impostazioni cultura neutre: cultura con una lingua specificata ma senza area. (ad esempio "en", "es")
* Impostazioni cultura specifiche: cultura con una lingua e un'area specificate. (ad esempio "en-US", "en-GB", "es-CL")
* Impostazioni padre: impostazioni cultura neutre con impostazioni cultura specifiche. (ad esempio, "en" rappresenta le impostazioni cultura padre di "en-US" e "en-GB")
* Impostazioni locali: le impostazioni locali corrispondono alle impostazioni cultura.

[!INCLUDE[](~/includes/localization/currency.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* [Progetto Localization.StarterWeb](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb) usato nell'articolo.
* [Globalizzazione e localizzazione di applicazioni .NET](/dotnet/standard/globalization-localization/index)
* [Risorse nei file con estensione resx](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft Multilingual App Toolkit](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [Localizzazione e generics](http://hishambinateya.com/localization-and-generics)

::: moniker-end

<!-- ASP.NET Core 5.x starts here -->
::: moniker range="> aspnetcore-3.1"

[Rick Anderson](https://twitter.com/RickAndMSFT), [Damien Bowden](https://twitter.com/damien_bod), [Bart Calixto](https://twitter.com/bartmax), [Nadeem Afana](https://afana.me/) e [Hisham Bin Ateya](https://twitter.com/hishambinateya)

Un sito Web multilingue consente al sito di raggiungere un pubblico più ampio. AP.NET Core offre servizi e middleware per la localizzazione in diverse lingue e culture.

L'internazionalizzazione implica la [globalizzazione](/dotnet/api/system.globalization) e la [localizzazione](/dotnet/standard/globalization-localization/localization). La globalizzazione è il processo di progettazione di app che supportano impostazioni cultura diverse. La globalizzazione aggiunge supporto per l'input, la visualizzazione e l'output di una specifica serie di script della lingua legati a determinate aree geografiche.

La localizzazione è il processo di adattamento di un'app globalizzata, già elaborata per la localizzazione, a particolari impostazioni cultura e impostazioni locali. Per altre informazioni, vedere **Termini relativi alla globalizzazione e alla localizzazione** nella parte finale di questo documento.

La localizzazione dell'app comporta quanto segue:

1. Rendere localizzabile il contenuto dell'app
1. Fornire le risorse localizzate per le lingue e le impostazioni cultura supportate
1. Implementare una strategia per la selezione della lingua o delle impostazioni cultura per ogni richiesta

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/localization/sample/2.x/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="make-the-apps-content-localizable"></a>Rendere localizzabile il contenuto dell'app

<xref:Microsoft.Extensions.Localization.IStringLocalizer> e <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> sono stati progettati per migliorare la produttività durante lo sviluppo di app localizzate. `IStringLocalizer` USA <xref:System.Resources.ResourceManager> e <xref:System.Resources.ResourceReader> per fornire risorse specifiche delle impostazioni cultura in fase di esecuzione. L'interfaccia ha un indicizzatore e un oggetto `IEnumerable` per la restituzione di stringhe localizzate. `IStringLocalizer` non richiede l'archiviazione delle stringhe di lingua predefinite in un file di risorse. È possibile sviluppare un'app per la localizzazione senza creare file di risorse nelle prime fasi di sviluppo. Il codice seguente illustra come eseguire il wrapping della stringa "About Title" per la localizzazione.

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

Nel codice precedente, l' `IStringLocalizer<T>` implementazione deriva dall' [inserimento delle dipendenze](dependency-injection.md). Se non viene trovato il valore localizzato di "About Title", viene restituita la chiave dell'indicizzatore, ovvero la stringa "About Title". È possibile lasciare le stringhe letterali della lingua predefinita nell'app ed eseguirne il wrapping nel localizzatore per potersi concentrare sullo sviluppo dell'app. Sviluppare l'app con la lingua predefinita e prepararla per la fase di localizzazione senza prima creare un file di risorse predefinito. In alternativa, è possibile usare l'approccio tradizionale e fornire una chiave per recuperare la stringa della lingua predefinita. Per molti sviluppatori il nuovo flusso di lavoro che prevede di non avere un file con estensione *resx* della lingua predefinita e di eseguire semplicemente il wrapping dei valori letterali di stringa consente di ridurre il sovraccarico della localizzazione di un'app. Altri sviluppatori preferiscono il flusso di lavoro tradizionale poiché semplifica l'uso di valori letterali di stringa più lunghi e l'aggiornamento delle stringhe localizzate.

Usare l'implementazione `IHtmlLocalizer<T>` per le risorse che contengono HTML. L'HTML di `IHtmlLocalizer` codifica gli argomenti formattati nella stringa di risorsa, ma non codifica in HTML la stringa di risorsa. Nell'esempio evidenziato di seguito, solo il valore del parametro `name` è codificato in HTML.

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> In genere, solo localizzare testo, non HTML.

Al livello più basso, è possibile ottenere `IStringLocalizerFactory` dall'[inserimento delle dipendenze](dependency-injection.md):

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

Il codice precedente illustra ognuno dei due metodi di creazione della factory.

È possibile suddividere le stringhe localizzate per controller o area oppure usare un unico contenitore. Nell'app di esempio, viene usata una classe fittizia denominata `SharedResource` per le risorse condivise.

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

Alcuni sviluppatori usano la classe `Startup` per contenere le stringhe globali o condivise. Nell'esempio seguente vengono usati i localizzatori `InfoController` e `SharedResource`:

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>Localizzazione delle visualizzazioni

Il servizio `IViewLocalizer` fornisce le stringhe localizzate per una [visualizzazione](xref:mvc/views/overview). La classe `ViewLocalizer` implementa questa interfaccia e individua la posizione della risorsa dal percorso del file della visualizzazione. Il codice seguente illustra come usare l'implementazione predefinita di `IViewLocalizer`:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

L'implementazione predefinita di `IViewLocalizer` individua il file di risorse in base al nome file della visualizzazione. Non è disponibile alcuna opzione per l'uso di un file di risorse condivise globali. `ViewLocalizer` implementa il localizzatore usando `IHtmlLocalizer` , quindi Razor non codifica la stringa localizzata in HTML. È possibile parametrizzare le stringhe di risorsa e `IViewLocalizer` codificherà in HTML i parametri ma non la stringa di risorsa. Si consideri il Razor markup seguente:

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

Un file di risorse francese può contenere quanto segue:

| Chiave | Valore |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

La visualizzazione di cui è stato eseguito il rendering conterrà il markup HTML del file di risorse.

> [!NOTE]
> In genere, solo localizzare testo, non HTML.

Per usare un file di risorse condivise in una visualizzazione, inserire `IHtmlLocalizer<T>`:

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>Localizzazione di DataAnnotations

I messaggi di errore DataAnnotations vengono localizzati con `IStringLocalizer<T>`. Usando l'opzione `ResourcesPath = "Resources"` è possibile memorizzare i messaggi di errore in `RegisterViewModel` in uno dei percorsi seguenti:

* *Risorse/ViewModels. account. RegisterViewModel. fr. resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

In ASP.NET Core MVC 1.1.0 e versioni successive, gli attributi non di convalida vengono localizzati. ASP.NET Core MVC 1.0 **non** ricerca le stringhe localizzate per gli attributi non di convalida.

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>Uso di un'unica stringa di risorsa per più classi

Il codice seguente illustra come usare una sola stringa di risorsa per gli attributi di convalida con più classi:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

Nel codice precedente `SharedResource` è la classe corrispondente al file con estensione resx in cui sono memorizzati i messaggi di convalida. Con questo approccio, DataAnnotations userà solo `SharedResource` anziché la risorsa per ogni classe.

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>Fornire le risorse localizzate per le lingue e le impostazioni cultura supportate

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures e SupportedUICultures

ASP.NET Core consente di specificare due valori di impostazioni cultura, `SupportedCultures` e `SupportedUICultures`. L'oggetto [CultureInfo](/dotnet/api/system.globalization.cultureinfo) per `SupportedCultures` determina i risultati delle funzioni dipendenti dalle impostazioni cultura, ad esempio date, ore, numeri e formattazione delle valute. `SupportedCultures` determina anche l'ordinamento del testo, le convenzioni di maiuscole e minuscole e i confronti di stringhe. Per altre informazioni su come il server ottiene le impostazioni cultura, vedere [CultureInfo.CurrentCulture](/dotnet/api/system.stringcomparer.currentculture#System_StringComparer_CurrentCulture). `SupportedUICultures`Determina quali stringhe tradotte (dai file *resx* ) vengono cercate da [ResourceManager](/dotnet/api/system.resources.resourcemanager). `ResourceManager` esegue la ricerca nelle stringhe specifiche delle impostazioni cultura determinate da `CurrentUICulture`. Ogni thread in .NET include oggetti `CurrentCulture` e `CurrentUICulture`. ASP.NET Core controlla questi valori quando viene eseguito il rendering delle funzioni dipendenti dalle impostazioni cultura. Ad esempio, se le impostazioni cultura del thread corrente sono impostate su "en-US" (inglese, Stati Uniti), `DateTime.Now.ToLongDateString()` visualizza "Thursday, February 18, 2016". Se invece `CurrentCulture` è impostato su "es-ES" (spagnolo, Spagna) l'output sarà "jueves, 18 de febrero de 2016".

## <a name="resource-files"></a>File di risorse

Un file di risorse è un meccanismo utile per la separazione delle stringhe localizzabili dal codice. Le stringhe tradotte per la lingua non predefinita sono isolate nei file di risorse *resx* . È possibile ad esempio che si voglia creare un file di risorse spagnolo denominato *Welcome.es.resx* contenente le stringhe tradotte. "es" è il codice di lingua per lo spagnolo. Per creare questo file di risorse in Visual Studio:

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella che conterrà il file di risorse > **Aggiungi** > **Nuovo elemento**.

   ![Menu contestuale annidato: in Esplora soluzioni viene visualizzato un menu contestuale per le risorse. Un secondo menu contestuale viene visualizzato per Aggiungi con il comando Nuovo elemento evidenziato.](localization/_static/newi.png)

1. Nella casella **Cerca modelli installati** immettere "risorse" e assegnare un nome al file.

   ![Finestra di dialogo Aggiungi nuovo elemento](localization/_static/res.png)

1. Immettere il valore della chiave (stringa nativa) nella colonna **Nome** e la stringa tradotta nella colonna **Valore**.

   ![File Welcome.es.resx (file di risorse Welcome per lo spagnolo) con la parola Hello nella colonna Nome e la parola Hola nella colonna Valore](localization/_static/hola.png)

   Visual Studio visualizza il file *Welcome.es.resx*.

   ![Esplora soluzioni con il file di risorse Welcome per lo spagnolo (es)](localization/_static/se.png)

## <a name="resource-file-naming"></a>Denominazione dei file di risorse

Le risorse sono denominate con il nome completo del tipo della relativa classe meno il nome dell'assembly. Ad esempio, una risorsa francese di un progetto il cui assembly principale è `LocalizationWebsite.Web.dll` per la classe `LocalizationWebsite.Web.Startup` viene denominata *Startup.fr.resx*. Una risorsa per la classe `LocalizationWebsite.Web.Controllers.HomeController` viene denominata *Controllers.HomeController.fr.resx*. Se lo spazio dei nomi della classe di destinazione non corrisponde al nome dell'assembly è necessario specificare il nome completo del tipo. Ad esempio, nel progetto di esempio una risorsa per il tipo `ExtraNamespace.Tools` verrebbe denominata *ExtraNamespace.Tools.fr.resx*.

Poiché nel progetto di esempio il metodo `ConfigureServices` imposta `ResourcesPath` su "Resources", il percorso relativo del progetto per il file di risorse francese del controller principale è *Resources/Controllers.HomeController.fr.resx*. In alternativa, è possibile usare le cartelle per organizzare i file di risorse. Per il controller principale, il percorso sarà *Resources/Controllers/HomeController.fr.resx*. Se non si usa l'opzione `ResourcesPath`, il file con estensione *resx* viene memorizzato nella directory di base del progetto. Il file di risorse per `HomeController` sarà denominato *Controllers.HomeController.fr.resx*. La scelta di usare la convenzione di denominazione con il punto o il percorso dipende da come si vuole organizzare i file di risorse.

| Nome risorsa | Denominazione con il punto o il percorso |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | Punto  |
| Resources/Controllers/HomeController.fr.resx  | Percorso |

I file di risorse che usano `@inject IViewLocalizer` nelle Razor viste seguono un modello simile. Il file di risorse per una visualizzazione può essere denominato usando la denominazione con il punto o con il percorso. Razor Visualizza file di risorse riproduce il percorso del file di visualizzazione associato. Se `ResourcesPath` viene impostato su "Resources", il file di risorse francese associato alla visualizzazione *Views/Home/About.cshtml* sarà uno dei seguenti:

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

Se non si usa l'opzione `ResourcesPath`, il file con estensione *resx* per una visualizzazione viene inserito nella stessa cartella della visualizzazione.

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

L' <xref:Microsoft.Extensions.Localization.RootNamespaceAttribute> attributo fornisce lo spazio dei nomi radice di un assembly quando lo spazio dei nomi radice di un assembly è diverso dal nome dell'assembly. 

> [!WARNING]
> Questo problema può verificarsi quando il nome di un progetto non è un identificatore .NET valido. Ad esempio `my-project-name.csproj` , utilizzerà lo spazio dei nomi radice `my_project_name` e il nome dell'assembly `my-project-name` che genera questo errore. 

Se lo spazio dei nomi radice di un assembly è diverso dal nome dell'assembly:

* Per impostazione predefinita, la localizzazione non funziona.
* Le risorse vengono cercate all'interno dell'assembly in un modo che impedisce la localizzazione. `RootNamespace` è un valore necessario in fase di compilazione che non è disponibile per il processo in esecuzione. 

Se l'attributo `RootNamespace` è diverso da `AssemblyName`, includere quanto segue in *AssemblyInfo.cs*, sostituendo i valori dei parametri con quelli effettivi:

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

Il codice precedente consente di risolvere correttamente i file resx.

## <a name="culture-fallback-behavior"></a>Comportamento di fallback delle impostazioni cultura

Durante la ricerca di una risorsa, la localizzazione esegue un "fallback delle impostazioni cultura". A partire dalle impostazioni cultura richieste, se queste non vengono trovate, il codice adotta le impostazioni cultura padre di quelle richieste. Tenere presente che la proprietà [CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) rappresenta le impostazioni cultura padre. Ciò comporta in genere (ma non sempre) la rimozione della notazione nazionale dal codice ISO. Ad esempio la versione della lingua spagnola parlata in Messico è "es-MX". L'elemento padre è "es", ovvero spagnolo non specifico per nessun paese.

Si supponga che il sito riceva una richiesta di una risorsa "Welcome" che usa le impostazioni cultura "fr-CA". Il sistema di localizzazione cerca le risorse seguenti nell'ordine elencato e seleziona la prima corrispondenza:

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx* (se `NeutralResourcesLanguage` è "fr-CA")

Ad esempio, se si rimuove l'indicatore delle impostazioni cultura ".fr" e le impostazioni cultura sono impostate sul francese, viene letto il file di risorse predefinito e vengono localizzate le stringhe. Lo strumento di gestione delle risorse imposta una risorsa predefinita o di fallback per i casi in cui nessun elemento soddisfa le impostazioni cultura richieste. Per restituire solo la chiave in caso di mancanza di una risorsa per le impostazioni cultura richieste è necessario che non sia specificato un file di risorse predefinito.

### <a name="generate-resource-files-with-visual-studio"></a>Generare file di risorse con Visual Studio

Se si crea un file di risorse in Visual Studio senza le impostazioni cultura nel nome del file (ad esempio, *Welcome.resx*), Visual Studio crea una classe C# con una proprietà per ogni stringa. In genere questo non è il risultato desiderato quando si usa ASP.NET Core. Solitamente non è presente un file di risorse con estensione *resx* predefinito (un file con estensione *resx* senza nome delle impostazioni cultura). Si consiglia di creare il file con estensione *resx* con un nome delle impostazioni cultura, ad esempio *Welcome.fr.resx*. Quando si crea un file con estensione *resx* con un nome delle impostazioni cultura, Visual Studio non genera il file di classe.

### <a name="add-other-cultures"></a>Aggiungere altre impostazioni cultura

Ogni combinazione di lingua e impostazioni cultura (diversa dalla lingua predefinita) richiede un file di risorse univoco. Creare file di risorse per impostazioni cultura e locali diverse creando nuovi file di risorse in cui i codici di lingua ISO sono inclusi nel nome file, ad esempio **en-us**, **fr-ca** e **en-gb**. I codici ISO vengono inseriti tra il nome file e l'estensione *resx* come in *Welcome.es-MX.resx* (spagnolo/Messico).

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>Implementare una strategia per la selezione della lingua o delle impostazioni cultura per ogni richiesta

### <a name="configure-localization"></a>Configurare la localizzazione

La localizzazione è configurata nel metodo `Startup.ConfigureServices`:

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` aggiunge i servizi di localizzazione al contenitore dei servizi. Il codice precedente imposta anche il percorso delle risorse su "Resources".

* `AddViewLocalization` aggiunge il supporto per i file di visualizzazione localizzati. Nell'esempio la localizzazione delle visualizzazioni è basata sul suffisso dei file di visualizzazione. Ad esempio "fr" nel file *Index.fr.cshtml*.

* `AddDataAnnotationsLocalization` aggiunge il supporto per `DataAnnotations` i messaggi di convalida localizzati tramite `IStringLocalizer` astrazioni.

### <a name="localization-middleware"></a>Middleware di localizzazione

Le impostazioni cultura correnti in un richiesta sono impostate nel [middleware](xref:fundamentals/middleware/index) di localizzazione. Il middleware di localizzazione viene abilitato nel metodo `Startup.Configure`. Il middleware di localizzazione deve essere configurato prima di qualsiasi middleware che potrebbe controllare le impostazioni cultura della richiesta (ad esempio, `app.UseMvcWithDefaultRoute()`).

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` inizializza un oggetto `RequestLocalizationOptions`. In ogni richiesta l'elenco di `RequestCultureProvider` in `RequestLocalizationOptions` è enumerato e viene usato il primo provider in grado di determinare le impostazioni cultura della richiesta. I provider predefiniti provengono dalla classe `RequestLocalizationOptions`:

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

L'elenco predefinito passa dal più specifico al meno specifico. Di seguito in questo articolo viene descritto come modificare l'ordine e aggiungere un provider delle impostazioni cultura personalizzato. Se nessuno dei provider è in grado di determinare le impostazioni cultura della richiesta, viene usato `DefaultRequestCulture`.

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

Alcune app utilizzeranno una stringa di query per impostare <xref:System.Globalization.CultureInfo> . Per le app che usano l' cookie approccio dell'intestazione o Accept-Language, l'aggiunta di una stringa di query all'URL è utile per il debug e il test del codice. Per impostazione predefinita, `QueryStringRequestCultureProvider` viene registrato come primo provider di localizzazione nell'elenco `RequestCultureProvider`. Passare i parametri della stringa di query `culture` e `ui-culture`. L'esempio seguente specifica le impostazioni cultura specifiche (lingua e area) per spagnolo/Messico:

```
http://localhost:5000/?culture=es-MX&ui-culture=es-MX
```

Se si passa uno solo dei due parametri (`culture` o `ui-culture`), il provider della stringa di query imposterà entrambi i valori usando il parametro passato. Ad esempio, se si specificano solo le impostazioni cultura verranno impostati entrambi i parametri `Culture` e `UICulture`:

```
http://localhost:5000/?culture=es-MX
```

### <a name="cookierequestcultureprovider"></a>CookieRequestCultureProvider

Le app di produzione forniranno spesso un meccanismo per impostare le impostazioni cultura con le impostazioni cultura ASP.NET Core cookie . Usare il `MakeCookieValue` metodo per creare un oggetto cookie .

`CookieRequestCultureProvider` `DefaultCookieName` Restituisce il nome predefinito cookie utilizzato per tenere traccia delle informazioni sulle impostazioni cultura preferite dell'utente. Il cookie nome predefinito è `.AspNetCore.Culture` .

Il cookie formato è `c=%LANGCODE%|uic=%LANGCODE%` , dove `c` è `Culture` e `uic` è `UICulture` , ad esempio:

```
c=en-UK|uic=en-US
```

Se si specificano solo le impostazioni cultura o le impostazioni cultura dell'interfaccia utente, le impostazioni specificate verranno usate per entrambi i tipi di impostazioni.

### <a name="the-accept-language-http-header"></a>Intestazione HTTP Accept-Language

L'[intestazione Accept-Language](https://www.w3.org/International/questions/qa-accept-lang-locales) può essere impostata nella maggior parte dei browser ed è stata originariamente progettata per specificare la lingua dell'utente. Questa impostazione indica la lingua impostata per l'invio da parte del browser o che il browser ha ereditato dal sistema operativo sottostante. L'intestazione HTTP Accept-Language di una richiesta del browser non è un metodo infallibile per individuare la lingua preferita dell'utente (vedere [Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php) (Impostazione delle preferenze di lingua in un browser)). Un'app di produzione deve consentire all'utente di personalizzare le impostazioni cultura.

### <a name="set-the-accept-language-http-header-in-ie"></a>Impostare l'intestazione HTTP Accept-Language in IE

1. Dall'icona a forma di ingranaggio, toccare **Opzioni Internet**.

1. Toccare **Lingue**.

   ![Opzioni Internet](localization/_static/lang.png)

1. Toccare **Imposta preferenze lingua**.

1. Toccare **Aggiungi una lingua**.

1. Aggiungere la lingua.

1. Toccare la lingua e quindi toccare **Sposta su**.

### <a name="the-content-language-http-header"></a>Intestazione HTTP Content-Language

Intestazione dell'entità [Content-Language](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Language) :

* Viene usato per descrivere le lingue destinate ai destinatari.
* Consente a un utente di distinguere in base alla lingua preferita degli utenti.

Le intestazioni delle entità vengono usate sia nelle richieste HTTP sia nelle risposte.

L' `Content-Language` intestazione può essere aggiunta impostando la proprietà `ApplyCurrentCultureToResponseHeaders` .

Aggiunta dell' `Content-Language` intestazione:

* Consente a RequestLocalizationMiddleware di impostare l' `Content-Language` intestazione con `CurrentUICulture` .
* Elimina la necessità di impostare l'intestazione della risposta in `Content-Language` modo esplicito.

```csharp
app.UseRequestLocalization(new RequestLocalizationOptions
{
    ApplyCurrentCultureToResponseHeaders = true
});
```

### <a name="use-a-custom-provider"></a>Usare un provider personalizzato

Si supponga di voler consentire agli utenti di memorizzare lingua e impostazioni cultura nei database. È possibile creare un provider per la ricerca di questi valori per l'utente. Il codice seguente illustra come aggiungere un provider personalizzato:

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

Usare `RequestLocalizationOptions` per aggiungere o rimuovere i provider di localizzazione.

### <a name="set-the-culture-programmatically"></a>Specificare le impostazioni cultura a livello di codice

Questo progetto di esempio **Localization.StarterWeb** in [GitHub](https://github.com/aspnet/entropy) contiene l'interfaccia utente per impostare `Culture`. Il file *Views/Shared/_SelectLanguagePartial.cshtml* consente di selezionare le impostazioni cultura dall'elenco delle impostazioni cultura supportate:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

Il file *Views/Shared/_SelectLanguagePartial.cshtml* viene aggiunto alla sezione `footer` del file di layout in modo che sia disponibile in tutte le visualizzazioni:

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

Il `SetLanguage` metodo imposta le impostazioni cultura cookie .

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

Non è possibile collegare *_SelectLanguagePartial.cshtml* al codice di esempio per questo progetto. Il progetto **Localization. StarterWeb** in [GitHub](https://github.com/aspnet/entropy) dispone di codice per il flusso `RequestLocalizationOptions` di a un oggetto Razor parziale tramite il contenitore di [inserimento delle dipendenze](dependency-injection.md) .

## <a name="model-binding-route-data-and-query-strings"></a>Associazione di modelli dati della route e stringhe di query

Vedere [comportamento di globalizzazione delle stringhe di query e dati della route di associazione di modelli](xref:mvc/models/model-binding#glob).

## <a name="globalization-and-localization-terms"></a>Termini relativi alla globalizzazione e alla localizzazione

Il processo di localizzazione dell'app richiede anche una conoscenza di base dei set di caratteri comunemente usati nello sviluppo del software moderno e dei problemi associati a essi. Sebbene tutti i computer memorizzano il testo come numeri (codici), sistemi diversi memorizzano lo stesso testo usando numeri diversi. Il processo di localizzazione si riferisce alla traduzione dell'interfaccia utente dell'app per impostazioni cultura o locali specifiche.

La [localizzabilità](/dotnet/standard/globalization-localization/localizability-review) è un processo intermedio che verifica che un'app globalizzata sia pronta per la localizzazione.

Il formato [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) per il nome delle impostazioni cultura è `<languagecode2>-<country/regioncode2>`, dove `<languagecode2>` è il codice della lingua e `<country/regioncode2>` è il codice della cultura secondaria. Ad esempio, `es-CL` per spagnolo (Cile), `en-US` per inglese (Stati Uniti) e `en-AU` per inglese (Australia). [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) è la combinazione di un codice di cultura in lettere minuscole di due lettere ISO 639 associato a una lingua e un codice di cultura secondaria in lettere maiuscole di due lettere ISO 3166 associato a un paese o un'area. Vedere <https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>.

L'internazionalizzazione è spesso abbreviata con "I18N". L'abbreviazione include la prima e l'ultima lettera e il numero di lettere che intercorrono tra di loro nel termine inglese "Internationalization". 18 è il numero di lettere tra la prima "I" e l'ultima "N". Lo stesso vale per globalizzazione (G11N) e localizzazione (L10N).

Termini:

* Globalizzazione (G11N): processo che consente a un'app di supportare lingue e aree diverse.
* Localizzazione (L10N): processo di personalizzazione di un'app per una determinata lingua e area.
* Internazionalizzazione (I18N): descrive la globalizzazione e la localizzazione.
* Impostazioni cultura: si tratta di una lingua e, facoltativamente, di un'area.
* Impostazioni cultura neutre: cultura con una lingua specificata ma senza area. (ad esempio "en", "es")
* Impostazioni cultura specifiche: cultura con una lingua e un'area specificate. (ad esempio "en-US", "en-GB", "es-CL")
* Impostazioni padre: impostazioni cultura neutre con impostazioni cultura specifiche. (ad esempio, "en" rappresenta le impostazioni cultura padre di "en-US" e "en-GB")
* Impostazioni locali: le impostazioni locali corrispondono alle impostazioni cultura.

[!INCLUDE[](~/includes/localization/currency.md)]

[!INCLUDE[](~/includes/localization/unsupported-culture-log-level.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* [Progetto Localization.StarterWeb](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb) usato nell'articolo.
* [Globalizzazione e localizzazione di applicazioni .NET](/dotnet/standard/globalization-localization/index)
* [Risorse nei file con estensione resx](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft Multilingual App Toolkit](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [Localizzazione e generics](http://hishambinateya.com/localization-and-generics)

::: moniker-end
