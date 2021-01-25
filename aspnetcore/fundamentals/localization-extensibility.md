---
title: Estendibilità della localizzazione
author: hishamco
description: Informazioni su come estendere le API di localizzazione nelle app ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/03/2019
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
uid: fundamentals/localization-extensibility
ms.openlocfilehash: c41ebc0506a565666963d9b4c70ed73df389c826
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751663"
---
# <a name="localization-extensibility"></a><span data-ttu-id="dc1bb-103">Estendibilità della localizzazione</span><span class="sxs-lookup"><span data-stu-id="dc1bb-103">Localization Extensibility</span></span>

<span data-ttu-id="dc1bb-104">Di [Hisham Bin Ateya](https://github.com/hishamco)</span><span class="sxs-lookup"><span data-stu-id="dc1bb-104">By [Hisham Bin Ateya](https://github.com/hishamco)</span></span>

<span data-ttu-id="dc1bb-105">Questo articolo:</span><span class="sxs-lookup"><span data-stu-id="dc1bb-105">This article:</span></span>

* <span data-ttu-id="dc1bb-106">Elenca i punti di estendibilità nelle API di localizzazione.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-106">Lists the extensibility points on the localization APIs.</span></span>
* <span data-ttu-id="dc1bb-107">Include le istruzioni per estendere la localizzazione delle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-107">Provides instructions on how to extend ASP.NET Core app localization.</span></span>

## <a name="extensible-points-in-localization-apis"></a><span data-ttu-id="dc1bb-108">Punti estendibili nelle API di localizzazione</span><span class="sxs-lookup"><span data-stu-id="dc1bb-108">Extensible Points in Localization APIs</span></span>

<span data-ttu-id="dc1bb-109">Le API di localizzazione di ASP.NET Core sono estendibili.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-109">ASP.NET Core localization APIs are built to be extensible.</span></span> <span data-ttu-id="dc1bb-110">L'estendibilità consente agli sviluppatori di personalizzare la localizzazione in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-110">Extensibility allows developers to customize the localization according to their needs.</span></span> <span data-ttu-id="dc1bb-111">[OrchardCore](https://github.com/orchardCMS/OrchardCore/), ad esempio, ha `POStringLocalizer`.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-111">For instance, [OrchardCore](https://github.com/orchardCMS/OrchardCore/) has a `POStringLocalizer`.</span></span> <span data-ttu-id="dc1bb-112">`POStringLocalizer` descrive in dettaglio l'uso della [localizzazione degli oggetti portabili](xref:fundamentals/portable-object-localization) per usare file `PO` per archiviare le risorse di localizzazione.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-112">`POStringLocalizer` describes in detail using [Portable Object localization](xref:fundamentals/portable-object-localization) to use `PO` files to store localization resources.</span></span>

<span data-ttu-id="dc1bb-113">Questo articolo elenca i due punti di estendibilità principali forniti dalle API di localizzazione:</span><span class="sxs-lookup"><span data-stu-id="dc1bb-113">This article lists the two main extensibility points that localization APIs provide:</span></span> 

* <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider>
* <xref:Microsoft.Extensions.Localization.IStringLocalizer>

## <a name="localization-culture-providers"></a><span data-ttu-id="dc1bb-114">Provider di impostazioni cultura per la localizzazione</span><span class="sxs-lookup"><span data-stu-id="dc1bb-114">Localization Culture Providers</span></span>

<span data-ttu-id="dc1bb-115">Le API di localizzazione di ASP.NET Core hanno quattro provider predefiniti che possono determinare le impostazioni cultura correnti di una richiesta in esecuzione:</span><span class="sxs-lookup"><span data-stu-id="dc1bb-115">ASP.NET Core localization APIs have four default providers that can determine the current culture of an executing request:</span></span>

* <xref:Microsoft.AspNetCore.Localization.QueryStringRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CookieRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.AcceptLanguageHeaderRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider>

<span data-ttu-id="dc1bb-116">I provider precedenti sono descritti in dettaglio nella documentazione del [middleware di localizzazione](xref:fundamentals/localization).</span><span class="sxs-lookup"><span data-stu-id="dc1bb-116">The preceding providers are described in detail in the [Localization Middleware](xref:fundamentals/localization) documentation.</span></span> <span data-ttu-id="dc1bb-117">Se i provider predefiniti non rispondono alle proprie esigenze, compilare un provider personalizzato usando uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="dc1bb-117">If the default providers don't meet your needs, build a custom provider using one of the following approaches:</span></span>

### <a name="use-customrequestcultureprovider"></a><span data-ttu-id="dc1bb-118">Usare CustomRequestCultureProvider</span><span class="sxs-lookup"><span data-stu-id="dc1bb-118">Use CustomRequestCultureProvider</span></span>

<span data-ttu-id="dc1bb-119"><xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider> fornisce una classe <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> personalizzata che usa un semplice delegato per determinare le impostazioni cultura correnti per la localizzazione:</span><span class="sxs-lookup"><span data-stu-id="dc1bb-119"><xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider> provides a custom <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> that uses a simple delegate to determine the current localization culture:</span></span>

::: moniker range="< aspnetcore-3.0"
```csharp
options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
{
    var currentCulture = "en";
    var segments = context.Request.Path.Value.Split(new char[] { '/' }, 
        StringSplitOptions.RemoveEmptyEntries);

    if (segments.Length > 1 && segments[0].Length == 2)
    {
        currentCulture = segments[0];
    }

    var requestCulture = new ProviderCultureResult(currentCulture);
    
    return Task.FromResult(requestCulture);
}));
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"
```csharp
options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
{
    var currentCulture = "en";
    var segments = context.Request.Path.Value.Split(new char[] { '/' }, 
        StringSplitOptions.RemoveEmptyEntries);

    if (segments.Length > 1 && segments[0].Length == 2)
    {
        currentCulture = segments[0];
    }

    var requestCulture = new ProviderCultureResult(currentCulture);
    
    return Task.FromResult(requestCulture);
}));
```

::: moniker-end

### <a name="use-a-new-implemetation-of-requestcultureprovider"></a><span data-ttu-id="dc1bb-120">Usare una nuova implementazione di RequestCultureProvider</span><span class="sxs-lookup"><span data-stu-id="dc1bb-120">Use a new implemetation of RequestCultureProvider</span></span>

<span data-ttu-id="dc1bb-121">Si può creare una nuova implementazione di <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> che determina le informazioni sulle impostazioni cultura della richiesta da un'origine personalizzata.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-121">A new implementation of <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> can be created that determines the request culture information from a custom source.</span></span> <span data-ttu-id="dc1bb-122">L'origine personalizzata, ad esempio, può essere un file o un database di configurazione.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-122">For example, the custom source can be a configuration file or database.</span></span>

<span data-ttu-id="dc1bb-123">Nell'esempio seguente viene illustrato `AppSettingsRequestCultureProvider` , che estende <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> per determinare le informazioni sulle impostazioni cultura della richiesta da *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="dc1bb-123">The following example shows `AppSettingsRequestCultureProvider`, which extends the <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> to determine the request culture information from *appsettings.json*:</span></span>

```csharp
public class AppSettingsRequestCultureProvider : RequestCultureProvider
{
    public string CultureKey { get; set; } = "culture";

