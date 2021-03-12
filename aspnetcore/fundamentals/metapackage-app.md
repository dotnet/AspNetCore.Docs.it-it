---
title: Metapacchetto Microsoft. AspNetCore. app per ASP.NET Core
author: Rick-Anderson
description: Framework condiviso Microsoft. AspNetCore. app
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 09/24/2019
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
uid: fundamentals/metapackage-app
ms.openlocfilehash: 5d9d9cd446a61cc3e573712a4626af04dc284e99
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589582"
---
# <a name="microsoftaspnetcoreapp-for-aspnet-core"></a><span data-ttu-id="3b756-103">Microsoft. AspNetCore. app per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="3b756-103">Microsoft.AspNetCore.App for ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

 <span data-ttu-id="3b756-104">Il Framework condiviso ASP.NET Core ( `Microsoft.AspNetCore.App` ) contiene gli assembly sviluppati e supportati da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="3b756-104">The ASP.NET Core shared framework (`Microsoft.AspNetCore.App`) contains assemblies that are developed and supported by Microsoft.</span></span> <span data-ttu-id="3b756-105">`Microsoft.AspNetCore.App` viene installato quando è installato [.NET Core 3,0 o versione successiva SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) .</span><span class="sxs-lookup"><span data-stu-id="3b756-105">`Microsoft.AspNetCore.App` is installed when the [.NET Core 3.0 or later SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) is installed.</span></span> <span data-ttu-id="3b756-106">Il *Framework condiviso* è il set di assembly (file con *estensione dll* ) installato nel computer e include un componente di runtime e un Targeting Pack.</span><span class="sxs-lookup"><span data-stu-id="3b756-106">The *shared framework* is the set of assemblies (*.dll* files) that are installed on the machine and includes a runtime component and a targeting pack.</span></span> <span data-ttu-id="3b756-107">Per altre informazioni, vedere [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/) (Il framework condiviso).</span><span class="sxs-lookup"><span data-stu-id="3b756-107">For more information, see [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/).</span></span>

* <span data-ttu-id="3b756-108">I progetti destinati all' `Microsoft.NET.Sdk.Web` SDK fanno riferimento in modo implicito al `Microsoft.AspNetCore.App` Framework.</span><span class="sxs-lookup"><span data-stu-id="3b756-108">Projects that target the `Microsoft.NET.Sdk.Web` SDK implicitly reference the `Microsoft.AspNetCore.App` framework.</span></span>

<span data-ttu-id="3b756-109">Per questi progetti non sono necessari riferimenti aggiuntivi:</span><span class="sxs-lookup"><span data-stu-id="3b756-109">No additional references are required for these projects:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>
    ...
