---
title: Ospitare e distribuire ASP.NET Core Blazor
author: guardrex
description: Scopri come ospitare e distribuire le Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/15/2020
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
uid: blazor/host-and-deploy/index
ms.openlocfilehash: e7bc44b396b46e2ac3e0279520c7cc8ea6679f5a
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279780"
---
# <a name="host-and-deploy-aspnet-core-blazor"></a><span data-ttu-id="8db00-103">Ospitare e distribuire ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="8db00-103">Host and deploy ASP.NET Core Blazor</span></span>

## <a name="publish-the-app"></a><span data-ttu-id="8db00-104">Pubblicare l'app</span><span class="sxs-lookup"><span data-stu-id="8db00-104">Publish the app</span></span>

<span data-ttu-id="8db00-105">Le app vengono pubblicate per la distribuzione nella configurazione per il rilascio.</span><span class="sxs-lookup"><span data-stu-id="8db00-105">Apps are published for deployment in Release configuration.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="8db00-106">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="8db00-106">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="8db00-107">Selezionare **Compila**  >  **pubblicazione {applicazione}** dalla barra di spostamento.</span><span class="sxs-lookup"><span data-stu-id="8db00-107">Select **Build** > **Publish {APPLICATION}** from the navigation bar.</span></span>
1. <span data-ttu-id="8db00-108">Selezionare la *destinazione di pubblicazione*.</span><span class="sxs-lookup"><span data-stu-id="8db00-108">Select the *publish target*.</span></span> <span data-ttu-id="8db00-109">Per pubblicare in locale, selezionare **Cartella**.</span><span class="sxs-lookup"><span data-stu-id="8db00-109">To publish locally, select **Folder**.</span></span>
1. <span data-ttu-id="8db00-110">Accettare il percorso predefinito nel campo **Scegliere una cartella** oppure specificarne uno diverso.</span><span class="sxs-lookup"><span data-stu-id="8db00-110">Accept the default location in the **Choose a folder** field or specify a different location.</span></span> <span data-ttu-id="8db00-111">Selezionare il pulsante **`Publish`**.</span><span class="sxs-lookup"><span data-stu-id="8db00-111">Select the **`Publish`** button.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="8db00-112">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="8db00-112">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="8db00-113">Selezionare **Compila**  >  **pubblicazione in cartella**.</span><span class="sxs-lookup"><span data-stu-id="8db00-113">Select **Build** > **Publish to Folder**.</span></span>
1. <span data-ttu-id="8db00-114">Confermare la cartella per ricevere gli asset pubblicati e selezionare **`Publish`** .</span><span class="sxs-lookup"><span data-stu-id="8db00-114">Confirm the folder to receive the published assets and select **`Publish`**.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="8db00-115">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="8db00-115">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="8db00-116">Usare il [`dotnet publish`](/dotnet/core/tools/dotnet-publish) comando per pubblicare l'app con una configurazione di rilascio:</span><span class="sxs-lookup"><span data-stu-id="8db00-116">Use the [`dotnet publish`](/dotnet/core/tools/dotnet-publish) command to publish the app with a Release configuration:</span></span>

```dotnetcli
dotnet publish -c Release
```

---

<span data-ttu-id="8db00-117">La pubblicazione dell'app attiva un [ripristino](/dotnet/core/tools/dotnet-restore) delle dipendenze del progetto e [compila](/dotnet/core/tools/dotnet-build) il progetto prima di creare gli asset per la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="8db00-117">Publishing the app triggers a [restore](/dotnet/core/tools/dotnet-restore) of the project's dependencies and [builds](/dotnet/core/tools/dotnet-build) the project before creating the assets for deployment.</span></span> <span data-ttu-id="8db00-118">Come parte del processo di compilazione, vengono rimossi gli assembly e i metodi non usati per ridurre le dimensioni del download e i tempi di caricamento dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-118">As part of the build process, unused methods and assemblies are removed to reduce app download size and load times.</span></span>

