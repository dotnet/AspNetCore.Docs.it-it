---
title: Creare helper tag in ASP.NET Core
author: rick-anderson
description: Informazioni su come creare helper tag in ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: mvc/views/tag-helpers/authoring
ms.openlocfilehash: e0c9f450e4eded49694cbbb0e9fa2614a221ab14
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586826"
---
# <a name="author-tag-helpers-in-aspnet-core"></a>Creare helper tag in ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/authoring/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="get-started-with-tag-helpers"></a>Guida introduttiva agli helper tag

Questa esercitazione rappresenta un'introduzione alla programmazione di helper tag. [Introduzione agli helper tag](intro.md) descrive i vantaggi offerti dagli helper tag.

Un helper tag è una classe che implementa l'interfaccia `ITagHelper`. Quando si crea un helper tag, tuttavia, in genere lo si deriva da `TagHelper`. In questo modo si ottiene l'accesso al metodo `Process`.

1. Creare un nuovo progetto ASP.NET Core denominato **AuthoringTagHelpers**. Per questo progetto non sarà necessaria l'autenticazione.

1. Creare una cartella che contenga gli helper tag denominata *TagHelpers*. La cartella *TagHelpers**non* è obbligatoria, ma rappresenta una convenzione sensata. È ora possibile iniziare a scrivere alcuni helper tag semplici.

## <a name="a-minimal-tag-helper"></a>Un helper tag piccolissimo

In questa sezione verrà scritto un helper tag che aggiorna un tag di posta elettronica. Ad esempio:

```html
<email>Support</email>
```

Il server userà l'helper tag di posta elettronica creato per convertire il markup nel codice seguente:

```html
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

In altre parole, un tag di ancoraggio che dà come risultato un collegamento di posta elettronica. È necessario eseguire questa operazione se si sta scrivendo un motore di blog e si vuole inviare posta elettronica di marketing o di supporto ad altri contatti, tutti nello stesso dominio.

1. Aggiungere la classe `EmailTagHelper` seguente alla cartella *TagHelpers*.

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1EmailTagHelperCopy.cs)]

   * Gli helper tag usano una convenzione di denominazione che interessa gli elementi del nome della classe di base (meno la parte *TagHelper* del nome della classe). In questo esempio, il nome radice di **EmailTagHelper** è *posta elettronica*, quindi il `<email>` tag verrà assegnato come destinazione. Questa convenzione di denominazione dovrebbe funzionare per la maggior parte degli helper tag. Più avanti verrà illustrato come eseguirne l'override.

   * La classe `EmailTagHelper` deriva da `TagHelper`. La classe `TagHelper` mette a disposizione metodi e proprietà per la scrittura di helper tag.

   * Il metodo `Process` sottoposto a override controlla ciò che l'helper tag fa quando viene eseguito. La classe `TagHelper` fornisce anche una versione asincrona (`ProcessAsync`) con gli stessi parametri.

   * Il parametro context di `Process` (e `ProcessAsync`) contiene informazioni associate all'esecuzione del tag HTML corrente.

   * Il parametro output di `Process` (e `ProcessAsync`) contiene un elemento HTML con stato rappresentativo dell'origine originale usato per generare un tag e del contenuto HTML.

   * Il nome della classe usata ha il suffisso **TagHelper**, che *non* è obbligatorio, ma il cui uso è considerato una convenzione basata su procedure consigliate. È possibile dichiarare la classe come:

   ```csharp
   public class Email : TagHelper
   ```

1. Per rendere la `EmailTagHelper` classe disponibile a tutte le Razor Visualizzazioni, aggiungere la `addTagHelper` direttiva al file *views/_ViewImports. cshtml* :

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopyEmail.cshtml?highlight=2,3)]

   Il codice qui sopra usa la sintassi con caratteri jolly per specificare che tutti gli helper tag in questo assembly saranno disponibili. La prima stringa dopo `@addTagHelper` specifica l'helper tag da caricare (usare "*" per indicare tutti gli helper tag), e la seconda stringa "AuthoringTagHelpers" specifica l'assembly in cui si trova l'helper tag. Si noti anche che la seconda riga introduce gli helper tag di ASP.NET Core MVC usando la sintassi con caratteri jolly (gli helper sono descritti in [Introduzione agli helper Tag](intro.md)). Si tratta della `@addTagHelper` direttiva che rende l'helper tag disponibile per la Razor visualizzazione. In alternativa, è possibile specificare il nome completo dell'helper tag, come illustrato di seguito:

```csharp
@using AuthoringTagHelpers
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.EmailTagHelper, AuthoringTagHelpers
```

<!--
the following snippet uses TagHelpers3 and should use TagHelpers (not the 3)
    [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImports.cshtml?highlight=3&range=1-3)]
-->

Per aggiungere un helper tag a una visualizzazione usando un nome completo, aggiungere prima il nome completo (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`) e quindi il **nome dell'assembly** (*AuthoringTagHelpers*, non necessariamente `namespace`). La maggior parte degli sviluppatori preferisce usare la sintassi con caratteri jolly. [Introduzione agli helper tag](intro.md) descrive in dettaglio la sintassi di aggiunta, rimozione, gerarchia degli helper tag, nonché la sintassi con caratteri jolly.

