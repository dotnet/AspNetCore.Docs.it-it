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
# <a name="aspnet-core-blazor-file-uploads"></a><span data-ttu-id="3a567-103">BlazorCaricamenti di file ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="3a567-103">ASP.NET Core Blazor file uploads</span></span>

<span data-ttu-id="3a567-104">Usare il `InputFile` componente per leggere i dati dei file del browser nel codice .NET, anche per i caricamenti di file.</span><span class="sxs-lookup"><span data-stu-id="3a567-104">Use the `InputFile` component to read browser file data into .NET code, including for file uploads.</span></span>

> [!WARNING]
> <span data-ttu-id="3a567-105">Seguire sempre le procedure consigliate per la sicurezza di caricamento file.</span><span class="sxs-lookup"><span data-stu-id="3a567-105">Always follow file upload security best practices.</span></span> <span data-ttu-id="3a567-106">Per altre informazioni, vedere <xref:mvc/models/file-uploads#security-considerations>.</span><span class="sxs-lookup"><span data-stu-id="3a567-106">For more information, see <xref:mvc/models/file-uploads#security-considerations>.</span></span>

## <a name="inputfile-component"></a><span data-ttu-id="3a567-107">Componente `InputFile`</span><span class="sxs-lookup"><span data-stu-id="3a567-107">`InputFile` component</span></span>

<span data-ttu-id="3a567-108">Il componente viene sottoposta `InputFile` a rendering come input HTML di tipo `file` .</span><span class="sxs-lookup"><span data-stu-id="3a567-108">The `InputFile` component renders as an HTML input of type `file`.</span></span>

<span data-ttu-id="3a567-109">Per impostazione predefinita, l'utente seleziona singoli file.</span><span class="sxs-lookup"><span data-stu-id="3a567-109">By default, the user selects single files.</span></span> <span data-ttu-id="3a567-110">Aggiungere l' `multiple` attributo per consentire all'utente di caricare più file contemporaneamente.</span><span class="sxs-lookup"><span data-stu-id="3a567-110">Add the `multiple` attribute to permit the user to upload multiple files at once.</span></span> <span data-ttu-id="3a567-111">Quando uno o più file vengono selezionati dall'utente, il `InputFile` componente genera un `OnChange` evento e passa un oggetto `InputFileChangeEventArgs` che fornisce l'accesso all'elenco di file selezionati e informazioni dettagliate su ogni file.</span><span class="sxs-lookup"><span data-stu-id="3a567-111">When one or more files is selected by the user, the `InputFile` component fires an `OnChange` event and passes in an `InputFileChangeEventArgs` that provides access to the selected file list and details about each file.</span></span>

<span data-ttu-id="3a567-112">Per leggere i dati da un file selezionato dall'utente:</span><span class="sxs-lookup"><span data-stu-id="3a567-112">To read data from a user-selected file:</span></span>

