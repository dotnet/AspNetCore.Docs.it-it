---
title: Configurare la localizzazione degli oggetti portabili in ASP.NET Core
author: sebastienros
description: Questo articolo presenta i file degli oggetti portabili e descrive i passaggi per l'uso dei file in un'applicazione ASP.NET Core con il framework Orchard Core.
ms.author: scaddie
ms.date: 09/26/2017
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
uid: fundamentals/portable-object-localization
ms.openlocfilehash: ed41657b364c7f845491b3e452db8a4d5c5aa389
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587177"
---
# <a name="configure-portable-object-localization-in-aspnet-core"></a>Configurare la localizzazione degli oggetti portabili in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Sébastien Ros](https://github.com/sebastienros), [Scott Addie](https://twitter.com/Scott_Addie) e [Hisham bin Ateya](https://github.com/hishamco)

Questo articolo descrive i passaggi per l'uso dei file degli oggetti portabili (Portable Object, PO) in un'applicazione ASP.NET Core con il framework [Orchard Core](https://github.com/OrchardCMS/OrchardCore).

**Nota:** Orchard Core non è un prodotto Microsoft. Di conseguenza, Microsoft non fornisce alcun supporto per questa funzionalità.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/localization/sample/3.x/POLocalization) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="what-is-a-po-file"></a>Che cos'è un file PO?

I file PO sono distribuiti come file di testo contenenti le stringhe tradotte per una determinata lingua. Di seguito sono riportati alcuni vantaggi dell'uso di file PO anziché di file *resx* :
- Pluralizzazione del supporto dei file PO; i file *.resx* non supportano la pluralizzazione.
- I file PO non vengono compilati come i file *.resx*. Di conseguenza, non sono richiesti strumenti e passaggi di compilazione specifici.
- I file PO funzionano perfettamente con gli strumenti di modifica online di collaborazione.

### <a name="example"></a>Esempio

Di seguito è riportato un file PO di esempio contenente le traduzioni di due stringhe in francese, inclusa la stringa con la forma plurale:

*fr.po*

```text
#: Services/EmailService.cs:29
msgid "Enter a comma separated list of email addresses."
msgstr "Entrez une liste d'emails séparés par une virgule."

#: Views/Email.cshtml:112
msgid "The email address is \"{0}\"."
msgid_plural "The email addresses are \"{0}\"."
msgstr[0] "L'adresse email est \"{0}\"."
msgstr[1] "Les adresses email sont \"{0}\""
```

Questo esempio usa la sintassi seguente:

- `#:`: commento che indica il contesto della stringa da tradurre. La stessa stringa può essere tradotta in modo diverso a seconda della posizione in cui viene usata.
- `msgid`: stringa non tradotta.
- `msgstr`: stringa tradotta.

Se è supportata la pluralizzazione, è possibile definire più voci.

- `msgid_plural`: stringa al plurale non tradotta.
- `msgstr[0]`: stringa tradotta per il caso 0.
- `msgstr[N]`: stringa tradotta per il caso N.

La specifica dei file PO è disponibile [qui](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html).

## <a name="configuring-po-file-support-in-aspnet-core"></a>Configurazione del supporto dei file PO in ASP.NET Core

Questo esempio è basato su un'applicazione ASP.NET Core MVC generata da un modello di progetto Visual Studio 2017.

### <a name="referencing-the-package"></a>Riferimento al pacchetto

Aggiungere un riferimento al pacchetto NuGet `OrchardCore.Localization.Core`.

Il file *.csproj* contiene ora una riga simile alla seguente (il numero di versione può variare):

[!code-xml[](localization/sample/3.x/POLocalization/POLocalization.csproj?range=8)]

### <a name="registering-the-service"></a>Registrazione del servizio

Aggiungere i servizi necessari al metodo `ConfigureServices` di *Startup.cs*:

[!code-csharp[](localization/sample/3.x/POLocalization/Startup.cs?name=snippet_ConfigureServices&highlight=4-21)]

Aggiungere il middleware necessario al metodo `Configure` di *Startup.cs*:

[!code-csharp[](localization/sample/3.x/POLocalization/Startup.cs?name=snippet_Configure&highlight=15)]

Aggiungere il codice seguente alla propria Razor visualizzazione scelta. In questo esempio viene usata *About.cshtml*.

[!code-cshtml[](localization/sample/3.x/POLocalization/Views/Home/About.cshtml)]

Viene inserita un'istanza `IViewLocalizer` che viene usata per tradurre il testo "Hello world!".

