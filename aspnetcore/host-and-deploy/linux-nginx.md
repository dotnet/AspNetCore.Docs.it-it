---
title: Hosting di ASP.NET Core in Linux con Nginx
author: rick-anderson
description: Informazioni su come configurare nginx come proxy inverso in Ubuntu 16,04 per l'inoltro del traffico HTTP a una ASP.NET Core app Web in esecuzione in gheppio.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/30/2020
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
uid: host-and-deploy/linux-nginx
ms.openlocfilehash: 8a593654fa31e643e7c239f361f035589c75ce98
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395253"
---
# <a name="host-aspnet-core-on-linux-with-nginx"></a><span data-ttu-id="590bc-103">Hosting di ASP.NET Core in Linux con Nginx</span><span class="sxs-lookup"><span data-stu-id="590bc-103">Host ASP.NET Core on Linux with Nginx</span></span>

<span data-ttu-id="590bc-104">Di [Sourabh Shirhatti](https://twitter.com/sshirhatti)</span><span class="sxs-lookup"><span data-stu-id="590bc-104">By [Sourabh Shirhatti](https://twitter.com/sshirhatti)</span></span>

<span data-ttu-id="590bc-105">Questa guida descrive come configurare un ambiente ASP.NET Core pronto per la produzione in un server Ubuntu 16.04.</span><span class="sxs-lookup"><span data-stu-id="590bc-105">This guide explains setting up a production-ready ASP.NET Core environment on an Ubuntu 16.04 server.</span></span> <span data-ttu-id="590bc-106">Queste istruzioni si applicano probabilmente anche alle versioni più recenti di Ubuntu, sebbene non siano state testate con le versioni più recenti.</span><span class="sxs-lookup"><span data-stu-id="590bc-106">These instructions likely work with newer versions of Ubuntu, but the instructions haven't been tested with newer versions.</span></span>

<span data-ttu-id="590bc-107">Per informazioni su altre distribuzioni Linux supportate da ASP.NET Core, vedere [Prerequisiti per .NET Core in Linux](/dotnet/core/linux-prerequisites).</span><span class="sxs-lookup"><span data-stu-id="590bc-107">For information on other Linux distributions supported by ASP.NET Core, see [Prerequisites for .NET Core on Linux](/dotnet/core/linux-prerequisites).</span></span>

> [!NOTE]
> <span data-ttu-id="590bc-108">Per Ubuntu 14,04, `supervisord` è consigliabile come soluzione per il monitoraggio del processo gheppio.</span><span class="sxs-lookup"><span data-stu-id="590bc-108">For Ubuntu 14.04, `supervisord` is recommended as a solution for monitoring the Kestrel process.</span></span> <span data-ttu-id="590bc-109">`systemd` non è disponibile in Ubuntu 14,04.</span><span class="sxs-lookup"><span data-stu-id="590bc-109">`systemd` isn't available on Ubuntu 14.04.</span></span> <span data-ttu-id="590bc-110">Per le istruzioni per Ubuntu 14.04, vedere la [versione precedente di questo argomento](https://github.com/dotnet/AspNetCore.Docs/blob/e9c1419175c4dd7e152df3746ba1df5935aaafd5/aspnetcore/publishing/linuxproduction.md).</span><span class="sxs-lookup"><span data-stu-id="590bc-110">For Ubuntu 14.04 instructions, see the [previous version of this topic](https://github.com/dotnet/AspNetCore.Docs/blob/e9c1419175c4dd7e152df3746ba1df5935aaafd5/aspnetcore/publishing/linuxproduction.md).</span></span>

<span data-ttu-id="590bc-111">In questa guida:</span><span class="sxs-lookup"><span data-stu-id="590bc-111">This guide:</span></span>

* <span data-ttu-id="590bc-112">Posizionare un'app ASP.NET Core esistente dietro un server proxy inverso.</span><span class="sxs-lookup"><span data-stu-id="590bc-112">Places an existing ASP.NET Core app behind a reverse proxy server.</span></span>
* <span data-ttu-id="590bc-113">Configurare il server proxy inverso per l'inoltro delle richieste al server Web Kestrel.</span><span class="sxs-lookup"><span data-stu-id="590bc-113">Sets up the reverse proxy server to forward requests to the Kestrel web server.</span></span>
* <span data-ttu-id="590bc-114">Verificare che l'app Web venga eseguita all'avvio come daemon.</span><span class="sxs-lookup"><span data-stu-id="590bc-114">Ensures the web app runs on startup as a daemon.</span></span>
* <span data-ttu-id="590bc-115">Configurare uno strumento di gestione del processo per consentire il riavvio dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="590bc-115">Configures a process management tool to help restart the web app.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="590bc-116">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="590bc-116">Prerequisites</span></span>

* <span data-ttu-id="590bc-117">Accedere a un server Ubuntu 16.04 con un account utente standard con privilegio sudo.</span><span class="sxs-lookup"><span data-stu-id="590bc-117">Access to an Ubuntu 16.04 server with a standard user account with sudo privilege.</span></span>
* <span data-ttu-id="590bc-118">Il [Runtime .NET](/dotnet/core/install/linux) non di anteprima più recente installato nel server.</span><span class="sxs-lookup"><span data-stu-id="590bc-118">The latest non-preview [.NET runtime installed](/dotnet/core/install/linux) on the server.</span></span>
* <span data-ttu-id="590bc-119">Un'app ASP.NET Core esistente.</span><span class="sxs-lookup"><span data-stu-id="590bc-119">An existing ASP.NET Core app.</span></span>

<span data-ttu-id="590bc-120">In qualsiasi momento in futuro dopo l'aggiornamento del Framework condiviso, riavviare le app ASP.NET Core ospitate dal server.</span><span class="sxs-lookup"><span data-stu-id="590bc-120">At any point in the future after upgrading the shared framework, restart the ASP.NET Core apps hosted by the server.</span></span>

## <a name="publish-and-copy-over-the-app"></a><span data-ttu-id="590bc-121">Pubblicare e copiare l'app</span><span class="sxs-lookup"><span data-stu-id="590bc-121">Publish and copy over the app</span></span>

<span data-ttu-id="590bc-122">Configurare l'app per una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd).</span><span class="sxs-lookup"><span data-stu-id="590bc-122">Configure the app for a [framework-dependent deployment](/dotnet/core/deploying/#framework-dependent-deployments-fdd).</span></span>

<span data-ttu-id="590bc-123">Se l'app viene eseguita in locale e non è configurata per connessioni sicure (HTTPS), adottare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="590bc-123">If the app is run locally and isn't configured to make secure connections (HTTPS), adopt either of the following approaches:</span></span>

* <span data-ttu-id="590bc-124">Configurare l'app per la gestione di connessioni locali sicure.</span><span class="sxs-lookup"><span data-stu-id="590bc-124">Configure the app to handle secure local connections.</span></span> <span data-ttu-id="590bc-125">Per altre informazioni, vedere la sezione [Configurazione HTTPS](#https-configuration).</span><span class="sxs-lookup"><span data-stu-id="590bc-125">For more information, see the [HTTPS configuration](#https-configuration) section.</span></span>
* <span data-ttu-id="590bc-126">Rimuovere `https://localhost:5001` , se presente, dalla `applicationUrl` proprietà nel `Properties/launchSettings.json` file.</span><span class="sxs-lookup"><span data-stu-id="590bc-126">Remove `https://localhost:5001` (if present) from the `applicationUrl` property in the `Properties/launchSettings.json` file.</span></span>

<span data-ttu-id="590bc-127">Eseguire [DotNet Publish](/dotnet/core/tools/dotnet-publish) dall'ambiente di sviluppo per creare un pacchetto di un'app in una directory (ad esempio, `bin/Release/{TARGET FRAMEWORK MONIKER}/publish` , dove il segnaposto `{TARGET FRAMEWORK MONIKER}` è il moniker del Framework di destinazione/TFM) che può essere eseguito nel server:</span><span class="sxs-lookup"><span data-stu-id="590bc-127">Run [dotnet publish](/dotnet/core/tools/dotnet-publish) from the development environment to package an app into a directory (for example, `bin/Release/{TARGET FRAMEWORK MONIKER}/publish`, where the placeholder `{TARGET FRAMEWORK MONIKER}` is the Target Framework Moniker/TFM) that can run on the server:</span></span>

```dotnetcli
dotnet publish --configuration Release
```

<span data-ttu-id="590bc-128">È anche possibile pubblicare l'app come [distribuzione completa](/dotnet/core/deploying/#self-contained-deployments-scd) se si preferisce non mantenere il runtime .NET Core nel server.</span><span class="sxs-lookup"><span data-stu-id="590bc-128">The app can also be published as a [self-contained deployment](/dotnet/core/deploying/#self-contained-deployments-scd) if you prefer not to maintain the .NET Core runtime on the server.</span></span>

<span data-ttu-id="590bc-129">Copiare l'app ASP.NET Core nel server usando uno strumento che si integra con il flusso di lavoro dell'organizzazione, ad esempio `SCP` , `SFTP` .</span><span class="sxs-lookup"><span data-stu-id="590bc-129">Copy the ASP.NET Core app to the server using a tool that integrates into the organization's workflow (for example, `SCP`, `SFTP`).</span></span> <span data-ttu-id="590bc-130">È comune individuare le app Web nella `var` Directory (ad esempio, `var/www/helloapp` ).</span><span class="sxs-lookup"><span data-stu-id="590bc-130">It's common to locate web apps under the `var` directory (for example, `var/www/helloapp`).</span></span>

> [!NOTE]
> <span data-ttu-id="590bc-131">In uno scenario di distribuzione di produzione un flusso di lavoro di integrazione continua esegue le operazioni di pubblicazione dell'app e di copia degli asset nel server.</span><span class="sxs-lookup"><span data-stu-id="590bc-131">Under a production deployment scenario, a continuous integration workflow does the work of publishing the app and copying the assets to the server.</span></span>

<span data-ttu-id="590bc-132">Eseguire il test dell'app:</span><span class="sxs-lookup"><span data-stu-id="590bc-132">Test the app:</span></span>

1. <span data-ttu-id="590bc-133">Dalla riga di comando, eseguire l'app: `dotnet <app_assembly>.dll`.</span><span class="sxs-lookup"><span data-stu-id="590bc-133">From the command line, run the app: `dotnet <app_assembly>.dll`.</span></span>
1. <span data-ttu-id="590bc-134">In un browser passare a `http://<serveraddress>:<port>` per verificare se l'app funziona in Linux in locale.</span><span class="sxs-lookup"><span data-stu-id="590bc-134">In a browser, navigate to `http://<serveraddress>:<port>` to verify the app works on Linux locally.</span></span>

## <a name="configure-a-reverse-proxy-server"></a><span data-ttu-id="590bc-135">Configurare un server proxy inverso</span><span class="sxs-lookup"><span data-stu-id="590bc-135">Configure a reverse proxy server</span></span>

<span data-ttu-id="590bc-136">Un proxy inverso è una configurazione comune per la gestione delle app Web dinamiche.</span><span class="sxs-lookup"><span data-stu-id="590bc-136">A reverse proxy is a common setup for serving dynamic web apps.</span></span> <span data-ttu-id="590bc-137">Un proxy inverso termina la richiesta HTTP e la inoltra all'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="590bc-137">A reverse proxy terminates the HTTP request and forwards it to the ASP.NET Core app.</span></span>

### <a name="use-a-reverse-proxy-server"></a><span data-ttu-id="590bc-138">Usare un server proxy inverso</span><span class="sxs-lookup"><span data-stu-id="590bc-138">Use a reverse proxy server</span></span>

<span data-ttu-id="590bc-139">Kestrel funziona perfettamente per la gestione del contenuto dinamico da ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="590bc-139">Kestrel is great for serving dynamic content from ASP.NET Core.</span></span> <span data-ttu-id="590bc-140">Tuttavia, le funzionalità di gestione Web sono meno avanzate rispetto a quelle offerte da server come IIS, Apache o Nginx.</span><span class="sxs-lookup"><span data-stu-id="590bc-140">However, the web serving capabilities aren't as feature rich as servers such as IIS, Apache, or Nginx.</span></span> <span data-ttu-id="590bc-141">Un server proxy inverso è in grado di eseguire l'offload di attività come la gestione del contenuto statico, la memorizzazione nella cache delle richieste, la compressione delle richieste e la terminazione HTTPS dal server HTTP.</span><span class="sxs-lookup"><span data-stu-id="590bc-141">A reverse proxy server can offload work such as serving static content, caching requests, compressing requests, and HTTPS termination from the HTTP server.</span></span> <span data-ttu-id="590bc-142">Un server proxy inverso può risiedere in un computer dedicato o essere distribuito insieme a un server HTTP.</span><span class="sxs-lookup"><span data-stu-id="590bc-142">A reverse proxy server may reside on a dedicated machine or may be deployed alongside an HTTP server.</span></span>

<span data-ttu-id="590bc-143">Ai fini di questa guida viene usata una singola istanza di Nginx.</span><span class="sxs-lookup"><span data-stu-id="590bc-143">For the purposes of this guide, a single instance of Nginx is used.</span></span> <span data-ttu-id="590bc-144">Viene eseguito sullo stesso server, insieme al server HTTP.</span><span class="sxs-lookup"><span data-stu-id="590bc-144">It runs on the same server, alongside the HTTP server.</span></span> <span data-ttu-id="590bc-145">In base ai requisiti, è possibile scegliere una configurazione diversa.</span><span class="sxs-lookup"><span data-stu-id="590bc-145">Based on requirements, a different setup may be chosen.</span></span>

<span data-ttu-id="590bc-146">Poiché le richieste vengono inviate dal proxy inverso, utilizzare il [middleware delle intestazioni inoltro](xref:host-and-deploy/proxy-load-balancer) dal [`Microsoft.AspNetCore.HttpOverrides`](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="590bc-146">Because requests are forwarded by reverse proxy, use the [Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer) from the [`Microsoft.AspNetCore.HttpOverrides`](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides) package.</span></span> <span data-ttu-id="590bc-147">Il middleware aggiorna `Request.Scheme` usando l'intestazione `X-Forwarded-Proto`, in modo che gli URI di reindirizzamento e altri criteri di sicurezza funzionino correttamente.</span><span class="sxs-lookup"><span data-stu-id="590bc-147">The middleware updates the `Request.Scheme`, using the `X-Forwarded-Proto` header, so that redirect URIs and other security policies work correctly.</span></span>

[!INCLUDE[](~/includes/ForwardedHeaders.md)]

<span data-ttu-id="590bc-148">Richiamare il <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders%2A> metodo all'inizio di `Startup.Configure` prima di chiamare altro middleware.</span><span class="sxs-lookup"><span data-stu-id="590bc-148">Invoke the <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders%2A> method at the top of `Startup.Configure` before calling other middleware.</span></span> <span data-ttu-id="590bc-149">Configurare il middleware per l'inoltro delle intestazioni `X-Forwarded-For` e `X-Forwarded-Proto`:</span><span class="sxs-lookup"><span data-stu-id="590bc-149">Configure the middleware to forward the `X-Forwarded-For` and `X-Forwarded-Proto` headers:</span></span>

```csharp
using Microsoft.AspNetCore.HttpOverrides;

...

app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

app.UseAuthentication();
```

<span data-ttu-id="590bc-150">Se non vengono specificate opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> per il middleware, le intestazioni predefinite per l'inoltro sono `None`.</span><span class="sxs-lookup"><span data-stu-id="590bc-150">If no <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> are specified to the middleware, the default headers to forward are `None`.</span></span>

<span data-ttu-id="590bc-151">I proxy eseguiti sugli indirizzi di loopback ( `127.0.0.0/8` , `[::1]` ), incluso l'indirizzo localhost standard ( `127.0.0.1` ), sono considerati attendibili per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="590bc-151">Proxies running on loopback addresses (`127.0.0.0/8`, `[::1]`), including the standard localhost address (`127.0.0.1`), are trusted by default.</span></span> <span data-ttu-id="590bc-152">Se le richieste tra Internet e il server Web vengono gestite anche da altri proxy o reti attendibili all'interno dell'organizzazione, aggiungerli all'elenco di <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies%2A> o <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks%2A> con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions>.</span><span class="sxs-lookup"><span data-stu-id="590bc-152">If other trusted proxies or networks within the organization handle requests between the Internet and the web server, add them to the list of <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies%2A> or <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks%2A> with <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions>.</span></span> <span data-ttu-id="590bc-153">L'esempio seguente aggiunge un server proxy attendibile all'indirizzo IP 10.0.0.100 nel middleware delle intestazioni inoltrate `KnownProxies` in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="590bc-153">The following example adds a trusted proxy server at IP address 10.0.0.100 to the Forwarded Headers Middleware `KnownProxies` in `Startup.ConfigureServices`:</span></span>

```csharp
using System.Net;

...

services.Configure<ForwardedHeadersOptions>(options =>
{
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.100"));
});
```

<span data-ttu-id="590bc-154">Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.</span><span class="sxs-lookup"><span data-stu-id="590bc-154">For more information, see <xref:host-and-deploy/proxy-load-balancer>.</span></span>

### <a name="install-nginx"></a><span data-ttu-id="590bc-155">Installare Nginx</span><span class="sxs-lookup"><span data-stu-id="590bc-155">Install Nginx</span></span>

<span data-ttu-id="590bc-156">Usare `apt-get` per installare Nginx.</span><span class="sxs-lookup"><span data-stu-id="590bc-156">Use `apt-get` to install Nginx.</span></span> <span data-ttu-id="590bc-157">Il programma di installazione crea uno `systemd` script di inizializzazione che esegue nginx come daemon all'avvio del sistema.</span><span class="sxs-lookup"><span data-stu-id="590bc-157">The installer creates a `systemd` init script that runs Nginx as daemon on system startup.</span></span> <span data-ttu-id="590bc-158">Seguire le istruzioni di installazione per Ubuntu riportate nell'articolo [Nginx: Official Debian/Ubuntu packages](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages) (Nginx: pacchetti Debian/Ubuntu ufficiali).</span><span class="sxs-lookup"><span data-stu-id="590bc-158">Follow the installation instructions for Ubuntu at [Nginx: Official Debian/Ubuntu packages](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages).</span></span>

> [!NOTE]
> <span data-ttu-id="590bc-159">Se sono richiesti moduli Nginx facoltativi, potrebbe essere necessario compilare Nginx dall'origine.</span><span class="sxs-lookup"><span data-stu-id="590bc-159">If optional Nginx modules are required, building Nginx from source might be required.</span></span>

<span data-ttu-id="590bc-160">Poiché Nginx è stato installato per la prima volta, avviarlo in modo esplicito eseguendo:</span><span class="sxs-lookup"><span data-stu-id="590bc-160">Since Nginx was installed for the first time, explicitly start it by running:</span></span>

```bash
sudo service nginx start
```

<span data-ttu-id="590bc-161">Verificare che un browser visualizzi la pagina di destinazione predefinita per Nginx.</span><span class="sxs-lookup"><span data-stu-id="590bc-161">Verify a browser displays the default landing page for Nginx.</span></span> <span data-ttu-id="590bc-162">La pagina di destinazione è raggiungibile all'indirizzo `http://<server_IP_address>/index.nginx-debian.html`.</span><span class="sxs-lookup"><span data-stu-id="590bc-162">The landing page is reachable at `http://<server_IP_address>/index.nginx-debian.html`.</span></span>

### <a name="configure-nginx"></a><span data-ttu-id="590bc-163">Configurare Nginx</span><span class="sxs-lookup"><span data-stu-id="590bc-163">Configure Nginx</span></span>

<span data-ttu-id="590bc-164">Per configurare nginx come proxy inverso per l'inoltro delle richieste HTTP all'app ASP.NET Core, modificare `/etc/nginx/sites-available/default` .</span><span class="sxs-lookup"><span data-stu-id="590bc-164">To configure Nginx as a reverse proxy to forward HTTP requests to your ASP.NET Core app, modify `/etc/nginx/sites-available/default`.</span></span> <span data-ttu-id="590bc-165">Aprirlo in un editor di testo e sostituire il contenuto con il frammento di codice seguente:</span><span class="sxs-lookup"><span data-stu-id="590bc-165">Open it in a text editor, and replace the contents with the following snippet:</span></span>

```nginx
server {
    listen        80;
    server_name   example.com *.example.com;
    location / {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

<span data-ttu-id="590bc-166">Se l'app è un' SignalR Blazor Server app o, vedere <xref:signalr/scale#linux-with-nginx> <xref:blazor/host-and-deploy/server#linux-with-nginx> rispettivamente e per altre informazioni.</span><span class="sxs-lookup"><span data-stu-id="590bc-166">If the app is a SignalR or Blazor Server app, see <xref:signalr/scale#linux-with-nginx> and <xref:blazor/host-and-deploy/server#linux-with-nginx> respectively for more information.</span></span>

<span data-ttu-id="590bc-167">Se nessun `server_name` corrisponde, Nginx usa il server predefinito.</span><span class="sxs-lookup"><span data-stu-id="590bc-167">When no `server_name` matches, Nginx uses the default server.</span></span> <span data-ttu-id="590bc-168">Se non è definito alcun server predefinito, il primo server nel file di configurazione è il server predefinito.</span><span class="sxs-lookup"><span data-stu-id="590bc-168">If no default server is defined, the first server in the configuration file is the default server.</span></span> <span data-ttu-id="590bc-169">Come procedura consigliata, aggiungere un server predefinito specifico che restituisca un codice di stato 444 nel file di configurazione.</span><span class="sxs-lookup"><span data-stu-id="590bc-169">As a best practice, add a specific default server that returns a status code of 444 in your configuration file.</span></span> <span data-ttu-id="590bc-170">Un esempio di configurazione del server predefinito è il seguente:</span><span class="sxs-lookup"><span data-stu-id="590bc-170">A default server configuration example is:</span></span>

```nginx
server {
    listen   80 default_server;
    # listen [::]:80 default_server deferred;
    return   444;
}
```

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="590bc-171">Con il file di configurazione precedente e il server predefinito, Nginx accetta il traffico pubblico sulla porta 80 con l'intestazione host `example.com` o `*.example.com`.</span><span class="sxs-lookup"><span data-stu-id="590bc-171">With the preceding configuration file and default server, Nginx accepts public traffic on port 80 with host header `example.com` or `*.example.com`.</span></span> <span data-ttu-id="590bc-172">Le richieste che non corrispondono a questi host non verranno inoltrate a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="590bc-172">Requests not matching these hosts won't get forwarded to Kestrel.</span></span> <span data-ttu-id="590bc-173">Nginx inoltra a Kestrel le richieste corrispondenti a `http://localhost:5000`.</span><span class="sxs-lookup"><span data-stu-id="590bc-173">Nginx forwards the matching requests to Kestrel at `http://localhost:5000`.</span></span> <span data-ttu-id="590bc-174">Per altre informazioni, vedere [come Nginx elabora una richiesta](https://nginx.org/docs/http/request_processing.html).</span><span class="sxs-lookup"><span data-stu-id="590bc-174">For more information, see [How nginx processes a request](https://nginx.org/docs/http/request_processing.html).</span></span> <span data-ttu-id="590bc-175">Per cambiare porta/IP di Kestrel, vedere [Kestrel: Configurazione dell'endpoint](xref:fundamentals/servers/kestrel/endpoints).</span><span class="sxs-lookup"><span data-stu-id="590bc-175">To change Kestrel's IP/port, see [Kestrel: Endpoint configuration](xref:fundamentals/servers/kestrel/endpoints).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="590bc-176">Con il file di configurazione precedente e il server predefinito, Nginx accetta il traffico pubblico sulla porta 80 con l'intestazione host `example.com` o `*.example.com`.</span><span class="sxs-lookup"><span data-stu-id="590bc-176">With the preceding configuration file and default server, Nginx accepts public traffic on port 80 with host header `example.com` or `*.example.com`.</span></span> <span data-ttu-id="590bc-177">Le richieste che non corrispondono a questi host non verranno inoltrate a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="590bc-177">Requests not matching these hosts won't get forwarded to Kestrel.</span></span> <span data-ttu-id="590bc-178">Nginx inoltra a Kestrel le richieste corrispondenti a `http://localhost:5000`.</span><span class="sxs-lookup"><span data-stu-id="590bc-178">Nginx forwards the matching requests to Kestrel at `http://localhost:5000`.</span></span> <span data-ttu-id="590bc-179">Per altre informazioni, vedere [come Nginx elabora una richiesta](https://nginx.org/docs/http/request_processing.html).</span><span class="sxs-lookup"><span data-stu-id="590bc-179">For more information, see [How nginx processes a request](https://nginx.org/docs/http/request_processing.html).</span></span> <span data-ttu-id="590bc-180">Per cambiare porta/IP di Kestrel, vedere [Kestrel: Configurazione dell'endpoint](xref:fundamentals/servers/kestrel#endpoint-configuration).</span><span class="sxs-lookup"><span data-stu-id="590bc-180">To change Kestrel's IP/port, see [Kestrel: Endpoint configuration](xref:fundamentals/servers/kestrel#endpoint-configuration).</span></span>

::: moniker-end

> [!WARNING]
> <span data-ttu-id="590bc-181">Se non si specifica una [direttiva server_name](https://nginx.org/docs/http/server_names.html) corretta, l'app può risultare esposta a vulnerabilità di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="590bc-181">Failure to specify a proper [server_name directive](https://nginx.org/docs/http/server_names.html) exposes your app to security vulnerabilities.</span></span> <span data-ttu-id="590bc-182">L'associazione con caratteri jolly del sottodominio (ad esempio, `*.example.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile).</span><span class="sxs-lookup"><span data-stu-id="590bc-182">Subdomain wildcard binding (for example, `*.example.com`) doesn't pose this security risk if you control the entire parent domain (as opposed to `*.com`, which is vulnerable).</span></span> <span data-ttu-id="590bc-183">Per ulteriori informazioni, vedere la [sezione rfc7230-5,4](https://tools.ietf.org/html/rfc7230#section-5.4).</span><span class="sxs-lookup"><span data-stu-id="590bc-183">For more information, see [rfc7230 section-5.4](https://tools.ietf.org/html/rfc7230#section-5.4).</span></span>

<span data-ttu-id="590bc-184">Una volta stabilita la configurazione di Nginx, eseguire `sudo nginx -t` per verificare la sintassi dei file di configurazione.</span><span class="sxs-lookup"><span data-stu-id="590bc-184">Once the Nginx configuration is established, run `sudo nginx -t` to verify the syntax of the configuration files.</span></span> <span data-ttu-id="590bc-185">Se il test dei file di configurazione ha esito positivo, è possibile forzare Nginx ad applicare le modifiche eseguendo `sudo nginx -s reload`.</span><span class="sxs-lookup"><span data-stu-id="590bc-185">If the configuration file test is successful, force Nginx to pick up the changes by running `sudo nginx -s reload`.</span></span>

<span data-ttu-id="590bc-186">Per eseguire direttamente l'app nel server:</span><span class="sxs-lookup"><span data-stu-id="590bc-186">To directly run the app on the server:</span></span>

1. <span data-ttu-id="590bc-187">Passare alla directory dell'app.</span><span class="sxs-lookup"><span data-stu-id="590bc-187">Navigate to the app's directory.</span></span>
1. <span data-ttu-id="590bc-188">Eseguire l'app: `dotnet <app_assembly.dll>`, dove `app_assembly.dll` è il nome di file di assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="590bc-188">Run the app: `dotnet <app_assembly.dll>`, where `app_assembly.dll` is the assembly file name of the app.</span></span>

<span data-ttu-id="590bc-189">Se l'app viene eseguita nel server ma non riesce a rispondere tramite Internet, controllare il firewall del server e verificare che la porta 80 sia aperta.</span><span class="sxs-lookup"><span data-stu-id="590bc-189">If the app runs on the server but fails to respond over the Internet, check the server's firewall and confirm port 80 is open.</span></span> <span data-ttu-id="590bc-190">Se si usa una macchina virtuale Ubuntu di Azure, aggiungere una regola del gruppo di sicurezza di rete (NSG) che abiliti il traffico in ingresso della porta 80.</span><span class="sxs-lookup"><span data-stu-id="590bc-190">If using an Azure Ubuntu VM, add a Network Security Group (NSG) rule that enables inbound port 80 traffic.</span></span> <span data-ttu-id="590bc-191">Non è necessario abilitare una regola per il traffico in uscita della porta 80, poiché il traffico in uscita viene autorizzato automaticamente quando viene abilitata la regola in ingresso.</span><span class="sxs-lookup"><span data-stu-id="590bc-191">There's no need to enable an outbound port 80 rule, as the outbound traffic is automatically granted when the inbound rule is enabled.</span></span>

<span data-ttu-id="590bc-192">Al termine del test dell'app, arrestare l'app con <kbd>CTRL</kbd>  +  <kbd>C</kbd> al prompt dei comandi.</span><span class="sxs-lookup"><span data-stu-id="590bc-192">When done testing the app, shut down the app with <kbd>Ctrl</kbd> + <kbd>C</kbd> at the command prompt.</span></span>

## <a name="monitor-the-app"></a><span data-ttu-id="590bc-193">Monitorare l'app</span><span class="sxs-lookup"><span data-stu-id="590bc-193">Monitor the app</span></span>

<span data-ttu-id="590bc-194">Il server è configurato per l'invio delle richieste effettuate a al `http://<serveraddress>:80` ASP.NET Core app in esecuzione in gheppio all'indirizzo `http://127.0.0.1:5000` .</span><span class="sxs-lookup"><span data-stu-id="590bc-194">The server is set up to forward requests made to `http://<serveraddress>:80` on to the ASP.NET Core app running on Kestrel at `http://127.0.0.1:5000`.</span></span> <span data-ttu-id="590bc-195">Tuttavia, Nginx non è configurato per la gestione del processo Kestrel.</span><span class="sxs-lookup"><span data-stu-id="590bc-195">However, Nginx isn't set up to manage the Kestrel process.</span></span> <span data-ttu-id="590bc-196">`systemd` può essere usato per creare un file di servizio per avviare e monitorare l'app Web sottostante.</span><span class="sxs-lookup"><span data-stu-id="590bc-196">`systemd` can be used to create a service file to start and monitor the underlying web app.</span></span> <span data-ttu-id="590bc-197">`systemd` è un sistema di inizializzazione che offre molte funzionalità potenti per l'avvio, l'arresto e la gestione dei processi.</span><span class="sxs-lookup"><span data-stu-id="590bc-197">`systemd` is an init system that provides many powerful features for starting, stopping, and managing processes.</span></span> 

### <a name="create-the-service-file"></a><span data-ttu-id="590bc-198">Creare il file del servizio</span><span class="sxs-lookup"><span data-stu-id="590bc-198">Create the service file</span></span>

<span data-ttu-id="590bc-199">Creare il file di definizione del servizio:</span><span class="sxs-lookup"><span data-stu-id="590bc-199">Create the service definition file:</span></span>

```bash
sudo nano /etc/systemd/system/kestrel-helloapp.service
```

<span data-ttu-id="590bc-200">L'esempio seguente è un file del servizio per l'app:</span><span class="sxs-lookup"><span data-stu-id="590bc-200">The following example is a service file for the app:</span></span>

```ini
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/www/helloapp
ExecStart=/usr/bin/dotnet /var/www/helloapp/helloapp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

<span data-ttu-id="590bc-201">Nell'esempio precedente, l'utente che gestisce il servizio viene specificato dall' `User` opzione.</span><span class="sxs-lookup"><span data-stu-id="590bc-201">In the preceding example, the user that manages the service is specified by the `User` option.</span></span> <span data-ttu-id="590bc-202">L'utente ( `www-data` ) deve esistere e avere la proprietà appropriata dei file dell'app.</span><span class="sxs-lookup"><span data-stu-id="590bc-202">The user (`www-data`) must exist and have proper ownership of the app's files.</span></span>

<span data-ttu-id="590bc-203">Usare `TimeoutStopSec` per configurare il tempo di attesa prima che l'app si arresti dopo aver ricevuto il segnale di interrupt iniziale.</span><span class="sxs-lookup"><span data-stu-id="590bc-203">Use `TimeoutStopSec` to configure the duration of time to wait for the app to shut down after it receives the initial interrupt signal.</span></span> <span data-ttu-id="590bc-204">Se l'app non si arresta in questo periodo, viene emesso il comando SIGKILL per terminare l'app.</span><span class="sxs-lookup"><span data-stu-id="590bc-204">If the app doesn't shut down in this period, SIGKILL is issued to terminate the app.</span></span> <span data-ttu-id="590bc-205">Specificare il valore in secondi senza unità di misura (ad esempio, `150`), un valore per l'intervallo di tempo (ad esempio, `2min 30s`) o `infinity` per disabilitare il timeout.</span><span class="sxs-lookup"><span data-stu-id="590bc-205">Provide the value as unitless seconds (for example, `150`), a time span value (for example, `2min 30s`), or `infinity` to disable the timeout.</span></span> <span data-ttu-id="590bc-206">`TimeoutStopSec` il valore predefinito è il valore di `DefaultTimeoutStopSec` nel file di configurazione Manager ( `systemd-system.conf` ,, `system.conf.d` `systemd-user.conf` , `user.conf.d` ).</span><span class="sxs-lookup"><span data-stu-id="590bc-206">`TimeoutStopSec` defaults to the value of `DefaultTimeoutStopSec` in the manager configuration file (`systemd-system.conf`, `system.conf.d`, `systemd-user.conf`, `user.conf.d`).</span></span> <span data-ttu-id="590bc-207">Il timeout predefinito per la maggior parte delle distribuzioni è di 90 secondi.</span><span class="sxs-lookup"><span data-stu-id="590bc-207">The default timeout for most distributions is 90 seconds.</span></span>

```
# The default value is 90 seconds for most distributions.
TimeoutStopSec=90
```

<span data-ttu-id="590bc-208">Linux dispone di un file system con distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="590bc-208">Linux has a case-sensitive file system.</span></span> <span data-ttu-id="590bc-209">Se `ASPNETCORE_ENVIRONMENT` si imposta su `Production` , la ricerca del file di configurazione non viene eseguita `appsettings.Production.json` `appsettings.production.json` .</span><span class="sxs-lookup"><span data-stu-id="590bc-209">Setting `ASPNETCORE_ENVIRONMENT` to `Production` results in searching for the configuration file `appsettings.Production.json`, not `appsettings.production.json`.</span></span>

<span data-ttu-id="590bc-210">Per alcuni valori (ad esempio, le stringhe di connessione SQL) è necessario usare caratteri di escape, in modo da consentire ai provider di configurazione di leggere le variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="590bc-210">Some values (for example, SQL connection strings) must be escaped for the configuration providers to read the environment variables.</span></span> <span data-ttu-id="590bc-211">Usare il comando seguente per generare un valore con caratteri di escape corretti per l'uso nel file di configurazione:</span><span class="sxs-lookup"><span data-stu-id="590bc-211">Use the following command to generate a properly escaped value for use in the configuration file:</span></span>

```console
systemd-escape "<value-to-escape>"
```

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="590bc-212">I due punti (`:`) separatori non sono supportati nei nomi delle variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="590bc-212">Colon (`:`) separators aren't supported in environment variable names.</span></span> <span data-ttu-id="590bc-213">Usare un doppio carattere di sottolineatura (`__`) al posto dei due punti.</span><span class="sxs-lookup"><span data-stu-id="590bc-213">Use a double underscore (`__`) in place of a colon.</span></span> <span data-ttu-id="590bc-214">Il [provider di configurazione delle variabili di ambiente](xref:fundamentals/configuration/index#environment-variables) converte le doppie sottolineature in due punti quando le variabili di ambiente vengono lette nella configurazione.</span><span class="sxs-lookup"><span data-stu-id="590bc-214">The [Environment Variables configuration provider](xref:fundamentals/configuration/index#environment-variables) converts double-underscores into colons when environment variables are read into configuration.</span></span> <span data-ttu-id="590bc-215">Nell'esempio seguente la chiave della stringa di connessione `ConnectionStrings:DefaultConnection` è impostata nel file di definizione del servizio come `ConnectionStrings__DefaultConnection`:</span><span class="sxs-lookup"><span data-stu-id="590bc-215">In the following example, the connection string key `ConnectionStrings:DefaultConnection` is set into the service definition file as `ConnectionStrings__DefaultConnection`:</span></span>

::: moniker-end
::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="590bc-216">I due punti (`:`) separatori non sono supportati nei nomi delle variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="590bc-216">Colon (`:`) separators aren't supported in environment variable names.</span></span> <span data-ttu-id="590bc-217">Usare un doppio carattere di sottolineatura (`__`) al posto dei due punti.</span><span class="sxs-lookup"><span data-stu-id="590bc-217">Use a double underscore (`__`) in place of a colon.</span></span> <span data-ttu-id="590bc-218">Il [provider di configurazione delle variabili di ambiente](xref:fundamentals/configuration/index#environment-variables-configuration-provider) converte le doppie sottolineature in due punti quando le variabili di ambiente vengono lette nella configurazione.</span><span class="sxs-lookup"><span data-stu-id="590bc-218">The [Environment Variables configuration provider](xref:fundamentals/configuration/index#environment-variables-configuration-provider) converts double-underscores into colons when environment variables are read into configuration.</span></span> <span data-ttu-id="590bc-219">Nell'esempio seguente la chiave della stringa di connessione `ConnectionStrings:DefaultConnection` è impostata nel file di definizione del servizio come `ConnectionStrings__DefaultConnection`:</span><span class="sxs-lookup"><span data-stu-id="590bc-219">In the following example, the connection string key `ConnectionStrings:DefaultConnection` is set into the service definition file as `ConnectionStrings__DefaultConnection`:</span></span>

::: moniker-end

```
Environment=ConnectionStrings__DefaultConnection={Connection String}
```

<span data-ttu-id="590bc-220">Salvare il file e abilitare il servizio.</span><span class="sxs-lookup"><span data-stu-id="590bc-220">Save the file and enable the service.</span></span>

```bash
sudo systemctl enable kestrel-helloapp.service
```

<span data-ttu-id="590bc-221">Avviare il servizio e verificare che sia in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="590bc-221">Start the service and verify that it's running.</span></span>

```
sudo systemctl start kestrel-helloapp.service
sudo systemctl status kestrel-helloapp.service

◝ kestrel-helloapp.service - Example .NET Web API App running on Ubuntu
    Loaded: loaded (/etc/systemd/system/kestrel-helloapp.service; enabled)
    Active: active (running) since Thu 2016-10-18 04:09:35 NZDT; 35s ago
Main PID: 9021 (dotnet)
    CGroup: /system.slice/kestrel-helloapp.service
            └─9021 /usr/local/bin/dotnet /var/www/helloapp/helloapp.dll
```

<span data-ttu-id="590bc-222">Con il proxy inverso configurato e il gheppio gestito tramite `systemd` , l'app Web è completamente configurata ed è accessibile da un browser nel computer locale all'indirizzo `http://localhost` .</span><span class="sxs-lookup"><span data-stu-id="590bc-222">With the reverse proxy configured and Kestrel managed through `systemd`, the web app is fully configured and can be accessed from a browser on the local machine at `http://localhost`.</span></span> <span data-ttu-id="590bc-223">È anche possibile accedervi da un computer remoto, escludendo eventuali firewall che potrebbero impedirlo.</span><span class="sxs-lookup"><span data-stu-id="590bc-223">It's also accessible from a remote machine, barring any firewall that might be blocking.</span></span> <span data-ttu-id="590bc-224">Se si esaminano le intestazioni di risposta, l'intestazione `Server` indica che l'app ASP.NET Core è gestita da Kestrel.</span><span class="sxs-lookup"><span data-stu-id="590bc-224">Inspecting the response headers, the `Server` header shows the ASP.NET Core app being served by Kestrel.</span></span>

```text
HTTP/1.1 200 OK
Date: Tue, 11 Oct 2016 16:22:23 GMT
Server: Kestrel
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
Transfer-Encoding: chunked
```

### <a name="view-logs"></a><span data-ttu-id="590bc-225">Visualizzare i log</span><span class="sxs-lookup"><span data-stu-id="590bc-225">View logs</span></span>

<span data-ttu-id="590bc-226">Poiché l'app Web che usa Kestrel viene gestita con `systemd`, tutti gli eventi e i processi vengono registrati in un giornale di registrazione centralizzato.</span><span class="sxs-lookup"><span data-stu-id="590bc-226">Since the web app using Kestrel is managed using `systemd`, all events and processes are logged to a centralized journal.</span></span> <span data-ttu-id="590bc-227">Tuttavia, questo giornale include tutte le voci per tutti i servizi e i processi gestiti da `systemd`.</span><span class="sxs-lookup"><span data-stu-id="590bc-227">However, this journal includes all entries for all services and processes managed by `systemd`.</span></span> <span data-ttu-id="590bc-228">Per visualizzare le voci specifiche di `kestrel-helloapp.service`, usare il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="590bc-228">To view the `kestrel-helloapp.service`-specific items, use the following command:</span></span>

```bash
sudo journalctl -fu kestrel-helloapp.service
```

<span data-ttu-id="590bc-229">Per un ulteriore filtro, le opzioni relative all'ora, ad esempio `--since today` , `--until 1 hour ago` o una combinazione di questi elementi possono ridurre il numero di voci restituite.</span><span class="sxs-lookup"><span data-stu-id="590bc-229">For further filtering, time options such as `--since today`, `--until 1 hour ago`, or a combination of these can reduce the number of entries returned.</span></span>

```bash
sudo journalctl -fu kestrel-helloapp.service --since "2016-10-18" --until "2016-10-18 04:00"
```

## <a name="data-protection"></a><span data-ttu-id="590bc-230">Protezione dei dati</span><span class="sxs-lookup"><span data-stu-id="590bc-230">Data protection</span></span>

<span data-ttu-id="590bc-231">Lo [stack di protezione dei dati ASP.NET Core](xref:security/data-protection/introduction) viene usato da diversi [middleware](xref:fundamentals/middleware/index)di ASP.NET Core, tra cui il middleware di autenticazione (ad esempio, cookie middleware) e le protezioni di richiesta tra siti (CSRF).</span><span class="sxs-lookup"><span data-stu-id="590bc-231">The [ASP.NET Core Data Protection stack](xref:security/data-protection/introduction) is used by several ASP.NET Core [middlewares](xref:fundamentals/middleware/index), including authentication middleware (for example, cookie middleware) and cross-site request forgery (CSRF) protections.</span></span> <span data-ttu-id="590bc-232">Anche se le DPAPI (Data Protection API) non vengono chiamate dal codice dell'utente, è consigliabile configurare la protezione dati per la creazione di un [archivio di chiavi](xref:security/data-protection/implementation/key-management) crittografiche permanente.</span><span class="sxs-lookup"><span data-stu-id="590bc-232">Even if Data Protection APIs aren't called by user code, data protection should be configured to create a persistent cryptographic [key store](xref:security/data-protection/implementation/key-management).</span></span> <span data-ttu-id="590bc-233">Se non si configura la protezione dei dati, le chiavi vengono mantenute in memoria ed eliminate al riavvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="590bc-233">If data protection isn't configured, the keys are held in memory and discarded when the app restarts.</span></span>

<span data-ttu-id="590bc-234">Se il gruppo di chiavi viene archiviato in memoria quando l'app viene riavviata:</span><span class="sxs-lookup"><span data-stu-id="590bc-234">If the key ring is stored in memory when the app restarts:</span></span>

* <span data-ttu-id="590bc-235">Tutti i cookie token di autenticazione basati su vengono invalidati.</span><span class="sxs-lookup"><span data-stu-id="590bc-235">All cookie-based authentication tokens are invalidated.</span></span>
* <span data-ttu-id="590bc-236">Gli utenti devono ripetere l'accesso alla richiesta successiva.</span><span class="sxs-lookup"><span data-stu-id="590bc-236">Users are required to sign in again on their next request.</span></span>
* <span data-ttu-id="590bc-237">Tutti i dati protetti con il gruppo di chiavi non possono più essere decrittografati.</span><span class="sxs-lookup"><span data-stu-id="590bc-237">Any data protected with the key ring can no longer be decrypted.</span></span> <span data-ttu-id="590bc-238">Questo può includere [i token CSRF](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) e [ASP.NET Core TempData MVC cookie ](xref:fundamentals/app-state#tempdata).</span><span class="sxs-lookup"><span data-stu-id="590bc-238">This may include [CSRF tokens](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) and [ASP.NET Core MVC TempData cookies](xref:fundamentals/app-state#tempdata).</span></span>

<span data-ttu-id="590bc-239">Per configurare la protezione dei dati in modo da rendere persistente il gruppo di chiavi e crittografarlo, vedere:</span><span class="sxs-lookup"><span data-stu-id="590bc-239">To configure data protection to persist and encrypt the key ring, see:</span></span>

* <xref:security/data-protection/implementation/key-storage-providers>
* <xref:security/data-protection/implementation/key-encryption-at-rest>

## <a name="long-request-header-fields"></a><span data-ttu-id="590bc-240">Campi di intestazione della richiesta di grandi dimensioni</span><span class="sxs-lookup"><span data-stu-id="590bc-240">Long request header fields</span></span>

<span data-ttu-id="590bc-241">Le impostazioni predefinite del server proxy limitano in genere i campi di intestazione della richiesta a 4 K o 8 K a seconda della piattaforma.</span><span class="sxs-lookup"><span data-stu-id="590bc-241">Proxy server default settings typically limit request header fields to 4 K or 8 K depending on the platform.</span></span> <span data-ttu-id="590bc-242">Un'app può richiedere campi più lunghi del valore predefinito, ad esempio le app che usano [Azure Active Directory](https://azure.microsoft.com/services/active-directory/).</span><span class="sxs-lookup"><span data-stu-id="590bc-242">An app may require fields longer than the default (for example, apps that use [Azure Active Directory](https://azure.microsoft.com/services/active-directory/)).</span></span> <span data-ttu-id="590bc-243">Se sono necessari campi più lunghi, le impostazioni predefinite del server proxy richiedono la regolazione.</span><span class="sxs-lookup"><span data-stu-id="590bc-243">If longer fields are required, the proxy server's default settings require adjustment.</span></span> <span data-ttu-id="590bc-244">I valori da applicare dipendono dallo scenario.</span><span class="sxs-lookup"><span data-stu-id="590bc-244">The values to apply depend on the scenario.</span></span> <span data-ttu-id="590bc-245">Per altre informazioni, vedere la documentazione del server.</span><span class="sxs-lookup"><span data-stu-id="590bc-245">For more information, see your server's documentation.</span></span>

* [<span data-ttu-id="590bc-246">proxy_buffer_size</span><span class="sxs-lookup"><span data-stu-id="590bc-246">proxy_buffer_size</span></span>](https://nginx.org/docs/http/ngx_http_proxy_module.html#proxy_buffer_size)
* [<span data-ttu-id="590bc-247">proxy_buffers</span><span class="sxs-lookup"><span data-stu-id="590bc-247">proxy_buffers</span></span>](https://nginx.org/docs/http/ngx_http_proxy_module.html#proxy_buffers)
* [<span data-ttu-id="590bc-248">proxy_busy_buffers_size</span><span class="sxs-lookup"><span data-stu-id="590bc-248">proxy_busy_buffers_size</span></span>](https://nginx.org/docs/http/ngx_http_proxy_module.html#proxy_busy_buffers_size)
* [<span data-ttu-id="590bc-249">large_client_header_buffers</span><span class="sxs-lookup"><span data-stu-id="590bc-249">large_client_header_buffers</span></span>](https://nginx.org/docs/http/ngx_http_core_module.html#large_client_header_buffers)

> [!WARNING]
> <span data-ttu-id="590bc-250">Aumentare i valori predefiniti dei buffer proxy solo se necessario.</span><span class="sxs-lookup"><span data-stu-id="590bc-250">Don't increase the default values of proxy buffers unless necessary.</span></span> <span data-ttu-id="590bc-251">Se si aumentano questi valori, si aumenta il rischio di sovraccarico del buffer (overflow) e di attacchi Denial of Service (DoS) da parte di utenti malintenzionati.</span><span class="sxs-lookup"><span data-stu-id="590bc-251">Increasing these values increases the risk of buffer overrun (overflow) and Denial of Service (DoS) attacks by malicious users.</span></span>

## <a name="secure-the-app"></a><span data-ttu-id="590bc-252">Proteggere l'app</span><span class="sxs-lookup"><span data-stu-id="590bc-252">Secure the app</span></span>

### <a name="enable-apparmor"></a><span data-ttu-id="590bc-253">Abilitare AppArmor</span><span class="sxs-lookup"><span data-stu-id="590bc-253">Enable AppArmor</span></span>

<span data-ttu-id="590bc-254">Linux Security Modules (LSM) è un framework che fa parte del kernel Linux a partire da Linux 2.6.</span><span class="sxs-lookup"><span data-stu-id="590bc-254">Linux Security Modules (LSM) is a framework that's part of the Linux kernel since Linux 2.6.</span></span> <span data-ttu-id="590bc-255">LSM supporta diverse implementazioni di moduli di protezione.</span><span class="sxs-lookup"><span data-stu-id="590bc-255">LSM supports different implementations of security modules.</span></span> <span data-ttu-id="590bc-256">[AppArmor](https://wiki.ubuntu.com/AppArmor) è un LSM che implementa un sistema di controllo degli accessi obbligatorio, che consente di limitare il programma a un set limitato di risorse.</span><span class="sxs-lookup"><span data-stu-id="590bc-256">[AppArmor](https://wiki.ubuntu.com/AppArmor) is an LSM that implements a Mandatory Access Control system, which allows confining the program to a limited set of resources.</span></span> <span data-ttu-id="590bc-257">Verificare che AppArmor sia abilitato e configurato correttamente.</span><span class="sxs-lookup"><span data-stu-id="590bc-257">Ensure AppArmor is enabled and properly configured.</span></span>

### <a name="configure-the-firewall"></a><span data-ttu-id="590bc-258">Configurare il firewall</span><span class="sxs-lookup"><span data-stu-id="590bc-258">Configure the firewall</span></span>

<span data-ttu-id="590bc-259">Chiudere tutte le porte esterne che non sono in uso.</span><span class="sxs-lookup"><span data-stu-id="590bc-259">Close off all external ports that aren't in use.</span></span> <span data-ttu-id="590bc-260">Un firewall semplice (ufw) fornisce un front-end per `iptables` fornendo un'interfaccia della riga di comando per la configurazione del firewall.</span><span class="sxs-lookup"><span data-stu-id="590bc-260">Uncomplicated firewall (ufw) provides a front end for `iptables` by providing a CLI for configuring the firewall.</span></span>

> [!WARNING]
> <span data-ttu-id="590bc-261">Se non è configurato correttamente, un firewall impedisce l'accesso all'intero sistema.</span><span class="sxs-lookup"><span data-stu-id="590bc-261">A firewall will prevent access to the whole system if not configured correctly.</span></span> <span data-ttu-id="590bc-262">Se non si specifica la porta SSH corretta, non sarà possibile accedere al sistema se si usa SSH per la connessione.</span><span class="sxs-lookup"><span data-stu-id="590bc-262">Failure to specify the correct SSH port will effectively lock you out of the system if you are using SSH to connect to it.</span></span> <span data-ttu-id="590bc-263">Il numero di porta predefinito è 22.</span><span class="sxs-lookup"><span data-stu-id="590bc-263">The default port is 22.</span></span> <span data-ttu-id="590bc-264">Per altre informazioni, vedere l'[introduzione a ufw](https://help.ubuntu.com/community/UFW) e il [manuale](https://manpages.ubuntu.com/manpages/bionic/man8/ufw.8.html).</span><span class="sxs-lookup"><span data-stu-id="590bc-264">For more information, see the [introduction to ufw](https://help.ubuntu.com/community/UFW) and the [manual](https://manpages.ubuntu.com/manpages/bionic/man8/ufw.8.html).</span></span>

<span data-ttu-id="590bc-265">Installare `ufw` e configurarlo per consentire il traffico su tutte le porte necessarie.</span><span class="sxs-lookup"><span data-stu-id="590bc-265">Install `ufw` and configure it to allow traffic on any ports needed.</span></span>

```bash
sudo apt-get install ufw

sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw enable
```

### <a name="secure-nginx"></a><span data-ttu-id="590bc-266">Proteggere Nginx</span><span class="sxs-lookup"><span data-stu-id="590bc-266">Secure Nginx</span></span>

#### <a name="change-the-nginx-response-name"></a><span data-ttu-id="590bc-267">Modificare il nome di risposta Nginx</span><span class="sxs-lookup"><span data-stu-id="590bc-267">Change the Nginx response name</span></span>

<span data-ttu-id="590bc-268">Modificare `src/http/ngx_http_header_filter_module.c`:</span><span class="sxs-lookup"><span data-stu-id="590bc-268">Edit `src/http/ngx_http_header_filter_module.c`:</span></span>

```
static char ngx_http_server_string[] = "Server: Web Server" CRLF;
static char ngx_http_server_full_string[] = "Server: Web Server" CRLF;
```

#### <a name="configure-options"></a><span data-ttu-id="590bc-269">Configurare le opzioni</span><span class="sxs-lookup"><span data-stu-id="590bc-269">Configure options</span></span>

<span data-ttu-id="590bc-270">Configurare il server con moduli aggiuntivi obbligatori.</span><span class="sxs-lookup"><span data-stu-id="590bc-270">Configure the server with additional required modules.</span></span> <span data-ttu-id="590bc-271">Può essere utile usare un firewall per app Web, ad esempio [ModSecurity](https://www.modsecurity.org/), per la protezione avanzata dell'app.</span><span class="sxs-lookup"><span data-stu-id="590bc-271">Consider using a web app firewall, such as [ModSecurity](https://www.modsecurity.org/), to harden the app.</span></span>

#### <a name="https-configuration"></a><span data-ttu-id="590bc-272">Configurazione HTTPS</span><span class="sxs-lookup"><span data-stu-id="590bc-272">HTTPS configuration</span></span>

<span data-ttu-id="590bc-273">**Configurare l'app per connessioni locali sicure (HTTPS)**</span><span class="sxs-lookup"><span data-stu-id="590bc-273">**Configure the app for secure (HTTPS) local connections**</span></span>

<span data-ttu-id="590bc-274">Il comando [DotNet Run](/dotnet/core/tools/dotnet-run) usa le *Proprietà/launchSettings.js* dell'app nel file, che configura l'app in modo che sia in ascolto sugli URL forniti dalla `applicationUrl` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="590bc-274">The [dotnet run](/dotnet/core/tools/dotnet-run) command uses the app's *Properties/launchSettings.json* file, which configures the app to listen on the URLs provided by the `applicationUrl` property.</span></span> <span data-ttu-id="590bc-275">Ad esempio: `https://localhost:5001;http://localhost:5000`.</span><span class="sxs-lookup"><span data-stu-id="590bc-275">For example, `https://localhost:5001;http://localhost:5000`.</span></span>

<span data-ttu-id="590bc-276">Configurare l'app per l'uso di un certificato in fase di sviluppo per il `dotnet run` comando o l'ambiente di sviluppo (<kbd>F5</kbd> o <kbd>CTRL</kbd> + <kbd>F5</kbd> in Visual Studio Code) usando uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="590bc-276">Configure the app to use a certificate in development for the `dotnet run` command or development environment (<kbd>F5</kbd> or <kbd>Ctrl</kbd>+<kbd>F5</kbd> in Visual Studio Code) using one of the following approaches:</span></span>

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="590bc-277">[Sostituire il certificato predefinito della configurazione](xref:fundamentals/servers/kestrel/endpoints#configuration) (*consigliato*)</span><span class="sxs-lookup"><span data-stu-id="590bc-277">[Replace the default certificate from configuration](xref:fundamentals/servers/kestrel/endpoints#configuration) (*Recommended*)</span></span>
* [<span data-ttu-id="590bc-278">KestrelServerOptions.ConfigureHttpsDefaults</span><span class="sxs-lookup"><span data-stu-id="590bc-278">KestrelServerOptions.ConfigureHttpsDefaults</span></span>](xref:fundamentals/servers/kestrel/endpoints#configurehttpsdefaultsactionhttpsconnectionadapteroptions)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <span data-ttu-id="590bc-279">[Sostituire il certificato predefinito della configurazione](xref:fundamentals/servers/kestrel#configuration) (*consigliato*)</span><span class="sxs-lookup"><span data-stu-id="590bc-279">[Replace the default certificate from configuration](xref:fundamentals/servers/kestrel#configuration) (*Recommended*)</span></span>
* [<span data-ttu-id="590bc-280">KestrelServerOptions.ConfigureHttpsDefaults</span><span class="sxs-lookup"><span data-stu-id="590bc-280">KestrelServerOptions.ConfigureHttpsDefaults</span></span>](xref:fundamentals/servers/kestrel#configurehttpsdefaultsactionhttpsconnectionadapteroptions)

::: moniker-end

<span data-ttu-id="590bc-281">**Configurare il proxy inverso per connessioni client sicure (HTTPS)**</span><span class="sxs-lookup"><span data-stu-id="590bc-281">**Configure the reverse proxy for secure (HTTPS) client connections**</span></span>

* <span data-ttu-id="590bc-282">Configurare il server per l'ascolto del traffico HTTPS sulla porta 443 specificando un certificato valido emesso da un'autorità di certificazione (CA) attendibile.</span><span class="sxs-lookup"><span data-stu-id="590bc-282">Configure the server to listen to HTTPS traffic on port 443 by specifying a valid certificate issued by a trusted Certificate Authority (CA).</span></span>

* <span data-ttu-id="590bc-283">Rafforzare la protezione adottando alcune delle procedure descritte nel file */etc/nginx/nginx.conf* che segue.</span><span class="sxs-lookup"><span data-stu-id="590bc-283">Harden the security by employing some of the practices depicted in the following */etc/nginx/nginx.conf* file.</span></span> <span data-ttu-id="590bc-284">Gli esempi includono la scelta di una crittografia più complessa e il reindirizzamento di tutto il traffico su HTTP a HTTPS.</span><span class="sxs-lookup"><span data-stu-id="590bc-284">Examples include choosing a stronger cipher and redirecting all traffic over HTTP to HTTPS.</span></span>

  > [!NOTE]
  > <span data-ttu-id="590bc-285">Per gli ambienti di sviluppo, è consigliabile usare reindirizzamenti temporanei (302) invece dei reindirizzamenti permanenti (301).</span><span class="sxs-lookup"><span data-stu-id="590bc-285">For development environments, we recommend using temporary redirects (302) rather than permanent redirects (301).</span></span> <span data-ttu-id="590bc-286">La memorizzazione nella cache dei collegamenti può causare un comportamento instabile negli ambienti di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="590bc-286">Link caching can cause unstable behavior in development environments.</span></span>

* <span data-ttu-id="590bc-287">L'aggiunta di un'intestazione `HTTP Strict-Transport-Security` (HSTS) garantisce che tutte le richieste successive vengano effettuate dal client via HTTPS.</span><span class="sxs-lookup"><span data-stu-id="590bc-287">Adding an `HTTP Strict-Transport-Security` (HSTS) header ensures all subsequent requests made by the client are over HTTPS.</span></span>

  <span data-ttu-id="590bc-288">Per informazioni importanti su HSTS, vedere <xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts> .</span><span class="sxs-lookup"><span data-stu-id="590bc-288">For important guidance on HSTS, see <xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts>.</span></span>

* <span data-ttu-id="590bc-289">Se HTTPS sarà disabilitato in futuro, usare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="590bc-289">If HTTPS will be disabled in the future, use one of the following approaches:</span></span>

  * <span data-ttu-id="590bc-290">Non aggiungere l'intestazione HSTS.</span><span class="sxs-lookup"><span data-stu-id="590bc-290">Don't add the HSTS header.</span></span>
  * <span data-ttu-id="590bc-291">Scegliere un `max-age` valore breve.</span><span class="sxs-lookup"><span data-stu-id="590bc-291">Choose a short `max-age` value.</span></span>

<span data-ttu-id="590bc-292">Aggiungere il file di configurazione */etc/nginx/proxy.conf*:</span><span class="sxs-lookup"><span data-stu-id="590bc-292">Add the */etc/nginx/proxy.conf* configuration file:</span></span>

[!code-nginx[](linux-nginx/proxy.conf)]

<span data-ttu-id="590bc-293">**Sostituire** il contenuto del file di configurazione */etc/nginx/nginx.conf* con il file seguente.</span><span class="sxs-lookup"><span data-stu-id="590bc-293">**Replace** the contents of the */etc/nginx/nginx.conf* configuration file with the following file.</span></span> <span data-ttu-id="590bc-294">L'esempio contiene entrambe le sezioni `http` e `server` in un unico file di configurazione.</span><span class="sxs-lookup"><span data-stu-id="590bc-294">The example contains both `http` and `server` sections in one configuration file.</span></span>

[!code-nginx[](linux-nginx/nginx.conf?highlight=2)]

> [!NOTE]
> <span data-ttu-id="590bc-295">Blazor WebAssembly le app richiedono un `burst` valore di parametro maggiore per contenere il numero maggiore di richieste effettuate da un'app.</span><span class="sxs-lookup"><span data-stu-id="590bc-295">Blazor WebAssembly apps require a larger `burst` parameter value to accommodate the larger number of requests made by an app.</span></span> <span data-ttu-id="590bc-296">Per altre informazioni, vedere <xref:blazor/host-and-deploy/webassembly#nginx>.</span><span class="sxs-lookup"><span data-stu-id="590bc-296">For more information, see <xref:blazor/host-and-deploy/webassembly#nginx>.</span></span>

#### <a name="secure-nginx-from-clickjacking"></a><span data-ttu-id="590bc-297">Proteggere Nginx dal clickjacking</span><span class="sxs-lookup"><span data-stu-id="590bc-297">Secure Nginx from clickjacking</span></span>

<span data-ttu-id="590bc-298">Il [clickjacking](https://blog.qualys.com/securitylabs/2015/10/20/clickjacking-a-common-implementation-mistake-that-can-put-your-websites-in-danger), anche noto come *attacco UI Redress*, è un attacco dannoso in cui un visitatore di un sito Web viene indotto a fare clic su un collegamento o un pulsante in una pagina diversa da quella che sta attualmente visualizzando.</span><span class="sxs-lookup"><span data-stu-id="590bc-298">[Clickjacking](https://blog.qualys.com/securitylabs/2015/10/20/clickjacking-a-common-implementation-mistake-that-can-put-your-websites-in-danger), also known as a *UI redress attack*, is a malicious attack where a website visitor is tricked into clicking a link or button on a different page than they're currently visiting.</span></span> <span data-ttu-id="590bc-299">Usare `X-FRAME-OPTIONS` per proteggere il sito.</span><span class="sxs-lookup"><span data-stu-id="590bc-299">Use `X-FRAME-OPTIONS` to secure the site.</span></span>

<span data-ttu-id="590bc-300">Per contrastare gli attacchi di clickjacking:</span><span class="sxs-lookup"><span data-stu-id="590bc-300">To mitigate clickjacking attacks:</span></span>

1. <span data-ttu-id="590bc-301">Modificare il file *nginx.conf*:</span><span class="sxs-lookup"><span data-stu-id="590bc-301">Edit the *nginx.conf* file:</span></span>

   ```bash
   sudo nano /etc/nginx/nginx.conf
   ```

   <span data-ttu-id="590bc-302">Aggiungere la riga: `add_header X-Frame-Options "SAMEORIGIN";`</span><span class="sxs-lookup"><span data-stu-id="590bc-302">Add the line: `add_header X-Frame-Options "SAMEORIGIN";`</span></span>

1. <span data-ttu-id="590bc-303">Salvare il file.</span><span class="sxs-lookup"><span data-stu-id="590bc-303">Save the file.</span></span>
1. <span data-ttu-id="590bc-304">Riavviare Nginx.</span><span class="sxs-lookup"><span data-stu-id="590bc-304">Restart Nginx.</span></span>

#### <a name="mime-type-sniffing"></a><span data-ttu-id="590bc-305">Analisi del tipo MIME</span><span class="sxs-lookup"><span data-stu-id="590bc-305">MIME-type sniffing</span></span>

<span data-ttu-id="590bc-306">Questa intestazione impedisce alla maggior parte dei browser di dedurre dall'analisi del tipo MIME un tipo di contenuto diverso da quello dichiarato, poiché l'intestazione indica al browser di non eseguire l'override del tipo di contenuto della risposta.</span><span class="sxs-lookup"><span data-stu-id="590bc-306">This header prevents most browsers from MIME-sniffing a response away from the declared content type, as the header instructs the browser not to override the response content type.</span></span> <span data-ttu-id="590bc-307">Con l' `nosniff` opzione, se il server indica che il contenuto è `text/html` , il browser ne esegue il rendering come `text/html` .</span><span class="sxs-lookup"><span data-stu-id="590bc-307">With the `nosniff` option, if the server says the content is `text/html`, the browser renders it as `text/html`.</span></span>

1. <span data-ttu-id="590bc-308">Modificare il file *nginx.conf*:</span><span class="sxs-lookup"><span data-stu-id="590bc-308">Edit the *nginx.conf* file:</span></span>

   ```bash
   sudo nano /etc/nginx/nginx.conf
   ```

   <span data-ttu-id="590bc-309">Aggiungere la riga: `add_header X-Content-Type-Options "nosniff";`</span><span class="sxs-lookup"><span data-stu-id="590bc-309">Add the line: `add_header X-Content-Type-Options "nosniff";`</span></span>

1. <span data-ttu-id="590bc-310">Salvare il file.</span><span class="sxs-lookup"><span data-stu-id="590bc-310">Save the file.</span></span>
1. <span data-ttu-id="590bc-311">Riavviare Nginx.</span><span class="sxs-lookup"><span data-stu-id="590bc-311">Restart Nginx.</span></span>

## <a name="additional-nginx-suggestions"></a><span data-ttu-id="590bc-312">Suggerimenti nginx aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="590bc-312">Additional Nginx suggestions</span></span>

<span data-ttu-id="590bc-313">Dopo aver aggiornato il Framework condiviso sul server, riavviare le app ASP.NET Core ospitate dal server.</span><span class="sxs-lookup"><span data-stu-id="590bc-313">After upgrading the shared framework on the server, restart the ASP.NET Core apps hosted by the server.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="590bc-314">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="590bc-314">Additional resources</span></span>

* [<span data-ttu-id="590bc-315">Prerequisiti per .NET Core in Linux</span><span class="sxs-lookup"><span data-stu-id="590bc-315">Prerequisites for .NET Core on Linux</span></span>](/dotnet/core/linux-prerequisites)
* <span data-ttu-id="590bc-316">[Nginx: Binary Releases: Official Debian/Ubuntu packages](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages) (Nginx: Versioni binarie: Pacchetti ufficiali Debian/Ubuntu)</span><span class="sxs-lookup"><span data-stu-id="590bc-316">[Nginx: Binary Releases: Official Debian/Ubuntu packages](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages)</span></span>
* <xref:test/troubleshoot>
* <xref:host-and-deploy/proxy-load-balancer>
* <span data-ttu-id="590bc-317">[NGINX: Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/) (NGINX: Uso dell'intestazione Forwarded)</span><span class="sxs-lookup"><span data-stu-id="590bc-317">[NGINX: Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/)</span></span>