* <span data-ttu-id="3a567-113">Chiamare `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` sul file e leggere dal flusso restituito.</span><span class="sxs-lookup"><span data-stu-id="3a567-113">Call `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` on the file and read from the returned stream.</span></span> <span data-ttu-id="3a567-114">Per ulteriori informazioni, vedere la sezione [flussi di file](#file-streams) .</span><span class="sxs-lookup"><span data-stu-id="3a567-114">For more information, see the [File streams](#file-streams) section.</span></span>
* <span data-ttu-id="3a567-115">L'oggetto <xref:System.IO.Stream> restituito da `OpenReadStream` impone una dimensione massima in byte dell'oggetto `Stream` letto.</span><span class="sxs-lookup"><span data-stu-id="3a567-115">The <xref:System.IO.Stream> returned by `OpenReadStream` enforces a maximum size in bytes of the `Stream` being read.</span></span> <span data-ttu-id="3a567-116">Per impostazione predefinita, le dimensioni dei file non superiori a 512.000 byte (500 KB) possono essere lette prima che altre letture provochino un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="3a567-116">By default, files no larger than 512,000 bytes (500 KB) in size are allowed to be read before any further reads would result in an exception.</span></span> <span data-ttu-id="3a567-117">Questo limite è presente per impedire agli sviluppatori di leggere accidentalmente file di grandi dimensioni in memoria.</span><span class="sxs-lookup"><span data-stu-id="3a567-117">This limit is present to prevent developers from accidentally reading large files in to memory.</span></span> <span data-ttu-id="3a567-118">Il `maxAllowedSize` parametro in `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` può essere utilizzato per specificare una dimensione maggiore, se necessario.</span><span class="sxs-lookup"><span data-stu-id="3a567-118">The `maxAllowedSize` parameter on `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` can be used to specify a larger size if required.</span></span>
* <span data-ttu-id="3a567-119">Evitare di leggere il flusso di file in ingresso direttamente in memoria.</span><span class="sxs-lookup"><span data-stu-id="3a567-119">Avoid reading the incoming file stream directly into memory.</span></span> <span data-ttu-id="3a567-120">Ad esempio, non copiare i byte di file in un oggetto <xref:System.IO.MemoryStream> o leggere come matrice di byte.</span><span class="sxs-lookup"><span data-stu-id="3a567-120">For example, don't copy file bytes into a <xref:System.IO.MemoryStream> or read as a byte array.</span></span> <span data-ttu-id="3a567-121">Questi approcci possono causare problemi di prestazioni e sicurezza, soprattutto in Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="3a567-121">These approaches can result in performance and security problems, especially in Blazor Server.</span></span> <span data-ttu-id="3a567-122">In alternativa, è consigliabile copiare i byte dei file in un archivio esterno, ad esempio un BLOB o un file su disco.</span><span class="sxs-lookup"><span data-stu-id="3a567-122">Instead, consider copying file bytes to an external store, such as a blob or a file on disk.</span></span>

<span data-ttu-id="3a567-123">Un componente che riceve un file di immagine può chiamare il `RequestImageFileAsync` metodo pratico sul file per ridimensionare i dati dell'immagine nel runtime JavaScript del browser prima che l'immagine venga trasmessa nell'app.</span><span class="sxs-lookup"><span data-stu-id="3a567-123">A component that receives an image file can call the `RequestImageFileAsync` convenience method on the file to resize the image data within the browser's JavaScript runtime before the image is streamed into the app.</span></span>

<span data-ttu-id="3a567-124">Nell'esempio seguente viene illustrato il caricamento di più file in un componente.</span><span class="sxs-lookup"><span data-stu-id="3a567-124">The following example demonstrates multiple file upload in a component.</span></span> <span data-ttu-id="3a567-125">`InputFileChangeEventArgs.GetMultipleFiles` consente la lettura di più file.</span><span class="sxs-lookup"><span data-stu-id="3a567-125">`InputFileChangeEventArgs.GetMultipleFiles` allows reading multiple files.</span></span> <span data-ttu-id="3a567-126">Specificare il numero massimo di file che si prevede di leggere per impedire a un utente malintenzionato di caricare un numero maggiore di file rispetto a quanto previsto dall'app.</span><span class="sxs-lookup"><span data-stu-id="3a567-126">Specify the maximum number of files you expect to read to prevent a malicious user from uploading a larger number of files than the app expects.</span></span> <span data-ttu-id="3a567-127">`InputFileChangeEventArgs.File` consente di leggere il primo e l'unico file se il caricamento del file non supporta più file.</span><span class="sxs-lookup"><span data-stu-id="3a567-127">`InputFileChangeEventArgs.File` allows reading the first and only file if the file upload doesn't support multiple files.</span></span>

> [!NOTE]
> <span data-ttu-id="3a567-128"><xref:Microsoft.AspNetCore.Components.Forms.InputFileChangeEventArgs> si trova nello <xref:Microsoft.AspNetCore.Components.Forms?displayProperty=fullName> spazio dei nomi, che in genere è uno degli spazi dei nomi nel `_Imports.razor` file dell'app.</span><span class="sxs-lookup"><span data-stu-id="3a567-128"><xref:Microsoft.AspNetCore.Components.Forms.InputFileChangeEventArgs> is in the <xref:Microsoft.AspNetCore.Components.Forms?displayProperty=fullName> namespace, which is typically one of the namespaces in the app's `_Imports.razor` file.</span></span>

<span data-ttu-id="3a567-129">`Pages/UploadFiles.razor`:</span><span class="sxs-lookup"><span data-stu-id="3a567-129">`Pages/UploadFiles.razor`:</span></span>

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

<span data-ttu-id="3a567-130">`IBrowserFile` restituisce [i metadati esposti dal browser](https://developer.mozilla.org/docs/Web/API/File#Instance_properties) come proprietà.</span><span class="sxs-lookup"><span data-stu-id="3a567-130">`IBrowserFile` returns metadata [exposed by the browser](https://developer.mozilla.org/docs/Web/API/File#Instance_properties) as properties.</span></span> <span data-ttu-id="3a567-131">Questi metadati possono essere utili per la convalida preliminare.</span><span class="sxs-lookup"><span data-stu-id="3a567-131">This metadata can be useful for preliminary validation.</span></span>

## <a name="upload-files-to-a-server"></a><span data-ttu-id="3a567-132">Caricare file in un server</span><span class="sxs-lookup"><span data-stu-id="3a567-132">Upload files to a server</span></span>

<span data-ttu-id="3a567-133">Nell'esempio seguente viene illustrato il caricamento di file in un server utilizzando una Blazor WebAssembly soluzione ospitata.</span><span class="sxs-lookup"><span data-stu-id="3a567-133">The following example demonstrates uploading files to a server using a hosted Blazor WebAssembly solution.</span></span>

> [!WARNING]
> <span data-ttu-id="3a567-134">Prestare attenzione quando si fornisce agli utenti la possibilità di caricare file in un server.</span><span class="sxs-lookup"><span data-stu-id="3a567-134">Use caution when providing users with the ability to upload files to a server.</span></span> <span data-ttu-id="3a567-135">Per altre informazioni, vedere <xref:mvc/models/file-uploads#security-considerations>.</span><span class="sxs-lookup"><span data-stu-id="3a567-135">For more information, see <xref:mvc/models/file-uploads#security-considerations>.</span></span>

<span data-ttu-id="3a567-136">La `UploadResult` classe seguente nel **`Shared`** progetto mantiene il risultato di un file caricato.</span><span class="sxs-lookup"><span data-stu-id="3a567-136">The following `UploadResult` class in the **`Shared`** project maintains the result of an uploaded file.</span></span> <span data-ttu-id="3a567-137">Quando non è possibile caricare un file nel server, viene restituito un codice di errore in `ErrorCode` per la visualizzazione all'utente.</span><span class="sxs-lookup"><span data-stu-id="3a567-137">When a file fails to upload on the server, an error code is returned in `ErrorCode` for display to the user.</span></span> <span data-ttu-id="3a567-138">Un nome file archiviato sicuro viene generato nel server per ogni file e restituito al client in per la `StoredFileName` visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="3a567-138">A safe stored file name is generated on the server for each file and returned to the client in `StoredFileName` for display.</span></span> <span data-ttu-id="3a567-139">I file vengono codificati tra il client e il server usando il nome file non sicuro/non attendibile in `FileName` .</span><span class="sxs-lookup"><span data-stu-id="3a567-139">Files are keyed between the client and server using the unsafe/untrusted file name in `FileName`.</span></span>

<span data-ttu-id="3a567-140">`UploadResult.cs`:</span><span class="sxs-lookup"><span data-stu-id="3a567-140">`UploadResult.cs`:</span></span>

```csharp
public class UploadResult
{
    public bool Uploaded { get; set; }

    public string FileName { get; set; }

    public string StoredFileName { get; set; }

    public int ErrorCode { get; set; }
}
```

<span data-ttu-id="3a567-141">Il `UploadFiles` componente seguente nel **`Client`** progetto:</span><span class="sxs-lookup"><span data-stu-id="3a567-141">The following `UploadFiles` component in the **`Client`** project:</span></span>

* <span data-ttu-id="3a567-142">Consente agli utenti di caricare file dal client.</span><span class="sxs-lookup"><span data-stu-id="3a567-142">Permits users to upload files from the client.</span></span>
* <span data-ttu-id="3a567-143">Visualizza il nome file non attendibile/non sicuro fornito dal client nell'interfaccia utente perché Razor codifica automaticamente le stringhe.</span><span class="sxs-lookup"><span data-stu-id="3a567-143">Displays the untrusted/unsafe file name provided by the client in the UI because Razor automatically HTML-encodes strings.</span></span> <span data-ttu-id="3a567-144">**Non considerare attendibili i nomi di file forniti dai client in altri scenari.**</span><span class="sxs-lookup"><span data-stu-id="3a567-144">**Don't trust file names supplied by clients in other scenarios.**</span></span>

<span data-ttu-id="3a567-145">`Pages/UploadFiles.razor`:</span><span class="sxs-lookup"><span data-stu-id="3a567-145">`Pages/UploadFiles.razor`:</span></span>

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

<span data-ttu-id="3a567-146">Per usare il codice seguente, creare una `Development/unsafe_uploads` cartella nella radice del **`Server`** progetto.</span><span class="sxs-lookup"><span data-stu-id="3a567-146">To use the following code, create a `Development/unsafe_uploads` folder at the root of the **`Server`** project.</span></span>

<span data-ttu-id="3a567-147">Il `FilesaveController` controller seguente nel **`Server`** progetto Salva i file caricati dal client.</span><span class="sxs-lookup"><span data-stu-id="3a567-147">The following `FilesaveController` controller in the **`Server`** project saves uploaded files from the client.</span></span>

> [!WARNING]
> <span data-ttu-id="3a567-148">Nell'esempio i file vengono salvati senza analizzarne il contenuto.</span><span class="sxs-lookup"><span data-stu-id="3a567-148">The example saves files without scanning their contents.</span></span> <span data-ttu-id="3a567-149">Nella maggior parte degli scenari di produzione, un'API antivirus/anti-malware scanner viene utilizzata sui file prima di renderli disponibili per il download o per l'utilizzo da parte di altri sistemi.</span><span class="sxs-lookup"><span data-stu-id="3a567-149">In most production scenarios, an anti-virus/anti-malware scanner API is used on files before making them available for download or for use by other systems.</span></span> <span data-ttu-id="3a567-150">Per ulteriori informazioni sulle considerazioni relative alla sicurezza quando si caricano file in un server, vedere <xref:mvc/models/file-uploads#security-considerations> .</span><span class="sxs-lookup"><span data-stu-id="3a567-150">For more information on security considerations when uploading files to a server, see <xref:mvc/models/file-uploads#security-considerations>.</span></span>

<span data-ttu-id="3a567-151">`Controllers/FilesaveController.cs`:</span><span class="sxs-lookup"><span data-stu-id="3a567-151">`Controllers/FilesaveController.cs`:</span></span>

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

## <a name="file-streams"></a><span data-ttu-id="3a567-152">Flussi di file</span><span class="sxs-lookup"><span data-stu-id="3a567-152">File streams</span></span>

<span data-ttu-id="3a567-153">In i Blazor WebAssembly dati dei file vengono trasmessi direttamente nel codice .NET all'interno del browser.</span><span class="sxs-lookup"><span data-stu-id="3a567-153">In Blazor WebAssembly, file data is streamed directly into the .NET code within the browser.</span></span>

<span data-ttu-id="3a567-154">In i Blazor Server dati dei file vengono trasmessi attraverso la SignalR connessione nel codice .NET sul server mentre il file viene letto dal flusso.</span><span class="sxs-lookup"><span data-stu-id="3a567-154">In Blazor Server, file data is streamed over the SignalR connection into .NET code on the server as the file is read from the stream.</span></span> <span data-ttu-id="3a567-155"><xref:Microsoft.AspNetCore.Components.Forms.RemoteBrowserFileStreamOptions> consente di configurare le caratteristiche di caricamento dei file per Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="3a567-155"><xref:Microsoft.AspNetCore.Components.Forms.RemoteBrowserFileStreamOptions> allows configuring file upload characteristics for Blazor Server.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="3a567-156">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="3a567-156">Additional resources</span></span>

* <xref:mvc/models/file-uploads#security-considerations>
* <xref:blazor/forms-validation>
