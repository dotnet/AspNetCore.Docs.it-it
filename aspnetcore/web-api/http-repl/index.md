---
title: Testare le API Web con HttpRepl
author: scottaddie
description: Informazioni su come usare lo strumento globale di HttpRepl .NET Core per esplorare e testare un'API Web di ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc, devx-track-azurecli
ms.date: 11/12/2020
no-loc:
- ':::no-loc(appsettings.json):::'
- ':::no-loc(ASP.NET Core Identity):::'
- ':::no-loc(cookie):::'
- ':::no-loc(Cookie):::'
- ':::no-loc(Blazor):::'
- ':::no-loc(Blazor Server):::'
- ':::no-loc(Blazor WebAssembly):::'
- ':::no-loc(Identity):::'
- ":::no-loc(Let's Encrypt):::"
- ':::no-loc(Razor):::'
- ':::no-loc(SignalR):::'
uid: web-api/http-repl
ms.openlocfilehash: c661141c1bec93c0f7b6404964dc0bca2319b529
ms.sourcegitcommit: 827e8be18cebbcc09b467c089e17fa6f5e430cb2
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 11/15/2020
ms.locfileid: "94634613"
---
# <a name="test-web-apis-with-the-httprepl"></a><span data-ttu-id="c8de1-103">Testare le API Web con HttpRepl</span><span class="sxs-lookup"><span data-stu-id="c8de1-103">Test web APIs with the HttpRepl</span></span>

