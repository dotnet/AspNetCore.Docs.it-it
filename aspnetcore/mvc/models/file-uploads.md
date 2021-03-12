---
title: Caricare file in ASP.NET Core
author: rick-anderson
description: Come usare l'associazione del modello e lo streaming per caricare i file in ASP.NET Core MVC.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/21/2020
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
uid: mvc/models/file-uploads
ms.openlocfilehash: 90bde63ac94ba3fd29a067962989cf773ec613db
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587239"
---
# <a name="upload-files-in-aspnet-core"></a>Caricare file in ASP.NET Core

Di [Steve Smith](https://ardalis.com/) e [Rutger Storm](https://github.com/rutix)

::: moniker range=">= aspnetcore-5.0"

ASP.NET Core supporta il caricamento di uno o più file mediante l'associazione di modelli memorizzati nel buffer per i file più piccoli e il flusso non memorizzato nel buffer per file di dimensioni maggiori.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="security-considerations"></a>Considerazioni relative alla sicurezza

Prestare attenzione quando si fornisce agli utenti la possibilità di caricare file in un server. Gli utenti malintenzionati possono tentare di:

* Eseguire attacchi [di tipo Denial of Service](/windows-hardware/drivers/ifs/denial-of-service) .
* Caricare virus o malware.
* Compromettere le reti e i server in altri modi.

I passaggi di sicurezza che riducono la probabilità di un attacco riuscito sono:

* Caricare i file in un'area di caricamento file dedicata, preferibilmente in un'unità non di sistema. Un percorso dedicato semplifica l'imposizione delle restrizioni di sicurezza sui file caricati. Disabilitare le autorizzazioni di esecuzione per il percorso di caricamento del file.&dagger;
* Non **rende permanente i** file caricati nella stessa struttura di directory dell'app.&dagger;
* Usare un nome file sicuro determinato dall'app. Non usare un nome file fornito dall'utente o il nome file non attendibile del file caricato. &dagger; HTML codifica il nome file non attendibile quando viene visualizzato. Ad esempio, registrare il nome del file o visualizzarlo nell'interfaccia utente (l' Razor output viene codificato automaticamente in HTML).
* Consenti solo le estensioni di file approvate per la specifica di progettazione dell'app.&dagger; <!-- * Check the file format signature to prevent a user from uploading a masqueraded file.&dagger; For example, don't permit a user to upload an *.exe* file with a *.txt* extension. Add this back when we get instructions how to do this.  -->
* Verificare che i controlli lato client vengano eseguiti sul server. &dagger; I controlli lato client sono facili da aggirare.
* Controllare le dimensioni di un file caricato. Impostare un limite di dimensioni massime per impedire caricamenti di grandi dimensioni.&dagger;
* Quando i file non devono essere sovrascritti da un file caricato con lo stesso nome, controllare il nome del file sul database o sull'archiviazione fisica prima di caricare il file.
* **Eseguire uno scanner virus/malware sul contenuto caricato prima che il file venga archiviato.**

&dagger;L'app di esempio illustra un approccio che soddisfa i criteri.