### <a name="creating-a-po-file"></a>Creazione di un file PO

Creare un file denominato *\<culture code> . po* nella cartella radice dell'applicazione. In questo esempio il nome file è *fr.po* poiché viene usata la lingua francese:

[!code-text[](localization/sample/3.x/POLocalization/fr.po)]

Il file memorizza la stringa da tradurre e la stringa tradotta in francese. Le traduzioni vengono ripristinate alla relativa impostazione cultura, se necessario. In questo esempio viene usato il file *fr.po* se le impostazioni cultura richieste sono `fr-FR` o `fr-CA`.

### <a name="testing-the-application"></a>Test dell'applicazione

Eseguire l'applicazione e passare all'URL `/Home/About`. Il testo **Hello world!** vengono visualizzati i puntini di sospensione (...).

Passare all'URL `/Home/About?culture=fr-FR`. Il testo **Bonjour le monde!** vengono visualizzati i puntini di sospensione (...).

## <a name="pluralization"></a>Pluralizzazione

I file PO supportano le forme di pluralizzazione, utili quando è necessario tradurre la stessa stringa in modo diverso in base a una cardinalità. Questa attività risulta complessa poiché ogni lingua definisce regole personalizzate per la selezione della stringa da usare in base alla cardinalità.

Il pacchetto di localizzazione Orchard offre un'API per richiamare automaticamente le diverse forme plurali.

### <a name="creating-pluralization-po-files"></a>Creazione di file PO di pluralizzazione

Aggiungere il contenuto seguente al file *fr.po* indicato in precedenza:

```text
msgid "There is one item."
msgid_plural "There are {0} items."
msgstr[0] "Il y a un élément."
msgstr[1] "Il y a {0} éléments."
```

