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
# <a name="localization-extensibility"></a>Estendibilità della localizzazione

Di [Hisham Bin Ateya](https://github.com/hishamco)

Questo articolo:

* Elenca i punti di estendibilità nelle API di localizzazione.
* Include le istruzioni per estendere la localizzazione delle app ASP.NET Core.

## <a name="extensible-points-in-localization-apis"></a>Punti estendibili nelle API di localizzazione

Le API di localizzazione di ASP.NET Core sono estendibili. L'estendibilità consente agli sviluppatori di personalizzare la localizzazione in base alle esigenze. [OrchardCore](https://github.com/orchardCMS/OrchardCore/), ad esempio, ha `POStringLocalizer`. `POStringLocalizer` descrive in dettaglio l'uso della [localizzazione degli oggetti portabili](xref:fundamentals/portable-object-localization) per usare file `PO` per archiviare le risorse di localizzazione.

Questo articolo elenca i due punti di estendibilità principali forniti dalle API di localizzazione: 

* <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider>
* <xref:Microsoft.Extensions.Localization.IStringLocalizer>

## <a name="localization-culture-providers"></a>Provider di impostazioni cultura per la localizzazione

Le API di localizzazione di ASP.NET Core hanno quattro provider predefiniti che possono determinare le impostazioni cultura correnti di una richiesta in esecuzione:

* <xref:Microsoft.AspNetCore.Localization.QueryStringRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CookieRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.AcceptLanguageHeaderRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider>

I provider precedenti sono descritti in dettaglio nella documentazione del [middleware di localizzazione](xref:fundamentals/localization). Se i provider predefiniti non rispondono alle proprie esigenze, compilare un provider personalizzato usando uno degli approcci seguenti:

### <a name="use-customrequestcultureprovider"></a>Usare CustomRequestCultureProvider

<xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider> fornisce una classe <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> personalizzata che usa un semplice delegato per determinare le impostazioni cultura correnti per la localizzazione:

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

### <a name="use-a-new-implemetation-of-requestcultureprovider"></a>Usare una nuova implementazione di RequestCultureProvider

Si può creare una nuova implementazione di <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> che determina le informazioni sulle impostazioni cultura della richiesta da un'origine personalizzata. L'origine personalizzata, ad esempio, può essere un file o un database di configurazione.

Nell'esempio seguente viene illustrato `AppSettingsRequestCultureProvider` , che estende <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> per determinare le informazioni sulle impostazioni cultura della richiesta da *appsettings.json* :

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

## <a name="localization-resources"></a>Risorse di localizzazione

La localizzazione di ASP.NET Core fornisce <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer>. <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer> è un'implementazione di <xref:Microsoft.Extensions.Localization.IStringLocalizer> che usa `resx` per archiviare le risorse di localizzazione.

Non è obbligatorio usare file `resx`. Implementando `IStringLocalizer`, si può usare qualsiasi origine dati.

I progetti di esempio seguenti implementano <xref:Microsoft.Extensions.Localization.IStringLocalizer>: 

* [EFStringLocalizer](https://github.com/aspnet/Entropy/tree/master/samples/Localization.EntityFramework)
* [JsonStringLocalizer](https://github.com/hishamco/My.Extensions.Localization.Json)
* [SqlLocalizer](https://github.com/damienbod/AspNetCoreLocalization)