</Project>
```

<span data-ttu-id="3b756-110">Il Framework condiviso ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="3b756-110">The ASP.NET Core shared framework:</span></span>

* <span data-ttu-id="3b756-111">Non include dipendenze di terze parti.</span><span class="sxs-lookup"><span data-stu-id="3b756-111">Doesn't include third-party dependencies.</span></span>
* <span data-ttu-id="3b756-112">Include tutti i pacchetti supportati dal team di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="3b756-112">Includes all supported packages by the ASP.NET Core team.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="3b756-113">Questa funzionalità richiede ASP.NET Core 2.x con destinazione .NET Core 2.x.</span><span class="sxs-lookup"><span data-stu-id="3b756-113">This feature requires ASP.NET Core 2.x targeting .NET Core 2.x.</span></span>

<span data-ttu-id="3b756-114">Il [metapacchetto](https://www.nuget.org/packages/Microsoft.AspNetCore.App) [Microsoft.AspNetCore.App](/dotnet/core/packages#metapackages) per ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="3b756-114">The [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App) [metapackage](/dotnet/core/packages#metapackages) for ASP.NET Core:</span></span>

* <span data-ttu-id="3b756-115">Non include le dipendenze di terze parti, ad eccezione di [Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/), [Remotion.Linq](https://www.nuget.org/packages/Remotion.Linq/) e [IX-Async](https://www.nuget.org/packages/System.Interactive.Async/).</span><span class="sxs-lookup"><span data-stu-id="3b756-115">Does not include third-party dependencies except for [Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/), [Remotion.Linq](https://www.nuget.org/packages/Remotion.Linq/), and [IX-Async](https://www.nuget.org/packages/System.Interactive.Async/).</span></span> <span data-ttu-id="3b756-116">Queste dipendenze di terze parti sono necessarie per garantire il funzionamento delle principali funzionalità dei framework.</span><span class="sxs-lookup"><span data-stu-id="3b756-116">These 3rd-party dependencies are deemed necessary to ensure the major frameworks features function.</span></span>
* <span data-ttu-id="3b756-117">Include tutti i pacchetti supportati dal team di ASP.NET Core ad eccezione di quelli che contengono dipendenze di terze parti (diverse da quelle indicate in precedenza).</span><span class="sxs-lookup"><span data-stu-id="3b756-117">Includes all supported packages by the ASP.NET Core team except those that contain third-party dependencies (other than those previously mentioned).</span></span>
* <span data-ttu-id="3b756-118">Include tutti i pacchetti supportati dal team di Entity Framework Core ad eccezione di quelli che contengono dipendenze di terze parti (diverse da quelle indicate in precedenza).</span><span class="sxs-lookup"><span data-stu-id="3b756-118">Includes all supported packages by the Entity Framework Core team except those that contain third-party dependencies (other than those previously mentioned).</span></span>

<span data-ttu-id="3b756-119">Tutte le funzionalità di ASP.NET Core 2.x e Entity Framework Core 2.x sono incluse nel pacchetto `Microsoft.AspNetCore.App`.</span><span class="sxs-lookup"><span data-stu-id="3b756-119">All the features of ASP.NET Core 2.x and Entity Framework Core 2.x are included in the `Microsoft.AspNetCore.App` package.</span></span> <span data-ttu-id="3b756-120">I modelli di progetto predefiniti destinati a ASP.NET Core 2. x usano questo pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3b756-120">The default project templates targeting ASP.NET Core 2.x use this package.</span></span> <span data-ttu-id="3b756-121">Per le applicazioni destinate a ASP.NET Core 2. x e Entity Framework Core 2. x, è consigliabile utilizzare il `Microsoft.AspNetCore.App` pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3b756-121">We recommend applications targeting ASP.NET Core 2.x and Entity Framework Core 2.x use the `Microsoft.AspNetCore.App` package.</span></span>

<span data-ttu-id="3b756-122">Il numero di versione del metapacchetto `Microsoft.AspNetCore.App` rappresenta le versioni minime di ASP.NET Core e di Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="3b756-122">The version number of the `Microsoft.AspNetCore.App` metapackage represents the minimum ASP.NET Core version and Entity Framework Core version.</span></span>

<span data-ttu-id="3b756-123">L'uso del metapacchetto `Microsoft.AspNetCore.App` implica restrizioni di versione che proteggono l'app:</span><span class="sxs-lookup"><span data-stu-id="3b756-123">Using the `Microsoft.AspNetCore.App` metapackage provides version restrictions that protect your app:</span></span>

* <span data-ttu-id="3b756-124">Se viene incluso un pacchetto che presenta una dipendenza transitiva (non diretta) in un pacchetto in `Microsoft.AspNetCore.App` e i numeri di versione sono diversi, NuGet genererà un errore.</span><span class="sxs-lookup"><span data-stu-id="3b756-124">If a package is included that has a transitive (not direct) dependency on a package in `Microsoft.AspNetCore.App`, and those version numbers differ, NuGet will generate an error.</span></span>
* <span data-ttu-id="3b756-125">Altri pacchetti aggiunti all'app non possono modificare la versione dei pacchetti inclusi in `Microsoft.AspNetCore.App`.</span><span class="sxs-lookup"><span data-stu-id="3b756-125">Other packages added to your app cannot change the version of packages included in `Microsoft.AspNetCore.App`.</span></span>
* <span data-ttu-id="3b756-126">La coerenza della versione garantisce un'esperienza affidabile.</span><span class="sxs-lookup"><span data-stu-id="3b756-126">Version consistency ensures a reliable experience.</span></span> <span data-ttu-id="3b756-127">`Microsoft.AspNetCore.App` è stato progettato per impedire le combinazioni di versioni non testate di bit correlati usati contemporaneamente nella stessa applicazione.</span><span class="sxs-lookup"><span data-stu-id="3b756-127">`Microsoft.AspNetCore.App` was designed to prevent untested version combinations of related bits being used together in the same app.</span></span>

<span data-ttu-id="3b756-128">Le applicazioni che usano il metapacchetto `Microsoft.AspNetCore.App` sfruttano automaticamente il framework condiviso di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="3b756-128">Applications that use the `Microsoft.AspNetCore.App` metapackage automatically take advantage of the ASP.NET Core shared framework.</span></span> <span data-ttu-id="3b756-129">Quando si usa il metapacchetto `Microsoft.AspNetCore.App`, con l'applicazione non&mdash; vengono distribuiti asset dai pacchetti NuGet di riferimento di ASP.NET Core. Questi asset sono infatti inclusi nel framework condiviso di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="3b756-129">When you use the `Microsoft.AspNetCore.App` metapackage, **no** assets from the referenced ASP.NET Core NuGet packages are deployed with the application&mdash;the ASP.NET Core shared framework contains these assets.</span></span> <span data-ttu-id="3b756-130">Gli asset contenuti nel framework condiviso sono precompilati per migliorare i tempi di avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="3b756-130">The assets in the shared framework are precompiled to improve application startup time.</span></span> <span data-ttu-id="3b756-131">Per altre informazioni, vedere [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/) (Il framework condiviso).</span><span class="sxs-lookup"><span data-stu-id="3b756-131">For more information, see [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/).</span></span>

<span data-ttu-id="3b756-132">Il file di progetto seguente fa riferimento al `Microsoft.AspNetCore.App` metapacchetto per ASP.NET Core e rappresenta un modello tipico ASP.NET Core 2,2:</span><span class="sxs-lookup"><span data-stu-id="3b756-132">The following project file references the `Microsoft.AspNetCore.App` metapackage for ASP.NET Core and represents a typical ASP.NET Core 2.2 template:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.2</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

</Project>
```