1. Aggiornare il markup nel file *Views/Home/Contact.cshtml* con queste modifiche:

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. Eseguire l'app e usare il browser preferito per visualizzare l'origine HTML e verificare che i tag di posta elettronica siano stati sostituiti dal markup di ancoraggio, ad esempio, `<a>Support</a>`. *Support* e *Marketing* sono visualizzati come collegamenti, ma non hanno un attributo `href` che li renda funzionali. Questo problema verrà corretto nella prossima sezione.

## <a name="setattribute-and-setcontent"></a>SetAttribute e SetContent

In questa sezione la classe `EmailTagHelper` verrà aggiornata in modo che crei un tag di ancoraggio valido per la posta elettronica. Il modulo verrà aggiornato in modo da ottenere le informazioni da una Razor vista (sotto forma di `mail-to` attributo) e usarle per generare l'ancoraggio.

Aggiornare la classe `EmailTagHelper` con il codice seguente:

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?range=6-22)]

* I nomi della classe e delle proprietà per gli helper tag, scritti secondo la convenzione Pascal, vengono convertiti nel formato [kebab case](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101) corrispondente. Per usare l'attributo `MailTo`, pertanto, si userà l'equivalente `<email mail-to="value"/>`.

* L'ultima riga imposta il contenuto completato per l'helper tag dalle minime funzioni.

* La riga evidenziata illustra la sintassi per l'aggiunta di attributi:

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?highlight=6&range=14-21)]

Questo approccio funziona per l'attributo "href" a condizione che non esista nella raccolta di attributi. È anche possibile usare il metodo `output.Attributes.Add` per aggiungere un attributo di helper tag alla fine della raccolta di attributi di tag.

1. Aggiornare il markup nel file *Views/Home/Contact.cshtml* con queste modifiche:

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/ContactCopy.cshtml?highlight=15,16)]

1. Eseguire l'app e verificare che generi collegamenti corretti.

<a name="self-closing"></a>

   > [!NOTE]
   > Se si scrive il tag di posta elettronica a chiusura automatica (`<email mail-to="Rick" />`), anche l'output finale è a chiusura automatica. Per consentire la scrittura del tag con solo un tag di inizio ( `<email mail-to="Rick">` ) è necessario contrassegnare la classe con quanto segue:
   >
   > [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailVoid.cs?highlight=1&range=6-10)]

   Con un helper tag di posta elettronica a chiusura automatica, l'output è `<a href="mailto:Rick@contoso.com" />`. I tag di ancoraggio a chiusura automatica non costituiscono codice HTML valido ed è quindi necessario evitare di crearli, ma può essere necessario creare un helper tag a chiusura automatica. Gli helper tag impostano il tipo della proprietà `TagMode` dopo la lettura di un tag.

### <a name="processasync"></a>ProcessAsync

In questa sezione verrà scritto un helper di posta elettronica asincrono.

1. Sostituire la classe `EmailTagHelper` con il codice seguente:

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelper.cs?range=6-17)]

   **Note:**

   * Questa versione usa il metodo `ProcessAsync` asincrono. `GetChildContentAsync` asincrono restituisce un `Task` contenente `TagHelperContent`.

   * Usare il parametro `output` per ottenere il contenuto dell'elemento HTML.

