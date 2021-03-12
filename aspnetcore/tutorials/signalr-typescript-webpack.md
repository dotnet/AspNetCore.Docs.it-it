---
title: Usare ASP.NET Core SignalR con typescript e Webpack
author: ssougnez
description: In questa esercitazione viene configurato Webpack per aggregare e compilare un' SignalR app web ASP.NET Core il cui client è scritto in typescript.
ms.author: bradyg
ms.custom: mvc
ms.date: 02/10/2020
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
uid: tutorials/signalr-typescript-webpack
ms.openlocfilehash: 9d52b72c669a9345cc7c5386876db22af55a97c7
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589751"
---
# <a name="use-aspnet-core-signalr-with-typescript-and-webpack"></a><span data-ttu-id="19375-103">Usare ASP.NET Core SignalR con typescript e Webpack</span><span class="sxs-lookup"><span data-stu-id="19375-103">Use ASP.NET Core SignalR with TypeScript and Webpack</span></span>

<span data-ttu-id="19375-104">Di [Sébastien Sougnez](https://twitter.com/ssougnez) e [Scott Addie](https://twitter.com/Scott_Addie)</span><span class="sxs-lookup"><span data-stu-id="19375-104">By [Sébastien Sougnez](https://twitter.com/ssougnez) and [Scott Addie](https://twitter.com/Scott_Addie)</span></span>

<span data-ttu-id="19375-105">[Webpack](https://webpack.js.org/) consente agli sviluppatori di creare un bundle delle risorse di un'app Web sul lato client-e di compilarle.</span><span class="sxs-lookup"><span data-stu-id="19375-105">[Webpack](https://webpack.js.org/) enables developers to bundle and build the client-side resources of a web app.</span></span> <span data-ttu-id="19375-106">Questa esercitazione illustra l'uso di Webpack in un' SignalR app web ASP.NET Core il cui client è scritto in [typescript](https://www.typescriptlang.org/).</span><span class="sxs-lookup"><span data-stu-id="19375-106">This tutorial demonstrates using Webpack in an ASP.NET Core SignalR web app whose client is written in [TypeScript](https://www.typescriptlang.org/).</span></span>

<span data-ttu-id="19375-107">In questa esercitazione verranno illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="19375-107">In this tutorial, you learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="19375-108">Impalcatura di un' SignalR app starter ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="19375-108">Scaffold a starter ASP.NET Core SignalR app</span></span>
> * <span data-ttu-id="19375-109">Configurare il SignalR client typescript</span><span class="sxs-lookup"><span data-stu-id="19375-109">Configure the SignalR TypeScript client</span></span>
> * <span data-ttu-id="19375-110">Configurare una pipeline di compilazione tramite Webpack</span><span class="sxs-lookup"><span data-stu-id="19375-110">Configure a build pipeline using Webpack</span></span>
> * <span data-ttu-id="19375-111">Configurare il SignalR Server</span><span class="sxs-lookup"><span data-stu-id="19375-111">Configure the SignalR server</span></span>
> * <span data-ttu-id="19375-112">Abilitare la comunicazione tra client e server</span><span class="sxs-lookup"><span data-stu-id="19375-112">Enable communication between client and server</span></span>

<span data-ttu-id="19375-113">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/signalr-typescript-webpack/sample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="19375-113">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/signalr-typescript-webpack/sample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a><span data-ttu-id="19375-114">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="19375-114">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="19375-115">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="19375-115">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="19375-116">[Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="19375-116">[Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [<span data-ttu-id="19375-117">.NET Core SDK 3.0 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="19375-117">.NET Core SDK 3.0 or later</span></span>](https://dotnet.microsoft.com/download/dotnet-core)
* <span data-ttu-id="19375-118">[Node.js](https://nodejs.org/) con [npm](https://www.npmjs.com/)</span><span class="sxs-lookup"><span data-stu-id="19375-118">[Node.js](https://nodejs.org/) with [npm](https://www.npmjs.com/)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="19375-119">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-119">Visual Studio Code</span></span>](#tab/visual-studio-code)

* [<span data-ttu-id="19375-120">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-120">Visual Studio Code</span></span>](https://code.visualstudio.com/download)
* [<span data-ttu-id="19375-121">.NET Core SDK 3.0 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="19375-121">.NET Core SDK 3.0 or later</span></span>](https://dotnet.microsoft.com/download/dotnet-core)
* [<span data-ttu-id="19375-122">C# per Visual Studio Code 1.17.1 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="19375-122">C# for Visual Studio Code version 1.17.1 or later</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* <span data-ttu-id="19375-123">[Node.js](https://nodejs.org/) con [npm](https://www.npmjs.com/)</span><span class="sxs-lookup"><span data-stu-id="19375-123">[Node.js](https://nodejs.org/) with [npm](https://www.npmjs.com/)</span></span>

---

## <a name="create-the-aspnet-core-web-app"></a><span data-ttu-id="19375-124">Creare l'app Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="19375-124">Create the ASP.NET Core web app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="19375-125">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="19375-125">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="19375-126">Configurare Visual Studio in modo che cerchi npm nella variabile di ambiente *PATH*.</span><span class="sxs-lookup"><span data-stu-id="19375-126">Configure Visual Studio to look for npm in the *PATH* environment variable.</span></span> <span data-ttu-id="19375-127">Per impostazione predefinita, Visual Studio usa la versione di npm trovata nella directory di installazione.</span><span class="sxs-lookup"><span data-stu-id="19375-127">By default, Visual Studio uses the version of npm found in its installation directory.</span></span> <span data-ttu-id="19375-128">Seguire queste istruzioni in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="19375-128">Follow these instructions in Visual Studio:</span></span>

1. <span data-ttu-id="19375-129">Avviare Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="19375-129">Launch Visual Studio.</span></span> <span data-ttu-id="19375-130">Nella finestra iniziale selezionare **continua senza codice**.</span><span class="sxs-lookup"><span data-stu-id="19375-130">At the start window, select **Continue without code**.</span></span>
1. <span data-ttu-id="19375-131">Passare a **strumenti** > **Opzioni** > **progetti e soluzioni** > **Web Gestione pacchetti** > **strumenti Web esterni**.</span><span class="sxs-lookup"><span data-stu-id="19375-131">Navigate to **Tools** > **Options** > **Projects and Solutions** > **Web Package Management** > **External Web Tools**.</span></span>
1. <span data-ttu-id="19375-132">Selezionare la voce *$(PATH)* dall'elenco.</span><span class="sxs-lookup"><span data-stu-id="19375-132">Select the *$(PATH)* entry from the list.</span></span> <span data-ttu-id="19375-133">Fare clic sulla freccia in su per spostare la voce nella seconda posizione nell'elenco e selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="19375-133">Click the up arrow to move the entry to the second position in the list, and select **OK**.</span></span>

    ![Configurazione di Visual Studio](signalr-typescript-webpack/_static/signalr-configure-path-visual-studio.png)

<span data-ttu-id="19375-135">La configurazione di Visual Studio è stata completata.</span><span class="sxs-lookup"><span data-stu-id="19375-135">Visual Studio configuration is complete.</span></span>

1. <span data-ttu-id="19375-136">Usare l'opzione di menu **file**  >  **nuovo**  >  **progetto** e scegliere il modello **applicazione Web ASP.NET Core** .</span><span class="sxs-lookup"><span data-stu-id="19375-136">Use the **File** > **New** > **Project** menu option and choose the **ASP.NET Core Web Application** template.</span></span> <span data-ttu-id="19375-137">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="19375-137">Select **Next**.</span></span>
1. <span data-ttu-id="19375-138">Assegnare al progetto il nome *SignalR Webpack* e selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="19375-138">Name the project *SignalRWebPack*, and select **Create**.</span></span>
1. <span data-ttu-id="19375-139">Selezionare *.NET Core* dall'elenco a discesa Framework di destinazione e selezionare *ASP.NET Core 3,1* dall'elenco a discesa del selettore del Framework.</span><span class="sxs-lookup"><span data-stu-id="19375-139">Select *.NET Core* from the target framework drop-down, and select *ASP.NET Core 3.1* from the framework selector drop-down.</span></span> <span data-ttu-id="19375-140">Selezionare il modello **vuoto** e selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="19375-140">Select the **Empty** template, and select **Create**.</span></span>

<span data-ttu-id="19375-141">Aggiungere il `Microsoft.TypeScript.MSBuild` pacchetto al progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-141">Add the `Microsoft.TypeScript.MSBuild` package to the project:</span></span>

1. <span data-ttu-id="19375-142">In **Esplora soluzioni** (riquadro a destra), fare clic con il pulsante destro del mouse sul nodo del progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="19375-142">In **Solution Explorer** (right pane), right-click the project node and select **Manage NuGet Packages**.</span></span> <span data-ttu-id="19375-143">Nella scheda **Sfoglia** cercare `Microsoft.TypeScript.MSBuild` , quindi fare clic su **Installa** a destra per installare il pacchetto.</span><span class="sxs-lookup"><span data-stu-id="19375-143">In the **Browse** tab, search for `Microsoft.TypeScript.MSBuild`, and then click **Install** on the right to install the package.</span></span>

<span data-ttu-id="19375-144">Visual Studio aggiunge il pacchetto NuGet nel nodo **dipendenze** in **Esplora soluzioni**, abilitando la compilazione typescript nel progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-144">Visual Studio adds the NuGet package under the **Dependencies** node in **Solution Explorer**, enabling TypeScript compilation in the project.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="19375-145">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-145">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="19375-146">Eseguire il comando seguente nel **Terminale integrato**:</span><span class="sxs-lookup"><span data-stu-id="19375-146">Run the following command in the **Integrated Terminal**:</span></span>

```dotnetcli
dotnet new web -o SignalRWebPack
code -r SignalRWebPack
```

* <span data-ttu-id="19375-147">Il `dotnet new` comando crea un'app web ASP.NET Core vuota in una directory di *SignalR Webpack* .</span><span class="sxs-lookup"><span data-stu-id="19375-147">The `dotnet new` command creates an empty ASP.NET Core web app in a *SignalRWebPack* directory.</span></span>
* <span data-ttu-id="19375-148">Il `code` comando apre la cartella *SignalR Webpack* nell'istanza corrente di Visual Studio Code.</span><span class="sxs-lookup"><span data-stu-id="19375-148">The `code` command opens the *SignalRWebPack* folder in the current instance of Visual Studio Code.</span></span>

<span data-ttu-id="19375-149">Eseguire il comando interfaccia della riga di comando di .NET Core seguente nel **terminale integrato**:</span><span class="sxs-lookup"><span data-stu-id="19375-149">Run the following .NET Core CLI command in the **Integrated Terminal**:</span></span>

```dotnetcli
dotnet add package Microsoft.TypeScript.MSBuild
```

<span data-ttu-id="19375-150">Il comando precedente aggiunge il pacchetto [Microsoft. typescript. MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/) , abilitando la compilazione typescript nel progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-150">The preceding command adds the [Microsoft.TypeScript.MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/) package, enabling TypeScript compilation in the project.</span></span>

---

## <a name="configure-webpack-and-typescript"></a><span data-ttu-id="19375-151">Configurare Webpack e TypeScript</span><span class="sxs-lookup"><span data-stu-id="19375-151">Configure Webpack and TypeScript</span></span>

<span data-ttu-id="19375-152">I passaggi seguenti consentono di configurare la conversione da TypeScript a JavaScript e di creare un bundle delle risorse sul lato client.</span><span class="sxs-lookup"><span data-stu-id="19375-152">The following steps configure the conversion of TypeScript to JavaScript and the bundling of client-side resources.</span></span>

1. <span data-ttu-id="19375-153">Eseguire il comando seguente nella radice del progetto per creare un *package.jsnel* file:</span><span class="sxs-lookup"><span data-stu-id="19375-153">Run the following command in the project root to create a *package.json* file:</span></span>

    ```console
    npm init -y
    ```

1. <span data-ttu-id="19375-154">Aggiungere la proprietà evidenziata al *package.jsnel* file e salvare le modifiche apportate al file:</span><span class="sxs-lookup"><span data-stu-id="19375-154">Add the highlighted property to the *package.json* file and save the file changes:</span></span>

    [!code-json[package.json](signalr-typescript-webpack/sample/3.x/snippets/package1.json?highlight=4)]

    <span data-ttu-id="19375-155">Impostando la proprietà `private` su `true` si evita la visualizzazione di avvisi di installazione del pacchetto nel passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="19375-155">Setting the `private` property to `true` prevents package installation warnings in the next step.</span></span>

1. <span data-ttu-id="19375-156">Installare i pacchetti npm necessari.</span><span class="sxs-lookup"><span data-stu-id="19375-156">Install the required npm packages.</span></span> <span data-ttu-id="19375-157">Eseguire il comando seguente dalla radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-157">Run the following command from the project root:</span></span>

    ```console
    npm i -D -E clean-webpack-plugin@3.0.0 css-loader@3.4.2 html-webpack-plugin@3.2.0 mini-css-extract-plugin@0.9.0 ts-loader@6.2.1 typescript@3.7.5 webpack@4.41.5 webpack-cli@3.3.10
    ```

    <span data-ttu-id="19375-158">Ecco alcuni dettagli del comando da tenere in considerazione:</span><span class="sxs-lookup"><span data-stu-id="19375-158">Some command details to note:</span></span>

    * <span data-ttu-id="19375-159">Un numero di versione segue il segno `@` per ogni nome di pacchetto.</span><span class="sxs-lookup"><span data-stu-id="19375-159">A version number follows the `@` sign for each package name.</span></span> <span data-ttu-id="19375-160">npm installa queste versioni del pacchetto specifiche.</span><span class="sxs-lookup"><span data-stu-id="19375-160">npm installs those specific package versions.</span></span>
    * <span data-ttu-id="19375-161">L'opzione `-E` disabilita il comportamento predefinito di npm, in base al quale gli operatori intervallo di [Versionamento Semantico](https://semver.org/) vengono scritti in *package.json*.</span><span class="sxs-lookup"><span data-stu-id="19375-161">The `-E` option disables npm's default behavior of writing [semantic versioning](https://semver.org/) range operators to *package.json*.</span></span> <span data-ttu-id="19375-162">Ad esempio, viene usato `"webpack": "4.41.5"` invece di `"webpack": "^4.41.5"`.</span><span class="sxs-lookup"><span data-stu-id="19375-162">For example, `"webpack": "4.41.5"` is used instead of `"webpack": "^4.41.5"`.</span></span> <span data-ttu-id="19375-163">Questa opzione impedisce che vengano eseguiti aggiornamenti non desiderati a versioni più recenti del pacchetto.</span><span class="sxs-lookup"><span data-stu-id="19375-163">This option prevents unintended upgrades to newer package versions.</span></span>

    <span data-ttu-id="19375-164">Per altri dettagli, vedere [NPM-install](https://docs.npmjs.com/cli/install) docs.</span><span class="sxs-lookup"><span data-stu-id="19375-164">See the [npm-install](https://docs.npmjs.com/cli/install) docs for more detail.</span></span>

1. <span data-ttu-id="19375-165">Sostituire la `scripts` proprietà del *package.jsnel* file con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-165">Replace the `scripts` property of the *package.json* file with the following code:</span></span>

    ```json
    "scripts": {
      "build": "webpack --mode=development --watch",
      "release": "webpack --mode=production",
      "publish": "npm run release && dotnet publish -c Release"
    },
    ```

    <span data-ttu-id="19375-166">Ecco una spiegazione degli script:</span><span class="sxs-lookup"><span data-stu-id="19375-166">Some explanation of the scripts:</span></span>

    * <span data-ttu-id="19375-167">`build`: Aggrega le risorse sul lato client in modalità di sviluppo e controlla le modifiche ai file.</span><span class="sxs-lookup"><span data-stu-id="19375-167">`build`: Bundles the client-side resources in development mode and watches for file changes.</span></span> <span data-ttu-id="19375-168">Questo controllo fa sì che il bundle venga rigenerato a ogni modifica del file di progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-168">The file watcher causes the bundle to regenerate each time a project file changes.</span></span> <span data-ttu-id="19375-169">L'opzione `mode` disabilita le ottimizzazioni di produzione, come l'eliminazione del codice non utilizzato e la minimizzazione.</span><span class="sxs-lookup"><span data-stu-id="19375-169">The `mode` option disables production optimizations, such as tree shaking and minification.</span></span> <span data-ttu-id="19375-170">Usare `build` solo in modalità di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="19375-170">Only use `build` in development.</span></span>
    * <span data-ttu-id="19375-171">`release`: Aggrega le risorse sul lato client in modalità di produzione.</span><span class="sxs-lookup"><span data-stu-id="19375-171">`release`: Bundles the client-side resources in production mode.</span></span>
    * <span data-ttu-id="19375-172">`publish`: esegue lo script `release` per creare il bundle delle risorse sul lato client in modalità di produzione.</span><span class="sxs-lookup"><span data-stu-id="19375-172">`publish`: Runs the `release` script to bundle the client-side resources in production mode.</span></span> <span data-ttu-id="19375-173">Chiama il comando [publish](/dotnet/core/tools/dotnet-publish) dell'interfaccia della riga di comando di .NET Core per pubblicare l'app.</span><span class="sxs-lookup"><span data-stu-id="19375-173">It calls the .NET Core CLI's [publish](/dotnet/core/tools/dotnet-publish) command to publish the app.</span></span>

1. <span data-ttu-id="19375-174">Creare un file denominato *webpack.config.js*, nella radice del progetto, con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-174">Create a file named *webpack.config.js*, in the project root, with the following code:</span></span>

    [!code-javascript[webpack.config.js](signalr-typescript-webpack/sample/3.x/webpack.config.js)]

    <span data-ttu-id="19375-175">Il file precedente configura la compilazione di Webpack.</span><span class="sxs-lookup"><span data-stu-id="19375-175">The preceding file configures the Webpack compilation.</span></span> <span data-ttu-id="19375-176">Ecco alcuni dettagli di configurazione del server da tenere in considerazione:</span><span class="sxs-lookup"><span data-stu-id="19375-176">Some configuration details to note:</span></span>

    * <span data-ttu-id="19375-177">La proprietà `output` esegue l'override del valore predefinito di *dist*.</span><span class="sxs-lookup"><span data-stu-id="19375-177">The `output` property overrides the default value of *dist*.</span></span> <span data-ttu-id="19375-178">Il bundle viene invece creato nella directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="19375-178">The bundle is instead emitted in the *wwwroot* directory.</span></span>
    * <span data-ttu-id="19375-179">La `resolve.extensions` matrice include *. js* per importare il SignalR codice JavaScript del client.</span><span class="sxs-lookup"><span data-stu-id="19375-179">The `resolve.extensions` array includes *.js* to import the SignalR client JavaScript.</span></span>

1. <span data-ttu-id="19375-180">Creare una nuova directory *src* nella radice del progetto per archiviare gli asset lato client del progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-180">Create a new *src* directory in the project root to store the project's client-side assets.</span></span>

1. <span data-ttu-id="19375-181">Creare *src/index.html* con il markup seguente.</span><span class="sxs-lookup"><span data-stu-id="19375-181">Create *src/index.html* with the following markup.</span></span>

    [!code-html[index.html](signalr-typescript-webpack/sample/3.x/src/index.html)]

    <span data-ttu-id="19375-182">Il codice HTML precedente definisce il markup del boilerplate della home page.</span><span class="sxs-lookup"><span data-stu-id="19375-182">The preceding HTML defines the homepage's boilerplate markup.</span></span>

1. <span data-ttu-id="19375-183">Creare una nuova directory *src/css*.</span><span class="sxs-lookup"><span data-stu-id="19375-183">Create a new *src/css* directory.</span></span> <span data-ttu-id="19375-184">La sua funzione è quella di archiviare i file *.css* del progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-184">Its purpose is to store the project's *.css* files.</span></span>

1. <span data-ttu-id="19375-185">Creare *src/CSS/Main. CSS* con il seguente CSS:</span><span class="sxs-lookup"><span data-stu-id="19375-185">Create *src/css/main.css* with the following CSS:</span></span>

    [!code-css[main.css](signalr-typescript-webpack/sample/3.x/src/css/main.css)]

    <span data-ttu-id="19375-186">Il file *main.css* precedente definisce lo stile dell'app.</span><span class="sxs-lookup"><span data-stu-id="19375-186">The preceding *main.css* file styles the app.</span></span>

1. <span data-ttu-id="19375-187">Creare *src/tsconfig.json* con il codice JSON seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-187">Create *src/tsconfig.json* with the following JSON:</span></span>

    [!code-json[tsconfig.json](signalr-typescript-webpack/sample/3.x/src/tsconfig.json)]

    <span data-ttu-id="19375-188">Il codice precedente configura il compilatore TypeScript in modo da produrre codice JavaScript compatibile con [ECMAScript](https://wikipedia.org/wiki/ECMAScript) 5.</span><span class="sxs-lookup"><span data-stu-id="19375-188">The preceding code configures the TypeScript compiler to produce [ECMAScript](https://wikipedia.org/wiki/ECMAScript) 5-compatible JavaScript.</span></span>

1. <span data-ttu-id="19375-189">Creare *src/index. TS* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-189">Create *src/index.ts* with the following code:</span></span>

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/3.x/snippets/index1.ts?name=snippet_IndexTsPhase1File)]

    <span data-ttu-id="19375-190">Il compilatore TypeScript precedente recupera i riferimenti agli elementi DOM e collega due gestori eventi:</span><span class="sxs-lookup"><span data-stu-id="19375-190">The preceding TypeScript retrieves references to DOM elements and attaches two event handlers:</span></span>

    * <span data-ttu-id="19375-191">`keyup`: Questo evento viene generato quando l'utente digita nella `tbMessage` casella di testo.</span><span class="sxs-lookup"><span data-stu-id="19375-191">`keyup`: This event fires when the user types in the `tbMessage`textbox.</span></span> <span data-ttu-id="19375-192">La funzione `send` viene chiamata quando l'utente preme **INVIO**.</span><span class="sxs-lookup"><span data-stu-id="19375-192">The `send` function is called when the user presses the **Enter** key.</span></span>
    * <span data-ttu-id="19375-193">`click`: questo evento viene generato quando l'utente fa clic sul pulsante **Invia**.</span><span class="sxs-lookup"><span data-stu-id="19375-193">`click`: This event fires when the user clicks the **Send** button.</span></span> <span data-ttu-id="19375-194">Viene chiamata la funzione `send`.</span><span class="sxs-lookup"><span data-stu-id="19375-194">The `send` function is called.</span></span>

## <a name="configure-the-app"></a><span data-ttu-id="19375-195">Configurare l'app</span><span class="sxs-lookup"><span data-stu-id="19375-195">Configure the app</span></span>

1. <span data-ttu-id="19375-196">In `Startup.Configure` aggiungere chiamate a [UseDefaultFiles](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) e [UseStaticFiles](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span><span class="sxs-lookup"><span data-stu-id="19375-196">In `Startup.Configure`, add calls to [UseDefaultFiles](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) and [UseStaticFiles](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span></span>

   [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_UseStaticDefaultFiles&highlight=9-10)]

   <span data-ttu-id="19375-197">Il codice precedente consente al server di individuare e gestire il file *index.html* .</span><span class="sxs-lookup"><span data-stu-id="19375-197">The preceding code allows the server to locate and serve the *index.html* file.</span></span>  <span data-ttu-id="19375-198">Il file viene servito se l'utente immette l'URL completo o l'URL radice dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="19375-198">The file is served whether the user enters its full URL or the root URL of the web app.</span></span>

1. <span data-ttu-id="19375-199">Alla fine di `Startup.Configure` , eseguire il mapping di una route */Hub* all' `ChatHub` Hub.</span><span class="sxs-lookup"><span data-stu-id="19375-199">At the end of `Startup.Configure`, map a */hub* route to the `ChatHub` hub.</span></span> <span data-ttu-id="19375-200">Sostituire il codice che visualizza *Hello World!*</span><span class="sxs-lookup"><span data-stu-id="19375-200">Replace the code that displays *Hello World!*</span></span> <span data-ttu-id="19375-201">con la riga seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-201">with the following line:</span></span> 

   [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_UseSignalR&highlight=3)]

1. <span data-ttu-id="19375-202">In `Startup.ConfigureServices` chiamare [Add SignalR ](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr#Microsoft_Extensions_DependencyInjection_SignalRDependencyInjectionExtensions_AddSignalR_Microsoft_Extensions_DependencyInjection_IServiceCollection_).</span><span class="sxs-lookup"><span data-stu-id="19375-202">In `Startup.ConfigureServices`, call [AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr#Microsoft_Extensions_DependencyInjection_SignalRDependencyInjectionExtensions_AddSignalR_Microsoft_Extensions_DependencyInjection_IServiceCollection_).</span></span>

   [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_AddSignalR)]

1. <span data-ttu-id="19375-203">Creare una nuova directory denominata *Hub* nel *SignalR Webpack* radice del progetto/per archiviare l' SignalR Hub.</span><span class="sxs-lookup"><span data-stu-id="19375-203">Create a new directory named *Hubs* in the project root *SignalRWebPack/* to store the SignalR hub.</span></span>

1. <span data-ttu-id="19375-204">Creare l'hub *Hubs/ChatHub.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-204">Create hub *Hubs/ChatHub.cs* with the following code:</span></span>

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/3.x/snippets/ChatHub.cs?name=snippet_ChatHubStubClass)]

1. <span data-ttu-id="19375-205">Aggiungere l' `using` istruzione seguente all'inizio del file *Startup.cs* per risolvere il `ChatHub` riferimento:</span><span class="sxs-lookup"><span data-stu-id="19375-205">Add the following `using` statement at the top of the *Startup.cs* file to resolve the `ChatHub` reference:</span></span>

    [!code-csharp[Startup](signalr-typescript-webpack/sample/3.x/Startup.cs?name=snippet_HubsNamespace)]

## <a name="enable-client-and-server-communication"></a><span data-ttu-id="19375-206">Abilitare la comunicazione tra client e server</span><span class="sxs-lookup"><span data-stu-id="19375-206">Enable client and server communication</span></span>

<span data-ttu-id="19375-207">L'app Visualizza attualmente un modulo di base per inviare i messaggi, ma non è ancora funzionante.</span><span class="sxs-lookup"><span data-stu-id="19375-207">The app currently displays a basic form to send messages, but is not yet functional.</span></span> <span data-ttu-id="19375-208">Il server è in ascolto di una route specifica, ma non esegue alcuna operazione con i messaggi inviati.</span><span class="sxs-lookup"><span data-stu-id="19375-208">The server is listening to a specific route but does nothing with sent messages.</span></span>

1. <span data-ttu-id="19375-209">Eseguire il comando seguente nella radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-209">Run the following command at the project root:</span></span>

    ```console
    npm i @microsoft/signalr @types/node
    ```

    <span data-ttu-id="19375-210">Il comando precedente installa:</span><span class="sxs-lookup"><span data-stu-id="19375-210">The preceding command installs:</span></span>

     * <span data-ttu-id="19375-211">Il [ SignalR client typescript](https://www.npmjs.com/package/@microsoft/signalr), che consente al client di inviare messaggi al server.</span><span class="sxs-lookup"><span data-stu-id="19375-211">The [SignalR TypeScript client](https://www.npmjs.com/package/@microsoft/signalr), which allows the client to send messages to the server.</span></span>
     * <span data-ttu-id="19375-212">Definizioni del tipo di TypeScript per Node.js, che consente il controllo in fase di compilazione dei tipi di Node.js.</span><span class="sxs-lookup"><span data-stu-id="19375-212">The TypeScript type definitions for Node.js, which enables compile-time checking of Node.js types.</span></span>

1. <span data-ttu-id="19375-213">Aggiungere il codice evidenziato al file *src/index.ts*:</span><span class="sxs-lookup"><span data-stu-id="19375-213">Add the highlighted code to the *src/index.ts* file:</span></span>

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/3.x/snippets/index2.ts?name=snippet_IndexTsPhase2File&highlight=2,9-23)]

    <span data-ttu-id="19375-214">Il codice precedente supporta la ricezione di messaggi dal server.</span><span class="sxs-lookup"><span data-stu-id="19375-214">The preceding code supports receiving messages from the server.</span></span> <span data-ttu-id="19375-215">La classe `HubConnectionBuilder` crea un nuovo compilatore per la configurazione della connessione al server.</span><span class="sxs-lookup"><span data-stu-id="19375-215">The `HubConnectionBuilder` class creates a new builder for configuring the server connection.</span></span> <span data-ttu-id="19375-216">La funzione `withUrl` configura l'URL dell'hub.</span><span class="sxs-lookup"><span data-stu-id="19375-216">The `withUrl` function configures the hub URL.</span></span>

    <span data-ttu-id="19375-217">SignalR consente lo scambio di messaggi tra un client e un server.</span><span class="sxs-lookup"><span data-stu-id="19375-217">SignalR enables the exchange of messages between a client and a server.</span></span> <span data-ttu-id="19375-218">Ogni messaggio ha un nome specifico.</span><span class="sxs-lookup"><span data-stu-id="19375-218">Each message has a specific name.</span></span> <span data-ttu-id="19375-219">Ad esempio, i messaggi con il nome `messageReceived` possono eseguire la logica responsabile della visualizzazione del nuovo messaggio nella zona messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-219">For example, messages with the name `messageReceived` can run the logic responsible for displaying the new message in the messages zone.</span></span> <span data-ttu-id="19375-220">L'ascolto di un messaggio specifico può essere eseguito tramite la funzione `on`.</span><span class="sxs-lookup"><span data-stu-id="19375-220">Listening to a specific message can be done via the `on` function.</span></span> <span data-ttu-id="19375-221">È possibile ascoltare un numero qualsiasi di nomi di messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-221">Any number of message names can be listened to.</span></span> <span data-ttu-id="19375-222">Si può anche passare parametri al messaggio, come il nome dell'autore e il contenuto del messaggio ricevuto.</span><span class="sxs-lookup"><span data-stu-id="19375-222">It's also possible to pass parameters to the message, such as the author's name and the content of the message received.</span></span> <span data-ttu-id="19375-223">Quando il client riceve un messaggio, viene creato un nuovo elemento `div` con il nome dell'autore e il contenuto del messaggio nell'attributo `innerHTML`.</span><span class="sxs-lookup"><span data-stu-id="19375-223">Once the client receives a message, a new `div` element is created with the author's name and the message content in its `innerHTML` attribute.</span></span> <span data-ttu-id="19375-224">Viene aggiunto all'elemento `div` principale che visualizza i messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-224">It's added to the main `div` element displaying the messages.</span></span>

1. <span data-ttu-id="19375-225">Ora che il client può ricevere messaggi, configurarlo anche per l'invio.</span><span class="sxs-lookup"><span data-stu-id="19375-225">Now that the client can receive a message, configure it to send messages.</span></span> <span data-ttu-id="19375-226">Aggiungere il codice evidenziato al file *src/index.ts*:</span><span class="sxs-lookup"><span data-stu-id="19375-226">Add the highlighted code to the *src/index.ts* file:</span></span>

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/3.x/src/index.ts?highlight=34-35)]

    <span data-ttu-id="19375-227">Per inviare un messaggio attraverso la connessione WebSockets è necessario chiamare il metodo `send`.</span><span class="sxs-lookup"><span data-stu-id="19375-227">Sending a message through the WebSockets connection requires calling the `send` method.</span></span> <span data-ttu-id="19375-228">Il primo parametro del metodo è il nome del messaggio.</span><span class="sxs-lookup"><span data-stu-id="19375-228">The method's first parameter is the message name.</span></span> <span data-ttu-id="19375-229">I dati del messaggio sono rappresentati dagli altri parametri.</span><span class="sxs-lookup"><span data-stu-id="19375-229">The message data inhabits the other parameters.</span></span> <span data-ttu-id="19375-230">In questo esempio un messaggio identificato come `newMessage` viene inviato al server.</span><span class="sxs-lookup"><span data-stu-id="19375-230">In this example, a message identified as `newMessage` is sent to the server.</span></span> <span data-ttu-id="19375-231">Il messaggio è costituito dal nome utente e dall'input dell'utente in una casella di testo.</span><span class="sxs-lookup"><span data-stu-id="19375-231">The message consists of the username and the user input from a text box.</span></span> <span data-ttu-id="19375-232">Se l'invio riesce, il valore della casella di testo viene cancellato.</span><span class="sxs-lookup"><span data-stu-id="19375-232">If the send works, the text box value is cleared.</span></span>

1. <span data-ttu-id="19375-233">Aggiungere il metodo `NewMessage` alla classe `ChatHub`:</span><span class="sxs-lookup"><span data-stu-id="19375-233">Add the `NewMessage` method to the `ChatHub` class:</span></span>

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/3.x/Hubs/ChatHub.cs?highlight=8-11)]

    <span data-ttu-id="19375-234">Il codice precedente trasmette i messaggi ricevuti a tutti gli utenti connessi dopo che il server li riceve.</span><span class="sxs-lookup"><span data-stu-id="19375-234">The preceding code broadcasts received messages to all connected users once the server receives them.</span></span> <span data-ttu-id="19375-235">Non è necessario avere un metodo generico `on` per ricevere tutti i messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-235">It's unnecessary to have a generic `on` method to receive all the messages.</span></span> <span data-ttu-id="19375-236">È sufficiente un metodo denominato dopo il nome del messaggio.</span><span class="sxs-lookup"><span data-stu-id="19375-236">A method named after the message name suffices.</span></span>

    <span data-ttu-id="19375-237">In questo esempio il client TypeScript invia un messaggio identificato come `newMessage`.</span><span class="sxs-lookup"><span data-stu-id="19375-237">In this example, the TypeScript client sends a message identified as `newMessage`.</span></span> <span data-ttu-id="19375-238">Il metodo C# `NewMessage` si aspetta i dati inviati dal client.</span><span class="sxs-lookup"><span data-stu-id="19375-238">The C# `NewMessage` method expects the data sent by the client.</span></span> <span data-ttu-id="19375-239">Viene eseguita una chiamata a [SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync) nei [client. All](/dotnet/api/microsoft.aspnetcore.signalr.ihubclients-1.all).</span><span class="sxs-lookup"><span data-stu-id="19375-239">A call is made to [SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync) on [Clients.All](/dotnet/api/microsoft.aspnetcore.signalr.ihubclients-1.all).</span></span> <span data-ttu-id="19375-240">I messaggi ricevuti vengono inviati a tutti i client connessi all'hub.</span><span class="sxs-lookup"><span data-stu-id="19375-240">The received messages are sent to all clients connected to the hub.</span></span>

## <a name="test-the-app"></a><span data-ttu-id="19375-241">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="19375-241">Test the app</span></span>

<span data-ttu-id="19375-242">Per verificare che l'app funzioni, eseguire la procedura seguente.</span><span class="sxs-lookup"><span data-stu-id="19375-242">Confirm that the app works with the following steps.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="19375-243">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="19375-243">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="19375-244">Eseguire Webpack in modalità *versione*.</span><span class="sxs-lookup"><span data-stu-id="19375-244">Run Webpack in *release* mode.</span></span> <span data-ttu-id="19375-245">Utilizzando la finestra **console di gestione pacchetti** , eseguire il comando seguente nella radice del progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-245">Using the **Package Manager Console** window, run the following command in the project root.</span></span> <span data-ttu-id="19375-246">Se non ci si trova nella directory radice del progetto, immettere `cd SignalRWebPack` prima di immettere il comando.</span><span class="sxs-lookup"><span data-stu-id="19375-246">If you are not in the project root, enter `cd SignalRWebPack` before entering the command.</span></span>

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. <span data-ttu-id="19375-247">Selezionare **debug**  >  **Avvia senza eseguire debug** per avviare l'app in un browser senza aggiungere il debugger.</span><span class="sxs-lookup"><span data-stu-id="19375-247">Select **Debug** > **Start without debugging** to launch the app in a browser without attaching the debugger.</span></span> <span data-ttu-id="19375-248">Il file *wwwroot/index.html* viene servito all'indirizzo `http://localhost:<port_number>`.</span><span class="sxs-lookup"><span data-stu-id="19375-248">The *wwwroot/index.html* file is served at `http://localhost:<port_number>`.</span></span>

   <span data-ttu-id="19375-249">Se vengono visualizzati errori di compilazione, provare a chiudere e riaprire la soluzione.</span><span class="sxs-lookup"><span data-stu-id="19375-249">If you get compile errors, try closing and reopening the solution.</span></span> 

1. <span data-ttu-id="19375-250">Aprire un'altra istanza del browser (qualsiasi browser).</span><span class="sxs-lookup"><span data-stu-id="19375-250">Open another browser instance (any browser).</span></span> <span data-ttu-id="19375-251">Incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="19375-251">Paste the URL in the address bar.</span></span>

1. <span data-ttu-id="19375-252">Scegliere un browser, digitare qualcosa nella casella di testo **Messaggio** e fare clic sul pulsante **Invia**.</span><span class="sxs-lookup"><span data-stu-id="19375-252">Choose either browser, type something in the **Message** text box, and click the **Send** button.</span></span> <span data-ttu-id="19375-253">Il nome utente e il messaggio univoci vengono visualizzati immediatamente in entrambe le pagine.</span><span class="sxs-lookup"><span data-stu-id="19375-253">The unique user name and message are displayed on both pages instantly.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="19375-254">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-254">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="19375-255">Eseguire Webpack in modalità *versione* eseguendo il comando seguente nella radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-255">Run Webpack in *release* mode by executing the following command in the project root:</span></span>

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. <span data-ttu-id="19375-256">Compilare ed eseguire l'app eseguendo il comando seguente nella radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-256">Build and run the app by executing the following command in the project root:</span></span>

    ```dotnetcli
    dotnet run
    ```

    <span data-ttu-id="19375-257">Il server Web avvia l'app e la rende disponibile su localhost.</span><span class="sxs-lookup"><span data-stu-id="19375-257">The web server starts the app and makes it available on localhost.</span></span>

1. <span data-ttu-id="19375-258">Aprire una finestra in `http://localhost:<port_number>`.</span><span class="sxs-lookup"><span data-stu-id="19375-258">Open a browser to `http://localhost:<port_number>`.</span></span> <span data-ttu-id="19375-259">Il file *wwwroot/index.html* viene servito.</span><span class="sxs-lookup"><span data-stu-id="19375-259">The *wwwroot/index.html* file is served.</span></span> <span data-ttu-id="19375-260">Copiare l'URL dalla barra dell'indirizzo.</span><span class="sxs-lookup"><span data-stu-id="19375-260">Copy the URL from the address bar.</span></span>

1. <span data-ttu-id="19375-261">Aprire un'altra istanza del browser (qualsiasi browser).</span><span class="sxs-lookup"><span data-stu-id="19375-261">Open another browser instance (any browser).</span></span> <span data-ttu-id="19375-262">Incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="19375-262">Paste the URL in the address bar.</span></span>

1. <span data-ttu-id="19375-263">Scegliere un browser, digitare qualcosa nella casella di testo **Messaggio** e fare clic sul pulsante **Invia**.</span><span class="sxs-lookup"><span data-stu-id="19375-263">Choose either browser, type something in the **Message** text box, and click the **Send** button.</span></span> <span data-ttu-id="19375-264">Il nome utente e il messaggio univoci vengono visualizzati immediatamente in entrambe le pagine.</span><span class="sxs-lookup"><span data-stu-id="19375-264">The unique user name and message are displayed on both pages instantly.</span></span>

---

![Messaggio visualizzato in entrambe le finestre del browser](signalr-typescript-webpack/_static/browsers-message-broadcast.png)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="prerequisites"></a><span data-ttu-id="19375-266">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="19375-266">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="19375-267">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="19375-267">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="19375-268">[Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="19375-268">[Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [<span data-ttu-id="19375-269">.NET Core SDK 2,2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="19375-269">.NET Core SDK 2.2 or later</span></span>](https://dotnet.microsoft.com/download/dotnet-core)
* <span data-ttu-id="19375-270">[Node.js](https://nodejs.org/) con [npm](https://www.npmjs.com/)</span><span class="sxs-lookup"><span data-stu-id="19375-270">[Node.js](https://nodejs.org/) with [npm](https://www.npmjs.com/)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="19375-271">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-271">Visual Studio Code</span></span>](#tab/visual-studio-code)

* [<span data-ttu-id="19375-272">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-272">Visual Studio Code</span></span>](https://code.visualstudio.com/download)
* [<span data-ttu-id="19375-273">.NET Core SDK 2,2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="19375-273">.NET Core SDK 2.2 or later</span></span>](https://dotnet.microsoft.com/download/dotnet-core)
* [<span data-ttu-id="19375-274">C# per Visual Studio Code 1.17.1 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="19375-274">C# for Visual Studio Code version 1.17.1 or later</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* <span data-ttu-id="19375-275">[Node.js](https://nodejs.org/) con [npm](https://www.npmjs.com/)</span><span class="sxs-lookup"><span data-stu-id="19375-275">[Node.js](https://nodejs.org/) with [npm](https://www.npmjs.com/)</span></span>

---

## <a name="create-the-aspnet-core-web-app"></a><span data-ttu-id="19375-276">Creare l'app Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="19375-276">Create the ASP.NET Core web app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="19375-277">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="19375-277">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="19375-278">Configurare Visual Studio in modo che cerchi npm nella variabile di ambiente *PATH*.</span><span class="sxs-lookup"><span data-stu-id="19375-278">Configure Visual Studio to look for npm in the *PATH* environment variable.</span></span> <span data-ttu-id="19375-279">Per impostazione predefinita, Visual Studio usa la versione di npm trovata nella directory di installazione.</span><span class="sxs-lookup"><span data-stu-id="19375-279">By default, Visual Studio uses the version of npm found in its installation directory.</span></span> <span data-ttu-id="19375-280">Seguire queste istruzioni in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="19375-280">Follow these instructions in Visual Studio:</span></span>

1. <span data-ttu-id="19375-281">Passare a **strumenti** > **Opzioni** > **progetti e soluzioni** > **Web Gestione pacchetti** > **strumenti Web esterni**.</span><span class="sxs-lookup"><span data-stu-id="19375-281">Navigate to **Tools** > **Options** > **Projects and Solutions** > **Web Package Management** > **External Web Tools**.</span></span>
1. <span data-ttu-id="19375-282">Selezionare la voce *$(PATH)* dall'elenco.</span><span class="sxs-lookup"><span data-stu-id="19375-282">Select the *$(PATH)* entry from the list.</span></span> <span data-ttu-id="19375-283">Fare clic sulla freccia in su per spostare la voce nella seconda posizione nell'elenco.</span><span class="sxs-lookup"><span data-stu-id="19375-283">Click the up arrow to move the entry to the second position in the list.</span></span>

    ![Configurazione di Visual Studio](signalr-typescript-webpack/_static/signalr-configure-path-visual-studio.png)

<span data-ttu-id="19375-285">La configurazione di Visual Studio è completata.</span><span class="sxs-lookup"><span data-stu-id="19375-285">Visual Studio configuration is completed.</span></span> <span data-ttu-id="19375-286">A questo punto è possibile creare il progetto</span><span class="sxs-lookup"><span data-stu-id="19375-286">It's time to create the project.</span></span>

1. <span data-ttu-id="19375-287">Selezionare l'opzione di menu **File** > **Nuovo** > **Progetto** e scegliere il modello **Applicazione Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="19375-287">Use the **File** > **New** > **Project** menu option and choose the **ASP.NET Core Web Application** template.</span></span>
1. <span data-ttu-id="19375-288">Assegnare al progetto il nome *SignalR Webpack* e selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="19375-288">Name the project *SignalRWebPack*, and select **Create**.</span></span>
1. <span data-ttu-id="19375-289">Selezionare *.NET Core* nell'elenco a discesa del framework di destinazione e quindi selezionare *ASP.NET Core 2.2* nell'elenco a discesa del selettore del framework.</span><span class="sxs-lookup"><span data-stu-id="19375-289">Select *.NET Core* from the target framework drop-down, and select *ASP.NET Core 2.2* from the framework selector drop-down.</span></span> <span data-ttu-id="19375-290">Selezionare il modello **vuoto** e selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="19375-290">Select the **Empty** template, and select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="19375-291">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-291">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="19375-292">Eseguire il comando seguente nel **Terminale integrato**:</span><span class="sxs-lookup"><span data-stu-id="19375-292">Run the following command in the **Integrated Terminal**:</span></span>

```dotnetcli
dotnet new web -o SignalRWebPack
```

<span data-ttu-id="19375-293">Un'app Web ASP.NET Core vuota, destinata a .NET Core, viene creata in una directory di *SignalR Webpack* .</span><span class="sxs-lookup"><span data-stu-id="19375-293">An empty ASP.NET Core web app, targeting .NET Core, is created in a *SignalRWebPack* directory.</span></span>

---

## <a name="configure-webpack-and-typescript"></a><span data-ttu-id="19375-294">Configurare Webpack e TypeScript</span><span class="sxs-lookup"><span data-stu-id="19375-294">Configure Webpack and TypeScript</span></span>

<span data-ttu-id="19375-295">I passaggi seguenti consentono di configurare la conversione da TypeScript a JavaScript e di creare un bundle delle risorse sul lato client.</span><span class="sxs-lookup"><span data-stu-id="19375-295">The following steps configure the conversion of TypeScript to JavaScript and the bundling of client-side resources.</span></span>

1. <span data-ttu-id="19375-296">Eseguire il comando seguente nella radice del progetto per creare un *package.jsnel* file:</span><span class="sxs-lookup"><span data-stu-id="19375-296">Run the following command in the project root to create a *package.json* file:</span></span>

    ```console
    npm init -y
    ```

1. <span data-ttu-id="19375-297">Aggiungere la proprietà evidenziata al file *package.json*:</span><span class="sxs-lookup"><span data-stu-id="19375-297">Add the highlighted property to the *package.json* file:</span></span>

    [!code-json[package.json](signalr-typescript-webpack/sample/2.x/snippets/package1.json?highlight=4)]

    <span data-ttu-id="19375-298">Impostando la proprietà `private` su `true` si evita la visualizzazione di avvisi di installazione del pacchetto nel passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="19375-298">Setting the `private` property to `true` prevents package installation warnings in the next step.</span></span>

1. <span data-ttu-id="19375-299">Installare i pacchetti npm necessari.</span><span class="sxs-lookup"><span data-stu-id="19375-299">Install the required npm packages.</span></span> <span data-ttu-id="19375-300">Eseguire il comando seguente dalla radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-300">Run the following command from the project root:</span></span>

    ```console
    npm install -D -E clean-webpack-plugin@1.0.1 css-loader@2.1.0 html-webpack-plugin@4.0.0-beta.5 mini-css-extract-plugin@0.5.0 ts-loader@5.3.3 typescript@3.3.3 webpack@4.29.3 webpack-cli@3.2.3
    ```

    <span data-ttu-id="19375-301">Ecco alcuni dettagli del comando da tenere in considerazione:</span><span class="sxs-lookup"><span data-stu-id="19375-301">Some command details to note:</span></span>

    * <span data-ttu-id="19375-302">Un numero di versione segue il segno `@` per ogni nome di pacchetto.</span><span class="sxs-lookup"><span data-stu-id="19375-302">A version number follows the `@` sign for each package name.</span></span> <span data-ttu-id="19375-303">npm installa queste versioni del pacchetto specifiche.</span><span class="sxs-lookup"><span data-stu-id="19375-303">npm installs those specific package versions.</span></span>
    * <span data-ttu-id="19375-304">L'opzione `-E` disabilita il comportamento predefinito di npm, in base al quale gli operatori intervallo di [Versionamento Semantico](https://semver.org/) vengono scritti in *package.json*.</span><span class="sxs-lookup"><span data-stu-id="19375-304">The `-E` option disables npm's default behavior of writing [semantic versioning](https://semver.org/) range operators to *package.json*.</span></span> <span data-ttu-id="19375-305">Ad esempio, viene usato `"webpack": "4.29.3"` invece di `"webpack": "^4.29.3"`.</span><span class="sxs-lookup"><span data-stu-id="19375-305">For example, `"webpack": "4.29.3"` is used instead of `"webpack": "^4.29.3"`.</span></span> <span data-ttu-id="19375-306">Questa opzione impedisce che vengano eseguiti aggiornamenti non desiderati a versioni più recenti del pacchetto.</span><span class="sxs-lookup"><span data-stu-id="19375-306">This option prevents unintended upgrades to newer package versions.</span></span>

    <span data-ttu-id="19375-307">Per altri dettagli, vedere [NPM-install](https://docs.npmjs.com/cli/install) docs.</span><span class="sxs-lookup"><span data-stu-id="19375-307">See the [npm-install](https://docs.npmjs.com/cli/install) docs for more detail.</span></span>

1. <span data-ttu-id="19375-308">Sostituire la `scripts` proprietà del *package.jsnel* file con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-308">Replace the `scripts` property of the *package.json* file with the following code:</span></span>

    ```json
    "scripts": {
      "build": "webpack --mode=development --watch",
      "release": "webpack --mode=production",
      "publish": "npm run release && dotnet publish -c Release"
    },
    ```

    <span data-ttu-id="19375-309">Ecco una spiegazione degli script:</span><span class="sxs-lookup"><span data-stu-id="19375-309">Some explanation of the scripts:</span></span>

    * <span data-ttu-id="19375-310">`build`: Aggrega le risorse sul lato client in modalità di sviluppo e controlla le modifiche ai file.</span><span class="sxs-lookup"><span data-stu-id="19375-310">`build`: Bundles the client-side resources in development mode and watches for file changes.</span></span> <span data-ttu-id="19375-311">Questo controllo fa sì che il bundle venga rigenerato a ogni modifica del file di progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-311">The file watcher causes the bundle to regenerate each time a project file changes.</span></span> <span data-ttu-id="19375-312">L'opzione `mode` disabilita le ottimizzazioni di produzione, come l'eliminazione del codice non utilizzato e la minimizzazione.</span><span class="sxs-lookup"><span data-stu-id="19375-312">The `mode` option disables production optimizations, such as tree shaking and minification.</span></span> <span data-ttu-id="19375-313">Usare `build` solo in modalità di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="19375-313">Only use `build` in development.</span></span>
    * <span data-ttu-id="19375-314">`release`: Aggrega le risorse sul lato client in modalità di produzione.</span><span class="sxs-lookup"><span data-stu-id="19375-314">`release`: Bundles the client-side resources in production mode.</span></span>
    * <span data-ttu-id="19375-315">`publish`: esegue lo script `release` per creare il bundle delle risorse sul lato client in modalità di produzione.</span><span class="sxs-lookup"><span data-stu-id="19375-315">`publish`: Runs the `release` script to bundle the client-side resources in production mode.</span></span> <span data-ttu-id="19375-316">Chiama il comando [publish](/dotnet/core/tools/dotnet-publish) dell'interfaccia della riga di comando di .NET Core per pubblicare l'app.</span><span class="sxs-lookup"><span data-stu-id="19375-316">It calls the .NET Core CLI's [publish](/dotnet/core/tools/dotnet-publish) command to publish the app.</span></span>

1. <span data-ttu-id="19375-317">Creare un file denominato *webpack.config.js* nella radice del progetto, con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-317">Create a file named *webpack.config.js* in the project root, with the following code:</span></span>

    [!code-javascript[webpack.config.js](signalr-typescript-webpack/sample/2.x/webpack.config.js)]

    <span data-ttu-id="19375-318">Il file precedente configura la compilazione di Webpack.</span><span class="sxs-lookup"><span data-stu-id="19375-318">The preceding file configures the Webpack compilation.</span></span> <span data-ttu-id="19375-319">Ecco alcuni dettagli di configurazione del server da tenere in considerazione:</span><span class="sxs-lookup"><span data-stu-id="19375-319">Some configuration details to note:</span></span>

    * <span data-ttu-id="19375-320">La proprietà `output` esegue l'override del valore predefinito di *dist*.</span><span class="sxs-lookup"><span data-stu-id="19375-320">The `output` property overrides the default value of *dist*.</span></span> <span data-ttu-id="19375-321">Il bundle viene invece creato nella directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="19375-321">The bundle is instead emitted in the *wwwroot* directory.</span></span>
    * <span data-ttu-id="19375-322">La `resolve.extensions` matrice include *. js* per importare il SignalR codice JavaScript del client.</span><span class="sxs-lookup"><span data-stu-id="19375-322">The `resolve.extensions` array includes *.js* to import the SignalR client JavaScript.</span></span>

1. <span data-ttu-id="19375-323">Creare una nuova directory *src* nella radice del progetto per archiviare gli asset lato client del progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-323">Create a new *src* directory in the project root to store the project's client-side assets.</span></span>

1. <span data-ttu-id="19375-324">Creare *src/index.html* con il markup seguente.</span><span class="sxs-lookup"><span data-stu-id="19375-324">Create *src/index.html* with the following markup.</span></span>

    [!code-html[index.html](signalr-typescript-webpack/sample/2.x/src/index.html)]

    <span data-ttu-id="19375-325">Il codice HTML precedente definisce il markup del boilerplate della home page.</span><span class="sxs-lookup"><span data-stu-id="19375-325">The preceding HTML defines the homepage's boilerplate markup.</span></span>

1. <span data-ttu-id="19375-326">Creare una nuova directory *src/css*.</span><span class="sxs-lookup"><span data-stu-id="19375-326">Create a new *src/css* directory.</span></span> <span data-ttu-id="19375-327">La sua funzione è quella di archiviare i file *.css* del progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-327">Its purpose is to store the project's *.css* files.</span></span>

1. <span data-ttu-id="19375-328">Creare *src/CSS/Main. CSS* con il markup seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-328">Create *src/css/main.css* with the following markup:</span></span>

    [!code-css[main.css](signalr-typescript-webpack/sample/2.x/src/css/main.css)]

    <span data-ttu-id="19375-329">Il file *main.css* precedente definisce lo stile dell'app.</span><span class="sxs-lookup"><span data-stu-id="19375-329">The preceding *main.css* file styles the app.</span></span>

1. <span data-ttu-id="19375-330">Creare *src/tsconfig.json* con il codice JSON seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-330">Create *src/tsconfig.json* with the following JSON:</span></span>

    [!code-json[tsconfig.json](signalr-typescript-webpack/sample/2.x/src/tsconfig.json)]

    <span data-ttu-id="19375-331">Il codice precedente configura il compilatore TypeScript in modo da produrre codice JavaScript compatibile con [ECMAScript](https://wikipedia.org/wiki/ECMAScript) 5.</span><span class="sxs-lookup"><span data-stu-id="19375-331">The preceding code configures the TypeScript compiler to produce [ECMAScript](https://wikipedia.org/wiki/ECMAScript) 5-compatible JavaScript.</span></span>

1. <span data-ttu-id="19375-332">Creare *src/index. TS* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-332">Create *src/index.ts* with the following code:</span></span>

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/2.x/snippets/index1.ts?name=snippet_IndexTsPhase1File)]

    <span data-ttu-id="19375-333">Il compilatore TypeScript precedente recupera i riferimenti agli elementi DOM e collega due gestori eventi:</span><span class="sxs-lookup"><span data-stu-id="19375-333">The preceding TypeScript retrieves references to DOM elements and attaches two event handlers:</span></span>

    * <span data-ttu-id="19375-334">`keyup`: Questo evento viene generato quando l'utente digita nella `tbMessage` casella di testo.</span><span class="sxs-lookup"><span data-stu-id="19375-334">`keyup`: This event fires when the user types in the `tbMessage` textbox.</span></span> <span data-ttu-id="19375-335">La funzione `send` viene chiamata quando l'utente preme **INVIO**.</span><span class="sxs-lookup"><span data-stu-id="19375-335">The `send` function is called when the user presses the **Enter** key.</span></span>
    * <span data-ttu-id="19375-336">`click`: questo evento viene generato quando l'utente fa clic sul pulsante **Invia**.</span><span class="sxs-lookup"><span data-stu-id="19375-336">`click`: This event fires when the user clicks the **Send** button.</span></span> <span data-ttu-id="19375-337">Viene chiamata la funzione `send`.</span><span class="sxs-lookup"><span data-stu-id="19375-337">The `send` function is called.</span></span>

## <a name="configure-the-aspnet-core-app"></a><span data-ttu-id="19375-338">Configurare l'app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="19375-338">Configure the ASP.NET Core app</span></span>

1. <span data-ttu-id="19375-339">Il codice fornito nel metodo `Startup.Configure` visualizza *Hello World!*.</span><span class="sxs-lookup"><span data-stu-id="19375-339">The code provided in the `Startup.Configure` method displays *Hello World!*.</span></span> <span data-ttu-id="19375-340">Sostituire la chiamata del metodo `app.Run` con chiamate a [UseDefaultFiles](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) e [UseStaticFiles](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span><span class="sxs-lookup"><span data-stu-id="19375-340">Replace the `app.Run` method call with calls to [UseDefaultFiles](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) and [UseStaticFiles](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span></span>

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_UseStaticDefaultFiles)]

    <span data-ttu-id="19375-341">Il codice precedente consente al server di individuare e servire il file *index.html*, sia che l'utente immetta l'URL completo o l'URL radice dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="19375-341">The preceding code allows the server to locate and serve the *index.html* file, whether the user enters its full URL or the root URL of the web app.</span></span>

1. <span data-ttu-id="19375-342">Chiamare [il SignalR componente aggiuntivo](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr#Microsoft_Extensions_DependencyInjection_SignalRDependencyInjectionExtensions_AddSignalR_Microsoft_Extensions_DependencyInjection_IServiceCollection_) `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="19375-342">Call [AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr#Microsoft_Extensions_DependencyInjection_SignalRDependencyInjectionExtensions_AddSignalR_Microsoft_Extensions_DependencyInjection_IServiceCollection_) in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="19375-343">Aggiunge i SignalR servizi al progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-343">It adds the SignalR services to the project.</span></span>

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_AddSignalR)]

1. <span data-ttu-id="19375-344">Eseguire il mapping di una route */hub* all'hub `ChatHub`.</span><span class="sxs-lookup"><span data-stu-id="19375-344">Map a */hub* route to the `ChatHub` hub.</span></span> <span data-ttu-id="19375-345">Aggiungere le righe seguenti alla fine di `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="19375-345">Add the following lines at the end of `Startup.Configure`:</span></span>

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_UseSignalR)]

1. <span data-ttu-id="19375-346">Creare una nuova directory denominata *Hubs* nella radice del progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-346">Create a new directory, called *Hubs*, in the project root.</span></span> <span data-ttu-id="19375-347">Lo scopo è archiviare l' SignalR Hub, che verrà creato nel passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="19375-347">Its purpose is to store the SignalR hub, which is created in the next step.</span></span>

1. <span data-ttu-id="19375-348">Creare l'hub *Hubs/ChatHub.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="19375-348">Create hub *Hubs/ChatHub.cs* with the following code:</span></span>

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/2.x/snippets/ChatHub.cs?name=snippet_ChatHubStubClass)]

1. <span data-ttu-id="19375-349">Aggiungere il codice seguente all'inizio del file *Startup.cs* per risolvere il riferimento a `ChatHub`:</span><span class="sxs-lookup"><span data-stu-id="19375-349">Add the following code at the top of the *Startup.cs* file to resolve the `ChatHub` reference:</span></span>

    [!code-csharp[Startup](signalr-typescript-webpack/sample/2.x/Startup.cs?name=snippet_HubsNamespace)]

## <a name="enable-client-and-server-communication"></a><span data-ttu-id="19375-350">Abilitare la comunicazione tra client e server</span><span class="sxs-lookup"><span data-stu-id="19375-350">Enable client and server communication</span></span>

<span data-ttu-id="19375-351">L'app attualmente visualizza un semplice form per l'invio di messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-351">The app currently displays a simple form to send messages.</span></span> <span data-ttu-id="19375-352">Quando si prova a eseguire questa operazione, non accade nulla.</span><span class="sxs-lookup"><span data-stu-id="19375-352">Nothing happens when you try to do so.</span></span> <span data-ttu-id="19375-353">Il server è in ascolto di una route specifica, ma non esegue alcuna operazione con i messaggi inviati.</span><span class="sxs-lookup"><span data-stu-id="19375-353">The server is listening to a specific route but does nothing with sent messages.</span></span>

1. <span data-ttu-id="19375-354">Eseguire il comando seguente nella radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-354">Run the following command at the project root:</span></span>

    ```console
    npm install @aspnet/signalr
    ```

    <span data-ttu-id="19375-355">Il comando precedente installa il [ SignalR client typescript](https://www.npmjs.com/package/@microsoft/signalr), che consente al client di inviare messaggi al server.</span><span class="sxs-lookup"><span data-stu-id="19375-355">The preceding command installs the [SignalR TypeScript client](https://www.npmjs.com/package/@microsoft/signalr), which allows the client to send messages to the server.</span></span>

1. <span data-ttu-id="19375-356">Aggiungere il codice evidenziato al file *src/index.ts*:</span><span class="sxs-lookup"><span data-stu-id="19375-356">Add the highlighted code to the *src/index.ts* file:</span></span>

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/2.x/snippets/index2.ts?name=snippet_IndexTsPhase2File&highlight=2,9-23)]

    <span data-ttu-id="19375-357">Il codice precedente supporta la ricezione di messaggi dal server.</span><span class="sxs-lookup"><span data-stu-id="19375-357">The preceding code supports receiving messages from the server.</span></span> <span data-ttu-id="19375-358">La classe `HubConnectionBuilder` crea un nuovo compilatore per la configurazione della connessione al server.</span><span class="sxs-lookup"><span data-stu-id="19375-358">The `HubConnectionBuilder` class creates a new builder for configuring the server connection.</span></span> <span data-ttu-id="19375-359">La funzione `withUrl` configura l'URL dell'hub.</span><span class="sxs-lookup"><span data-stu-id="19375-359">The `withUrl` function configures the hub URL.</span></span>

    <span data-ttu-id="19375-360">SignalR consente lo scambio di messaggi tra un client e un server.</span><span class="sxs-lookup"><span data-stu-id="19375-360">SignalR enables the exchange of messages between a client and a server.</span></span> <span data-ttu-id="19375-361">Ogni messaggio ha un nome specifico.</span><span class="sxs-lookup"><span data-stu-id="19375-361">Each message has a specific name.</span></span> <span data-ttu-id="19375-362">Ad esempio, i messaggi con il nome `messageReceived` possono eseguire la logica responsabile della visualizzazione del nuovo messaggio nella zona messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-362">For example, messages with the name `messageReceived` can run the logic responsible for displaying the new message in the messages zone.</span></span> <span data-ttu-id="19375-363">L'ascolto di un messaggio specifico può essere eseguito tramite la funzione `on`.</span><span class="sxs-lookup"><span data-stu-id="19375-363">Listening to a specific message can be done via the `on` function.</span></span> <span data-ttu-id="19375-364">È possibile restare in ascolto di un numero indefinito di nomi di messaggio.</span><span class="sxs-lookup"><span data-stu-id="19375-364">You can listen to any number of message names.</span></span> <span data-ttu-id="19375-365">Si può anche passare parametri al messaggio, come il nome dell'autore e il contenuto del messaggio ricevuto.</span><span class="sxs-lookup"><span data-stu-id="19375-365">It's also possible to pass parameters to the message, such as the author's name and the content of the message received.</span></span> <span data-ttu-id="19375-366">Quando il client riceve un messaggio, viene creato un nuovo elemento `div` con il nome dell'autore e il contenuto del messaggio nell'attributo `innerHTML`.</span><span class="sxs-lookup"><span data-stu-id="19375-366">Once the client receives a message, a new `div` element is created with the author's name and the message content in its `innerHTML` attribute.</span></span> <span data-ttu-id="19375-367">Il nuovo messaggio viene aggiunto all'elemento principale `div` visualizzando i messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-367">The new message is added to the main `div` element displaying the messages.</span></span>

1. <span data-ttu-id="19375-368">Ora che il client può ricevere messaggi, configurarlo anche per l'invio.</span><span class="sxs-lookup"><span data-stu-id="19375-368">Now that the client can receive a message, configure it to send messages.</span></span> <span data-ttu-id="19375-369">Aggiungere il codice evidenziato al file *src/index.ts*:</span><span class="sxs-lookup"><span data-stu-id="19375-369">Add the highlighted code to the *src/index.ts* file:</span></span>

    [!code-typescript[index.ts](signalr-typescript-webpack/sample/2.x/src/index.ts?highlight=34-35)]

    <span data-ttu-id="19375-370">Per inviare un messaggio attraverso la connessione WebSockets è necessario chiamare il metodo `send`.</span><span class="sxs-lookup"><span data-stu-id="19375-370">Sending a message through the WebSockets connection requires calling the `send` method.</span></span> <span data-ttu-id="19375-371">Il primo parametro del metodo è il nome del messaggio.</span><span class="sxs-lookup"><span data-stu-id="19375-371">The method's first parameter is the message name.</span></span> <span data-ttu-id="19375-372">I dati del messaggio sono rappresentati dagli altri parametri.</span><span class="sxs-lookup"><span data-stu-id="19375-372">The message data inhabits the other parameters.</span></span> <span data-ttu-id="19375-373">In questo esempio un messaggio identificato come `newMessage` viene inviato al server.</span><span class="sxs-lookup"><span data-stu-id="19375-373">In this example, a message identified as `newMessage` is sent to the server.</span></span> <span data-ttu-id="19375-374">Il messaggio è costituito dal nome utente e dall'input dell'utente in una casella di testo.</span><span class="sxs-lookup"><span data-stu-id="19375-374">The message consists of the username and the user input from a text box.</span></span> <span data-ttu-id="19375-375">Se l'invio riesce, il valore della casella di testo viene cancellato.</span><span class="sxs-lookup"><span data-stu-id="19375-375">If the send works, the text box value is cleared.</span></span>

1. <span data-ttu-id="19375-376">Aggiungere il metodo `NewMessage` alla classe `ChatHub`:</span><span class="sxs-lookup"><span data-stu-id="19375-376">Add the `NewMessage` method to the `ChatHub` class:</span></span>

    [!code-csharp[ChatHub](signalr-typescript-webpack/sample/2.x/Hubs/ChatHub.cs?highlight=8-11)]

    <span data-ttu-id="19375-377">Il codice precedente trasmette i messaggi ricevuti a tutti gli utenti connessi dopo che il server li riceve.</span><span class="sxs-lookup"><span data-stu-id="19375-377">The preceding code broadcasts received messages to all connected users once the server receives them.</span></span> <span data-ttu-id="19375-378">Non è necessario avere un metodo generico `on` per ricevere tutti i messaggi.</span><span class="sxs-lookup"><span data-stu-id="19375-378">It's unnecessary to have a generic `on` method to receive all the messages.</span></span> <span data-ttu-id="19375-379">È sufficiente un metodo denominato dopo il nome del messaggio.</span><span class="sxs-lookup"><span data-stu-id="19375-379">A method named after the message name suffices.</span></span>

    <span data-ttu-id="19375-380">In questo esempio il client TypeScript invia un messaggio identificato come `newMessage`.</span><span class="sxs-lookup"><span data-stu-id="19375-380">In this example, the TypeScript client sends a message identified as `newMessage`.</span></span> <span data-ttu-id="19375-381">Il metodo C# `NewMessage` si aspetta i dati inviati dal client.</span><span class="sxs-lookup"><span data-stu-id="19375-381">The C# `NewMessage` method expects the data sent by the client.</span></span> <span data-ttu-id="19375-382">Viene eseguita una chiamata a [SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync) nei [client. All](/dotnet/api/microsoft.aspnetcore.signalr.ihubclients-1.all).</span><span class="sxs-lookup"><span data-stu-id="19375-382">A call is made to [SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync) on [Clients.All](/dotnet/api/microsoft.aspnetcore.signalr.ihubclients-1.all).</span></span> <span data-ttu-id="19375-383">I messaggi ricevuti vengono inviati a tutti i client connessi all'hub.</span><span class="sxs-lookup"><span data-stu-id="19375-383">The received messages are sent to all clients connected to the hub.</span></span>

## <a name="test-the-app"></a><span data-ttu-id="19375-384">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="19375-384">Test the app</span></span>

<span data-ttu-id="19375-385">Per verificare che l'app funzioni, eseguire la procedura seguente.</span><span class="sxs-lookup"><span data-stu-id="19375-385">Confirm that the app works with the following steps.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="19375-386">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="19375-386">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="19375-387">Eseguire Webpack in modalità *versione*.</span><span class="sxs-lookup"><span data-stu-id="19375-387">Run Webpack in *release* mode.</span></span> <span data-ttu-id="19375-388">Utilizzando la finestra **console di gestione pacchetti** , eseguire il comando seguente nella radice del progetto.</span><span class="sxs-lookup"><span data-stu-id="19375-388">Using the **Package Manager Console** window, run the following command in the project root.</span></span> <span data-ttu-id="19375-389">Se non ci si trova nella directory radice del progetto, immettere `cd SignalRWebPack` prima di immettere il comando.</span><span class="sxs-lookup"><span data-stu-id="19375-389">If you are not in the project root, enter `cd SignalRWebPack` before entering the command.</span></span>

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. <span data-ttu-id="19375-390">Selezionare **debug**  >  **Avvia senza eseguire debug** per avviare l'app in un browser senza aggiungere il debugger.</span><span class="sxs-lookup"><span data-stu-id="19375-390">Select **Debug** > **Start without debugging** to launch the app in a browser without attaching the debugger.</span></span> <span data-ttu-id="19375-391">Il file *wwwroot/index.html* viene servito all'indirizzo `http://localhost:<port_number>`.</span><span class="sxs-lookup"><span data-stu-id="19375-391">The *wwwroot/index.html* file is served at `http://localhost:<port_number>`.</span></span>

1. <span data-ttu-id="19375-392">Aprire un'altra istanza del browser (qualsiasi browser).</span><span class="sxs-lookup"><span data-stu-id="19375-392">Open another browser instance (any browser).</span></span> <span data-ttu-id="19375-393">Incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="19375-393">Paste the URL in the address bar.</span></span>

1. <span data-ttu-id="19375-394">Scegliere un browser, digitare qualcosa nella casella di testo **Messaggio** e fare clic sul pulsante **Invia**.</span><span class="sxs-lookup"><span data-stu-id="19375-394">Choose either browser, type something in the **Message** text box, and click the **Send** button.</span></span> <span data-ttu-id="19375-395">Il nome utente e il messaggio univoci vengono visualizzati immediatamente in entrambe le pagine.</span><span class="sxs-lookup"><span data-stu-id="19375-395">The unique user name and message are displayed on both pages instantly.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="19375-396">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="19375-396">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="19375-397">Eseguire Webpack in modalità *versione* eseguendo il comando seguente nella radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-397">Run Webpack in *release* mode by executing the following command in the project root:</span></span>

    [!INCLUDE [npm-run-release](../includes/signalr-typescript-webpack/npm-run-release.md)]

1. <span data-ttu-id="19375-398">Compilare ed eseguire l'app eseguendo il comando seguente nella radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="19375-398">Build and run the app by executing the following command in the project root:</span></span>

    ```dotnetcli
    dotnet run
    ```

    <span data-ttu-id="19375-399">Il server Web avvia l'app e la rende disponibile su localhost.</span><span class="sxs-lookup"><span data-stu-id="19375-399">The web server starts the app and makes it available on localhost.</span></span>

1. <span data-ttu-id="19375-400">Aprire una finestra in `http://localhost:<port_number>`.</span><span class="sxs-lookup"><span data-stu-id="19375-400">Open a browser to `http://localhost:<port_number>`.</span></span> <span data-ttu-id="19375-401">Il file *wwwroot/index.html* viene servito.</span><span class="sxs-lookup"><span data-stu-id="19375-401">The *wwwroot/index.html* file is served.</span></span> <span data-ttu-id="19375-402">Copiare l'URL dalla barra dell'indirizzo.</span><span class="sxs-lookup"><span data-stu-id="19375-402">Copy the URL from the address bar.</span></span>

1. <span data-ttu-id="19375-403">Aprire un'altra istanza del browser (qualsiasi browser).</span><span class="sxs-lookup"><span data-stu-id="19375-403">Open another browser instance (any browser).</span></span> <span data-ttu-id="19375-404">Incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="19375-404">Paste the URL in the address bar.</span></span>

1. <span data-ttu-id="19375-405">Scegliere un browser, digitare qualcosa nella casella di testo **Messaggio** e fare clic sul pulsante **Invia**.</span><span class="sxs-lookup"><span data-stu-id="19375-405">Choose either browser, type something in the **Message** text box, and click the **Send** button.</span></span> <span data-ttu-id="19375-406">Il nome utente e il messaggio univoci vengono visualizzati immediatamente in entrambe le pagine.</span><span class="sxs-lookup"><span data-stu-id="19375-406">The unique user name and message are displayed on both pages instantly.</span></span>

---

![Messaggio visualizzato in entrambe le finestre del browser](signalr-typescript-webpack/_static/browsers-message-broadcast.png)

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="19375-408">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="19375-408">Additional resources</span></span>

* <xref:signalr/javascript-client>
* <xref:signalr/hubs>