<span data-ttu-id="8db00-119">Percorsi di pubblicazione:</span><span class="sxs-lookup"><span data-stu-id="8db00-119">Publish locations:</span></span>

* Blazor WebAssembly
  * <span data-ttu-id="8db00-120">Autonomo: l'app viene pubblicata nella `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` cartella.</span><span class="sxs-lookup"><span data-stu-id="8db00-120">Standalone: The app is published into the `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` folder.</span></span> <span data-ttu-id="8db00-121">Per distribuire l'app come sito statico, copiare il contenuto della cartella nell' `wwwroot` host del sito statico.</span><span class="sxs-lookup"><span data-stu-id="8db00-121">To deploy the app as a static site, copy the contents of the `wwwroot` folder to the static site host.</span></span>
  * <span data-ttu-id="8db00-122">Hosted: l' Blazor WebAssembly app client viene pubblicata nella `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` cartella dell'app Server, insieme a eventuali altre risorse Web statiche dell'app Server.</span><span class="sxs-lookup"><span data-stu-id="8db00-122">Hosted: The client Blazor WebAssembly app is published into the `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` folder of the server app, along with any other static web assets of the server app.</span></span> <span data-ttu-id="8db00-123">Distribuire il contenuto della `publish` cartella nell'host.</span><span class="sxs-lookup"><span data-stu-id="8db00-123">Deploy the contents of the `publish` folder to the host.</span></span>
* <span data-ttu-id="8db00-124">Blazor Server: L'app viene pubblicata nella `/bin/Release/{TARGET FRAMEWORK}/publish` cartella.</span><span class="sxs-lookup"><span data-stu-id="8db00-124">Blazor Server: The app is published into the `/bin/Release/{TARGET FRAMEWORK}/publish` folder.</span></span> <span data-ttu-id="8db00-125">Distribuire il contenuto della `publish` cartella nell'host.</span><span class="sxs-lookup"><span data-stu-id="8db00-125">Deploy the contents of the `publish` folder to the host.</span></span>

<span data-ttu-id="8db00-126">Gli asset nella cartella vengono distribuiti nel server Web.</span><span class="sxs-lookup"><span data-stu-id="8db00-126">The assets in the folder are deployed to the web server.</span></span> <span data-ttu-id="8db00-127">La distribuzione potrebbe essere un processo manuale o automatizzato a seconda degli strumenti di sviluppo in uso.</span><span class="sxs-lookup"><span data-stu-id="8db00-127">Deployment might be a manual or automated process depending on the development tools in use.</span></span>

## <a name="app-base-path"></a><span data-ttu-id="8db00-128">Percorso di base dell'app</span><span class="sxs-lookup"><span data-stu-id="8db00-128">App base path</span></span>

<span data-ttu-id="8db00-129">Il *percorso di base dell'app* è il percorso dell'URL radice dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-129">The *app base path* is the app's root URL path.</span></span> <span data-ttu-id="8db00-130">Si considerino i seguenti ASP.NET Core app e l'app Blazor secondaria:</span><span class="sxs-lookup"><span data-stu-id="8db00-130">Consider the following ASP.NET Core app and Blazor sub-app:</span></span>

* <span data-ttu-id="8db00-131">L'app ASP.NET Core è denominata `MyApp` :</span><span class="sxs-lookup"><span data-stu-id="8db00-131">The ASP.NET Core app is named `MyApp`:</span></span>
  * <span data-ttu-id="8db00-132">L'app risiede fisicamente in `d:/MyApp` .</span><span class="sxs-lookup"><span data-stu-id="8db00-132">The app physically resides at `d:/MyApp`.</span></span>
  * <span data-ttu-id="8db00-133">Le richieste vengono ricevute all'indirizzo `https://www.contoso.com/{MYAPP RESOURCE}` .</span><span class="sxs-lookup"><span data-stu-id="8db00-133">Requests are received at `https://www.contoso.com/{MYAPP RESOURCE}`.</span></span>
