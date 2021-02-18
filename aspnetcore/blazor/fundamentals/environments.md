---
title: Ambienti di ASP.NET Core Blazor
author: guardrex
description: Informazioni sugli ambienti in Blazor , incluse informazioni su come impostare l'ambiente di un' Blazor WebAssembly app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/11/2020
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
uid: blazor/fundamentals/environments
ms.openlocfilehash: a5ead59e467da331b585e8daefb1d7d259c7edba
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101048"
---
# <a name="aspnet-core-blazor-environments"></a>Ambienti di ASP.NET Core Blazor

> [!NOTE]
> Questo argomento si applica a Blazor WebAssembly . Per indicazioni generali sulla configurazione dell'app ASP.NET Core, che descrive gli approcci da usare per le Blazor Server app, vedere <xref:fundamentals/environments> .

Quando si esegue un'app localmente, per impostazione predefinita viene impostato lo sviluppo per l'ambiente. Quando l'app viene pubblicata, per impostazione predefinita viene impostato l'ambiente di produzione.

L'app sul lato client Blazor ( *`Client`* ) di una soluzione ospitata Blazor WebAssembly determina l'ambiente dall' *`Server`* app della soluzione tramite un middleware che comunica l'ambiente al browser. L' *`Server`* app aggiunge un'intestazione denominata `blazor-environment` con l'ambiente come valore dell'intestazione. L' *`Client`* intestazione viene letta dall'app. L' *`Server`* app della soluzione è un'app ASP.NET Core, quindi ulteriori informazioni su come configurare l'ambiente si trovano in <xref:fundamentals/environments> .

Per un'app autonoma Blazor WebAssembly in esecuzione in locale, il server di sviluppo aggiunge l' `blazor-environment` intestazione per specificare l'ambiente di sviluppo. Per specificare l'ambiente per altri ambienti host, aggiungere l' `blazor-environment` intestazione.

Nell'esempio seguente per IIS, l'intestazione personalizzata ( `blazor-environment` ) viene aggiunta al `web.config` file pubblicato. Il `web.config` file si trova nella `bin/Release/{TARGET FRAMEWORK}/publish` cartella, dove il segnaposto `{TARGET FRAMEWORK}` è il Framework di destinazione:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>

    ...

    <httpProtocol>
      <customHeaders>
        <add name="blazor-environment" value="Staging" />
      </customHeaders>
    </httpProtocol>
  </system.webServer>
</configuration>
```

> [!NOTE]
> Per usare un `web.config` file personalizzato per IIS che non viene sovrascritto quando l'app viene pubblicata nella `publish` cartella, vedere <xref:blazor/host-and-deploy/webassembly#use-a-custom-webconfig> .

Ottenere l'ambiente dell'app in un componente inserendo <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> e leggendo la <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.Environment> Proprietà.

`Pages/ReadEnvironment.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/environments/ReadEnvironment.razor?highlight=3,7)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/environments/ReadEnvironment.razor?highlight=3,7)]

::: moniker-end

Durante l'avvio, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder> espone l'oggetto <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> tramite la <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.HostEnvironment> proprietà, che consente la logica specifica dell'ambiente nel codice del generatore host.

In `Program.Main` di `Program.cs` :

```csharp
if (builder.HostEnvironment.Environment == "Custom")
{
    ...
};
```

I metodi di estensione pratici seguenti forniti tramite <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions> consentono di controllare l'ambiente corrente per i nomi di sviluppo, produzione, gestione temporanea e ambiente personalizzato:

* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsDevelopment%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsProduction%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsStaging%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsEnvironment%2A>

In `Program.Main` di `Program.cs` :

```csharp
if (builder.HostEnvironment.IsStaging())
{
    ...
};

if (builder.HostEnvironment.IsEnvironment("Custom"))
{
    ...
};
```

La <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> proprietà può essere utilizzata durante l'avvio quando il <xref:Microsoft.AspNetCore.Components.NavigationManager> servizio non è disponibile.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/environments>
