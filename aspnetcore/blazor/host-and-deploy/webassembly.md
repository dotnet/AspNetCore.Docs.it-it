---
title: Ospitare e distribuire ASP.NET Core Blazor WebAssembly
author: guardrex
description: Informazioni su come ospitare e distribuire un' Blazor app usando ASP.NET Core, le reti per la distribuzione di contenuti (CDN), i file server e le pagine github.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/12/2021
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
uid: blazor/host-and-deploy/webassembly
ms.openlocfilehash: b849cbfe803ef51cbb845566b4d5d27b2761e00c
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100106869"
---
# <a name="host-and-deploy-aspnet-core-blazor-webassembly"></a><span data-ttu-id="275b6-103">Ospitare e distribuire ASP.NET Core Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="275b6-103">Host and deploy ASP.NET Core Blazor WebAssembly</span></span>

<span data-ttu-id="275b6-104">Di [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com), [Daniel Roth](https://github.com/danroth27), [ben Adams](https://twitter.com/ben_a_adams)e [Safia Amodio](https://safia.rocks)</span><span class="sxs-lookup"><span data-stu-id="275b6-104">By [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com), [Daniel Roth](https://github.com/danroth27), [Ben Adams](https://twitter.com/ben_a_adams), and [Safia Abdalla](https://safia.rocks)</span></span>

<span data-ttu-id="275b6-105">Con il [ Blazor WebAssembly modello di hosting](xref:blazor/hosting-models#blazor-webassembly):</span><span class="sxs-lookup"><span data-stu-id="275b6-105">With the [Blazor WebAssembly hosting model](xref:blazor/hosting-models#blazor-webassembly):</span></span>

* <span data-ttu-id="275b6-106">L' Blazor app, le relative dipendenze e il Runtime .NET vengono scaricati nel browser in parallelo.</span><span class="sxs-lookup"><span data-stu-id="275b6-106">The Blazor app, its dependencies, and the .NET runtime are downloaded to the browser in parallel.</span></span>
* <span data-ttu-id="275b6-107">L'app viene eseguita direttamente nel thread dell'interfaccia utente del browser.</span><span class="sxs-lookup"><span data-stu-id="275b6-107">The app is executed directly on the browser UI thread.</span></span>

<span data-ttu-id="275b6-108">Sono supportate le strategie di distribuzione seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-108">The following deployment strategies are supported:</span></span>

* <span data-ttu-id="275b6-109">L' Blazor app viene gestita da un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="275b6-109">The Blazor app is served by an ASP.NET Core app.</span></span> <span data-ttu-id="275b6-110">Questa strategia viene trattata nella sezione [Distribuzione ospitata con ASP.NET Core](#hosted-deployment-with-aspnet-core).</span><span class="sxs-lookup"><span data-stu-id="275b6-110">This strategy is covered in the [Hosted deployment with ASP.NET Core](#hosted-deployment-with-aspnet-core) section.</span></span>
* <span data-ttu-id="275b6-111">L' Blazor app si trova in un server Web o in un servizio di hosting statico, in cui .NET non viene usato per gestire l' Blazor app.</span><span class="sxs-lookup"><span data-stu-id="275b6-111">The Blazor app is placed on a static hosting web server or service, where .NET isn't used to serve the Blazor app.</span></span> <span data-ttu-id="275b6-112">Questa strategia è illustrata nella sezione relativa alla [distribuzione autonoma](#standalone-deployment) , che include informazioni sull'hosting di un' Blazor WebAssembly app come app secondaria IIS.</span><span class="sxs-lookup"><span data-stu-id="275b6-112">This strategy is covered in the [Standalone deployment](#standalone-deployment) section, which includes information on hosting a Blazor WebAssembly app as an IIS sub-app.</span></span>

## <a name="compression"></a><span data-ttu-id="275b6-113">Compressione</span><span class="sxs-lookup"><span data-stu-id="275b6-113">Compression</span></span>

<span data-ttu-id="275b6-114">Quando Blazor WebAssembly viene pubblicata un'app, l'output viene compresso in modo statico durante la pubblicazione per ridurre le dimensioni dell'app e rimuovere l'overhead per la compressione in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-114">When a Blazor WebAssembly app is published, the output is statically compressed during publish to reduce the app's size and remove the overhead for runtime compression.</span></span> <span data-ttu-id="275b6-115">Vengono utilizzati gli algoritmi di compressione seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-115">The following compression algorithms are used:</span></span>

* <span data-ttu-id="275b6-116">[Brotli](https://tools.ietf.org/html/rfc7932) (livello massimo)</span><span class="sxs-lookup"><span data-stu-id="275b6-116">[Brotli](https://tools.ietf.org/html/rfc7932) (highest level)</span></span>
* [<span data-ttu-id="275b6-117">Gzip</span><span class="sxs-lookup"><span data-stu-id="275b6-117">Gzip</span></span>](https://tools.ietf.org/html/rfc1952)

<span data-ttu-id="275b6-118">Blazor si basa sull'host per gestire i file compressi appropriati.</span><span class="sxs-lookup"><span data-stu-id="275b6-118">Blazor relies on the host to the serve the appropriate compressed files.</span></span> <span data-ttu-id="275b6-119">Quando si usa un ASP.NET Core progetto ospitato, il progetto host è in grado di eseguire la negoziazione del contenuto e di servire i file compressi in modo statico.</span><span class="sxs-lookup"><span data-stu-id="275b6-119">When using an ASP.NET Core hosted project, the host project is capable of performing content negotiation and serving the statically-compressed files.</span></span> <span data-ttu-id="275b6-120">Quando si ospita un' Blazor WebAssembly app autonoma, potrebbe essere necessario lavoro aggiuntivo per garantire che vengano serviti file compressi in modo statico:</span><span class="sxs-lookup"><span data-stu-id="275b6-120">When hosting a Blazor WebAssembly standalone app, additional work might be required to ensure that statically-compressed files are served:</span></span>

* <span data-ttu-id="275b6-121">Per `web.config` la configurazione della compressione IIS, vedere la sezione [IIS: Brotli e la compressione gzip](#brotli-and-gzip-compression) .</span><span class="sxs-lookup"><span data-stu-id="275b6-121">For IIS `web.config` compression configuration, see the [IIS: Brotli and Gzip compression](#brotli-and-gzip-compression) section.</span></span> 
* <span data-ttu-id="275b6-122">Quando si ospitano soluzioni di hosting statiche che non supportano la negoziazione del contenuto di file compressi in modo statico, ad esempio pagine di GitHub, è consigliabile configurare l'app per recuperare e decodificare i file compressi Brotli:</span><span class="sxs-lookup"><span data-stu-id="275b6-122">When hosting on static hosting solutions that don't support statically-compressed file content negotiation, such as GitHub Pages, consider configuring the app to fetch and decode Brotli compressed files:</span></span>

  * <span data-ttu-id="275b6-123">Ottenere il decodificatore JavaScript Brotli dal [repository GitHub Google/Brotli](https://github.com/google/brotli).</span><span class="sxs-lookup"><span data-stu-id="275b6-123">Obtain the JavaScript Brotli decoder from the [google/brotli GitHub repository](https://github.com/google/brotli).</span></span> <span data-ttu-id="275b6-124">Il file del decodificatore è denominato `decode.js` e viene trovato nella [ `js` cartella](https://github.com/google/brotli/tree/master/js)del repository.</span><span class="sxs-lookup"><span data-stu-id="275b6-124">The decoder file is named `decode.js` and found in the repository's [`js` folder](https://github.com/google/brotli/tree/master/js).</span></span>
  
    > [!NOTE]
    > <span data-ttu-id="275b6-125">Una regressione è presente nella versione minimizzati dello `decode.js` script ( `decode.min.js` ) nel [repository GitHub Google/brotli](https://github.com/google/brotli).</span><span class="sxs-lookup"><span data-stu-id="275b6-125">A regression is present in the minified version of the `decode.js` script (`decode.min.js`) in the [google/brotli GitHub repository](https://github.com/google/brotli).</span></span> <span data-ttu-id="275b6-126">Minimizzare lo script autonomamente o usare il [pacchetto NPM](https://www.npmjs.com/package/brotli) fino a quando non viene risolto il problema [TypeError in decode.min.js (google/brotli #881)](https://github.com/google/brotli/issues/881) .</span><span class="sxs-lookup"><span data-stu-id="275b6-126">Either minify the script on your own or use the [npm package](https://www.npmjs.com/package/brotli) until the issue [TypeError in decode.min.js (google/brotli #881)](https://github.com/google/brotli/issues/881) is resolved.</span></span> <span data-ttu-id="275b6-127">Il codice di esempio in questa sezione usa la versione **unminified** dello script.</span><span class="sxs-lookup"><span data-stu-id="275b6-127">The example code in this section uses the **unminified** version of the script.</span></span>

  * <span data-ttu-id="275b6-128">Aggiornare l'app per l'uso del decodificatore.</span><span class="sxs-lookup"><span data-stu-id="275b6-128">Update the app to use the decoder.</span></span> <span data-ttu-id="275b6-129">Modificare il markup all'interno del `<body>` tag di chiusura in `wwwroot/index.html` come segue:</span><span class="sxs-lookup"><span data-stu-id="275b6-129">Change the markup inside the closing `<body>` tag in `wwwroot/index.html` to the following:</span></span>
  
    ```html
    <script src="decode.js"></script>
    <script src="_framework/blazor.webassembly.js" autostart="false"></script>
    <script>
      Blazor.start({
        loadBootResource: function (type, name, defaultUri, integrity) {
          if (type !== 'dotnetjs' && location.hostname !== 'localhost') {
            return (async function () {
              const response = await fetch(defaultUri + '.br', { cache: 'no-cache' });
              if (!response.ok) {
                throw new Error(response.statusText);
              }
              const originalResponseBuffer = await response.arrayBuffer();
              const originalResponseArray = new Int8Array(originalResponseBuffer);
              const decompressedResponseArray = BrotliDecode(originalResponseArray);
              const contentType = type === 
                'dotnetwasm' ? 'application/wasm' : 'application/octet-stream';
              return new Response(decompressedResponseArray, 
                { headers: { 'content-type': contentType } });
            })();
          }
        }
      });
    </script>
    ```
 
<span data-ttu-id="275b6-130">Per disabilitare la compressione, aggiungere la `BlazorEnableCompression` Proprietà MSBuild al file di progetto dell'app e impostare il valore su `false` :</span><span class="sxs-lookup"><span data-stu-id="275b6-130">To disable compression, add the `BlazorEnableCompression` MSBuild property to the app's project file and set the value to `false`:</span></span>

```xml
<PropertyGroup>
  <BlazorEnableCompression>false</BlazorEnableCompression>
</PropertyGroup>
```

<span data-ttu-id="275b6-131">La `BlazorEnableCompression` proprietà può essere passata al [`dotnet publish`](/dotnet/core/tools/dotnet-publish) comando con la sintassi seguente in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="275b6-131">The `BlazorEnableCompression` property can be passed to the [`dotnet publish`](/dotnet/core/tools/dotnet-publish) command with the following syntax in a command shell:</span></span>

```dotnetcli
dotnet publish -p:BlazorEnableCompression=false
```

## <a name="rewrite-urls-for-correct-routing"></a><span data-ttu-id="275b6-132">Riscrivere gli URL per il routing corretto</span><span class="sxs-lookup"><span data-stu-id="275b6-132">Rewrite URLs for correct routing</span></span>

<span data-ttu-id="275b6-133">Il routing delle richieste per i componenti della pagina in un' Blazor WebAssembly app non è semplice come il routing delle richieste in un' Blazor Server app ospitata.</span><span class="sxs-lookup"><span data-stu-id="275b6-133">Routing requests for page components in a Blazor WebAssembly app isn't as straightforward as routing requests in a Blazor Server, hosted app.</span></span> <span data-ttu-id="275b6-134">Si consideri un' Blazor WebAssembly app con due componenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-134">Consider a Blazor WebAssembly app with two components:</span></span>

* <span data-ttu-id="275b6-135">`Main.razor`: Carica alla radice dell'app e contiene un collegamento al `About` componente ( `href="About"` ).</span><span class="sxs-lookup"><span data-stu-id="275b6-135">`Main.razor`: Loads at the root of the app and contains a link to the `About` component (`href="About"`).</span></span>
* <span data-ttu-id="275b6-136">`About.razor`: `About` componente.</span><span class="sxs-lookup"><span data-stu-id="275b6-136">`About.razor`: `About` component.</span></span>

<span data-ttu-id="275b6-137">Quando viene richiesto il documento predefinito dell'app usando la barra degli indirizzi del browser (ad esempio, `https://www.contoso.com/`):</span><span class="sxs-lookup"><span data-stu-id="275b6-137">When the app's default document is requested using the browser's address bar (for example, `https://www.contoso.com/`):</span></span>

1. <span data-ttu-id="275b6-138">Il browser invia una richiesta.</span><span class="sxs-lookup"><span data-stu-id="275b6-138">The browser makes a request.</span></span>
1. <span data-ttu-id="275b6-139">Viene restituita la pagina predefinita, in genere `index.html` .</span><span class="sxs-lookup"><span data-stu-id="275b6-139">The default page is returned, which is usually `index.html`.</span></span>
1. <span data-ttu-id="275b6-140">`index.html` avvia l'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-140">`index.html` bootstraps the app.</span></span>
1. <span data-ttu-id="275b6-141">Blazoril router viene caricato e il componente viene sottoposto a Razor `Main` rendering.</span><span class="sxs-lookup"><span data-stu-id="275b6-141">Blazor's router loads, and the Razor `Main` component is rendered.</span></span>

<span data-ttu-id="275b6-142">Nella pagina principale, selezionando il collegamento al `About` componente funziona sul client perché il Blazor router interrompe il browser per effettuare una richiesta su Internet per `www.contoso.com` e serve il componente di cui è stato `About` eseguito il rendering `About` .</span><span class="sxs-lookup"><span data-stu-id="275b6-142">In the Main page, selecting the link to the `About` component works on the client because the Blazor router stops the browser from making a request on the Internet to `www.contoso.com` for `About` and serves the rendered `About` component itself.</span></span> <span data-ttu-id="275b6-143">Tutte le richieste per gli endpoint interni *all'interno dell' Blazor WebAssembly app* funzionano allo stesso modo: le richieste non attivano richieste basate su browser a risorse ospitate su server su Internet.</span><span class="sxs-lookup"><span data-stu-id="275b6-143">All of the requests for internal endpoints *within the Blazor WebAssembly app* work the same way: Requests don't trigger browser-based requests to server-hosted resources on the Internet.</span></span> <span data-ttu-id="275b6-144">Le richieste vengono gestite internamente dal router.</span><span class="sxs-lookup"><span data-stu-id="275b6-144">The router handles the requests internally.</span></span>

<span data-ttu-id="275b6-145">Se una richiesta viene effettuata usando la barra degli indirizzi del browser per `www.contoso.com/About`, la richiesta ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="275b6-145">If a request is made using the browser's address bar for `www.contoso.com/About`, the request fails.</span></span> <span data-ttu-id="275b6-146">La risorsa non esiste nell'host Internet dell'app, quindi viene restituita la risposta *404 non trovato*.</span><span class="sxs-lookup"><span data-stu-id="275b6-146">No such resource exists on the app's Internet host, so a *404 - Not Found* response is returned.</span></span>

<span data-ttu-id="275b6-147">Poiché i browser effettuano richieste a host basati su Internet per le pagine lato client, i server Web e i servizi di hosting devono riscrivere tutte le richieste di risorse non fisicamente presenti sul server nella `index.html` pagina.</span><span class="sxs-lookup"><span data-stu-id="275b6-147">Because browsers make requests to Internet-based hosts for client-side pages, web servers and hosting services must rewrite all requests for resources not physically on the server to the `index.html` page.</span></span> <span data-ttu-id="275b6-148">Quando `index.html` viene restituito, il router dell'app Blazor accetta e risponde con la risorsa corretta.</span><span class="sxs-lookup"><span data-stu-id="275b6-148">When `index.html` is returned, the app's Blazor router takes over and responds with the correct resource.</span></span>

<span data-ttu-id="275b6-149">Quando si esegue la distribuzione in un server IIS, è possibile usare il modulo URL Rewrite con il file pubblicato dell'app `web.config` .</span><span class="sxs-lookup"><span data-stu-id="275b6-149">When deploying to an IIS server, you can use the URL Rewrite Module with the app's published `web.config` file.</span></span> <span data-ttu-id="275b6-150">Per ulteriori informazioni, vedere la sezione [IIS](#iis) .</span><span class="sxs-lookup"><span data-stu-id="275b6-150">For more information, see the [IIS](#iis) section.</span></span>

## <a name="hosted-deployment-with-aspnet-core"></a><span data-ttu-id="275b6-151">Distribuzione ospitata con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="275b6-151">Hosted deployment with ASP.NET Core</span></span>

<span data-ttu-id="275b6-152">Una *distribuzione ospitata* serve all' Blazor WebAssembly app per i browser da un' [app ASP.NET Core](xref:index) eseguita in un server Web.</span><span class="sxs-lookup"><span data-stu-id="275b6-152">A *hosted deployment* serves the Blazor WebAssembly app to browsers from an [ASP.NET Core app](xref:index) that runs on a web server.</span></span>

<span data-ttu-id="275b6-153">L' Blazor WebAssembly app client viene pubblicata nella `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` cartella dell'app Server, insieme a qualsiasi altra risorsa Web statica dell'app Server.</span><span class="sxs-lookup"><span data-stu-id="275b6-153">The client Blazor WebAssembly app is published into the `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` folder of the server app, along with any other static web assets of the server app.</span></span> <span data-ttu-id="275b6-154">Le due app vengono distribuite insieme.</span><span class="sxs-lookup"><span data-stu-id="275b6-154">The two apps are deployed together.</span></span> <span data-ttu-id="275b6-155">È necessario un server Web in grado di ospitare un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="275b6-155">A web server that is capable of hosting an ASP.NET Core app is required.</span></span> <span data-ttu-id="275b6-156">Per una distribuzione ospitata, Visual Studio include il modello di progetto **Blazor WebAssembly app** ( `blazorwasm` modello quando si usa il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando) con l' **`Hosted`** opzione selezionata ( `-ho|--hosted` quando si usa il `dotnet new` comando).</span><span class="sxs-lookup"><span data-stu-id="275b6-156">For a hosted deployment, Visual Studio includes the **Blazor WebAssembly App** project template (`blazorwasm` template when using the [`dotnet new`](/dotnet/core/tools/dotnet-new) command) with the **`Hosted`** option selected (`-ho|--hosted` when using the `dotnet new` command).</span></span>

<span data-ttu-id="275b6-157">Per altre informazioni su hosting e distribuzione di app ASP.NET Core, vedere <xref:host-and-deploy/index>.</span><span class="sxs-lookup"><span data-stu-id="275b6-157">For more information on ASP.NET Core app hosting and deployment, see <xref:host-and-deploy/index>.</span></span>

<span data-ttu-id="275b6-158">Per informazioni sulla distribuzione in Servizio app di Azure, vedere <xref:tutorials/publish-to-azure-webapp-using-vs>.</span><span class="sxs-lookup"><span data-stu-id="275b6-158">For information on deploying to Azure App Service, see <xref:tutorials/publish-to-azure-webapp-using-vs>.</span></span>

## <a name="hosted-deployment-with-multiple-blazor-webassembly-apps"></a><span data-ttu-id="275b6-159">Distribuzione ospitata con più Blazor WebAssembly app</span><span class="sxs-lookup"><span data-stu-id="275b6-159">Hosted deployment with multiple Blazor WebAssembly apps</span></span>

### <a name="app-configuration"></a><span data-ttu-id="275b6-160">Configurazione delle app</span><span class="sxs-lookup"><span data-stu-id="275b6-160">App configuration</span></span>

<span data-ttu-id="275b6-161">Le Blazor soluzioni ospitate possono gestire più Blazor WebAssembly app.</span><span class="sxs-lookup"><span data-stu-id="275b6-161">Hosted Blazor solutions can serve multiple Blazor WebAssembly apps.</span></span>

> [!NOTE]
> <span data-ttu-id="275b6-162">L'esempio in questa sezione fa riferimento all'uso di una *soluzione* di Visual Studio, ma l'uso di Visual Studio e di una soluzione di Visual Studio non è necessario per il funzionamento di più app client in uno scenario di app ospitate Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="275b6-162">The example in this section references the use of a Visual Studio *solution*, but the use of Visual Studio and a Visual Studio solution isn't required for multiple client apps to work in a hosted Blazor WebAssembly apps scenario.</span></span> <span data-ttu-id="275b6-163">Se non si usa Visual Studio, ignorare il `{SOLUTION NAME}.sln` file e tutti gli altri file creati per Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="275b6-163">If you aren't using Visual Studio, ignore the `{SOLUTION NAME}.sln` file and any other files created for Visual Studio.</span></span>

<span data-ttu-id="275b6-164">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="275b6-164">In the following example:</span></span>

* <span data-ttu-id="275b6-165">L'app client iniziale (prima) è il progetto client predefinito di una soluzione creata dal Blazor WebAssembly modello di progetto.</span><span class="sxs-lookup"><span data-stu-id="275b6-165">The initial (first) client app is the default client project of a solution created from the Blazor WebAssembly project template.</span></span> <span data-ttu-id="275b6-166">La prima app client è accessibile in un browser dall'URL `/FirstApp` sulla porta 5001 o con un host di `firstapp.com` .</span><span class="sxs-lookup"><span data-stu-id="275b6-166">The first client app is accessible in a browser from the URL `/FirstApp` on either port 5001 or with a host of `firstapp.com`.</span></span>
* <span data-ttu-id="275b6-167">Alla soluzione viene aggiunta una seconda app client, `SecondBlazorApp.Client` .</span><span class="sxs-lookup"><span data-stu-id="275b6-167">A second client app is added to the solution, `SecondBlazorApp.Client`.</span></span> <span data-ttu-id="275b6-168">La seconda app client è accessibile in un browser dall'URL `/SecondApp` sulla porta 5002 o con un host di `secondapp.com` .</span><span class="sxs-lookup"><span data-stu-id="275b6-168">The second client app is accessible in a browser from the the URL `/SecondApp` on either port 5002 or with a host of `secondapp.com`.</span></span>

<span data-ttu-id="275b6-169">Usa una soluzione ospitata esistente Blazor o crea una nuova soluzione dal Blazor modello di progetto ospitato:</span><span class="sxs-lookup"><span data-stu-id="275b6-169">Use an existing hosted Blazor solution or create a new solution from the Blazor Hosted project template:</span></span>

* <span data-ttu-id="275b6-170">Nel file di progetto dell'app client aggiungere una `<StaticWebAssetBasePath>` proprietà a `<PropertyGroup>` con un valore di `FirstApp` per impostare il percorso di base per gli asset statici del progetto:</span><span class="sxs-lookup"><span data-stu-id="275b6-170">In the client app's project file, add a `<StaticWebAssetBasePath>` property to the `<PropertyGroup>` with a value of `FirstApp` to set the base path for the project's static assets:</span></span>

  ```xml
  <PropertyGroup>
    ...
    <StaticWebAssetBasePath>FirstApp</StaticWebAssetBasePath>
  </PropertyGroup>
  ```

* <span data-ttu-id="275b6-171">Aggiungere una seconda app client alla soluzione:</span><span class="sxs-lookup"><span data-stu-id="275b6-171">Add a second client app to the solution:</span></span>

  * <span data-ttu-id="275b6-172">Aggiungere una cartella denominata `SecondClient` alla cartella della soluzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-172">Add a folder named `SecondClient` to the solution's folder.</span></span> <span data-ttu-id="275b6-173">La cartella della soluzione creata dal modello di progetto contiene le cartelle e i file di soluzione seguenti dopo l' `SecondClient` aggiunta della cartella:</span><span class="sxs-lookup"><span data-stu-id="275b6-173">The solution folder created from the project template contains the following solution file and folders after the `SecondClient` folder is added:</span></span>
  
    * <span data-ttu-id="275b6-174">`Client` cartella</span><span class="sxs-lookup"><span data-stu-id="275b6-174">`Client` (folder)</span></span>
    * <span data-ttu-id="275b6-175">`SecondClient` cartella</span><span class="sxs-lookup"><span data-stu-id="275b6-175">`SecondClient` (folder)</span></span>
    * <span data-ttu-id="275b6-176">`Server` cartella</span><span class="sxs-lookup"><span data-stu-id="275b6-176">`Server` (folder)</span></span>
    * <span data-ttu-id="275b6-177">`Shared` cartella</span><span class="sxs-lookup"><span data-stu-id="275b6-177">`Shared` (folder)</span></span>
    * <span data-ttu-id="275b6-178">`{SOLUTION NAME}.sln` file</span><span class="sxs-lookup"><span data-stu-id="275b6-178">`{SOLUTION NAME}.sln` (file)</span></span>
    
    <span data-ttu-id="275b6-179">Il segnaposto `{SOLUTION NAME}` è il nome della soluzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-179">The placeholder `{SOLUTION NAME}` is the solution's name.</span></span>

  * <span data-ttu-id="275b6-180">Creare un' Blazor WebAssembly App denominata `SecondBlazorApp.Client` nella `SecondClient` cartella dal modello di Blazor WebAssembly progetto.</span><span class="sxs-lookup"><span data-stu-id="275b6-180">Create a Blazor WebAssembly app named `SecondBlazorApp.Client` in the `SecondClient` folder from the Blazor WebAssembly project template.</span></span>

  * <span data-ttu-id="275b6-181">Nel `SecondBlazorApp.Client` file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="275b6-181">In the `SecondBlazorApp.Client` app's project file:</span></span>

    * <span data-ttu-id="275b6-182">Aggiungere una `<StaticWebAssetBasePath>` proprietà a `<PropertyGroup>` con un valore di `SecondApp` :</span><span class="sxs-lookup"><span data-stu-id="275b6-182">Add a `<StaticWebAssetBasePath>` property to the `<PropertyGroup>` with a value of `SecondApp`:</span></span>

      ```xml
      <PropertyGroup>
        ...
        <StaticWebAssetBasePath>SecondApp</StaticWebAssetBasePath>
      </PropertyGroup>
      ```

    * <span data-ttu-id="275b6-183">Aggiungere un riferimento al progetto `Shared` :</span><span class="sxs-lookup"><span data-stu-id="275b6-183">Add a project reference to the `Shared` project:</span></span>

      ```xml
      <ItemGroup>
        <ProjectReference Include="..\Shared\{SOLUTION NAME}.Shared.csproj" />
      </ItemGroup>
      ```

      <span data-ttu-id="275b6-184">Il segnaposto `{SOLUTION NAME}` è il nome della soluzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-184">The placeholder `{SOLUTION NAME}` is the solution's name.</span></span>

* <span data-ttu-id="275b6-185">Nel file di progetto dell'app Server creare un riferimento al progetto per l' `SecondBlazorApp.Client` app client aggiunta:</span><span class="sxs-lookup"><span data-stu-id="275b6-185">In the server app's project file, create a project reference for the added `SecondBlazorApp.Client` client app:</span></span>

  ```xml
  <ItemGroup>
    <ProjectReference Include="..\Client\{SOLUTION NAME}.Client.csproj" />
    <ProjectReference Include="..\SecondClient\SecondBlazorApp.Client.csproj" />
    <ProjectReference Include="..\Shared\{SOLUTION NAME}.Shared.csproj" />
  </ItemGroup>
  ```
  
  <span data-ttu-id="275b6-186">Il segnaposto `{SOLUTION NAME}` è il nome della soluzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-186">The placeholder `{SOLUTION NAME}` is the solution's name.</span></span>

* <span data-ttu-id="275b6-187">Nel file dell'app Server `Properties/launchSettings.json` configurare il `applicationUrl` del profilo gheppio ( `{SOLUTION NAME}.Server` ) per accedere alle app client sulle porte 5001 e 5002:</span><span class="sxs-lookup"><span data-stu-id="275b6-187">In the server app's `Properties/launchSettings.json` file, configure the `applicationUrl` of the Kestrel profile (`{SOLUTION NAME}.Server`) to access the client apps at ports 5001 and 5002:</span></span>

  ```json
  "applicationUrl": "https://localhost:5001;https://localhost:5002",
  ```

* <span data-ttu-id="275b6-188">Nel metodo dell'app Server `Startup.Configure` ( `Startup.cs` ) rimuovere le righe seguenti, che vengono visualizzate dopo la chiamata a <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> :</span><span class="sxs-lookup"><span data-stu-id="275b6-188">In the server app's `Startup.Configure` method (`Startup.cs`), remove the following lines, which appear after the call to <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A>:</span></span>

  ```csharp
  app.UseBlazorFrameworkFiles();
  app.UseStaticFiles();

  app.UseRouting();

  app.UseEndpoints(endpoints =>
  {
      endpoints.MapRazorPages();
      endpoints.MapControllers();
      endpoints.MapFallbackToFile("index.html");
  });
  ```

  <span data-ttu-id="275b6-189">Aggiungere un middleware che esegue il mapping delle richieste alle app client.</span><span class="sxs-lookup"><span data-stu-id="275b6-189">Add middleware that maps requests to the client apps.</span></span> <span data-ttu-id="275b6-190">Nell'esempio seguente viene configurato il middleware per l'esecuzione nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-190">The following example configures the middleware to run when:</span></span>

  * <span data-ttu-id="275b6-191">La porta di richiesta è 5001 per l'app client originale o 5002 per l'app client aggiunta.</span><span class="sxs-lookup"><span data-stu-id="275b6-191">The request port is either 5001 for the original client app or 5002 for the added client app.</span></span>
  * <span data-ttu-id="275b6-192">L'host della richiesta è `firstapp.com` per l'app client originale o `secondapp.com` per l'app client aggiunta.</span><span class="sxs-lookup"><span data-stu-id="275b6-192">The request host is either `firstapp.com` for the original client app or `secondapp.com` for the added client app.</span></span>

    > [!NOTE]
    > <span data-ttu-id="275b6-193">L'esempio illustrato in questa sezione richiede una configurazione aggiuntiva per:</span><span class="sxs-lookup"><span data-stu-id="275b6-193">The example shown in this section requires additional configuration for:</span></span>
    >
    > * <span data-ttu-id="275b6-194">Accesso alle app nei domini host di esempio, `firstapp.com` e `secondapp.com` .</span><span class="sxs-lookup"><span data-stu-id="275b6-194">Accessing the apps at the example host domains, `firstapp.com` and `secondapp.com`.</span></span>
    > * <span data-ttu-id="275b6-195">Certificati per le app client per abilitare la sicurezza TLS (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="275b6-195">Certificates for the client apps to enable TLS security (HTTPS).</span></span>
    >
    > <span data-ttu-id="275b6-196">La configurazione richiesta esula dall'ambito di questo articolo e dipende dalla modalità di hosting della soluzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-196">The required configuration is beyond the scope of this article and depends on how the solution is hosted.</span></span> <span data-ttu-id="275b6-197">Per ulteriori informazioni, vedere gli [articoli host e deploy](xref:host-and-deploy/index).</span><span class="sxs-lookup"><span data-stu-id="275b6-197">For more information see the [Host and deploy articles](xref:host-and-deploy/index).</span></span>

  <span data-ttu-id="275b6-198">Inserire il codice seguente in cui le righe sono state rimosse in precedenza:</span><span class="sxs-lookup"><span data-stu-id="275b6-198">Place the following code where the lines were removed earlier:</span></span>

  ```csharp
  app.MapWhen(ctx => ctx.Request.Host.Port == 5001 || 
      ctx.Request.Host.Equals("firstapp.com"), first =>
  {
      first.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/FirstApp" + ctx.Request.Path;
          return nxt();
      });

      first.UseBlazorFrameworkFiles("/FirstApp");
      first.UseStaticFiles();
      first.UseStaticFiles("/FirstApp");
      first.UseRouting();

      first.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/FirstApp/{*path:nonfile}", 
              "FirstApp/index.html");
      });
  });
  
  app.MapWhen(ctx => ctx.Request.Host.Port == 5002 || 
      ctx.Request.Host.Equals("secondapp.com"), second =>
  {
      second.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/SecondApp" + ctx.Request.Path;
          return nxt();
      });

      second.UseBlazorFrameworkFiles("/SecondApp");
      second.UseStaticFiles();
      second.UseStaticFiles("/SecondApp");
      second.UseRouting();

      second.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/SecondApp/{*path:nonfile}", 
              "SecondApp/index.html");
      });
  });
  ```

* <span data-ttu-id="275b6-199">Nel controller delle previsioni meteorologiche dell'app Server ( `Controllers/WeatherForecastController.cs` ) sostituire la route esistente ( `[Route("[controller]")]` ) `WeatherForecastController` con le route seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-199">In the server app's weather forecast controller (`Controllers/WeatherForecastController.cs`), replace the existing route (`[Route("[controller]")]`) to `WeatherForecastController` with the following routes:</span></span>

  ```csharp
  [Route("FirstApp/[controller]")]
  [Route("SecondApp/[controller]")]
  ```

  <span data-ttu-id="275b6-200">Il middleware aggiunto al metodo dell'app Server `Startup.Configure` precedente modifica le richieste in ingresso in `/WeatherForecast` in `/FirstApp/WeatherForecast` o a `/SecondApp/WeatherForecast` seconda della porta (5001/5002) o del dominio ( `firstapp.com` / `secondapp.com` ).</span><span class="sxs-lookup"><span data-stu-id="275b6-200">The middleware added to the server app's `Startup.Configure` method earlier modifies incoming requests to `/WeatherForecast` to either `/FirstApp/WeatherForecast` or `/SecondApp/WeatherForecast` depending on the port (5001/5002) or domain (`firstapp.com`/`secondapp.com`).</span></span> <span data-ttu-id="275b6-201">Le route del controller precedenti sono necessarie per restituire i dati meteo dall'app Server alle app client.</span><span class="sxs-lookup"><span data-stu-id="275b6-201">The preceding controller routes are required in order to return weather data from the server app to the client apps.</span></span>

### <a name="static-assets-and-class-libraries"></a><span data-ttu-id="275b6-202">Asset statici e librerie di classi</span><span class="sxs-lookup"><span data-stu-id="275b6-202">Static assets and class libraries</span></span>

<span data-ttu-id="275b6-203">Usare gli approcci seguenti per gli asset statici:</span><span class="sxs-lookup"><span data-stu-id="275b6-203">Use the following approaches for static assets:</span></span>

* <span data-ttu-id="275b6-204">Quando l'asset si trova nella cartella dell'app client `wwwroot` , fornire i percorsi in genere:</span><span class="sxs-lookup"><span data-stu-id="275b6-204">When the asset is in the client app's `wwwroot` folder, provide their paths normally:</span></span>

  ```razor
  <img alt="..." src="/{ASSET FILE NAME}" />
  ```

* <span data-ttu-id="275b6-205">Quando l'asset si trova nella `wwwroot` cartella di una [ Razor libreria di classi (RCL)](xref:blazor/components/class-libraries), fare riferimento all'asset statico nell'app client secondo le indicazioni contenute nell' [articolo RCL](xref:razor-pages/ui-class#consume-content-from-a-referenced-rcl):</span><span class="sxs-lookup"><span data-stu-id="275b6-205">When the asset is in the `wwwroot` folder of a [Razor Class Library (RCL)](xref:blazor/components/class-libraries), reference the static asset in the client app per the guidance in the [RCL article](xref:razor-pages/ui-class#consume-content-from-a-referenced-rcl):</span></span>

  ```razor
  <img alt="..." src="_content/{LIBRARY NAME}/{ASSET FILE NAME}" />
  ```

<!-- HOLD for reactivation at 5.x

::: moniker range=">= aspnetcore-5.0"

Components provided to a client app by a class library are referenced normally. If any components require stylesheets or JavaScript files, use either of the following approaches to obtain the static assets:

* The client app's `wwwroot/index.html` file can link (`<link>`) to the static assets.
* The component can use the framework's [`Link` component](xref:blazor/fundamentals/signalr#influence-html-head-tag-elements) to obtain the static assets.

The preceding approaches are demonstrated in the following examples.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

-->

<span data-ttu-id="275b6-206">Per i componenti forniti a un'app client da una libreria di classi viene fatto riferimento normalmente.</span><span class="sxs-lookup"><span data-stu-id="275b6-206">Components provided to a client app by a class library are referenced normally.</span></span> <span data-ttu-id="275b6-207">Se i componenti richiedono fogli di stile o file JavaScript, il file dell'app client `wwwroot/index.html` deve includere i collegamenti statici corretti.</span><span class="sxs-lookup"><span data-stu-id="275b6-207">If any components require stylesheets or JavaScript files, the client app's `wwwroot/index.html` file must include the correct static asset links.</span></span> <span data-ttu-id="275b6-208">Questi approcci sono illustrati negli esempi seguenti.</span><span class="sxs-lookup"><span data-stu-id="275b6-208">These approaches are demonstrated in the following examples.</span></span>

<!-- HOLD for reactivation at 5.x

::: moniker-end

-->

<span data-ttu-id="275b6-209">Aggiungere il `Jeep` componente seguente a una delle app client.</span><span class="sxs-lookup"><span data-stu-id="275b6-209">Add the following `Jeep` component to one of the client apps.</span></span> <span data-ttu-id="275b6-210">Il `Jeep` componente USA:</span><span class="sxs-lookup"><span data-stu-id="275b6-210">The `Jeep` component uses:</span></span>

* <span data-ttu-id="275b6-211">Un'immagine dalla cartella dell'app client `wwwroot` ( `jeep-cj.png` ).</span><span class="sxs-lookup"><span data-stu-id="275b6-211">An image from the client app's `wwwroot` folder (`jeep-cj.png`).</span></span>
* <span data-ttu-id="275b6-212">Un'immagine da una cartella della [ Razor libreria dei componenti](xref:blazor/components/class-libraries) () aggiunta () `JeepImage` `wwwroot` `jeep-yj.png` .</span><span class="sxs-lookup"><span data-stu-id="275b6-212">An image from an [added Razor component library](xref:blazor/components/class-libraries) (`JeepImage`) `wwwroot` folder (`jeep-yj.png`).</span></span>
* <span data-ttu-id="275b6-213">Il componente di esempio ( `Component1` ) viene creato automaticamente dal modello di progetto RCL quando la `JeepImage` libreria viene aggiunta alla soluzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-213">The example component (`Component1`) is created automatically by the RCL project template when the `JeepImage` library is added to the solution.</span></span>

```razor
@page "/Jeep"

<h1>1979 Jeep CJ-5&trade;</h1>

<p>
    <img alt="1979 Jeep CJ-5&trade;" src="/jeep-cj.png" />
</p>

<h1>1991 Jeep YJ&trade;</h1>

<p>
    <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
</p>

<p>
    <em>Jeep CJ-5</em> and <em>Jeep YJ</em> are a trademarks of 
    <a href="https://www.fcagroup.com">Fiat Chrysler Automobiles</a>.
</p>

<JeepImage.Component1 />
```

> [!WARNING]
> <span data-ttu-id="275b6-214">**Non** pubblicare le immagini dei veicoli pubblicamente, a meno che non si possiedano le immagini.</span><span class="sxs-lookup"><span data-stu-id="275b6-214">Do **not** publish images of vehicles publicly unless you own the images.</span></span> <span data-ttu-id="275b6-215">In caso contrario, si rischia di violare il copyright.</span><span class="sxs-lookup"><span data-stu-id="275b6-215">Otherwise, you risk copyright infringement.</span></span>

<!-- HOLD for reactivation at 5.x

::: moniker range=">= aspnetcore-5.0"

The library's `jeep-yj.png` image can also be added to the library's `Component1` component (`Component1.razor`). To provide the `my-component` CSS class to the client app's page, link to the library's stylesheet using the framework's [`Link` component](xref:blazor/fundamentals/signalr#influence-html-head-tag-elements):

```razor
<div class="my-component">
    <Link href="_content/JeepImage/styles.css" rel="stylesheet" />

    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

An alternative to using the [`Link` component](xref:blazor/fundamentals/signalr#influence-html-head-tag-elements) is to load the stylesheet from the client app's `wwwroot/index.html` file. This approach makes the stylesheet available to all of the components in the client app:

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

-->

<span data-ttu-id="275b6-216">È `jeep-yj.png` possibile aggiungere l'immagine della libreria anche al componente della libreria `Component1` ( `Component1.razor` ):</span><span class="sxs-lookup"><span data-stu-id="275b6-216">The library's `jeep-yj.png` image can also be added to the library's `Component1` component (`Component1.razor`):</span></span>

```razor
<div class="my-component">
    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

<span data-ttu-id="275b6-217">Il file dell'app client `wwwroot/index.html` richiede il foglio di stile della libreria con il seguente `<link>` tag aggiunto:</span><span class="sxs-lookup"><span data-stu-id="275b6-217">The client app's `wwwroot/index.html` file requests the library's stylesheet with the following added `<link>` tag:</span></span>

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

<!-- HOLD for reactivation at 5.x

::: moniker-end

-->

<span data-ttu-id="275b6-218">Aggiungere la navigazione al `Jeep` componente nel componente dell'app client `NavMenu` ( `Shared/NavMenu.razor` ):</span><span class="sxs-lookup"><span data-stu-id="275b6-218">Add navigation to the `Jeep` component in the client app's `NavMenu` component (`Shared/NavMenu.razor`):</span></span>

```razor
<li class="nav-item px-3">
    <NavLink class="nav-link" href="Jeep">
        <span class="oi oi-list-rich" aria-hidden="true"></span> Jeep
    </NavLink>
</li>
```

<span data-ttu-id="275b6-219">Per ulteriori informazioni su RCL, vedere:</span><span class="sxs-lookup"><span data-stu-id="275b6-219">For more information on RCLs, see:</span></span>

* <xref:blazor/components/class-libraries>
* <xref:razor-pages/ui-class>

## <a name="standalone-deployment"></a><span data-ttu-id="275b6-220">Distribuzione autonoma</span><span class="sxs-lookup"><span data-stu-id="275b6-220">Standalone deployment</span></span>

<span data-ttu-id="275b6-221">Una *distribuzione autonoma* serve l' Blazor WebAssembly app come un set di file statici richiesti direttamente dai client.</span><span class="sxs-lookup"><span data-stu-id="275b6-221">A *standalone deployment* serves the Blazor WebAssembly app as a set of static files that are requested directly by clients.</span></span> <span data-ttu-id="275b6-222">Qualsiasi file server statico è in grado di gestire l' Blazor app.</span><span class="sxs-lookup"><span data-stu-id="275b6-222">Any static file server is able to serve the Blazor app.</span></span>

<span data-ttu-id="275b6-223">Le risorse di distribuzione autonome vengono pubblicate nella `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` cartella.</span><span class="sxs-lookup"><span data-stu-id="275b6-223">Standalone deployment assets are published into the `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` folder.</span></span>

### <a name="azure-app-service"></a><span data-ttu-id="275b6-224">Servizio app di Azure</span><span class="sxs-lookup"><span data-stu-id="275b6-224">Azure App Service</span></span>

<span data-ttu-id="275b6-225">Blazor WebAssembly le app possono essere distribuite in app Azure Services in Windows, che ospita l'app in [IIS](#iis).</span><span class="sxs-lookup"><span data-stu-id="275b6-225">Blazor WebAssembly apps can be deployed to Azure App Services on Windows, which hosts the app on [IIS](#iis).</span></span>

<span data-ttu-id="275b6-226">La distribuzione di un'app autonoma Blazor WebAssembly nel servizio app Azure per Linux non è attualmente supportata.</span><span class="sxs-lookup"><span data-stu-id="275b6-226">Deploying a standalone Blazor WebAssembly app to Azure App Service for Linux isn't currently supported.</span></span> <span data-ttu-id="275b6-227">Un'immagine server Linux per ospitare l'app non è disponibile in questo momento.</span><span class="sxs-lookup"><span data-stu-id="275b6-227">A Linux server image to host the app isn't available at this time.</span></span> <span data-ttu-id="275b6-228">Il lavoro è in corso per abilitare questo scenario.</span><span class="sxs-lookup"><span data-stu-id="275b6-228">Work is in progress to enable this scenario.</span></span>

### <a name="iis"></a><span data-ttu-id="275b6-229">IIS</span><span class="sxs-lookup"><span data-stu-id="275b6-229">IIS</span></span>

<span data-ttu-id="275b6-230">IIS è un file server statico idoneo per le Blazor app.</span><span class="sxs-lookup"><span data-stu-id="275b6-230">IIS is a capable static file server for Blazor apps.</span></span> <span data-ttu-id="275b6-231">Per configurare IIS per l'hosting Blazor , vedere [la pagina relativa alla creazione di un sito Web statico in IIS](/iis/manage/creating-websites/scenario-build-a-static-website-on-iis).</span><span class="sxs-lookup"><span data-stu-id="275b6-231">To configure IIS to host Blazor, see [Build a Static Website on IIS](/iis/manage/creating-websites/scenario-build-a-static-website-on-iis).</span></span>

<span data-ttu-id="275b6-232">Gli asset pubblicati vengono creati nella `/bin/Release/{TARGET FRAMEWORK}/publish` cartella.</span><span class="sxs-lookup"><span data-stu-id="275b6-232">Published assets are created in the `/bin/Release/{TARGET FRAMEWORK}/publish` folder.</span></span> <span data-ttu-id="275b6-233">Ospitare il contenuto della `publish` cartella nel server Web o nel servizio di hosting.</span><span class="sxs-lookup"><span data-stu-id="275b6-233">Host the contents of the `publish` folder on the web server or hosting service.</span></span>

#### <a name="webconfig"></a><span data-ttu-id="275b6-234">web.config</span><span class="sxs-lookup"><span data-stu-id="275b6-234">web.config</span></span>

<span data-ttu-id="275b6-235">Quando Blazor viene pubblicato un progetto, `web.config` viene creato un file con la seguente configurazione di IIS:</span><span class="sxs-lookup"><span data-stu-id="275b6-235">When a Blazor project is published, a `web.config` file is created with the following IIS configuration:</span></span>

* <span data-ttu-id="275b6-236">Vengono impostati tipi MIME per le estensioni di file seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-236">MIME types are set for the following file extensions:</span></span>
  * <span data-ttu-id="275b6-237">`.dll`: `application/octet-stream`</span><span class="sxs-lookup"><span data-stu-id="275b6-237">`.dll`: `application/octet-stream`</span></span>
  * <span data-ttu-id="275b6-238">`.json`: `application/json`</span><span class="sxs-lookup"><span data-stu-id="275b6-238">`.json`: `application/json`</span></span>
  * <span data-ttu-id="275b6-239">`.wasm`: `application/wasm`</span><span class="sxs-lookup"><span data-stu-id="275b6-239">`.wasm`: `application/wasm`</span></span>
  * <span data-ttu-id="275b6-240">`.woff`: `application/font-woff`</span><span class="sxs-lookup"><span data-stu-id="275b6-240">`.woff`: `application/font-woff`</span></span>
  * <span data-ttu-id="275b6-241">`.woff2`: `application/font-woff`</span><span class="sxs-lookup"><span data-stu-id="275b6-241">`.woff2`: `application/font-woff`</span></span>
* <span data-ttu-id="275b6-242">Viene abilitata la compressione HTTP per i tipi MIME seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-242">HTTP compression is enabled for the following MIME types:</span></span>
  * `application/octet-stream`
  * `application/wasm`
* <span data-ttu-id="275b6-243">Vengono stabilite le regole URL Rewrite Module:</span><span class="sxs-lookup"><span data-stu-id="275b6-243">URL Rewrite Module rules are established:</span></span>
  * <span data-ttu-id="275b6-244">Gestire la sottodirectory in cui risiedono gli asset statici dell'app ( `wwwroot/{PATH REQUESTED}` ).</span><span class="sxs-lookup"><span data-stu-id="275b6-244">Serve the sub-directory where the app's static assets reside (`wwwroot/{PATH REQUESTED}`).</span></span>
  * <span data-ttu-id="275b6-245">Creare il routing di fallback SPA in modo che le richieste di risorse non file vengano reindirizzate al documento predefinito dell'app nella cartella Asset statici ( `wwwroot/index.html` ).</span><span class="sxs-lookup"><span data-stu-id="275b6-245">Create SPA fallback routing so that requests for non-file assets are redirected to the app's default document in its static assets folder (`wwwroot/index.html`).</span></span>
  
#### <a name="use-a-custom-webconfig"></a><span data-ttu-id="275b6-246">Usare un web.config personalizzato</span><span class="sxs-lookup"><span data-stu-id="275b6-246">Use a custom web.config</span></span>

<span data-ttu-id="275b6-247">Per usare un `web.config` file personalizzato, inserire il `web.config` file personalizzato nella radice della cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="275b6-247">To use a custom `web.config` file, place the custom `web.config` file at the root of the project folder.</span></span> <span data-ttu-id="275b6-248">Configurare il progetto per pubblicare asset specifici di IIS usando `PublishIISAssets` nel file di progetto dell'app e pubblicare il progetto:</span><span class="sxs-lookup"><span data-stu-id="275b6-248">Configure the project to publish IIS-specific assets using `PublishIISAssets` in the app's project file and publish the project:</span></span>

```xml
<PropertyGroup>
  <PublishIISAssets>true</PublishIISAssets>
</PropertyGroup>
```

#### <a name="install-the-url-rewrite-module"></a><span data-ttu-id="275b6-249">Installare URL Rewrite Module</span><span class="sxs-lookup"><span data-stu-id="275b6-249">Install the URL Rewrite Module</span></span>

<span data-ttu-id="275b6-250">[URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) è necessario per riscrivere gli URL.</span><span class="sxs-lookup"><span data-stu-id="275b6-250">The [URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) is required to rewrite URLs.</span></span> <span data-ttu-id="275b6-251">Il modulo non viene installato per impostazione predefinita e non è disponibile per l'installazione come funzionalità del servizio ruolo Server Web (IIS).</span><span class="sxs-lookup"><span data-stu-id="275b6-251">The module isn't installed by default, and it isn't available for install as a Web Server (IIS) role service feature.</span></span> <span data-ttu-id="275b6-252">Il modulo deve essere scaricato dal sito Web IIS.</span><span class="sxs-lookup"><span data-stu-id="275b6-252">The module must be downloaded from the IIS website.</span></span> <span data-ttu-id="275b6-253">Usare Installazione guidata piattaforma Web per installare il modulo:</span><span class="sxs-lookup"><span data-stu-id="275b6-253">Use the Web Platform Installer to install the module:</span></span>

1. <span data-ttu-id="275b6-254">In locale, passare alla [pagina di download di URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads).</span><span class="sxs-lookup"><span data-stu-id="275b6-254">Locally, navigate to the [URL Rewrite Module downloads page](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads).</span></span> <span data-ttu-id="275b6-255">Per la versione in lingua inglese selezionare **WebPI** per scaricare il programma di installazione di Installazione guidata piattaforma Web.</span><span class="sxs-lookup"><span data-stu-id="275b6-255">For the English version, select **WebPI** to download the WebPI installer.</span></span> <span data-ttu-id="275b6-256">Per altre lingue, selezionare l'architettura appropriata per il server (x86 o x64) per scaricare il programma di installazione.</span><span class="sxs-lookup"><span data-stu-id="275b6-256">For other languages, select the appropriate architecture for the server (x86/x64) to download the installer.</span></span>
1. <span data-ttu-id="275b6-257">Copiare il programma di installazione nel server.</span><span class="sxs-lookup"><span data-stu-id="275b6-257">Copy the installer to the server.</span></span> <span data-ttu-id="275b6-258">Eseguire il programma di installazione.</span><span class="sxs-lookup"><span data-stu-id="275b6-258">Run the installer.</span></span> <span data-ttu-id="275b6-259">Selezionare il pulsante **Installa** e accettare le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="275b6-259">Select the **Install** button and accept the license terms.</span></span> <span data-ttu-id="275b6-260">Al termine dell'installazione non è necessario un riavvio del server.</span><span class="sxs-lookup"><span data-stu-id="275b6-260">A server restart isn't required after the install completes.</span></span>

#### <a name="configure-the-website"></a><span data-ttu-id="275b6-261">Configurare il sito Web</span><span class="sxs-lookup"><span data-stu-id="275b6-261">Configure the website</span></span>

<span data-ttu-id="275b6-262">Impostare il **percorso fisico** del sito Web sulla cartella dell'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-262">Set the website's **Physical path** to the app's folder.</span></span> <span data-ttu-id="275b6-263">La cartella contiene:</span><span class="sxs-lookup"><span data-stu-id="275b6-263">The folder contains:</span></span>

* <span data-ttu-id="275b6-264">Il `web.config` file utilizzato da IIS per configurare il sito Web, incluse le regole di reindirizzamento richieste e i tipi di contenuto del file.</span><span class="sxs-lookup"><span data-stu-id="275b6-264">The `web.config` file that IIS uses to configure the website, including the required redirect rules and file content types.</span></span>
* <span data-ttu-id="275b6-265">Cartella degli asset statici dell'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-265">The app's static asset folder.</span></span>

#### <a name="host-as-an-iis-sub-app"></a><span data-ttu-id="275b6-266">Ospitare come applicazione secondaria IIS</span><span class="sxs-lookup"><span data-stu-id="275b6-266">Host as an IIS sub-app</span></span>

<span data-ttu-id="275b6-267">Se un'app autonoma è ospitata come una sub-app IIS, eseguire una delle operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-267">If a standalone app is hosted as an IIS sub-app, perform either of the following:</span></span>

* <span data-ttu-id="275b6-268">Disabilitare il gestore del modulo ASP.NET Core ereditato.</span><span class="sxs-lookup"><span data-stu-id="275b6-268">Disable the inherited ASP.NET Core Module handler.</span></span>

  <span data-ttu-id="275b6-269">Rimuovere il gestore nel Blazor file pubblicato dell'app `web.config` aggiungendo una `<handlers>` sezione al file:</span><span class="sxs-lookup"><span data-stu-id="275b6-269">Remove the handler in the Blazor app's published `web.config` file by adding a `<handlers>` section to the file:</span></span>

  ```xml
  <handlers>
    <remove name="aspNetCore" />
  </handlers>
  ```

* <span data-ttu-id="275b6-270">Disabilitare l'ereditarietà della sezione radice (padre) dell'app `<system.webServer>` usando un `<location>` elemento con `inheritInChildApplications` impostato su `false` :</span><span class="sxs-lookup"><span data-stu-id="275b6-270">Disable inheritance of the root (parent) app's `<system.webServer>` section using a `<location>` element with `inheritInChildApplications` set to `false`:</span></span>

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <handlers>
          <add name="aspNetCore" ... />
        </handlers>
        <aspNetCore ... />
      </system.webServer>
    </location>
  </configuration>
  ```

<span data-ttu-id="275b6-271">La rimozione del gestore o la disabilitazione dell'ereditarietà viene eseguita oltre alla [configurazione del percorso di base dell'applicazione](xref:blazor/host-and-deploy/index#app-base-path).</span><span class="sxs-lookup"><span data-stu-id="275b6-271">Removing the handler or disabling inheritance is performed in addition to [configuring the app's base path](xref:blazor/host-and-deploy/index#app-base-path).</span></span> <span data-ttu-id="275b6-272">Impostare il percorso di base dell'app nel file dell'app `index.html` sull'alias IIS usato durante la configurazione della Sub-app in IIS.</span><span class="sxs-lookup"><span data-stu-id="275b6-272">Set the app base path in the app's `index.html` file to the IIS alias used when configuring the sub-app in IIS.</span></span>

#### <a name="brotli-and-gzip-compression"></a><span data-ttu-id="275b6-273">Compressione Brotli e gzip</span><span class="sxs-lookup"><span data-stu-id="275b6-273">Brotli and Gzip compression</span></span>

<span data-ttu-id="275b6-274">*Questa sezione si applica solo alle app autonome Blazor WebAssembly . Blazor le app ospitate usano un file di ASP.NET Core app predefinito `web.config` , non il file collegato in questa sezione.*</span><span class="sxs-lookup"><span data-stu-id="275b6-274">*This section only applies to standalone Blazor WebAssembly apps. Hosted Blazor apps use a default ASP.NET Core app `web.config` file, not the file linked in this section.*</span></span>

<span data-ttu-id="275b6-275">IIS può essere configurato tramite `web.config` per gestire asset Brotli o gzip compressi Blazor per le Blazor WebAssembly app autonome.</span><span class="sxs-lookup"><span data-stu-id="275b6-275">IIS can be configured via `web.config` to serve Brotli or Gzip compressed Blazor assets for standalone Blazor WebAssembly apps.</span></span> <span data-ttu-id="275b6-276">Per un esempio di file di configurazione, vedere [`web.config`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/web.config?raw=true) .</span><span class="sxs-lookup"><span data-stu-id="275b6-276">For an example configuration file, see [`web.config`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/web.config?raw=true).</span></span>

<span data-ttu-id="275b6-277">È possibile che sia necessaria una configurazione aggiuntiva del file di esempio `web.config` negli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-277">Additional configuration of the example `web.config` file might be required in the following scenarios:</span></span>

* <span data-ttu-id="275b6-278">La specifica dell'app chiama per uno dei seguenti elementi:</span><span class="sxs-lookup"><span data-stu-id="275b6-278">The app's specification calls for either of the following:</span></span>
  * <span data-ttu-id="275b6-279">Vengono serviti file compressi che non sono configurati dal file di esempio `web.config` .</span><span class="sxs-lookup"><span data-stu-id="275b6-279">Serving compressed files that aren't configured by the example `web.config` file.</span></span>
  * <span data-ttu-id="275b6-280">Conservazione dei file compressi configurati dal file di esempio `web.config` in un formato non compresso.</span><span class="sxs-lookup"><span data-stu-id="275b6-280">Serving compressed files configured by the example `web.config` file in an uncompressed format.</span></span>
* <span data-ttu-id="275b6-281">La configurazione IIS del server (ad esempio, `applicationHost.config` ) fornisce le impostazioni predefinite di IIS a livello di server.</span><span class="sxs-lookup"><span data-stu-id="275b6-281">The server's IIS configuration (for example, `applicationHost.config`) provides server-level IIS defaults.</span></span> <span data-ttu-id="275b6-282">A seconda della configurazione a livello di server, è possibile che l'app richieda una configurazione IIS diversa rispetto a quella `web.config` contenuta nel file di esempio.</span><span class="sxs-lookup"><span data-stu-id="275b6-282">Depending on the server-level configuration, the app might require a different IIS configuration than what the example `web.config` file contains.</span></span>

#### <a name="troubleshooting"></a><span data-ttu-id="275b6-283">Risoluzione dei problemi</span><span class="sxs-lookup"><span data-stu-id="275b6-283">Troubleshooting</span></span>

<span data-ttu-id="275b6-284">Se si riceve un *errore interno del server 500* e Gestione IIS genera errori durante il tentativo di accedere alla configurazione del sito Web, verificare che sia installato URL Rewrite Module.</span><span class="sxs-lookup"><span data-stu-id="275b6-284">If a *500 - Internal Server Error* is received and IIS Manager throws errors when attempting to access the website's configuration, confirm that the URL Rewrite Module is installed.</span></span> <span data-ttu-id="275b6-285">Quando il modulo non è installato, il `web.config` file non può essere analizzato da IIS.</span><span class="sxs-lookup"><span data-stu-id="275b6-285">When the module isn't installed, the `web.config` file can't be parsed by IIS.</span></span> <span data-ttu-id="275b6-286">In questo modo si impedisce al gestore IIS di caricare la configurazione del sito Web e il sito Web dai Blazor file statici del servizio.</span><span class="sxs-lookup"><span data-stu-id="275b6-286">This prevents the IIS Manager from loading the website's configuration and the website from serving Blazor's static files.</span></span>

<span data-ttu-id="275b6-287">Per altre informazioni sulla risoluzione dei problemi relativi alle distribuzioni in IIS, vedere <xref:test/troubleshoot-azure-iis>.</span><span class="sxs-lookup"><span data-stu-id="275b6-287">For more information on troubleshooting deployments to IIS, see <xref:test/troubleshoot-azure-iis>.</span></span>

### <a name="azure-storage"></a><span data-ttu-id="275b6-288">Archiviazione di Azure</span><span class="sxs-lookup"><span data-stu-id="275b6-288">Azure Storage</span></span>

<span data-ttu-id="275b6-289">L'hosting di file statici di [archiviazione di Azure](/azure/storage/) consente l'hosting di app senza server Blazor .</span><span class="sxs-lookup"><span data-stu-id="275b6-289">[Azure Storage](/azure/storage/) static file hosting allows serverless Blazor app hosting.</span></span> <span data-ttu-id="275b6-290">Sono supportati nomi di dominio personalizzati, la rete per la distribuzione di contenuti (rete CDN) di Azure e HTTPS.</span><span class="sxs-lookup"><span data-stu-id="275b6-290">Custom domain names, the Azure Content Delivery Network (CDN), and HTTPS are supported.</span></span>

<span data-ttu-id="275b6-291">Quando il servizio BLOB è abilitato per l'hosting di siti Web statici in un account di archiviazione:</span><span class="sxs-lookup"><span data-stu-id="275b6-291">When the blob service is enabled for static website hosting on a storage account:</span></span>

* <span data-ttu-id="275b6-292">Impostare **Nome del documento di indice** su `index.html`.</span><span class="sxs-lookup"><span data-stu-id="275b6-292">Set the **Index document name** to `index.html`.</span></span>
* <span data-ttu-id="275b6-293">Impostare **Percorso del documento di errore** su `index.html`.</span><span class="sxs-lookup"><span data-stu-id="275b6-293">Set the **Error document path** to `index.html`.</span></span> <span data-ttu-id="275b6-294">Razor i componenti e altri endpoint non di file non si trovano in percorsi fisici nel contenuto statico archiviato dal servizio BLOB.</span><span class="sxs-lookup"><span data-stu-id="275b6-294">Razor components and other non-file endpoints don't reside at physical paths in the static content stored by the blob service.</span></span> <span data-ttu-id="275b6-295">Quando viene ricevuta una richiesta per una di queste risorse che il Blazor router deve gestire, l'errore *404 non trovato* generato dal servizio BLOB instrada la richiesta al **percorso del documento di errore**.</span><span class="sxs-lookup"><span data-stu-id="275b6-295">When a request for one of these resources is received that the Blazor router should handle, the *404 - Not Found* error generated by the blob service routes the request to the **Error document path**.</span></span> <span data-ttu-id="275b6-296">`index.html`Viene restituito il BLOB e il Blazor router carica ed elabora il percorso.</span><span class="sxs-lookup"><span data-stu-id="275b6-296">The `index.html` blob is returned, and the Blazor router loads and processes the path.</span></span>

<span data-ttu-id="275b6-297">Se i file non vengono caricati in fase di esecuzione a causa di tipi MIME non appropriati nelle intestazioni dei file `Content-Type` , eseguire una delle azioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-297">If files aren't loaded at runtime due to inappropriate MIME types in the files' `Content-Type` headers, take either of the following actions:</span></span>

* <span data-ttu-id="275b6-298">Configurare gli strumenti per impostare i tipi MIME corretti ( `Content-Type` intestazioni) quando vengono distribuiti i file.</span><span class="sxs-lookup"><span data-stu-id="275b6-298">Configure your tooling to set the correct MIME types (`Content-Type` headers) when the files are deployed.</span></span>
* <span data-ttu-id="275b6-299">Modificare i tipi MIME ( `Content-Type` intestazioni) per i file dopo la distribuzione dell'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-299">Change the MIME types (`Content-Type` headers) for the files after the app is deployed.</span></span>

  <span data-ttu-id="275b6-300">In Storage Explorer (portale di Azure) per ogni file:</span><span class="sxs-lookup"><span data-stu-id="275b6-300">In Storage Explorer (Azure portal) for each file:</span></span>
  
  1. <span data-ttu-id="275b6-301">Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà**.</span><span class="sxs-lookup"><span data-stu-id="275b6-301">Right-click the file and select **Properties**.</span></span>
  1. <span data-ttu-id="275b6-302">Impostare **ContentType** e selezionare il pulsante **Salva** .</span><span class="sxs-lookup"><span data-stu-id="275b6-302">Set the **ContentType** and select the **Save** button.</span></span>

<span data-ttu-id="275b6-303">Per altre informazioni, vedere [Hosting di siti Web statici in Archiviazione di Azure](/azure/storage/blobs/storage-blob-static-website).</span><span class="sxs-lookup"><span data-stu-id="275b6-303">For more information, see [Static website hosting in Azure Storage](/azure/storage/blobs/storage-blob-static-website).</span></span>

### <a name="nginx"></a><span data-ttu-id="275b6-304">Nginx</span><span class="sxs-lookup"><span data-stu-id="275b6-304">Nginx</span></span>

<span data-ttu-id="275b6-305">Il `nginx.conf` file seguente è stato semplificato per illustrare come configurare Nginx per inviare il `index.html` file ogni volta che non riesce a trovare un file corrispondente sul disco.</span><span class="sxs-lookup"><span data-stu-id="275b6-305">The following `nginx.conf` file is simplified to show how to configure Nginx to send the `index.html` file whenever it can't find a corresponding file on disk.</span></span>

```
events { }
http {
    server {
        listen 80;

        location / {
            root      /usr/share/nginx/html;
            try_files $uri $uri/ /index.html =404;
        }
    }
}
```

<span data-ttu-id="275b6-306">Quando si imposta il [limite di velocità](https://www.nginx.com/blog/rate-limiting-nginx/#bursts) di espansione nginx con, le [`limit_req`](https://nginx.org/docs/http/ngx_http_limit_req_module.html#limit_req) Blazor WebAssembly app possono richiedere un `burst` valore di parametro di grandi dimensioni per contenere il numero relativamente elevato di richieste effettuate da un'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-306">When setting the [NGINX burst rate limit](https://www.nginx.com/blog/rate-limiting-nginx/#bursts) with [`limit_req`](https://nginx.org/docs/http/ngx_http_limit_req_module.html#limit_req), Blazor WebAssembly apps may require a large `burst` parameter value to accommodate the relatively large number of requests made by an app.</span></span> <span data-ttu-id="275b6-307">Inizialmente, impostare il valore su almeno 60:</span><span class="sxs-lookup"><span data-stu-id="275b6-307">Initially, set the value to at least 60:</span></span>

```
http {
    server {
        ...

        location / {
            ...

            limit_req zone=one burst=60 nodelay;
        }
    }
}
```

<span data-ttu-id="275b6-308">Aumentare il valore se browser Developer Tools o uno strumento di traffico di rete indica che le richieste ricevono un codice di stato *503-servizio non disponibile* .</span><span class="sxs-lookup"><span data-stu-id="275b6-308">Increase the value if browser developer tools or a network traffic tool indicates that requests are receiving a *503 - Service Unavailable* status code.</span></span>

<span data-ttu-id="275b6-309">Per altre informazioni sulla configurazione del server Web Nginx in ambiente di produzione, vedere [Creating NGINX Plus and NGINX Configuration Files](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/) (Creazione di file di configurazione NGINX Plus e NGINX).</span><span class="sxs-lookup"><span data-stu-id="275b6-309">For more information on production Nginx web server configuration, see [Creating NGINX Plus and NGINX Configuration Files](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/).</span></span>

### <a name="apache"></a><span data-ttu-id="275b6-310">Apache</span><span class="sxs-lookup"><span data-stu-id="275b6-310">Apache</span></span>

<span data-ttu-id="275b6-311">Per distribuire un' Blazor WebAssembly app in CentOS 7 o versione successiva:</span><span class="sxs-lookup"><span data-stu-id="275b6-311">To deploy a Blazor WebAssembly app to CentOS 7 or later:</span></span>

1. <span data-ttu-id="275b6-312">Creare il file di configurazione Apache.</span><span class="sxs-lookup"><span data-stu-id="275b6-312">Create the Apache configuration file.</span></span> <span data-ttu-id="275b6-313">L'esempio seguente è un file di configurazione semplificato ( `blazorapp.config` ):</span><span class="sxs-lookup"><span data-stu-id="275b6-313">The following example is a simplified configuration file (`blazorapp.config`):</span></span>

   ```
   <VirtualHost *:80>
       ServerName www.example.com
       ServerAlias *.example.com

       DocumentRoot "/var/www/blazorapp"
       ErrorDocument 404 /index.html

       AddType application/wasm .wasm
       AddType application/octet-stream .dll
   
       <Directory "/var/www/blazorapp">
           Options -Indexes
           AllowOverride None
       </Directory>

       <IfModule mod_deflate.c>
           AddOutputFilterByType DEFLATE text/css
           AddOutputFilterByType DEFLATE application/javascript
           AddOutputFilterByType DEFLATE text/html
           AddOutputFilterByType DEFLATE application/octet-stream
           AddOutputFilterByType DEFLATE application/wasm
           <IfModule mod_setenvif.c>
           BrowserMatch ^Mozilla/4 gzip-only-text/html
           BrowserMatch ^Mozilla/4.0[678] no-gzip
           BrowserMatch bMSIE !no-gzip !gzip-only-text/html
       </IfModule>
       </IfModule>

       ErrorLog /var/log/httpd/blazorapp-error.log
       CustomLog /var/log/httpd/blazorapp-access.log common
   </VirtualHost>
   ```

1. <span data-ttu-id="275b6-314">Inserire il file di configurazione Apache nella `/etc/httpd/conf.d/` Directory, ovvero la directory di configurazione predefinita di Apache in CentOS 7.</span><span class="sxs-lookup"><span data-stu-id="275b6-314">Place the Apache configuration file into the `/etc/httpd/conf.d/` directory, which is the default Apache configuration directory in CentOS 7.</span></span>

1. <span data-ttu-id="275b6-315">Inserire i file dell'app nella `/var/www/blazorapp` Directory (il percorso specificato `DocumentRoot` nel file di configurazione).</span><span class="sxs-lookup"><span data-stu-id="275b6-315">Place the app's files into the `/var/www/blazorapp` directory (the location specified to `DocumentRoot` in the configuration file).</span></span>

1. <span data-ttu-id="275b6-316">Riavviare il servizio Apache.</span><span class="sxs-lookup"><span data-stu-id="275b6-316">Restart the Apache service.</span></span>

<span data-ttu-id="275b6-317">Per altre informazioni, vedere [`mod_mime`](https://httpd.apache.org/docs/2.4/mod/mod_mime.html) e [`mod_deflate`](https://httpd.apache.org/docs/current/mod/mod_deflate.html).</span><span class="sxs-lookup"><span data-stu-id="275b6-317">For more information, see [`mod_mime`](https://httpd.apache.org/docs/2.4/mod/mod_mime.html) and [`mod_deflate`](https://httpd.apache.org/docs/current/mod/mod_deflate.html).</span></span>

### <a name="github-pages"></a><span data-ttu-id="275b6-318">GitHub Pages</span><span class="sxs-lookup"><span data-stu-id="275b6-318">GitHub Pages</span></span>

<span data-ttu-id="275b6-319">Per gestire le riscritture URL, aggiungere un `wwwroot/404.html` file con uno script che gestisce il reindirizzamento della richiesta alla `index.html` pagina.</span><span class="sxs-lookup"><span data-stu-id="275b6-319">To handle URL rewrites, add a `wwwroot/404.html` file with a script that handles redirecting the request to the `index.html` page.</span></span> <span data-ttu-id="275b6-320">Per un esempio, vedere il [ Blazor repository GitHub SteveSandersonMS/OnGitHubPages](https://github.com/SteveSandersonMS/BlazorOnGitHubPages):</span><span class="sxs-lookup"><span data-stu-id="275b6-320">For an example, see the [SteveSandersonMS/BlazorOnGitHubPages GitHub repository](https://github.com/SteveSandersonMS/BlazorOnGitHubPages):</span></span>

* [`wwwroot/404.html`](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/wwwroot/404.html)
* <span data-ttu-id="275b6-321">[Sito Live](https://stevesandersonms.github.io/BlazorOnGitHubPages/))</span><span class="sxs-lookup"><span data-stu-id="275b6-321">[Live site](https://stevesandersonms.github.io/BlazorOnGitHubPages/))</span></span>

<span data-ttu-id="275b6-322">Quando si usa un sito del progetto anziché un sito dell'organizzazione, aggiornare il `<base>` tag in `wwwroot/index.html` .</span><span class="sxs-lookup"><span data-stu-id="275b6-322">When using a project site instead of an organization site, update the `<base>` tag in `wwwroot/index.html`.</span></span> <span data-ttu-id="275b6-323">Impostare il `href` valore dell'attributo sul nome del repository GitHub con una barra finale (ad esempio, `/my-repository/` ).</span><span class="sxs-lookup"><span data-stu-id="275b6-323">Set the `href` attribute value to the GitHub repository name with a trailing slash (for example, `/my-repository/`).</span></span> <span data-ttu-id="275b6-324">Nel [ Blazor repository GitHub SteveSandersonMS/OnGitHubPages](https://github.com/SteveSandersonMS/BlazorOnGitHubPages)la base `href` viene aggiornata in fase di pubblicazione dal [ `.github/workflows/main.yml` file di configurazione](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/.github/workflows/main.yml).</span><span class="sxs-lookup"><span data-stu-id="275b6-324">In the [SteveSandersonMS/BlazorOnGitHubPages GitHub repository](https://github.com/SteveSandersonMS/BlazorOnGitHubPages), the base `href` is updated at publish by the [`.github/workflows/main.yml` configuration file](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/.github/workflows/main.yml).</span></span>

> [!NOTE]
> <span data-ttu-id="275b6-325">Il [ Blazor repository GitHub SteveSandersonMS/OnGitHubPages](https://github.com/SteveSandersonMS/BlazorOnGitHubPages) non è di proprietà, è stato mantenuto o supportato da .NET Foundation o Microsoft.</span><span class="sxs-lookup"><span data-stu-id="275b6-325">The [SteveSandersonMS/BlazorOnGitHubPages GitHub repository](https://github.com/SteveSandersonMS/BlazorOnGitHubPages) isn't owned, maintained, or supported by the .NET Foundation or Microsoft.</span></span>

## <a name="host-configuration-values"></a><span data-ttu-id="275b6-326">Valori di configurazione dell'host</span><span class="sxs-lookup"><span data-stu-id="275b6-326">Host configuration values</span></span>

<span data-ttu-id="275b6-327">le [ Blazor WebAssembly app](xref:blazor/hosting-models#blazor-webassembly) possono accettare i valori di configurazione host seguenti come argomenti della riga di comando in fase di esecuzione nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="275b6-327">[Blazor WebAssembly apps](xref:blazor/hosting-models#blazor-webassembly) can accept the following host configuration values as command-line arguments at runtime in the development environment.</span></span>

### <a name="content-root"></a><span data-ttu-id="275b6-328">Radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="275b6-328">Content root</span></span>

<span data-ttu-id="275b6-329">L' `--contentroot` argomento imposta il percorso assoluto della directory che contiene i file di contenuto dell'app ([radice del contenuto](xref:fundamentals/index#content-root)).</span><span class="sxs-lookup"><span data-stu-id="275b6-329">The `--contentroot` argument sets the absolute path to the directory that contains the app's content files ([content root](xref:fundamentals/index#content-root)).</span></span> <span data-ttu-id="275b6-330">Negli esempi seguenti `/content-root-path` è il percorso radice del contenuto dell'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-330">In the following examples, `/content-root-path` is the app's content root path.</span></span>

* <span data-ttu-id="275b6-331">Passare l'argomento quando si esegue localmente l'app a un prompt dei comandi.</span><span class="sxs-lookup"><span data-stu-id="275b6-331">Pass the argument when running the app locally at a command prompt.</span></span> <span data-ttu-id="275b6-332">Dalla directory dell'app, eseguire:</span><span class="sxs-lookup"><span data-stu-id="275b6-332">From the app's directory, execute:</span></span>

  ```dotnetcli
  dotnet run --contentroot=/content-root-path
  ```

* <span data-ttu-id="275b6-333">Aggiungere una voce al file dell'app `launchSettings.json` nel profilo di **IIS Express** .</span><span class="sxs-lookup"><span data-stu-id="275b6-333">Add an entry to the app's `launchSettings.json` file in the **IIS Express** profile.</span></span> <span data-ttu-id="275b6-334">Questa impostazione viene usata quando l'app viene eseguita con il debugger di Visual Studio e da un prompt dei comandi con `dotnet run`.</span><span class="sxs-lookup"><span data-stu-id="275b6-334">This setting is used when the app is run with the Visual Studio Debugger and from a command prompt with `dotnet run`.</span></span>

  ```json
  "commandLineArgs": "--contentroot=/content-root-path"
  ```

* <span data-ttu-id="275b6-335">In Visual Studio specificare l'argomento in **Proprietà**  >  **debug**  >  **argomenti dell'applicazione**.</span><span class="sxs-lookup"><span data-stu-id="275b6-335">In Visual Studio, specify the argument in **Properties** > **Debug** > **Application arguments**.</span></span> <span data-ttu-id="275b6-336">L'impostazione dell'argomento nella pagina delle proprietà di Visual Studio consente di aggiungere l'argomento al `launchSettings.json` file.</span><span class="sxs-lookup"><span data-stu-id="275b6-336">Setting the argument in the Visual Studio property page adds the argument to the `launchSettings.json` file.</span></span>

  ```console
  --contentroot=/content-root-path
  ```

### <a name="path-base"></a><span data-ttu-id="275b6-337">Base del percorso</span><span class="sxs-lookup"><span data-stu-id="275b6-337">Path base</span></span>

<span data-ttu-id="275b6-338">L' `--pathbase` argomento imposta il percorso di base dell'app per un'app eseguita localmente con un percorso URL relativo non radice (il `<base>` tag `href` è impostato su un percorso diverso da `/` per la gestione temporanea e la produzione).</span><span class="sxs-lookup"><span data-stu-id="275b6-338">The `--pathbase` argument sets the app base path for an app run locally with a non-root relative URL path (the `<base>` tag `href` is set to a path other than `/` for staging and production).</span></span> <span data-ttu-id="275b6-339">Negli esempi seguenti `/relative-URL-path` è la base del percorso dell'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-339">In the following examples, `/relative-URL-path` is the app's path base.</span></span> <span data-ttu-id="275b6-340">Per altre informazioni, vedere [percorso di base dell'app](xref:blazor/host-and-deploy/index#app-base-path).</span><span class="sxs-lookup"><span data-stu-id="275b6-340">For more information, see [App base path](xref:blazor/host-and-deploy/index#app-base-path).</span></span>

> [!IMPORTANT]
> <span data-ttu-id="275b6-341">A differenza del percorso specificato per `href` del tag `<base>`, non includere una barra finale (`/`) quando si passa il valore dell'argomento `--pathbase`.</span><span class="sxs-lookup"><span data-stu-id="275b6-341">Unlike the path provided to `href` of the `<base>` tag, don't include a trailing slash (`/`) when passing the `--pathbase` argument value.</span></span> <span data-ttu-id="275b6-342">Se il percorso di base dell'app non viene specificato nel tag `<base>` come `<base href="/CoolApp/">` (include una barra finale), passare il valore dell'argomento della riga di comando come `--pathbase=/CoolApp` (senza barra finale).</span><span class="sxs-lookup"><span data-stu-id="275b6-342">If the app base path is provided in the `<base>` tag as `<base href="/CoolApp/">` (includes a trailing slash), pass the command-line argument value as `--pathbase=/CoolApp` (no trailing slash).</span></span>

* <span data-ttu-id="275b6-343">Passare l'argomento quando si esegue localmente l'app a un prompt dei comandi.</span><span class="sxs-lookup"><span data-stu-id="275b6-343">Pass the argument when running the app locally at a command prompt.</span></span> <span data-ttu-id="275b6-344">Dalla directory dell'app, eseguire:</span><span class="sxs-lookup"><span data-stu-id="275b6-344">From the app's directory, execute:</span></span>

  ```dotnetcli
  dotnet run --pathbase=/relative-URL-path
  ```

* <span data-ttu-id="275b6-345">Aggiungere una voce al file dell'app `launchSettings.json` nel profilo di **IIS Express** .</span><span class="sxs-lookup"><span data-stu-id="275b6-345">Add an entry to the app's `launchSettings.json` file in the **IIS Express** profile.</span></span> <span data-ttu-id="275b6-346">Questa impostazione viene usata quando l'app viene eseguita con il debugger di Visual Studio e da un prompt dei comandi con `dotnet run`.</span><span class="sxs-lookup"><span data-stu-id="275b6-346">This setting is used when running the app with the Visual Studio Debugger and from a command prompt with `dotnet run`.</span></span>

  ```json
  "commandLineArgs": "--pathbase=/relative-URL-path"
  ```

* <span data-ttu-id="275b6-347">In Visual Studio specificare l'argomento in **Proprietà**  >  **debug**  >  **argomenti dell'applicazione**.</span><span class="sxs-lookup"><span data-stu-id="275b6-347">In Visual Studio, specify the argument in **Properties** > **Debug** > **Application arguments**.</span></span> <span data-ttu-id="275b6-348">L'impostazione dell'argomento nella pagina delle proprietà di Visual Studio consente di aggiungere l'argomento al `launchSettings.json` file.</span><span class="sxs-lookup"><span data-stu-id="275b6-348">Setting the argument in the Visual Studio property page adds the argument to the `launchSettings.json` file.</span></span>

  ```console
  --pathbase=/relative-URL-path
  ```

### <a name="urls"></a><span data-ttu-id="275b6-349">URL</span><span class="sxs-lookup"><span data-stu-id="275b6-349">URLs</span></span>

<span data-ttu-id="275b6-350">L'argomento `--urls` imposta gli indirizzi IP o gli indirizzi host con le porte e i protocolli su cui eseguire l'ascolto per le richieste.</span><span class="sxs-lookup"><span data-stu-id="275b6-350">The `--urls` argument sets the IP addresses or host addresses with ports and protocols to listen on for requests.</span></span>

* <span data-ttu-id="275b6-351">Passare l'argomento quando si esegue localmente l'app a un prompt dei comandi.</span><span class="sxs-lookup"><span data-stu-id="275b6-351">Pass the argument when running the app locally at a command prompt.</span></span> <span data-ttu-id="275b6-352">Dalla directory dell'app, eseguire:</span><span class="sxs-lookup"><span data-stu-id="275b6-352">From the app's directory, execute:</span></span>

  ```dotnetcli
  dotnet run --urls=http://127.0.0.1:0
  ```

* <span data-ttu-id="275b6-353">Aggiungere una voce al file dell'app `launchSettings.json` nel profilo di **IIS Express** .</span><span class="sxs-lookup"><span data-stu-id="275b6-353">Add an entry to the app's `launchSettings.json` file in the **IIS Express** profile.</span></span> <span data-ttu-id="275b6-354">Questa impostazione viene usata quando l'app viene eseguita con il debugger di Visual Studio e da un prompt dei comandi con `dotnet run`.</span><span class="sxs-lookup"><span data-stu-id="275b6-354">This setting is used when running the app with the Visual Studio Debugger and from a command prompt with `dotnet run`.</span></span>

  ```json
  "commandLineArgs": "--urls=http://127.0.0.1:0"
  ```

* <span data-ttu-id="275b6-355">In Visual Studio specificare l'argomento in **Proprietà**  >  **debug**  >  **argomenti dell'applicazione**.</span><span class="sxs-lookup"><span data-stu-id="275b6-355">In Visual Studio, specify the argument in **Properties** > **Debug** > **Application arguments**.</span></span> <span data-ttu-id="275b6-356">L'impostazione dell'argomento nella pagina delle proprietà di Visual Studio consente di aggiungere l'argomento al `launchSettings.json` file.</span><span class="sxs-lookup"><span data-stu-id="275b6-356">Setting the argument in the Visual Studio property page adds the argument to the `launchSettings.json` file.</span></span>

  ```console
  --urls=http://127.0.0.1:0
  ```

::: moniker range=">= aspnetcore-5.0"

## <a name="configure-the-trimmer"></a><span data-ttu-id="275b6-357">Configurare il trimmer</span><span class="sxs-lookup"><span data-stu-id="275b6-357">Configure the Trimmer</span></span>

<span data-ttu-id="275b6-358">Blazor esegue il trimming del linguaggio intermedio (IL) in ogni build di rilascio per rimuovere IL linguaggio intermedio non necessario dagli assembly di output.</span><span class="sxs-lookup"><span data-stu-id="275b6-358">Blazor performs Intermediate Language (IL) trimming on each Release build to remove unnecessary IL from the output assemblies.</span></span> <span data-ttu-id="275b6-359">Per altre informazioni, vedere <xref:blazor/host-and-deploy/configure-trimmer>.</span><span class="sxs-lookup"><span data-stu-id="275b6-359">For more information, see <xref:blazor/host-and-deploy/configure-trimmer>.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="configure-the-linker"></a><span data-ttu-id="275b6-360">Configurare il linker</span><span class="sxs-lookup"><span data-stu-id="275b6-360">Configure the Linker</span></span>

<span data-ttu-id="275b6-361">Blazor esegue il collegamento Intermediate Language (IL) a ogni build di rilascio per rimuovere IL linguaggio intermedio non necessario dagli assembly di output.</span><span class="sxs-lookup"><span data-stu-id="275b6-361">Blazor performs Intermediate Language (IL) linking on each Release build to remove unnecessary IL from the output assemblies.</span></span> <span data-ttu-id="275b6-362">Per altre informazioni, vedere <xref:blazor/host-and-deploy/configure-linker>.</span><span class="sxs-lookup"><span data-stu-id="275b6-362">For more information, see <xref:blazor/host-and-deploy/configure-linker>.</span></span>

::: moniker-end

## <a name="custom-boot-resource-loading"></a><span data-ttu-id="275b6-363">Caricamento di risorse di avvio personalizzate</span><span class="sxs-lookup"><span data-stu-id="275b6-363">Custom boot resource loading</span></span>

<span data-ttu-id="275b6-364">Un' Blazor WebAssembly app può essere inizializzata con la `loadBootResource` funzione per eseguire l'override del meccanismo di caricamento delle risorse di avvio predefinito.</span><span class="sxs-lookup"><span data-stu-id="275b6-364">A Blazor WebAssembly app can be initialized with the `loadBootResource` function to override the built-in boot resource loading mechanism.</span></span> <span data-ttu-id="275b6-365">Utilizzare `loadBootResource` per gli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-365">Use `loadBootResource` for the following scenarios:</span></span>

* <span data-ttu-id="275b6-366">Consente agli utenti di caricare risorse statiche, ad esempio dati del fuso orario o `dotnet.wasm` da una rete CDN.</span><span class="sxs-lookup"><span data-stu-id="275b6-366">Allow users to load static resources, such as timezone data or `dotnet.wasm` from a CDN.</span></span>
* <span data-ttu-id="275b6-367">Caricare gli assembly compressi usando una richiesta HTTP e decomprimerli sul client per gli host che non supportano il recupero di contenuti compressi dal server.</span><span class="sxs-lookup"><span data-stu-id="275b6-367">Load compressed assemblies using an HTTP request and decompress them on the client for hosts that don't support fetching compressed contents from the server.</span></span>
* <span data-ttu-id="275b6-368">Eseguire l'alias delle risorse con un nome diverso reindirizzando ogni `fetch` richiesta a un nuovo nome.</span><span class="sxs-lookup"><span data-stu-id="275b6-368">Alias resources to a different name by redirecting each `fetch` request to a new name.</span></span>

<span data-ttu-id="275b6-369">`loadBootResource` i parametri vengono visualizzati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="275b6-369">`loadBootResource` parameters appear in the following table.</span></span>

| <span data-ttu-id="275b6-370">Parametro</span><span class="sxs-lookup"><span data-stu-id="275b6-370">Parameter</span></span>    | <span data-ttu-id="275b6-371">Descrizione</span><span class="sxs-lookup"><span data-stu-id="275b6-371">Description</span></span> |
| ------------ | ----------- |
| `type`       | <span data-ttu-id="275b6-372">Tipo di risorsa.</span><span class="sxs-lookup"><span data-stu-id="275b6-372">The type of the resource.</span></span> <span data-ttu-id="275b6-373">Tipi di consentiti: `assembly` , `pdb` , `dotnetjs` , `dotnetwasm` , `timezonedata`</span><span class="sxs-lookup"><span data-stu-id="275b6-373">Permissable types: `assembly`, `pdb`, `dotnetjs`, `dotnetwasm`, `timezonedata`</span></span> |
| `name`       | <span data-ttu-id="275b6-374">Nome della risorsa.</span><span class="sxs-lookup"><span data-stu-id="275b6-374">The name of the resource.</span></span> |
| `defaultUri` | <span data-ttu-id="275b6-375">URI relativo o assoluto della risorsa.</span><span class="sxs-lookup"><span data-stu-id="275b6-375">The relative or absolute URI of the resource.</span></span> |
| `integrity`  | <span data-ttu-id="275b6-376">Stringa di integrità che rappresenta il contenuto previsto nella risposta.</span><span class="sxs-lookup"><span data-stu-id="275b6-376">The integrity string representing the expected content in the response.</span></span> |

<span data-ttu-id="275b6-377">`loadBootResource` restituisce uno dei seguenti elementi per eseguire l'override del processo di caricamento:</span><span class="sxs-lookup"><span data-stu-id="275b6-377">`loadBootResource` returns any of the following to override the loading process:</span></span>

* <span data-ttu-id="275b6-378">Stringa URI.</span><span class="sxs-lookup"><span data-stu-id="275b6-378">URI string.</span></span> <span data-ttu-id="275b6-379">Nell'esempio seguente ( `wwwroot/index.html` ), i file seguenti vengono serviti da una rete CDN all'indirizzo `https://my-awesome-cdn.com/` :</span><span class="sxs-lookup"><span data-stu-id="275b6-379">In the following example (`wwwroot/index.html`), the following files are served from a CDN at `https://my-awesome-cdn.com/`:</span></span>

  * `dotnet.*.js`
  * `dotnet.wasm`
  * <span data-ttu-id="275b6-380">Dati TimeZone</span><span class="sxs-lookup"><span data-stu-id="275b6-380">Timezone data</span></span>

  ```html
  ...

  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        console.log(`Loading: '${type}', '${name}', '${defaultUri}', '${integrity}'`);
        switch (type) {
          case 'dotnetjs':
          case 'dotnetwasm':
          case 'timezonedata':
            return `https://my-awesome-cdn.com/blazorwebassembly/3.2.0/${name}`;
        }
      }
    });
  </script>
  ```

* <span data-ttu-id="275b6-381">`Promise<Response>`.</span><span class="sxs-lookup"><span data-stu-id="275b6-381">`Promise<Response>`.</span></span> <span data-ttu-id="275b6-382">Passare il `integrity` parametro in un'intestazione per mantenere il comportamento predefinito di controllo dell'integrità.</span><span class="sxs-lookup"><span data-stu-id="275b6-382">Pass the `integrity` parameter in a header to retain the default integrity-checking behavior.</span></span>

  <span data-ttu-id="275b6-383">L'esempio seguente ( `wwwroot/index.html` ) aggiunge un'intestazione HTTP personalizzata alle richieste in uscita e passa il `integrity` parametro tramite alla `fetch` chiamata:</span><span class="sxs-lookup"><span data-stu-id="275b6-383">The following example (`wwwroot/index.html`) adds a custom HTTP header to the outbound requests and passes the `integrity` parameter through to the `fetch` call:</span></span>
  
  ```html
  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        return fetch(defaultUri, { 
          cache: 'no-cache',
          integrity: integrity,
          headers: { 'MyCustomHeader': 'My custom value' }
        });
      }
    });
  </script>
  ```

* <span data-ttu-id="275b6-384">`null`/`undefined`, che comporta il comportamento di caricamento predefinito.</span><span class="sxs-lookup"><span data-stu-id="275b6-384">`null`/`undefined`, which results in the default loading behavior.</span></span>

<span data-ttu-id="275b6-385">Per consentire il caricamento delle risorse tra le origini, le origini esterne devono restituire le intestazioni CORS necessarie per i browser.</span><span class="sxs-lookup"><span data-stu-id="275b6-385">External sources must return the required CORS headers for browsers to allow the cross-origin resource loading.</span></span> <span data-ttu-id="275b6-386">Per impostazione predefinita, CDNs fornisce in genere le intestazioni obbligatorie.</span><span class="sxs-lookup"><span data-stu-id="275b6-386">CDNs usually provide the required headers by default.</span></span>

<span data-ttu-id="275b6-387">È sufficiente specificare i tipi per i comportamenti personalizzati.</span><span class="sxs-lookup"><span data-stu-id="275b6-387">You only need to specify types for custom behaviors.</span></span> <span data-ttu-id="275b6-388">I tipi non specificati in `loadBootResource` vengono caricati dal Framework in base ai relativi comportamenti di caricamento predefiniti.</span><span class="sxs-lookup"><span data-stu-id="275b6-388">Types not specified to `loadBootResource` are loaded by the framework per their default loading behaviors.</span></span>

## <a name="change-the-filename-extension-of-dll-files"></a><span data-ttu-id="275b6-389">Modificare l'estensione di file DLL</span><span class="sxs-lookup"><span data-stu-id="275b6-389">Change the filename extension of DLL files</span></span>

<span data-ttu-id="275b6-390">Se è necessario modificare le estensioni dei file pubblicati dell'app `.dll` , seguire le istruzioni riportate in questa sezione.</span><span class="sxs-lookup"><span data-stu-id="275b6-390">In case you have a need to change the filename extensions of the app's published `.dll` files, follow the guidance in this section.</span></span>

<span data-ttu-id="275b6-391">Dopo la pubblicazione dell'app, usare uno script della shell o una pipeline di compilazione DevOps per rinominare `.dll` i file in modo da usare un'estensione di file diversa.</span><span class="sxs-lookup"><span data-stu-id="275b6-391">After publishing the app, use a shell script or DevOps build pipeline to rename `.dll` files to use a different file extension.</span></span> <span data-ttu-id="275b6-392">Specificare come destinazione i `.dll` file nella `wwwroot` directory dell'output pubblicato dell'app (ad esempio, `{CONTENT ROOT}/bin/Release/netstandard2.1/publish/wwwroot` ).</span><span class="sxs-lookup"><span data-stu-id="275b6-392">Target the `.dll` files in the `wwwroot` directory of the app's published output (for example, `{CONTENT ROOT}/bin/Release/netstandard2.1/publish/wwwroot`).</span></span>

<span data-ttu-id="275b6-393">Negli esempi seguenti, `.dll` i file vengono rinominati per l'utilizzo dell' `.bin` estensione di file.</span><span class="sxs-lookup"><span data-stu-id="275b6-393">In the following examples, `.dll` files are renamed to use the `.bin` file extension.</span></span>

<span data-ttu-id="275b6-394">In Windows:</span><span class="sxs-lookup"><span data-stu-id="275b6-394">On Windows:</span></span>

```powershell
dir .\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content .\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content .\_framework\blazor.boot.json
```

<span data-ttu-id="275b6-395">Se gli asset di lavoro del servizio sono anche in uso, aggiungere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="275b6-395">If service worker assets are also in use, add the following command:</span></span>

```powershell
((Get-Content .\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content .\service-worker-assets.js
```

<span data-ttu-id="275b6-396">In Linux o macOS:</span><span class="sxs-lookup"><span data-stu-id="275b6-396">On Linux or macOS:</span></span>

```console
for f in _framework/_bin/*; do mv "$f" "`echo $f | sed -e 's/\.dll/.bin/g'`"; done
sed -i 's/\.dll"/.bin"/g' _framework/blazor.boot.json
```

<span data-ttu-id="275b6-397">Se gli asset di lavoro del servizio sono anche in uso, aggiungere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="275b6-397">If service worker assets are also in use, add the following command:</span></span>

```console
sed -i 's/\.dll"/.bin"/g' service-worker-assets.js
```
   
<span data-ttu-id="275b6-398">Per usare un'estensione di file diversa da `.bin` , sostituire `.bin` nei comandi precedenti.</span><span class="sxs-lookup"><span data-stu-id="275b6-398">To use a different file extension than `.bin`, replace `.bin` in the preceding commands.</span></span>

<span data-ttu-id="275b6-399">Per risolvere i `blazor.boot.json.gz` file compressi e `blazor.boot.json.br` , adottare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-399">To address the compressed `blazor.boot.json.gz` and `blazor.boot.json.br` files, adopt either of the following approaches:</span></span>

* <span data-ttu-id="275b6-400">Rimuovere i `blazor.boot.json.gz` file compressi e `blazor.boot.json.br` .</span><span class="sxs-lookup"><span data-stu-id="275b6-400">Remove the compressed `blazor.boot.json.gz` and `blazor.boot.json.br` files.</span></span> <span data-ttu-id="275b6-401">La compressione è disabilitata con questo approccio.</span><span class="sxs-lookup"><span data-stu-id="275b6-401">Compression is disabled with this approach.</span></span>
* <span data-ttu-id="275b6-402">Ricomprimere il `blazor.boot.json` file aggiornato.</span><span class="sxs-lookup"><span data-stu-id="275b6-402">Recompress the updated `blazor.boot.json` file.</span></span>

<span data-ttu-id="275b6-403">Le linee guida precedenti si applicano anche quando le risorse di lavoro del servizio sono in uso.</span><span class="sxs-lookup"><span data-stu-id="275b6-403">The preceding guidance also applies when service worker assets are in use.</span></span> <span data-ttu-id="275b6-404">Rimuovere o ricomprimere `wwwroot/service-worker-assets.js.br` e `wwwroot/service-worker-assets.js.gz` .</span><span class="sxs-lookup"><span data-stu-id="275b6-404">Remove or recompress `wwwroot/service-worker-assets.js.br` and `wwwroot/service-worker-assets.js.gz`.</span></span> <span data-ttu-id="275b6-405">In caso contrario, i controlli di integrità dei file avranno esito negativo nel browser.</span><span class="sxs-lookup"><span data-stu-id="275b6-405">Otherwise, file integrity checks fail in the browser.</span></span>

<span data-ttu-id="275b6-406">Nell'esempio di Windows seguente viene usato uno script di PowerShell inserito nella radice del progetto.</span><span class="sxs-lookup"><span data-stu-id="275b6-406">The following Windows example uses a PowerShell script placed at the root of the project.</span></span>

<span data-ttu-id="275b6-407">`ChangeDLLExtensions.ps1:`:</span><span class="sxs-lookup"><span data-stu-id="275b6-407">`ChangeDLLExtensions.ps1:`:</span></span>

```powershell
param([string]$filepath,[string]$tfm)
dir $filepath\bin\Release\$tfm\wwwroot\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json
Remove-Item $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json.gz
```

<span data-ttu-id="275b6-408">Se gli asset di lavoro del servizio sono anche in uso, aggiungere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="275b6-408">If service worker assets are also in use, add the following command:</span></span>

```powershell
((Get-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js
```

<span data-ttu-id="275b6-409">Nel file di progetto, lo script viene eseguito dopo la pubblicazione dell'app:</span><span class="sxs-lookup"><span data-stu-id="275b6-409">In the project file, the script is run after publishing the app:</span></span>

```xml
<Target Name="ChangeDLLFileExtensions" AfterTargets="Publish" Condition="'$(Configuration)'=='Release'">
  <Exec Command="powershell.exe -command &quot;&amp; { .\ChangeDLLExtensions.ps1 '$(SolutionDir)' '$(TargetFramework)'}&quot;" />
</Target>
```

> [!NOTE]
> <span data-ttu-id="275b6-410">Quando si rinominano e si caricano in modalità lazy gli stessi assembly, vedere le istruzioni in <xref:blazor/webassembly-lazy-load-assemblies#onnavigateasync-events-and-renamed-assembly-files> .</span><span class="sxs-lookup"><span data-stu-id="275b6-410">When renaming and lazy loading the same assemblies, see the guidance in <xref:blazor/webassembly-lazy-load-assemblies#onnavigateasync-events-and-renamed-assembly-files>.</span></span>

## <a name="resolve-integrity-check-failures"></a><span data-ttu-id="275b6-411">Risolvere gli errori di controllo dell'integrità</span><span class="sxs-lookup"><span data-stu-id="275b6-411">Resolve integrity check failures</span></span>

<span data-ttu-id="275b6-412">Quando Blazor WebAssembly Scarica i file di avvio di un'app, indica al browser di eseguire controlli di integrità sulle risposte.</span><span class="sxs-lookup"><span data-stu-id="275b6-412">When Blazor WebAssembly downloads an app's startup files, it instructs the browser to perform integrity checks on the responses.</span></span> <span data-ttu-id="275b6-413">Usa le informazioni nel `blazor.boot.json` file per specificare i valori hash SHA-256 previsti per `.dll` , `.wasm` e altri file.</span><span class="sxs-lookup"><span data-stu-id="275b6-413">It uses information in the `blazor.boot.json` file to specify the expected SHA-256 hash values for `.dll`, `.wasm`, and other files.</span></span> <span data-ttu-id="275b6-414">Questa operazione è utile per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="275b6-414">This is beneficial for the following reasons:</span></span>

* <span data-ttu-id="275b6-415">Garantisce che non si rischi il caricamento di un set di file incoerente, ad esempio se viene applicata una nuova distribuzione al server Web mentre è in corso il download dei file dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="275b6-415">It ensures you don't risk loading an inconsistent set of files, for example if a new deployment is applied to your web server while the user is in the process of downloading the application files.</span></span> <span data-ttu-id="275b6-416">I file incoerenti possono causare un comportamento indefinito.</span><span class="sxs-lookup"><span data-stu-id="275b6-416">Inconsistent files could lead to undefined behavior.</span></span>
* <span data-ttu-id="275b6-417">Garantisce che il browser dell'utente non memorizza mai nella cache le risposte incoerenti o non valide, che potrebbero impedire l'avvio dell'app anche se aggiornano manualmente la pagina.</span><span class="sxs-lookup"><span data-stu-id="275b6-417">It ensures the user's browser never caches inconsistent or invalid responses, which could prevent them from starting the app even if they manually refresh the page.</span></span>
* <span data-ttu-id="275b6-418">Consente di memorizzare nella cache le risposte in modo sicuro e non di controllare le modifiche sul lato server fino a quando non si modificano gli hash SHA-256 previsti, quindi i carichi di pagina successivi coinvolgono un minor numero di richieste e vengono completate molto più velocemente.</span><span class="sxs-lookup"><span data-stu-id="275b6-418">It makes it safe to cache the responses and not even check for server-side changes until the expected SHA-256 hashes themselves change, so subsequent page loads involve fewer requests and complete much faster.</span></span>

<span data-ttu-id="275b6-419">Se il server Web restituisce risposte non corrispondenti agli hash SHA-256 previsti, nella console per sviluppatori del browser verrà visualizzato un errore simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="275b6-419">If your web server returns responses that don't match the expected SHA-256 hashes, you will see an error similar to the following appear in the browser's developer console:</span></span>

> <span data-ttu-id="275b6-420">Non è stato possibile trovare un digest valido nell'attributo ' Integrity ' per la risorsa ' https://myapp.example.com/\_framework/My BlazorApp.dll' con integrità di SHA-256 calcolata ' IIa70iwvmEg5WiDV17OpQ5eCztNYqL186J56852RpJY ='.</span><span class="sxs-lookup"><span data-stu-id="275b6-420">Failed to find a valid digest in the 'integrity' attribute for resource 'https://myapp.example.com/\_framework/MyBlazorApp.dll' with computed SHA-256 integrity 'IIa70iwvmEg5WiDV17OpQ5eCztNYqL186J56852RpJY='.</span></span> <span data-ttu-id="275b6-421">La risorsa è stata bloccata.</span><span class="sxs-lookup"><span data-stu-id="275b6-421">The resource has been blocked.</span></span>

<span data-ttu-id="275b6-422">Nella maggior parte dei casi, *non* si tratta di un problema con il controllo dell'integrità.</span><span class="sxs-lookup"><span data-stu-id="275b6-422">In most cases, this is *not* a problem with integrity checking itself.</span></span> <span data-ttu-id="275b6-423">Al contrario, significa che è presente un altro problema e che il controllo di integrità è in grado di avvisare dell'altro problema.</span><span class="sxs-lookup"><span data-stu-id="275b6-423">Instead, it means there is some other problem, and the integrity check is warning you about that other problem.</span></span>

### <a name="diagnosing-integrity-problems"></a><span data-ttu-id="275b6-424">Diagnosi dei problemi di integrità</span><span class="sxs-lookup"><span data-stu-id="275b6-424">Diagnosing integrity problems</span></span>

<span data-ttu-id="275b6-425">Quando viene compilata un'app, il `blazor.boot.json` manifesto generato descrive gli hash SHA-256 delle risorse di avvio (ad esempio,, `.dll` `.wasm` e altri file) nel momento in cui viene generato l'output di compilazione.</span><span class="sxs-lookup"><span data-stu-id="275b6-425">When an app is built, the generated `blazor.boot.json` manifest describes the SHA-256 hashes of your boot resources (for example, `.dll`, `.wasm`, and other files) at the time that the build output is produced.</span></span> <span data-ttu-id="275b6-426">Il controllo di integrità passa a condizione che gli hash SHA-256 `blazor.boot.json` corrispondano ai file recapitati al browser.</span><span class="sxs-lookup"><span data-stu-id="275b6-426">The integrity check passes as long as the SHA-256 hashes in `blazor.boot.json` match the files delivered to the browser.</span></span>

<span data-ttu-id="275b6-427">I motivi più comuni per cui questo errore sono:</span><span class="sxs-lookup"><span data-stu-id="275b6-427">Common reasons why this fails are:</span></span>

 * <span data-ttu-id="275b6-428">La risposta del server Web è un errore (ad esempio, *404-non trovato* o un *errore del server interno 500*) invece del file richiesto dal browser.</span><span class="sxs-lookup"><span data-stu-id="275b6-428">The web server's response is an error (for example, a *404 - Not Found* or a *500 - Internal Server Error*) instead of the file the browser requested.</span></span> <span data-ttu-id="275b6-429">Viene segnalato dal browser come un errore di controllo dell'integrità e non come un errore di risposta.</span><span class="sxs-lookup"><span data-stu-id="275b6-429">This is reported by the browser as an integrity check failure and not as a response failure.</span></span>
 * <span data-ttu-id="275b6-430">Un elemento ha modificato il contenuto dei file tra la compilazione e il recapito dei file nel browser.</span><span class="sxs-lookup"><span data-stu-id="275b6-430">Something has changed the contents of the files between the build and delivery of the files to the browser.</span></span> <span data-ttu-id="275b6-431">Questo problema può verificarsi:</span><span class="sxs-lookup"><span data-stu-id="275b6-431">This might happen:</span></span>
   * <span data-ttu-id="275b6-432">Se gli strumenti di compilazione o di compilazione modificano manualmente l'output di compilazione.</span><span class="sxs-lookup"><span data-stu-id="275b6-432">If you or build tools manually modify the build output.</span></span>
   * <span data-ttu-id="275b6-433">Se alcuni aspetti del processo di distribuzione hanno modificato i file.</span><span class="sxs-lookup"><span data-stu-id="275b6-433">If some aspect of the deployment process modified the files.</span></span> <span data-ttu-id="275b6-434">Se, ad esempio, si usa un meccanismo di distribuzione basato su git, tenere presente che git converte in modo trasparente le terminazioni riga di tipo Windows in terminazioni riga di tipo Unix se si esegue il commit di file in Windows e li si estrae in Linux.</span><span class="sxs-lookup"><span data-stu-id="275b6-434">For example if you use a Git-based deployment mechanism, bear in mind that Git transparently converts Windows-style line endings to Unix-style line endings if you commit files on Windows and check them out on Linux.</span></span> <span data-ttu-id="275b6-435">La modifica delle terminazioni di riga del file modifica gli hash SHA-256.</span><span class="sxs-lookup"><span data-stu-id="275b6-435">Changing file line endings change the SHA-256 hashes.</span></span> <span data-ttu-id="275b6-436">Per evitare questo problema, considerare l' [utilizzo `.gitattributes` di per considerare gli artefatti di compilazione come `binary` file](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes).</span><span class="sxs-lookup"><span data-stu-id="275b6-436">To avoid this problem, consider [using `.gitattributes` to treat build artifacts as `binary` files](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes).</span></span>
   * <span data-ttu-id="275b6-437">Il server Web modifica il contenuto del file come parte del servizio.</span><span class="sxs-lookup"><span data-stu-id="275b6-437">The web server modifies the file contents as part of serving them.</span></span> <span data-ttu-id="275b6-438">Ad esempio, alcune reti di distribuzione del contenuto (CDNs) tentano automaticamente di [minimizzare](xref:client-side/bundling-and-minification#minification) HTML, quindi la modifica.</span><span class="sxs-lookup"><span data-stu-id="275b6-438">For example, some content distribution networks (CDNs) automatically attempt to [minify](xref:client-side/bundling-and-minification#minification) HTML, thereby modifying it.</span></span> <span data-ttu-id="275b6-439">Potrebbe essere necessario disabilitare queste funzionalità.</span><span class="sxs-lookup"><span data-stu-id="275b6-439">You may need to disable such features.</span></span>

<span data-ttu-id="275b6-440">Per diagnosticare quali di queste condizioni si applicano nel caso:</span><span class="sxs-lookup"><span data-stu-id="275b6-440">To diagnose which of these applies in your case:</span></span>

 1. <span data-ttu-id="275b6-441">Si noti il file che sta attivando l'errore leggendo il messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="275b6-441">Note which file is triggering the error by reading the error message.</span></span>
 1. <span data-ttu-id="275b6-442">Aprire gli strumenti di sviluppo del browser ed esaminare la scheda *rete* . Se necessario, ricaricare la pagina per visualizzare l'elenco di richieste e risposte.</span><span class="sxs-lookup"><span data-stu-id="275b6-442">Open your browser's developer tools and look in the *Network* tab. If necessary, reload the page to see the list of requests and responses.</span></span> <span data-ttu-id="275b6-443">Individuare il file che sta attivando l'errore in tale elenco.</span><span class="sxs-lookup"><span data-stu-id="275b6-443">Find the file that is triggering the error in that list.</span></span>
 1. <span data-ttu-id="275b6-444">Verificare il codice di stato HTTP nella risposta.</span><span class="sxs-lookup"><span data-stu-id="275b6-444">Check the HTTP status code in the response.</span></span> <span data-ttu-id="275b6-445">Se il server restituisce un valore diverso da *200-OK* (o un altro codice di stato 2xx), è necessario un problema sul lato server da diagnosticare.</span><span class="sxs-lookup"><span data-stu-id="275b6-445">If the server returns anything other than *200 - OK* (or another 2xx status code), then you have a server-side problem to diagnose.</span></span> <span data-ttu-id="275b6-446">Ad esempio, il codice di stato 403 indica che si è verificato un problema di autorizzazione, mentre il codice di stato 500 indica che il server ha esito negativo in modo non specificato.</span><span class="sxs-lookup"><span data-stu-id="275b6-446">For example, status code 403 means there's an authorization problem, whereas status code 500 means the server is failing in an unspecified manner.</span></span> <span data-ttu-id="275b6-447">Consultare log lato server per diagnosticare e correggere l'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-447">Consult server-side logs to diagnose and fix the app.</span></span>
 1. <span data-ttu-id="275b6-448">Se il codice di stato è *200-OK* per la risorsa, esaminare il contenuto della risposta negli strumenti di sviluppo del browser e verificare che il contenuto corrisponda ai dati previsti.</span><span class="sxs-lookup"><span data-stu-id="275b6-448">If the status code is *200 - OK* for the resource, look at the response content in browser's developer tools and check that the content matches up with the data expected.</span></span> <span data-ttu-id="275b6-449">Un problema comune, ad esempio, è quello di configurare correttamente il routing, in modo che le richieste restituiscano i `index.html` dati anche per altri file.</span><span class="sxs-lookup"><span data-stu-id="275b6-449">For example, a common problem is to misconfigure routing so that requests return your `index.html` data even for other files.</span></span> <span data-ttu-id="275b6-450">Assicurarsi che le risposte alle `.wasm` richieste siano file binari webassembly e che le risposte alle `.dll` richieste siano file binari dell'assembly .NET.</span><span class="sxs-lookup"><span data-stu-id="275b6-450">Make sure that responses to `.wasm` requests are WebAssembly binaries and that responses to `.dll` requests are .NET assembly binaries.</span></span> <span data-ttu-id="275b6-451">In caso contrario, si disporrà di un problema di routing lato server da diagnosticare.</span><span class="sxs-lookup"><span data-stu-id="275b6-451">If not, you have a server-side routing problem to diagnose.</span></span>
 1. <span data-ttu-id="275b6-452">Cercare per convalidare l'output pubblicato e distribuito dell'app con lo script di PowerShell per la [risoluzione dei problemi di integrità](#troubleshoot-integrity-powershell-script).</span><span class="sxs-lookup"><span data-stu-id="275b6-452">Seek to validate the app's published and deployed output with the [Troubleshoot integrity PowerShell script](#troubleshoot-integrity-powershell-script).</span></span>

<span data-ttu-id="275b6-453">Se si conferma che il server restituisce dati corretti in modo plausibile, è necessario modificare il contenuto tra la compilazione e il recapito del file.</span><span class="sxs-lookup"><span data-stu-id="275b6-453">If you confirm that the server is returning plausibly correct data, there must be something else modifying the contents in between build and delivery of the file.</span></span> <span data-ttu-id="275b6-454">Per esaminare questa operazione:</span><span class="sxs-lookup"><span data-stu-id="275b6-454">To investigate this:</span></span>

 * <span data-ttu-id="275b6-455">Esaminare il meccanismo di distribuzione e la combinazione di elementi di compilazione nel caso in cui modifichino i file dopo la compilazione dei file.</span><span class="sxs-lookup"><span data-stu-id="275b6-455">Examine the build toolchain and deployment mechanism in case they're modifying files after the files are built.</span></span> <span data-ttu-id="275b6-456">Un esempio è quando git trasforma le terminazioni riga file, come descritto in precedenza.</span><span class="sxs-lookup"><span data-stu-id="275b6-456">An example of this is when Git transforms file line endings, as described earlier.</span></span>
 * <span data-ttu-id="275b6-457">Esaminare la configurazione del server Web o della rete CDN nel caso in cui siano configurati per modificare dinamicamente le risposte, ad esempio provando a minimizzare HTML.</span><span class="sxs-lookup"><span data-stu-id="275b6-457">Examine the web server or CDN configuration in case they're set up to modify responses dynamically (for example, trying to minify HTML).</span></span> <span data-ttu-id="275b6-458">Il server Web implementa la compressione HTTP (ad esempio, restituendo `content-encoding: br` o `content-encoding: gzip` ), perché questo non influisce sul risultato dopo la decompressione.</span><span class="sxs-lookup"><span data-stu-id="275b6-458">It's fine for the web server to implement HTTP compression (for example, returning `content-encoding: br` or `content-encoding: gzip`), since this doesn't affect the result after decompression.</span></span> <span data-ttu-id="275b6-459">Tuttavia, non è corretto per il server Web modificare i dati *non* compressi.</span><span class="sxs-lookup"><span data-stu-id="275b6-459">However, it's *not* fine for the web server to modify the uncompressed data.</span></span>

### <a name="troubleshoot-integrity-powershell-script"></a><span data-ttu-id="275b6-460">Risolvere i problemi dello script di PowerShell di integrità</span><span class="sxs-lookup"><span data-stu-id="275b6-460">Troubleshoot integrity PowerShell script</span></span>

<span data-ttu-id="275b6-461">Usare lo [`integrity.ps1`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/integrity.ps1?raw=true) script di PowerShell per convalidare un'app pubblicata e distribuita Blazor .</span><span class="sxs-lookup"><span data-stu-id="275b6-461">Use the [`integrity.ps1`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/integrity.ps1?raw=true) PowerShell script to validate a published and deployed Blazor app.</span></span> <span data-ttu-id="275b6-462">Lo script viene fornito come punto di partenza quando l'app presenta problemi di integrità che il Blazor Framework non è in grado di identificare.</span><span class="sxs-lookup"><span data-stu-id="275b6-462">The script is provided as a starting point when the app has integrity issues that the Blazor framework can't identify.</span></span> <span data-ttu-id="275b6-463">Potrebbe essere necessario personalizzare lo script per le app.</span><span class="sxs-lookup"><span data-stu-id="275b6-463">Customization of the script might be required for your apps.</span></span>

<span data-ttu-id="275b6-464">Lo script controlla i file nella `publish` cartella e scaricati dall'app distribuita per rilevare i problemi nei diversi manifesti che contengono hash di integrità.</span><span class="sxs-lookup"><span data-stu-id="275b6-464">The script checks the files in the `publish` folder and downloaded from the deployed app to detect issues in the different manifests that contain integrity hashes.</span></span> <span data-ttu-id="275b6-465">Questi controlli dovrebbero rilevare i problemi più comuni:</span><span class="sxs-lookup"><span data-stu-id="275b6-465">These checks should detect the most common problems:</span></span>

* <span data-ttu-id="275b6-466">È stato modificato un file nell'output pubblicato senza rendersene conto.</span><span class="sxs-lookup"><span data-stu-id="275b6-466">You modified a file in the published output without realizing it.</span></span>
* <span data-ttu-id="275b6-467">L'app non è stata distribuita correttamente nella destinazione di distribuzione o è stata modificata nell'ambiente della destinazione di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-467">The app wasn't correctly deployed to the deployment target, or something changed within the deployment target's environment.</span></span>
* <span data-ttu-id="275b6-468">Esistono differenze tra l'app distribuita e l'output della pubblicazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="275b6-468">There are differences between the deployed app and the output from publishing the app.</span></span>

<span data-ttu-id="275b6-469">Richiamare lo script con il comando seguente in una shell dei comandi di PowerShell:</span><span class="sxs-lookup"><span data-stu-id="275b6-469">Invoke the script with the following command in a PowerShell command shell:</span></span>

```powershell
.\integrity.ps1 {BASE URL} {PUBLISH OUTPUT FOLDER}
```

<span data-ttu-id="275b6-470">Segnaposto</span><span class="sxs-lookup"><span data-stu-id="275b6-470">Placeholders:</span></span>

* <span data-ttu-id="275b6-471">`{BASE URL}`: URL dell'app distribuita.</span><span class="sxs-lookup"><span data-stu-id="275b6-471">`{BASE URL}`: The URL of the deployed app.</span></span>
* <span data-ttu-id="275b6-472">`{PUBLISH OUTPUT FOLDER}`: Percorso della cartella o della posizione dell'app in `publish` cui è pubblicata l'app per la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-472">`{PUBLISH OUTPUT FOLDER}`: The path to the app's `publish` folder or location where the app is published for deployment.</span></span>

> [!NOTE]
> <span data-ttu-id="275b6-473">Per clonare il `dotnet/AspNetCore.Docs` repository GitHub in un sistema che usa [BitDefender](https://www.bitdefender.com) virus scanner, aggiungere un'eccezione a BitDefender per lo `integrity.ps1` script.</span><span class="sxs-lookup"><span data-stu-id="275b6-473">To clone the `dotnet/AspNetCore.Docs` GitHub repository to a system that uses the [Bitdefender](https://www.bitdefender.com) virus scanner, add an exception to Bitdefender for the `integrity.ps1` script.</span></span> <span data-ttu-id="275b6-474">Aggiungere l'eccezione a BitDefender prima di clonare il repository per evitare che lo script venga messo in quarantena dal programma antivirus.</span><span class="sxs-lookup"><span data-stu-id="275b6-474">Add the exception to Bitdefender before cloning the repo to avoid having the script quarantined by the virus scanner.</span></span> <span data-ttu-id="275b6-475">L'esempio seguente è un percorso tipico dello script per il repository clonato in un sistema Windows.</span><span class="sxs-lookup"><span data-stu-id="275b6-475">The following example is a typical path to the script for the cloned repo on a Windows system.</span></span> <span data-ttu-id="275b6-476">Modificare il percorso in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="275b6-476">Adjust the path as needed.</span></span> <span data-ttu-id="275b6-477">Il segnaposto `{USER}` è il segmento del percorso dell'utente.</span><span class="sxs-lookup"><span data-stu-id="275b6-477">The placeholder `{USER}` is the user's path segment.</span></span>
>
> ```
> C:\Users\{USER}\Documents\GitHub\AspNetCore.Docs\aspnetcore\blazor\host-and-deploy\webassembly\_samples\integrity.ps1
> ```

### <a name="disable-integrity-checking-for-non-pwa-apps"></a><span data-ttu-id="275b6-478">Disabilitare il controllo dell'integrità per le app non PWA</span><span class="sxs-lookup"><span data-stu-id="275b6-478">Disable integrity checking for non-PWA apps</span></span>

<span data-ttu-id="275b6-479">Nella maggior parte dei casi, non disabilitare il controllo dell'integrità.</span><span class="sxs-lookup"><span data-stu-id="275b6-479">In most cases, don't disable integrity checking.</span></span> <span data-ttu-id="275b6-480">La disabilitazione del controllo di integrità non risolve il problema sottostante che ha causato le risposte impreviste e comporta la perdita dei vantaggi elencati in precedenza.</span><span class="sxs-lookup"><span data-stu-id="275b6-480">Disabling integrity checking doesn't solve the underlying problem that has caused the unexpected responses and results in losing the benefits listed earlier.</span></span>

<span data-ttu-id="275b6-481">In alcuni casi non è possibile fare affidamento sul server Web per restituire risposte coerenti e non è possibile scegliere di disabilitare i controlli di integrità.</span><span class="sxs-lookup"><span data-stu-id="275b6-481">There may be cases where the web server can't be relied upon to return consistent responses, and you have no choice but to disable integrity checks.</span></span> <span data-ttu-id="275b6-482">Per disabilitare i controlli di integrità, aggiungere il codice seguente a un gruppo di proprietà nel Blazor WebAssembly file del progetto `.csproj` :</span><span class="sxs-lookup"><span data-stu-id="275b6-482">To disable integrity checks, add the following to a property group in the Blazor WebAssembly project's `.csproj` file:</span></span>

```xml
<BlazorCacheBootResources>false</BlazorCacheBootResources>
```

<span data-ttu-id="275b6-483">`BlazorCacheBootResources` Disabilita inoltre il Blazor comportamento predefinito di memorizzazione nella cache di `.dll` , `.wasm` e altri file basati sui rispettivi hash SHA-256, perché la proprietà indica che non è possibile fare affidamento sugli hash SHA-256 per correttezza.</span><span class="sxs-lookup"><span data-stu-id="275b6-483">`BlazorCacheBootResources` also disables Blazor's default behavior of caching the `.dll`, `.wasm`, and other files based on their SHA-256 hashes because the property indicates that the SHA-256 hashes can't be relied upon for correctness.</span></span> <span data-ttu-id="275b6-484">Anche con questa impostazione, la cache HTTP normale del browser può comunque memorizzare nella cache tali file, ma indipendentemente dalla configurazione del server Web e dalle `cache-control` intestazioni che svolge.</span><span class="sxs-lookup"><span data-stu-id="275b6-484">Even with this setting, the browser's normal HTTP cache may still cache those files, but whether or not this happens depends on your web server configuration and the `cache-control` headers that it serves.</span></span>

> [!NOTE]
> <span data-ttu-id="275b6-485">La `BlazorCacheBootResources` proprietà non Disabilita i controlli di integrità per [le applicazioni Web progressive (PWA)](xref:blazor/progressive-web-app).</span><span class="sxs-lookup"><span data-stu-id="275b6-485">The `BlazorCacheBootResources` property doesn't disable integrity checks for [Progressive Web Applications (PWAs)](xref:blazor/progressive-web-app).</span></span> <span data-ttu-id="275b6-486">Per informazioni aggiuntive relative a PWA, vedere la sezione [disabilitare il controllo dell'integrità per PWA](#disable-integrity-checking-for-pwas) .</span><span class="sxs-lookup"><span data-stu-id="275b6-486">For guidance pertaining to PWAs, see the [Disable integrity checking for PWAs](#disable-integrity-checking-for-pwas) section.</span></span>

### <a name="disable-integrity-checking-for-pwas"></a><span data-ttu-id="275b6-487">Disabilitare il controllo dell'integrità per PWA</span><span class="sxs-lookup"><span data-stu-id="275b6-487">Disable integrity checking for PWAs</span></span>

<span data-ttu-id="275b6-488">Blazoril modello di applicazione Web progressiva (PWA) contiene un `service-worker.published.js` file suggerito che è responsabile del recupero e dell'archiviazione dei file dell'applicazione per l'uso offline.</span><span class="sxs-lookup"><span data-stu-id="275b6-488">Blazor's Progressive Web Application (PWA) template contains a suggested `service-worker.published.js` file that's responsible for fetching and storing application files for offline use.</span></span> <span data-ttu-id="275b6-489">Si tratta di un processo separato dal meccanismo di avvio normale dell'app e ha una propria logica di controllo dell'integrità separata.</span><span class="sxs-lookup"><span data-stu-id="275b6-489">This is a separate process from the normal app startup mechanism and has its own separate integrity checking logic.</span></span>

<span data-ttu-id="275b6-490">All'interno del `service-worker.published.js` file è presente la riga seguente:</span><span class="sxs-lookup"><span data-stu-id="275b6-490">Inside the `service-worker.published.js` file, following line is present:</span></span>

```javascript
.map(asset => new Request(asset.url, { integrity: asset.hash }));
```

<span data-ttu-id="275b6-491">Per disabilitare il controllo dell'integrità, rimuovere il `integrity` parametro cambiando la riga come indicato di seguito:</span><span class="sxs-lookup"><span data-stu-id="275b6-491">To disable integrity checking, remove the `integrity` parameter by changing the line to the following:</span></span>

```javascript
.map(asset => new Request(asset.url));
```

<span data-ttu-id="275b6-492">Anche in questo caso, la disabilitazione del controllo dell'integrità comporta la perdita delle garanzie di sicurezza offerte dal controllo dell'integrità.</span><span class="sxs-lookup"><span data-stu-id="275b6-492">Again, disabling integrity checking means that you lose the safety guarantees offered by integrity checking.</span></span> <span data-ttu-id="275b6-493">Ad esempio, esiste il rischio che se il browser dell'utente memorizza nella cache l'app nel momento esatto in cui si distribuisce una nuova versione, potrebbe memorizzare nella cache alcuni file della distribuzione precedente e alcuni dalla nuova distribuzione.</span><span class="sxs-lookup"><span data-stu-id="275b6-493">For example, there is a risk that if the user's browser is caching the app at the exact moment that you deploy a new version, it could cache some files from the old deployment and some from the new deployment.</span></span> <span data-ttu-id="275b6-494">In tal caso, l'app si blocca in uno stato di interruzione finché non si distribuisce un altro aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="275b6-494">If that happens, the app becomes stuck in a broken state until you deploy a further update.</span></span>