1. Apportare le seguenti modifiche al file *Views/Home/Contact.cshtml*, in modo che l'helper tag possa ottenere la posta elettronica di destinazione.

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. Eseguire l'app e verificare che generi collegamenti di posta elettronica validi.

### <a name="removeall-precontentsethtmlcontent-and-postcontentsethtmlcontent"></a>RemoveAll, PreContent.SetHtmlContent e PostContent.SetHtmlContent

1. Aggiungere la classe `BoldTagHelper` seguente alla cartella *TagHelpers*.

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/BoldTagHelper.cs)]

   * L'attributo `[HtmlTargetElement]` passa un parametro di attributo che specifica che qualsiasi elemento HTML che contenga un attributo HTML denominato "bold" corrisponderà e che quindi verrà eseguito il metodo di override `Process` nella classe. Nell'esempio, il metodo `Process` rimuove l'attributo "bold" e racchiude il markup tra `<strong></strong>`.

   * Dato che non si deve sostituire il contenuto esistente del tag, è necessario scrivere il tag `<strong>` di apertura con il metodo `PreContent.SetHtmlContent` e il tag `</strong>` di chiusura con il metodo `PostContent.SetHtmlContent`.

1. Modificare la visualizzazione *About.cshtml* in modo che contenga un valore di attributo `bold`. Il codice completo è illustrato di seguito.

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutBoldOnly.cshtml?highlight=7)]

1. Eseguire l'app. È possibile usare il browser preferito per controllare l'origine e verificare il markup.

   L'attributo `[HtmlTargetElement]` qui sopra ha come destinazione solo il markup HTML che fornisce un nome di attributo corrispondente a "bold". L'elemento `<bold>` non è stato modificato dall'helper tag.

1. Impostare come commento la riga dell'attributo `[HtmlTargetElement]`. Per impostazione predefinita, userà quindi tag `<bold>`, vale a dire il markup HTML nel formato `<bold>`. Tenere presente che la convenzione di denominazione predefinita corrisponderà al nome della classe **Bold** TagHelper per i tag `<bold>`.

1. Eseguire l'applicazione e verificare che il tag `<bold>` venga elaborato dall'helper tag.

La decorazione di una classe con più attributi `[HtmlTargetElement]` risulta in un OR logico delle destinazioni. Usando il codice riportato di seguito, ad esempio, un tag bold o un attributo bold corrisponderanno.

[!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zBoldTagHelperCopy.cs?highlight=1,2&range=5-15)]

Se si aggiungono più attributi alla stessa istruzione, il runtime li gestisce come AND logici. Nel codice riportato di seguito, ad esempio, perché corrisponda, un elemento HTML deve essere denominato "bold" con un attributo denominato "bold" (`<bold bold />`).

```csharp
[HtmlTargetElement("bold", Attributes = "bold")]
   ```

È anche possibile usare `[HtmlTargetElement]` per modificare il nome dell'elemento di destinazione. Se ad esempio si vuole che `BoldTagHelper` abbia come destinazione tag `<MyBold>`, usare l'attributo seguente:

```csharp
[HtmlTargetElement("MyBold")]
```

## <a name="pass-a-model-to-a-tag-helper"></a>Passare un modello a un helper tag

1. Aggiungere una cartella *Models*.

1. Aggiungere la classe `WebsiteContext` seguente alla cartella *Models*:

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Models/WebsiteContext.cs)]

