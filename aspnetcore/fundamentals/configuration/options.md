---
title: Modello di opzioni in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare il modello di opzioni per rappresentare gruppi di opzioni correlate nelle app ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/20/2020
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
uid: fundamentals/configuration/options
ms.openlocfilehash: cb78147050ebdafc7de4ad150ae2644689d6efbc
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586215"
---
# <a name="options-pattern-in-aspnet-core"></a><span data-ttu-id="2f06f-103">Modello di opzioni in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2f06f-103">Options pattern in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="2f06f-104">Di [Kirk Larkin](https://twitter.com/serpent5) e [Rick Anderson](https://twitter.com/RickAndMSFT).</span><span class="sxs-lookup"><span data-stu-id="2f06f-104">By [Kirk Larkin](https://twitter.com/serpent5) and [Rick Anderson](https://twitter.com/RickAndMSFT).</span></span>

<span data-ttu-id="2f06f-105">Il modello options usa le classi per fornire accesso fortemente tipizzato ai gruppi di impostazioni correlate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-105">The options pattern uses classes to provide strongly typed access to groups of related settings.</span></span> <span data-ttu-id="2f06f-106">Quando le [impostazioni di configurazione](xref:fundamentals/configuration/index) vengono isolate in base allo scenario in classi separate, l'app aderisce a due importanti principi di progettazione del software:</span><span class="sxs-lookup"><span data-stu-id="2f06f-106">When [configuration settings](xref:fundamentals/configuration/index) are isolated by scenario into separate classes, the app adheres to two important software engineering principles:</span></span>