Vedere [Che cos'è un file PO?](#what-is-a-po-file) per una descrizione di ogni voce di questo esempio.

### <a name="adding-a-language-using-different-pluralization-forms"></a>Aggiunta di una lingua che usa forme di pluralizzazione diverse

Nell'esempio precedente sono state usate stringhe in inglese e in francese. Inglese e francese hanno solo due forme di pluralizzazione e condividono le stesse regole di formato, ovvero una cardinalità viene mappata alla prima forma plurale. Un'eventuale altra cardinalità viene mappata alla seconda forma plurale.

Non tutte le lingue condividono le stesse regole. La lingua ceca, ad esempio, ha tre forme plurali.

Creare il file `cs.po` come descritto di seguito e notare che la pluralizzazione richiede tre traduzioni diverse:

[!code-text[](localization/sample/3.x/POLocalization/cs.po)]

Per accettare le localizzazioni ceche, aggiungere `"cs"` all'elenco delle impostazioni cultura supportate nel metodo `ConfigureServices`:

```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    new CultureInfo("cs")
};
```

Modificare il file *Views/Home/About.cshtml* per eseguire il rendering delle stringhe plurali localizzate per più cardinalità:

```cshtml
<p>@Localizer.Plural(1, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(2, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(5, "There is one item.", "There are {0} items.")</p>
```

**Nota:** in uno scenario reale potrebbe essere usata una variabile per rappresentare il conteggio. In questo caso, viene ripetuto lo stesso codice con tre valori diversi per esporre un caso molto specifico.

Per le diverse impostazioni cultura, si ottiene quanto segue:

Per `/Home/About`:

```html
There is one item.
There are 2 items.
There are 5 items.
```

Per `/Home/About?culture=fr`:

```html
Il y a un élément.
Il y a 2 éléments.
Il y a 5 éléments.
```

Per `/Home/About?culture=cs`:

```html
Existuje jedna položka.
Existují 2 položky.
Existuje 5 položek.
```

Si noti che per le impostazioni cultura per la lingua ceca, le tre traduzioni sono diverse. Le impostazioni cultura per la lingua inglese e francese condividono la stessa costruzione per le ultime due stringhe tradotte.

## <a name="advanced-tasks"></a>Attività avanzate

### <a name="contextualizing-strings"></a>Contestualizzazione delle stringhe

Le applicazioni spesso contengono le stringhe da tradurre in posizioni diverse. La stessa stringa può avere una traduzione diversa in determinate posizioni all'interno di un'app ( Razor viste o file di classe). Un file PO supporta la nozione di contesto del file, che è possibile usare per categorizzare la stringa rappresentata. Usando un contesto di file è possibile tradurre una stringa in modo diverso a seconda del contesto o della mancanza del contesto.

I servizi di localizzazione PO usano il nome della classe completa o la visualizzazione usata durante la traduzione di una stringa. Questa operazione viene eseguita impostando il valore nella voce `msgctxt`.

Si supponga di eseguire un'aggiunta minore all'esempio *fr.po* precedente. Una Razor visualizzazione che si trova in *views/Home/About. cshtml* può essere definita come contesto del file impostando il `msgctxt` valore della voce riservata:

```text
msgctxt "Views.Home.About"
msgid "Hello world!"
msgstr "Bonjour le monde!"
```

Con `msgctxt` impostato in questo modo, la traduzione del testo si verifica quando si passa a `/Home/About?culture=fr-FR`. La traduzione non si verifica quando si passa a `/Home/Contact?culture=fr-FR`.

Quando nessuna voce corrisponde al contesto di file specificato, il meccanismo di fallback di Orchard Core cerca un file PO appropriato senza contesto. Presupponendo che non sia stato definito alcun contesto di file specifico per *Views/Home/Contact.cshtml*, il passaggio a `/Home/Contact?culture=fr-FR` carica un file PO come:

[!code-text[](localization/sample/3.x/POLocalization/fr.po)]

### <a name="changing-the-location-of-po-files"></a>Modifica della posizione dei file PO

La posizione predefinita dei file PO può essere modificata in `ConfigureServices`:

```csharp
services.AddPortableObjectLocalization(options => options.ResourcesPath = "Localization");
```

In questo esempio i file PO vengono caricati dalla cartella *Localization*.

### <a name="implementing-a-custom-logic-for-finding-localization-files"></a>Implementazione di una logica personalizzata per la ricerca dei file di localizzazione

Quando è necessaria una logica più complessa per individuare i file PO, l'interfaccia `OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider` può essere implementata e registrata come servizio. Ciò è utile quando i file PO possono essere memorizzati in posizioni diverse o quando i file devono essere cercati all'interno di una gerarchia di cartelle.

### <a name="using-a-different-default-pluralized-language"></a>Uso di una lingua pluralizzata predefinita diversa

Il pacchetto include un metodo di estensione `Plural` specifico di due forme plurali. Per le lingue che richiedono ulteriori forme plurali, creare un metodo di estensione. Con un metodo di estensione, non sarà necessario specificare un file di localizzazione per la lingua predefinita &mdash; le stringhe originali sono già disponibili direttamente nel codice.

È possibile usare l'overload `Plural(int count, string[] pluralForms, params object[] arguments)` più generico che accetta una matrice di stringhe di traduzioni.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[Sébastien Ros](https://github.com/sebastienros) e [Scott Addie](https://twitter.com/Scott_Addie)

Questo articolo descrive i passaggi per l'uso dei file degli oggetti portabili (Portable Object, PO) in un'applicazione ASP.NET Core con il framework [Orchard Core](https://github.com/OrchardCMS/OrchardCore).

**Nota:** Orchard Core non è un prodotto Microsoft. Di conseguenza, Microsoft non fornisce alcun supporto per questa funzionalità.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/localization/sample/2.x/POLocalization) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="what-is-a-po-file"></a>Che cos'è un file PO?

I file PO sono distribuiti come file di testo contenenti le stringhe tradotte per una determinata lingua. Alcuni dei vantaggi offerti dall'uso dei file PO al posto dei file *.resx* sono i seguenti:
- Pluralizzazione del supporto dei file PO; i file *.resx* non supportano la pluralizzazione.
- I file PO non vengono compilati come i file *.resx*. Di conseguenza, non sono richiesti strumenti e passaggi di compilazione specifici.
- I file PO funzionano perfettamente con gli strumenti di modifica online di collaborazione.

### <a name="example"></a>Esempio

Di seguito è riportato un file PO di esempio contenente le traduzioni di due stringhe in francese, inclusa la stringa con la forma plurale:

*fr.po*

```text
#: Services/EmailService.cs:29
msgid "Enter a comma separated list of email addresses."
msgstr "Entrez une liste d'emails séparés par une virgule."

#: Views/Email.cshtml:112
msgid "The email address is \"{0}\"."
msgid_plural "The email addresses are \"{0}\"."
msgstr[0] "L'adresse email est \"{0}\"."
msgstr[1] "Les adresses email sont \"{0}\""
```

Questo esempio usa la sintassi seguente:

- `#:`: commento che indica il contesto della stringa da tradurre. La stessa stringa può essere tradotta in modo diverso a seconda della posizione in cui viene usata.
- `msgid`: stringa non tradotta.
- `msgstr`: stringa tradotta.

Se è supportata la pluralizzazione, è possibile definire più voci.

- `msgid_plural`: stringa al plurale non tradotta.
- `msgstr[0]`: stringa tradotta per il caso 0.
- `msgstr[N]`: stringa tradotta per il caso N.

La specifica dei file PO è disponibile [qui](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html).

## <a name="configuring-po-file-support-in-aspnet-core"></a>Configurazione del supporto dei file PO in ASP.NET Core

Questo esempio è basato su un'applicazione ASP.NET Core MVC generata da un modello di progetto Visual Studio 2017.

### <a name="referencing-the-package"></a>Riferimento al pacchetto

Aggiungere un riferimento al pacchetto NuGet `OrchardCore.Localization.Core`.

Il file *.csproj* contiene ora una riga simile alla seguente (il numero di versione può variare):

[!code-xml[](localization/sample/2.x/POLocalization/POLocalization.csproj?range=9)]

### <a name="registering-the-service"></a>Registrazione del servizio

Aggiungere i servizi necessari al metodo `ConfigureServices` di *Startup.cs*:

[!code-csharp[](localization/sample/2.x/POLocalization/Startup.cs?name=snippet_ConfigureServices&highlight=4-21)]

Aggiungere il middleware necessario al metodo `Configure` di *Startup.cs*:

[!code-csharp[](localization/sample/2.x/POLocalization/Startup.cs?name=snippet_Configure&highlight=15)]

Aggiungere il codice seguente alla propria Razor visualizzazione scelta. In questo esempio viene usata *About.cshtml*.

[!code-cshtml[](localization/sample/2.x/POLocalization/Views/Home/About.cshtml)]

Viene inserita un'istanza `IViewLocalizer` che viene usata per tradurre il testo "Hello world!".

### <a name="creating-a-po-file"></a>Creazione di un file PO

Creare un file denominato *\<culture code> . po* nella cartella radice dell'applicazione. In questo esempio il nome file è *fr.po* poiché viene usata la lingua francese:

[!code-text[](localization/sample/2.x/POLocalization/fr.po)]

Il file memorizza la stringa da tradurre e la stringa tradotta in francese. Le traduzioni vengono ripristinate alla relativa impostazione cultura, se necessario. In questo esempio viene usato il file *fr.po* se le impostazioni cultura richieste sono `fr-FR` o `fr-CA`.

### <a name="testing-the-application"></a>Test dell'applicazione

Eseguire l'applicazione e passare all'URL `/Home/About`. Il testo **Hello world!** vengono visualizzati i puntini di sospensione (...).

Passare all'URL `/Home/About?culture=fr-FR`. Il testo **Bonjour le monde!** vengono visualizzati i puntini di sospensione (...).

## <a name="pluralization"></a>Pluralizzazione

I file PO supportano le forme di pluralizzazione, utili quando è necessario tradurre la stessa stringa in modo diverso in base a una cardinalità. Questa attività risulta complessa poiché ogni lingua definisce regole personalizzate per la selezione della stringa da usare in base alla cardinalità.

Il pacchetto di localizzazione Orchard offre un'API per richiamare automaticamente le diverse forme plurali.

### <a name="creating-pluralization-po-files"></a>Creazione di file PO di pluralizzazione

Aggiungere il contenuto seguente al file *fr.po* indicato in precedenza:

```text
msgid "There is one item."
msgid_plural "There are {0} items."
msgstr[0] "Il y a un élément."
msgstr[1] "Il y a {0} éléments."
```

Vedere [Che cos'è un file PO?](#what-is-a-po-file) per una descrizione di ogni voce di questo esempio.

### <a name="adding-a-language-using-different-pluralization-forms"></a>Aggiunta di una lingua che usa forme di pluralizzazione diverse

Nell'esempio precedente sono state usate stringhe in inglese e in francese. Inglese e francese hanno solo due forme di pluralizzazione e condividono le stesse regole di formato, ovvero una cardinalità viene mappata alla prima forma plurale. Un'eventuale altra cardinalità viene mappata alla seconda forma plurale.

Non tutte le lingue condividono le stesse regole. La lingua ceca, ad esempio, ha tre forme plurali.

Creare il file `cs.po` come descritto di seguito e notare che la pluralizzazione richiede tre traduzioni diverse:

[!code-text[](localization/sample/2.x/POLocalization/cs.po)]

Per accettare le localizzazioni ceche, aggiungere `"cs"` all'elenco delle impostazioni cultura supportate nel metodo `ConfigureServices`:

```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    new CultureInfo("cs")
};
```

Modificare il file *Views/Home/About.cshtml* per eseguire il rendering delle stringhe plurali localizzate per più cardinalità:

```cshtml
<p>@Localizer.Plural(1, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(2, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(5, "There is one item.", "There are {0} items.")</p>
```

**Nota:** in uno scenario reale potrebbe essere usata una variabile per rappresentare il conteggio. In questo caso, viene ripetuto lo stesso codice con tre valori diversi per esporre un caso molto specifico.

Per le diverse impostazioni cultura, si ottiene quanto segue:

Per `/Home/About`:

```html
There is one item.
There are 2 items.
There are 5 items.
```

Per `/Home/About?culture=fr`:

```html
Il y a un élément.
Il y a 2 éléments.
Il y a 5 éléments.
```

Per `/Home/About?culture=cs`:

```html
Existuje jedna položka.
Existují 2 položky.
Existuje 5 položek.
```

Si noti che per le impostazioni cultura per la lingua ceca, le tre traduzioni sono diverse. Le impostazioni cultura per la lingua inglese e francese condividono la stessa costruzione per le ultime due stringhe tradotte.

## <a name="advanced-tasks"></a>Attività avanzate

### <a name="contextualizing-strings"></a>Contestualizzazione delle stringhe

Le applicazioni spesso contengono le stringhe da tradurre in posizioni diverse. La stessa stringa può avere una traduzione diversa in determinate posizioni all'interno di un'app ( Razor viste o file di classe). Un file PO supporta la nozione di contesto del file, che è possibile usare per categorizzare la stringa rappresentata. Usando un contesto di file è possibile tradurre una stringa in modo diverso a seconda del contesto o della mancanza del contesto.

I servizi di localizzazione PO usano il nome della classe completa o la visualizzazione usata durante la traduzione di una stringa. Questa operazione viene eseguita impostando il valore nella voce `msgctxt`.

Si supponga di eseguire un'aggiunta minore all'esempio *fr.po* precedente. Una Razor visualizzazione che si trova in *views/Home/About. cshtml* può essere definita come contesto del file impostando il `msgctxt` valore della voce riservata:

```text
msgctxt "Views.Home.About"
msgid "Hello world!"
msgstr "Bonjour le monde!"
```

Con `msgctxt` impostato in questo modo, la traduzione del testo si verifica quando si passa a `/Home/About?culture=fr-FR`. La traduzione non si verifica quando si passa a `/Home/Contact?culture=fr-FR`.

Quando nessuna voce corrisponde al contesto di file specificato, il meccanismo di fallback di Orchard Core cerca un file PO appropriato senza contesto. Presupponendo che non sia stato definito alcun contesto di file specifico per *Views/Home/Contact.cshtml*, il passaggio a `/Home/Contact?culture=fr-FR` carica un file PO come:

[!code-text[](localization/sample/2.x/POLocalization/fr.po)]

### <a name="changing-the-location-of-po-files"></a>Modifica della posizione dei file PO

La posizione predefinita dei file PO può essere modificata in `ConfigureServices`:

```csharp
services.AddPortableObjectLocalization(options => options.ResourcesPath = "Localization");
```

In questo esempio i file PO vengono caricati dalla cartella *Localization*.

### <a name="implementing-a-custom-logic-for-finding-localization-files"></a>Implementazione di una logica personalizzata per la ricerca dei file di localizzazione

Quando è necessaria una logica più complessa per individuare i file PO, l'interfaccia `OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider` può essere implementata e registrata come servizio. Ciò è utile quando i file PO possono essere memorizzati in posizioni diverse o quando i file devono essere cercati all'interno di una gerarchia di cartelle.

### <a name="using-a-different-default-pluralized-language"></a>Uso di una lingua pluralizzata predefinita diversa

Il pacchetto include un metodo di estensione `Plural` specifico di due forme plurali. Per le lingue che richiedono ulteriori forme plurali, creare un metodo di estensione. Con un metodo di estensione, non sarà necessario specificare un file di localizzazione per la lingua predefinita &mdash; le stringhe originali sono già disponibili direttamente nel codice.

È possibile usare l'overload `Plural(int count, string[] pluralForms, params object[] arguments)` più generico che accetta una matrice di stringhe di traduzioni.

::: moniker-end