* <span data-ttu-id="8db00-134">Un' Blazor App denominata `CoolApp` è un'app secondaria di `MyApp` :</span><span class="sxs-lookup"><span data-stu-id="8db00-134">A Blazor app named `CoolApp` is a sub-app of `MyApp`:</span></span>
  * <span data-ttu-id="8db00-135">La Sub-App risiede fisicamente in `d:/MyApp/CoolApp` .</span><span class="sxs-lookup"><span data-stu-id="8db00-135">The sub-app physically resides at `d:/MyApp/CoolApp`.</span></span>
  * <span data-ttu-id="8db00-136">Le richieste vengono ricevute all'indirizzo `https://www.contoso.com/CoolApp/{COOLAPP RESOURCE}` .</span><span class="sxs-lookup"><span data-stu-id="8db00-136">Requests are received at `https://www.contoso.com/CoolApp/{COOLAPP RESOURCE}`.</span></span>

<span data-ttu-id="8db00-137">Se non si specifica una configurazione aggiuntiva per `CoolApp` , la sottoapp in questo scenario non è a conoscenza della posizione in cui risiede nel server.</span><span class="sxs-lookup"><span data-stu-id="8db00-137">Without specifying additional configuration for `CoolApp`, the sub-app in this scenario has no knowledge of where it resides on the server.</span></span> <span data-ttu-id="8db00-138">Ad esempio, l'app non può costruire URL relativi corretti per le risorse senza sapere che risiede nel percorso URL relativo `/CoolApp/` .</span><span class="sxs-lookup"><span data-stu-id="8db00-138">For example, the app can't construct correct relative URLs to its resources without knowing that it resides at the relative URL path `/CoolApp/`.</span></span>

<span data-ttu-id="8db00-139">Per fornire la configurazione per il Blazor percorso di base dell'app `https://www.contoso.com/CoolApp/` , l' `<base>` attributo del tag `href` viene impostato sul percorso radice relativo nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ).</span><span class="sxs-lookup"><span data-stu-id="8db00-139">To provide configuration for the Blazor app's base path of `https://www.contoso.com/CoolApp/`, the `<base>` tag's `href` attribute is set to the relative root path in the `wwwroot/index.html` file (Blazor WebAssembly) or `Pages/_Host.cshtml` file (Blazor Server).</span></span>

<span data-ttu-id="8db00-140">Blazor WebAssembly (`wwwroot/index.html`):</span><span class="sxs-lookup"><span data-stu-id="8db00-140">Blazor WebAssembly (`wwwroot/index.html`):</span></span>

```html
<base href="/CoolApp/">
```

<span data-ttu-id="8db00-141">Blazor Server (`Pages/_Host.cshtml`):</span><span class="sxs-lookup"><span data-stu-id="8db00-141">Blazor Server (`Pages/_Host.cshtml`):</span></span>

```html
<base href="~/CoolApp/">
```

<span data-ttu-id="8db00-142">Blazor Server app consente inoltre di impostare il percorso di base lato server chiamando <xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*> nella pipeline delle richieste dell'app di `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="8db00-142">Blazor Server apps additionally set the server-side base path by calling <xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*> in the app's request pipeline of `Startup.Configure`:</span></span>

```csharp
app.UsePathBase("/CoolApp");
```

<span data-ttu-id="8db00-143">Fornendo il percorso URL relativo, un componente che non si trova nella directory radice può costruire URL relativi al percorso radice dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-143">By providing the relative URL path, a component that isn't in the root directory can construct URLs relative to the app's root path.</span></span> <span data-ttu-id="8db00-144">I componenti a livelli diversi della struttura di directory possono creare collegamenti ad altre risorse in posizioni all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-144">Components at different levels of the directory structure can build links to other resources at locations throughout the app.</span></span> <span data-ttu-id="8db00-145">Il percorso di base dell'app viene usato anche per intercettare i collegamenti ipertestuali selezionati in cui la `href` destinazione del collegamento si trova nello spazio URI del percorso di base dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-145">The app base path is also used to intercept selected hyperlinks where the `href` target of the link is within the app base path URI space.</span></span> <span data-ttu-id="8db00-146">Il Blazor router gestisce la navigazione interna.</span><span class="sxs-lookup"><span data-stu-id="8db00-146">The Blazor router handles the internal navigation.</span></span>