<span data-ttu-id="3b756-133">Il markup precedente rappresenta un tipico modello di ASP.NET Core 2. x.</span><span class="sxs-lookup"><span data-stu-id="3b756-133">The preceding markup represents a typical ASP.NET Core 2.x template.</span></span> <span data-ttu-id="3b756-134">Non specifica un numero di versione per il pacchetto di riferimento `Microsoft.AspNetCore.App`.</span><span class="sxs-lookup"><span data-stu-id="3b756-134">It doesn't specify a version number for the `Microsoft.AspNetCore.App` package reference.</span></span> <span data-ttu-id="3b756-135">Quando la versione non è specificata, una versione [implicita](https://github.com/dotnet/core/blob/main/release-notes/1.0/sdk/1.0-rc3-implicit-package-refs.md) viene specificata dall'SDK, vale a dire `Microsoft.NET.Sdk.Web`.</span><span class="sxs-lookup"><span data-stu-id="3b756-135">When the version is not specified, an [implicit](https://github.com/dotnet/core/blob/main/release-notes/1.0/sdk/1.0-rc3-implicit-package-refs.md) version is specified by the SDK, that is, `Microsoft.NET.Sdk.Web`.</span></span> <span data-ttu-id="3b756-136">È consigliabile basarsi sulla versione implicita specificata dall'SDK e non impostando in modo esplicito il numero di versione sul riferimento al pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3b756-136">We recommend relying on the implicit version specified by the SDK and not explicitly setting the version number on the package reference.</span></span> <span data-ttu-id="3b756-137">In caso di domande su questo approccio, è possibile lasciare un commento GitHub nella pagina della [discussione per la versione implicita Microsoft.AspNetCore.App](https://github.com/dotnet/AspNetCore.Docs/issues/6430).</span><span class="sxs-lookup"><span data-stu-id="3b756-137">If you have questions on this approach, leave a GitHub comment at the [Discussion for the Microsoft.AspNetCore.App implicit version](https://github.com/dotnet/AspNetCore.Docs/issues/6430).</span></span>

<span data-ttu-id="3b756-138">La versione implicita è impostata su `major.minor.0` per le app portabili.</span><span class="sxs-lookup"><span data-stu-id="3b756-138">The implicit version is set to `major.minor.0` for portable apps.</span></span> <span data-ttu-id="3b756-139">Il meccanismo di roll forward del framework condiviso eseguirà l'app sulla versione compatibile più recente tra i framework condivisi installati.</span><span class="sxs-lookup"><span data-stu-id="3b756-139">The shared framework roll-forward mechanism will run the app on the latest compatible version among the installed shared frameworks.</span></span> <span data-ttu-id="3b756-140">Per garantire che venga usata la stessa versione in fase di sviluppo, test e produzione, verificare che in tutti gli ambienti sia installata la stessa versione del framework condiviso.</span><span class="sxs-lookup"><span data-stu-id="3b756-140">To guarantee the same version is used in development, test, and production, ensure the same version of the shared framework is installed in all environments.</span></span> <span data-ttu-id="3b756-141">Per le app autonome, il numero di versione implicita è impostato sul valore `major.minor.patch` del framework condiviso nel bundle nell'SDK installato.</span><span class="sxs-lookup"><span data-stu-id="3b756-141">For self contained apps, the implicit version number is set to the `major.minor.patch` of the shared framework bundled in the installed SDK.</span></span>

<span data-ttu-id="3b756-142">Specificando un numero di versione nel riferimento `Microsoft.AspNetCore.App`**non** si garantisce che verrà scelta la versione del framework condiviso.</span><span class="sxs-lookup"><span data-stu-id="3b756-142">Specifying a version number on the `Microsoft.AspNetCore.App` reference does **not** guarantee that version of the shared framework will be chosen.</span></span> <span data-ttu-id="3b756-143">Si supponga, ad esempio, che venga specificata la versione "2.2.1", ma che "2.2.3" sia installato.</span><span class="sxs-lookup"><span data-stu-id="3b756-143">For example, suppose version "2.2.1" is specified, but "2.2.3" is installed.</span></span> <span data-ttu-id="3b756-144">In tal caso, l'app userà "2.2.3".</span><span class="sxs-lookup"><span data-stu-id="3b756-144">In that case, the app will use "2.2.3".</span></span> <span data-ttu-id="3b756-145">Benché non sia consigliabile, è possibile disabilitare il roll forward (patch e/o versioni secondarie).</span><span class="sxs-lookup"><span data-stu-id="3b756-145">Although not recommended, you can disable roll forward (patch and/or minor).</span></span> <span data-ttu-id="3b756-146">Per altre informazioni su come eseguire il roll forward dell'host dotnet e configurarne il comportamento, vedere [dotnet host rollforward](https://github.com/dotnet/core-setup/blob/master/Documentation/design-docs/roll-forward-on-no-candidate-fx.md) (Roll forward dell'host dotnet).</span><span class="sxs-lookup"><span data-stu-id="3b756-146">For more information regarding dotnet host roll-forward and how to configure its behavior, see [dotnet host roll forward](https://github.com/dotnet/core-setup/blob/master/Documentation/design-docs/roll-forward-on-no-candidate-fx.md).</span></span>

::: moniker-end

::: moniker range="= aspnetcore-2.1"

<span data-ttu-id="3b756-147">`<Project Sdk` deve essere impostato su `Microsoft.NET.Sdk.Web` per usare la versione implicita `Microsoft.AspNetCore.App`.</span><span class="sxs-lookup"><span data-stu-id="3b756-147">`<Project Sdk` must be set to `Microsoft.NET.Sdk.Web` to use the implicit version `Microsoft.AspNetCore.App`.</span></span> <span data-ttu-id="3b756-148">Quando si usa `<Project Sdk="Microsoft.NET.Sdk">` (senza il carattere finale `.Web`):</span><span class="sxs-lookup"><span data-stu-id="3b756-148">When `<Project Sdk="Microsoft.NET.Sdk">` (without the trailing `.Web`) is used:</span></span>

* <span data-ttu-id="3b756-149">Viene generato l'avviso seguente:</span><span class="sxs-lookup"><span data-stu-id="3b756-149">The following warning is generated:</span></span>

  <span data-ttu-id="3b756-150">*Avviso NU1604: la dipendenza del progetto Microsoft. AspNetCore. app non contiene un limite inferiore inclusivo. Includere un limite inferiore nella versione della dipendenza per garantire risultati di ripristino coerenti.*</span><span class="sxs-lookup"><span data-stu-id="3b756-150">*Warning NU1604: Project dependency Microsoft.AspNetCore.App does not contain an inclusive lower bound. Include a lower bound in the dependency version to ensure consistent restore results.*</span></span>

* <span data-ttu-id="3b756-151">Si tratta di un problema noto in .NET Core 2.1 SDK.</span><span class="sxs-lookup"><span data-stu-id="3b756-151">This is a known issue with the .NET Core 2.1 SDK.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<a name="update"></a>

## <a name="update-aspnet-core"></a><span data-ttu-id="3b756-152">Aggiornare ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="3b756-152">Update ASP.NET Core</span></span>

<span data-ttu-id="3b756-153">Il  [metapacchetto](/dotnet/core/packages#metapackages)`Microsoft.AspNetCore.App` non è un pacchetto tradizionale aggiornato da NuGet.</span><span class="sxs-lookup"><span data-stu-id="3b756-153">The `Microsoft.AspNetCore.App` [metapackage](/dotnet/core/packages#metapackages) isn't a traditional package that's updated from NuGet.</span></span> <span data-ttu-id="3b756-154">Analogamente a `Microsoft.NETCore.App`, `Microsoft.AspNetCore.App` rappresenta un runtime condiviso, con una semantica di controllo delle versioni speciale gestita all'esterno di NuGet.</span><span class="sxs-lookup"><span data-stu-id="3b756-154">Similar to `Microsoft.NETCore.App`, `Microsoft.AspNetCore.App` represents a shared runtime, which has special versioning semantics handled outside of NuGet.</span></span> <span data-ttu-id="3b756-155">Per altre informazioni, vedere [Pacchetti, metapacchetti e framework](/dotnet/core/packages).</span><span class="sxs-lookup"><span data-stu-id="3b756-155">For more information, see [Packages, metapackages and frameworks](/dotnet/core/packages).</span></span>

<span data-ttu-id="3b756-156">Per aggiornare ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="3b756-156">To update ASP.NET Core:</span></span>

* <span data-ttu-id="3b756-157">Nei computer di sviluppo e nei server di compilazione: scaricare e installare [.NET Core SDK](https://dotnet.microsoft.com/download).</span><span class="sxs-lookup"><span data-stu-id="3b756-157">On development machines and build servers: Download and install the [.NET Core SDK](https://dotnet.microsoft.com/download).</span></span>
* <span data-ttu-id="3b756-158">Nei server di distribuzione: scaricare e installare il [runtime di .NET Core](https://dotnet.microsoft.com/download).</span><span class="sxs-lookup"><span data-stu-id="3b756-158">On deployment servers: Download and install the [.NET Core runtime](https://dotnet.microsoft.com/download).</span></span>

 <span data-ttu-id="3b756-159">Le applicazioni eseguiranno il roll forward all'ultima versione installata al riavvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="3b756-159">Applications will roll forward to the latest installed version on application restart.</span></span> <span data-ttu-id="3b756-160">Non è necessario aggiornare il numero di versione di `Microsoft.AspNetCore.App` nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="3b756-160">It's not necessary to update the `Microsoft.AspNetCore.App` version number in the project file.</span></span> <span data-ttu-id="3b756-161">Per altre informazioni, vedere [Roll forward delle app dipendenti dal framework](/dotnet/core/versions/selection#framework-dependent-apps-roll-forward).</span><span class="sxs-lookup"><span data-stu-id="3b756-161">For more information, see [Framework-dependent apps roll forward](/dotnet/core/versions/selection#framework-dependent-apps-roll-forward).</span></span>

<span data-ttu-id="3b756-162">Se l'applicazione usava in precedenza `Microsoft.AspNetCore.All`, vedere [Migrazione da Microsoft.AspNetCore.All a Microsoft.AspNetCore.App](xref:fundamentals/metapackage#migrate).</span><span class="sxs-lookup"><span data-stu-id="3b756-162">If your application previously used `Microsoft.AspNetCore.All`, see [Migrating from Microsoft.AspNetCore.All to Microsoft.AspNetCore.App](xref:fundamentals/metapackage#migrate).</span></span>

::: moniker-end