1. Aggiungere la classe `WebsiteInformationTagHelper` seguente alla cartella *TagHelpers*.

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/WebsiteInformationTagHelper.cs)]

   * Come affermato in precedenza, gli helper tag convertono i nomi delle relative classi e proprietà C#, definiti secondo la convenzione Pascal, in formato [kebab case](https://wiki.c2.com/?KebabCase). Pertanto, per utilizzare `WebsiteInformationTagHelper` in Razor , si scriverà `<website-information />` .

   * Non si sta identificando in modo esplicito l'elemento di destinazione con l'attributo `[HtmlTargetElement]` e quindi verrà considerato come destinazione il valore predefinito di `website-information`. Se è stato applicato l'attributo seguente (si noti che non è nel formato kebab case ma corrisponde al nome della classe):

   ```csharp
   [HtmlTargetElement("WebsiteInformation")]
   ```

   Il tag nel formato kebab case `<website-information />` non corrisponde. Se si vuole usare l'attributo `[HtmlTargetElement]`, è necessario usare il formato kebab case come illustrato di seguito:

   ```csharp
   [HtmlTargetElement("Website-Information")]
   ```

   * Gli elementi a chiusura automatica non hanno contenuto. Per questo esempio, il Razor markup userà un tag di chiusura automatica, ma l'helper Tag creerà un elemento [Section](https://www.w3.org/TR/html5/sections.html#the-section-element) , che non è a chiusura automatica e si sta scrivendo contenuto all'interno dell' `section` elemento. Per scrivere output, è pertanto necessario impostare `TagMode` su `StartTagAndEndTag`. In alternativa, è possibile impostare come commento la riga che imposta `TagMode` e scrivere markup con un tag di chiusura. Del markup di esempio viene fornito più avanti in questa esercitazione.

   * Il simbolo `$` (segno di dollaro) nella riga seguente usa una [stringa interpolata](/dotnet/csharp/language-reference/keywords/interpolated-strings):

   ```cshtml
   $@"<ul><li><strong>Version:</strong> {Info.Version}</li>
   ```

1. Aggiungere il markup seguente alla visualizzazione *About.cshtml*. Il markup evidenziato visualizza le informazioni sul sito Web.

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?highlight=1,4-8, 18-999)]

   > [!NOTE]
   > Nel Razor markup mostrato di seguito:
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?range=18-18)]
   >
   > Razor sa `info` che l'attributo è una classe, non una stringa, e si vuole scrivere il codice C#. Qualsiasi attributo di helper tag non di tipo stringa deve essere scritto senza il carattere `@`.

1. Eseguire l'app e passare alla visualizzazione About per visualizzare le informazioni sul sito Web.

   > [!NOTE]
   > È possibile usare il markup seguente con un tag di chiusura e rimuovere la riga con `TagMode.StartTagAndEndTag` nell'helper tag:
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutNotSelfClosing.cshtml?range=20-21)]

## <a name="condition-tag-helper"></a>Helper tag di condizione

L'helper tag di condizione esegue il rendering dell'output quando riceve un valore true.

1. Aggiungere la classe `ConditionTagHelper` seguente alla cartella *TagHelpers*.

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/ConditionTagHelper.cs)]

1. Sostituire il contenuto del file *Views/Home/Index.cshtml* con il markup seguente:

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Index.cshtml)]

1. Sostituire il metodo `Index` nel controller `Home` con il codice seguente:

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Controllers/HomeController.cs?range=9-18)]

1. Eseguire l'app e passare alla home page. Non verrà eseguito il rendering del markup in `div` condizionale. Accodare la stringa di query `?approved=true` all'URL (ad esempio, `http://localhost:1235/Home/Index?approved=true`). `approved`è impostato su true e il markup condizionale verrà visualizzato.

> [!NOTE]
> Usare l'operatore [nameof](/dotnet/csharp/language-reference/keywords/nameof) per specificare l'attributo di destinazione, anziché specificare una stringa, come è stato fatto per l'helper tag bold:
>
> [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zConditionTagHelperCopy.cs?highlight=1,2,5&range=5-18)]
>
> L'operatore [nameof](/dotnet/csharp/language-reference/keywords/nameof) proteggerà il codice nel caso venisse effettuato il refactoring (potrebbe essere necessario modificare il nome in `RedCondition`).

### <a name="avoid-tag-helper-conflicts"></a>Evitare conflitti di helper tag

In questa sezione verrà scritta una coppia di helper tag a collegamento automatico. Il primo sostituirà il markup contenente un URL che inizia con HTTP con un tag di ancoraggio HTML contenente lo stesso URL (e che quindi genera un collegamento all'URL). Il secondo eseguirà la stessa operazione per un URL che inizia con WWW.

Poiché questi due helper sono strettamente correlati e in futuro potrebbero essere sottoposti a refactoring, verranno mantenuti nello stesso file.

1. Aggiungere la classe `AutoLinkerHttpTagHelper` seguente alla cartella *TagHelpers*.

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=7-19)]

   >[!NOTE]
   >La classe `AutoLinkerHttpTagHelper` ha come destinazioni elementi `p` e usa [Regex](/dotnet/standard/base-types/regular-expression-language-quick-reference) per creare l'ancoraggio.