> [!WARNING]
> Il caricamento di codice dannoso in un sistema è spesso il primo passaggio per l'esecuzione di codice in grado di:
>
> * Ottenere il controllo completo di un sistema.
> * Sovraccaricare un sistema con il risultato che si verifica un arresto anomalo del sistema.
> * Compromettere i dati di sistemi o utenti.
> * Applicare i graffiti a un'interfaccia utente pubblica.
>
> Per informazioni sulla riduzione della superficie di attacco quando si accettano file dagli utenti, vedere le risorse seguenti:
>
> * [Caricamento di file senza restrizioni](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
> * [Protezione di Azure: Verificare i controlli appropriati quando si accettano file dagli utenti](/azure/security/azure-security-threat-modeling-tool-input-validation#controls-users)

Per ulteriori informazioni sull'implementazione di misure di sicurezza, inclusi esempi dall'app di esempio, vedere la sezione [convalida](#validation) .

## <a name="storage-scenarios"></a>Scenari di archiviazione

Le opzioni di archiviazione comuni per i file includono:

* Database

  * Per i caricamenti di file di piccole dimensioni, un database è spesso più veloce rispetto alle opzioni di archiviazione fisica (file system o condivisione di rete).
  * Un database è spesso più pratico delle opzioni di archiviazione fisica perché il recupero di un record di database per i dati utente può fornire simultaneamente il contenuto del file, ad esempio un'immagine avatar.
  * Un database è potenzialmente meno costoso rispetto all'utilizzo di un servizio di archiviazione dati.

* Archiviazione fisica (file system o condivisione di rete)

  * Per i caricamenti di file di grandi dimensioni:
    * I limiti del database possono limitare le dimensioni del caricamento.
    * L'archiviazione fisica è spesso meno economica rispetto all'archiviazione in un database.
  * L'archiviazione fisica è potenzialmente meno costosa rispetto all'utilizzo di un servizio di archiviazione dati.
  * Il processo dell'app deve avere le autorizzazioni di lettura e scrittura per il percorso di archiviazione. **Non concedere mai l'autorizzazione Execute.**

* Servizio di archiviazione dei dati (ad esempio, [archiviazione BLOB di Azure](https://azure.microsoft.com/services/storage/blobs/))

  * I servizi in genere offrono una maggiore scalabilità e resilienza rispetto a soluzioni locali che in genere sono soggette a singoli punti di errore.
  * I servizi sono un costo potenzialmente inferiore in scenari di infrastruttura di archiviazione di grandi dimensioni.

  Per altre informazioni, vedere [Guida introduttiva: usare .NET per creare un BLOB nell'archivio oggetti](/azure/storage/blobs/storage-quickstart-blobs-dotnet).

## <a name="file-upload-scenarios"></a>Scenari di caricamento di file

Due approcci generali per il caricamento di file sono il buffering e il flusso.

**responseBuffering**

L'intero file viene letto in un oggetto <xref:Microsoft.AspNetCore.Http.IFormFile> , ovvero una rappresentazione C# del file usato per elaborare o salvare il file.

Le risorse (disco, memoria) utilizzate dai caricamenti di file dipendono dal numero e dalle dimensioni dei caricamenti di file simultanei. Se un'app tenta di memorizzare nel buffer un numero eccessivo di caricamenti, il sito si arresta in modo anomalo quando esaurisce la memoria o lo spazio su disco. Se le dimensioni o la frequenza dei caricamenti di file esauriscono le risorse dell'app, usare il flusso.

> [!NOTE]
> Ogni singolo file memorizzato nel buffer che supera 64 KB viene spostato dalla memoria in un file temporaneo su disco.

Il buffering di file di piccole dimensioni viene trattato nelle sezioni seguenti di questo argomento:

* [Archiviazione fisica](#upload-small-files-with-buffered-model-binding-to-physical-storage)
* [Database](#upload-small-files-with-buffered-model-binding-to-a-database)

**Streaming**

Il file viene ricevuto da una richiesta multipart ed elaborato o salvato direttamente dall'app. Il flusso non migliora significativamente le prestazioni. Il flusso riduce le richieste di memoria o di spazio su disco durante il caricamento dei file.

Il flusso di file di grandi dimensioni è trattato nella sezione [caricare file di grandi dimensioni con flusso](#upload-large-files-with-streaming) .

### <a name="upload-small-files-with-buffered-model-binding-to-physical-storage"></a>Caricare file di piccole dimensioni con associazione di modelli memorizzati nel buffer all'archiviazione fisica

Per caricare file di piccole dimensioni, usare un modulo multipart o creare una richiesta POST usando JavaScript.

L'esempio seguente illustra l'uso di un Razor form di pagine per caricare un singolo file (*pages/BufferedSingleFileUploadPhysical. cshtml* nell'app di esempio):

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
            <span asp-validation-for="FileUpload.FormFile"></span>
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload" />
</form>
```

L'esempio seguente è analogo all'esempio precedente, ad eccezione del fatto che:

* Per inviare i dati del modulo, viene usato JavaScript ([Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API)).
* Nessuna convalida.

```cshtml
<form action="BufferedSingleFileUploadPhysical/?handler=Upload" 
      enctype="multipart/form-data" onsubmit="AJAXSubmit(this);return false;" 
      method="post">
    <dl>
        <dt>
            <label for="FileUpload_FormFile">File</label>
        </dt>
        <dd>
            <input id="FileUpload_FormFile" type="file" 
                name="FileUpload.FormFile" />
        </dd>
    </dl>

    <input class="btn" type="submit" value="Upload" />

    <div style="margin-top:15px">
        <output name="result"></output>
    </div>
</form>

<script>
  async function AJAXSubmit (oFormElement) {
    var resultElement = oFormElement.elements.namedItem("result");
    const formData = new FormData(oFormElement);

    try {
    const response = await fetch(oFormElement.action, {
      method: 'POST',
      body: formData
    });

    if (response.ok) {
      window.location.href = '/';
    }

    resultElement.value = 'Result: ' + response.status + ' ' + 
      response.statusText;
    } catch (error) {
      console.error('Error:', error);
    }
  }
</script>
```

Per eseguire il POST del form in JavaScript per i client che [non supportano l'API fetch](https://caniuse.com/#feat=fetch), usare uno degli approcci seguenti:

* Usare un riempimento di recupero (ad esempio, [Window. fetch Refill (github/fetch)](https://github.com/github/fetch)).
* Usare `XMLHttpRequest`. Ad esempio:

  ```javascript
  <script>
    "use strict";

    function AJAXSubmit (oFormElement) {
      var oReq = new XMLHttpRequest();
      oReq.onload = function(e) { 
      oFormElement.elements.namedItem("result").value = 
        'Result: ' + this.status + ' ' + this.statusText;
      };
      oReq.open("post", oFormElement.action);
      oReq.send(new FormData(oFormElement));
    }
  </script>
  ```

Per supportare il caricamento di file, i moduli HTML devono specificare un tipo di codifica ( `enctype` ) di `multipart/form-data` .

Affinché un `files` elemento input supporti il caricamento di più file, fornire l' `multiple` attributo nell' `<input>` elemento:

```cshtml
<input asp-for="FileUpload.FormFiles" type="file" multiple>
```

È possibile accedere ai singoli file caricati nel server tramite l' [associazione di modelli](xref:mvc/models/model-binding) tramite <xref:Microsoft.AspNetCore.Http.IFormFile> . L'app di esempio illustra più caricamenti di file memorizzati nel buffer per scenari di archiviazione di database e fisici.

<a name="filename"></a>

> [!WARNING]
> Non **utilizzare la** `FileName` proprietà di <xref:Microsoft.AspNetCore.Http.IFormFile> diverso da per la visualizzazione e la registrazione. Durante la visualizzazione o la registrazione, HTML codifica il nome del file. Un utente malintenzionato può fornire un nome file dannoso, inclusi percorsi completi o percorsi relativi. Le applicazioni devono:
>
> * Rimuovere il percorso dal nome file specificato dall'utente.
> * Salvare il nome file con codifica HTML, percorso-rimosso per l'interfaccia utente o la registrazione.
> * Genera un nuovo nome file casuale per l'archiviazione.
>
> Il codice seguente rimuove il percorso dal nome del file:
>
> ```csharp
> string untrustedFileName = Path.GetFileName(pathName);
> ```
>
> Gli esempi forniti finora non prendono in considerazione le considerazioni sulla sicurezza. Le informazioni aggiuntive sono fornite nelle sezioni seguenti e nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/):
>
> * [Considerazioni sulla sicurezza](#security-considerations)
> * [Convalida](#validation)

Quando si caricano file usando l'associazione di modelli e <xref:Microsoft.AspNetCore.Http.IFormFile> , il metodo di azione può accettare:

* Singolo oggetto <xref:Microsoft.AspNetCore.Http.IFormFile> .
* Qualsiasi raccolta seguente che rappresenta diversi file:
  * <xref:Microsoft.AspNetCore.Http.IFormFileCollection>
  * <xref:System.Collections.IEnumerable>\<<xref:Microsoft.AspNetCore.Http.IFormFile>>
  * [Elenco](xref:System.Collections.Generic.List`1)\<<xref:Microsoft.AspNetCore.Http.IFormFile>>

> [!NOTE]
> L'associazione corrisponde ai file di form in base al nome. Ad esempio, il `name` valore HTML in `<input type="file" name="formFile">` deve corrispondere al parametro/proprietà di C# associato ( `FormFile` ). Per ulteriori informazioni, vedere la sezione [corrispondenza del nome dell'attributo del nome del parametro del metodo post](#match-name-attribute-value-to-parameter-name-of-post-method) .

L'esempio seguente:

* Esegue il ciclo di uno o più file caricati.
* USA [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) per restituire un percorso completo per un file, incluso il nome del file. 
* Salva i file nella file system locale usando un nome file generato dall'app.
* Restituisce il numero totale e le dimensioni dei file caricati.

```csharp
public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> files)
{
    long size = files.Sum(f => f.Length);

    foreach (var formFile in files)
    {
        if (formFile.Length > 0)
        {
            var filePath = Path.GetTempFileName();

            using (var stream = System.IO.File.Create(filePath))
            {
                await formFile.CopyToAsync(stream);
            }
        }
    }

    // Process uploaded files
    // Don't rely on or trust the FileName property without validation.

    return Ok(new { count = files.Count, size });
}
```

Utilizzare `Path.GetRandomFileName` per generare un nome file senza percorso. Nell'esempio seguente il percorso viene ottenuto dalla configurazione:

```csharp
foreach (var formFile in files)
{
    if (formFile.Length > 0)
    {
        var filePath = Path.Combine(_config["StoredFilesPath"], 
            Path.GetRandomFileName());

        using (var stream = System.IO.File.Create(filePath))
        {
            await formFile.CopyToAsync(stream);
        }
    }
}
```

Il percorso passato a <xref:System.IO.FileStream> *deve* includere il nome del file. Se il nome file non viene specificato, <xref:System.UnauthorizedAccessException> viene generata un'eccezione in fase di esecuzione.

I file caricati con la <xref:Microsoft.AspNetCore.Http.IFormFile> tecnica vengono memorizzati nel buffer in memoria o su disco sul server prima dell'elaborazione. All'interno del metodo di azione, il <xref:Microsoft.AspNetCore.Http.IFormFile> contenuto è accessibile come <xref:System.IO.Stream> . Oltre al file system locale, i file possono essere salvati in una condivisione di rete o in un servizio di archiviazione file, ad esempio [archiviazione BLOB di Azure](/azure/visual-studio/vs-storage-aspnet5-getting-started-blobs).

Per un altro esempio che esegue il ciclo su più file per il caricamento e usa nomi di file sicuri, vedere *pages/BufferedMultipleFileUploadPhysical. cshtml. cs* nell'app di esempio.

> [!WARNING]
> [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) genera un'eccezione <xref:System.IO.IOException> se vengono creati più di 65.535 file senza eliminare i file temporanei precedenti. Il limite di 65.535 file è un limite per server. Per ulteriori informazioni su questo limite per il sistema operativo Windows, vedere la sezione Osservazioni negli argomenti seguenti:
>
> * [GetTempFileNameA (funzione)](/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea#remarks)
> * <xref:System.IO.Path.GetTempFileName*>

### <a name="upload-small-files-with-buffered-model-binding-to-a-database"></a>Caricare file di piccole dimensioni con associazione di modelli memorizzati nel buffer in un database

Per archiviare i dati dei file binari in un database utilizzando [Entity Framework](/ef/core/index), definire una <xref:System.Byte> proprietà di matrice nell'entità:

```csharp
public class AppFile
{
    public int Id { get; set; }
    public byte[] Content { get; set; }
}
```

Specificare una proprietà del modello di pagina per la classe che include <xref:Microsoft.AspNetCore.Http.IFormFile> :

```csharp
public class BufferedSingleFileUploadDbModel : PageModel
{
    ...

    [BindProperty]
    public BufferedSingleFileUploadDb FileUpload { get; set; }

    ...
}

public class BufferedSingleFileUploadDb
{
    [Required]
    [Display(Name="File")]
    public IFormFile FormFile { get; set; }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Http.IFormFile> può essere usato direttamente come parametro del metodo di azione o come proprietà del modello associato. Nell'esempio precedente viene utilizzata una proprietà del modello associato.

Il `FileUpload` viene utilizzato nel Razor form pagine:

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload">
</form>
```

Quando il modulo viene inviato al server, copiarlo in <xref:Microsoft.AspNetCore.Http.IFormFile> un flusso e salvarlo come una matrice di byte nel database. Nell'esempio seguente, `_dbContext` archivia il contesto di database dell'app:

```csharp
public async Task<IActionResult> OnPostUploadAsync()
{
    using (var memoryStream = new MemoryStream())
    {
        await FileUpload.FormFile.CopyToAsync(memoryStream);

        // Upload the file if less than 2 MB
        if (memoryStream.Length < 2097152)
        {
            var file = new AppFile()
            {
                Content = memoryStream.ToArray()
            };

            _dbContext.File.Add(file);

            await _dbContext.SaveChangesAsync();
        }
        else
        {
            ModelState.AddModelError("File", "The file is too large.");
        }
    }

    return Page();
}
```

L'esempio precedente è simile a uno scenario illustrato nell'app di esempio:

* *Pages/BufferedSingleFileUploadDb. cshtml*
* *Pages/BufferedSingleFileUploadDb. cshtml. cs*

> [!WARNING]
> Quando si archiviano dati binari all'interno di database relazionali, è necessario prestare attenzione, perché l'operazione può influire negativamente sulle prestazioni.
>
> Non fare affidamento o considerare attendibile la `FileName` proprietà di <xref:Microsoft.AspNetCore.Http.IFormFile> senza convalida. La `FileName` proprietà deve essere usata solo a scopo di visualizzazione e solo dopo la codifica HTML.
>
> Gli esempi forniti non prendono in considerazione le considerazioni sulla sicurezza. Le informazioni aggiuntive sono fornite nelle sezioni seguenti e nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/):
>
> * [Considerazioni sulla sicurezza](#security-considerations)
> * [Convalida](#validation)

### <a name="upload-large-files-with-streaming"></a>Caricare file di grandi dimensioni con flusso

Nell'esempio seguente viene illustrato come utilizzare JavaScript per trasmettere un file a un'azione del controller. Il token antifalsificazione del file viene generato usando un attributo di filtro personalizzato e passato alle intestazioni HTTP del client anziché nel corpo della richiesta. Poiché il metodo di azione elabora direttamente i dati caricati, l'associazione del modello di modulo viene disabilitata da un altro filtro personalizzato. All'interno dell'azione, il contenuto del form viene letto tramite un `MultipartReader`, che legge ogni singola `MultipartSection`, elaborando il file o archiviandone il contenuto, come appropriato. Una volta lette le sezioni multiparte, l'azione esegue una propria associazione di modelli.

La risposta della pagina iniziale carica il form e salva un token antifalsificazione in un cookie (tramite l' `GenerateAntiforgeryTokenCookieAttribute` attributo). L'attributo usa il [supporto antifalsificazione](xref:security/anti-request-forgery) predefinito di ASP.NET Core per impostare un cookie con un token di richiesta:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/Antiforgery.cs?name=snippet_GenerateAntiforgeryTokenCookieAttribute)]

`DisableFormValueModelBindingAttribute`Viene usato per disabilitare l'associazione di modelli:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/ModelBinding.cs?name=snippet_DisableFormValueModelBindingAttribute)]

Nell'app di esempio `GenerateAntiforgeryTokenCookieAttribute` e `DisableFormValueModelBindingAttribute` vengono applicati come filtri ai modelli di applicazione della pagina di `/StreamedSingleFileUploadDb` e `/StreamedSingleFileUploadPhysical` all' `Startup.ConfigureServices` uso delle [ Razor convenzioni delle pagine](xref:razor-pages/razor-pages-conventions):

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Startup.cs?name=snippet_AddRazorPages&highlight=7-10,16-19)]

Poiché l'associazione di modelli non legge il modulo, i parametri associati al modulo non vengono associati (la query, la route e l'intestazione continuano a funzionare). Il metodo di azione funziona direttamente con la `Request` Proprietà. Per leggere ogni sezione, viene usato un `MultipartReader`. I dati chiave/valore vengono archiviati in un oggetto `KeyValueAccumulator` . Una volta lette le sezioni multipart, il contenuto di `KeyValueAccumulator` viene utilizzato per associare i dati del modulo a un tipo di modello.

Metodo completo `StreamingController.UploadDatabase` per lo streaming in un database con EF Core:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadDatabase)]

`MultipartRequestHelper` (*Utilities/MultipartRequestHelper. cs*):

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Utilities/MultipartRequestHelper.cs)]

Il `StreamingController.UploadPhysical` metodo completo per lo streaming in una posizione fisica:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadPhysical)]

Nell'app di esempio, i controlli di convalida vengono gestiti da `FileHelpers.ProcessStreamedFile` .

## <a name="validation"></a>Convalida

La classe dell'app di esempio `FileHelpers` illustra alcuni controlli per i caricamenti di file memorizzati nel buffer <xref:Microsoft.AspNetCore.Http.IFormFile> e in flusso. Per l'elaborazione dei <xref:Microsoft.AspNetCore.Http.IFormFile> caricamenti di file memorizzati nel buffer nell'app di esempio, vedere il `ProcessFormFile` metodo nel file *Utilities/FileHelpers. cs* . Per l'elaborazione di file trasmessi, vedere il `ProcessStreamedFile` Metodo nello stesso file.

> [!WARNING]
> I metodi di elaborazione della convalida illustrati nell'app di esempio non analizzano il contenuto dei file caricati. Nella maggior parte degli scenari di produzione, nel file viene usata un'API scanner di virus/malware prima di rendere disponibile il file per gli utenti o altri sistemi.
>
> Sebbene l'esempio di argomento fornisca un esempio funzionante di tecniche di convalida, non implementare la `FileHelpers` classe in un'app di produzione a meno che non si:
>
> * Comprendere completamente l'implementazione.
> * Modificare l'implementazione in modo appropriato per l'ambiente e le specifiche dell'app.
>
> **Non implementare mai indiscriminatamente il codice di sicurezza in un'app senza soddisfare questi requisiti.**

### <a name="content-validation"></a>Convalida contenuto

**Usare un'API di analisi di virus/malware di terze parti nel contenuto caricato.**

L'analisi dei file richiede le risorse del server in scenari con volumi elevati. Se le prestazioni di elaborazione delle richieste sono diminuite a causa dell'analisi dei file, provare a eseguire l'offload del lavoro di analisi in un [servizio in background](xref:fundamentals/host/hosted-services), possibilmente in un servizio in esecuzione in un server diverso dal server dell'app. In genere, i file caricati vengono mantenuti in un'area in quarantena finché non vengono controllati dal programma antivirus in background. Quando un file passa, il file viene spostato nel percorso di archiviazione file normale. Questi passaggi vengono in genere eseguiti insieme a un record di database che indica lo stato di analisi di un file. Usando un approccio di questo tipo, l'app e il server app rimangono incentrati sulla risposta alle richieste.

### <a name="file-extension-validation"></a>Convalida dell'estensione di file

L'estensione del file caricato deve essere verificata rispetto a un elenco di estensioni consentite. Ad esempio:

```csharp
private string[] permittedExtensions = { ".txt", ".pdf" };

var ext = Path.GetExtension(uploadedFileName).ToLowerInvariant();

if (string.IsNullOrEmpty(ext) || !permittedExtensions.Contains(ext))
{
    // The extension is invalid ... discontinue processing the file
}
```

### <a name="file-signature-validation"></a>Convalida della firma del file

La firma di un file è determinata dai primi byte all'inizio di un file. Questi byte possono essere usati per indicare se l'estensione corrisponde al contenuto del file. L'app di esempio controlla le firme dei file per alcuni tipi di file comuni. Nell'esempio seguente, la firma del file per un'immagine JPEG viene verificata rispetto al file:

```csharp
private static readonly Dictionary<string, List<byte[]>> _fileSignature = 
    new Dictionary<string, List<byte[]>>
{
    { ".jpeg", new List<byte[]>
        {
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 },
        }
    },
};

using (var reader = new BinaryReader(uploadedFileData))
{
    var signatures = _fileSignature[ext];
    var headerBytes = reader.ReadBytes(signatures.Max(m => m.Length));
    
    return signatures.Any(signature => 
        headerBytes.Take(signature.Length).SequenceEqual(signature));
}
```

Per ottenere ulteriori firme di file, vedere il [database delle firme file](https://www.filesignatures.net/) e le specifiche del file ufficiale.

### <a name="file-name-security"></a>Sicurezza del nome file

Non usare mai un nome file fornito dal client per salvare un file nell'archiviazione fisica. Creare un nome file sicuro per il file usando [Path. GetRandomFileName](xref:System.IO.Path.GetRandomFileName*) o [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) per creare un percorso completo (incluso il nome file) per l'archiviazione temporanea.

Razor codifica HTML automaticamente i valori delle proprietà per la visualizzazione. Il codice seguente è sicuro da usare:

```cshtml
@foreach (var file in Model.DatabaseFiles) {
    <tr>
        <td>
            @file.UntrustedName
        </td>
    </tr>
}
```

Al di fuori di Razor , sempre il <xref:System.Net.WebUtility.HtmlEncode*> contenuto del nome file da una richiesta dell'utente.

Molte implementazioni devono includere un controllo per verificare che il file esista. in caso contrario, il file viene sovrascritto da un file con lo stesso nome. Fornire logica aggiuntiva per soddisfare le specifiche dell'app.

### <a name="size-validation"></a>Convalida dimensioni

Limitare le dimensioni dei file caricati.

Nell'app di esempio, le dimensioni del file sono limitate a 2 MB (indicate in byte). Il limite viene fornito tramite la [configurazione](xref:fundamentals/configuration/index) dal *appsettings.json* file:

```json
{
  "FileSizeLimit": 2097152
}
```

`FileSizeLimit`Viene inserito nelle `PageModel` classi:

```csharp
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    private readonly long _fileSizeLimit;

    public BufferedSingleFileUploadPhysicalModel(IConfiguration config)
    {
        _fileSizeLimit = config.GetValue<long>("FileSizeLimit");
    }

    ...
}
```

Quando le dimensioni di un file superano il limite, il file viene rifiutato:

```csharp
if (formFile.Length > _fileSizeLimit)
{
    // The file is too large ... discontinue processing the file
}
```

### <a name="match-name-attribute-value-to-parameter-name-of-post-method"></a>Corrisponde al valore dell'attributo Name al nome del parametro del metodo POST

In non Razor form che inviano dati del modulo o usano direttamente il nome di JavaScript `FormData` , il nome specificato nell'elemento del form o `FormData` deve corrispondere al nome del parametro nell'azione del controller.

Nell'esempio seguente:

* Quando si usa un `<input>` elemento, l' `name` attributo viene impostato sul valore `battlePlans` :

  ```html
  <input type="file" name="battlePlans" multiple>
  ```

* Quando `FormData` si usa in JavaScript, il nome viene impostato sul valore `battlePlans` :

  ```javascript
  var formData = new FormData();

  for (var file in files) {
    formData.append("battlePlans", file, file.name);
  }
  ```

Usare un nome corrispondente per il parametro del metodo C# ( `battlePlans` ):

* Per un Razor metodo gestore pagina pagine denominato `Upload` :

  ```csharp
  public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> battlePlans)
  ```

* Per un metodo di azione POST controller MVC:

  ```csharp
  public async Task<IActionResult> Post(List<IFormFile> battlePlans)
  ```

## <a name="server-and-app-configuration"></a>Configurazione del server e dell'app

### <a name="multipart-body-length-limit"></a>Limite di lunghezza del corpo multipart

<xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> imposta il limite per la lunghezza di ogni corpo multipart. Le sezioni del modulo che superano questo limite generano un'espressione <xref:System.IO.InvalidDataException> quando vengono analizzate. Il valore predefinito è 134.217.728 (128 MB). Personalizzare il limite usando l' <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> impostazione in `Startup.ConfigureServices` :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        // Set the limit to 256 MB
        options.MultipartBodyLengthLimit = 268435456;
    });
}
```

<xref:Microsoft.AspNetCore.Mvc.RequestFormLimitsAttribute> viene usato per impostare <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> per una singola pagina o un'azione.

In un' Razor app pagine, applicare il filtro con una [convenzione](xref:razor-pages/razor-pages-conventions) in `Startup.ConfigureServices` :

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model.Filters.Add(
                new RequestFormLimitsAttribute()
                {
                    // Set the limit to 256 MB
                    MultipartBodyLengthLimit = 268435456
                });
});
```