    public string UICultureKey { get; set; } = "ui-culture";

    public override Task<ProviderCultureResult> DetermineProviderCultureResult(HttpContext httpContext)
    {
        if (httpContext == null)
        {
            throw new ArgumentNullException();
        }

        var configuration = httpContext.RequestServices.GetService<IConfigurationRoot>();
        var culture = configuration[CultureKey];
        var uiCulture = configuration[UICultureKey];

        if (culture == null && uiCulture == null)
        {
            return Task.FromResult((ProviderCultureResult)null);
        }

        if (culture != null && uiCulture == null)
        {
            uiCulture = culture;
        }

        if (culture == null && uiCulture != null)
        {
            culture = uiCulture;
        }
        
        var providerResultCulture = new ProviderCultureResult(culture, uiCulture);

        return Task.FromResult(providerResultCulture);
    }
}
```

## <a name="localization-resources"></a><span data-ttu-id="dc1bb-124">Risorse di localizzazione</span><span class="sxs-lookup"><span data-stu-id="dc1bb-124">Localization resources</span></span>

<span data-ttu-id="dc1bb-125">La localizzazione di ASP.NET Core fornisce <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer>.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-125">ASP.NET Core localization provides <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer>.</span></span> <span data-ttu-id="dc1bb-126"><xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer> è un'implementazione di <xref:Microsoft.Extensions.Localization.IStringLocalizer> che usa `resx` per archiviare le risorse di localizzazione.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-126"><xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer> is an implementation of <xref:Microsoft.Extensions.Localization.IStringLocalizer> that is uses `resx` to store localization resources.</span></span>

<span data-ttu-id="dc1bb-127">Non è obbligatorio usare file `resx`.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-127">You aren't limited to using `resx` files.</span></span> <span data-ttu-id="dc1bb-128">Implementando `IStringLocalizer`, si può usare qualsiasi origine dati.</span><span class="sxs-lookup"><span data-stu-id="dc1bb-128">By implementing `IStringLocalizer`, any data source can be used.</span></span>

<span data-ttu-id="dc1bb-129">I progetti di esempio seguenti implementano <xref:Microsoft.Extensions.Localization.IStringLocalizer>:</span><span class="sxs-lookup"><span data-stu-id="dc1bb-129">The following example projects implement <xref:Microsoft.Extensions.Localization.IStringLocalizer>:</span></span> 

* [<span data-ttu-id="dc1bb-130">EFStringLocalizer</span><span class="sxs-lookup"><span data-stu-id="dc1bb-130">EFStringLocalizer</span></span>](https://github.com/aspnet/Entropy/tree/master/samples/Localization.EntityFramework)
* [<span data-ttu-id="dc1bb-131">JsonStringLocalizer</span><span class="sxs-lookup"><span data-stu-id="dc1bb-131">JsonStringLocalizer</span></span>](https://github.com/hishamco/My.Extensions.Localization.Json)
* [<span data-ttu-id="dc1bb-132">SqlLocalizer</span><span class="sxs-lookup"><span data-stu-id="dc1bb-132">SqlLocalizer</span></span>](https://github.com/damienbod/AspNetCoreLocalization)