<span data-ttu-id="c8de1-104">Di [Scott Addie](https://twitter.com/Scott_Addie)</span><span class="sxs-lookup"><span data-stu-id="c8de1-104">By [Scott Addie](https://twitter.com/Scott_Addie)</span></span>

<span data-ttu-id="c8de1-105">Il ciclo Read-Eval-Print (REPL) HTTP:</span><span class="sxs-lookup"><span data-stu-id="c8de1-105">The HTTP Read-Eval-Print Loop (REPL) is:</span></span>

* <span data-ttu-id="c8de1-106">È un strumento da riga di comando multipiattaforma leggero supportato ovunque sia supportato .NET Core.</span><span class="sxs-lookup"><span data-stu-id="c8de1-106">A lightweight, cross-platform command-line tool that's supported everywhere .NET Core is supported.</span></span>
* <span data-ttu-id="c8de1-107">Consente di eseguire richieste HTTP per testare le API Web ASP.NET Core (e le API Web non ASP.NET Core) e visualizzare i relativi risultati.</span><span class="sxs-lookup"><span data-stu-id="c8de1-107">Used for making HTTP requests to test ASP.NET Core web APIs (and non-ASP.NET Core web APIs) and view their results.</span></span>
* <span data-ttu-id="c8de1-108">Riesce a testare le API Web ospitate in qualsiasi ambiente, inclusi localhost e Servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="c8de1-108">Capable of testing web APIs hosted in any environment, including localhost and Azure App Service.</span></span>

<span data-ttu-id="c8de1-109">Sono supportati i [verbi HTTP](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods) seguenti:</span><span class="sxs-lookup"><span data-stu-id="c8de1-109">The following [HTTP verbs](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods) are supported:</span></span>

* [<span data-ttu-id="c8de1-110">DELETE</span><span class="sxs-lookup"><span data-stu-id="c8de1-110">DELETE</span></span>](#test-http-delete-requests)
* [<span data-ttu-id="c8de1-111">GET</span><span class="sxs-lookup"><span data-stu-id="c8de1-111">GET</span></span>](#test-http-get-requests)
* [<span data-ttu-id="c8de1-112">HEAD</span><span class="sxs-lookup"><span data-stu-id="c8de1-112">HEAD</span></span>](#test-http-head-requests)
* [<span data-ttu-id="c8de1-113">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-113">OPTIONS</span></span>](#test-http-options-requests)
* [<span data-ttu-id="c8de1-114">PATCH</span><span class="sxs-lookup"><span data-stu-id="c8de1-114">PATCH</span></span>](#test-http-patch-requests)
* [<span data-ttu-id="c8de1-115">POST</span><span class="sxs-lookup"><span data-stu-id="c8de1-115">POST</span></span>](#test-http-post-requests)
* [<span data-ttu-id="c8de1-116">PUT</span><span class="sxs-lookup"><span data-stu-id="c8de1-116">PUT</span></span>](#test-http-put-requests)

<span data-ttu-id="c8de1-117">Per continuare, [visualizzare o scaricare l'API Web ASP.NET Core di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/http-repl/samples) ([come scaricare](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="c8de1-117">To follow along, [view or download the sample ASP.NET Core web API](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/http-repl/samples) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="c8de1-118">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="c8de1-118">Prerequisites</span></span>

* [!INCLUDE [2.1-SDK](~/includes/2.1-SDK.md)]

## <a name="installation"></a><span data-ttu-id="c8de1-119">Installazione</span><span class="sxs-lookup"><span data-stu-id="c8de1-119">Installation</span></span>

<span data-ttu-id="c8de1-120">Per installare HttpRepl, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-120">To install the HttpRepl, run the following command:</span></span>

```dotnetcli
dotnet tool install -g Microsoft.dotnet-httprepl
```

<span data-ttu-id="c8de1-121">Viene installato uno [strumento globale .NET Core](/dotnet/core/tools/global-tools#install-a-global-tool) dal pacchetto NuGet [Microsoft.dotnet-httprepl](https://www.nuget.org/packages/Microsoft.dotnet-httprepl).</span><span class="sxs-lookup"><span data-stu-id="c8de1-121">A [.NET Core Global Tool](/dotnet/core/tools/global-tools#install-a-global-tool) is installed from the [Microsoft.dotnet-httprepl](https://www.nuget.org/packages/Microsoft.dotnet-httprepl) NuGet package.</span></span>

## <a name="usage"></a><span data-ttu-id="c8de1-122">Utilizzo</span><span class="sxs-lookup"><span data-stu-id="c8de1-122">Usage</span></span>

<span data-ttu-id="c8de1-123">Dopo aver completato l'installazione dello strumento, eseguire il comando seguente per avviare il HttpRepl:</span><span class="sxs-lookup"><span data-stu-id="c8de1-123">After successful installation of the tool, run the following command to start the HttpRepl:</span></span>

```console
httprepl
```

<span data-ttu-id="c8de1-124">Per visualizzare i comandi HttpRepl disponibili, eseguire uno dei comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="c8de1-124">To view the available HttpRepl commands, run one of the following commands:</span></span>

```console
httprepl -h
```

```console
httprepl --help
```

<span data-ttu-id="c8de1-125">Verrà visualizzato l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-125">The following output is displayed:</span></span>

```console
Usage:
  httprepl [<BASE_ADDRESS>] [options]

Arguments:
  <BASE_ADDRESS> - The initial base address for the REPL.

Options:
  -h|--help - Show help information.

Once the REPL starts, these commands are valid:

Setup Commands:
Use these commands to configure the tool for your API server

connect        Configures the directory structure and base address of the api server
set header     Sets or clears a header for all requests. e.g. `set header content-type application/json`

HTTP Commands:
Use these commands to execute requests against your application.

GET            get - Issues a GET request
POST           post - Issues a POST request
PUT            put - Issues a PUT request
DELETE         delete - Issues a DELETE request
PATCH          patch - Issues a PATCH request
HEAD           head - Issues a HEAD request
OPTIONS        options - Issues a OPTIONS request

Navigation Commands:
The REPL allows you to navigate your URL space and focus on specific APIs that you are working on.

ls             Show all endpoints for the current path
cd             Append the given directory to the currently selected path, or move up a path when using `cd ..`

Shell Commands:
Use these commands to interact with the REPL shell.

clear          Removes all text from the shell
echo [on/off]  Turns request echoing on or off, show the request that was made when using request commands
exit           Exit the shell

REPL Customization Commands:
Use these commands to customize the REPL behavior.

pref [get/set] Allows viewing or changing preferences, e.g. 'pref set editor.command.default 'C:\\Program Files\\Microsoft VS Code\\Code.exe'`
run            Runs the script at the given path. A script is a set of commands that can be typed with one command per line
ui             Displays the Swagger UI page, if available, in the default browser

Use `help <COMMAND>` for more detail on an individual command. e.g. `help get`.
For detailed tool info, see https://aka.ms/http-repl-doc.
```

<span data-ttu-id="c8de1-126">Il HttpRepl offre il completamento del comando.</span><span class="sxs-lookup"><span data-stu-id="c8de1-126">The HttpRepl offers command completion.</span></span> <span data-ttu-id="c8de1-127">Premendo il tasto <kbd>TAB</kbd> è possibile scorrere l'elenco dei comandi che completano i caratteri o l'endpoint API digitato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-127">Pressing the <kbd>Tab</kbd> key iterates through the list of commands that complete the characters or API endpoint that you typed.</span></span> <span data-ttu-id="c8de1-128">Le sezioni seguenti descrivono i comandi dell'interfaccia della riga di comando disponibili.</span><span class="sxs-lookup"><span data-stu-id="c8de1-128">The following sections outline the available CLI commands.</span></span>

## <a name="connect-to-the-web-api"></a><span data-ttu-id="c8de1-129">Connettersi all'API Web</span><span class="sxs-lookup"><span data-stu-id="c8de1-129">Connect to the web API</span></span>

<span data-ttu-id="c8de1-130">Connettersi all'API Web eseguendo il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-130">Connect to a web API by running the following command:</span></span>

```console
httprepl <ROOT URI>
```

<span data-ttu-id="c8de1-131">`<ROOT URI>` è l'URI di base dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="c8de1-131">`<ROOT URI>` is the base URI for the web API.</span></span> <span data-ttu-id="c8de1-132">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-132">For example:</span></span>

```console
httprepl https://localhost:5001
```

<span data-ttu-id="c8de1-133">In alternativa, eseguire il comando seguente in qualsiasi momento mentre il HttpRepl è in esecuzione:</span><span class="sxs-lookup"><span data-stu-id="c8de1-133">Alternatively, run the following command at any time while the HttpRepl is running:</span></span>

```console
connect <ROOT URI>
```

<span data-ttu-id="c8de1-134">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-134">For example:</span></span>

```console
(Disconnected)> connect https://localhost:5001
```

### <a name="manually-point-to-the-openapi-description-for-the-web-api"></a><span data-ttu-id="c8de1-135">Puntare manualmente alla descrizione di OpenAPI per l'API Web</span><span class="sxs-lookup"><span data-stu-id="c8de1-135">Manually point to the OpenAPI description for the web API</span></span>

<span data-ttu-id="c8de1-136">Il comando Connect precedente tenterà di individuare automaticamente la descrizione di OpenAPI.</span><span class="sxs-lookup"><span data-stu-id="c8de1-136">The connect command above will attempt to find the OpenAPI description automatically.</span></span> <span data-ttu-id="c8de1-137">Se per qualche motivo non è in grado di eseguire questa operazione, è possibile specificare l'URI della descrizione OpenAPI per l'API Web usando l' `--openapi` opzione:</span><span class="sxs-lookup"><span data-stu-id="c8de1-137">If for some reason it's unable to do so, you can specify the URI of the OpenAPI description for the web API by using the `--openapi` option:</span></span>

```console
connect <ROOT URI> --openapi <OPENAPI DESCRIPTION ADDRESS>
```

<span data-ttu-id="c8de1-138">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-138">For example:</span></span>

```console
(Disconnected)> connect https://localhost:5001 --openapi /swagger/v1/swagger.json
```

### <a name="enable-verbose-output-for-details-on-openapi-description-searching-parsing-and-validation"></a><span data-ttu-id="c8de1-139">Abilitare l'output dettagliato per informazioni dettagliate sulla ricerca, l'analisi e la convalida di OpenAPI Description</span><span class="sxs-lookup"><span data-stu-id="c8de1-139">Enable verbose output for details on OpenAPI description searching, parsing, and validation</span></span>

<span data-ttu-id="c8de1-140">Se si specifica l' `--verbose` opzione con il `connect` comando, verranno prodotti maggiori dettagli quando lo strumento cercherà la descrizione di openapi, lo analizzerà e lo convaliderà.</span><span class="sxs-lookup"><span data-stu-id="c8de1-140">Specifying the `--verbose` option with the `connect` command will produce more details when the tool searches for the OpenAPI description, parses, and validates it.</span></span>

```console
connect <ROOT URI> --verbose
```

<span data-ttu-id="c8de1-141">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-141">For example:</span></span>

```console
(Disconnected)> connect https://localhost:5001 --verbose
Checking https://localhost:5001/swagger.json... 404 NotFound
Checking https://localhost:5001/swagger/v1/swagger.json... 404 NotFound
Checking https://localhost:5001/openapi.json... Found
Parsing... Successful (with warnings)
The field 'info' in 'document' object is REQUIRED [#/info]
The field 'paths' in 'document' object is REQUIRED [#/paths]
```

## <a name="navigate-the-web-api"></a><span data-ttu-id="c8de1-142">Esplorare l'API Web</span><span class="sxs-lookup"><span data-stu-id="c8de1-142">Navigate the web API</span></span>

### <a name="view-available-endpoints"></a><span data-ttu-id="c8de1-143">Visualizzare gli endpoint disponibili</span><span class="sxs-lookup"><span data-stu-id="c8de1-143">View available endpoints</span></span>

<span data-ttu-id="c8de1-144">Per visualizzare l'elenco dei diversi endpoint (controller) nel percorso corrente dell'indirizzo dell'API Web, eseguire il comando `ls` o `dir`:</span><span class="sxs-lookup"><span data-stu-id="c8de1-144">To list the different endpoints (controllers) at the current path of the web API address, run the `ls` or `dir` command:</span></span>

```console
https://localhost:5001/> ls
```

<span data-ttu-id="c8de1-145">Viene visualizzato il formato di output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-145">The following output format is displayed:</span></span>

```console
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/>
```

<span data-ttu-id="c8de1-146">L'output precedente indica che sono disponibili due controller: `Fruits` e `People`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-146">The preceding output indicates that there are two controllers available: `Fruits` and `People`.</span></span> <span data-ttu-id="c8de1-147">Entrambi i controller supportano operazioni HTTP GET e POST senza parametri.</span><span class="sxs-lookup"><span data-stu-id="c8de1-147">Both controllers support parameterless HTTP GET and POST operations.</span></span>

<span data-ttu-id="c8de1-148">L'esplorazione di un controller specifico offre altri dettagli.</span><span class="sxs-lookup"><span data-stu-id="c8de1-148">Navigating into a specific controller reveals more detail.</span></span> <span data-ttu-id="c8de1-149">Ad esempio, l'output del comando seguente mostra che il controller `Fruits` supporta anche le operazioni HTTP GET, PUT e DELETE.</span><span class="sxs-lookup"><span data-stu-id="c8de1-149">For example, the following command's output shows the `Fruits` controller also supports HTTP GET, PUT, and DELETE operations.</span></span> <span data-ttu-id="c8de1-150">Ognuna di queste operazioni prevede un parametro `id` nella route:</span><span class="sxs-lookup"><span data-stu-id="c8de1-150">Each of these operations expects an `id` parameter in the route:</span></span>

```console
https://localhost:5001/fruits> ls
.      [get|post]
..     []
{id}   [get|put|delete]

https://localhost:5001/fruits>
```

<span data-ttu-id="c8de1-151">In alternativa, eseguire il comando `ui` per aprire la pagina dell'interfaccia utente di Swagger dell'API Web in un browser.</span><span class="sxs-lookup"><span data-stu-id="c8de1-151">Alternatively, run the `ui` command to open the web API's Swagger UI page in a browser.</span></span> <span data-ttu-id="c8de1-152">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-152">For example:</span></span>

```console
https://localhost:5001/> ui
```

### <a name="navigate-to-an-endpoint"></a><span data-ttu-id="c8de1-153">Passare a un endpoint</span><span class="sxs-lookup"><span data-stu-id="c8de1-153">Navigate to an endpoint</span></span>

<span data-ttu-id="c8de1-154">Per passare a un endpoint diverso nell'API Web, eseguire il comando `cd`:</span><span class="sxs-lookup"><span data-stu-id="c8de1-154">To navigate to a different endpoint on the web API, run the `cd` command:</span></span>

```console
https://localhost:5001/> cd people
```

<span data-ttu-id="c8de1-155">Nel percorso che segue il comando `cd` non viene fatta distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="c8de1-155">The path following the `cd` command is case insensitive.</span></span> <span data-ttu-id="c8de1-156">Viene visualizzato il formato di output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-156">The following output format is displayed:</span></span>

```console
/people    [get|post]

https://localhost:5001/people>
```

## <a name="customize-the-httprepl"></a><span data-ttu-id="c8de1-157">Personalizzare HttpRepl</span><span class="sxs-lookup"><span data-stu-id="c8de1-157">Customize the HttpRepl</span></span>

<span data-ttu-id="c8de1-158">È possibile personalizzare i [colori](#set-color-preferences) predefiniti di HttpRepl.</span><span class="sxs-lookup"><span data-stu-id="c8de1-158">The HttpRepl's default [colors](#set-color-preferences) can be customized.</span></span> <span data-ttu-id="c8de1-159">È anche possibile definire un [editor di testo predefinito](#set-the-default-text-editor).</span><span class="sxs-lookup"><span data-stu-id="c8de1-159">Additionally, a [default text editor](#set-the-default-text-editor) can be defined.</span></span> <span data-ttu-id="c8de1-160">Le preferenze HttpRepl sono rese permanente nella sessione corrente e vengono rispettate in sessioni future.</span><span class="sxs-lookup"><span data-stu-id="c8de1-160">The HttpRepl preferences are persisted across the current session and are honored in future sessions.</span></span> <span data-ttu-id="c8de1-161">Dopo essere state modificate, le preferenze vengono archiviate nel file seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-161">Once modified, the preferences are stored in the following file:</span></span>

# <a name="linux"></a>[<span data-ttu-id="c8de1-162">Linux</span><span class="sxs-lookup"><span data-stu-id="c8de1-162">Linux</span></span>](#tab/linux)

<span data-ttu-id="c8de1-163">*%HOME%/.httpreplprefs*</span><span class="sxs-lookup"><span data-stu-id="c8de1-163">*%HOME%/.httpreplprefs*</span></span>

# <a name="macos"></a>[<span data-ttu-id="c8de1-164">macOS</span><span class="sxs-lookup"><span data-stu-id="c8de1-164">macOS</span></span>](#tab/macos)

<span data-ttu-id="c8de1-165">*%HOME%/.httpreplprefs*</span><span class="sxs-lookup"><span data-stu-id="c8de1-165">*%HOME%/.httpreplprefs*</span></span>

# <a name="windows"></a>[<span data-ttu-id="c8de1-166">Windows</span><span class="sxs-lookup"><span data-stu-id="c8de1-166">Windows</span></span>](#tab/windows)

<span data-ttu-id="c8de1-167">*%USERPROFILE%\\.httpreplprefs*</span><span class="sxs-lookup"><span data-stu-id="c8de1-167">*%USERPROFILE%\\.httpreplprefs*</span></span>

---

<span data-ttu-id="c8de1-168">Il file con estensione *httpreplprefs* viene caricato all'avvio e non viene monitorato per le modifiche in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="c8de1-168">The *.httpreplprefs* file is loaded on startup and not monitored for changes at runtime.</span></span> <span data-ttu-id="c8de1-169">Le modifiche manuali apportate al file diventano effettive solo dopo il riavvio dello strumento.</span><span class="sxs-lookup"><span data-stu-id="c8de1-169">Manual modifications to the file take effect only after restarting the tool.</span></span>

### <a name="view-the-settings"></a><span data-ttu-id="c8de1-170">Visualizzare le impostazioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-170">View the settings</span></span>

<span data-ttu-id="c8de1-171">Per visualizzare le impostazioni disponibili, eseguire il comando `pref get`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-171">To view the available settings, run the `pref get` command.</span></span> <span data-ttu-id="c8de1-172">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-172">For example:</span></span>

```console
https://localhost:5001/> pref get
```

<span data-ttu-id="c8de1-173">Il comando precedente visualizza le coppie chiave-valore disponibili:</span><span class="sxs-lookup"><span data-stu-id="c8de1-173">The preceding command displays the available key-value pairs:</span></span>

```console
colors.json=Green
colors.json.arrayBrace=BoldCyan
colors.json.comma=BoldYellow
colors.json.name=BoldMagenta
colors.json.nameSeparator=BoldWhite
colors.json.objectBrace=Cyan
colors.protocol=BoldGreen
colors.status=BoldYellow
```

### <a name="set-color-preferences"></a><span data-ttu-id="c8de1-174">Impostare le preferenze colori</span><span class="sxs-lookup"><span data-stu-id="c8de1-174">Set color preferences</span></span>

<span data-ttu-id="c8de1-175">La colorazione delle risposte è attualmente supportata solo per JSON.</span><span class="sxs-lookup"><span data-stu-id="c8de1-175">Response colorization is currently supported for JSON only.</span></span> <span data-ttu-id="c8de1-176">Per personalizzare la colorazione predefinita dello strumento HttpRepl, individuare la chiave corrispondente al colore da modificare.</span><span class="sxs-lookup"><span data-stu-id="c8de1-176">To customize the default HttpRepl tool coloring, locate the key corresponding to the color to be changed.</span></span> <span data-ttu-id="c8de1-177">Per istruzioni su come trovare le chiavi, vedere la sezione [Visualizzare le impostazioni](#view-the-settings).</span><span class="sxs-lookup"><span data-stu-id="c8de1-177">For instructions on how to find the keys, see the [View the settings](#view-the-settings) section.</span></span> <span data-ttu-id="c8de1-178">Ad esempio, modificare il valore della chiave `colors.json` da `Green` a `White` come indicato di seguito:</span><span class="sxs-lookup"><span data-stu-id="c8de1-178">For example, change the `colors.json` key value from `Green` to `White` as follows:</span></span>

```console
https://localhost:5001/people> pref set colors.json White
```

<span data-ttu-id="c8de1-179">È possibile usare solo i [colori consentiti](https://github.com/dotnet/HttpRepl/blob/01d5c3c3373e98fe566ff5ef8a17c571de880293/src/Microsoft.Repl/ConsoleHandling/AllowedColors.cs).</span><span class="sxs-lookup"><span data-stu-id="c8de1-179">Only the [allowed colors](https://github.com/dotnet/HttpRepl/blob/01d5c3c3373e98fe566ff5ef8a17c571de880293/src/Microsoft.Repl/ConsoleHandling/AllowedColors.cs) may be used.</span></span> <span data-ttu-id="c8de1-180">Output delle richieste HTTP successive con la nuova colorazione.</span><span class="sxs-lookup"><span data-stu-id="c8de1-180">Subsequent HTTP requests display output with the new coloring.</span></span>

<span data-ttu-id="c8de1-181">Quando non sono impostate chiavi di colore specifiche, vengono considerate chiavi più generiche.</span><span class="sxs-lookup"><span data-stu-id="c8de1-181">When specific color keys aren't set, more generic keys are considered.</span></span> <span data-ttu-id="c8de1-182">Per comprendere questo comportamento di fallback, si consideri l'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-182">To demonstrate this fallback behavior, consider the following example:</span></span>

* <span data-ttu-id="c8de1-183">Se `colors.json.name` non ha un valore, viene usato `colors.json.string`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-183">If `colors.json.name` doesn't have a value, `colors.json.string` is used.</span></span>
* <span data-ttu-id="c8de1-184">Se `colors.json.string` non ha un valore, viene usato `colors.json.literal`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-184">If `colors.json.string` doesn't have a value, `colors.json.literal` is used.</span></span>
* <span data-ttu-id="c8de1-185">Se `colors.json.literal` non ha un valore, viene usato `colors.json`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-185">If `colors.json.literal` doesn't have a value, `colors.json` is used.</span></span> 
* <span data-ttu-id="c8de1-186">Se `colors.json` non ha un valore, viene usato il colore del testo predefinito della shell dei comandi (`AllowedColors.None`).</span><span class="sxs-lookup"><span data-stu-id="c8de1-186">If `colors.json` doesn't have a value, the command shell's default text color (`AllowedColors.None`) is used.</span></span>

### <a name="set-indentation-size"></a><span data-ttu-id="c8de1-187">Impostare la dimensione del rientro</span><span class="sxs-lookup"><span data-stu-id="c8de1-187">Set indentation size</span></span>

<span data-ttu-id="c8de1-188">La personalizzazione della dimensione del rientro delle risposte è attualmente supportata solo per JSON.</span><span class="sxs-lookup"><span data-stu-id="c8de1-188">Response indentation size customization is currently supported for JSON only.</span></span> <span data-ttu-id="c8de1-189">La dimensione del rientro predefinita è di due spazi.</span><span class="sxs-lookup"><span data-stu-id="c8de1-189">The default size is two spaces.</span></span> <span data-ttu-id="c8de1-190">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-190">For example:</span></span>

```json
[
  {
    "id": 1,
    "name": "Apple"
  },
  {
    "id": 2,
    "name": "Orange"
  },
  {
    "id": 3,
    "name": "Strawberry"
  }
]
```

<span data-ttu-id="c8de1-191">Per modificare la dimensione predefinita, impostare la chiave `formatting.json.indentSize`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-191">To change the default size, set the `formatting.json.indentSize` key.</span></span> <span data-ttu-id="c8de1-192">Ad esempio, per usare sempre quattro spazi:</span><span class="sxs-lookup"><span data-stu-id="c8de1-192">For example, to always use four spaces:</span></span>

```console
pref set formatting.json.indentSize 4
```

<span data-ttu-id="c8de1-193">L'impostazione dei quattro spazi viene applicata alle risposte successive:</span><span class="sxs-lookup"><span data-stu-id="c8de1-193">Subsequent responses honor the setting of four spaces:</span></span>

```json
[
    {
        "id": 1,
        "name": "Apple"
    },
    {
        "id": 2,
        "name": "Orange"
    },
    {
        "id": 3,
        "name": "Strawberry"
    }
]
```

### <a name="set-the-default-text-editor"></a><span data-ttu-id="c8de1-194">Impostare l'editor di testo predefinito</span><span class="sxs-lookup"><span data-stu-id="c8de1-194">Set the default text editor</span></span>

<span data-ttu-id="c8de1-195">Per impostazione predefinita, HttpRepl non dispone di un editor di testo configurato per l'utilizzo.</span><span class="sxs-lookup"><span data-stu-id="c8de1-195">By default, the HttpRepl has no text editor configured for use.</span></span> <span data-ttu-id="c8de1-196">Per testare i metodi dell'API Web che richiedono un corpo della richiesta HTTP, è necessario impostare un editor di testo predefinito.</span><span class="sxs-lookup"><span data-stu-id="c8de1-196">To test web API methods requiring an HTTP request body, a default text editor must be set.</span></span> <span data-ttu-id="c8de1-197">Lo strumento HttpRepl avvia l'editor di testo configurato al solo scopo di comporre il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="c8de1-197">The HttpRepl tool launches the configured text editor for the sole purpose of composing the request body.</span></span> <span data-ttu-id="c8de1-198">Eseguire il comando seguente per impostare l'editor di testo preferito come predefinito:</span><span class="sxs-lookup"><span data-stu-id="c8de1-198">Run the following command to set your preferred text editor as the default:</span></span>

```console
pref set editor.command.default "<EXECUTABLE>"
```

<span data-ttu-id="c8de1-199">Nel comando precedente `<EXECUTABLE>` è il percorso completo del file eseguibile dell'editor di testo.</span><span class="sxs-lookup"><span data-stu-id="c8de1-199">In the preceding command, `<EXECUTABLE>` is the full path to the text editor's executable file.</span></span> <span data-ttu-id="c8de1-200">Ad esempio, eseguire il comando seguente per impostare Visual Studio Code come editor di testo predefinito:</span><span class="sxs-lookup"><span data-stu-id="c8de1-200">For example, run the following command to set Visual Studio Code as the default text editor:</span></span>

# <a name="linux"></a>[<span data-ttu-id="c8de1-201">Linux</span><span class="sxs-lookup"><span data-stu-id="c8de1-201">Linux</span></span>](#tab/linux)

```console
pref set editor.command.default "/usr/bin/code"
```

# <a name="macos"></a>[<span data-ttu-id="c8de1-202">macOS</span><span class="sxs-lookup"><span data-stu-id="c8de1-202">macOS</span></span>](#tab/macos)

```console
pref set editor.command.default "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code"
```

# <a name="windows"></a>[<span data-ttu-id="c8de1-203">Windows</span><span class="sxs-lookup"><span data-stu-id="c8de1-203">Windows</span></span>](#tab/windows)

```console
pref set editor.command.default "C:\Program Files\Microsoft VS Code\Code.exe"
```

---

<span data-ttu-id="c8de1-204">Per avviare l'editor di testo predefinito con argomenti dell'interfaccia della riga di comando specifici, impostare la chiave `editor.command.default.arguments`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-204">To launch the default text editor with specific CLI arguments, set the `editor.command.default.arguments` key.</span></span> <span data-ttu-id="c8de1-205">Si supponga, ad esempio, che Visual Studio Code sia l'editor di testo predefinito e che si desideri sempre che HttpRepl apra Visual Studio Code in una nuova sessione con estensioni disabilitate.</span><span class="sxs-lookup"><span data-stu-id="c8de1-205">For example, assume Visual Studio Code is the default text editor and that you always want the HttpRepl to open Visual Studio Code in a new session with extensions disabled.</span></span> <span data-ttu-id="c8de1-206">Eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-206">Run the following command:</span></span>

```console
pref set editor.command.default.arguments "--disable-extensions --new-window"
```

> [!TIP]
> <span data-ttu-id="c8de1-207">Se l'editor predefinito è Visual Studio Code, in genere si desidera passare l' `-w` argomento o `--wait` per forzare Visual Studio Code ad attendere la chiusura del file prima della restituzione.</span><span class="sxs-lookup"><span data-stu-id="c8de1-207">If your default editor is Visual Studio Code, you'll usually want to pass the `-w` or `--wait` argument to force Visual Studio Code to wait for you to close the file before returning.</span></span>

### <a name="set-the-openapi-description-search-paths"></a><span data-ttu-id="c8de1-208">Imposta i percorsi di ricerca della descrizione OpenAPI</span><span class="sxs-lookup"><span data-stu-id="c8de1-208">Set the OpenAPI Description search paths</span></span>

<span data-ttu-id="c8de1-209">Per impostazione predefinita, HttpRepl dispone di un set di percorsi relativi usato per trovare la descrizione di OpenAPI quando si esegue il `connect` comando senza l' `--openapi` opzione.</span><span class="sxs-lookup"><span data-stu-id="c8de1-209">By default, the HttpRepl has a set of relative paths that it uses to find the OpenAPI description when executing the `connect` command without the `--openapi` option.</span></span> <span data-ttu-id="c8de1-210">Questi percorsi relativi vengono combinati con i percorsi radice e di base specificati nel comando `connect`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-210">These relative paths are combined with the root and base paths specified in the `connect` command.</span></span> <span data-ttu-id="c8de1-211">I percorsi relativi predefiniti sono i seguenti:</span><span class="sxs-lookup"><span data-stu-id="c8de1-211">The default relative paths are:</span></span>

- <span data-ttu-id="c8de1-212">*swagger.js*</span><span class="sxs-lookup"><span data-stu-id="c8de1-212">*swagger.json*</span></span>
- <span data-ttu-id="c8de1-213">*spavalderia/V1/swagger.js*</span><span class="sxs-lookup"><span data-stu-id="c8de1-213">*swagger/v1/swagger.json*</span></span>
- <span data-ttu-id="c8de1-214">*/swagger.js*</span><span class="sxs-lookup"><span data-stu-id="c8de1-214">*/swagger.json*</span></span>
- <span data-ttu-id="c8de1-215">*/swagger/v1/swagger.json*</span><span class="sxs-lookup"><span data-stu-id="c8de1-215">*/swagger/v1/swagger.json*</span></span>
- <span data-ttu-id="c8de1-216">*openapi.js*</span><span class="sxs-lookup"><span data-stu-id="c8de1-216">*openapi.json*</span></span>
- <span data-ttu-id="c8de1-217">*/openapi.js*</span><span class="sxs-lookup"><span data-stu-id="c8de1-217">*/openapi.json*</span></span>

<span data-ttu-id="c8de1-218">Per usare un set di percorsi di ricerca diverso nell'ambiente in uso, impostare la preferenza `swagger.searchPaths`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-218">To use a different set of search paths in your environment, set the `swagger.searchPaths` preference.</span></span> <span data-ttu-id="c8de1-219">Il valore deve essere un elenco di percorsi relativi delimitati da pipe.</span><span class="sxs-lookup"><span data-stu-id="c8de1-219">The value must be a pipe-delimited list of relative paths.</span></span> <span data-ttu-id="c8de1-220">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-220">For example:</span></span>

```console
pref set swagger.searchPaths "swagger/v2/swagger.json|swagger/v3/swagger.json"
```

<span data-ttu-id="c8de1-221">Anziché sostituire completamente l'elenco predefinito, l'elenco può anche essere modificato aggiungendo o rimuovendo percorsi.</span><span class="sxs-lookup"><span data-stu-id="c8de1-221">Instead of replacing the default list altogether, the list can also be modified by adding or removing paths.</span></span>

<span data-ttu-id="c8de1-222">Per aggiungere uno o più percorsi di ricerca all'elenco predefinito, impostare la `swagger.addToSearchPaths` preferenza.</span><span class="sxs-lookup"><span data-stu-id="c8de1-222">To add one or more search paths to the default list, set the `swagger.addToSearchPaths` preference.</span></span> <span data-ttu-id="c8de1-223">Il valore deve essere un elenco di percorsi relativi delimitati da pipe.</span><span class="sxs-lookup"><span data-stu-id="c8de1-223">The value must be a pipe-delimited list of relative paths.</span></span> <span data-ttu-id="c8de1-224">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-224">For example:</span></span>

```console
pref set swagger.addToSearchPaths "openapi/v2/openapi.json|openapi/v3/openapi.json"
```

<span data-ttu-id="c8de1-225">Per rimuovere uno o più percorsi di ricerca dall'elenco predefinito, impostare la `swagger.addToSearchPaths` preferenza.</span><span class="sxs-lookup"><span data-stu-id="c8de1-225">To remove one or more search paths from the default list, set the `swagger.addToSearchPaths` preference.</span></span> <span data-ttu-id="c8de1-226">Il valore deve essere un elenco di percorsi relativi delimitati da pipe.</span><span class="sxs-lookup"><span data-stu-id="c8de1-226">The value must be a pipe-delimited list of relative paths.</span></span> <span data-ttu-id="c8de1-227">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-227">For example:</span></span>

```console
pref set swagger.removeFromSearchPaths "swagger.json|/swagger.json"
```

## <a name="test-http-get-requests"></a><span data-ttu-id="c8de1-228">Testare le richieste HTTP GET</span><span class="sxs-lookup"><span data-stu-id="c8de1-228">Test HTTP GET requests</span></span>

### <a name="synopsis"></a><span data-ttu-id="c8de1-229">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="c8de1-229">Synopsis</span></span>

```console
get <PARAMETER> [-F|--no-formatting] [-h|--header] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a><span data-ttu-id="c8de1-230">Argomenti</span><span class="sxs-lookup"><span data-stu-id="c8de1-230">Arguments</span></span>

`PARAMETER`

<span data-ttu-id="c8de1-231">Parametro di route, se presente, previsto dal metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-231">The route parameter, if any, expected by the associated controller action method.</span></span>

### <a name="options"></a><span data-ttu-id="c8de1-232">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-232">Options</span></span>

<span data-ttu-id="c8de1-233">Per il comando `get` sono disponibili le opzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="c8de1-233">The following options are available for the `get` command:</span></span>

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a><span data-ttu-id="c8de1-234">Esempio</span><span class="sxs-lookup"><span data-stu-id="c8de1-234">Example</span></span>

<span data-ttu-id="c8de1-235">Per inviare una richiesta HTTP GET:</span><span class="sxs-lookup"><span data-stu-id="c8de1-235">To issue an HTTP GET request:</span></span>

1. <span data-ttu-id="c8de1-236">Eseguire il comando `get` in un endpoint che lo supporta:</span><span class="sxs-lookup"><span data-stu-id="c8de1-236">Run the `get` command on an endpoint that supports it:</span></span>

    ```console
    https://localhost:5001/people> get
    ```

    <span data-ttu-id="c8de1-237">Il comando precedente visualizza il formato di output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-237">The preceding command displays the following output format:</span></span>

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 03:38:45 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "name": "Scott Hunter"
      },
      {
        "id": 2,
        "name": "Scott Hanselman"
      },
      {
        "id": 3,
        "name": "Scott Guthrie"
      }
    ]


    https://localhost:5001/people>
    ```

1. <span data-ttu-id="c8de1-238">Recuperare un record specifico passando un parametro al comando `get`:</span><span class="sxs-lookup"><span data-stu-id="c8de1-238">Retrieve a specific record by passing a parameter to the `get` command:</span></span>

    ```console
    https://localhost:5001/people> get 2
    ```

    <span data-ttu-id="c8de1-239">Il comando precedente visualizza il formato di output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-239">The preceding command displays the following output format:</span></span>

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 06:17:57 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 2,
        "name": "Scott Hanselman"
      }
    ]


    https://localhost:5001/people>
    ```

## <a name="test-http-post-requests"></a><span data-ttu-id="c8de1-240">Testare le richieste HTTP POST</span><span class="sxs-lookup"><span data-stu-id="c8de1-240">Test HTTP POST requests</span></span>

### <a name="synopsis"></a><span data-ttu-id="c8de1-241">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="c8de1-241">Synopsis</span></span>

```console
post <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a><span data-ttu-id="c8de1-242">Argomenti</span><span class="sxs-lookup"><span data-stu-id="c8de1-242">Arguments</span></span>

`PARAMETER`

<span data-ttu-id="c8de1-243">Parametro di route, se presente, previsto dal metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-243">The route parameter, if any, expected by the associated controller action method.</span></span>

### <a name="options"></a><span data-ttu-id="c8de1-244">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-244">Options</span></span>

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a><span data-ttu-id="c8de1-245">Esempio</span><span class="sxs-lookup"><span data-stu-id="c8de1-245">Example</span></span>

<span data-ttu-id="c8de1-246">Per inviare una richiesta HTTP POST:</span><span class="sxs-lookup"><span data-stu-id="c8de1-246">To issue an HTTP POST request:</span></span>

1. <span data-ttu-id="c8de1-247">Eseguire il comando `post` in un endpoint che lo supporta:</span><span class="sxs-lookup"><span data-stu-id="c8de1-247">Run the `post` command on an endpoint that supports it:</span></span>

    ```console
    https://localhost:5001/people> post -h Content-Type=application/json
    ```

    <span data-ttu-id="c8de1-248">Nel comando precedente l'intestazione della richiesta HTTP `Content-Type` è impostata per indicare un tipo di supporto del corpo della richiesta JSON.</span><span class="sxs-lookup"><span data-stu-id="c8de1-248">In the preceding command, the `Content-Type` HTTP request header is set to indicate a request body media type of JSON.</span></span> <span data-ttu-id="c8de1-249">L'editor di testo predefinito apre un file con estensione *tmp* con un modello JSON che rappresenta il corpo della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="c8de1-249">The default text editor opens a *.tmp* file with a JSON template representing the HTTP request body.</span></span> <span data-ttu-id="c8de1-250">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-250">For example:</span></span>

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > <span data-ttu-id="c8de1-251">Per impostare l'editor di testo predefinito, vedere la sezione [Impostare l'editor di testo predefinito](#set-the-default-text-editor).</span><span class="sxs-lookup"><span data-stu-id="c8de1-251">To set the default text editor, see the [Set the default text editor](#set-the-default-text-editor) section.</span></span>

1. <span data-ttu-id="c8de1-252">Modificare il modello JSON per soddisfare i requisiti di convalida del modello:</span><span class="sxs-lookup"><span data-stu-id="c8de1-252">Modify the JSON template to satisfy model validation requirements:</span></span>

    ```json
    {
      "id": 0,
      "name": "Scott Addie"
    }
    ```

1. <span data-ttu-id="c8de1-253">Salvare il file con estensione *tmp* e chiudere l'editor di testo.</span><span class="sxs-lookup"><span data-stu-id="c8de1-253">Save the *.tmp* file, and close the text editor.</span></span> <span data-ttu-id="c8de1-254">Nella shell dei comandi viene visualizzato l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-254">The following output appears in the command shell:</span></span>

    ```console
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Date: Thu, 27 Jun 2019 21:24:18 GMT
    Location: https://localhost:5001/people/4
    Server: Kestrel
    Transfer-Encoding: chunked

    {
      "id": 4,
      "name": "Scott Addie"
    }


    https://localhost:5001/people>
    ```

## <a name="test-http-put-requests"></a><span data-ttu-id="c8de1-255">Testare le richieste HTTP PUT</span><span class="sxs-lookup"><span data-stu-id="c8de1-255">Test HTTP PUT requests</span></span>

### <a name="synopsis"></a><span data-ttu-id="c8de1-256">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="c8de1-256">Synopsis</span></span>

```console
put <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a><span data-ttu-id="c8de1-257">Argomenti</span><span class="sxs-lookup"><span data-stu-id="c8de1-257">Arguments</span></span>

`PARAMETER`

<span data-ttu-id="c8de1-258">Parametro di route, se presente, previsto dal metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-258">The route parameter, if any, expected by the associated controller action method.</span></span>

### <a name="options"></a><span data-ttu-id="c8de1-259">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-259">Options</span></span>

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a><span data-ttu-id="c8de1-260">Esempio</span><span class="sxs-lookup"><span data-stu-id="c8de1-260">Example</span></span>

<span data-ttu-id="c8de1-261">Per inviare una richiesta HTTP PUT:</span><span class="sxs-lookup"><span data-stu-id="c8de1-261">To issue an HTTP PUT request:</span></span>

1. <span data-ttu-id="c8de1-262">*Facoltativo* : eseguire il `get` comando per visualizzare i dati prima di modificarli:</span><span class="sxs-lookup"><span data-stu-id="c8de1-262">*Optional* : Run the `get` command to view the data before modifying it:</span></span>

    ```console
    https://localhost:5001/fruits> get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]
    ```

1. <span data-ttu-id="c8de1-263">Eseguire il comando `put` in un endpoint che lo supporta:</span><span class="sxs-lookup"><span data-stu-id="c8de1-263">Run the `put` command on an endpoint that supports it:</span></span>

    ```console
    https://localhost:5001/fruits> put 2 -h Content-Type=application/json
    ```

    <span data-ttu-id="c8de1-264">Nel comando precedente l'intestazione della richiesta HTTP `Content-Type` è impostata per indicare un tipo di supporto del corpo della richiesta JSON.</span><span class="sxs-lookup"><span data-stu-id="c8de1-264">In the preceding command, the `Content-Type` HTTP request header is set to indicate a request body media type of JSON.</span></span> <span data-ttu-id="c8de1-265">L'editor di testo predefinito apre un file con estensione *tmp* con un modello JSON che rappresenta il corpo della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="c8de1-265">The default text editor opens a *.tmp* file with a JSON template representing the HTTP request body.</span></span> <span data-ttu-id="c8de1-266">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-266">For example:</span></span>

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > <span data-ttu-id="c8de1-267">Per impostare l'editor di testo predefinito, vedere la sezione [Impostare l'editor di testo predefinito](#set-the-default-text-editor).</span><span class="sxs-lookup"><span data-stu-id="c8de1-267">To set the default text editor, see the [Set the default text editor](#set-the-default-text-editor) section.</span></span>

1. <span data-ttu-id="c8de1-268">Modificare il modello JSON per soddisfare i requisiti di convalida del modello:</span><span class="sxs-lookup"><span data-stu-id="c8de1-268">Modify the JSON template to satisfy model validation requirements:</span></span>

    ```json
    {
      "id": 2,
      "name": "Cherry"
    }
    ```

1. <span data-ttu-id="c8de1-269">Salvare il file con estensione *tmp* e chiudere l'editor di testo.</span><span class="sxs-lookup"><span data-stu-id="c8de1-269">Save the *.tmp* file, and close the text editor.</span></span> <span data-ttu-id="c8de1-270">Nella shell dei comandi viene visualizzato l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-270">The following output appears in the command shell:</span></span>

    ```console
    [main 2019-06-28T17:27:01.805Z] update#setState idle
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:28:21 GMT
    Server: Kestrel
    ```

1. <span data-ttu-id="c8de1-271">*Facoltativo* : eseguire un `get` comando per visualizzare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="c8de1-271">*Optional* : Issue a `get` command to see the modifications.</span></span> <span data-ttu-id="c8de1-272">Se ad esempio è stato digitato "Cherry" nell'editor di testo, `get` restituisce l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-272">For example, if you typed "Cherry" in the text editor, a `get` returns the following output:</span></span>

    ```console
    https://localhost:5001/fruits> get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:08:20 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Cherry"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits>
    ```

## <a name="test-http-delete-requests"></a><span data-ttu-id="c8de1-273">Testare le richieste HTTP DELETE</span><span class="sxs-lookup"><span data-stu-id="c8de1-273">Test HTTP DELETE requests</span></span>

### <a name="synopsis"></a><span data-ttu-id="c8de1-274">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="c8de1-274">Synopsis</span></span>

```console
delete <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a><span data-ttu-id="c8de1-275">Argomenti</span><span class="sxs-lookup"><span data-stu-id="c8de1-275">Arguments</span></span>

`PARAMETER`

<span data-ttu-id="c8de1-276">Parametro di route, se presente, previsto dal metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-276">The route parameter, if any, expected by the associated controller action method.</span></span>

### <a name="options"></a><span data-ttu-id="c8de1-277">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-277">Options</span></span>

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a><span data-ttu-id="c8de1-278">Esempio</span><span class="sxs-lookup"><span data-stu-id="c8de1-278">Example</span></span>

<span data-ttu-id="c8de1-279">Per inviare una richiesta HTTP DELETE:</span><span class="sxs-lookup"><span data-stu-id="c8de1-279">To issue an HTTP DELETE request:</span></span>

1. <span data-ttu-id="c8de1-280">*Facoltativo* : eseguire il `get` comando per visualizzare i dati prima di modificarli:</span><span class="sxs-lookup"><span data-stu-id="c8de1-280">*Optional* : Run the `get` command to view the data before modifying it:</span></span>

    ```console
    https://localhost:5001/fruits> get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]
    ```

1. <span data-ttu-id="c8de1-281">Eseguire il comando `delete` in un endpoint che lo supporta:</span><span class="sxs-lookup"><span data-stu-id="c8de1-281">Run the `delete` command on an endpoint that supports it:</span></span>

    ```console
    https://localhost:5001/fruits> delete 2
    ```

    <span data-ttu-id="c8de1-282">Il comando precedente visualizza il formato di output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-282">The preceding command displays the following output format:</span></span>

    ```console
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:36:42 GMT
    Server: Kestrel
    ```

1. <span data-ttu-id="c8de1-283">*Facoltativo* : eseguire un `get` comando per visualizzare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="c8de1-283">*Optional* : Issue a `get` command to see the modifications.</span></span> <span data-ttu-id="c8de1-284">In questo esempio, `get` restituisce l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-284">In this example, a `get` returns the following output:</span></span>

    ```console
    https://localhost:5001/fruits> get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:16:30 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits>
    ```

## <a name="test-http-patch-requests"></a><span data-ttu-id="c8de1-285">Testare le richieste HTTP PATCH</span><span class="sxs-lookup"><span data-stu-id="c8de1-285">Test HTTP PATCH requests</span></span>

### <a name="synopsis"></a><span data-ttu-id="c8de1-286">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="c8de1-286">Synopsis</span></span>

```console
patch <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a><span data-ttu-id="c8de1-287">Argomenti</span><span class="sxs-lookup"><span data-stu-id="c8de1-287">Arguments</span></span>

`PARAMETER`

<span data-ttu-id="c8de1-288">Parametro di route, se presente, previsto dal metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-288">The route parameter, if any, expected by the associated controller action method.</span></span>

### <a name="options"></a><span data-ttu-id="c8de1-289">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-289">Options</span></span>

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

## <a name="test-http-head-requests"></a><span data-ttu-id="c8de1-290">Testare le richieste HTTP HEAD</span><span class="sxs-lookup"><span data-stu-id="c8de1-290">Test HTTP HEAD requests</span></span>

### <a name="synopsis"></a><span data-ttu-id="c8de1-291">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="c8de1-291">Synopsis</span></span>

```console
head <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a><span data-ttu-id="c8de1-292">Argomenti</span><span class="sxs-lookup"><span data-stu-id="c8de1-292">Arguments</span></span>

`PARAMETER`

<span data-ttu-id="c8de1-293">Parametro di route, se presente, previsto dal metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-293">The route parameter, if any, expected by the associated controller action method.</span></span>

### <a name="options"></a><span data-ttu-id="c8de1-294">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-294">Options</span></span>

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="test-http-options-requests"></a><span data-ttu-id="c8de1-295">Testare le richieste HTTP OPTIONS</span><span class="sxs-lookup"><span data-stu-id="c8de1-295">Test HTTP OPTIONS requests</span></span>

### <a name="synopsis"></a><span data-ttu-id="c8de1-296">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="c8de1-296">Synopsis</span></span>

```console
options <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a><span data-ttu-id="c8de1-297">Argomenti</span><span class="sxs-lookup"><span data-stu-id="c8de1-297">Arguments</span></span>

`PARAMETER`

<span data-ttu-id="c8de1-298">Parametro di route, se presente, previsto dal metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="c8de1-298">The route parameter, if any, expected by the associated controller action method.</span></span>

### <a name="options"></a><span data-ttu-id="c8de1-299">Opzioni</span><span class="sxs-lookup"><span data-stu-id="c8de1-299">Options</span></span>

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="set-http-request-headers"></a><span data-ttu-id="c8de1-300">Impostare le intestazioni delle richieste HTTP</span><span class="sxs-lookup"><span data-stu-id="c8de1-300">Set HTTP request headers</span></span>

<span data-ttu-id="c8de1-301">Per impostare l'intestazione di una richiesta HTTP, usare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="c8de1-301">To set an HTTP request header, use one of the following approaches:</span></span>

* <span data-ttu-id="c8de1-302">Impostare inline con la richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="c8de1-302">Set inline with the HTTP request.</span></span> <span data-ttu-id="c8de1-303">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-303">For example:</span></span>

    ```console
    https://localhost:5001/people> post -h Content-Type=application/json
    ```
    
    <span data-ttu-id="c8de1-304">Con l'approccio precedente, ogni singola intestazione di richiesta HTTP richiede una propria opzione `-h`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-304">With the preceding approach, each distinct HTTP request header requires its own `-h` option.</span></span>

* <span data-ttu-id="c8de1-305">Impostare prima di inviare la richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="c8de1-305">Set before sending the HTTP request.</span></span> <span data-ttu-id="c8de1-306">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-306">For example:</span></span>

    ```console
    https://localhost:5001/people> set header Content-Type application/json
    ```
    
    <span data-ttu-id="c8de1-307">Quando si imposta l'intestazione prima di inviare una richiesta, l'intestazione rimane impostata per la durata della sessione della shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="c8de1-307">When setting the header before sending a request, the header remains set for the duration of the command shell session.</span></span> <span data-ttu-id="c8de1-308">Per cancellare l'intestazione, specificare un valore vuoto.</span><span class="sxs-lookup"><span data-stu-id="c8de1-308">To clear the header, provide an empty value.</span></span> <span data-ttu-id="c8de1-309">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-309">For example:</span></span>
    
    ```console
    https://localhost:5001/people> set header Content-Type
    ```

## <a name="test-secured-endpoints"></a><span data-ttu-id="c8de1-310">Testare endpoint protetti</span><span class="sxs-lookup"><span data-stu-id="c8de1-310">Test secured endpoints</span></span>

<span data-ttu-id="c8de1-311">HttpRepl supporta il test degli endpoint protetti nei modi seguenti:</span><span class="sxs-lookup"><span data-stu-id="c8de1-311">The HttpRepl supports the testing of secured endpoints in the following ways:</span></span>

* <span data-ttu-id="c8de1-312">Tramite le credenziali predefinite dell'utente connesso.</span><span class="sxs-lookup"><span data-stu-id="c8de1-312">Via the default credentials of the logged in user.</span></span>
* <span data-ttu-id="c8de1-313">Tramite l'utilizzo delle intestazioni della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="c8de1-313">Through the use of HTTP request headers.</span></span>

### <a name="default-credentials"></a><span data-ttu-id="c8de1-314">Credenziali predefinite</span><span class="sxs-lookup"><span data-stu-id="c8de1-314">Default credentials</span></span>

<span data-ttu-id="c8de1-315">Si consideri un'API Web che si sta testando, ospitata in IIS e protetta con l'autenticazione di Windows.</span><span class="sxs-lookup"><span data-stu-id="c8de1-315">Consider a web API you're testing that's hosted in IIS and secured with Windows authentication.</span></span> <span data-ttu-id="c8de1-316">Si desidera che le credenziali dell'utente che esegue lo strumento scorrano tra gli endpoint HTTP sottoposti a test.</span><span class="sxs-lookup"><span data-stu-id="c8de1-316">You want the credentials of the user running the tool to flow across to the HTTP endpoints being tested.</span></span> <span data-ttu-id="c8de1-317">Per passare le credenziali predefinite dell'utente connesso:</span><span class="sxs-lookup"><span data-stu-id="c8de1-317">To pass the default credentials of the logged in user:</span></span>

1. <span data-ttu-id="c8de1-318">Impostare la `httpClient.useDefaultCredentials` preferenza su `true` :</span><span class="sxs-lookup"><span data-stu-id="c8de1-318">Set the `httpClient.useDefaultCredentials` preference to `true`:</span></span>

    ```console
    pref set httpClient.useDefaultCredentials true
    ```

1. <span data-ttu-id="c8de1-319">Chiudere e riavviare lo strumento prima di inviare un'altra richiesta all'API Web.</span><span class="sxs-lookup"><span data-stu-id="c8de1-319">Exit and restart the tool before sending another request to the web API.</span></span>
 
### <a name="default-proxy-credentials"></a><span data-ttu-id="c8de1-320">Credenziali proxy predefinite</span><span class="sxs-lookup"><span data-stu-id="c8de1-320">Default proxy credentials</span></span>

<span data-ttu-id="c8de1-321">Si consideri uno scenario in cui l'API Web che si sta testando si trova dietro un proxy protetto con l'autenticazione di Windows.</span><span class="sxs-lookup"><span data-stu-id="c8de1-321">Consider a scenario in which the web API you're testing is behind a proxy secured with Windows authentication.</span></span> <span data-ttu-id="c8de1-322">Si desidera che le credenziali dell'utente che esegue lo strumento fluiscano sul proxy.</span><span class="sxs-lookup"><span data-stu-id="c8de1-322">You want the credentials of the user running the tool to flow to the proxy.</span></span> <span data-ttu-id="c8de1-323">Per passare le credenziali predefinite dell'utente connesso:</span><span class="sxs-lookup"><span data-stu-id="c8de1-323">To pass the default credentials of the logged in user:</span></span>

1. <span data-ttu-id="c8de1-324">Impostare la `httpClient.proxy.useDefaultCredentials` preferenza su `true` :</span><span class="sxs-lookup"><span data-stu-id="c8de1-324">Set the `httpClient.proxy.useDefaultCredentials` preference to `true`:</span></span>

    ```console
    pref set httpClient.proxy.useDefaultCredentials true
    ```

1. <span data-ttu-id="c8de1-325">Chiudere e riavviare lo strumento prima di inviare un'altra richiesta all'API Web.</span><span class="sxs-lookup"><span data-stu-id="c8de1-325">Exit and restart the tool before sending another request to the web API.</span></span>

### <a name="http-request-headers"></a><span data-ttu-id="c8de1-326">Intestazioni di richiesta HTTP</span><span class="sxs-lookup"><span data-stu-id="c8de1-326">HTTP request headers</span></span>

<span data-ttu-id="c8de1-327">Esempi di schemi di autenticazione e autorizzazione supportati includono:</span><span class="sxs-lookup"><span data-stu-id="c8de1-327">Examples of supported authentication and authorization schemes include:</span></span>

* <span data-ttu-id="c8de1-328">basic authentication</span><span class="sxs-lookup"><span data-stu-id="c8de1-328">basic authentication</span></span>
* <span data-ttu-id="c8de1-329">Token di porta JWT</span><span class="sxs-lookup"><span data-stu-id="c8de1-329">JWT bearer tokens</span></span>
* <span data-ttu-id="c8de1-330">autenticazione digest</span><span class="sxs-lookup"><span data-stu-id="c8de1-330">digest authentication</span></span>

<span data-ttu-id="c8de1-331">Ad esempio, è possibile inviare un bearer token a un endpoint con il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-331">For example, you can send a bearer token to an endpoint with the following command:</span></span>

```console
set header Authorization "bearer <TOKEN VALUE>"
```

<span data-ttu-id="c8de1-332">Per accedere a un endpoint ospitato da Azure o per usare l' [API REST di Azure](/rest/api/azure/), è necessario un Bearer token.</span><span class="sxs-lookup"><span data-stu-id="c8de1-332">To access an Azure-hosted endpoint or to use the [Azure REST API](/rest/api/azure/), you need a bearer token.</span></span> <span data-ttu-id="c8de1-333">Usare la procedura seguente per ottenere un bearer token per la sottoscrizione di Azure tramite l'interfaccia della riga di comando di [Azure](/cli/azure/).</span><span class="sxs-lookup"><span data-stu-id="c8de1-333">Use the following steps to obtain a bearer token for your Azure subscription via the [Azure CLI](/cli/azure/).</span></span> <span data-ttu-id="c8de1-334">HttpRepl imposta il bearer token in un'intestazione della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="c8de1-334">The HttpRepl sets the bearer token in an HTTP request header.</span></span> <span data-ttu-id="c8de1-335">Viene recuperato un elenco di app Azure app Web del servizio.</span><span class="sxs-lookup"><span data-stu-id="c8de1-335">A list of Azure App Service Web Apps is retrieved.</span></span>

1. <span data-ttu-id="c8de1-336">Accedere ad Azure:</span><span class="sxs-lookup"><span data-stu-id="c8de1-336">Sign in to Azure:</span></span>

    ```azurecli
    az login
    ```

1. <span data-ttu-id="c8de1-337">Ottenere l'ID sottoscrizione con il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-337">Get your subscription ID with the following command:</span></span>

    ```azurecli
    az account show --query id
    ```

1. <span data-ttu-id="c8de1-338">Copiare l'ID sottoscrizione ed eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-338">Copy your subscription ID and run the following command:</span></span>

    ```azurecli
    az account set --subscription "<SUBSCRIPTION ID>"
    ```

1. <span data-ttu-id="c8de1-339">Ottenere il bearer token con il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-339">Get your bearer token with the following command:</span></span>

    ```azurecli
    az account get-access-token --query accessToken
    ```

1. <span data-ttu-id="c8de1-340">Connettersi all'API REST di Azure tramite HttpRepl:</span><span class="sxs-lookup"><span data-stu-id="c8de1-340">Connect to the Azure REST API via the HttpRepl:</span></span>

    ```console
    httprepl https://management.azure.com
    ```

1. <span data-ttu-id="c8de1-341">Impostare l' `Authorization` intestazione della richiesta http:</span><span class="sxs-lookup"><span data-stu-id="c8de1-341">Set the `Authorization` HTTP request header:</span></span>

    ```console
    https://management.azure.com/> set header Authorization "bearer <ACCESS TOKEN>"
    ```

1. <span data-ttu-id="c8de1-342">Passare alla sottoscrizione:</span><span class="sxs-lookup"><span data-stu-id="c8de1-342">Navigate to the subscription:</span></span>

    ```console
    https://management.azure.com/> cd subscriptions/<SUBSCRIPTION ID>
    ```

1. <span data-ttu-id="c8de1-343">Ottenere un elenco delle app Web del servizio app Azure della sottoscrizione:</span><span class="sxs-lookup"><span data-stu-id="c8de1-343">Get a list of your subscription's Azure App Service Web Apps:</span></span>

    ```console
    https://management.azure.com/subscriptions/{SUBSCRIPTION ID}> get providers/Microsoft.Web/sites?api-version=2016-08-01
    ```

    <span data-ttu-id="c8de1-344">Viene visualizzata la risposta seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-344">The following response is displayed:</span></span>

    ```console
    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 35948
    Content-Type: application/json; charset=utf-8
    Date: Thu, 19 Sep 2019 23:04:03 GMT
    Expires: -1
    Pragma: no-cache
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    X-Content-Type-Options: nosniff
    x-ms-correlation-request-id: <em>xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</em>
    x-ms-original-request-ids: <em>xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx;xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</em>
    x-ms-ratelimit-remaining-subscription-reads: 11999
    x-ms-request-id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    x-ms-routing-request-id: WESTUS:xxxxxxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx
    {
      "value": [
        <AZURE RESOURCES LIST>
      ]
    }
    ```

## <a name="toggle-http-request-display"></a><span data-ttu-id="c8de1-345">Attivare o disattivare la visualizzazione delle richieste HTTP</span><span class="sxs-lookup"><span data-stu-id="c8de1-345">Toggle HTTP request display</span></span>

<span data-ttu-id="c8de1-346">Per impostazione predefinita, la visualizzazione della richiesta HTTP inviata viene eliminata.</span><span class="sxs-lookup"><span data-stu-id="c8de1-346">By default, display of the HTTP request being sent is suppressed.</span></span> <span data-ttu-id="c8de1-347">È possibile modificare l'impostazione corrispondente per la durata della sessione della shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="c8de1-347">It's possible to change the corresponding setting for the duration of the command shell session.</span></span>

### <a name="enable-request-display"></a><span data-ttu-id="c8de1-348">Abilitare la visualizzazione delle richieste</span><span class="sxs-lookup"><span data-stu-id="c8de1-348">Enable request display</span></span>

<span data-ttu-id="c8de1-349">Visualizzare la richiesta HTTP inviata eseguendo il comando `echo on`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-349">View the HTTP request being sent by running the `echo on` command.</span></span> <span data-ttu-id="c8de1-350">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-350">For example:</span></span>

```console
https://localhost:5001/people> echo on
Request echoing is on
```

<span data-ttu-id="c8de1-351">Le richieste HTTP successive della sessione corrente visualizzano le intestazioni delle richieste.</span><span class="sxs-lookup"><span data-stu-id="c8de1-351">Subsequent HTTP requests in the current session display the request headers.</span></span> <span data-ttu-id="c8de1-352">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-352">For example:</span></span>

```console
https://localhost:5001/people> post

[main 2019-06-28T18:50:11.930Z] update#setState idle
Request to https://localhost:5001...

POST /people HTTP/1.1
Content-Length: 41
Content-Type: application/json
User-Agent: HTTP-REPL

{
  "id": 0,
  "name": "Scott Addie"
}

Response from https://localhost:5001...

HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Jun 2019 18:50:21 GMT
Location: https://localhost:5001/people/4
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 4,
  "name": "Scott Addie"
}


https://localhost:5001/people>
```

### <a name="disable-request-display"></a><span data-ttu-id="c8de1-353">Disabilitare la visualizzazione delle richieste</span><span class="sxs-lookup"><span data-stu-id="c8de1-353">Disable request display</span></span>

<span data-ttu-id="c8de1-354">Eliminare la visualizzazione della richiesta HTTP inviata eseguendo il comando `echo off`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-354">Suppress display of the HTTP request being sent by running the `echo off` command.</span></span> <span data-ttu-id="c8de1-355">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-355">For example:</span></span>

```console
https://localhost:5001/people> echo off
Request echoing is off
```

## <a name="run-a-script"></a><span data-ttu-id="c8de1-356">Esegui uno script</span><span class="sxs-lookup"><span data-stu-id="c8de1-356">Run a script</span></span>

<span data-ttu-id="c8de1-357">Se si esegue spesso lo stesso set di comandi HttpRepl, è consigliabile archiviarli in un file di testo.</span><span class="sxs-lookup"><span data-stu-id="c8de1-357">If you frequently execute the same set of HttpRepl commands, consider storing them in a text file.</span></span> <span data-ttu-id="c8de1-358">I comandi nel file hanno lo stesso formato dei comandi eseguiti manualmente nella riga di comando.</span><span class="sxs-lookup"><span data-stu-id="c8de1-358">Commands in the file take the same form as commands executed manually on the command line.</span></span> <span data-ttu-id="c8de1-359">I comandi possono essere eseguiti in modalità batch usando il comando `run`.</span><span class="sxs-lookup"><span data-stu-id="c8de1-359">The commands can be executed in a batched fashion using the `run` command.</span></span> <span data-ttu-id="c8de1-360">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-360">For example:</span></span>

1. <span data-ttu-id="c8de1-361">Creare un file di testo contenente un set di comandi delimitati da una nuova riga.</span><span class="sxs-lookup"><span data-stu-id="c8de1-361">Create a text file containing a set of newline-delimited commands.</span></span> <span data-ttu-id="c8de1-362">Si consideri ad esempio un file *people-script.txt* contenente i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="c8de1-362">To illustrate, consider a *people-script.txt* file containing the following commands:</span></span>

    ```text
    set base https://localhost:5001
    ls
    cd People
    ls
    get 1
    ```

1. <span data-ttu-id="c8de1-363">Eseguire il comando `run` passando il percorso del file di testo.</span><span class="sxs-lookup"><span data-stu-id="c8de1-363">Execute the `run` command, passing in the text file's path.</span></span> <span data-ttu-id="c8de1-364">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c8de1-364">For example:</span></span>

    ```console
    https://localhost:5001/> run C:\http-repl-scripts\people-script.txt
    ```

    <span data-ttu-id="c8de1-365">Viene visualizzato l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-365">The following output appears:</span></span>

    ```console
    https://localhost:5001/> set base https://localhost:5001
    Using OpenAPI description at https://localhost:5001/swagger/v1/swagger.json
    
    https://localhost:5001/> ls
    .        []
    Fruits   [get|post]
    People   [get|post]
    
    https://localhost:5001/> cd People
    /People    [get|post]
    
    https://localhost:5001/People> ls
    .      [get|post]
    ..     []
    {id}   [get|put|delete]
    
    https://localhost:5001/People> get 1
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 12 Jul 2019 19:20:10 GMT
    Server: Kestrel
    Transfer-Encoding: chunked
    
    {
      "id": 1,
      "name": "Scott Hunter"
    }
    
    
    https://localhost:5001/People>
    ```

## <a name="clear-the-output"></a><span data-ttu-id="c8de1-366">Cancellare l'output</span><span class="sxs-lookup"><span data-stu-id="c8de1-366">Clear the output</span></span>

<span data-ttu-id="c8de1-367">Per rimuovere tutti i risultati scritti nella shell dei comandi mediante lo strumento HttpRepl, eseguire `clear` il `cls` comando o.</span><span class="sxs-lookup"><span data-stu-id="c8de1-367">To remove all output written to the command shell by the HttpRepl tool, run the `clear` or `cls` command.</span></span> <span data-ttu-id="c8de1-368">Si supponga ad esempio che la shell dei comandi includa l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-368">To illustrate, imagine the command shell contains the following output:</span></span>

```console
httprepl https://localhost:5001
(Disconnected)> set base "https://localhost:5001"
Using OpenAPI description at https://localhost:5001/swagger/v1/swagger.json

https://localhost:5001/> ls
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/>
```

<span data-ttu-id="c8de1-369">Eseguire il comando seguente per cancellare l'output:</span><span class="sxs-lookup"><span data-stu-id="c8de1-369">Run the following command to clear the output:</span></span>

```console
https://localhost:5001/> clear
```

<span data-ttu-id="c8de1-370">Dopo aver eseguito il comando precedente, la shell dei comandi contiene solo l'output seguente:</span><span class="sxs-lookup"><span data-stu-id="c8de1-370">After running the preceding command, the command shell contains only the following output:</span></span>

```console
https://localhost:5001/>
```

## <a name="additional-resources"></a><span data-ttu-id="c8de1-371">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="c8de1-371">Additional resources</span></span>

* [<span data-ttu-id="c8de1-372">Richieste API REST</span><span class="sxs-lookup"><span data-stu-id="c8de1-372">REST API requests</span></span>](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods)
* [<span data-ttu-id="c8de1-373">Repository GitHub HttpRepl</span><span class="sxs-lookup"><span data-stu-id="c8de1-373">HttpRepl GitHub repository</span></span>](https://github.com/dotnet/HttpRepl)
* [<span data-ttu-id="c8de1-374">Configurare Visual Studio per avviare HttpRepl</span><span class="sxs-lookup"><span data-stu-id="c8de1-374">Configure Visual Studio to launch HttpRepl</span></span>](https://devblogs.microsoft.com/aspnet/httprepl-a-command-line-tool-for-interacting-with-restful-http-services/#configure-visual-studio-for-windows-to-launch-httprepl-on-f5)
* [<span data-ttu-id="c8de1-375">Configurare Visual Studio Code per avviare HttpRepl</span><span class="sxs-lookup"><span data-stu-id="c8de1-375">Configure Visual Studio Code to launch HttpRepl</span></span>](https://devblogs.microsoft.com/aspnet/httprepl-a-command-line-tool-for-interacting-with-restful-http-services/#configure-visual-studio-code-to-launch-httprepl-on-debug)
* [<span data-ttu-id="c8de1-376">Configurare Visual Studio per Mac per avviare HttpRepl</span><span class="sxs-lookup"><span data-stu-id="c8de1-376">Configure Visual Studio for Mac to launch HttpRepl</span></span>](https://devblogs.microsoft.com/aspnet/httprepl-a-command-line-tool-for-interacting-with-restful-http-services/#configure-visual-studio-for-mac-to-launch-httprepl-as-a-custom-tool)