<span data-ttu-id="8db00-147">In molti scenari di hosting il percorso URL relativo dell'app è la radice dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-147">In many hosting scenarios, the relative URL path to the app is the root of the app.</span></span> <span data-ttu-id="8db00-148">In questi casi, il percorso di base dell'URL relativo dell'app è una barra ( `<base href="/" />` ), ovvero la configurazione predefinita per un' Blazor app.</span><span class="sxs-lookup"><span data-stu-id="8db00-148">In these cases, the app's relative URL base path is a forward slash (`<base href="/" />`), which is the default configuration for a Blazor app.</span></span> <span data-ttu-id="8db00-149">In altri scenari di hosting, ad esempio pagine di GitHub e sottoapp di IIS, il percorso di base dell'app deve essere impostato sul percorso URL relativo del server dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-149">In other hosting scenarios, such as GitHub Pages and IIS sub-apps, the app base path must be set to the server's relative URL path of the app.</span></span>

<span data-ttu-id="8db00-150">Per impostare il percorso di base dell'app, aggiornare il `<base>` tag negli `<head>` elementi tag del `Pages/_Host.cshtml` file ( Blazor Server ) o del `wwwroot/index.html` file ( Blazor WebAssembly ).</span><span class="sxs-lookup"><span data-stu-id="8db00-150">To set the app's base path, update the `<base>` tag within the `<head>` tag elements of the `Pages/_Host.cshtml` file (Blazor Server) or `wwwroot/index.html` file (Blazor WebAssembly).</span></span> <span data-ttu-id="8db00-151">Impostare il `href` valore dell'attributo su `/{RELATIVE URL PATH}/` ( Blazor WebAssembly ) o `~/{RELATIVE URL PATH}/` ( Blazor Server ).</span><span class="sxs-lookup"><span data-stu-id="8db00-151">Set the `href` attribute value to `/{RELATIVE URL PATH}/` (Blazor WebAssembly) or `~/{RELATIVE URL PATH}/` (Blazor Server).</span></span> <span data-ttu-id="8db00-152">**La barra finale è obbligatoria.**</span><span class="sxs-lookup"><span data-stu-id="8db00-152">**The trailing slash is required.**</span></span> <span data-ttu-id="8db00-153">Il segnaposto `{RELATIVE URL PATH}` è il percorso URL relativo completo dell'app.</span><span class="sxs-lookup"><span data-stu-id="8db00-153">The placeholder `{RELATIVE URL PATH}` is the app's full relative URL path.</span></span>

<span data-ttu-id="8db00-154">Per un' Blazor WebAssembly app con un percorso URL relativo non radice (ad esempio, `<base href="/CoolApp/">` ), l'app non riesce a trovare le proprie risorse *quando viene eseguita localmente*.</span><span class="sxs-lookup"><span data-stu-id="8db00-154">For a Blazor WebAssembly app with a non-root relative URL path (for example, `<base href="/CoolApp/">`), the app fails to find its resources *when run locally*.</span></span> <span data-ttu-id="8db00-155">Per risolvere questo problema durante sviluppo e test locali, è possibile specificare un argomento *base del percorso* corrispondente al valore `href` del tag `<base>` in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="8db00-155">To overcome this problem during local development and testing, you can supply a *path base* argument that matches the `href` value of the `<base>` tag at runtime.</span></span> <span data-ttu-id="8db00-156">**Non includere una barra finale.**</span><span class="sxs-lookup"><span data-stu-id="8db00-156">**Don't include a trailing slash.**</span></span> <span data-ttu-id="8db00-157">Per passare l'argomento di base del percorso quando si esegue l'app in locale, eseguire il `dotnet run` comando dalla directory dell'app con l' `--pathbase` opzione:</span><span class="sxs-lookup"><span data-stu-id="8db00-157">To pass the path base argument when running the app locally, execute the `dotnet run` command from the app's directory with the `--pathbase` option:</span></span>