1. Aggiungere il markup seguente alla fine del file *Views/Home/Contact.cshtml*:

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=19)]

1. Eseguire l'app e verificare che l'helper tag esegua correttamente il rendering dell'ancoraggio.

1. Aggiornare la classe `AutoLinker` per includere `AutoLinkerWwwTagHelper` che convertirà il testo www in un tag di ancoraggio contenente anch'esso il testo www originale. Il codice aggiornato è evidenziato di seguito:

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?highlight=15-34&range=7-34)]

1. Eseguire l'app. Si noti che il testo www è visualizzato come collegamento, ma il testo HTTP non lo è. Se si inserisce un punto di interruzione in entrambe le classi, è possibile notare che la classe helper tag HTTP viene eseguita per prima. Il problema è che l'output dell'helper tag viene memorizzato nella cache. Quando viene eseguito l'helper tag WWW, quest'ultimo sovrascrive l'output dell'helper tag HTTP memorizzato nella cache. Più avanti in questa esercitazione verrà illustrato come controllare l'ordine di esecuzione degli helper tag. È possibile correggere il problema con il codice seguente:

   [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10,21,22,26&range=8-37)]

   > [!NOTE]
   > Nella prima edizione degli helper tag a collegamento automatico, per ottenere il contenuto della destinazione è stato usato il codice seguente:
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=12)]
   >
   > In altre parole, si effettua una chiamata a `GetChildContentAsync` usando `TagHelperOutput` passato al metodo `ProcessAsync`. Come affermato in precedenza, poiché l'output viene memorizzato nella cache, l'ultimo helper eseguito prevale. Il problema è stato risolto con il codice seguente:
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?range=34-35)]
   >
   > Il codice precedente controlla se il contenuto è stato modificato e, in caso affermativo, ottiene il contenuto dal buffer di output.

1. Eseguire l'app e verificare che i due collegamenti funzionino come previsto. Può sembrare che l'helper tag a collegamento automatico sia corretto e completo, ma presenta un problema insidioso. Se l'helper tag WWW viene eseguito per primo, i collegamenti www non sono corretti. Aggiornare il codice aggiungendo l'overload `Order` per controllare l'ordine in cui viene eseguito il tag. La proprietà `Order` determina l'ordine di esecuzione rispetto agli altri helper tag che hanno come destinazione lo stesso elemento. Il valore dell'ordine predefinito è zero e le istanze con valori inferiori vengono eseguite per prime.

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?highlight=5,6,7,8&range=8-15)]

   Il codice precedente assicura che l'helper tag HTTP venga eseguito prima dell'helper tag WWW. Modificare `Order` in `MaxValue` e verificare che il markup generato per il tag WWW non è corretto.

## <a name="inspect-and-retrieve-child-content"></a>Controllare e recuperare il contenuto figlio

Gli helper tag mettono a disposizione diverse proprietà per recuperare il contenuto.

* Il risultato di `GetChildContentAsync` può essere accodato a `output.Content`.
* È possibile controllare il risultato di `GetChildContentAsync` con `GetContent`.
* Se si modifica `output.Content`, il corpo di TagHelper viene eseguito, o ne viene eseguito il rendering, solo se si chiama `GetChildContentAsync`, come nell'esempio di helper tag a collegamento automatico:

[!code-csharp[](../../views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10&range=8-21)]

* Più chiamate al metodo `GetChildContentAsync` restituiscono lo stesso valore e rieseguono nuovamente il corpo di `TagHelper` solo se viene passato un parametro false per indicare di non usare il risultato memorizzato nella cache.

## <a name="load-minified-partial-view-taghelper"></a>TagHelper per caricare visualizzazioni parziali minimizzate

Negli ambienti di produzione le prestazioni possono essere migliorate caricando visualizzazioni parziali minimizzate. Per sfruttare i vantaggi della visualizzazione parziale minimizzata nell'ambiente di produzione:

* Creare/configurare un processo di pre-compilazione che minimizza visualizzazioni parziali.
* Usare il codice seguente per caricare le visualizzazioni parziali minimizzate in ambienti non di sviluppo.

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/MinifiedVersionTagHelper.cs?name=snippet)]
