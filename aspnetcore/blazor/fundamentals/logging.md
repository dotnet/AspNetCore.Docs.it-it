---
title: BlazorRegistrazione ASP.NET Core
author: guardrex
description: Informazioni sulla registrazione nelle Blazor app, inclusa la configurazione a livello di log e su come scrivere messaggi di log dai Razor componenti.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/16/2020
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
uid: blazor/fundamentals/logging
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: e87b1a0dd54eb03dc1bdfdc7f2189a59df272026
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101028"
---
# <a name="aspnet-core-blazor-logging"></a>BlazorRegistrazione ASP.NET Core

::: zone pivot="webassembly"

Configurare la registrazione personalizzata nelle Blazor WebAssembly app con la <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.Logging?displayProperty=nameWithType> Proprietà.

Aggiungere lo spazio dei nomi per <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting?displayProperty=fullName> a `Program.cs` :

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;
```

In `Program.Main` di `Program.cs` impostare il livello di registrazione minimo con <xref:Microsoft.Extensions.Logging.LoggingBuilderExtensions.SetMinimumLevel%2A?displayProperty=nameWithType> e aggiungere il provider di registrazione personalizzato:

```csharp
var builder = WebAssemblyHostBuilder.CreateDefault(args);
...
builder.Logging.SetMinimumLevel(LogLevel.Debug);
builder.Logging.AddProvider(new CustomLoggingProvider());
```

La `Logging` proprietà è di tipo <xref:Microsoft.Extensions.Logging.ILoggingBuilder> , pertanto tutti i metodi di estensione disponibili in <xref:Microsoft.Extensions.Logging.ILoggingBuilder> sono disponibili anche in `Logging` .

La configurazione della registrazione può essere caricata da file di impostazioni dell'app. Per altre informazioni, vedere <xref:blazor/fundamentals/configuration#logging-configuration>.

## <a name="signalr-net-client-logging"></a>SignalR Registrazione client .NET

Inserire un oggetto <xref:Microsoft.Extensions.Logging.ILoggerProvider> per aggiungere un oggetto `WebAssemblyConsoleLogger` ai provider di registrazione passati a <xref:Microsoft.AspNetCore.SignalR.Client.HubConnectionBuilder> . Diversamente da un tradizionale <xref:Microsoft.Extensions.Logging.Console.ConsoleLogger> , `WebAssemblyConsoleLogger` è un wrapper per le API di registrazione specifiche del browser (ad esempio, `console.log` ). L'uso di `WebAssemblyConsoleLogger` rende possibile la registrazione in mono all'interno di un contesto del browser.

> [!NOTE]
> `WebAssemblyConsoleLogger` è [interno](/dotnet/csharp/language-reference/keywords/internal) e non è disponibile per l'uso diretto nel codice dello sviluppatore.

Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.Logging?displayProperty=fullName> e inserire un <xref:Microsoft.Extensions.Logging.ILoggerProvider> elemento nel componente:

```csharp
@using Microsoft.Extensions.Logging
@inject ILoggerProvider LoggerProvider
```

Nel [ `OnInitializedAsync` Metodo](xref:blazor/components/lifecycle#component-initialization-methods)del componente usare <xref:Microsoft.AspNetCore.SignalR.Client.HubConnectionBuilderExtensions.ConfigureLogging%2A?displayProperty=nameWithType> :

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl(NavigationManager.ToAbsoluteUri("/chathub"))
    .ConfigureLogging(logging => logging.AddProvider(LoggerProvider))
    .Build();
```

::: zone-end

::: zone pivot="server"

Per informazioni generali sulla registrazione di ASP.NET Core relative a Blazor Server , vedere <xref:fundamentals/logging/index> .

::: zone-end

## <a name="log-in-razor-components"></a>Componenti di accesso Razor

I logger rispettano la configurazione di avvio dell'app.

La `using` direttiva per <xref:Microsoft.Extensions.Logging> è necessaria per supportare i completamenti [IntelliSense](/visualstudio/ide/using-intellisense) per le API, ad esempio <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogWarning%2A> e <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogError%2A> .

Nell'esempio seguente viene illustrata la registrazione con un <xref:Microsoft.Extensions.Logging.ILogger> in Components.

`Pages/Counter.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/logging/Counter1.razor?highlight=3,16)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/logging/Counter1.razor?highlight=3,16)]

::: moniker-end

Nell'esempio seguente viene illustrata la registrazione con un <xref:Microsoft.Extensions.Logging.ILoggerFactory> in Components.

`Pages/Counter.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/logging/Counter2.razor?highlight=3,16-17)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/logging/Counter2.razor?highlight=3,16-17)]

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/logging/index>