In un' Razor app pagine o in un'app MVC, applicare il filtro al modello di pagina o al metodo di azione:

```csharp
// Set the limit to 256 MB
[RequestFormLimits(MultipartBodyLengthLimit = 268435456)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="kestrel-maximum-request-body-size"></a>Dimensioni massime del corpo della richiesta di gheppio

Per le app ospitate da gheppio, le dimensioni massime predefinite del corpo della richiesta sono pari a 30 milioni byte, ovvero circa 28,6 MB. Personalizzare il limite usando l'opzione del server [MaxRequestBodySize](xref:fundamentals/servers/kestrel/options#maximum-request-body-size) gheppio:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel((context, options) =>
            {
                // Handle requests up to 50 MB
                options.Limits.MaxRequestBodySize = 52428800;
            })
            .UseStartup<Startup>();
        });
```

<xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> viene usato per impostare [MaxRequestBodySize](xref:fundamentals/servers/kestrel/options#maximum-request-body-size) per una singola pagina o un'azione.

In un' Razor app pagine, applicare il filtro con una [convenzione](xref:razor-pages/razor-pages-conventions) in `Startup.ConfigureServices` :

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model =>
            {
                // Handle requests up to 50 MB
                model.Filters.Add(
                    new RequestSizeLimitAttribute(52428800));
            });
});
```

In un' Razor app pagine o in un'app MVC, applicare il filtro alla classe del gestore di pagina o al metodo di azione:

```csharp
// Handle requests up to 50 MB
[RequestSizeLimit(52428800)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

Il `RequestSizeLimitAttribute` può essere applicato anche usando la [`@attribute`](xref:mvc/views/razor#attribute) Razor direttiva:

```cshtml
@attribute [RequestSizeLimitAttribute(52428800)]
```

### <a name="other-kestrel-limits"></a>Altri limiti del gheppio

Per le app ospitate da gheppio possono essere applicati altri limiti di Gheppio:

* [Numero massimo di connessioni client](xref:fundamentals/servers/kestrel/options#maximum-client-connections)
* [Frequenza dei dati di richiesta e risposta](xref:fundamentals/servers/kestrel/options#minimum-request-body-data-rate)

### <a name="iis"></a>IIS

Il limite di richieste predefinito ( `maxAllowedContentLength` ) è 30 milioni byte, che corrisponde a circa 28,6 MB. Personalizzare il limite nel `web.config` file. Nell'esempio seguente il limite è impostato su 50 MB (52.428.800 byte):

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

L' `maxAllowedContentLength` impostazione si applica solo a IIS. Per altre informazioni, vedere [limiti `<requestLimits>` della richiesta ](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).

## <a name="troubleshoot"></a>Risolvere problemi

Di seguito sono trattati alcuni problemi comuni riscontrati durante il caricamento di file, con le soluzioni possibili corrispondenti.

### <a name="not-found-error-when-deployed-to-an-iis-server"></a>Errore non trovato quando viene distribuito in un server IIS

L'errore seguente indica che il file caricato supera la lunghezza del contenuto configurata del server:

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

Per ulteriori informazioni, vedere la sezione [IIS](#iis) .

### <a name="connection-failure"></a>Errore di connessione

Un errore di connessione e una connessione al server di reimpostazione probabilmente indicano che il file caricato supera le dimensioni massime del corpo della richiesta del gheppio. Per ulteriori informazioni, vedere la sezione relativa alle [dimensioni massime del corpo della richiesta di Gheppio](#kestrel-maximum-request-body-size) . I limiti di connessione del client gheppio possono richiedere anche modifiche.

### <a name="null-reference-exception-with-iformfile"></a>Eccezione per riferimento Null con IFormFile

Se il controller accetta file caricati utilizzando <xref:Microsoft.AspNetCore.Http.IFormFile> , ma il valore è `null` , verificare che il form HTML specifichi un `enctype` valore di `multipart/form-data` . Se questo attributo non è impostato sull' `<form>` elemento, il caricamento del file non viene eseguito e gli <xref:Microsoft.AspNetCore.Http.IFormFile> argomenti associati sono `null` . Verificare inoltre che la [denominazione di caricamento nei dati del modulo corrisponda alla denominazione dell'app](#match-name-attribute-value-to-parameter-name-of-post-method).

### <a name="stream-was-too-long"></a>Il flusso è troppo lungo

Gli esempi in questo argomento si basano su <xref:System.IO.MemoryStream> per conservare il contenuto del file caricato. Il limite delle dimensioni di un oggetto `MemoryStream` è `int.MaxValue` . Se lo scenario di caricamento dei file dell'app richiede un contenuto del file di dimensioni superiori a 50 MB, usare un approccio alternativo che non si basa su un singolo `MemoryStream` per contenere il contenuto di un file caricato.

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

ASP.NET Core supporta il caricamento di uno o più file mediante l'associazione di modelli memorizzati nel buffer per i file più piccoli e il flusso non memorizzato nel buffer per file di dimensioni maggiori.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="security-considerations"></a>Considerazioni relative alla sicurezza

Prestare attenzione quando si fornisce agli utenti la possibilità di caricare file in un server. Gli utenti malintenzionati possono tentare di:

* Eseguire attacchi [di tipo Denial of Service](/windows-hardware/drivers/ifs/denial-of-service) .
* Caricare virus o malware.
* Compromettere le reti e i server in altri modi.

I passaggi di sicurezza che riducono la probabilità di un attacco riuscito sono:

* Caricare i file in un'area di caricamento file dedicata, preferibilmente in un'unità non di sistema. Un percorso dedicato semplifica l'imposizione delle restrizioni di sicurezza sui file caricati. Disabilitare le autorizzazioni di esecuzione per il percorso di caricamento del file.&dagger;
* Non **rende permanente i** file caricati nella stessa struttura di directory dell'app.&dagger;
* Usare un nome file sicuro determinato dall'app. Non usare un nome file fornito dall'utente o il nome file non attendibile del file caricato. &dagger; HTML codifica il nome file non attendibile quando viene visualizzato. Ad esempio, registrare il nome del file o visualizzarlo nell'interfaccia utente (l' Razor output viene codificato automaticamente in HTML).
* Consenti solo le estensioni di file approvate per la specifica di progettazione dell'app.&dagger; <!-- * Check the file format signature to prevent a user from uploading a masqueraded file.&dagger; For example, don't permit a user to upload an *.exe* file with a *.txt* extension. Add this back when we get instructions how to do this.  -->
* Verificare che i controlli lato client vengano eseguiti sul server. &dagger; I controlli lato client sono facili da aggirare.
* Controllare le dimensioni di un file caricato. Impostare un limite di dimensioni massime per impedire caricamenti di grandi dimensioni.&dagger;
* Quando i file non devono essere sovrascritti da un file caricato con lo stesso nome, controllare il nome del file sul database o sull'archiviazione fisica prima di caricare il file.
* **Eseguire uno scanner virus/malware sul contenuto caricato prima che il file venga archiviato.**

&dagger;L'app di esempio illustra un approccio che soddisfa i criteri.

> [!WARNING]
> Il caricamento di codice dannoso in un sistema è spesso il primo passaggio per l'esecuzione di codice in grado di:
>
> * Ottenere il controllo completo di un sistema.
> * Sovraccaricare un sistema con il risultato che si verifica un arresto anomalo del sistema.
> * Compromettere i dati di sistemi o utenti.
> * Applicare i graffiti a un'interfaccia utente pubblica.
>
> Per informazioni sulla riduzione della superficie di attacco quando si accettano file dagli utenti, vedere le risorse seguenti:
>
> * [Caricamento di file senza restrizioni](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
> * [Protezione di Azure: Verificare i controlli appropriati quando si accettano file dagli utenti](/azure/security/azure-security-threat-modeling-tool-input-validation#controls-users)

Per ulteriori informazioni sull'implementazione di misure di sicurezza, inclusi esempi dall'app di esempio, vedere la sezione [convalida](#validation) .

## <a name="storage-scenarios"></a>Scenari di archiviazione

Le opzioni di archiviazione comuni per i file includono:

* Database

  * Per i caricamenti di file di piccole dimensioni, un database è spesso più veloce rispetto alle opzioni di archiviazione fisica (file system o condivisione di rete).
  * Un database è spesso più pratico delle opzioni di archiviazione fisica perché il recupero di un record di database per i dati utente può fornire simultaneamente il contenuto del file, ad esempio un'immagine avatar.
  * Un database è potenzialmente meno costoso rispetto all'utilizzo di un servizio di archiviazione dati.

* Archiviazione fisica (file system o condivisione di rete)

  * Per i caricamenti di file di grandi dimensioni:
    * I limiti del database possono limitare le dimensioni del caricamento.
    * L'archiviazione fisica è spesso meno economica rispetto all'archiviazione in un database.
  * L'archiviazione fisica è potenzialmente meno costosa rispetto all'utilizzo di un servizio di archiviazione dati.
  * Il processo dell'app deve avere le autorizzazioni di lettura e scrittura per il percorso di archiviazione. **Non concedere mai l'autorizzazione Execute.**

* Servizio di archiviazione dei dati (ad esempio, [archiviazione BLOB di Azure](https://azure.microsoft.com/services/storage/blobs/))

  * I servizi in genere offrono una maggiore scalabilità e resilienza rispetto a soluzioni locali che in genere sono soggette a singoli punti di errore.
  * I servizi sono un costo potenzialmente inferiore in scenari di infrastruttura di archiviazione di grandi dimensioni.

  Per altre informazioni, vedere [Guida introduttiva: usare .NET per creare un BLOB nell'archivio oggetti](/azure/storage/blobs/storage-quickstart-blobs-dotnet).

## <a name="file-upload-scenarios"></a>Scenari di caricamento di file

Due approcci generali per il caricamento di file sono il buffering e il flusso.

**responseBuffering**

L'intero file viene letto in un oggetto <xref:Microsoft.AspNetCore.Http.IFormFile> , ovvero una rappresentazione C# del file usato per elaborare o salvare il file.

Le risorse (disco, memoria) utilizzate dai caricamenti di file dipendono dal numero e dalle dimensioni dei caricamenti di file simultanei. Se un'app tenta di memorizzare nel buffer un numero eccessivo di caricamenti, il sito si arresta in modo anomalo quando esaurisce la memoria o lo spazio su disco. Se le dimensioni o la frequenza dei caricamenti di file esauriscono le risorse dell'app, usare il flusso.

> [!NOTE]
> Ogni singolo file memorizzato nel buffer che supera 64 KB viene spostato dalla memoria in un file temporaneo su disco.

Il buffering di file di piccole dimensioni viene trattato nelle sezioni seguenti di questo argomento:

* [Archiviazione fisica](#upload-small-files-with-buffered-model-binding-to-physical-storage)
* [Database](#upload-small-files-with-buffered-model-binding-to-a-database)

**Streaming**

Il file viene ricevuto da una richiesta multipart ed elaborato o salvato direttamente dall'app. Il flusso non migliora significativamente le prestazioni. Il flusso riduce le richieste di memoria o di spazio su disco durante il caricamento dei file.

Il flusso di file di grandi dimensioni è trattato nella sezione [caricare file di grandi dimensioni con flusso](#upload-large-files-with-streaming) .

### <a name="upload-small-files-with-buffered-model-binding-to-physical-storage"></a>Caricare file di piccole dimensioni con associazione di modelli memorizzati nel buffer all'archiviazione fisica

Per caricare file di piccole dimensioni, usare un modulo multipart o creare una richiesta POST usando JavaScript.

L'esempio seguente illustra l'uso di un Razor form di pagine per caricare un singolo file (*pages/BufferedSingleFileUploadPhysical. cshtml* nell'app di esempio):

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
            <span asp-validation-for="FileUpload.FormFile"></span>
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload" />
</form>
```

L'esempio seguente è analogo all'esempio precedente, ad eccezione del fatto che:

* Per inviare i dati del modulo, viene usato JavaScript ([Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API)).
* Nessuna convalida.

```cshtml
<form action="BufferedSingleFileUploadPhysical/?handler=Upload" 
      enctype="multipart/form-data" onsubmit="AJAXSubmit(this);return false;" 
      method="post">
    <dl>
        <dt>
            <label for="FileUpload_FormFile">File</label>
        </dt>
        <dd>
            <input id="FileUpload_FormFile" type="file" 
                name="FileUpload.FormFile" />
        </dd>
    </dl>

    <input class="btn" type="submit" value="Upload" />

    <div style="margin-top:15px">
        <output name="result"></output>
    </div>
</form>

<script>
  async function AJAXSubmit (oFormElement) {
    var resultElement = oFormElement.elements.namedItem("result");
    const formData = new FormData(oFormElement);

    try {
    const response = await fetch(oFormElement.action, {
      method: 'POST',
      body: formData
    });

    if (response.ok) {
      window.location.href = '/';
    }

    resultElement.value = 'Result: ' + response.status + ' ' + 
      response.statusText;
    } catch (error) {
      console.error('Error:', error);
    }
  }
</script>
```

Per eseguire il POST del form in JavaScript per i client che [non supportano l'API fetch](https://caniuse.com/#feat=fetch), usare uno degli approcci seguenti:

* Usare un riempimento di recupero (ad esempio, [Window. fetch Refill (github/fetch)](https://github.com/github/fetch)).
* Usare `XMLHttpRequest`. Ad esempio:

  ```javascript
  <script>
    "use strict";

    function AJAXSubmit (oFormElement) {
      var oReq = new XMLHttpRequest();
      oReq.onload = function(e) { 
      oFormElement.elements.namedItem("result").value = 
        'Result: ' + this.status + ' ' + this.statusText;
      };
      oReq.open("post", oFormElement.action);
      oReq.send(new FormData(oFormElement));
    }
  </script>
  ```

Per supportare il caricamento di file, i moduli HTML devono specificare un tipo di codifica ( `enctype` ) di `multipart/form-data` .

Affinché un `files` elemento input supporti il caricamento di più file, fornire l' `multiple` attributo nell' `<input>` elemento:

```cshtml
<input asp-for="FileUpload.FormFiles" type="file" multiple>
```

È possibile accedere ai singoli file caricati nel server tramite l' [associazione di modelli](xref:mvc/models/model-binding) tramite <xref:Microsoft.AspNetCore.Http.IFormFile> . L'app di esempio illustra più caricamenti di file memorizzati nel buffer per scenari di archiviazione di database e fisici.

<a name="filename"></a>

> [!WARNING]
> Non **utilizzare la** `FileName` proprietà di <xref:Microsoft.AspNetCore.Http.IFormFile> diverso da per la visualizzazione e la registrazione. Durante la visualizzazione o la registrazione, HTML codifica il nome del file. Un utente malintenzionato può fornire un nome file dannoso, inclusi percorsi completi o percorsi relativi. Le applicazioni devono:
>
> * Rimuovere il percorso dal nome file specificato dall'utente.
> * Salvare il nome file con codifica HTML, percorso-rimosso per l'interfaccia utente o la registrazione.
> * Genera un nuovo nome file casuale per l'archiviazione.
>
> Il codice seguente rimuove il percorso dal nome del file:
>
> ```csharp
> string untrustedFileName = Path.GetFileName(pathName);
> ```
>
> Gli esempi forniti finora non prendono in considerazione le considerazioni sulla sicurezza. Le informazioni aggiuntive sono fornite nelle sezioni seguenti e nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/):
>
> * [Considerazioni sulla sicurezza](#security-considerations)
> * [Convalida](#validation)

Quando si caricano file usando l'associazione di modelli e <xref:Microsoft.AspNetCore.Http.IFormFile> , il metodo di azione può accettare:

* Singolo oggetto <xref:Microsoft.AspNetCore.Http.IFormFile> .
* Qualsiasi raccolta seguente che rappresenta diversi file:
  * <xref:Microsoft.AspNetCore.Http.IFormFileCollection>
  * <xref:System.Collections.IEnumerable>\<<xref:Microsoft.AspNetCore.Http.IFormFile>>
  * [Elenco](xref:System.Collections.Generic.List`1)\<<xref:Microsoft.AspNetCore.Http.IFormFile>>

> [!NOTE]
> L'associazione corrisponde ai file di form in base al nome. Ad esempio, il `name` valore HTML in `<input type="file" name="formFile">` deve corrispondere al parametro/proprietà di C# associato ( `FormFile` ). Per ulteriori informazioni, vedere la sezione [corrispondenza del nome dell'attributo del nome del parametro del metodo post](#match-name-attribute-value-to-parameter-name-of-post-method) .

L'esempio seguente:

* Esegue il ciclo di uno o più file caricati.
* USA [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) per restituire un percorso completo per un file, incluso il nome del file. 
* Salva i file nella file system locale usando un nome file generato dall'app.
* Restituisce il numero totale e le dimensioni dei file caricati.

```csharp
public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> files)
{
    long size = files.Sum(f => f.Length);

    foreach (var formFile in files)
    {
        if (formFile.Length > 0)
        {
            var filePath = Path.GetTempFileName();

            using (var stream = System.IO.File.Create(filePath))
            {
                await formFile.CopyToAsync(stream);
            }
        }
    }

    // Process uploaded files
    // Don't rely on or trust the FileName property without validation.

    return Ok(new { count = files.Count, size });
}
```

Utilizzare `Path.GetRandomFileName` per generare un nome file senza percorso. Nell'esempio seguente il percorso viene ottenuto dalla configurazione:

```csharp
foreach (var formFile in files)
{
    if (formFile.Length > 0)
    {
        var filePath = Path.Combine(_config["StoredFilesPath"], 
            Path.GetRandomFileName());

        using (var stream = System.IO.File.Create(filePath))
        {
            await formFile.CopyToAsync(stream);
        }
    }
}
```

Il percorso passato a <xref:System.IO.FileStream> *deve* includere il nome del file. Se il nome file non viene specificato, <xref:System.UnauthorizedAccessException> viene generata un'eccezione in fase di esecuzione.

I file caricati con la <xref:Microsoft.AspNetCore.Http.IFormFile> tecnica vengono memorizzati nel buffer in memoria o su disco sul server prima dell'elaborazione. All'interno del metodo di azione, il <xref:Microsoft.AspNetCore.Http.IFormFile> contenuto è accessibile come <xref:System.IO.Stream> . Oltre al file system locale, i file possono essere salvati in una condivisione di rete o in un servizio di archiviazione file, ad esempio [archiviazione BLOB di Azure](/azure/visual-studio/vs-storage-aspnet5-getting-started-blobs).

Per un altro esempio che esegue il ciclo su più file per il caricamento e usa nomi di file sicuri, vedere *pages/BufferedMultipleFileUploadPhysical. cshtml. cs* nell'app di esempio.

> [!WARNING]
> [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) genera un'eccezione <xref:System.IO.IOException> se vengono creati più di 65.535 file senza eliminare i file temporanei precedenti. Il limite di 65.535 file è un limite per server. Per ulteriori informazioni su questo limite per il sistema operativo Windows, vedere la sezione Osservazioni negli argomenti seguenti:
>
> * [GetTempFileNameA (funzione)](/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea#remarks)
> * <xref:System.IO.Path.GetTempFileName*>

### <a name="upload-small-files-with-buffered-model-binding-to-a-database"></a>Caricare file di piccole dimensioni con associazione di modelli memorizzati nel buffer in un database

Per archiviare i dati dei file binari in un database utilizzando [Entity Framework](/ef/core/index), definire una <xref:System.Byte> proprietà di matrice nell'entità:

```csharp
public class AppFile
{
    public int Id { get; set; }
    public byte[] Content { get; set; }
}
```

Specificare una proprietà del modello di pagina per la classe che include <xref:Microsoft.AspNetCore.Http.IFormFile> :

```csharp
public class BufferedSingleFileUploadDbModel : PageModel
{
    ...

    [BindProperty]
    public BufferedSingleFileUploadDb FileUpload { get; set; }

    ...
}

public class BufferedSingleFileUploadDb
{
    [Required]
    [Display(Name="File")]
    public IFormFile FormFile { get; set; }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Http.IFormFile> può essere usato direttamente come parametro del metodo di azione o come proprietà del modello associato. Nell'esempio precedente viene utilizzata una proprietà del modello associato.

Il `FileUpload` viene utilizzato nel Razor form pagine:

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload">
</form>
```

Quando il modulo viene inviato al server, copiarlo in <xref:Microsoft.AspNetCore.Http.IFormFile> un flusso e salvarlo come una matrice di byte nel database. Nell'esempio seguente, `_dbContext` archivia il contesto di database dell'app:

```csharp
public async Task<IActionResult> OnPostUploadAsync()
{
    using (var memoryStream = new MemoryStream())
    {
        await FileUpload.FormFile.CopyToAsync(memoryStream);

        // Upload the file if less than 2 MB
        if (memoryStream.Length < 2097152)
        {
            var file = new AppFile()
            {
                Content = memoryStream.ToArray()
            };

            _dbContext.File.Add(file);

            await _dbContext.SaveChangesAsync();
        }
        else
        {
            ModelState.AddModelError("File", "The file is too large.");
        }
    }

    return Page();
}
```

L'esempio precedente è simile a uno scenario illustrato nell'app di esempio:

* *Pages/BufferedSingleFileUploadDb. cshtml*
* *Pages/BufferedSingleFileUploadDb. cshtml. cs*

> [!WARNING]
> Quando si archiviano dati binari all'interno di database relazionali, è necessario prestare attenzione, perché l'operazione può influire negativamente sulle prestazioni.
>
> Non fare affidamento o considerare attendibile la `FileName` proprietà di <xref:Microsoft.AspNetCore.Http.IFormFile> senza convalida. La `FileName` proprietà deve essere usata solo a scopo di visualizzazione e solo dopo la codifica HTML.
>
> Gli esempi forniti non prendono in considerazione le considerazioni sulla sicurezza. Le informazioni aggiuntive sono fornite nelle sezioni seguenti e nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/):
>
> * [Considerazioni sulla sicurezza](#security-considerations)
> * [Convalida](#validation)

### <a name="upload-large-files-with-streaming"></a>Caricare file di grandi dimensioni con flusso

Nell'esempio seguente viene illustrato come utilizzare JavaScript per trasmettere un file a un'azione del controller. Il token antifalsificazione del file viene generato usando un attributo di filtro personalizzato e passato alle intestazioni HTTP del client anziché nel corpo della richiesta. Poiché il metodo di azione elabora direttamente i dati caricati, l'associazione del modello di modulo viene disabilitata da un altro filtro personalizzato. All'interno dell'azione, il contenuto del form viene letto tramite un `MultipartReader`, che legge ogni singola `MultipartSection`, elaborando il file o archiviandone il contenuto, come appropriato. Una volta lette le sezioni multiparte, l'azione esegue una propria associazione di modelli.

La risposta della pagina iniziale carica il form e salva un token antifalsificazione in un cookie (tramite l' `GenerateAntiforgeryTokenCookieAttribute` attributo). L'attributo usa il [supporto antifalsificazione](xref:security/anti-request-forgery) predefinito di ASP.NET Core per impostare un cookie con un token di richiesta:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/Antiforgery.cs?name=snippet_GenerateAntiforgeryTokenCookieAttribute)]

`DisableFormValueModelBindingAttribute`Viene usato per disabilitare l'associazione di modelli:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/ModelBinding.cs?name=snippet_DisableFormValueModelBindingAttribute)]

Nell'app di esempio `GenerateAntiforgeryTokenCookieAttribute` e `DisableFormValueModelBindingAttribute` vengono applicati come filtri ai modelli di applicazione della pagina di `/StreamedSingleFileUploadDb` e `/StreamedSingleFileUploadPhysical` all' `Startup.ConfigureServices` uso delle [ Razor convenzioni delle pagine](xref:razor-pages/razor-pages-conventions):

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Startup.cs?name=snippet_AddRazorPages&highlight=7-10,16-19)]

Poiché l'associazione di modelli non legge il modulo, i parametri associati al modulo non vengono associati (la query, la route e l'intestazione continuano a funzionare). Il metodo di azione funziona direttamente con la `Request` Proprietà. Per leggere ogni sezione, viene usato un `MultipartReader`. I dati chiave/valore vengono archiviati in un oggetto `KeyValueAccumulator` . Una volta lette le sezioni multipart, il contenuto di `KeyValueAccumulator` viene utilizzato per associare i dati del modulo a un tipo di modello.

Metodo completo `StreamingController.UploadDatabase` per lo streaming in un database con EF Core:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadDatabase)]

`MultipartRequestHelper` (*Utilities/MultipartRequestHelper. cs*):

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Utilities/MultipartRequestHelper.cs)]

Il `StreamingController.UploadPhysical` metodo completo per lo streaming in una posizione fisica:

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadPhysical)]

Nell'app di esempio, i controlli di convalida vengono gestiti da `FileHelpers.ProcessStreamedFile` .

## <a name="validation"></a>Convalida

La classe dell'app di esempio `FileHelpers` illustra alcuni controlli per i caricamenti di file memorizzati nel buffer <xref:Microsoft.AspNetCore.Http.IFormFile> e in flusso. Per l'elaborazione dei <xref:Microsoft.AspNetCore.Http.IFormFile> caricamenti di file memorizzati nel buffer nell'app di esempio, vedere il `ProcessFormFile` metodo nel file *Utilities/FileHelpers. cs* . Per l'elaborazione di file trasmessi, vedere il `ProcessStreamedFile` Metodo nello stesso file.

> [!WARNING]
> I metodi di elaborazione della convalida illustrati nell'app di esempio non analizzano il contenuto dei file caricati. Nella maggior parte degli scenari di produzione, nel file viene usata un'API scanner di virus/malware prima di rendere disponibile il file per gli utenti o altri sistemi.
>
> Sebbene l'esempio di argomento fornisca un esempio funzionante di tecniche di convalida, non implementare la `FileHelpers` classe in un'app di produzione a meno che non si:
>
> * Comprendere completamente l'implementazione.
> * Modificare l'implementazione in modo appropriato per l'ambiente e le specifiche dell'app.
>
> **Non implementare mai indiscriminatamente il codice di sicurezza in un'app senza soddisfare questi requisiti.**

### <a name="content-validation"></a>Convalida contenuto

**Usare un'API di analisi di virus/malware di terze parti nel contenuto caricato.**

L'analisi dei file richiede le risorse del server in scenari con volumi elevati. Se le prestazioni di elaborazione delle richieste sono diminuite a causa dell'analisi dei file, provare a eseguire l'offload del lavoro di analisi in un [servizio in background](xref:fundamentals/host/hosted-services), possibilmente in un servizio in esecuzione in un server diverso dal server dell'app. In genere, i file caricati vengono mantenuti in un'area in quarantena finché non vengono controllati dal programma antivirus in background. Quando un file passa, il file viene spostato nel percorso di archiviazione file normale. Questi passaggi vengono in genere eseguiti insieme a un record di database che indica lo stato di analisi di un file. Usando un approccio di questo tipo, l'app e il server app rimangono incentrati sulla risposta alle richieste.

### <a name="file-extension-validation"></a>Convalida dell'estensione di file

L'estensione del file caricato deve essere verificata rispetto a un elenco di estensioni consentite. Ad esempio:

```csharp
private string[] permittedExtensions = { ".txt", ".pdf" };

var ext = Path.GetExtension(uploadedFileName).ToLowerInvariant();

if (string.IsNullOrEmpty(ext) || !permittedExtensions.Contains(ext))
{
    // The extension is invalid ... discontinue processing the file
}
```

### <a name="file-signature-validation"></a>Convalida della firma del file

La firma di un file è determinata dai primi byte all'inizio di un file. Questi byte possono essere usati per indicare se l'estensione corrisponde al contenuto del file. L'app di esempio controlla le firme dei file per alcuni tipi di file comuni. Nell'esempio seguente, la firma del file per un'immagine JPEG viene verificata rispetto al file:

```csharp
private static readonly Dictionary<string, List<byte[]>> _fileSignature = 
    new Dictionary<string, List<byte[]>>
{
    { ".jpeg", new List<byte[]>
        {
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 },
        }
    },
};

using (var reader = new BinaryReader(uploadedFileData))
{
    var signatures = _fileSignature[ext];
    var headerBytes = reader.ReadBytes(signatures.Max(m => m.Length));
    
    return signatures.Any(signature => 
        headerBytes.Take(signature.Length).SequenceEqual(signature));
}
```

Per ottenere ulteriori firme di file, vedere il [database delle firme file](https://www.filesignatures.net/) e le specifiche del file ufficiale.

### <a name="file-name-security"></a>Sicurezza del nome file

Non usare mai un nome file fornito dal client per salvare un file nell'archiviazione fisica. Creare un nome file sicuro per il file usando [Path. GetRandomFileName](xref:System.IO.Path.GetRandomFileName*) o [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) per creare un percorso completo (incluso il nome file) per l'archiviazione temporanea.

Razor codifica HTML automaticamente i valori delle proprietà per la visualizzazione. Il codice seguente è sicuro da usare:

```cshtml
@foreach (var file in Model.DatabaseFiles) {
    <tr>
        <td>
            @file.UntrustedName
        </td>
    </tr>
}
```

Al di fuori di Razor , sempre il <xref:System.Net.WebUtility.HtmlEncode*> contenuto del nome file da una richiesta dell'utente.

Molte implementazioni devono includere un controllo per verificare che il file esista. in caso contrario, il file viene sovrascritto da un file con lo stesso nome. Fornire logica aggiuntiva per soddisfare le specifiche dell'app.

### <a name="size-validation"></a>Convalida dimensioni

Limitare le dimensioni dei file caricati.

Nell'app di esempio, le dimensioni del file sono limitate a 2 MB (indicate in byte). Il limite viene fornito tramite la [configurazione](xref:fundamentals/configuration/index) dal *appsettings.json* file:

```json
{
  "FileSizeLimit": 2097152
}
```

`FileSizeLimit`Viene inserito nelle `PageModel` classi:

```csharp
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    private readonly long _fileSizeLimit;

    public BufferedSingleFileUploadPhysicalModel(IConfiguration config)
    {
        _fileSizeLimit = config.GetValue<long>("FileSizeLimit");
    }

    ...
}
```

Quando le dimensioni di un file superano il limite, il file viene rifiutato:

```csharp
if (formFile.Length > _fileSizeLimit)
{
    // The file is too large ... discontinue processing the file
}
```

### <a name="match-name-attribute-value-to-parameter-name-of-post-method"></a>Corrisponde al valore dell'attributo Name al nome del parametro del metodo POST

In non Razor form che inviano dati del modulo o usano direttamente il nome di JavaScript `FormData` , il nome specificato nell'elemento del form o `FormData` deve corrispondere al nome del parametro nell'azione del controller.

Nell'esempio seguente:

* Quando si usa un `<input>` elemento, l' `name` attributo viene impostato sul valore `battlePlans` :

  ```html
  <input type="file" name="battlePlans" multiple>
  ```

* Quando `FormData` si usa in JavaScript, il nome viene impostato sul valore `battlePlans` :

  ```javascript
  var formData = new FormData();

  for (var file in files) {
    formData.append("battlePlans", file, file.name);
  }
  ```

Usare un nome corrispondente per il parametro del metodo C# ( `battlePlans` ):

* Per un Razor metodo gestore pagina pagine denominato `Upload` :

  ```csharp
  public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> battlePlans)
  ```

* Per un metodo di azione POST controller MVC:

  ```csharp
  public async Task<IActionResult> Post(List<IFormFile> battlePlans)
  ```

## <a name="server-and-app-configuration"></a>Configurazione del server e dell'app

### <a name="multipart-body-length-limit"></a>Limite di lunghezza del corpo multipart

<xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> imposta il limite per la lunghezza di ogni corpo multipart. Le sezioni del modulo che superano questo limite generano un'espressione <xref:System.IO.InvalidDataException> quando vengono analizzate. Il valore predefinito è 134.217.728 (128 MB). Personalizzare il limite usando l' <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> impostazione in `Startup.ConfigureServices` :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        // Set the limit to 256 MB
        options.MultipartBodyLengthLimit = 268435456;
    });
}
```

<xref:Microsoft.AspNetCore.Mvc.RequestFormLimitsAttribute> viene usato per impostare <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> per una singola pagina o un'azione.

In un' Razor app pagine, applicare il filtro con una [convenzione](xref:razor-pages/razor-pages-conventions) in `Startup.ConfigureServices` :

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model.Filters.Add(
                new RequestFormLimitsAttribute()
                {
                    // Set the limit to 256 MB
                    MultipartBodyLengthLimit = 268435456
                });
});
```

In un' Razor app pagine o in un'app MVC, applicare il filtro al modello di pagina o al metodo di azione:

```csharp
// Set the limit to 256 MB
[RequestFormLimits(MultipartBodyLengthLimit = 268435456)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="kestrel-maximum-request-body-size"></a>Dimensioni massime del corpo della richiesta di gheppio

Per le app ospitate da gheppio, le dimensioni massime predefinite del corpo della richiesta sono pari a 30 milioni byte, ovvero circa 28,6 MB. Personalizzare il limite usando l'opzione del server [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size) gheppio:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel((context, options) =>
            {
                // Handle requests up to 50 MB
                options.Limits.MaxRequestBodySize = 52428800;
            })
            .UseStartup<Startup>();
        });
```

<xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> viene usato per impostare [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size) per una singola pagina o un'azione.

In un' Razor app pagine, applicare il filtro con una [convenzione](xref:razor-pages/razor-pages-conventions) in `Startup.ConfigureServices` :

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model =>
            {
                // Handle requests up to 50 MB
                model.Filters.Add(
                    new RequestSizeLimitAttribute(52428800));
            });
});
```

In un' Razor app pagine o in un'app MVC, applicare il filtro alla classe del gestore di pagina o al metodo di azione:

```csharp
// Handle requests up to 50 MB
[RequestSizeLimit(52428800)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

Il `RequestSizeLimitAttribute` può essere applicato anche usando la [`@attribute`](xref:mvc/views/razor#attribute) Razor direttiva:

```cshtml
@attribute [RequestSizeLimitAttribute(52428800)]
```

### <a name="other-kestrel-limits"></a>Altri limiti del gheppio

Per le app ospitate da gheppio possono essere applicati altri limiti di Gheppio:

* [Numero massimo di connessioni client](xref:fundamentals/servers/kestrel#maximum-client-connections)
* [Frequenza dei dati di richiesta e risposta](xref:fundamentals/servers/kestrel#minimum-request-body-data-rate)

### <a name="iis"></a>IIS

Il limite di richieste predefinito ( `maxAllowedContentLength` ) è 30 milioni byte, che corrisponde a circa 28,6 MB. Personalizzare il limite nel `web.config` file. Nell'esempio seguente il limite è impostato su 50 MB (52.428.800 byte):

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

L' `maxAllowedContentLength` impostazione si applica solo a IIS. Per altre informazioni, vedere [limiti `<requestLimits>` della richiesta ](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).

Aumentare le dimensioni massime del corpo della richiesta per la richiesta HTTP impostando <xref:Microsoft.AspNetCore.Builder.IISServerOptions.MaxRequestBodySize%2A?displayProperty=nameWithType> in `Startup.ConfigureServices` . Nell'esempio seguente il limite è impostato su 50 MB (52.428.800 byte):

```csharp
services.Configure<IISServerOptions>(options =>
{
    options.MaxRequestBodySize = 52428800;
});
```

Per altre informazioni, vedere <xref:host-and-deploy/iis/index#iis-options>.

## <a name="troubleshoot"></a>Risolvere problemi

Di seguito sono trattati alcuni problemi comuni riscontrati durante il caricamento di file, con le soluzioni possibili corrispondenti.

### <a name="not-found-error-when-deployed-to-an-iis-server"></a>Errore non trovato quando viene distribuito in un server IIS

L'errore seguente indica che il file caricato supera la lunghezza del contenuto configurata del server:

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

Per ulteriori informazioni, vedere la sezione [IIS](#iis) .

### <a name="connection-failure"></a>Errore di connessione

Un errore di connessione e una connessione al server di reimpostazione probabilmente indicano che il file caricato supera le dimensioni massime del corpo della richiesta del gheppio. Per ulteriori informazioni, vedere la sezione relativa alle [dimensioni massime del corpo della richiesta di Gheppio](#kestrel-maximum-request-body-size) . I limiti di connessione del client gheppio possono richiedere anche modifiche.

### <a name="null-reference-exception-with-iformfile"></a>Eccezione per riferimento Null con IFormFile

Se il controller accetta file caricati utilizzando <xref:Microsoft.AspNetCore.Http.IFormFile> , ma il valore è `null` , verificare che il form HTML specifichi un `enctype` valore di `multipart/form-data` . Se questo attributo non è impostato sull' `<form>` elemento, il caricamento del file non viene eseguito e gli <xref:Microsoft.AspNetCore.Http.IFormFile> argomenti associati sono `null` . Verificare inoltre che la [denominazione di caricamento nei dati del modulo corrisponda alla denominazione dell'app](#match-name-attribute-value-to-parameter-name-of-post-method).

### <a name="stream-was-too-long"></a>Il flusso è troppo lungo

Gli esempi in questo argomento si basano su <xref:System.IO.MemoryStream> per conservare il contenuto del file caricato. Il limite delle dimensioni di un oggetto `MemoryStream` è `int.MaxValue` . Se lo scenario di caricamento dei file dell'app richiede un contenuto del file di dimensioni superiori a 50 MB, usare un approccio alternativo che non si basa su un singolo `MemoryStream` per contenere il contenuto di un file caricato.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core supporta il caricamento di uno o più file mediante l'associazione di modelli memorizzati nel buffer per i file più piccoli e il flusso non memorizzato nel buffer per file di dimensioni maggiori.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="security-considerations"></a>Considerazioni relative alla sicurezza

Prestare attenzione quando si fornisce agli utenti la possibilità di caricare file in un server. Gli utenti malintenzionati possono tentare di:

* Eseguire attacchi [di tipo Denial of Service](/windows-hardware/drivers/ifs/denial-of-service) .
* Caricare virus o malware.
* Compromettere le reti e i server in altri modi.

I passaggi di sicurezza che riducono la probabilità di un attacco riuscito sono:

* Caricare i file in un'area di caricamento file dedicata, preferibilmente in un'unità non di sistema. Un percorso dedicato semplifica l'imposizione delle restrizioni di sicurezza sui file caricati. Disabilitare le autorizzazioni di esecuzione per il percorso di caricamento del file.&dagger;
* Non **rende permanente i** file caricati nella stessa struttura di directory dell'app.&dagger;
* Usare un nome file sicuro determinato dall'app. Non usare un nome file fornito dall'utente o il nome file non attendibile del file caricato. &dagger; HTML codifica il nome file non attendibile quando viene visualizzato. Ad esempio, registrare il nome del file o visualizzarlo nell'interfaccia utente (l' Razor output viene codificato automaticamente in HTML).
* Consenti solo le estensioni di file approvate per la specifica di progettazione dell'app.&dagger; <!-- * Check the file format signature to prevent a user from uploading a masqueraded file.&dagger; For example, don't permit a user to upload an *.exe* file with a *.txt* extension. Add this back when we get instructions how to do this.  -->
* Verificare che i controlli lato client vengano eseguiti sul server. &dagger; I controlli lato client sono facili da aggirare.
* Controllare le dimensioni di un file caricato. Impostare un limite di dimensioni massime per impedire caricamenti di grandi dimensioni.&dagger;
* Quando i file non devono essere sovrascritti da un file caricato con lo stesso nome, controllare il nome del file sul database o sull'archiviazione fisica prima di caricare il file.
* **Eseguire uno scanner virus/malware sul contenuto caricato prima che il file venga archiviato.**

&dagger;L'app di esempio illustra un approccio che soddisfa i criteri.

> [!WARNING]
> Il caricamento di codice dannoso in un sistema è spesso il primo passaggio per l'esecuzione di codice in grado di:
>
> * Ottenere il controllo completo di un sistema.
> * Sovraccaricare un sistema con il risultato che si verifica un arresto anomalo del sistema.
> * Compromettere i dati di sistemi o utenti.
> * Applicare i graffiti a un'interfaccia utente pubblica.
>
> Per informazioni sulla riduzione della superficie di attacco quando si accettano file dagli utenti, vedere le risorse seguenti:
>
> * [Caricamento di file senza restrizioni](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
> * [Protezione di Azure: Verificare i controlli appropriati quando si accettano file dagli utenti](/azure/security/azure-security-threat-modeling-tool-input-validation#controls-users)

Per ulteriori informazioni sull'implementazione di misure di sicurezza, inclusi esempi dall'app di esempio, vedere la sezione [convalida](#validation) .

## <a name="storage-scenarios"></a>Scenari di archiviazione

Le opzioni di archiviazione comuni per i file includono:

* Database

  * Per i caricamenti di file di piccole dimensioni, un database è spesso più veloce rispetto alle opzioni di archiviazione fisica (file system o condivisione di rete).
  * Un database è spesso più pratico delle opzioni di archiviazione fisica perché il recupero di un record di database per i dati utente può fornire simultaneamente il contenuto del file, ad esempio un'immagine avatar.
  * Un database è potenzialmente meno costoso rispetto all'utilizzo di un servizio di archiviazione dati.

* Archiviazione fisica (file system o condivisione di rete)

  * Per i caricamenti di file di grandi dimensioni:
    * I limiti del database possono limitare le dimensioni del caricamento.
    * L'archiviazione fisica è spesso meno economica rispetto all'archiviazione in un database.
  * L'archiviazione fisica è potenzialmente meno costosa rispetto all'utilizzo di un servizio di archiviazione dati.
  * Il processo dell'app deve avere le autorizzazioni di lettura e scrittura per il percorso di archiviazione. **Non concedere mai l'autorizzazione Execute.**

* Servizio di archiviazione dei dati (ad esempio, [archiviazione BLOB di Azure](https://azure.microsoft.com/services/storage/blobs/))

  * I servizi in genere offrono una maggiore scalabilità e resilienza rispetto a soluzioni locali che in genere sono soggette a singoli punti di errore.
  * I servizi sono un costo potenzialmente inferiore in scenari di infrastruttura di archiviazione di grandi dimensioni.

  Per altre informazioni, vedere [Guida introduttiva: usare .NET per creare un BLOB nell'archivio oggetti](/azure/storage/blobs/storage-quickstart-blobs-dotnet). Nell'argomento viene illustrato <xref:Microsoft.Azure.Storage.File.CloudFile.UploadFromFileAsync*> , ma <xref:Microsoft.Azure.Storage.File.CloudFile.UploadFromStreamAsync*> è possibile utilizzare per salvare un <xref:System.IO.FileStream> nell'archivio BLOB quando si utilizza un <xref:System.IO.Stream> .

## <a name="file-upload-scenarios"></a>Scenari di caricamento di file

Due approcci generali per il caricamento di file sono il buffering e il flusso.

**responseBuffering**

L'intero file viene letto in un oggetto <xref:Microsoft.AspNetCore.Http.IFormFile> , ovvero una rappresentazione C# del file usato per elaborare o salvare il file.

Le risorse (disco, memoria) utilizzate dai caricamenti di file dipendono dal numero e dalle dimensioni dei caricamenti di file simultanei. Se un'app tenta di memorizzare nel buffer un numero eccessivo di caricamenti, il sito si arresta in modo anomalo quando esaurisce la memoria o lo spazio su disco. Se le dimensioni o la frequenza dei caricamenti di file esauriscono le risorse dell'app, usare il flusso.

> [!NOTE]
> Ogni singolo file memorizzato nel buffer che supera 64 KB viene spostato dalla memoria in un file temporaneo su disco.

Il buffering di file di piccole dimensioni viene trattato nelle sezioni seguenti di questo argomento:

* [Archiviazione fisica](#upload-small-files-with-buffered-model-binding-to-physical-storage)
* [Database](#upload-small-files-with-buffered-model-binding-to-a-database)

**Streaming**

Il file viene ricevuto da una richiesta multipart ed elaborato o salvato direttamente dall'app. Il flusso non migliora significativamente le prestazioni. Il flusso riduce le richieste di memoria o di spazio su disco durante il caricamento dei file.

Il flusso di file di grandi dimensioni è trattato nella sezione [caricare file di grandi dimensioni con flusso](#upload-large-files-with-streaming) .

### <a name="upload-small-files-with-buffered-model-binding-to-physical-storage"></a>Caricare file di piccole dimensioni con associazione di modelli memorizzati nel buffer all'archiviazione fisica

Per caricare file di piccole dimensioni, usare un modulo multipart o creare una richiesta POST usando JavaScript.

L'esempio seguente illustra l'uso di un Razor form di pagine per caricare un singolo file (*pages/BufferedSingleFileUploadPhysical. cshtml* nell'app di esempio):

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
            <span asp-validation-for="FileUpload.FormFile"></span>
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload" />
</form>
```

L'esempio seguente è analogo all'esempio precedente, ad eccezione del fatto che:

* Per inviare i dati del modulo, viene usato JavaScript ([Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API)).
* Nessuna convalida.

```cshtml
<form action="BufferedSingleFileUploadPhysical/?handler=Upload" 
      enctype="multipart/form-data" onsubmit="AJAXSubmit(this);return false;" 
      method="post">
    <dl>
        <dt>
            <label for="FileUpload_FormFile">File</label>
        </dt>
        <dd>
            <input id="FileUpload_FormFile" type="file" 
                name="FileUpload.FormFile" />
        </dd>
    </dl>

    <input class="btn" type="submit" value="Upload" />

    <div style="margin-top:15px">
        <output name="result"></output>
    </div>
</form>

<script>
  async function AJAXSubmit (oFormElement) {
    var resultElement = oFormElement.elements.namedItem("result");
    const formData = new FormData(oFormElement);

    try {
    const response = await fetch(oFormElement.action, {
      method: 'POST',
      body: formData
    });

    if (response.ok) {
      window.location.href = '/';
    }

    resultElement.value = 'Result: ' + response.status + ' ' + 
      response.statusText;
    } catch (error) {
      console.error('Error:', error);
    }
  }
</script>
```

Per eseguire il POST del form in JavaScript per i client che [non supportano l'API fetch](https://caniuse.com/#feat=fetch), usare uno degli approcci seguenti:

* Usare un riempimento di recupero (ad esempio, [Window. fetch Refill (github/fetch)](https://github.com/github/fetch)).
* Usare `XMLHttpRequest`. Ad esempio:

  ```javascript
  <script>
    "use strict";

    function AJAXSubmit (oFormElement) {
      var oReq = new XMLHttpRequest();
      oReq.onload = function(e) { 
      oFormElement.elements.namedItem("result").value = 
        'Result: ' + this.status + ' ' + this.statusText;
      };
      oReq.open("post", oFormElement.action);
      oReq.send(new FormData(oFormElement));
    }
  </script>
  ```

Per supportare il caricamento di file, i moduli HTML devono specificare un tipo di codifica ( `enctype` ) di `multipart/form-data` .

Affinché un `files` elemento input supporti il caricamento di più file, fornire l' `multiple` attributo nell' `<input>` elemento:

```cshtml
<input asp-for="FileUpload.FormFiles" type="file" multiple>
```

È possibile accedere ai singoli file caricati nel server tramite l' [associazione di modelli](xref:mvc/models/model-binding) tramite <xref:Microsoft.AspNetCore.Http.IFormFile> . L'app di esempio illustra più caricamenti di file memorizzati nel buffer per scenari di archiviazione di database e fisici.

<a name="filename2"></a>

> [!WARNING]
> Non **utilizzare la** `FileName` proprietà di <xref:Microsoft.AspNetCore.Http.IFormFile> diverso da per la visualizzazione e la registrazione. Durante la visualizzazione o la registrazione, HTML codifica il nome del file. Un utente malintenzionato può fornire un nome file dannoso, inclusi percorsi completi o percorsi relativi. Le applicazioni devono:
>
> * Rimuovere il percorso dal nome file specificato dall'utente.
> * Salvare il nome file con codifica HTML, percorso-rimosso per l'interfaccia utente o la registrazione.
> * Genera un nuovo nome file casuale per l'archiviazione.
>
> Il codice seguente rimuove il percorso dal nome del file:
>
> ```csharp
> string untrustedFileName = Path.GetFileName(pathName);
> ```
>
> Gli esempi forniti finora non prendono in considerazione le considerazioni sulla sicurezza. Le informazioni aggiuntive sono fornite nelle sezioni seguenti e nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/):
>
> * [Considerazioni sulla sicurezza](#security-considerations)
> * [Convalida](#validation)

Quando si caricano file usando l'associazione di modelli e <xref:Microsoft.AspNetCore.Http.IFormFile> , il metodo di azione può accettare:

* Singolo oggetto <xref:Microsoft.AspNetCore.Http.IFormFile> .
* Qualsiasi raccolta seguente che rappresenta diversi file:
  * <xref:Microsoft.AspNetCore.Http.IFormFileCollection>
  * <xref:System.Collections.IEnumerable>\<<xref:Microsoft.AspNetCore.Http.IFormFile>>
  * [Elenco](xref:System.Collections.Generic.List`1)\<<xref:Microsoft.AspNetCore.Http.IFormFile>>

> [!NOTE]
> L'associazione corrisponde ai file di form in base al nome. Ad esempio, il `name` valore HTML in `<input type="file" name="formFile">` deve corrispondere al parametro/proprietà di C# associato ( `FormFile` ). Per ulteriori informazioni, vedere la sezione [corrispondenza del nome dell'attributo del nome del parametro del metodo post](#match-name-attribute-value-to-parameter-name-of-post-method) .

L'esempio seguente:

* Esegue il ciclo di uno o più file caricati.
* USA [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) per restituire un percorso completo per un file, incluso il nome del file. 
* Salva i file nella file system locale usando un nome file generato dall'app.
* Restituisce il numero totale e le dimensioni dei file caricati.

```csharp
public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> files)
{
    long size = files.Sum(f => f.Length);

    foreach (var formFile in files)
    {
        if (formFile.Length > 0)
        {
            var filePath = Path.GetTempFileName();

            using (var stream = System.IO.File.Create(filePath))
            {
                await formFile.CopyToAsync(stream);
            }
        }
    }

    // Process uploaded files
    // Don't rely on or trust the FileName property without validation.

    return Ok(new { count = files.Count, size });
}
```

Utilizzare `Path.GetRandomFileName` per generare un nome file senza percorso. Nell'esempio seguente il percorso viene ottenuto dalla configurazione:

```csharp
foreach (var formFile in files)
{
    if (formFile.Length > 0)
    {
        var filePath = Path.Combine(_config["StoredFilesPath"], 
            Path.GetRandomFileName());

        using (var stream = System.IO.File.Create(filePath))
        {
            await formFile.CopyToAsync(stream);
        }
    }
}
```

Il percorso passato a <xref:System.IO.FileStream> *deve* includere il nome del file. Se il nome file non viene specificato, <xref:System.UnauthorizedAccessException> viene generata un'eccezione in fase di esecuzione.

I file caricati con la <xref:Microsoft.AspNetCore.Http.IFormFile> tecnica vengono memorizzati nel buffer in memoria o su disco sul server prima dell'elaborazione. All'interno del metodo di azione, il <xref:Microsoft.AspNetCore.Http.IFormFile> contenuto è accessibile come <xref:System.IO.Stream> . Oltre al file system locale, i file possono essere salvati in una condivisione di rete o in un servizio di archiviazione file, ad esempio [archiviazione BLOB di Azure](/azure/visual-studio/vs-storage-aspnet5-getting-started-blobs).

Per un altro esempio che esegue il ciclo su più file per il caricamento e usa nomi di file sicuri, vedere *pages/BufferedMultipleFileUploadPhysical. cshtml. cs* nell'app di esempio.

> [!WARNING]
> [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) genera un'eccezione <xref:System.IO.IOException> se vengono creati più di 65.535 file senza eliminare i file temporanei precedenti. Il limite di 65.535 file è un limite per server. Per ulteriori informazioni su questo limite per il sistema operativo Windows, vedere la sezione Osservazioni negli argomenti seguenti:
>
> * [GetTempFileNameA (funzione)](/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea#remarks)
> * <xref:System.IO.Path.GetTempFileName*>

### <a name="upload-small-files-with-buffered-model-binding-to-a-database"></a>Caricare file di piccole dimensioni con associazione di modelli memorizzati nel buffer in un database

Per archiviare i dati dei file binari in un database utilizzando [Entity Framework](/ef/core/index), definire una <xref:System.Byte> proprietà di matrice nell'entità:

```csharp
public class AppFile
{
    public int Id { get; set; }
    public byte[] Content { get; set; }
}
```

Specificare una proprietà del modello di pagina per la classe che include <xref:Microsoft.AspNetCore.Http.IFormFile> :

```csharp
public class BufferedSingleFileUploadDbModel : PageModel
{
    ...

    [BindProperty]
    public BufferedSingleFileUploadDb FileUpload { get; set; }

    ...
}

public class BufferedSingleFileUploadDb
{
    [Required]
    [Display(Name="File")]
    public IFormFile FormFile { get; set; }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Http.IFormFile> può essere usato direttamente come parametro del metodo di azione o come proprietà del modello associato. Nell'esempio precedente viene utilizzata una proprietà del modello associato.

Il `FileUpload` viene utilizzato nel Razor form pagine:

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload">
</form>
```

Quando il modulo viene inviato al server, copiarlo in <xref:Microsoft.AspNetCore.Http.IFormFile> un flusso e salvarlo come una matrice di byte nel database. Nell'esempio seguente, `_dbContext` archivia il contesto di database dell'app:

```csharp
public async Task<IActionResult> OnPostUploadAsync()
{
    using (var memoryStream = new MemoryStream())
    {
        await FileUpload.FormFile.CopyToAsync(memoryStream);

        // Upload the file if less than 2 MB
        if (memoryStream.Length < 2097152)
        {
            var file = new AppFile()
            {
                Content = memoryStream.ToArray()
            };

            _dbContext.File.Add(file);

            await _dbContext.SaveChangesAsync();
        }
        else
        {
            ModelState.AddModelError("File", "The file is too large.");
        }
    }

    return Page();
}
```

L'esempio precedente è simile a uno scenario illustrato nell'app di esempio:

* *Pages/BufferedSingleFileUploadDb. cshtml*
* *Pages/BufferedSingleFileUploadDb. cshtml. cs*

> [!WARNING]
> Quando si archiviano dati binari all'interno di database relazionali, è necessario prestare attenzione, perché l'operazione può influire negativamente sulle prestazioni.
>
> Non fare affidamento o considerare attendibile la `FileName` proprietà di <xref:Microsoft.AspNetCore.Http.IFormFile> senza convalida. La `FileName` proprietà deve essere usata solo a scopo di visualizzazione e solo dopo la codifica HTML.
>
> Gli esempi forniti non prendono in considerazione le considerazioni sulla sicurezza. Le informazioni aggiuntive sono fornite nelle sezioni seguenti e nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/file-uploads/samples/):
>
> * [Considerazioni sulla sicurezza](#security-considerations)
> * [Convalida](#validation)

### <a name="upload-large-files-with-streaming"></a>Caricare file di grandi dimensioni con flusso

Nell'esempio seguente viene illustrato come utilizzare JavaScript per trasmettere un file a un'azione del controller. Il token antifalsificazione del file viene generato usando un attributo di filtro personalizzato e passato alle intestazioni HTTP del client anziché nel corpo della richiesta. Poiché il metodo di azione elabora direttamente i dati caricati, l'associazione del modello di modulo viene disabilitata da un altro filtro personalizzato. All'interno dell'azione, il contenuto del form viene letto tramite un `MultipartReader`, che legge ogni singola `MultipartSection`, elaborando il file o archiviandone il contenuto, come appropriato. Una volta lette le sezioni multiparte, l'azione esegue una propria associazione di modelli.

La risposta della pagina iniziale carica il form e salva un token antifalsificazione in un cookie (tramite l' `GenerateAntiforgeryTokenCookieAttribute` attributo). L'attributo usa il [supporto antifalsificazione](xref:security/anti-request-forgery) predefinito di ASP.NET Core per impostare un cookie con un token di richiesta:

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Filters/Antiforgery.cs?name=snippet_GenerateAntiforgeryTokenCookieAttribute)]

`DisableFormValueModelBindingAttribute`Viene usato per disabilitare l'associazione di modelli:

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Filters/ModelBinding.cs?name=snippet_DisableFormValueModelBindingAttribute)]

Nell'app di esempio `GenerateAntiforgeryTokenCookieAttribute` e `DisableFormValueModelBindingAttribute` vengono applicati come filtri ai modelli di applicazione della pagina di `/StreamedSingleFileUploadDb` e `/StreamedSingleFileUploadPhysical` all' `Startup.ConfigureServices` uso delle [ Razor convenzioni delle pagine](xref:razor-pages/razor-pages-conventions):

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Startup.cs?name=snippet_AddMvc&highlight=8-11,17-20)]

Poiché l'associazione di modelli non legge il modulo, i parametri associati al modulo non vengono associati (la query, la route e l'intestazione continuano a funzionare). Il metodo di azione funziona direttamente con la `Request` Proprietà. Per leggere ogni sezione, viene usato un `MultipartReader`. I dati chiave/valore vengono archiviati in un oggetto `KeyValueAccumulator` . Una volta lette le sezioni multipart, il contenuto di `KeyValueAccumulator` viene utilizzato per associare i dati del modulo a un tipo di modello.

Metodo completo `StreamingController.UploadDatabase` per lo streaming in un database con EF Core:

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadDatabase)]

`MultipartRequestHelper` (*Utilities/MultipartRequestHelper. cs*):

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Utilities/MultipartRequestHelper.cs)]

Il `StreamingController.UploadPhysical` metodo completo per lo streaming in una posizione fisica:

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadPhysical)]

Nell'app di esempio, i controlli di convalida vengono gestiti da `FileHelpers.ProcessStreamedFile` .

## <a name="validation"></a>Convalida

La classe dell'app di esempio `FileHelpers` illustra alcuni controlli per i caricamenti di file memorizzati nel buffer <xref:Microsoft.AspNetCore.Http.IFormFile> e in flusso. Per l'elaborazione dei <xref:Microsoft.AspNetCore.Http.IFormFile> caricamenti di file memorizzati nel buffer nell'app di esempio, vedere il `ProcessFormFile` metodo nel file *Utilities/FileHelpers. cs* . Per l'elaborazione di file trasmessi, vedere il `ProcessStreamedFile` Metodo nello stesso file.

> [!WARNING]
> I metodi di elaborazione della convalida illustrati nell'app di esempio non analizzano il contenuto dei file caricati. Nella maggior parte degli scenari di produzione, nel file viene usata un'API scanner di virus/malware prima di rendere disponibile il file per gli utenti o altri sistemi.
>
> Sebbene l'esempio di argomento fornisca un esempio funzionante di tecniche di convalida, non implementare la `FileHelpers` classe in un'app di produzione a meno che non si:
>
> * Comprendere completamente l'implementazione.
> * Modificare l'implementazione in modo appropriato per l'ambiente e le specifiche dell'app.
>
> **Non implementare mai indiscriminatamente il codice di sicurezza in un'app senza soddisfare questi requisiti.**

### <a name="content-validation"></a>Convalida contenuto

**Usare un'API di analisi di virus/malware di terze parti nel contenuto caricato.**

L'analisi dei file richiede le risorse del server in scenari con volumi elevati. Se le prestazioni di elaborazione delle richieste sono diminuite a causa dell'analisi dei file, provare a eseguire l'offload del lavoro di analisi in un [servizio in background](xref:fundamentals/host/hosted-services), possibilmente in un servizio in esecuzione in un server diverso dal server dell'app. In genere, i file caricati vengono mantenuti in un'area in quarantena finché non vengono controllati dal programma antivirus in background. Quando un file passa, il file viene spostato nel percorso di archiviazione file normale. Questi passaggi vengono in genere eseguiti insieme a un record di database che indica lo stato di analisi di un file. Usando un approccio di questo tipo, l'app e il server app rimangono incentrati sulla risposta alle richieste.

### <a name="file-extension-validation"></a>Convalida dell'estensione di file

L'estensione del file caricato deve essere verificata rispetto a un elenco di estensioni consentite. Ad esempio:

```csharp
private string[] permittedExtensions = { ".txt", ".pdf" };

var ext = Path.GetExtension(uploadedFileName).ToLowerInvariant();

if (string.IsNullOrEmpty(ext) || !permittedExtensions.Contains(ext))
{
    // The extension is invalid ... discontinue processing the file
}
```

### <a name="file-signature-validation"></a>Convalida della firma del file

La firma di un file è determinata dai primi byte all'inizio di un file. Questi byte possono essere usati per indicare se l'estensione corrisponde al contenuto del file. L'app di esempio controlla le firme dei file per alcuni tipi di file comuni. Nell'esempio seguente, la firma del file per un'immagine JPEG viene verificata rispetto al file:

```csharp
private static readonly Dictionary<string, List<byte[]>> _fileSignature = 
    new Dictionary<string, List<byte[]>>
{
    { ".jpeg", new List<byte[]>
        {
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 },
        }
    },
};

using (var reader = new BinaryReader(uploadedFileData))
{
    var signatures = _fileSignature[ext];
    var headerBytes = reader.ReadBytes(signatures.Max(m => m.Length));
    
    return signatures.Any(signature => 
        headerBytes.Take(signature.Length).SequenceEqual(signature));
}
```

Per ottenere ulteriori firme di file, vedere il [database delle firme file](https://www.filesignatures.net/) e le specifiche del file ufficiale.

### <a name="file-name-security"></a>Sicurezza del nome file

Non usare mai un nome file fornito dal client per salvare un file nell'archiviazione fisica. Creare un nome file sicuro per il file usando [Path. GetRandomFileName](xref:System.IO.Path.GetRandomFileName*) o [Path. GetTempFileName](xref:System.IO.Path.GetTempFileName*) per creare un percorso completo (incluso il nome file) per l'archiviazione temporanea.

Razor codifica HTML automaticamente i valori delle proprietà per la visualizzazione. Il codice seguente è sicuro da usare:

```cshtml
@foreach (var file in Model.DatabaseFiles) {
    <tr>
        <td>
            @file.UntrustedName
        </td>
    </tr>
}
```

Al di fuori di Razor , sempre il <xref:System.Net.WebUtility.HtmlEncode*> contenuto del nome file da una richiesta dell'utente.

Molte implementazioni devono includere un controllo per verificare che il file esista. in caso contrario, il file viene sovrascritto da un file con lo stesso nome. Fornire logica aggiuntiva per soddisfare le specifiche dell'app.

### <a name="size-validation"></a>Convalida dimensioni

Limitare le dimensioni dei file caricati.

Nell'app di esempio, le dimensioni del file sono limitate a 2 MB (indicate in byte). Il limite viene fornito tramite la [configurazione](xref:fundamentals/configuration/index) dal *appsettings.json* file:

```json
{
  "FileSizeLimit": 2097152
}
```

`FileSizeLimit`Viene inserito nelle `PageModel` classi:

```csharp
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    private readonly long _fileSizeLimit;

    public BufferedSingleFileUploadPhysicalModel(IConfiguration config)
    {
        _fileSizeLimit = config.GetValue<long>("FileSizeLimit");
    }

    ...
}
```

Quando le dimensioni di un file superano il limite, il file viene rifiutato:

```csharp
if (formFile.Length > _fileSizeLimit)
{
    // The file is too large ... discontinue processing the file
}
```

### <a name="match-name-attribute-value-to-parameter-name-of-post-method"></a>Corrisponde al valore dell'attributo Name al nome del parametro del metodo POST

In non Razor form che inviano dati del modulo o usano direttamente il nome di JavaScript `FormData` , il nome specificato nell'elemento del form o `FormData` deve corrispondere al nome del parametro nell'azione del controller.

Nell'esempio seguente:

* Quando si usa un `<input>` elemento, l' `name` attributo viene impostato sul valore `battlePlans` :

  ```html
  <input type="file" name="battlePlans" multiple>
  ```

* Quando `FormData` si usa in JavaScript, il nome viene impostato sul valore `battlePlans` :

  ```javascript
  var formData = new FormData();

  for (var file in files) {
    formData.append("battlePlans", file, file.name);
  }
  ```

Usare un nome corrispondente per il parametro del metodo C# ( `battlePlans` ):

* Per un Razor metodo gestore pagina pagine denominato `Upload` :

  ```csharp
  public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> battlePlans)
  ```

* Per un metodo di azione POST controller MVC:

  ```csharp
  public async Task<IActionResult> Post(List<IFormFile> battlePlans)
  ```

## <a name="server-and-app-configuration"></a>Configurazione del server e dell'app

### <a name="multipart-body-length-limit"></a>Limite di lunghezza del corpo multipart

<xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> imposta il limite per la lunghezza di ogni corpo multipart. Le sezioni del modulo che superano questo limite generano un'espressione <xref:System.IO.InvalidDataException> quando vengono analizzate. Il valore predefinito è 134.217.728 (128 MB). Personalizzare il limite usando l' <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> impostazione in `Startup.ConfigureServices` :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        // Set the limit to 256 MB
        options.MultipartBodyLengthLimit = 268435456;
    });
}
```

<xref:Microsoft.AspNetCore.Mvc.RequestFormLimitsAttribute> viene usato per impostare <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> per una singola pagina o un'azione.

In un' Razor app pagine, applicare il filtro con una [convenzione](xref:razor-pages/razor-pages-conventions) in `Startup.ConfigureServices` :

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.Conventions
            .AddPageApplicationModelConvention("/FileUploadPage",
                model.Filters.Add(
                    new RequestFormLimitsAttribute()
                    {
                        // Set the limit to 256 MB
                        MultipartBodyLengthLimit = 268435456
                    });
    })
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

In un' Razor app pagine o in un'app MVC, applicare il filtro al modello di pagina o al metodo di azione:

```csharp
// Set the limit to 256 MB
[RequestFormLimits(MultipartBodyLengthLimit = 268435456)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="kestrel-maximum-request-body-size"></a>Dimensioni massime del corpo della richiesta di gheppio

Per le app ospitate da gheppio, le dimensioni massime predefinite del corpo della richiesta sono pari a 30 milioni byte, ovvero circa 28,6 MB. Personalizzare il limite usando l'opzione del server [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size) gheppio:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, options) =>
        {
            // Handle requests up to 50 MB
            options.Limits.MaxRequestBodySize = 52428800;
        });
```

<xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> viene usato per impostare [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size) per una singola pagina o un'azione.

In un' Razor app pagine, applicare il filtro con una [convenzione](xref:razor-pages/razor-pages-conventions) in `Startup.ConfigureServices` :

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.Conventions
            .AddPageApplicationModelConvention("/FileUploadPage",
                model =>
                {
                    // Handle requests up to 50 MB
                    model.Filters.Add(
                        new RequestSizeLimitAttribute(52428800));
                });
    })
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

In un' Razor app pagine o in un'app MVC, applicare il filtro alla classe del gestore di pagina o al metodo di azione:

```csharp
// Handle requests up to 50 MB
[RequestSizeLimit(52428800)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="other-kestrel-limits"></a>Altri limiti del gheppio

Per le app ospitate da gheppio possono essere applicati altri limiti di Gheppio:

* [Numero massimo di connessioni client](xref:fundamentals/servers/kestrel#maximum-client-connections)
* [Frequenza dei dati di richiesta e risposta](xref:fundamentals/servers/kestrel#minimum-request-body-data-rate)

### <a name="iis"></a>IIS

Il limite di richieste predefinito ( `maxAllowedContentLength` ) è 30 milioni byte, che corrisponde a circa 28,6 MB. Personalizzare il limite nel `web.config` file. Nell'esempio seguente il limite è impostato su 50 MB (52.428.800 byte):

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

L' `maxAllowedContentLength` impostazione si applica solo a IIS. Per altre informazioni, vedere [limiti `<requestLimits>` della richiesta ](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).

Aumentare le dimensioni massime del corpo della richiesta per la richiesta HTTP impostando <xref:Microsoft.AspNetCore.Builder.IISServerOptions.MaxRequestBodySize%2A?displayProperty=nameWithType> in `Startup.ConfigureServices` . Nell'esempio seguente il limite è impostato su 50 MB (52.428.800 byte):

```csharp
services.Configure<IISServerOptions>(options =>
{
    options.MaxRequestBodySize = 52428800;
});
```

Per altre informazioni, vedere <xref:host-and-deploy/iis/index#iis-options>.

## <a name="troubleshoot"></a>Risolvere problemi

Di seguito sono trattati alcuni problemi comuni riscontrati durante il caricamento di file, con le soluzioni possibili corrispondenti.

### <a name="not-found-error-when-deployed-to-an-iis-server"></a>Errore non trovato quando viene distribuito in un server IIS

L'errore seguente indica che il file caricato supera la lunghezza del contenuto configurata del server:

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

Per ulteriori informazioni, vedere la sezione [IIS](#iis) .

### <a name="connection-failure"></a>Errore di connessione

Un errore di connessione e una connessione al server di reimpostazione probabilmente indicano che il file caricato supera le dimensioni massime del corpo della richiesta del gheppio. Per ulteriori informazioni, vedere la sezione relativa alle [dimensioni massime del corpo della richiesta di Gheppio](#kestrel-maximum-request-body-size) . I limiti di connessione del client gheppio possono richiedere anche modifiche.

### <a name="null-reference-exception-with-iformfile"></a>Eccezione per riferimento Null con IFormFile

Se il controller accetta file caricati utilizzando <xref:Microsoft.AspNetCore.Http.IFormFile> , ma il valore è `null` , verificare che il form HTML specifichi un `enctype` valore di `multipart/form-data` . Se questo attributo non è impostato sull' `<form>` elemento, il caricamento del file non viene eseguito e gli <xref:Microsoft.AspNetCore.Http.IFormFile> argomenti associati sono `null` . Verificare inoltre che la [denominazione di caricamento nei dati del modulo corrisponda alla denominazione dell'app](#match-name-attribute-value-to-parameter-name-of-post-method).

### <a name="stream-was-too-long"></a>Il flusso è troppo lungo

Gli esempi in questo argomento si basano su <xref:System.IO.MemoryStream> per conservare il contenuto del file caricato. Il limite delle dimensioni di un oggetto `MemoryStream` è `int.MaxValue` . Se lo scenario di caricamento dei file dell'app richiede un contenuto del file di dimensioni superiori a 50 MB, usare un approccio alternativo che non si basa su un singolo `MemoryStream` per contenere il contenuto di un file caricato.

::: moniker-end


## <a name="additional-resources"></a>Risorse aggiuntive

::: moniker range="< aspnetcore-5.0"
* [Svuotamento della richiesta di connessione HTTP](xref:fundamentals/servers/kestrel#http11-request-draining)
::: moniker-end
::: moniker range=">= aspnetcore-5.0"
* [Svuotamento della richiesta di connessione HTTP](xref:fundamentals/servers/kestrel/request-draining)
::: moniker-end

* [Caricamento di file senza restrizioni](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
* [Sicurezza di Azure: frame di sicurezza: convalida dell'input | Attenuazioni](/azure/security/azure-security-threat-modeling-tool-input-validation)
* [Modelli di progettazione cloud di Azure: modello di chiave del posteggiatore](/azure/architecture/patterns/valet-key)