```dotnetcli
dotnet run --pathbase=/{RELATIVE URL PATH (no trailing slash)}
```

<span data-ttu-id="8db00-158">Per un' Blazor WebAssembly app con un percorso URL relativo `/CoolApp/` ( `<base href="/CoolApp/">` ), il comando è:</span><span class="sxs-lookup"><span data-stu-id="8db00-158">For a Blazor WebAssembly app with a relative URL path of `/CoolApp/` (`<base href="/CoolApp/">`), the command is:</span></span>

```dotnetcli
dotnet run --pathbase=/CoolApp
```

<span data-ttu-id="8db00-159">L' Blazor WebAssembly app risponde localmente all'indirizzo `http://localhost:port/CoolApp` .</span><span class="sxs-lookup"><span data-stu-id="8db00-159">The Blazor WebAssembly app responds locally at `http://localhost:port/CoolApp`.</span></span>

<span data-ttu-id="8db00-160">**Blazor Server`MapFallbackToPage`configurazione di**</span><span class="sxs-lookup"><span data-stu-id="8db00-160">**Blazor Server `MapFallbackToPage` configuration**</span></span>

<span data-ttu-id="8db00-161">Passare il seguente percorso a <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A> in `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="8db00-161">Pass the following path to <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A> in `Startup.Configure`:</span></span>

```csharp
endpoints.MapFallbackToPage("/{RELATIVE PATH}/{**path:nonfile}");
```

<span data-ttu-id="8db00-162">Il segnaposto `{RELATIVE PATH}` è il percorso non radice sul server.</span><span class="sxs-lookup"><span data-stu-id="8db00-162">The placeholder `{RELATIVE PATH}` is the non-root path on the server.</span></span> <span data-ttu-id="8db00-163">Ad esempio, `CoolApp` è il segmento segnaposto se l'URL non radice dell'app è `https://{HOST}:{PORT}/CoolApp/` ):</span><span class="sxs-lookup"><span data-stu-id="8db00-163">For example, `CoolApp` is the placeholder segment if the non-root URL to the app is `https://{HOST}:{PORT}/CoolApp/`):</span></span>

```csharp
endpoints.MapFallbackToPage("/CoolApp/{**path:nonfile}");
```

<span data-ttu-id="8db00-164">**Ospita più Blazor WebAssembly app**</span><span class="sxs-lookup"><span data-stu-id="8db00-164">**Host multiple Blazor WebAssembly apps**</span></span>

<span data-ttu-id="8db00-165">Per ulteriori informazioni sull'hosting Blazor WebAssembly di più app in una Blazor soluzione ospitata, vedere <xref:blazor/host-and-deploy/webassembly#hosted-deployment-with-multiple-blazor-webassembly-apps> .</span><span class="sxs-lookup"><span data-stu-id="8db00-165">For more information on hosting multiple Blazor WebAssembly apps in a hosted Blazor solution, see <xref:blazor/host-and-deploy/webassembly#hosted-deployment-with-multiple-blazor-webassembly-apps>.</span></span>

## <a name="deployment"></a><span data-ttu-id="8db00-166">Distribuzione</span><span class="sxs-lookup"><span data-stu-id="8db00-166">Deployment</span></span>

<span data-ttu-id="8db00-167">Per indicazioni sulla distribuzione, vedere gli argomenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="8db00-167">For deployment guidance, see the following topics:</span></span>

* <xref:blazor/host-and-deploy/webassembly>
* <xref:blazor/host-and-deploy/server>