* <span data-ttu-id="2f06f-107">Il [principio di separazione dell'interfaccia (ISP) o l'incapsulamento](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): scenari (classi) che dipendono dalle impostazioni di configurazione dipendono solo dalle impostazioni di configurazione che usano.</span><span class="sxs-lookup"><span data-stu-id="2f06f-107">The [Interface Segregation Principle (ISP) or Encapsulation](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): Scenarios (classes) that depend on configuration settings depend only on the configuration settings that they use.</span></span>
* <span data-ttu-id="2f06f-108">[Separazione delle competenze (Separation of Concerns, SoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): le impostazioni di parti diverse dell'app non sono dipendenti o accoppiate l'una con l'altra.</span><span class="sxs-lookup"><span data-stu-id="2f06f-108">[Separation of Concerns](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): Settings for different parts of the app aren't dependent or coupled to one another.</span></span>

<span data-ttu-id="2f06f-109">Le opzioni offrono anche un meccanismo per convalidare i dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-109">Options also provide a mechanism to validate configuration data.</span></span> <span data-ttu-id="2f06f-110">Per altre informazioni, vedere la sezione [Opzioni di convalida](#options-validation).</span><span class="sxs-lookup"><span data-stu-id="2f06f-110">For more information, see the [Options validation](#options-validation) section.</span></span>

<span data-ttu-id="2f06f-111">In questo argomento vengono fornite informazioni sul modello di opzioni in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="2f06f-111">This topic provides information on the options pattern in ASP.NET Core.</span></span> <span data-ttu-id="2f06f-112">Per informazioni sull'uso del modello di opzioni nelle app console, vedere [modello di opzioni in .NET](/dotnet/core/extensions/options).</span><span class="sxs-lookup"><span data-stu-id="2f06f-112">For information on using the options pattern in console apps, see [Options pattern in .NET](/dotnet/core/extensions/options).</span></span>

<span data-ttu-id="2f06f-113">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="2f06f-113">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<a name="optpat"></a>

## <a name="bind-hierarchical-configuration"></a><span data-ttu-id="2f06f-114">Associa configurazione gerarchica</span><span class="sxs-lookup"><span data-stu-id="2f06f-114">Bind hierarchical configuration</span></span>

[!INCLUDE[](~/includes/bind.md)]

<a name="oi"></a>

## <a name="options-interfaces"></a><span data-ttu-id="2f06f-115">Interfacce per le opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-115">Options interfaces</span></span>

<span data-ttu-id="2f06f-116"><xref:Microsoft.Extensions.Options.IOptions%601>:</span><span class="sxs-lookup"><span data-stu-id="2f06f-116"><xref:Microsoft.Extensions.Options.IOptions%601>:</span></span>

* <span data-ttu-id="2f06f-117">Non ***supporta:***</span><span class="sxs-lookup"><span data-stu-id="2f06f-117">Does ***not*** support:</span></span>
  * <span data-ttu-id="2f06f-118">Lettura dei dati di configurazione dopo l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="2f06f-118">Reading of configuration data after the app has started.</span></span>
  * [<span data-ttu-id="2f06f-119">Opzioni denominate</span><span class="sxs-lookup"><span data-stu-id="2f06f-119">Named options</span></span>](#named)
* <span data-ttu-id="2f06f-120">Viene registrato come [singleton](xref:fundamentals/dependency-injection#singleton) e può essere inserito in qualsiasi durata del [servizio](xref:fundamentals/dependency-injection#service-lifetimes).</span><span class="sxs-lookup"><span data-stu-id="2f06f-120">Is registered as a [Singleton](xref:fundamentals/dependency-injection#singleton) and can be injected into any [service lifetime](xref:fundamentals/dependency-injection#service-lifetimes).</span></span>

<span data-ttu-id="2f06f-121"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601>:</span><span class="sxs-lookup"><span data-stu-id="2f06f-121"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601>:</span></span>

* <span data-ttu-id="2f06f-122">È utile negli scenari in cui le opzioni devono essere ricalcolate a ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-122">Is useful in scenarios where options should be recomputed on every request.</span></span> <span data-ttu-id="2f06f-123">Per altre informazioni, vedere [usare IOptionsSnapshot per leggere i dati aggiornati](#ios).</span><span class="sxs-lookup"><span data-stu-id="2f06f-123">For more information, see [Use IOptionsSnapshot to read updated data](#ios).</span></span>
* <span data-ttu-id="2f06f-124">Viene registrato come [ambito](xref:fundamentals/dependency-injection#scoped) e pertanto non può essere inserito in un servizio singleton.</span><span class="sxs-lookup"><span data-stu-id="2f06f-124">Is registered as [Scoped](xref:fundamentals/dependency-injection#scoped) and therefore cannot be injected into a Singleton service.</span></span>
* <span data-ttu-id="2f06f-125">Supporta le [opzioni denominate](#named)</span><span class="sxs-lookup"><span data-stu-id="2f06f-125">Supports [named options](#named)</span></span>

<span data-ttu-id="2f06f-126"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601>:</span><span class="sxs-lookup"><span data-stu-id="2f06f-126"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601>:</span></span>

* <span data-ttu-id="2f06f-127">Viene usato per recuperare le opzioni e gestire le notifiche delle opzioni per le `TOptions` istanze.</span><span class="sxs-lookup"><span data-stu-id="2f06f-127">Is used to retrieve options and manage options notifications for `TOptions` instances.</span></span>
* <span data-ttu-id="2f06f-128">Viene registrato come [singleton](xref:fundamentals/dependency-injection#singleton) e può essere inserito in qualsiasi durata del [servizio](xref:fundamentals/dependency-injection#service-lifetimes).</span><span class="sxs-lookup"><span data-stu-id="2f06f-128">Is registered as a [Singleton](xref:fundamentals/dependency-injection#singleton) and can be injected into any [service lifetime](xref:fundamentals/dependency-injection#service-lifetimes).</span></span>
* <span data-ttu-id="2f06f-129">Supporto:</span><span class="sxs-lookup"><span data-stu-id="2f06f-129">Supports:</span></span>
  * <span data-ttu-id="2f06f-130">Notifiche di modifica</span><span class="sxs-lookup"><span data-stu-id="2f06f-130">Change notifications</span></span>
  * [<span data-ttu-id="2f06f-131">Opzioni denominate</span><span class="sxs-lookup"><span data-stu-id="2f06f-131">Named options</span></span>](#named-options-support-with-iconfigurenamedoptions)
  * [<span data-ttu-id="2f06f-132">Configurazione ricaricabile</span><span class="sxs-lookup"><span data-stu-id="2f06f-132">Reloadable configuration</span></span>](#ios)
  * <span data-ttu-id="2f06f-133">Invalidamento selettivo di opzioni (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)</span><span class="sxs-lookup"><span data-stu-id="2f06f-133">Selective options invalidation (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)</span></span>
  
<span data-ttu-id="2f06f-134">Gli scenari [di post-configurazione](#options-post-configuration) consentono di impostare o modificare le opzioni dopo la <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-134">[Post-configuration](#options-post-configuration) scenarios enable setting or changing options after all <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configuration occurs.</span></span>

<span data-ttu-id="2f06f-135"><xref:Microsoft.Extensions.Options.IOptionsFactory%601> è responsabile della creazione di nuove istanze di opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-135"><xref:Microsoft.Extensions.Options.IOptionsFactory%601> is responsible for creating new options instances.</span></span> <span data-ttu-id="2f06f-136">Include un singolo metodo <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-136">It has a single <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*> method.</span></span> <span data-ttu-id="2f06f-137">L'implementazione predefinita accetta tutte le interfacce <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> registrate ed esegue tutte le configurazioni, seguite dalla post-configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-137">The default implementation takes all registered <xref:Microsoft.Extensions.Options.IConfigureOptions%601> and <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> and runs all the configurations first, followed by the post-configuration.</span></span> <span data-ttu-id="2f06f-138">Fa distinzione tra <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e chiama solo l'interfaccia appropriata.</span><span class="sxs-lookup"><span data-stu-id="2f06f-138">It distinguishes between <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> and <xref:Microsoft.Extensions.Options.IConfigureOptions%601> and only calls the appropriate interface.</span></span>

<span data-ttu-id="2f06f-139"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> viene usata da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per memorizzare nella cache le istanze di `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-139"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> is used by <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> to cache `TOptions` instances.</span></span> <span data-ttu-id="2f06f-140"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalida le istanze delle opzioni nel monitoraggio in modo che il valore venga ricalcolato (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>).</span><span class="sxs-lookup"><span data-stu-id="2f06f-140">The <xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalidates options instances in the monitor so that the value is recomputed (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>).</span></span> <span data-ttu-id="2f06f-141">I valori possono essere introdotti manualmente con <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-141">Values can be manually introduced with <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>.</span></span> <span data-ttu-id="2f06f-142">Il metodo <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> viene usato quando tutte le istanze denominate devono essere ricreate su richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-142">The <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> method is used when all named instances should be recreated on demand.</span></span>

<a name="ios"></a>

## <a name="use-ioptionssnapshot-to-read-updated-data"></a><span data-ttu-id="2f06f-143">Usare IOptionsSnapshot per leggere i dati aggiornati</span><span class="sxs-lookup"><span data-stu-id="2f06f-143">Use IOptionsSnapshot to read updated data</span></span>

<span data-ttu-id="2f06f-144">Usando <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> , le opzioni vengono calcolate una volta per ogni richiesta in caso di accesso e memorizzati nella cache per la durata della richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-144">Using <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601>, options are computed once per request when accessed and cached for the lifetime of the request.</span></span> <span data-ttu-id="2f06f-145">Le modifiche alla configurazione vengono lette dopo l'avvio dell'app quando si usano i provider di configurazione che supportano la lettura dei valori di configurazione aggiornati.</span><span class="sxs-lookup"><span data-stu-id="2f06f-145">Changes to the configuration are read after the app starts when using configuration providers that support reading updated configuration values.</span></span>

<span data-ttu-id="2f06f-146">La differenza tra `IOptionsMonitor` e `IOptionsSnapshot` è che:</span><span class="sxs-lookup"><span data-stu-id="2f06f-146">The difference between `IOptionsMonitor` and `IOptionsSnapshot` is that:</span></span>

* <span data-ttu-id="2f06f-147">`IOptionsMonitor` è un [servizio singleton](xref:fundamentals/dependency-injection#singleton) che recupera i valori correnti delle opzioni in qualsiasi momento, operazione particolarmente utile nelle dipendenze singleton.</span><span class="sxs-lookup"><span data-stu-id="2f06f-147">`IOptionsMonitor` is a [singleton service](xref:fundamentals/dependency-injection#singleton) that retrieves current option values at any time, which is especially useful in singleton dependencies.</span></span>
* <span data-ttu-id="2f06f-148">`IOptionsSnapshot` è un [servizio con ambito](xref:fundamentals/dependency-injection#scoped) e fornisce uno snapshot delle opzioni nel momento in cui `IOptionsSnapshot<T>` viene costruito l'oggetto.</span><span class="sxs-lookup"><span data-stu-id="2f06f-148">`IOptionsSnapshot` is a [scoped service](xref:fundamentals/dependency-injection#scoped) and provides a snapshot of the options at the time the `IOptionsSnapshot<T>` object is constructed.</span></span> <span data-ttu-id="2f06f-149">Gli snapshot delle opzioni sono progettati per l'uso con dipendenze temporanee e con ambito.</span><span class="sxs-lookup"><span data-stu-id="2f06f-149">Options snapshots are designed for use with transient and scoped dependencies.</span></span>

<span data-ttu-id="2f06f-150">Il codice seguente usa <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> .</span><span class="sxs-lookup"><span data-stu-id="2f06f-150">The following code uses <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601>.</span></span>

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/TestSnap.cshtml.cs?name=snippet)]

<span data-ttu-id="2f06f-151">Il codice seguente registra un'istanza di configurazione `MyOptions` con cui viene eseguita l'associazione:</span><span class="sxs-lookup"><span data-stu-id="2f06f-151">The following code registers a configuration instance which `MyOptions` binds against:</span></span>

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup3.cs?name=snippet_Example2)]

<span data-ttu-id="2f06f-152">Nel codice precedente, le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app vengono lette.</span><span class="sxs-lookup"><span data-stu-id="2f06f-152">In the preceding code, changes to the JSON configuration file after the app has started are read.</span></span>

## <a name="ioptionsmonitor"></a><span data-ttu-id="2f06f-153">IOptionsMonitor</span><span class="sxs-lookup"><span data-stu-id="2f06f-153">IOptionsMonitor</span></span>

<span data-ttu-id="2f06f-154">Il codice seguente registra un'istanza di configurazione `MyOptions` con cui viene eseguita l'associazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-154">The following code registers a configuration instance which `MyOptions` binds against.</span></span>

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup3.cs?name=snippet_Example2)]

<span data-ttu-id="2f06f-155">Nell'esempio seguente viene usato <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>:</span><span class="sxs-lookup"><span data-stu-id="2f06f-155">The following example uses <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>:</span></span>

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/TestMonitor.cshtml.cs?name=snippet)]

<span data-ttu-id="2f06f-156">Nel codice precedente, per impostazione predefinita, vengono lette le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="2f06f-156">In the preceding code, by default, changes to the JSON configuration file after the app has started are read.</span></span>

<a name="named"></a>

## <a name="named-options-support-using-iconfigurenamedoptions"></a><span data-ttu-id="2f06f-157">Supporto delle opzioni denominate con IConfigureNamedOptions</span><span class="sxs-lookup"><span data-stu-id="2f06f-157">Named options support using IConfigureNamedOptions</span></span>

<span data-ttu-id="2f06f-158">Opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-158">Named options:</span></span>

* <span data-ttu-id="2f06f-159">Sono utili quando più sezioni di configurazione sono associate alle stesse proprietà.</span><span class="sxs-lookup"><span data-stu-id="2f06f-159">Are useful when multiple configuration sections bind to the same properties.</span></span>
* <span data-ttu-id="2f06f-160">Distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="2f06f-160">Are case sensitive.</span></span>

<span data-ttu-id="2f06f-161">Si consideri il seguente *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="2f06f-161">Consider the following *appsettings.json* file:</span></span>

[!code-json[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/appsettings.NO.json)]

<span data-ttu-id="2f06f-162">Anziché creare due classi da associare `TopItem:Month` e `TopItem:Year` , per ogni sezione viene usata la classe seguente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-162">Rather than creating two classes to bind `TopItem:Month` and `TopItem:Year`, the following class is used for each section:</span></span>

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Models/TopItemSettings.cs)]

<span data-ttu-id="2f06f-163">Il codice seguente configura le opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-163">The following code configures the named options:</span></span>

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/StartupNO.cs?name=snippet_Example2)]

<span data-ttu-id="2f06f-164">Il codice seguente consente di visualizzare le opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-164">The following code displays the named options:</span></span>

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/TestNO.cshtml.cs?name=snippet)]

<span data-ttu-id="2f06f-165">Tutte le opzioni sono istanze denominate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-165">All options are named instances.</span></span> <span data-ttu-id="2f06f-166"><xref:Microsoft.Extensions.Options.IConfigureOptions%601> le istanze vengono considerate come destinazione dell' `Options.DefaultName` istanza, ovvero `string.Empty` .</span><span class="sxs-lookup"><span data-stu-id="2f06f-166"><xref:Microsoft.Extensions.Options.IConfigureOptions%601> instances are treated as targeting the `Options.DefaultName` instance, which is `string.Empty`.</span></span> <span data-ttu-id="2f06f-167"><xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> implementa anche <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-167"><xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> also implements <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span></span> <span data-ttu-id="2f06f-168">L'implementazione predefinita di <xref:Microsoft.Extensions.Options.IOptionsFactory%601> include la logica per usarle in modo appropriato.</span><span class="sxs-lookup"><span data-stu-id="2f06f-168">The default implementation of the <xref:Microsoft.Extensions.Options.IOptionsFactory%601> has logic to use each appropriately.</span></span> <span data-ttu-id="2f06f-169">L' `null` opzione denominata viene utilizzata per fare riferimento a tutte le istanze denominate anziché a un'istanza denominata specifica.</span><span class="sxs-lookup"><span data-stu-id="2f06f-169">The `null` named option is used to target all of the named instances instead of a specific named instance.</span></span> <span data-ttu-id="2f06f-170"><xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> e <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> usano questa convenzione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-170"><xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> and <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> use this convention.</span></span>

## <a name="optionsbuilder-api"></a><span data-ttu-id="2f06f-171">API OptionsBuilder</span><span class="sxs-lookup"><span data-stu-id="2f06f-171">OptionsBuilder API</span></span>

<span data-ttu-id="2f06f-172"><xref:Microsoft.Extensions.Options.OptionsBuilder%601> viene usata per configurare le istanze di `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-172"><xref:Microsoft.Extensions.Options.OptionsBuilder%601> is used to configure `TOptions` instances.</span></span> <span data-ttu-id="2f06f-173">`OptionsBuilder` semplifica la creazione di opzioni denominate perché è costituita da un singolo parametro nella chiamata iniziale ad `AddOptions<TOptions>(string optionsName)` invece di essere visualizzata in tutte le chiamate successive.</span><span class="sxs-lookup"><span data-stu-id="2f06f-173">`OptionsBuilder` streamlines creating named options as it's only a single parameter to the initial `AddOptions<TOptions>(string optionsName)` call instead of appearing in all of the subsequent calls.</span></span> <span data-ttu-id="2f06f-174">La convalida delle opzioni e gli overload `ConfigureOptions` che accettano le dipendenze dei servizi sono disponibili solo tramite `OptionsBuilder`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-174">Options validation and the `ConfigureOptions` overloads that accept service dependencies are only available via `OptionsBuilder`.</span></span>

<span data-ttu-id="2f06f-175">`OptionsBuilder` viene usato nella sezione [convalida opzioni](#val) .</span><span class="sxs-lookup"><span data-stu-id="2f06f-175">`OptionsBuilder` is used in the [Options validation](#val) section.</span></span>

## <a name="use-di-services-to-configure-options"></a><span data-ttu-id="2f06f-176">Usare i servizi di inserimento delle dipendenze per configurare le opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-176">Use DI services to configure options</span></span>

<span data-ttu-id="2f06f-177">È possibile accedere ai servizi dall'inserimento delle dipendenze durante la configurazione delle opzioni in due modi:</span><span class="sxs-lookup"><span data-stu-id="2f06f-177">Services can be accessed from dependency injection while configuring options in two ways:</span></span>

* <span data-ttu-id="2f06f-178">Passare un delegato di configurazione da [configurare](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) in [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1).</span><span class="sxs-lookup"><span data-stu-id="2f06f-178">Pass a configuration delegate to [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) on [OptionsBuilder\<TOptions>](xref:Microsoft.Extensions.Options.OptionsBuilder`1).</span></span> <span data-ttu-id="2f06f-179">`OptionsBuilder<TOptions>` fornisce overload di [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) che consentono l'uso di un massimo di cinque servizi per configurare le opzioni:</span><span class="sxs-lookup"><span data-stu-id="2f06f-179">`OptionsBuilder<TOptions>` provides overloads of [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) that allow use of up to five services to configure options:</span></span>

  ```csharp
  services.AddOptions<MyOptions>("optionalName")
      .Configure<Service1, Service2, Service3, Service4, Service5>(
          (o, s, s2, s3, s4, s5) => 
              o.Property = DoSomethingWith(s, s2, s3, s4, s5));
  ```

* <span data-ttu-id="2f06f-180">Creare un tipo che implementi <xref:Microsoft.Extensions.Options.IConfigureOptions%601> o <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e registri il tipo come servizio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-180">Create a type that implements <xref:Microsoft.Extensions.Options.IConfigureOptions%601> or <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> and register the type as a service.</span></span>

<span data-ttu-id="2f06f-181">È consigliabile passare un delegato di configurazione a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) perché la creazione di un servizio è più complessa.</span><span class="sxs-lookup"><span data-stu-id="2f06f-181">We recommend passing a configuration delegate to [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*), since creating a service is more complex.</span></span> <span data-ttu-id="2f06f-182">La creazione di un tipo è equivalente a quella eseguita dal framework quando si chiama [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*).</span><span class="sxs-lookup"><span data-stu-id="2f06f-182">Creating a type is equivalent to what the framework does when calling [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*).</span></span> <span data-ttu-id="2f06f-183">La chiamata a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registra un'interfaccia <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> generica temporanea, che ha un costruttore che accetta i tipi di servizi generici specificati.</span><span class="sxs-lookup"><span data-stu-id="2f06f-183">Calling [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registers a transient generic <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601>, which has a constructor that accepts the generic service types specified.</span></span> 

<a name="val"></a>

## <a name="options-validation"></a><span data-ttu-id="2f06f-184">Convalida delle opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-184">Options validation</span></span>

<span data-ttu-id="2f06f-185">La convalida delle opzioni consente di convalidare i valori dell'opzione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-185">Options validation enables option values to be validated.</span></span>

<span data-ttu-id="2f06f-186">Si consideri il seguente *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="2f06f-186">Consider the following *appsettings.json* file:</span></span>

[!code-json[](~/fundamentals/configuration/options/samples/3.x/OptionsValidationSample/appsettings.Dev2.json)]

<span data-ttu-id="2f06f-187">La classe seguente viene associata alla `"MyConfig"` sezione di configurazione e applica due `DataAnnotations` regole:</span><span class="sxs-lookup"><span data-stu-id="2f06f-187">The following class binds to the `"MyConfig"` configuration section and applies a couple of `DataAnnotations` rules:</span></span>

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Configuration/MyConfigOptions.cs?name=snippet)]

<span data-ttu-id="2f06f-188">Il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-188">The following code:</span></span>

* <span data-ttu-id="2f06f-189">Chiama <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions%2A> per ottenere un [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1) associato alla `MyConfigOptions` classe.</span><span class="sxs-lookup"><span data-stu-id="2f06f-189">Calls <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions%2A> to get an [OptionsBuilder\<TOptions>](xref:Microsoft.Extensions.Options.OptionsBuilder`1) that binds to the `MyConfigOptions` class.</span></span>
* <span data-ttu-id="2f06f-190">Chiama <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations%2A> per abilitare la convalida usando `DataAnnotations` .</span><span class="sxs-lookup"><span data-stu-id="2f06f-190">Calls <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations%2A> to enable validation using `DataAnnotations`.</span></span>

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Startup.cs?name=snippet)]

<span data-ttu-id="2f06f-191">Il `ValidateDataAnnotations` metodo di estensione è definito nel pacchetto NuGet [Microsoft. Extensions. Options. DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) .</span><span class="sxs-lookup"><span data-stu-id="2f06f-191">The `ValidateDataAnnotations` extension method is defined in the [Microsoft.Extensions.Options.DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) NuGet package.</span></span> <span data-ttu-id="2f06f-192">Per le app Web che usano l' `Microsoft.NET.Sdk.Web` SDK, a questo pacchetto viene fatto riferimento in modo implicito dal Framework condiviso.</span><span class="sxs-lookup"><span data-stu-id="2f06f-192">For web apps that use the `Microsoft.NET.Sdk.Web` SDK, this package is referenced implicitly from the shared framework.</span></span>

<span data-ttu-id="2f06f-193">Nel codice seguente vengono visualizzati i valori di configurazione o gli errori di convalida:</span><span class="sxs-lookup"><span data-stu-id="2f06f-193">The following code displays the configuration values or the validation errors:</span></span>

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Controllers/HomeController.cs?name=snippet)]

<span data-ttu-id="2f06f-194">Il codice seguente applica una regola di convalida più complessa usando un delegato:</span><span class="sxs-lookup"><span data-stu-id="2f06f-194">The following code applies a more complex validation rule using a delegate:</span></span>

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Startup2.cs?name=snippet)]

### <a name="ivalidateoptions-for-complex-validation"></a><span data-ttu-id="2f06f-195">IValidateOptions per la convalida complessa</span><span class="sxs-lookup"><span data-stu-id="2f06f-195">IValidateOptions for complex validation</span></span>

<span data-ttu-id="2f06f-196">La classe seguente implementa <xref:Microsoft.Extensions.Options.IValidateOptions`1> :</span><span class="sxs-lookup"><span data-stu-id="2f06f-196">The following class implements <xref:Microsoft.Extensions.Options.IValidateOptions`1>:</span></span>

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Configuration/MyConfigValidation.cs?name=snippet)]

<span data-ttu-id="2f06f-197">`IValidateOptions` consente di trasferire il codice di convalida da `StartUp` e in una classe.</span><span class="sxs-lookup"><span data-stu-id="2f06f-197">`IValidateOptions` enables moving the validation code out of `StartUp` and into a class.</span></span>

<span data-ttu-id="2f06f-198">Utilizzando il codice precedente, la convalida è abilitata in `Startup.ConfigureServices` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-198">Using the preceding code, validation is enabled in `Startup.ConfigureServices` with the following code:</span></span>

[!code-csharp[](options/samples/3.x/OptionsValidationSample/StartupValidation.cs?name=snippet)]

<!-- The following comment doesn't seem that useful 
Options validation doesn't guard against options modifications after the options instance is created. For example, `IOptionsSnapshot` options are created and validated once per request when the options are first accessed. The `IOptionsSnapshot` options aren't validated again on subsequent access attempts *for the same request*.

The `Validate` method accepts a `Func<TOptions, bool>`. To fully customize validation, implement `IValidateOptions<TOptions>`, which allows:

* Validation of multiple options types: `class ValidateTwo : IValidateOptions<Option1>, IValidationOptions<Option2>`
* Validation that depends on another option type: `public DependsOnAnotherOptionValidator(IOptionsMonitor<AnotherOption> options)`

`IValidateOptions` validates:

* A specific named options instance.
* All options when `name` is `null`.

Return a `ValidateOptionsResult` from your implementation of the interface:

```csharp
public interface IValidateOptions<TOptions> where TOptions : class
{
    ValidateOptionsResult Validate(string name, TOptions options);
}
```

Data Annotation-based validation is available from the [Microsoft.Extensions.Options.DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) package by calling the <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations*> method on `OptionsBuilder<TOptions>`. `Microsoft.Extensions.Options.DataAnnotations` is implicitly referenced in ASP.NET Core apps.

-->

## <a name="options-post-configuration"></a><span data-ttu-id="2f06f-199">Post-configurazione delle opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-199">Options post-configuration</span></span>

<span data-ttu-id="2f06f-200">Impostare la post-configurazione con <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-200">Set post-configuration with <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>.</span></span> <span data-ttu-id="2f06f-201">La post-configurazione viene eseguita dopo il completamento della configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>:</span><span class="sxs-lookup"><span data-stu-id="2f06f-201">Post-configuration runs after all <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configuration occurs:</span></span>

```csharp
services.PostConfigure<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<span data-ttu-id="2f06f-202"><xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> è disponibile per la post-configurazione delle opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-202"><xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> is available to post-configure named options:</span></span>

```csharp
services.PostConfigure<MyOptions>("named_options_1", myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<span data-ttu-id="2f06f-203">Usare <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> per la post-configurazione di tutte le istanze di configurazione:</span><span class="sxs-lookup"><span data-stu-id="2f06f-203">Use <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> to post-configure all configuration instances:</span></span>

```csharp
services.PostConfigureAll<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

## <a name="accessing-options-during-startup"></a><span data-ttu-id="2f06f-204">Accesso alle opzioni durante l'avvio</span><span class="sxs-lookup"><span data-stu-id="2f06f-204">Accessing options during startup</span></span>

<span data-ttu-id="2f06f-205"><xref:Microsoft.Extensions.Options.IOptions%601> e <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> possono essere usate in `Startup.Configure`, perché i servizi vengono compilati prima dell'esecuzione del metodo `Configure`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-205"><xref:Microsoft.Extensions.Options.IOptions%601> and <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> can be used in `Startup.Configure`, since services are built before the `Configure` method executes.</span></span>

```csharp
public void Configure(IApplicationBuilder app, 
    IOptionsMonitor<MyOptions> optionsAccessor)
{
    var option1 = optionsAccessor.CurrentValue.Option1;
}
```

<span data-ttu-id="2f06f-206">Non usare <xref:Microsoft.Extensions.Options.IOptions%601> oppure <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-206">Don't use <xref:Microsoft.Extensions.Options.IOptions%601> or <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="2f06f-207">Le opzioni potrebbero avere uno stato incoerente a causa dell'ordinamento delle registrazioni dei servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-207">An inconsistent options state may exist due to the ordering of service registrations.</span></span>

## <a name="optionsconfigurationextensions-nuget-package"></a><span data-ttu-id="2f06f-208">Options.Configpacchetto NuGet urationExtensions</span><span class="sxs-lookup"><span data-stu-id="2f06f-208">Options.ConfigurationExtensions NuGet package</span></span>

<span data-ttu-id="2f06f-209">Il pacchetto [ urationExtensionsMicrosoft.Extensions.Options.Config](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/) viene fatto riferimento in modo implicito nelle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="2f06f-209">The [Microsoft.Extensions.Options.ConfigurationExtensions](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/) package is implicitly referenced in ASP.NET Core apps.</span></span>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="2f06f-210">Il modello di opzioni usa le classi per rappresentare i gruppi di impostazioni correlate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-210">The options pattern uses classes to represent groups of related settings.</span></span> <span data-ttu-id="2f06f-211">Quando le [impostazioni di configurazione](xref:fundamentals/configuration/index) vengono isolate in base allo scenario in classi separate, l'app aderisce a due importanti principi di progettazione del software:</span><span class="sxs-lookup"><span data-stu-id="2f06f-211">When [configuration settings](xref:fundamentals/configuration/index) are isolated by scenario into separate classes, the app adheres to two important software engineering principles:</span></span>

* <span data-ttu-id="2f06f-212">Il [principio di separazione dell'interfaccia (ISP) o l'incapsulamento](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): scenari (classi) che dipendono dalle impostazioni di configurazione dipendono solo dalle impostazioni di configurazione che usano.</span><span class="sxs-lookup"><span data-stu-id="2f06f-212">The [Interface Segregation Principle (ISP) or Encapsulation](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): Scenarios (classes) that depend on configuration settings depend only on the configuration settings that they use.</span></span>
* <span data-ttu-id="2f06f-213">[Separazione delle competenze (Separation of Concerns, SoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): le impostazioni di parti diverse dell'app non sono dipendenti o accoppiate l'una con l'altra.</span><span class="sxs-lookup"><span data-stu-id="2f06f-213">[Separation of Concerns](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): Settings for different parts of the app aren't dependent or coupled to one another.</span></span>

<span data-ttu-id="2f06f-214">Le opzioni offrono anche un meccanismo per convalidare i dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-214">Options also provide a mechanism to validate configuration data.</span></span> <span data-ttu-id="2f06f-215">Per altre informazioni, vedere la sezione [Opzioni di convalida](#options-validation).</span><span class="sxs-lookup"><span data-stu-id="2f06f-215">For more information, see the [Options validation](#options-validation) section.</span></span>

<span data-ttu-id="2f06f-216">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="2f06f-216">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2f06f-217">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2f06f-217">Prerequisites</span></span>

<span data-ttu-id="2f06f-218">Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) oppure aggiungere un riferimento al pacchetto [Microsoft.Extensions.Options.ConfigurationExtensions](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/).</span><span class="sxs-lookup"><span data-stu-id="2f06f-218">Reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) or add a package reference to the [Microsoft.Extensions.Options.ConfigurationExtensions](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/) package.</span></span>

## <a name="options-interfaces"></a><span data-ttu-id="2f06f-219">Interfacce per le opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-219">Options interfaces</span></span>

<span data-ttu-id="2f06f-220"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> consente di recuperare le opzioni e gestire le notifiche di opzioni per le istanze `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-220"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> is used to retrieve options and manage options notifications for `TOptions` instances.</span></span> <span data-ttu-id="2f06f-221"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> supporta gli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="2f06f-221"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> supports the following scenarios:</span></span>

* <span data-ttu-id="2f06f-222">Notifiche di modifica</span><span class="sxs-lookup"><span data-stu-id="2f06f-222">Change notifications</span></span>
* [<span data-ttu-id="2f06f-223">Opzioni denominate</span><span class="sxs-lookup"><span data-stu-id="2f06f-223">Named options</span></span>](#named-options-support-with-iconfigurenamedoptions)
* [<span data-ttu-id="2f06f-224">Configurazione ricaricabile</span><span class="sxs-lookup"><span data-stu-id="2f06f-224">Reloadable configuration</span></span>](#reload-configuration-data-with-ioptionssnapshot)
* <span data-ttu-id="2f06f-225">Invalidamento selettivo di opzioni (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)</span><span class="sxs-lookup"><span data-stu-id="2f06f-225">Selective options invalidation (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)</span></span>

<span data-ttu-id="2f06f-226">Gli scenari di [post-configurazione](#options-post-configuration) consentono di impostare o modificare le opzioni dopo la configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-226">[Post-configuration](#options-post-configuration) scenarios allow you to set or change options after all <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configuration occurs.</span></span>

<span data-ttu-id="2f06f-227"><xref:Microsoft.Extensions.Options.IOptionsFactory%601> è responsabile della creazione di nuove istanze di opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-227"><xref:Microsoft.Extensions.Options.IOptionsFactory%601> is responsible for creating new options instances.</span></span> <span data-ttu-id="2f06f-228">Include un singolo metodo <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-228">It has a single <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*> method.</span></span> <span data-ttu-id="2f06f-229">L'implementazione predefinita accetta tutte le interfacce <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> registrate ed esegue tutte le configurazioni, seguite dalla post-configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-229">The default implementation takes all registered <xref:Microsoft.Extensions.Options.IConfigureOptions%601> and <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> and runs all the configurations first, followed by the post-configuration.</span></span> <span data-ttu-id="2f06f-230">Fa distinzione tra <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e chiama solo l'interfaccia appropriata.</span><span class="sxs-lookup"><span data-stu-id="2f06f-230">It distinguishes between <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> and <xref:Microsoft.Extensions.Options.IConfigureOptions%601> and only calls the appropriate interface.</span></span>

<span data-ttu-id="2f06f-231"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> viene usata da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per memorizzare nella cache le istanze di `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-231"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> is used by <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> to cache `TOptions` instances.</span></span> <span data-ttu-id="2f06f-232"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalida le istanze delle opzioni nel monitoraggio in modo che il valore venga ricalcolato (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>).</span><span class="sxs-lookup"><span data-stu-id="2f06f-232">The <xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalidates options instances in the monitor so that the value is recomputed (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>).</span></span> <span data-ttu-id="2f06f-233">I valori possono essere introdotti manualmente con <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-233">Values can be manually introduced with <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>.</span></span> <span data-ttu-id="2f06f-234">Il metodo <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> viene usato quando tutte le istanze denominate devono essere ricreate su richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-234">The <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> method is used when all named instances should be recreated on demand.</span></span>

<span data-ttu-id="2f06f-235"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> è utile negli scenari in cui le opzioni devono essere ricalcolate a ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-235"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> is useful in scenarios where options should be recomputed on every request.</span></span> <span data-ttu-id="2f06f-236">Per altre informazioni, vedere la sezione [Ricaricare i dati di configurazione con IOptionsSnapshot](#reload-configuration-data-with-ioptionssnapshot).</span><span class="sxs-lookup"><span data-stu-id="2f06f-236">For more information, see the [Reload configuration data with IOptionsSnapshot](#reload-configuration-data-with-ioptionssnapshot) section.</span></span>

<span data-ttu-id="2f06f-237"><xref:Microsoft.Extensions.Options.IOptions%601> può essere usata a supporto delle opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-237"><xref:Microsoft.Extensions.Options.IOptions%601> can be used to support options.</span></span> <span data-ttu-id="2f06f-238">Tuttavia <xref:Microsoft.Extensions.Options.IOptions%601> non supporta gli scenari precedenti di <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-238">However, <xref:Microsoft.Extensions.Options.IOptions%601> doesn't support the preceding scenarios of <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span></span> <span data-ttu-id="2f06f-239">È possibile continuare a usare <xref:Microsoft.Extensions.Options.IOptions%601> in framework e librerie esistenti che già usano l'interfaccia <xref:Microsoft.Extensions.Options.IOptions%601> e non richiedono gli scenari forniti da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-239">You may continue to use <xref:Microsoft.Extensions.Options.IOptions%601> in existing frameworks and libraries that already use the <xref:Microsoft.Extensions.Options.IOptions%601> interface and don't require the scenarios provided by <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span></span>

## <a name="general-options-configuration"></a><span data-ttu-id="2f06f-240">Configurazione delle opzioni generali</span><span class="sxs-lookup"><span data-stu-id="2f06f-240">General options configuration</span></span>

<span data-ttu-id="2f06f-241">La configurazione delle opzioni generali è illustrata nell'Esempio 1 nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-241">General options configuration is demonstrated as Example 1 in the sample app.</span></span>

<span data-ttu-id="2f06f-242">Una classe di opzioni deve essere non astratta con un costruttore pubblico senza parametri.</span><span class="sxs-lookup"><span data-stu-id="2f06f-242">An options class must be non-abstract with a public parameterless constructor.</span></span> <span data-ttu-id="2f06f-243">La classe seguente, `MyOptions`, ha due proprietà, `Option1` e `Option2`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-243">The following class, `MyOptions`, has two properties, `Option1` and `Option2`.</span></span> <span data-ttu-id="2f06f-244">Sebbene l'impostazione dei valori predefiniti sia facoltativa, il costruttore della classe nell'esempio seguente imposta il valore predefinito di `Option1`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-244">Setting default values is optional, but the class constructor in the following example sets the default value of `Option1`.</span></span> <span data-ttu-id="2f06f-245">`Option2` ha un valore impostato tramite l'inizializzazione diretta della proprietà (*Models/MyOptions.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-245">`Option2` has a default value set by initializing the property directly (*Models/MyOptions.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptions.cs?name=snippet1)]

<span data-ttu-id="2f06f-246">La classe `MyOptions` viene aggiunta al contenitore di servizi con <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> e associata alla configurazione:</span><span class="sxs-lookup"><span data-stu-id="2f06f-246">The `MyOptions` class is added to the service container with <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> and bound to configuration:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example1)]

<span data-ttu-id="2f06f-247">Il modello di pagina seguente usa l'[inserimento delle dipendenze del costruttore](xref:mvc/controllers/dependency-injection) con <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per accedere alle impostazioni (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-247">The following page model uses [constructor dependency injection](xref:mvc/controllers/dependency-injection) with <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> to access the settings (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example1)]

<span data-ttu-id="2f06f-248">Il file dell'esempio *appsettings.json* specifica i valori per `option1` e `option2` :</span><span class="sxs-lookup"><span data-stu-id="2f06f-248">The sample's *appsettings.json* file specifies values for `option1` and `option2`:</span></span>

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=2-3)]

<span data-ttu-id="2f06f-249">Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:</span><span class="sxs-lookup"><span data-stu-id="2f06f-249">When the app is run, the page model's `OnGet` method returns a string showing the option class values:</span></span>

```html
option1 = value1_from_json, option2 = -1
```

> [!NOTE]
> <span data-ttu-id="2f06f-250">Se si usa un <xref:System.Configuration.ConfigurationBuilder> personalizzato per caricare la configurazione delle opzioni da un file di impostazioni, verificare che il percorso base sia impostato correttamente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-250">When using a custom <xref:System.Configuration.ConfigurationBuilder> to load options configuration from a settings file, confirm that the base path is set correctly:</span></span>
>
> ```csharp
> var configBuilder = new ConfigurationBuilder()
>    .SetBasePath(Directory.GetCurrentDirectory())
>    .AddJsonFile("appsettings.json", optional: true);
> var config = configBuilder.Build();
>
> services.Configure<MyOptions>(config);
> ```
>
> <span data-ttu-id="2f06f-251">L'impostazione esplicita del percorso base non è necessaria se si carica la configurazione delle opzioni dal file di impostazioni tramite <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-251">Explicitly setting the base path isn't required when loading options configuration from the settings file via <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.</span></span>

## <a name="configure-simple-options-with-a-delegate"></a><span data-ttu-id="2f06f-252">Configurare opzioni semplici con un delegato</span><span class="sxs-lookup"><span data-stu-id="2f06f-252">Configure simple options with a delegate</span></span>

<span data-ttu-id="2f06f-253">La configurazione di opzioni semplici con un delegato è illustrata nell'Esempio 2 dell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-253">Configuring simple options with a delegate is demonstrated as Example 2 in the sample app.</span></span>

<span data-ttu-id="2f06f-254">Usare un delegato per impostare i valori delle opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-254">Use a delegate to set options values.</span></span> <span data-ttu-id="2f06f-255">L'app di esempio usa la classe `MyOptionsWithDelegateConfig` (*Models/MyOptionsWithDelegateConfig.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-255">The sample app uses the `MyOptionsWithDelegateConfig` class (*Models/MyOptionsWithDelegateConfig.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptionsWithDelegateConfig.cs?name=snippet1)]

<span data-ttu-id="2f06f-256">Nel codice seguente viene aggiunto un secondo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-256">In the following code, a second <xref:Microsoft.Extensions.Options.IConfigureOptions%601> service is added to the service container.</span></span> <span data-ttu-id="2f06f-257">Viene usato un delegato per configurare l'associazione a `MyOptionsWithDelegateConfig`:</span><span class="sxs-lookup"><span data-stu-id="2f06f-257">It uses a delegate to configure the binding with `MyOptionsWithDelegateConfig`:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example2)]

<span data-ttu-id="2f06f-258">*Index.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="2f06f-258">*Index.cshtml.cs*:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3,9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example2)]

<span data-ttu-id="2f06f-259">È possibile aggiungere più provider di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-259">You can add multiple configuration providers.</span></span> <span data-ttu-id="2f06f-260">I provider di configurazione sono disponibili dai pacchetti NuGet e vengono applicati nell'ordine in cui sono registrati.</span><span class="sxs-lookup"><span data-stu-id="2f06f-260">Configuration providers are available from NuGet packages and are applied in the order that they're registered.</span></span> <span data-ttu-id="2f06f-261">Per altre informazioni, vedere <xref:fundamentals/configuration/index>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-261">For more information, see <xref:fundamentals/configuration/index>.</span></span>

<span data-ttu-id="2f06f-262">Ogni chiamata a <xref:Microsoft.Extensions.Options.IConfigureOptions%601.Configure*> aggiunge un servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-262">Each call to <xref:Microsoft.Extensions.Options.IConfigureOptions%601.Configure*> adds an <xref:Microsoft.Extensions.Options.IConfigureOptions%601> service to the service container.</span></span> <span data-ttu-id="2f06f-263">Nell'esempio precedente, i valori di `Option1` e `Option2` sono entrambi specificati in *appsettings.json* , ma i valori di `Option1` e `Option2` vengono sottoposti a override dal delegato configurato.</span><span class="sxs-lookup"><span data-stu-id="2f06f-263">In the preceding example, the values of `Option1` and `Option2` are both specified in *appsettings.json*, but the values of `Option1` and `Option2` are overridden by the configured delegate.</span></span>

<span data-ttu-id="2f06f-264">Quando sono abilitati più servizi di configurazione, l'origine di configurazione più recente specificata *ha priorità* e imposta il valore di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-264">When more than one configuration service is enabled, the last configuration source specified *wins* and sets the configuration value.</span></span> <span data-ttu-id="2f06f-265">Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:</span><span class="sxs-lookup"><span data-stu-id="2f06f-265">When the app is run, the page model's `OnGet` method returns a string showing the option class values:</span></span>

```html
delegate_option1 = value1_configured_by_delegate, delegate_option2 = 500
```

## <a name="suboptions-configuration"></a><span data-ttu-id="2f06f-266">Configurazione delle opzioni secondarie</span><span class="sxs-lookup"><span data-stu-id="2f06f-266">Suboptions configuration</span></span>

<span data-ttu-id="2f06f-267">La configurazione delle opzioni secondarie è illustrata nell'Esempio 3 dell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-267">Suboptions configuration is demonstrated as Example 3 in the sample app.</span></span>

<span data-ttu-id="2f06f-268">Le app devono creare classi di opzioni che riguardano gruppi di scenari (classi) specifici nell'app.</span><span class="sxs-lookup"><span data-stu-id="2f06f-268">Apps should create options classes that pertain to specific scenario groups (classes) in the app.</span></span> <span data-ttu-id="2f06f-269">Le parti dell'app che richiedono valori di configurazione devono avere accesso solo ai valori di configurazione necessari.</span><span class="sxs-lookup"><span data-stu-id="2f06f-269">Parts of the app that require configuration values should only have access to the configuration values that they use.</span></span>

<span data-ttu-id="2f06f-270">Durante l'associazione delle opzioni alla configurazione, ogni proprietà del tipo di opzioni viene associata a una chiave di configurazione con formato `property[:sub-property:]`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-270">When binding options to configuration, each property in the options type is bound to a configuration key of the form `property[:sub-property:]`.</span></span> <span data-ttu-id="2f06f-271">Ad esempio, la `MyOptions.Option1` proprietà è associata alla chiave `Option1` , che viene letta dalla `option1` Proprietà in *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="2f06f-271">For example, the `MyOptions.Option1` property is bound to the key `Option1`, which is read from the `option1` property in *appsettings.json*.</span></span>

<span data-ttu-id="2f06f-272">Nel codice seguente viene aggiunto un terzo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-272">In the following code, a third <xref:Microsoft.Extensions.Options.IConfigureOptions%601> service is added to the service container.</span></span> <span data-ttu-id="2f06f-273">Viene associato `MySubOptions` alla sezione `subsection` del *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="2f06f-273">It binds `MySubOptions` to the section `subsection` of the *appsettings.json* file:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example3)]

<span data-ttu-id="2f06f-274">Il `GetSection` metodo richiede lo <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> spazio dei nomi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-274">The `GetSection` method requires the <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> namespace.</span></span>

<span data-ttu-id="2f06f-275">Il file dell'esempio *appsettings.json* definisce un `subsection` membro con chiavi per `suboption1` e `suboption2` :</span><span class="sxs-lookup"><span data-stu-id="2f06f-275">The sample's *appsettings.json* file defines a `subsection` member with keys for `suboption1` and `suboption2`:</span></span>

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=4-7)]

<span data-ttu-id="2f06f-276">La classe `MySubOptions` definisce le proprietà `SubOption1` e `SubOption2` per contenere i valori delle opzioni (*Models/MySubOptions.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-276">The `MySubOptions` class defines properties, `SubOption1` and `SubOption2`, to hold the options values (*Models/MySubOptions.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MySubOptions.cs?name=snippet1)]

<span data-ttu-id="2f06f-277">Il metodo `OnGet` del modello di pagina restituisce una stringa con i valori delle opzioni (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-277">The page model's `OnGet` method returns a string with the options values (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=4,10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example3)]

<span data-ttu-id="2f06f-278">Quando viene eseguita l'app, il metodo `OnGet` restituisce una stringa che mostra i valori delle classi delle opzioni secondarie:</span><span class="sxs-lookup"><span data-stu-id="2f06f-278">When the app is run, the `OnGet` method returns a string showing the suboption class values:</span></span>

```html
subOption1 = subvalue1_from_json, subOption2 = 200
```

## <a name="options-injection"></a><span data-ttu-id="2f06f-279">Inserimento di opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-279">Options injection</span></span>

<span data-ttu-id="2f06f-280">L'inserimento di opzioni viene illustrato come esempio 4 nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-280">Options injection is demonstrated as Example 4 in the sample app.</span></span>

<span data-ttu-id="2f06f-281">Inserisci <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in:</span><span class="sxs-lookup"><span data-stu-id="2f06f-281">Inject <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> into:</span></span>

* <span data-ttu-id="2f06f-282">Una Razor pagina o una visualizzazione MVC con la [`@inject`](xref:mvc/views/razor#inject) Razor direttiva.</span><span class="sxs-lookup"><span data-stu-id="2f06f-282">A Razor page or MVC view with the [`@inject`](xref:mvc/views/razor#inject) Razor directive.</span></span>
* <span data-ttu-id="2f06f-283">Modello di pagina o di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-283">A page or view model.</span></span>

<span data-ttu-id="2f06f-284">L'esempio seguente dall'app di esempio inserisce <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> un modello di pagina (*pages/index. cshtml. cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-284">The following example from the sample app injects <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> into a page model (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example4)]

<span data-ttu-id="2f06f-285">L'app di esempio mostra come inserire `IOptionsMonitor<MyOptions>` con una direttiva `@inject`:</span><span class="sxs-lookup"><span data-stu-id="2f06f-285">The sample app shows how to inject `IOptionsMonitor<MyOptions>` with an `@inject` directive:</span></span>

[!code-cshtml[](options/samples/2.x/OptionsSample/Pages/Index.cshtml?range=1-10&highlight=4)]

<span data-ttu-id="2f06f-286">Quando l'app viene eseguita, i valori delle opzioni sono visibili nella pagina di cui è stato eseguito il rendering:</span><span class="sxs-lookup"><span data-stu-id="2f06f-286">When the app is run, the options values are shown in the rendered page:</span></span>

![I valori delle opzioni Option1: value1_from_json e Option2: -1 vengono caricati dal modello e tramite inserimento nella visualizzazione.](options/_static/view.png)

## <a name="reload-configuration-data-with-ioptionssnapshot"></a><span data-ttu-id="2f06f-288">Ricaricare i dati di configurazione con IOptionsSnapshot</span><span class="sxs-lookup"><span data-stu-id="2f06f-288">Reload configuration data with IOptionsSnapshot</span></span>

<span data-ttu-id="2f06f-289">Il ricaricamento dei dati di configurazione con <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> viene illustrato nell'esempio 5 nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-289">Reloading configuration data with <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> is demonstrated in Example 5 in the sample app.</span></span>

<span data-ttu-id="2f06f-290">Usando <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> , le opzioni vengono calcolate una volta per ogni richiesta in caso di accesso e memorizzati nella cache per la durata della richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-290">Using <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601>, options are computed once per request when accessed and cached for the lifetime of the request.</span></span>

<span data-ttu-id="2f06f-291">La differenza tra `IOptionsMonitor` e `IOptionsSnapshot` è che:</span><span class="sxs-lookup"><span data-stu-id="2f06f-291">The difference between `IOptionsMonitor` and `IOptionsSnapshot` is that:</span></span>

* <span data-ttu-id="2f06f-292">`IOptionsMonitor` è un [servizio singleton](xref:fundamentals/dependency-injection#singleton) che recupera i valori correnti delle opzioni in qualsiasi momento, operazione particolarmente utile nelle dipendenze singleton.</span><span class="sxs-lookup"><span data-stu-id="2f06f-292">`IOptionsMonitor` is a [singleton service](xref:fundamentals/dependency-injection#singleton) that retrieves current option values at any time, which is especially useful in singleton dependencies.</span></span>
* <span data-ttu-id="2f06f-293">`IOptionsSnapshot` è un [servizio con ambito](xref:fundamentals/dependency-injection#scoped) e fornisce uno snapshot delle opzioni nel momento in cui `IOptionsSnapshot<T>` viene costruito l'oggetto.</span><span class="sxs-lookup"><span data-stu-id="2f06f-293">`IOptionsSnapshot` is a [scoped service](xref:fundamentals/dependency-injection#scoped) and provides a snapshot of the options at the time the `IOptionsSnapshot<T>` object is constructed.</span></span> <span data-ttu-id="2f06f-294">Gli snapshot delle opzioni sono progettati per l'uso con dipendenze temporanee e con ambito.</span><span class="sxs-lookup"><span data-stu-id="2f06f-294">Options snapshots are designed for use with transient and scoped dependencies.</span></span>

<span data-ttu-id="2f06f-295">Nell'esempio seguente viene illustrato come viene creato un nuovo oggetto <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> dopo *appsettings.json* le modifiche (*pages/index. cshtml. cs*).</span><span class="sxs-lookup"><span data-stu-id="2f06f-295">The following example demonstrates how a new <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> is created after *appsettings.json* changes (*Pages/Index.cshtml.cs*).</span></span> <span data-ttu-id="2f06f-296">Più richieste al server restituiscono valori costanti forniti dal *appsettings.json* file fino a quando il file non viene modificato e la configurazione viene ricaricata.</span><span class="sxs-lookup"><span data-stu-id="2f06f-296">Multiple requests to the server return constant values provided by the *appsettings.json* file until the file is changed and configuration reloads.</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=12)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=5,11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example5)]

<span data-ttu-id="2f06f-297">Nell'immagine seguente vengono illustrati `option1` i `option2` valori iniziali e caricati dal *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="2f06f-297">The following image shows the initial `option1` and `option2` values loaded from the *appsettings.json* file:</span></span>

```html
snapshot option1 = value1_from_json, snapshot option2 = -1
```

<span data-ttu-id="2f06f-298">Modificare i valori nel *appsettings.json* file in `value1_from_json UPDATED` e `200` .</span><span class="sxs-lookup"><span data-stu-id="2f06f-298">Change the values in the *appsettings.json* file to `value1_from_json UPDATED` and `200`.</span></span> <span data-ttu-id="2f06f-299">Salvare il file *appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="2f06f-299">Save the *appsettings.json* file.</span></span> <span data-ttu-id="2f06f-300">Aggiornare il browser per visualizzare i valori delle opzioni aggiornati:</span><span class="sxs-lookup"><span data-stu-id="2f06f-300">Refresh the browser to see that the options values are updated:</span></span>

```html
snapshot option1 = value1_from_json UPDATED, snapshot option2 = 200
```

## <a name="named-options-support-with-iconfigurenamedoptions"></a><span data-ttu-id="2f06f-301">Supporto delle opzioni denominate con IConfigureNamedOptions</span><span class="sxs-lookup"><span data-stu-id="2f06f-301">Named options support with IConfigureNamedOptions</span></span>

<span data-ttu-id="2f06f-302">Il supporto delle opzioni denominate con <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> è illustrato nell'Esempio 6 nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-302">Named options support with <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> is demonstrated as Example 6 in the sample app.</span></span>

<span data-ttu-id="2f06f-303">Il supporto delle opzioni denominate consente all'app di distinguere le configurazioni delle opzioni denominate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-303">Named options support allows the app to distinguish between named options configurations.</span></span> <span data-ttu-id="2f06f-304">Nell'app di esempio le opzioni denominate sono dichiarate con [OptionsServiceCollectionExtensions.ConfigUre](xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure*), che chiama [ConfigureNamedOptions \<TOptions> . Configurare](xref:Microsoft.Extensions.Options.ConfigureNamedOptions`1.Configure*) il metodo di estensione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-304">In the sample app, named options are declared with [OptionsServiceCollectionExtensions.Configure](xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure*), which calls the [ConfigureNamedOptions\<TOptions>.Configure](xref:Microsoft.Extensions.Options.ConfigureNamedOptions`1.Configure*) extension method.</span></span> <span data-ttu-id="2f06f-305">Le opzioni denominate fanno distinzione maiuscole/minuscole</span><span class="sxs-lookup"><span data-stu-id="2f06f-305">Named options are case sensitive.</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example6)]

<span data-ttu-id="2f06f-306">L'app di esempio accede alle opzioni denominate con <xref:Microsoft.Extensions.Options.IOptionsSnapshot`1.Get*> (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-306">The sample app accesses the named options with <xref:Microsoft.Extensions.Options.IOptionsSnapshot`1.Get*> (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=13-14)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=6,12-13)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example6)]

<span data-ttu-id="2f06f-307">Eseguendo l'app di esempio, vengono restituite le opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-307">Running the sample app, the named options are returned:</span></span>

```html
named_options_1: option1 = value1_from_json, option2 = -1
named_options_2: option1 = named_options_2_value1_from_action, option2 = 5
```

<span data-ttu-id="2f06f-308">`named_options_1` i valori vengono forniti dalla configurazione, che vengono caricati dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="2f06f-308">`named_options_1` values are provided from configuration, which are loaded from the *appsettings.json* file.</span></span> <span data-ttu-id="2f06f-309">I valori `named_options_2` sono specificati da:</span><span class="sxs-lookup"><span data-stu-id="2f06f-309">`named_options_2` values are provided by:</span></span>

* <span data-ttu-id="2f06f-310">Delegato `named_options_2` in `ConfigureServices` per `Option1`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-310">The `named_options_2` delegate in `ConfigureServices` for `Option1`.</span></span>
* <span data-ttu-id="2f06f-311">Valore predefinito per `Option2` specificato dalla classe `MyOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-311">The default value for `Option2` provided by the `MyOptions` class.</span></span>

## <a name="configure-all-options-with-the-configureall-method"></a><span data-ttu-id="2f06f-312">Configurare tutte le opzioni con il metodo ConfigureAll</span><span class="sxs-lookup"><span data-stu-id="2f06f-312">Configure all options with the ConfigureAll method</span></span>

<span data-ttu-id="2f06f-313">Configurare tutte le istanze delle opzioni con il metodo <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-313">Configure all options instances with the <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> method.</span></span> <span data-ttu-id="2f06f-314">Il codice seguente configura `Option1` per tutte le istanze di configurazione con un valore comune.</span><span class="sxs-lookup"><span data-stu-id="2f06f-314">The following code configures `Option1` for all configuration instances with a common value.</span></span> <span data-ttu-id="2f06f-315">Aggiungere manualmente il codice seguente al metodo `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2f06f-315">Add the following code manually to the `Startup.ConfigureServices` method:</span></span>

```csharp
services.ConfigureAll<MyOptions>(myOptions => 
{
    myOptions.Option1 = "ConfigureAll replacement value";
});
```

<span data-ttu-id="2f06f-316">L'esecuzione dell'app di esempio dopo l'aggiunta del codice produce il risultato seguente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-316">Running the sample app after adding the code produces the following result:</span></span>

```html
named_options_1: option1 = ConfigureAll replacement value, option2 = -1
named_options_2: option1 = ConfigureAll replacement value, option2 = 5
```

> [!NOTE]
> <span data-ttu-id="2f06f-317">Tutte le opzioni sono istanze denominate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-317">All options are named instances.</span></span> <span data-ttu-id="2f06f-318">Le istanze di <xref:Microsoft.Extensions.Options.IConfigureOptions%601> esistenti sono trattate come se fossero indirizzate all'istanza di `Options.DefaultName`, ovvero `string.Empty`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-318">Existing <xref:Microsoft.Extensions.Options.IConfigureOptions%601> instances are treated as targeting the `Options.DefaultName` instance, which is `string.Empty`.</span></span> <span data-ttu-id="2f06f-319"><xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> implementa anche <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-319"><xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> also implements <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span></span> <span data-ttu-id="2f06f-320">L'implementazione predefinita di <xref:Microsoft.Extensions.Options.IOptionsFactory%601> include la logica per usarle in modo appropriato.</span><span class="sxs-lookup"><span data-stu-id="2f06f-320">The default implementation of the <xref:Microsoft.Extensions.Options.IOptionsFactory%601> has logic to use each appropriately.</span></span> <span data-ttu-id="2f06f-321">L'opzione denominata `null` viene usata per avere come destinazione tutte le istanze denominate anziché un'istanza specifica denominata (questa convenzione è usata da <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> e <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*>).</span><span class="sxs-lookup"><span data-stu-id="2f06f-321">The `null` named option is used to target all of the named instances instead of a specific named instance (<xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> and <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> use this convention).</span></span>

## <a name="optionsbuilder-api"></a><span data-ttu-id="2f06f-322">API OptionsBuilder</span><span class="sxs-lookup"><span data-stu-id="2f06f-322">OptionsBuilder API</span></span>

<span data-ttu-id="2f06f-323"><xref:Microsoft.Extensions.Options.OptionsBuilder%601> viene usata per configurare le istanze di `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-323"><xref:Microsoft.Extensions.Options.OptionsBuilder%601> is used to configure `TOptions` instances.</span></span> <span data-ttu-id="2f06f-324">`OptionsBuilder` semplifica la creazione di opzioni denominate perché è costituita da un singolo parametro nella chiamata iniziale ad `AddOptions<TOptions>(string optionsName)` invece di essere visualizzata in tutte le chiamate successive.</span><span class="sxs-lookup"><span data-stu-id="2f06f-324">`OptionsBuilder` streamlines creating named options as it's only a single parameter to the initial `AddOptions<TOptions>(string optionsName)` call instead of appearing in all of the subsequent calls.</span></span> <span data-ttu-id="2f06f-325">La convalida delle opzioni e gli overload `ConfigureOptions` che accettano le dipendenze dei servizi sono disponibili solo tramite `OptionsBuilder`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-325">Options validation and the `ConfigureOptions` overloads that accept service dependencies are only available via `OptionsBuilder`.</span></span>

```csharp
// Options.DefaultName = "" is used.
services.AddOptions<MyOptions>().Configure(o => o.Property = "default");

services.AddOptions<MyOptions>("optionalName")
    .Configure(o => o.Property = "named");
```

## <a name="use-di-services-to-configure-options"></a><span data-ttu-id="2f06f-326">Usare i servizi di inserimento delle dipendenze per configurare le opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-326">Use DI services to configure options</span></span>

<span data-ttu-id="2f06f-327">È possibile accedere ad altri servizi dall'inserimento delle dipendenze durante la configurazione delle opzioni in due modi:</span><span class="sxs-lookup"><span data-stu-id="2f06f-327">You can access other services from dependency injection while configuring options in two ways:</span></span>

* <span data-ttu-id="2f06f-328">Passare un delegato di configurazione da [configurare](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) in [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1).</span><span class="sxs-lookup"><span data-stu-id="2f06f-328">Pass a configuration delegate to [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) on [OptionsBuilder\<TOptions>](xref:Microsoft.Extensions.Options.OptionsBuilder`1).</span></span> <span data-ttu-id="2f06f-329">[OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1) fornisce overload di [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) che consentono di usare fino a cinque servizi per configurare le opzioni:</span><span class="sxs-lookup"><span data-stu-id="2f06f-329">[OptionsBuilder\<TOptions>](xref:Microsoft.Extensions.Options.OptionsBuilder`1) provides overloads of [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) that allow you to use up to five services to configure options:</span></span>

  ```csharp
  services.AddOptions<MyOptions>("optionalName")
      .Configure<Service1, Service2, Service3, Service4, Service5>(
          (o, s, s2, s3, s4, s5) => 
              o.Property = DoSomethingWith(s, s2, s3, s4, s5));
  ```

* <span data-ttu-id="2f06f-330">Creare un tipo personalizzato che implementa <xref:Microsoft.Extensions.Options.IConfigureOptions%601> o <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e registrare il tipo come servizio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-330">Create your own type that implements <xref:Microsoft.Extensions.Options.IConfigureOptions%601> or <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> and register the type as a service.</span></span>

<span data-ttu-id="2f06f-331">È consigliabile passare un delegato di configurazione a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) perché la creazione di un servizio è più complessa.</span><span class="sxs-lookup"><span data-stu-id="2f06f-331">We recommend passing a configuration delegate to [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*), since creating a service is more complex.</span></span> <span data-ttu-id="2f06f-332">La creazione di un tipo personalizzato equivale alle operazioni che il framework esegue automaticamente quando si usa [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*).</span><span class="sxs-lookup"><span data-stu-id="2f06f-332">Creating your own type is equivalent to what the framework does for you when you use [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*).</span></span> <span data-ttu-id="2f06f-333">La chiamata a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registra un'interfaccia <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> generica temporanea, che ha un costruttore che accetta i tipi di servizi generici specificati.</span><span class="sxs-lookup"><span data-stu-id="2f06f-333">Calling [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registers a transient generic <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601>, which has a constructor that accepts the generic service types specified.</span></span> 

## <a name="options-validation"></a><span data-ttu-id="2f06f-334">Convalida delle opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-334">Options validation</span></span>

<span data-ttu-id="2f06f-335">La convalida le opzioni consente di convalidare le opzioni quando vengono configurate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-335">Options validation allows you to validate options when options are configured.</span></span> <span data-ttu-id="2f06f-336">Chiamare `Validate` con un metodo di convalida che restituisce `true` se le opzioni sono valide e `false` se non sono valide:</span><span class="sxs-lookup"><span data-stu-id="2f06f-336">Call `Validate` with a validation method that returns `true` if options are valid and `false` if they aren't valid:</span></span>

```csharp
// Registration
services.AddOptions<MyOptions>("optionalOptionsName")
    .Configure(o => { }) // Configure the options
    .Validate(o => YourValidationShouldReturnTrueIfValid(o), 
        "custom error");

// Consumption
var monitor = services.BuildServiceProvider()
    .GetService<IOptionsMonitor<MyOptions>>();
  
try
{
    var options = monitor.Get("optionalOptionsName");
}
catch (OptionsValidationException e) 
{
   // e.OptionsName returns "optionalOptionsName"
   // e.OptionsType returns typeof(MyOptions)
   // e.Failures returns a list of errors, which would contain 
   //     "custom error"
}
```

<span data-ttu-id="2f06f-337">L'esempio precedente imposta l'istanza di opzioni denominata su `optionalOptionsName`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-337">The preceding example sets the named options instance to `optionalOptionsName`.</span></span> <span data-ttu-id="2f06f-338">L'istanza di opzioni predefinita è `Options.DefaultName`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-338">The default options instance is `Options.DefaultName`.</span></span>

<span data-ttu-id="2f06f-339">La convalida viene eseguita quando viene creata l'istanza di opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-339">Validation runs when the options instance is created.</span></span> <span data-ttu-id="2f06f-340">Si garantisce che un'istanza di opzioni superi la convalida la prima volta che si accede.</span><span class="sxs-lookup"><span data-stu-id="2f06f-340">An options instance is guaranteed to pass validation the first time it's accessed.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="2f06f-341">La convalida delle opzioni non protegge le modifiche delle opzioni dopo la creazione dell'istanza di Options.</span><span class="sxs-lookup"><span data-stu-id="2f06f-341">Options validation doesn't guard against options modifications after the options instance is created.</span></span> <span data-ttu-id="2f06f-342">Le opzioni, ad esempio, `IOptionsSnapshot` vengono create e convalidate una volta per ogni richiesta quando si accede per la prima volta alle opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-342">For example, `IOptionsSnapshot` options are created and validated once per request when the options are first accessed.</span></span> <span data-ttu-id="2f06f-343">Le `IOptionsSnapshot` Opzioni non vengono convalidate nuovamente ai tentativi *di accesso successivi per la stessa richiesta*.</span><span class="sxs-lookup"><span data-stu-id="2f06f-343">The `IOptionsSnapshot` options aren't validated again on subsequent access attempts *for the same request*.</span></span>

<span data-ttu-id="2f06f-344">Il metodo `Validate` accetta `Func<TOptions, bool>`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-344">The `Validate` method accepts a `Func<TOptions, bool>`.</span></span> <span data-ttu-id="2f06f-345">Per personalizzare completamente la convalida, implementare `IValidateOptions<TOptions>`, che consente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-345">To fully customize validation, implement `IValidateOptions<TOptions>`, which allows:</span></span>

* <span data-ttu-id="2f06f-346">La convalida di più tipi di opzioni: `class ValidateTwo : IValidateOptions<Option1>, IValidationOptions<Option2>`</span><span class="sxs-lookup"><span data-stu-id="2f06f-346">Validation of multiple options types: `class ValidateTwo : IValidateOptions<Option1>, IValidationOptions<Option2>`</span></span>
* <span data-ttu-id="2f06f-347">La convalida che dipende da un altro tipo di opzione: `public DependsOnAnotherOptionValidator(IOptionsMonitor<AnotherOption> options)`</span><span class="sxs-lookup"><span data-stu-id="2f06f-347">Validation that depends on another option type: `public DependsOnAnotherOptionValidator(IOptionsMonitor<AnotherOption> options)`</span></span>

<span data-ttu-id="2f06f-348">`IValidateOptions` convalida:</span><span class="sxs-lookup"><span data-stu-id="2f06f-348">`IValidateOptions` validates:</span></span>

* <span data-ttu-id="2f06f-349">Un'istanza di opzioni denominata specifica.</span><span class="sxs-lookup"><span data-stu-id="2f06f-349">A specific named options instance.</span></span>
* <span data-ttu-id="2f06f-350">Tutte le opzioni quando `name` è `null`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-350">All options when `name` is `null`.</span></span>

<span data-ttu-id="2f06f-351">Restituire `ValidateOptionsResult` dall'implementazione dell'interfaccia:</span><span class="sxs-lookup"><span data-stu-id="2f06f-351">Return a `ValidateOptionsResult` from your implementation of the interface:</span></span>

```csharp
public interface IValidateOptions<TOptions> where TOptions : class
{
    ValidateOptionsResult Validate(string name, TOptions options);
}
```

<span data-ttu-id="2f06f-352">La convalida basata sull'annotazione dei dati è disponibile dal pacchetto [Microsoft. Extensions. Options. DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) chiamando il <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations*> metodo su `OptionsBuilder<TOptions>` .</span><span class="sxs-lookup"><span data-stu-id="2f06f-352">Data Annotation-based validation is available from the [Microsoft.Extensions.Options.DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) package by calling the <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations*> method on `OptionsBuilder<TOptions>`.</span></span> <span data-ttu-id="2f06f-353">`Microsoft.Extensions.Options.DataAnnotations` è incluso nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).</span><span class="sxs-lookup"><span data-stu-id="2f06f-353">`Microsoft.Extensions.Options.DataAnnotations` is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app).</span></span>

```csharp
using Microsoft.Extensions.DependencyInjection;

private class AnnotatedOptions
{
    [Required]
    public string Required { get; set; }

    [StringLength(5, ErrorMessage = "Too long.")]
    public string StringLength { get; set; }

    [Range(-5, 5, ErrorMessage = "Out of range.")]
    public int IntRange { get; set; }
}

[Fact]
public void CanValidateDataAnnotations()
{
    var services = new ServiceCollection();
    services.AddOptions<AnnotatedOptions>()
        .Configure(o =>
        {
            o.StringLength = "111111";
            o.IntRange = 10;
            o.Custom = "nowhere";
        })
        .ValidateDataAnnotations();

    var sp = services.BuildServiceProvider();

    var error = Assert.Throws<OptionsValidationException>(() => 
        sp.GetRequiredService<IOptionsMonitor<AnnotatedOptions>>().CurrentValue);
    ValidateFailure<AnnotatedOptions>(error, Options.DefaultName, 1,
        "DataAnnotation validation failed for members Required " +
            "with the error 'The Required field is required.'.",
        "DataAnnotation validation failed for members StringLength " +
            "with the error 'Too long.'.",
        "DataAnnotation validation failed for members IntRange " +
            "with the error 'Out of range.'.");
}
```

<span data-ttu-id="2f06f-354">La convalida eager (fallimento e risposta immediata agli errori all'avvio) è pianificata per una versione futura.</span><span class="sxs-lookup"><span data-stu-id="2f06f-354">Eager validation (fail fast at startup) is under consideration for a future release.</span></span>

## <a name="options-post-configuration"></a><span data-ttu-id="2f06f-355">Post-configurazione delle opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-355">Options post-configuration</span></span>

<span data-ttu-id="2f06f-356">Impostare la post-configurazione con <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-356">Set post-configuration with <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>.</span></span> <span data-ttu-id="2f06f-357">La post-configurazione viene eseguita dopo il completamento della configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>:</span><span class="sxs-lookup"><span data-stu-id="2f06f-357">Post-configuration runs after all <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configuration occurs:</span></span>

```csharp
services.PostConfigure<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<span data-ttu-id="2f06f-358"><xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> è disponibile per la post-configurazione delle opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-358"><xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> is available to post-configure named options:</span></span>

```csharp
services.PostConfigure<MyOptions>("named_options_1", myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<span data-ttu-id="2f06f-359">Usare <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> per la post-configurazione di tutte le istanze di configurazione:</span><span class="sxs-lookup"><span data-stu-id="2f06f-359">Use <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> to post-configure all configuration instances:</span></span>

```csharp
services.PostConfigureAll<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

## <a name="accessing-options-during-startup"></a><span data-ttu-id="2f06f-360">Accesso alle opzioni durante l'avvio</span><span class="sxs-lookup"><span data-stu-id="2f06f-360">Accessing options during startup</span></span>

<span data-ttu-id="2f06f-361"><xref:Microsoft.Extensions.Options.IOptions%601> e <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> possono essere usate in `Startup.Configure`, perché i servizi vengono compilati prima dell'esecuzione del metodo `Configure`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-361"><xref:Microsoft.Extensions.Options.IOptions%601> and <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> can be used in `Startup.Configure`, since services are built before the `Configure` method executes.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IOptionsMonitor<MyOptions> optionsAccessor)
{
    var option1 = optionsAccessor.CurrentValue.Option1;
}
```

<span data-ttu-id="2f06f-362">Non usare <xref:Microsoft.Extensions.Options.IOptions%601> oppure <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-362">Don't use <xref:Microsoft.Extensions.Options.IOptions%601> or <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="2f06f-363">Le opzioni potrebbero avere uno stato incoerente a causa dell'ordinamento delle registrazioni dei servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-363">An inconsistent options state may exist due to the ordering of service registrations.</span></span>

::: moniker-end

::: moniker range="= aspnetcore-2.1"

<span data-ttu-id="2f06f-364">Il modello di opzioni usa le classi per rappresentare i gruppi di impostazioni correlate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-364">The options pattern uses classes to represent groups of related settings.</span></span> <span data-ttu-id="2f06f-365">Quando le [impostazioni di configurazione](xref:fundamentals/configuration/index) vengono isolate in base allo scenario in classi separate, l'app aderisce a due importanti principi di progettazione del software:</span><span class="sxs-lookup"><span data-stu-id="2f06f-365">When [configuration settings](xref:fundamentals/configuration/index) are isolated by scenario into separate classes, the app adheres to two important software engineering principles:</span></span>

* <span data-ttu-id="2f06f-366">Il [principio di separazione dell'interfaccia (ISP) o l'incapsulamento](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): scenari (classi) che dipendono dalle impostazioni di configurazione dipendono solo dalle impostazioni di configurazione che usano.</span><span class="sxs-lookup"><span data-stu-id="2f06f-366">The [Interface Segregation Principle (ISP) or Encapsulation](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): Scenarios (classes) that depend on configuration settings depend only on the configuration settings that they use.</span></span>
* <span data-ttu-id="2f06f-367">[Separazione delle competenze (Separation of Concerns, SoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): le impostazioni di parti diverse dell'app non sono dipendenti o accoppiate l'una con l'altra.</span><span class="sxs-lookup"><span data-stu-id="2f06f-367">[Separation of Concerns](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): Settings for different parts of the app aren't dependent or coupled to one another.</span></span>

<span data-ttu-id="2f06f-368">Le opzioni offrono anche un meccanismo per convalidare i dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-368">Options also provide a mechanism to validate configuration data.</span></span> <span data-ttu-id="2f06f-369">Per altre informazioni, vedere la sezione [Opzioni di convalida](#options-validation).</span><span class="sxs-lookup"><span data-stu-id="2f06f-369">For more information, see the [Options validation](#options-validation) section.</span></span>

<span data-ttu-id="2f06f-370">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="2f06f-370">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2f06f-371">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2f06f-371">Prerequisites</span></span>

<span data-ttu-id="2f06f-372">Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) oppure aggiungere un riferimento al pacchetto [Microsoft.Extensions.Options.ConfigurationExtensions](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/).</span><span class="sxs-lookup"><span data-stu-id="2f06f-372">Reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) or add a package reference to the [Microsoft.Extensions.Options.ConfigurationExtensions](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/) package.</span></span>

## <a name="options-interfaces"></a><span data-ttu-id="2f06f-373">Interfacce per le opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-373">Options interfaces</span></span>

<span data-ttu-id="2f06f-374"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> consente di recuperare le opzioni e gestire le notifiche di opzioni per le istanze `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-374"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> is used to retrieve options and manage options notifications for `TOptions` instances.</span></span> <span data-ttu-id="2f06f-375"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> supporta gli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="2f06f-375"><xref:Microsoft.Extensions.Options.IOptionsMonitor%601> supports the following scenarios:</span></span>

* <span data-ttu-id="2f06f-376">Notifiche di modifica</span><span class="sxs-lookup"><span data-stu-id="2f06f-376">Change notifications</span></span>
* [<span data-ttu-id="2f06f-377">Opzioni denominate</span><span class="sxs-lookup"><span data-stu-id="2f06f-377">Named options</span></span>](#named-options-support-with-iconfigurenamedoptions)
* [<span data-ttu-id="2f06f-378">Configurazione ricaricabile</span><span class="sxs-lookup"><span data-stu-id="2f06f-378">Reloadable configuration</span></span>](#reload-configuration-data-with-ioptionssnapshot)
* <span data-ttu-id="2f06f-379">Invalidamento selettivo di opzioni (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)</span><span class="sxs-lookup"><span data-stu-id="2f06f-379">Selective options invalidation (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)</span></span>

<span data-ttu-id="2f06f-380">Gli scenari di [post-configurazione](#options-post-configuration) consentono di impostare o modificare le opzioni dopo la configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-380">[Post-configuration](#options-post-configuration) scenarios allow you to set or change options after all <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configuration occurs.</span></span>

<span data-ttu-id="2f06f-381"><xref:Microsoft.Extensions.Options.IOptionsFactory%601> è responsabile della creazione di nuove istanze di opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-381"><xref:Microsoft.Extensions.Options.IOptionsFactory%601> is responsible for creating new options instances.</span></span> <span data-ttu-id="2f06f-382">Include un singolo metodo <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-382">It has a single <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*> method.</span></span> <span data-ttu-id="2f06f-383">L'implementazione predefinita accetta tutte le interfacce <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> registrate ed esegue tutte le configurazioni, seguite dalla post-configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-383">The default implementation takes all registered <xref:Microsoft.Extensions.Options.IConfigureOptions%601> and <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> and runs all the configurations first, followed by the post-configuration.</span></span> <span data-ttu-id="2f06f-384">Fa distinzione tra <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e chiama solo l'interfaccia appropriata.</span><span class="sxs-lookup"><span data-stu-id="2f06f-384">It distinguishes between <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> and <xref:Microsoft.Extensions.Options.IConfigureOptions%601> and only calls the appropriate interface.</span></span>

<span data-ttu-id="2f06f-385"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> viene usata da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per memorizzare nella cache le istanze di `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-385"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> is used by <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> to cache `TOptions` instances.</span></span> <span data-ttu-id="2f06f-386"><xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalida le istanze delle opzioni nel monitoraggio in modo che il valore venga ricalcolato (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>).</span><span class="sxs-lookup"><span data-stu-id="2f06f-386">The <xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalidates options instances in the monitor so that the value is recomputed (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>).</span></span> <span data-ttu-id="2f06f-387">I valori possono essere introdotti manualmente con <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-387">Values can be manually introduced with <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>.</span></span> <span data-ttu-id="2f06f-388">Il metodo <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> viene usato quando tutte le istanze denominate devono essere ricreate su richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-388">The <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> method is used when all named instances should be recreated on demand.</span></span>

<span data-ttu-id="2f06f-389"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> è utile negli scenari in cui le opzioni devono essere ricalcolate a ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-389"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> is useful in scenarios where options should be recomputed on every request.</span></span> <span data-ttu-id="2f06f-390">Per altre informazioni, vedere la sezione [Ricaricare i dati di configurazione con IOptionsSnapshot](#reload-configuration-data-with-ioptionssnapshot).</span><span class="sxs-lookup"><span data-stu-id="2f06f-390">For more information, see the [Reload configuration data with IOptionsSnapshot](#reload-configuration-data-with-ioptionssnapshot) section.</span></span>

<span data-ttu-id="2f06f-391"><xref:Microsoft.Extensions.Options.IOptions%601> può essere usata a supporto delle opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-391"><xref:Microsoft.Extensions.Options.IOptions%601> can be used to support options.</span></span> <span data-ttu-id="2f06f-392">Tuttavia <xref:Microsoft.Extensions.Options.IOptions%601> non supporta gli scenari precedenti di <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-392">However, <xref:Microsoft.Extensions.Options.IOptions%601> doesn't support the preceding scenarios of <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span></span> <span data-ttu-id="2f06f-393">È possibile continuare a usare <xref:Microsoft.Extensions.Options.IOptions%601> in framework e librerie esistenti che già usano l'interfaccia <xref:Microsoft.Extensions.Options.IOptions%601> e non richiedono gli scenari forniti da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-393">You may continue to use <xref:Microsoft.Extensions.Options.IOptions%601> in existing frameworks and libraries that already use the <xref:Microsoft.Extensions.Options.IOptions%601> interface and don't require the scenarios provided by <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.</span></span>

## <a name="general-options-configuration"></a><span data-ttu-id="2f06f-394">Configurazione delle opzioni generali</span><span class="sxs-lookup"><span data-stu-id="2f06f-394">General options configuration</span></span>

<span data-ttu-id="2f06f-395">La configurazione delle opzioni generali è illustrata nell'Esempio 1 nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-395">General options configuration is demonstrated as Example 1 in the sample app.</span></span>

<span data-ttu-id="2f06f-396">Una classe di opzioni deve essere non astratta con un costruttore pubblico senza parametri.</span><span class="sxs-lookup"><span data-stu-id="2f06f-396">An options class must be non-abstract with a public parameterless constructor.</span></span> <span data-ttu-id="2f06f-397">La classe seguente, `MyOptions`, ha due proprietà, `Option1` e `Option2`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-397">The following class, `MyOptions`, has two properties, `Option1` and `Option2`.</span></span> <span data-ttu-id="2f06f-398">Sebbene l'impostazione dei valori predefiniti sia facoltativa, il costruttore della classe nell'esempio seguente imposta il valore predefinito di `Option1`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-398">Setting default values is optional, but the class constructor in the following example sets the default value of `Option1`.</span></span> <span data-ttu-id="2f06f-399">`Option2` ha un valore impostato tramite l'inizializzazione diretta della proprietà (*Models/MyOptions.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-399">`Option2` has a default value set by initializing the property directly (*Models/MyOptions.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptions.cs?name=snippet1)]

<span data-ttu-id="2f06f-400">La classe `MyOptions` viene aggiunta al contenitore di servizi con <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> e associata alla configurazione:</span><span class="sxs-lookup"><span data-stu-id="2f06f-400">The `MyOptions` class is added to the service container with <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> and bound to configuration:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example1)]

<span data-ttu-id="2f06f-401">Il modello di pagina seguente usa l'[inserimento delle dipendenze del costruttore](xref:mvc/controllers/dependency-injection) con <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per accedere alle impostazioni (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-401">The following page model uses [constructor dependency injection](xref:mvc/controllers/dependency-injection) with <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> to access the settings (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example1)]

<span data-ttu-id="2f06f-402">Il file dell'esempio *appsettings.json* specifica i valori per `option1` e `option2` :</span><span class="sxs-lookup"><span data-stu-id="2f06f-402">The sample's *appsettings.json* file specifies values for `option1` and `option2`:</span></span>

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=2-3)]

<span data-ttu-id="2f06f-403">Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:</span><span class="sxs-lookup"><span data-stu-id="2f06f-403">When the app is run, the page model's `OnGet` method returns a string showing the option class values:</span></span>

```html
option1 = value1_from_json, option2 = -1
```

> [!NOTE]
> <span data-ttu-id="2f06f-404">Se si usa un <xref:System.Configuration.ConfigurationBuilder> personalizzato per caricare la configurazione delle opzioni da un file di impostazioni, verificare che il percorso base sia impostato correttamente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-404">When using a custom <xref:System.Configuration.ConfigurationBuilder> to load options configuration from a settings file, confirm that the base path is set correctly:</span></span>
>
> ```csharp
> var configBuilder = new ConfigurationBuilder()
>    .SetBasePath(Directory.GetCurrentDirectory())
>    .AddJsonFile("appsettings.json", optional: true);
> var config = configBuilder.Build();
>
> services.Configure<MyOptions>(config);
> ```
>
> <span data-ttu-id="2f06f-405">L'impostazione esplicita del percorso base non è necessaria se si carica la configurazione delle opzioni dal file di impostazioni tramite <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-405">Explicitly setting the base path isn't required when loading options configuration from the settings file via <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.</span></span>

## <a name="configure-simple-options-with-a-delegate"></a><span data-ttu-id="2f06f-406">Configurare opzioni semplici con un delegato</span><span class="sxs-lookup"><span data-stu-id="2f06f-406">Configure simple options with a delegate</span></span>

<span data-ttu-id="2f06f-407">La configurazione di opzioni semplici con un delegato è illustrata nell'Esempio 2 dell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-407">Configuring simple options with a delegate is demonstrated as Example 2 in the sample app.</span></span>

<span data-ttu-id="2f06f-408">Usare un delegato per impostare i valori delle opzioni.</span><span class="sxs-lookup"><span data-stu-id="2f06f-408">Use a delegate to set options values.</span></span> <span data-ttu-id="2f06f-409">L'app di esempio usa la classe `MyOptionsWithDelegateConfig` (*Models/MyOptionsWithDelegateConfig.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-409">The sample app uses the `MyOptionsWithDelegateConfig` class (*Models/MyOptionsWithDelegateConfig.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptionsWithDelegateConfig.cs?name=snippet1)]

<span data-ttu-id="2f06f-410">Nel codice seguente viene aggiunto un secondo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-410">In the following code, a second <xref:Microsoft.Extensions.Options.IConfigureOptions%601> service is added to the service container.</span></span> <span data-ttu-id="2f06f-411">Viene usato un delegato per configurare l'associazione a `MyOptionsWithDelegateConfig`:</span><span class="sxs-lookup"><span data-stu-id="2f06f-411">It uses a delegate to configure the binding with `MyOptionsWithDelegateConfig`:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example2)]

<span data-ttu-id="2f06f-412">*Index.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="2f06f-412">*Index.cshtml.cs*:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3,9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example2)]

<span data-ttu-id="2f06f-413">È possibile aggiungere più provider di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-413">You can add multiple configuration providers.</span></span> <span data-ttu-id="2f06f-414">I provider di configurazione sono disponibili dai pacchetti NuGet e vengono applicati nell'ordine in cui sono registrati.</span><span class="sxs-lookup"><span data-stu-id="2f06f-414">Configuration providers are available from NuGet packages and are applied in the order that they're registered.</span></span> <span data-ttu-id="2f06f-415">Per altre informazioni, vedere <xref:fundamentals/configuration/index>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-415">For more information, see <xref:fundamentals/configuration/index>.</span></span>

<span data-ttu-id="2f06f-416">Ogni chiamata a <xref:Microsoft.Extensions.Options.IConfigureOptions%601.Configure*> aggiunge un servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-416">Each call to <xref:Microsoft.Extensions.Options.IConfigureOptions%601.Configure*> adds an <xref:Microsoft.Extensions.Options.IConfigureOptions%601> service to the service container.</span></span> <span data-ttu-id="2f06f-417">Nell'esempio precedente, i valori di `Option1` e `Option2` sono entrambi specificati in *appsettings.json* , ma i valori di `Option1` e `Option2` vengono sottoposti a override dal delegato configurato.</span><span class="sxs-lookup"><span data-stu-id="2f06f-417">In the preceding example, the values of `Option1` and `Option2` are both specified in *appsettings.json*, but the values of `Option1` and `Option2` are overridden by the configured delegate.</span></span>

<span data-ttu-id="2f06f-418">Quando sono abilitati più servizi di configurazione, l'origine di configurazione più recente specificata *ha priorità* e imposta il valore di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-418">When more than one configuration service is enabled, the last configuration source specified *wins* and sets the configuration value.</span></span> <span data-ttu-id="2f06f-419">Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:</span><span class="sxs-lookup"><span data-stu-id="2f06f-419">When the app is run, the page model's `OnGet` method returns a string showing the option class values:</span></span>

```html
delegate_option1 = value1_configured_by_delegate, delegate_option2 = 500
```

## <a name="suboptions-configuration"></a><span data-ttu-id="2f06f-420">Configurazione delle opzioni secondarie</span><span class="sxs-lookup"><span data-stu-id="2f06f-420">Suboptions configuration</span></span>

<span data-ttu-id="2f06f-421">La configurazione delle opzioni secondarie è illustrata nell'Esempio 3 dell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-421">Suboptions configuration is demonstrated as Example 3 in the sample app.</span></span>

<span data-ttu-id="2f06f-422">Le app devono creare classi di opzioni che riguardano gruppi di scenari (classi) specifici nell'app.</span><span class="sxs-lookup"><span data-stu-id="2f06f-422">Apps should create options classes that pertain to specific scenario groups (classes) in the app.</span></span> <span data-ttu-id="2f06f-423">Le parti dell'app che richiedono valori di configurazione devono avere accesso solo ai valori di configurazione necessari.</span><span class="sxs-lookup"><span data-stu-id="2f06f-423">Parts of the app that require configuration values should only have access to the configuration values that they use.</span></span>

<span data-ttu-id="2f06f-424">Durante l'associazione delle opzioni alla configurazione, ogni proprietà del tipo di opzioni viene associata a una chiave di configurazione con formato `property[:sub-property:]`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-424">When binding options to configuration, each property in the options type is bound to a configuration key of the form `property[:sub-property:]`.</span></span> <span data-ttu-id="2f06f-425">Ad esempio, la `MyOptions.Option1` proprietà è associata alla chiave `Option1` , che viene letta dalla `option1` Proprietà in *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="2f06f-425">For example, the `MyOptions.Option1` property is bound to the key `Option1`, which is read from the `option1` property in *appsettings.json*.</span></span>

<span data-ttu-id="2f06f-426">Nel codice seguente viene aggiunto un terzo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-426">In the following code, a third <xref:Microsoft.Extensions.Options.IConfigureOptions%601> service is added to the service container.</span></span> <span data-ttu-id="2f06f-427">Viene associato `MySubOptions` alla sezione `subsection` del *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="2f06f-427">It binds `MySubOptions` to the section `subsection` of the *appsettings.json* file:</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example3)]

<span data-ttu-id="2f06f-428">Il `GetSection` metodo richiede lo <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> spazio dei nomi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-428">The `GetSection` method requires the <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> namespace.</span></span>

<span data-ttu-id="2f06f-429">Il file dell'esempio *appsettings.json* definisce un `subsection` membro con chiavi per `suboption1` e `suboption2` :</span><span class="sxs-lookup"><span data-stu-id="2f06f-429">The sample's *appsettings.json* file defines a `subsection` member with keys for `suboption1` and `suboption2`:</span></span>

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=4-7)]

<span data-ttu-id="2f06f-430">La classe `MySubOptions` definisce le proprietà `SubOption1` e `SubOption2` per contenere i valori delle opzioni (*Models/MySubOptions.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-430">The `MySubOptions` class defines properties, `SubOption1` and `SubOption2`, to hold the options values (*Models/MySubOptions.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MySubOptions.cs?name=snippet1)]

<span data-ttu-id="2f06f-431">Il metodo `OnGet` del modello di pagina restituisce una stringa con i valori delle opzioni (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-431">The page model's `OnGet` method returns a string with the options values (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=4,10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example3)]

<span data-ttu-id="2f06f-432">Quando viene eseguita l'app, il metodo `OnGet` restituisce una stringa che mostra i valori delle classi delle opzioni secondarie:</span><span class="sxs-lookup"><span data-stu-id="2f06f-432">When the app is run, the `OnGet` method returns a string showing the suboption class values:</span></span>

```html
subOption1 = subvalue1_from_json, subOption2 = 200
```

## <a name="options-provided-by-a-view-model-or-with-direct-view-injection"></a><span data-ttu-id="2f06f-433">Opzioni fornite da un modello di visualizzazione o con l'inserimento diretto della visualizzazione</span><span class="sxs-lookup"><span data-stu-id="2f06f-433">Options provided by a view model or with direct view injection</span></span>

<span data-ttu-id="2f06f-434">Le opzioni fornite da un modello di visualizzazione o con l'inserimento diretto della visualizzazione sono illustrate nell'Esempio 4 dell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-434">Options provided by a view model or with direct view injection is demonstrated as Example 4 in the sample app.</span></span>

<span data-ttu-id="2f06f-435">Le opzioni possono essere rese disponibili in un modello di visualizzazione oppure inserendo <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> direttamente in una visualizzazione (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-435">Options can be supplied in a view model or by injecting <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> directly into a view (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example4)]

<span data-ttu-id="2f06f-436">L'app di esempio mostra come inserire `IOptionsMonitor<MyOptions>` con una direttiva `@inject`:</span><span class="sxs-lookup"><span data-stu-id="2f06f-436">The sample app shows how to inject `IOptionsMonitor<MyOptions>` with an `@inject` directive:</span></span>

[!code-cshtml[](options/samples/2.x/OptionsSample/Pages/Index.cshtml?range=1-10&highlight=4)]

<span data-ttu-id="2f06f-437">Quando l'app viene eseguita, i valori delle opzioni sono visibili nella pagina di cui è stato eseguito il rendering:</span><span class="sxs-lookup"><span data-stu-id="2f06f-437">When the app is run, the options values are shown in the rendered page:</span></span>

![I valori delle opzioni Option1: value1_from_json e Option2: -1 vengono caricati dal modello e tramite inserimento nella visualizzazione.](options/_static/view.png)

## <a name="reload-configuration-data-with-ioptionssnapshot"></a><span data-ttu-id="2f06f-439">Ricaricare i dati di configurazione con IOptionsSnapshot</span><span class="sxs-lookup"><span data-stu-id="2f06f-439">Reload configuration data with IOptionsSnapshot</span></span>

<span data-ttu-id="2f06f-440">Il ricaricamento dei dati di configurazione con <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> viene illustrato nell'esempio 5 nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-440">Reloading configuration data with <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> is demonstrated in Example 5 in the sample app.</span></span>

<span data-ttu-id="2f06f-441"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> supporta il ricaricamento delle opzioni con un overhead di elaborazione minimo.</span><span class="sxs-lookup"><span data-stu-id="2f06f-441"><xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> supports reloading options with minimal processing overhead.</span></span>

<span data-ttu-id="2f06f-442">Le opzioni vengono calcolate una volta per richiesta quando viene eseguito l'accesso e la memorizzazione nella cache per la durata della richiesta.</span><span class="sxs-lookup"><span data-stu-id="2f06f-442">Options are computed once per request when accessed and cached for the lifetime of the request.</span></span>

<span data-ttu-id="2f06f-443">Nell'esempio seguente viene illustrato come viene creato un nuovo oggetto <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> dopo *appsettings.json* le modifiche (*pages/index. cshtml. cs*).</span><span class="sxs-lookup"><span data-stu-id="2f06f-443">The following example demonstrates how a new <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> is created after *appsettings.json* changes (*Pages/Index.cshtml.cs*).</span></span> <span data-ttu-id="2f06f-444">Più richieste al server restituiscono valori costanti forniti dal *appsettings.json* file fino a quando il file non viene modificato e la configurazione viene ricaricata.</span><span class="sxs-lookup"><span data-stu-id="2f06f-444">Multiple requests to the server return constant values provided by the *appsettings.json* file until the file is changed and configuration reloads.</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=12)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=5,11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example5)]

<span data-ttu-id="2f06f-445">Nell'immagine seguente vengono illustrati `option1` i `option2` valori iniziali e caricati dal *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="2f06f-445">The following image shows the initial `option1` and `option2` values loaded from the *appsettings.json* file:</span></span>

```html
snapshot option1 = value1_from_json, snapshot option2 = -1
```

<span data-ttu-id="2f06f-446">Modificare i valori nel *appsettings.json* file in `value1_from_json UPDATED` e `200` .</span><span class="sxs-lookup"><span data-stu-id="2f06f-446">Change the values in the *appsettings.json* file to `value1_from_json UPDATED` and `200`.</span></span> <span data-ttu-id="2f06f-447">Salvare il file *appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="2f06f-447">Save the *appsettings.json* file.</span></span> <span data-ttu-id="2f06f-448">Aggiornare il browser per visualizzare i valori delle opzioni aggiornati:</span><span class="sxs-lookup"><span data-stu-id="2f06f-448">Refresh the browser to see that the options values are updated:</span></span>

```html
snapshot option1 = value1_from_json UPDATED, snapshot option2 = 200
```

## <a name="named-options-support-with-iconfigurenamedoptions"></a><span data-ttu-id="2f06f-449">Supporto delle opzioni denominate con IConfigureNamedOptions</span><span class="sxs-lookup"><span data-stu-id="2f06f-449">Named options support with IConfigureNamedOptions</span></span>

<span data-ttu-id="2f06f-450">Il supporto delle opzioni denominate con <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> è illustrato nell'Esempio 6 nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-450">Named options support with <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> is demonstrated as Example 6 in the sample app.</span></span>

<span data-ttu-id="2f06f-451">Il supporto delle opzioni denominate consente all'app di distinguere le configurazioni delle opzioni denominate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-451">Named options support allows the app to distinguish between named options configurations.</span></span> <span data-ttu-id="2f06f-452">Nell'app di esempio le opzioni denominate sono dichiarate con [OptionsServiceCollectionExtensions.ConfigUre](xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure*), che chiama [ConfigureNamedOptions \<TOptions> . Configurare](xref:Microsoft.Extensions.Options.ConfigureNamedOptions`1.Configure*) il metodo di estensione.</span><span class="sxs-lookup"><span data-stu-id="2f06f-452">In the sample app, named options are declared with [OptionsServiceCollectionExtensions.Configure](xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure*), which calls the [ConfigureNamedOptions\<TOptions>.Configure](xref:Microsoft.Extensions.Options.ConfigureNamedOptions`1.Configure*) extension method.</span></span> <span data-ttu-id="2f06f-453">Le opzioni denominate fanno distinzione maiuscole/minuscole</span><span class="sxs-lookup"><span data-stu-id="2f06f-453">Named options are case sensitive.</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example6)]

<span data-ttu-id="2f06f-454">L'app di esempio accede alle opzioni denominate con <xref:Microsoft.Extensions.Options.IOptionsSnapshot`1.Get*> (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2f06f-454">The sample app accesses the named options with <xref:Microsoft.Extensions.Options.IOptionsSnapshot`1.Get*> (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=13-14)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=6,12-13)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example6)]

<span data-ttu-id="2f06f-455">Eseguendo l'app di esempio, vengono restituite le opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-455">Running the sample app, the named options are returned:</span></span>

```html
named_options_1: option1 = value1_from_json, option2 = -1
named_options_2: option1 = named_options_2_value1_from_action, option2 = 5
```

<span data-ttu-id="2f06f-456">`named_options_1` i valori vengono forniti dalla configurazione, che vengono caricati dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="2f06f-456">`named_options_1` values are provided from configuration, which are loaded from the *appsettings.json* file.</span></span> <span data-ttu-id="2f06f-457">I valori `named_options_2` sono specificati da:</span><span class="sxs-lookup"><span data-stu-id="2f06f-457">`named_options_2` values are provided by:</span></span>

* <span data-ttu-id="2f06f-458">Delegato `named_options_2` in `ConfigureServices` per `Option1`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-458">The `named_options_2` delegate in `ConfigureServices` for `Option1`.</span></span>
* <span data-ttu-id="2f06f-459">Valore predefinito per `Option2` specificato dalla classe `MyOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-459">The default value for `Option2` provided by the `MyOptions` class.</span></span>

## <a name="configure-all-options-with-the-configureall-method"></a><span data-ttu-id="2f06f-460">Configurare tutte le opzioni con il metodo ConfigureAll</span><span class="sxs-lookup"><span data-stu-id="2f06f-460">Configure all options with the ConfigureAll method</span></span>

<span data-ttu-id="2f06f-461">Configurare tutte le istanze delle opzioni con il metodo <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-461">Configure all options instances with the <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> method.</span></span> <span data-ttu-id="2f06f-462">Il codice seguente configura `Option1` per tutte le istanze di configurazione con un valore comune.</span><span class="sxs-lookup"><span data-stu-id="2f06f-462">The following code configures `Option1` for all configuration instances with a common value.</span></span> <span data-ttu-id="2f06f-463">Aggiungere manualmente il codice seguente al metodo `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2f06f-463">Add the following code manually to the `Startup.ConfigureServices` method:</span></span>

```csharp
services.ConfigureAll<MyOptions>(myOptions => 
{
    myOptions.Option1 = "ConfigureAll replacement value";
});
```

<span data-ttu-id="2f06f-464">L'esecuzione dell'app di esempio dopo l'aggiunta del codice produce il risultato seguente:</span><span class="sxs-lookup"><span data-stu-id="2f06f-464">Running the sample app after adding the code produces the following result:</span></span>

```html
named_options_1: option1 = ConfigureAll replacement value, option2 = -1
named_options_2: option1 = ConfigureAll replacement value, option2 = 5
```

> [!NOTE]
> <span data-ttu-id="2f06f-465">Tutte le opzioni sono istanze denominate.</span><span class="sxs-lookup"><span data-stu-id="2f06f-465">All options are named instances.</span></span> <span data-ttu-id="2f06f-466">Le istanze di <xref:Microsoft.Extensions.Options.IConfigureOptions%601> esistenti sono trattate come se fossero indirizzate all'istanza di `Options.DefaultName`, ovvero `string.Empty`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-466">Existing <xref:Microsoft.Extensions.Options.IConfigureOptions%601> instances are treated as targeting the `Options.DefaultName` instance, which is `string.Empty`.</span></span> <span data-ttu-id="2f06f-467"><xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> implementa anche <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-467"><xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> also implements <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.</span></span> <span data-ttu-id="2f06f-468">L'implementazione predefinita di <xref:Microsoft.Extensions.Options.IOptionsFactory%601> include la logica per usarle in modo appropriato.</span><span class="sxs-lookup"><span data-stu-id="2f06f-468">The default implementation of the <xref:Microsoft.Extensions.Options.IOptionsFactory%601> has logic to use each appropriately.</span></span> <span data-ttu-id="2f06f-469">L'opzione denominata `null` viene usata per avere come destinazione tutte le istanze denominate anziché un'istanza specifica denominata (questa convenzione è usata da <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> e <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*>).</span><span class="sxs-lookup"><span data-stu-id="2f06f-469">The `null` named option is used to target all of the named instances instead of a specific named instance (<xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> and <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> use this convention).</span></span>

## <a name="optionsbuilder-api"></a><span data-ttu-id="2f06f-470">API OptionsBuilder</span><span class="sxs-lookup"><span data-stu-id="2f06f-470">OptionsBuilder API</span></span>

<span data-ttu-id="2f06f-471"><xref:Microsoft.Extensions.Options.OptionsBuilder%601> viene usata per configurare le istanze di `TOptions`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-471"><xref:Microsoft.Extensions.Options.OptionsBuilder%601> is used to configure `TOptions` instances.</span></span> <span data-ttu-id="2f06f-472">`OptionsBuilder` semplifica la creazione di opzioni denominate perché è costituita da un singolo parametro nella chiamata iniziale ad `AddOptions<TOptions>(string optionsName)` invece di essere visualizzata in tutte le chiamate successive.</span><span class="sxs-lookup"><span data-stu-id="2f06f-472">`OptionsBuilder` streamlines creating named options as it's only a single parameter to the initial `AddOptions<TOptions>(string optionsName)` call instead of appearing in all of the subsequent calls.</span></span> <span data-ttu-id="2f06f-473">La convalida delle opzioni e gli overload `ConfigureOptions` che accettano le dipendenze dei servizi sono disponibili solo tramite `OptionsBuilder`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-473">Options validation and the `ConfigureOptions` overloads that accept service dependencies are only available via `OptionsBuilder`.</span></span>

```csharp
// Options.DefaultName = "" is used.
services.AddOptions<MyOptions>().Configure(o => o.Property = "default");

services.AddOptions<MyOptions>("optionalName")
    .Configure(o => o.Property = "named");
```

## <a name="use-di-services-to-configure-options"></a><span data-ttu-id="2f06f-474">Usare i servizi di inserimento delle dipendenze per configurare le opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-474">Use DI services to configure options</span></span>

<span data-ttu-id="2f06f-475">È possibile accedere ad altri servizi dall'inserimento delle dipendenze durante la configurazione delle opzioni in due modi:</span><span class="sxs-lookup"><span data-stu-id="2f06f-475">You can access other services from dependency injection while configuring options in two ways:</span></span>

* <span data-ttu-id="2f06f-476">Passare un delegato di configurazione da [configurare](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) in [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1).</span><span class="sxs-lookup"><span data-stu-id="2f06f-476">Pass a configuration delegate to [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) on [OptionsBuilder\<TOptions>](xref:Microsoft.Extensions.Options.OptionsBuilder`1).</span></span> <span data-ttu-id="2f06f-477">[OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1) fornisce overload di [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) che consentono di usare fino a cinque servizi per configurare le opzioni:</span><span class="sxs-lookup"><span data-stu-id="2f06f-477">[OptionsBuilder\<TOptions>](xref:Microsoft.Extensions.Options.OptionsBuilder`1) provides overloads of [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) that allow you to use up to five services to configure options:</span></span>

  ```csharp
  services.AddOptions<MyOptions>("optionalName")
      .Configure<Service1, Service2, Service3, Service4, Service5>(
          (o, s, s2, s3, s4, s5) => 
              o.Property = DoSomethingWith(s, s2, s3, s4, s5));
  ```

* <span data-ttu-id="2f06f-478">Creare un tipo personalizzato che implementa <xref:Microsoft.Extensions.Options.IConfigureOptions%601> o <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e registrare il tipo come servizio.</span><span class="sxs-lookup"><span data-stu-id="2f06f-478">Create your own type that implements <xref:Microsoft.Extensions.Options.IConfigureOptions%601> or <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> and register the type as a service.</span></span>

<span data-ttu-id="2f06f-479">È consigliabile passare un delegato di configurazione a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) perché la creazione di un servizio è più complessa.</span><span class="sxs-lookup"><span data-stu-id="2f06f-479">We recommend passing a configuration delegate to [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*), since creating a service is more complex.</span></span> <span data-ttu-id="2f06f-480">La creazione di un tipo personalizzato equivale alle operazioni che il framework esegue automaticamente quando si usa [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*).</span><span class="sxs-lookup"><span data-stu-id="2f06f-480">Creating your own type is equivalent to what the framework does for you when you use [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*).</span></span> <span data-ttu-id="2f06f-481">La chiamata a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registra un'interfaccia <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> generica temporanea, che ha un costruttore che accetta i tipi di servizi generici specificati.</span><span class="sxs-lookup"><span data-stu-id="2f06f-481">Calling [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registers a transient generic <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601>, which has a constructor that accepts the generic service types specified.</span></span> 

## <a name="options-post-configuration"></a><span data-ttu-id="2f06f-482">Post-configurazione delle opzioni</span><span class="sxs-lookup"><span data-stu-id="2f06f-482">Options post-configuration</span></span>

<span data-ttu-id="2f06f-483">Impostare la post-configurazione con <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>.</span><span class="sxs-lookup"><span data-stu-id="2f06f-483">Set post-configuration with <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>.</span></span> <span data-ttu-id="2f06f-484">La post-configurazione viene eseguita dopo il completamento della configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>:</span><span class="sxs-lookup"><span data-stu-id="2f06f-484">Post-configuration runs after all <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configuration occurs:</span></span>

```csharp
services.PostConfigure<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<span data-ttu-id="2f06f-485"><xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> è disponibile per la post-configurazione delle opzioni denominate:</span><span class="sxs-lookup"><span data-stu-id="2f06f-485"><xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> is available to post-configure named options:</span></span>

```csharp
services.PostConfigure<MyOptions>("named_options_1", myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<span data-ttu-id="2f06f-486">Usare <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> per la post-configurazione di tutte le istanze di configurazione:</span><span class="sxs-lookup"><span data-stu-id="2f06f-486">Use <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> to post-configure all configuration instances:</span></span>

```csharp
services.PostConfigureAll<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

## <a name="accessing-options-during-startup"></a><span data-ttu-id="2f06f-487">Accesso alle opzioni durante l'avvio</span><span class="sxs-lookup"><span data-stu-id="2f06f-487">Accessing options during startup</span></span>

<span data-ttu-id="2f06f-488"><xref:Microsoft.Extensions.Options.IOptions%601> e <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> possono essere usate in `Startup.Configure`, perché i servizi vengono compilati prima dell'esecuzione del metodo `Configure`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-488"><xref:Microsoft.Extensions.Options.IOptions%601> and <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> can be used in `Startup.Configure`, since services are built before the `Configure` method executes.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IOptionsMonitor<MyOptions> optionsAccessor)
{
    var option1 = optionsAccessor.CurrentValue.Option1;
}
```

<span data-ttu-id="2f06f-489">Non usare <xref:Microsoft.Extensions.Options.IOptions%601> oppure <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="2f06f-489">Don't use <xref:Microsoft.Extensions.Options.IOptions%601> or <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="2f06f-490">Le opzioni potrebbero avere uno stato incoerente a causa dell'ordinamento delle registrazioni dei servizi.</span><span class="sxs-lookup"><span data-stu-id="2f06f-490">An inconsistent options state may exist due to the ordering of service registrations.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="2f06f-491">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="2f06f-491">Additional resources</span></span>

* <xref:fundamentals/configuration/index>
