# <a name="contribute-to-the-aspnet-core-documentation"></a>Contribuire alla documentazione ASP.NET Core

Questo documento illustra il processo per offrire il proprio contributo per gli articoli e gli esempi di codice ospitati nel [sito della documentazione di ASP.NET](https://docs.microsoft.com/aspnet/). Sono benvenuti contributi come correzioni di errori e nuovi articoli.

## <a name="how-to-make-a-simple-correction-or-suggestion"></a>Come effettuare una semplice correzione o suggerire una modifica

Gli articoli sono archiviati nel repository come file markdown. È possibile apportare modifiche semplici al contenuto di un file markdown nel browser selezionando il collegamento **Edit** (Modifica) nell'angolo superiore destro della finestra del browser. In una finestra del browser ristretta espandere la barra delle **Opzioni** per visualizzare il collegamento **modifica** . Seguire le istruzioni per creare una richiesta pull (PR). La richiesta pull verrà rivista e accettata oppure verranno suggerite modifiche.

## <a name="how-to-make-a-more-complex-submission"></a>Come effettuare un invio più complesso

Sono necessarie conoscenze di base di [Git e GitHub.com](https://guides.github.com/activities/hello-world/).

* Aprire un [problema](https://github.com/dotnet/AspNetCore.Docs/issues/new) che descrive ciò che si vuole fare, ad esempio modificare un articolo esistente o crearne uno nuovo. Viene spesso richiesta una traccia del documento per i nuovi argomenti suggeriti. Attendere l'approvazione dal team prima di investire molto tempo.
* Creare una copia tramite fork del repository [aspnet/Docs](https://github.com/dotnet/AspNetCore.Docs/) e creare un ramo per le modifiche.
* Inviare una richiesta pull al ramo *principale* con le modifiche apportate.
* Se alla richiesta pull viene assegnata l'etichetta 'cla-required', [sottoscrivere il contratto di licenza per contributi](https://cla.dotnetfoundation.org/).
* Rispondere ai commenti e suggerimenti per la richiesta pull.

Per un esempio in cui questo processo ha portato alla pubblicazione di un nuovo articolo, vedere il [problema 1477](https://github.com/dotnet/docs/issues/1477) e la [richiesta pull 18955](https://github.com/dotnet/docs/pull/18955) nel repository di documentazione .NET. Il nuovo articolo [usa code coverage per gli unit test](/dotnet/core/testing/unit-testing-code-coverage).

## <a name="docs-authoring-pack-extension-in-visual-studio-code"></a>Estensione Docs Authoring Pack in Visual Studio Code

Se si usa Visual Studio Code per contribuire alla documentazione ASP.NET, è possibile diventare più produttivi installando l'estensione [Docs Authoring Pack](https://marketplace.visualstudio.com/items?itemName=docsmsft.docs-authoring-pack). L'estensione offre un'ampia gamma di strumenti utili per il lint di Markdown, il controllo ortografico del codice e i modelli di articolo.

## <a name="markdown-syntax"></a>Sintassi di Markdown

Gli articoli sono scritti nel linguaggio [DocFx Flavored Markdown (DFM)](https://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html), che è un superset di [GitHub Flavored Markdown (GFM)](https://guides.github.com/features/mastering-markdown/). Per esempi della sintassi DFM per le funzionalità dell'interfaccia utente comunemente usate nella documentazione di ASP.NET, vedere il [modello di metadati e Markdown](https://github.com/dotnet/docs/blob/main/styleguide/template.md) nella guida di stile del repository .NET Docs. 

## <a name="folder-structure-conventions"></a>Convenzioni per la struttura di cartelle

Per ogni file markdown può esistere una cartella per le immagini e una cartella per il codice di esempio. Se l'articolo è [fundamentals/configuration/index.md](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/fundamentals/configuration/index.md), le immagini sono in [fundamentals/configuration/index/\_static](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/_static) e i file di progetto dell'app di esempio sono in [fundamentals/configuration/index/sample](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/index/sample). Il rendering di un'immagine nel file *fundamentals/configuration/index.md* viene eseguito dal codice Markdown seguente:

```md
![description of image for alt attribute](configuration/index/_static/imagename.png)
```

Tutte le immagini devono avere il [testo alternativo (alt)](https://wikipedia.org/wiki/Alt_attribute). Per consigli su come specificare il testo alternativo, vedere le risorse online, ad esempio [WebAIM: Alternative Text](https://webaim.org/techniques/alttext/) (WebAIM: testo alternativo).

Usare lettere minuscole per i nomi di file markdown e i nomi di file di immagine.

## <a name="internal-links"></a>Collegamenti interni

I collegamenti interni devono usare l'`uid` dell'articolo di destinazione con un collegamento xref (il testo del collegamento è impostato sul titolo del contenuto collegato):

```md
<xref:uid_of_the_topic>
```

Se il titolo dell'articolo non è idoneo per il testo del collegamento (ad esempio, il testo del collegamento è rappresentato da una parola o una locuzione in una frase), specificare il collegamento xref e il testo del collegamento come segue:

```md
[link text](xref:uid_of_the_topic)
```

Per altre informazioni, vedere [DocFX Cross Reference](https://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html#cross-reference) (Riferimenti incrociati DocFX).

## <a name="images-and-screenshots"></a>Immagini e screenshot

Non includere immagini negli articoli, ad eccezione dei seguenti casi:

* Nelle esercitazioni di onboarding (per principianti).
* Quando un'immagine è necessaria per garantire maggior chiarezza.

Queste limitazioni consentono di ridurre le dimensioni del repository.

Come passaggio facoltativo, assicurarsi che eventuali immagini e screenshot usati nella documentazione siano compressi, perché ciò serve per le dimensioni dei file e le prestazioni di caricamento delle pagine. Alcuni strumenti noti sono TinyPNG (usando il [sito Web TinyPNG](https://tinypng.com/) o l'[API TinyPNG](https://tinypng.com/developers)) o l'estensione di Visual Studio [Image Optimizer](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.ImageOptimizer). 

## <a name="code-snippets"></a>Frammenti di codice

Gli articoli contengono spesso frammenti di codice per illustrare gli argomenti discussi. DFM consente di copiare codice nel file markdown o di fare riferimento a un file di codice separato. È preferibile usare i file di codice separati quando possibile, per ridurre al minimo la probabilità di errori nel codice. I file di codice vengono archiviati nel repository usando la struttura di cartelle descritta in precedenza per i progetti di esempio. 

Gli esempi seguenti illustrano la [sintassi per i frammenti di codice DFM](https://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html#code-snippet) da usare in un file *configuration/index.md*.

Per eseguire il rendering di un intero file di codice come un frammento:

```md
[!code-csharp[](configuration/index/sample/Program.cs)]
```

Per eseguire il rendering di una parte di un file come un frammento usando i numeri di riga:

```md
[!code-csharp[](configuration/index/sample/Program.cs?range=1-10,20,30,40-50]
[!code-html[](configuration/index/sample/Views/Home/Index.cshtml?range=1-10,20,30,40-50]
```

Per i frammenti C#, fare riferimento a un'[area C#](https://docs.microsoft.com/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-region). Quando possibile, usare le aree anziché i numeri di riga, perché i numeri di riga in un file di codice tendono a cambiare e perdere la sincronizzazione con i riferimenti ai numeri di riga in Markdown. Le aree C# possono essere annidate. Se fanno riferimento all'area esterna, le direttive `#region` e `#endregion` interne non vengono incluse nel rendering in un frammento. 

Per eseguire il rendering di un'area C# denominata "snippet_Example":

```md
[!code-csharp[](configuration/index/sample/Program.cs?name=snippet_Example)]
```

Per evidenziare le righe selezionate in un frammento di codice sottoposto a rendering (in genere il rendering viene eseguito con un colore di sfondo giallo):

```md
[!code-csharp[](configuration/index/sample/Program.cs?name=snippet_Example&highlight=1-3,10,20-25)]
[!code-csharp[](configuration/index/sample/Program.cs?range=10-20&highlight=1-3]
[!code-html[](configuration/index/sample/Views/Home/Index.cshtml?range=10-20&highlight=1-3]
[!code-javascript[](configuration/index/sample/UsingOptionsSample.csproj?range=10-20&highlight=1-3]
```

## <a name="test-changes-with-docfx"></a>Testare le modifiche con DocFX

Testare le modifiche con lo [strumento da riga di comando DocFX](https://dotnet.github.io/docfx/tutorial/docfx_getting_started.html#2-use-docfx-as-a-command-line-tool), che consente di creare una versione ospitata in locale del sito. DocFX non esegue il rendering delle estensioni di stile e del sito create per docs.microsoft.com.

DocFX richiede:

* .NET Framework su Windows.
* Mono per Linux o macOS. 

### <a name="windows-instructions"></a>Istruzioni per Windows

* Scaricare e decomprimere *docfx.zip* da [DocFX releases](https://github.com/dotnet/docfx/releases) (Versioni di DocFX).
* Aggiungere DocFX a PATH.
* In una shell di comando, passare alla cartella che contiene il file *docfx.json* (*aspnet* per il contenuto ASP.NET o *aspnetcore* per il contenuto ASP.NET Core) ed eseguire il comando seguente:

  ```console
  docfx --serve
  ```

* In un browser passare a `http://localhost:8080/group1-dest/`.

### <a name="mono-instructions"></a>Istruzioni per Mono

* Installare Mono tramite Homebrew:

  ```console
  brew install mono
  ```

* Scarica la [versione più recente di DocFX](https://github.com/dotnet/docfx/releases).
* Estrarre l'archivio in *$HOME/bin/docfx*.
* Creare una coppia di alias per **docfx** in una shell Bash. Il primo alias viene usato per creare la documentazione. Il secondo alias viene usato per creare e gestire la documentazione.

  ```console
  alias docfx='mono $HOME/bin/docfx/docfx.exe'
  alias docfx-serve='mono $HOME/bin/docfx/docfx.exe --serve'
  ```

* In una shell di comando, passare alla cartella che contiene il file *docfx.json* (*aspnet* per il contenuto ASP.NET o *aspnetcore* per il contenuto ASP.NET Core) ed eseguire il comando seguente per compilare e fornire la documentazione tramite il relativo alias:

  ```console
  docfx-serve
  ```

* In un browser passare a `http://localhost:8080/group1-dest/`.

## <a name="voice-and-tone"></a>Voce e tono

L'obiettivo è quello di scrivere documentazione facilmente comprensibile per il più ampio pubblico possibile. A tale scopo, sono state definite linee guida per lo stile di scrittura che i collaboratori dovranno seguire. Per altre informazioni, vedere [Linee guida per la voce e il tono](https://github.com/dotnet/docs/blob/main/styleguide/voice-tone.md) nel repository .NET.

## <a name="microsoft-writing-style-guide"></a>Guida di stile di scrittura Microsoft

La [Microsoft Writing Style Guide](https://docs.microsoft.com/style-guide/welcome/) (Guida di stile di scrittura Microsoft) include linee guida per lo stile di scrittura e la terminologia per tutte le forme di comunicazione per argomenti tecnologici, inclusa la documentazione di ASP.NET Core.

## <a name="redirects"></a>Reindirizzamenti

Se si elimina un articolo, si modifica il nome di file o lo si sposta in un'altra cartella, creare un reindirizzamento in modo che le persone che hanno contrassegnato l'articolo con un segnalibro non ricevano l'errore *404 Non trovato*. Aggiungere i reindirizzamenti al [file di reindirizzamento principale](https://github.com/dotnet/AspNetCore.Docs/blob/main/.openpublishing.redirection.json).
