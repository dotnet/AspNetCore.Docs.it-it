---
title: BlazorCaricamenti di file ASP.NET Core
author: guardrex
description: Informazioni su come caricare file in Blazor con il componente inputfile.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
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
ms.date: 02/18/2021
uid: blazor/file-uploads
ms.openlocfilehash: a31821f03efd39d774a4a3c61d027983a1783e2d
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395123"
---
# <a name="aspnet-core-blazor-file-uploads"></a>BlazorCaricamenti di file ASP.NET Core

Usare il `InputFile` componente per leggere i dati dei file del browser nel codice .NET, anche per i caricamenti di file.

> [!WARNING]
> Seguire sempre le procedure consigliate per la sicurezza di caricamento file. Per altre informazioni, vedere <xref:mvc/models/file-uploads#security-considerations>.

## <a name="inputfile-component"></a>Componente `InputFile`

Il componente viene sottoposta `InputFile` a rendering come input HTML di tipo `file` .

Per impostazione predefinita, l'utente seleziona singoli file. Aggiungere l' `multiple` attributo per consentire all'utente di caricare più file contemporaneamente. Quando uno o più file vengono selezionati dall'utente, il `InputFile` componente genera un `OnChange` evento e passa un oggetto `InputFileChangeEventArgs` che fornisce l'accesso all'elenco di file selezionati e informazioni dettagliate su ogni file.

Per leggere i dati da un file selezionato dall'utente:

