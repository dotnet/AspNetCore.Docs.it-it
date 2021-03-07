---
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
ms.openlocfilehash: f58da78475d65cbb70b0b177e1b0443535e97d55
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102402300"
---
## <a name="troubleshoot"></a><span data-ttu-id="a0d1e-101">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="a0d1e-101">Troubleshoot</span></span>

### <a name="common-errors"></a><span data-ttu-id="a0d1e-102">Errori comuni</span><span class="sxs-lookup"><span data-stu-id="a0d1e-102">Common errors</span></span>

* <span data-ttu-id="a0d1e-103">Configurazione errata dell'app o del Identity provider (IP)</span><span class="sxs-lookup"><span data-stu-id="a0d1e-103">Misconfiguration of the app or Identity Provider (IP)</span></span>

  <span data-ttu-id="a0d1e-104">Gli errori più comuni sono causati da una configurazione non corretta.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-104">The most common errors are caused by incorrect configuration.</span></span> <span data-ttu-id="a0d1e-105">Di seguito sono riportati alcuni esempi:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-105">The following are a few examples:</span></span>
  
  * <span data-ttu-id="a0d1e-106">A seconda dei requisiti dello scenario, un'autorità mancante o errata, un'istanza, un ID tenant, un dominio tenant, un ID client o un URI di reindirizzamento impedisce a un'app di autenticare i client.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-106">Depending on the requirements of the scenario, a missing or incorrect Authority, Instance, Tenant ID, Tenant domain, Client ID, or Redirect URI prevents an app from authenticating clients.</span></span>
  * <span data-ttu-id="a0d1e-107">Un ambito del token di accesso non corretto impedisce ai client di accedere agli endpoint dell'API Web del server.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-107">An incorrect access token scope prevents clients from accessing server web API endpoints.</span></span>
  * <span data-ttu-id="a0d1e-108">Le autorizzazioni dell'API server errate o mancanti impediscono ai client di accedere agli endpoint dell'API Web del server.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-108">Incorrect or missing server API permissions prevent clients from accessing server web API endpoints.</span></span>
  * <span data-ttu-id="a0d1e-109">Esecuzione dell'app in una porta diversa da quella configurata nell'URI di reindirizzamento della Identity registrazione dell'app del provider.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-109">Running the app at a different port than is configured in the Redirect URI of the Identity Provider's app registration.</span></span>
  
  <span data-ttu-id="a0d1e-110">Le sezioni di configurazione delle linee guida di questo articolo mostrano esempi della configurazione corretta.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-110">Configuration sections of this article's guidance show examples of the correct configuration.</span></span> <span data-ttu-id="a0d1e-111">Controllare attentamente ogni sezione dell'articolo ricerca di app e configurazione errata dell'IP.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-111">Carefully check each section of the article looking for app and IP misconfiguration.</span></span>
  
  <span data-ttu-id="a0d1e-112">Se la configurazione appare corretta:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-112">If the configuration appears correct:</span></span>
  
  * <span data-ttu-id="a0d1e-113">Analizzare i registri applicazioni.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-113">Analyze application logs.</span></span>
  * <span data-ttu-id="a0d1e-114">Esaminare il traffico di rete tra l'app client e l'app IP o server con gli strumenti di sviluppo del browser.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-114">Examine the network traffic between the client app and the IP or server app with the browser's developer tools.</span></span> <span data-ttu-id="a0d1e-115">Spesso un messaggio di errore esatto o un messaggio con un indizio sulla causa del problema vengono restituiti al client dall'app IP o server dopo aver effettuato una richiesta.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-115">Often, an exact error message or a message with a clue to what's causing the problem is returned to the client by the IP or server app after making a request.</span></span> <span data-ttu-id="a0d1e-116">Le linee guida per gli strumenti di sviluppo sono disponibili negli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-116">Developer tools guidance is found in the following articles:</span></span>

    * <span data-ttu-id="a0d1e-117">[Google Chrome](https://developers.google.com/web/tools/chrome-devtools/network) (documentazione di Google)</span><span class="sxs-lookup"><span data-stu-id="a0d1e-117">[Google Chrome](https://developers.google.com/web/tools/chrome-devtools/network) (Google documentation)</span></span>
    * [<span data-ttu-id="a0d1e-118">Microsoft Edge</span><span class="sxs-lookup"><span data-stu-id="a0d1e-118">Microsoft Edge</span></span>](/microsoft-edge/devtools-guide-chromium/network/)
    * <span data-ttu-id="a0d1e-119">[Mozilla Firefox](https://developer.mozilla.org/docs/Tools/Network_Monitor) (documentazione di Mozilla)</span><span class="sxs-lookup"><span data-stu-id="a0d1e-119">[Mozilla Firefox](https://developer.mozilla.org/docs/Tools/Network_Monitor) (Mozilla documentation)</span></span>

  * <span data-ttu-id="a0d1e-120">Decodifica il contenuto di un token JSON Web (JWT) usato per l'autenticazione di un client o l'accesso a un'API Web Server, a seconda della posizione in cui si è verificato il problema.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-120">Decode the contents of a JSON Web Token (JWT) used for authenticating a client or accessing a server web API, depending on where the problem is occurring.</span></span> <span data-ttu-id="a0d1e-121">Per altre informazioni, vedere [esaminare il contenuto di un token JSON Web (JWT)](#inspect-the-content-of-a-json-web-token-jwt).</span><span class="sxs-lookup"><span data-stu-id="a0d1e-121">For more information, see [Inspect the content of a JSON Web Token (JWT)](#inspect-the-content-of-a-json-web-token-jwt).</span></span>
  
  <span data-ttu-id="a0d1e-122">Il team di documentazione risponde al feedback dei documenti e ai bug negli articoli (si apre un problema dalla sezione feedback della **pagina** ), ma non è in grado di fornire supporto tecnico.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-122">The documenation team responds to document feedback and bugs in articles (open an issue from the **This page** feedback section) but is unable to provide product support.</span></span> <span data-ttu-id="a0d1e-123">Sono disponibili diversi forum di supporto pubblico per semplificare la risoluzione dei problemi di un'app.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-123">Several public support forums are available to assist with troubleshooting an app.</span></span> <span data-ttu-id="a0d1e-124">È consigliabile attenersi alle linee guida seguenti:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-124">We recommend the following:</span></span>
  
  * [<span data-ttu-id="a0d1e-125">Stack Overflow (Tag: `blazor` )</span><span class="sxs-lookup"><span data-stu-id="a0d1e-125">Stack Overflow (tag: `blazor`)</span></span>](https://stackoverflow.com/questions/tagged/blazor)
  * [<span data-ttu-id="a0d1e-126">Team di ASP.NET Core Slack</span><span class="sxs-lookup"><span data-stu-id="a0d1e-126">ASP.NET Core Slack Team</span></span>](http://tattoocoder.com/aspnet-slack-sign-up/)
  * <span data-ttu-id="a0d1e-127">[Blazor Gitter](https://gitter.im/aspnet/Blazor)</span><span class="sxs-lookup"><span data-stu-id="a0d1e-127">[Blazor Gitter](https://gitter.im/aspnet/Blazor)</span></span>
  
  <span data-ttu-id="a0d1e-128">Per i report sui bug di Framework riproducibili non riservati, non sensibili e non riservati, [aprire un problema con l'unità di prodotto ASP.NET Core](https://github.com/dotnet/aspnetcore/issues).</span><span class="sxs-lookup"><span data-stu-id="a0d1e-128">For non-security, non-sensitive, and non-confidential reproducible framework bug reports, [open an issue with the ASP.NET Core product unit](https://github.com/dotnet/aspnetcore/issues).</span></span> <span data-ttu-id="a0d1e-129">Non aprire un problema con l'unità di prodotto fino a quando non si è approfondita la causa di un problema e non è possibile risolverlo autonomamente e con l'aiuto della community su un forum di supporto pubblico.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-129">Don't open an issue with the product unit until you've thoroughly investigated the cause of a problem and can't resolve it on your own and with the help of the community on a public support forum.</span></span> <span data-ttu-id="a0d1e-130">L'unità prodotto non è in grado di risolvere i problemi relativi alle singole app interrotte a causa di semplici errori di configurazione o casi di utilizzo che coinvolgono servizi di terze parti.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-130">The product unit isn't able to troubleshoot individual apps that are broken due to simple misconfiguration or use cases involving third-party services.</span></span> <span data-ttu-id="a0d1e-131">Se un report è sensibile o riservato in natura o descrive un potenziale difetto di sicurezza nel prodotto che gli utenti malintenzionati possono sfruttare, vedere [segnalazione di problemi di sicurezza e bug (repository GitHub DotNet/aspnetcore)](https://github.com/dotnet/aspnetcore/blob/main/CONTRIBUTING.md#reporting-security-issues-and-bugs).</span><span class="sxs-lookup"><span data-stu-id="a0d1e-131">If a report is sensitive or confidential in nature or describes a potential security flaw in the product that attackers may exploit, see [Reporting security issues and bugs (dotnet/aspnetcore GitHub repository)](https://github.com/dotnet/aspnetcore/blob/main/CONTRIBUTING.md#reporting-security-issues-and-bugs).</span></span>

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="a0d1e-132">Client non autorizzato per AAD</span><span class="sxs-lookup"><span data-stu-id="a0d1e-132">Unauthorized client for AAD</span></span>

  > <span data-ttu-id="a0d1e-133">Info: autorizzazione Microsoft. AspNetCore. Authorization. DefaultAuthorizationService [2] non riuscita.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-133">info: Microsoft.AspNetCore.Authorization.DefaultAuthorizationService[2] Authorization failed.</span></span> <span data-ttu-id="a0d1e-134">Questi requisiti non sono soddisfatti: DenyAnonymousAuthorizationRequirement: richiede un utente autenticato.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-134">These requirements were not met: DenyAnonymousAuthorizationRequirement: Requires an authenticated user.</span></span>

  <span data-ttu-id="a0d1e-135">Errore di callback di accesso da AAD:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-135">Login callback error from AAD:</span></span>

  * <span data-ttu-id="a0d1e-136">Errore: `unauthorized_client`</span><span class="sxs-lookup"><span data-stu-id="a0d1e-136">Error: `unauthorized_client`</span></span>
  * <span data-ttu-id="a0d1e-137">Descrizione `AADB2C90058: The provided application is not configured to allow public clients.`</span><span class="sxs-lookup"><span data-stu-id="a0d1e-137">Description: `AADB2C90058: The provided application is not configured to allow public clients.`</span></span>

  <span data-ttu-id="a0d1e-138">Per risolvere l'errore:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-138">To resolve the error:</span></span>

  1. <span data-ttu-id="a0d1e-139">Nel portale di Azure accedere al manifesto dell' [app](/azure/active-directory/develop/reference-app-manifest).</span><span class="sxs-lookup"><span data-stu-id="a0d1e-139">In the Azure portal, access the [app's manifest](/azure/active-directory/develop/reference-app-manifest).</span></span>
  1. <span data-ttu-id="a0d1e-140">Impostare l' [ `allowPublicClient` attributo](/azure/active-directory/develop/reference-app-manifest#allowpublicclient-attribute) su `null` o su `true` .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-140">Set the [`allowPublicClient` attribute](/azure/active-directory/develop/reference-app-manifest#allowpublicclient-attribute) to `null` or `true`.</span></span>

::: moniker-end

### <a name="cookies-and-site-data"></a><span data-ttu-id="a0d1e-141">Cookies e dati del sito</span><span class="sxs-lookup"><span data-stu-id="a0d1e-141">Cookies and site data</span></span>

<span data-ttu-id="a0d1e-142">Cookiee i dati del sito possono essere mantenuti tra gli aggiornamenti delle app e interferire con i test e la risoluzione dei problemi</span><span class="sxs-lookup"><span data-stu-id="a0d1e-142">Cookies and site data can persist across app updates and interfere with testing and troubleshooting.</span></span> <span data-ttu-id="a0d1e-143">Quando si apportano modifiche al codice dell'app, si cancellano le modifiche dell'account utente con il provider o le modifiche alla configurazione dell'app provider:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-143">Clear the following when making app code changes, user account changes with the provider, or provider app configuration changes:</span></span>

* <span data-ttu-id="a0d1e-144">Accesso utente cookie</span><span class="sxs-lookup"><span data-stu-id="a0d1e-144">User sign-in cookies</span></span>
* <span data-ttu-id="a0d1e-145">App cookie s</span><span class="sxs-lookup"><span data-stu-id="a0d1e-145">App cookies</span></span>
* <span data-ttu-id="a0d1e-146">Dati del sito memorizzati nella cache e archiviati</span><span class="sxs-lookup"><span data-stu-id="a0d1e-146">Cached and stored site data</span></span>

<span data-ttu-id="a0d1e-147">Un approccio per impedire che i cookie dati del sito e di quelli persistenti interferiscano con i test e la risoluzione dei problemi sono:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-147">One approach to prevent lingering cookies and site data from interfering with testing and troubleshooting is to:</span></span>

* <span data-ttu-id="a0d1e-148">Configurare un browser</span><span class="sxs-lookup"><span data-stu-id="a0d1e-148">Configure a browser</span></span>
  * <span data-ttu-id="a0d1e-149">Utilizzare un browser per i test che è possibile configurare per eliminare tutti i cookie dati di e del sito ogni volta che il browser viene chiuso.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-149">Use a browser for testing that you can configure to delete all cookie and site data each time the browser is closed.</span></span>
  * <span data-ttu-id="a0d1e-150">Verificare che il browser sia chiuso manualmente o dall'IDE per qualsiasi modifica all'app, all'utente di test o alla configurazione del provider.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-150">Make sure that the browser is closed manually or by the IDE for any change to the app, test user, or provider configuration.</span></span>
* <span data-ttu-id="a0d1e-151">Usare un comando personalizzato per aprire un browser in modalità privata o in incognito in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-151">Use a custom command to open a browser in incognito or private mode in Visual Studio:</span></span>
  * <span data-ttu-id="a0d1e-152">Aprire la finestra di dialogo **Sfoglia con** dal pulsante di **esecuzione** di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-152">Open **Browse With** dialog box from Visual Studio's **Run** button.</span></span>
  * <span data-ttu-id="a0d1e-153">Fare clic sul pulsante **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-153">Select the **Add** button.</span></span>
  * <span data-ttu-id="a0d1e-154">Consente di specificare il percorso del browser nel campo del **programma** .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-154">Provide the path to your browser in the **Program** field.</span></span> <span data-ttu-id="a0d1e-155">I percorsi eseguibili seguenti sono i percorsi di installazione tipici di Windows 10.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-155">The following executable paths are typical installation locations for Windows 10.</span></span> <span data-ttu-id="a0d1e-156">Se il browser è installato in un percorso diverso o non si usa Windows 10, specificare il percorso del file eseguibile del browser.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-156">If your browser is installed in a different location or you aren't using Windows 10, provide the path to the browser's executable.</span></span>
    * <span data-ttu-id="a0d1e-157">Microsoft Edge: `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`</span><span class="sxs-lookup"><span data-stu-id="a0d1e-157">Microsoft Edge: `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`</span></span>
    * <span data-ttu-id="a0d1e-158">Google Chrome: `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`</span><span class="sxs-lookup"><span data-stu-id="a0d1e-158">Google Chrome: `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`</span></span>
    * <span data-ttu-id="a0d1e-159">Mozilla Firefox: `C:\Program Files\Mozilla Firefox\firefox.exe`</span><span class="sxs-lookup"><span data-stu-id="a0d1e-159">Mozilla Firefox: `C:\Program Files\Mozilla Firefox\firefox.exe`</span></span>
  * <span data-ttu-id="a0d1e-160">Nel campo **arguments (argomenti** ) specificare l'opzione della riga di comando utilizzata dal browser per aprire in modalità privata o in incognito.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-160">In the **Arguments** field, provide the command-line option that the browser uses to open in incognito or private mode.</span></span> <span data-ttu-id="a0d1e-161">Alcuni browser richiedono l'URL dell'app.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-161">Some browsers require the URL of the app.</span></span>
    * <span data-ttu-id="a0d1e-162">Microsoft Edge: usare `-inprivate` .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-162">Microsoft Edge: Use `-inprivate`.</span></span>
    * <span data-ttu-id="a0d1e-163">Google Chrome: usare `--incognito --new-window {URL}` , dove il segnaposto `{URL}` è l'URL da aprire, ad esempio `https://localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-163">Google Chrome: Use `--incognito --new-window {URL}`, where the placeholder `{URL}` is the URL to open (for example, `https://localhost:5001`).</span></span>
    * <span data-ttu-id="a0d1e-164">Mozilla Firefox: usare `-private -url {URL}` , dove il segnaposto `{URL}` è l'URL da aprire, ad esempio `https://localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-164">Mozilla Firefox: Use `-private -url {URL}`, where the placeholder `{URL}` is the URL to open (for example, `https://localhost:5001`).</span></span>
  * <span data-ttu-id="a0d1e-165">Specificare un nome nel campo **nome descrittivo** .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-165">Provide a name in the **Friendly name** field.</span></span> <span data-ttu-id="a0d1e-166">Ad esempio: `Firefox Auth Testing`.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-166">For example, `Firefox Auth Testing`.</span></span>
  * <span data-ttu-id="a0d1e-167">Selezionare il pulsante **OK** .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-167">Select the **OK** button.</span></span>
  * <span data-ttu-id="a0d1e-168">Per evitare di dover selezionare il profilo del browser per ogni iterazione del test con un'app, impostare il profilo come predefinito con il pulsante **Imposta come predefinito** .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-168">To avoid having to select the browser profile for each iteration of testing with an app, set the profile as the default with the **Set as Default** button.</span></span>
  * <span data-ttu-id="a0d1e-169">Verificare che il browser sia chiuso dall'IDE per qualsiasi modifica all'app, all'utente di test o alla configurazione del provider.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-169">Make sure that the browser is closed by the IDE for any change to the app, test user, or provider configuration.</span></span>

### <a name="app-upgrades"></a><span data-ttu-id="a0d1e-170">Aggiornamenti di app</span><span class="sxs-lookup"><span data-stu-id="a0d1e-170">App upgrades</span></span>

<span data-ttu-id="a0d1e-171">Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-171">A functioning app may fail immediately after upgrading either the .NET Core SDK on the development machine or changing package versions within the app.</span></span> <span data-ttu-id="a0d1e-172">In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-172">In some cases, incoherent packages may break an app when performing major upgrades.</span></span> <span data-ttu-id="a0d1e-173">La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-173">Most of these issues can be fixed by following these instructions:</span></span>

1. <span data-ttu-id="a0d1e-174">Cancellare le cache dei pacchetti NuGet del sistema locale eseguendo [`dotnet nuget locals all --clear`](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-174">Clear the local system's NuGet package caches by executing [`dotnet nuget locals all --clear`](/dotnet/core/tools/dotnet-nuget-locals) from a command shell.</span></span>
1. <span data-ttu-id="a0d1e-175">Eliminare le `bin` cartelle e del progetto `obj` .</span><span class="sxs-lookup"><span data-stu-id="a0d1e-175">Delete the project's `bin` and `obj` folders.</span></span>
1. <span data-ttu-id="a0d1e-176">Ripristinare e ricompilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-176">Restore and rebuild the project.</span></span>
1. <span data-ttu-id="a0d1e-177">Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-177">Delete all of the files in the deployment folder on the server prior to redeploying the app.</span></span>

> [!NOTE]
> <span data-ttu-id="a0d1e-178">L'uso di versioni dei pacchetti incompatibili con il Framework di destinazione dell'app non è supportato.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-178">Use of package versions incompatible with the app's target framework isn't supported.</span></span> <span data-ttu-id="a0d1e-179">Per informazioni su un pacchetto, usare la [raccolta NuGet](https://www.nuget.org) o [FuGet Package Explorer](https://www.fuget.org).</span><span class="sxs-lookup"><span data-stu-id="a0d1e-179">For information on a package, use the [NuGet Gallery](https://www.nuget.org) or [FuGet Package Explorer](https://www.fuget.org).</span></span>

### <a name="run-the-server-app"></a><span data-ttu-id="a0d1e-180">Eseguire l'app Server</span><span class="sxs-lookup"><span data-stu-id="a0d1e-180">Run the Server app</span></span>

<span data-ttu-id="a0d1e-181">Durante il test e la risoluzione dei problemi di una soluzione ospitata Blazor , assicurarsi di eseguire l'app dal **`Server`** progetto.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-181">When testing and troubleshooting a hosted Blazor solution, make sure that you're running the app from the **`Server`** project.</span></span> <span data-ttu-id="a0d1e-182">Ad esempio, in Visual Studio verificare che il progetto server sia evidenziato in **Esplora soluzioni** prima di avviare l'app con uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-182">For example in Visual Studio, confirm that the Server project is highlighted in **Solution Explorer** before you start the app with any of the following approaches:</span></span>

* <span data-ttu-id="a0d1e-183">Selezionare il pulsante **Run** (Esegui).</span><span class="sxs-lookup"><span data-stu-id="a0d1e-183">Select the **Run** button.</span></span>
* <span data-ttu-id="a0d1e-184">Usare **debug**  >  **Avvia debug** dal menu.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-184">Use **Debug** > **Start Debugging** from the menu.</span></span>
* <span data-ttu-id="a0d1e-185">Premere <kbd>F5</kbd>.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-185">Press <kbd>F5</kbd>.</span></span>

### <a name="inspect-the-content-of-a-json-web-token-jwt"></a><span data-ttu-id="a0d1e-186">Esaminare il contenuto di un token JSON Web (JWT)</span><span class="sxs-lookup"><span data-stu-id="a0d1e-186">Inspect the content of a JSON Web Token (JWT)</span></span>

<span data-ttu-id="a0d1e-187">Per decodificare un token JSON Web (JWT), usare lo strumento [JWT.ms](https://jwt.ms/) di Microsoft.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-187">To decode a JSON Web Token (JWT), use Microsoft's [jwt.ms](https://jwt.ms/) tool.</span></span> <span data-ttu-id="a0d1e-188">I valori nell'interfaccia utente non lasciano mai il browser.</span><span class="sxs-lookup"><span data-stu-id="a0d1e-188">Values in the UI never leave your browser.</span></span>

<span data-ttu-id="a0d1e-189">Esempio di JWT codificato (abbreviato per la visualizzazione):</span><span class="sxs-lookup"><span data-stu-id="a0d1e-189">Example encoded JWT (shortened for display):</span></span>

> <span data-ttu-id="a0d1e-190">eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrNHh5b2pORnVtMWtsMll0djhkbE5QNC1j ... bQdHBHGcQQRbW7Wmo6SWYG4V_bU55Ug_PW4pLPr20tTS8Ct7_uwy9DWrzCMzpD-EiwT5IjXwlGX3IXVjHIlX50IVIydBoPQtadvT7saKo1G5Jmutgq41o-dmz6-yBMKV2_nXA25Q</span><span class="sxs-lookup"><span data-stu-id="a0d1e-190">eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrNHh5b2pORnVtMWtsMll0djhkbE5QNC1j ... bQdHBHGcQQRbW7Wmo6SWYG4V_bU55Ug_PW4pLPr20tTS8Ct7_uwy9DWrzCMzpD-EiwT5IjXwlGX3IXVjHIlX50IVIydBoPQtadvT7saKo1G5Jmutgq41o-dmz6-yBMKV2_nXA25Q</span></span>

<span data-ttu-id="a0d1e-191">Esempio di JWT decodificato dallo strumento per un'app che esegue l'autenticazione con AAD B2C di Azure:</span><span class="sxs-lookup"><span data-stu-id="a0d1e-191">Example JWT decoded by the tool for an app that authenticates against Azure AAD B2C:</span></span>

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1610059429,
  "nbf": 1610055829,
  "ver": "1.0",
  "iss": "https://mysiteb2c.b2clogin.com/5cc15ea8-a296-4aa3-97e4-226dcc9ad298/v2.0/",
  "sub": "5ee963fb-24d6-4d72-a1b6-889c6e2c7438",
  "aud": "70bde375-fce3-4b82-984a-b247d823a03f",
  "nonce": "b2641f54-8dc4-42ca-97ea-7f12ff4af871",
  "iat": 1610055829,
  "auth_time": 1610055822,
  "idp": "idp.com",
  "tfp": "B2C_1_signupsignin"
}.[Signature]
```