* Chiamare `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` sul file e leggere dal flusso restituito. Per ulteriori informazioni, vedere la sezione [flussi di file](#file-streams) .
* L'oggetto <xref:System.IO.Stream> restituito da `OpenReadStream` impone una dimensione massima in byte dell'oggetto `Stream` letto. Per impostazione predefinita, le dimensioni dei file non superiori a 512.000 byte (500 KB) possono essere lette prima che altre letture provochino un'eccezione. Questo limite è presente per impedire agli sviluppatori di leggere accidentalmente file di grandi dimensioni in memoria. Il `maxAllowedSize` parametro in `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` può essere utilizzato per specificare una dimensione maggiore, se necessario.
* Evitare di leggere il flusso di file in ingresso direttamente in memoria. Ad esempio, non copiare i byte di file in un oggetto <xref:System.IO.MemoryStream> o leggere come matrice di byte. Questi approcci possono causare problemi di prestazioni e sicurezza, soprattutto in Blazor Server . In alternativa, è consigliabile copiare i byte dei file in un archivio esterno, ad esempio un BLOB o un file su disco.

Un componente che riceve un file di immagine può chiamare il `RequestImageFileAsync` metodo pratico sul file per ridimensionare i dati dell'immagine nel runtime JavaScript del browser prima che l'immagine venga trasmessa nell'app.

Nell'esempio seguente viene illustrato il caricamento di più file in un componente. `InputFileChangeEventArgs.GetMultipleFiles` consente la lettura di più file. Specificare il numero massimo di file che si prevede di leggere per impedire a un utente malintenzionato di caricare un numero maggiore di file rispetto a quanto previsto dall'app. `InputFileChangeEventArgs.File` consente di leggere il primo e l'unico file se il caricamento del file non supporta più file.

> [!NOTE]
> <xref:Microsoft.AspNetCore.Components.Forms.InputFileChangeEventArgs> si trova nello <xref:Microsoft.AspNetCore.Components.Forms?displayProperty=fullName> spazio dei nomi, che in genere è uno degli spazi dei nomi nel `_Imports.razor` file dell'app.

`Pages/UploadFiles.razor`:

```razor
@page "/upload-files"
@using System.IO

<h3>Upload Files</h3>

<p>
    <label>
        Max file size:
        <input type="number" @bind="maxFileSize" />
    </label>
</p>

<p>
    <label>
        Max allowed files:
        <input type="number" @bind="maxAllowedFiles" />
    </label>
</p>

<p>
    <label>
        Upload up to @maxAllowedFiles files of up to @maxFileSize bytes each:
        <InputFile OnChange="@LoadFiles" multiple />
    </label>
</p>

<p>@exceptionMessage</p>

@if (isLoading)
{
    <p>Loading...</p>
}

<ul>
    @foreach (var (file, content) in loadedFiles)
    {
        <li>
            <ul>
                <li>Name: @file.Name</li>
                <li>Last modified: @file.LastModified.ToString()</li>
                <li>Size (bytes): @file.Size</li>
                <li>Content type: @file.ContentType</li>
                <li>Content: @content</li>
            </ul>
        </li>
    }
</ul>

@code {
    private Dictionary<IBrowserFile, string> loadedFiles =
        new Dictionary<IBrowserFile, string>();
    private long maxFileSize = 1024 * 15;
    private int maxAllowedFiles = 3;
    private bool isLoading;
    string exceptionMessage;

    async Task LoadFiles(InputFileChangeEventArgs e)
    {
        isLoading = true;
        loadedFiles.Clear();
        exceptionMessage = string.Empty;

        try
        {
            foreach (var file in e.GetMultipleFiles(maxAllowedFiles))
            {
                using var reader = 
                    new StreamReader(file.OpenReadStream(maxFileSize));

                loadedFiles.Add(file, await reader.ReadToEndAsync());
            }
        }
        catch (Exception ex)
        {
            exceptionMessage = ex.Message;
        }

        isLoading = false;
    }
}
```

`IBrowserFile` restituisce [i metadati esposti dal browser](https://developer.mozilla.org/docs/Web/API/File#Instance_properties) come proprietà. Questi metadati possono essere utili per la convalida preliminare.

## <a name="upload-files-to-a-server"></a>Caricare file in un server

Nell'esempio seguente viene illustrato il caricamento di file in un server utilizzando una Blazor WebAssembly soluzione ospitata.

> [!WARNING]
> Prestare attenzione quando si fornisce agli utenti la possibilità di caricare file in un server. Per altre informazioni, vedere <xref:mvc/models/file-uploads#security-considerations>.

La `UploadResult` classe seguente nel **`Shared`** progetto mantiene il risultato di un file caricato. Quando non è possibile caricare un file nel server, viene restituito un codice di errore in `ErrorCode` per la visualizzazione all'utente. Un nome file archiviato sicuro viene generato nel server per ogni file e restituito al client in per la `StoredFileName` visualizzazione. I file vengono codificati tra il client e il server usando il nome file non sicuro/non attendibile in `FileName` .

`UploadResult.cs`:

```csharp
public class UploadResult
{
    public bool Uploaded { get; set; }

    public string FileName { get; set; }

    public string StoredFileName { get; set; }

    public int ErrorCode { get; set; }
}
```

Il `UploadFiles` componente seguente nel **`Client`** progetto:

* Consente agli utenti di caricare file dal client.
* Visualizza il nome file non attendibile/non sicuro fornito dal client nell'interfaccia utente perché Razor codifica automaticamente le stringhe. **Non considerare attendibili i nomi di file forniti dai client in altri scenari.**

`Pages/UploadFiles.razor`:

```razor
@page "/upload-files"
@using System.Linq
@using Microsoft.Extensions.Logging
@inject HttpClient Http
@inject ILogger<UploadFiles> logger

<h1>Upload Files</h1>

<p>
    <label>
        Upload up to @maxAllowedFiles files:
        <InputFile OnChange="@OnInputFileChange" multiple />
    </label>
</p>

@if (files.Count > 0)
{
    <div class="card">
        <div class="card-body">
            <ul>
                @foreach (var file in files)
                {
                    <li>
                        File: @file.Name
                        <br>
                        @if (FileUpload(uploadResults, file.Name, logger,
                            out var result))
                        {
                            <span>
                                Stored File Name: @result.StoredFileName
                            </span>
                        }
                        else
                        {
                            <span>
                                There was an error uploading the file
                                (Error: @result.ErrorCode).
                            </span>
                        }
                    </li>
                }
            </ul>
        </div>
    </div>
}

@code {
    private IList<File> files = new List<File>();
    private IList<UploadResult> uploadResults = new List<UploadResult>();
    private int maxAllowedFiles = 3;
    private bool shouldRender;

    protected override bool ShouldRender() => shouldRender;

    private async Task OnInputFileChange(InputFileChangeEventArgs e)
    {
        shouldRender = false;
        long maxFileSize = 1024 * 1024 * 15;
        var upload = false;

        using var content = new MultipartFormDataContent();

        foreach (var file in e.GetMultipleFiles(maxAllowedFiles))
        {
            if (uploadResults.SingleOrDefault(
                f => f.FileName == file.Name) is null)
            {
                var fileContent = new StreamContent(file.OpenReadStream());

                files.Add(
                    new File()
                    {
                        Name = file.Name,
                    });

                if (file.Size < maxFileSize)
                {
                    content.Add(
                        content: fileContent,
                        name: "\"files\"",
                        fileName: file.Name);

                    upload = true;
                }
                else
                {
                    logger.LogInformation("{FileName} not uploaded", file.Name);

                    uploadResults.Add(
                        new UploadResult()
                        {
                            FileName = file.Name,
                            ErrorCode = 6,
                            Uploaded = false,
                        });
                }
            }
        }

        if (upload)
        {
            var response = await Http.PostAsync("/Filesave", content);

            var newUploadResults = await response.Content
                .ReadFromJsonAsync<IList<UploadResult>>();

            uploadResults = uploadResults.Concat(newUploadResults).ToList();
        }

        shouldRender = true;
    }

    private static bool FileUpload(IList<UploadResult> uploadResults,
        string fileName, ILogger<UploadFiles> logger, out UploadResult result)
    {
        result = uploadResults.SingleOrDefault(f => f.FileName == fileName);

        if (result is null)
        {
            logger.LogInformation("{FileName} not uploaded", fileName);
            result = new UploadResult();
            result.ErrorCode = 5;
        }

        return result.Uploaded;
    }

    private class File
    {
        public string Name { get; set; }
    }
}
```

Per usare il codice seguente, creare una `Development/unsafe_uploads` cartella nella radice del **`Server`** progetto.

Il `FilesaveController` controller seguente nel **`Server`** progetto Salva i file caricati dal client.

> [!WARNING]
> Nell'esempio i file vengono salvati senza analizzarne il contenuto. Nella maggior parte degli scenari di produzione, un'API antivirus/anti-malware scanner viene utilizzata sui file prima di renderli disponibili per il download o per l'utilizzo da parte di altri sistemi. Per ulteriori informazioni sulle considerazioni relative alla sicurezza quando si caricano file in un server, vedere <xref:mvc/models/file-uploads#security-considerations> .

`Controllers/FilesaveController.cs`:

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

[ApiController]
[Route("[controller]")]
public class FilesaveController : ControllerBase
{
    private readonly IWebHostEnvironment env;
    private readonly ILogger<FilesaveController> logger;

    public FilesaveController(IWebHostEnvironment env, 
        ILogger<FilesaveController> logger)
    {
        this.env = env;
        this.logger = logger;
    }

    [HttpPost]
    public async Task<ActionResult<IList<UploadResult>>> PostFile(
        [FromForm] IEnumerable<IFormFile> files)
    {
        var maxAllowedFiles = 3;
        long maxFileSize = 1024 * 1024 * 15;
        var filesProcessed = 0;
        var resourcePath = new Uri($"{Request.Scheme}://{Request.Host}/");
        IList<UploadResult> uploadResults = new List<UploadResult>();

        foreach (var file in files)
        {
            var uploadResult = new UploadResult();
            string trustedFileNameForFileStorage;
            var untrustedFileName = file.FileName;
            uploadResult.FileName = untrustedFileName;
            var trustedFileNameForDisplay = 
                WebUtility.HtmlEncode(untrustedFileName);

            if (filesProcessed < maxAllowedFiles)
            {
                if (file.Length == 0)
                {
                    logger.LogInformation("{FileName} length is 0", 
                        trustedFileNameForDisplay);
                    uploadResult.ErrorCode = 1;
                }
                else if (file.Length > maxFileSize)
                {
                    logger.LogInformation("{FileName} of {Length} bytes is " +
                        "larger than the limit of {Limit} bytes", 
                        trustedFileNameForDisplay, file.Length, maxFileSize);
                    uploadResult.ErrorCode = 2;
                }
                else
                {
                    try
                    {
                        trustedFileNameForFileStorage = Path.GetRandomFileName();
                        var path = Path.Combine(env.ContentRootPath, 
                            env.EnvironmentName, "unsafe_uploads", 
                            trustedFileNameForFileStorage);
                        using MemoryStream ms = new();
                        await file.CopyToAsync(ms);
                        await System.IO.File.WriteAllBytesAsync(path, ms.ToArray());
                        logger.LogInformation("{FileName} saved at {Path}", 
                            trustedFileNameForDisplay, path);
                        uploadResult.Uploaded = true;
                        uploadResult.StoredFileName = trustedFileNameForFileStorage;
                    }
                    catch (IOException ex)
                    {
                        logger.LogError("{FileName} error on upload: {Message}", 
                            trustedFileNameForDisplay, ex.Message);
                        uploadResult.ErrorCode = 3;
                    }
                }

                filesProcessed++;
            }
            else
            {
                logger.LogInformation("{FileName} not uploaded because the " +
                    "request exceeded the allowed {Count} of files", 
                    trustedFileNameForDisplay, maxAllowedFiles);
                uploadResult.ErrorCode = 4;
            }

            uploadResults.Add(uploadResult);
        }

        return new CreatedResult(resourcePath, uploadResults);
    }
}
```

## <a name="file-streams"></a>Flussi di file

In i Blazor WebAssembly dati dei file vengono trasmessi direttamente nel codice .NET all'interno del browser.

In i Blazor Server dati dei file vengono trasmessi attraverso la SignalR connessione nel codice .NET sul server mentre il file viene letto dal flusso. <xref:Microsoft.AspNetCore.Components.Forms.RemoteBrowserFileStreamOptions> consente di configurare le caratteristiche di caricamento dei file per Blazor Server .

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/models/file-uploads#security-considerations>
* <xref:blazor/forms-validation>
