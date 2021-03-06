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
ms.openlocfilehash: 86484fe2e2ce8e20e397a78f118d8b1bbe0cc88a
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395110"
---
# <a name="aspnet-core-blazor-environments"></a><span data-ttu-id="8049a-103">Ambienti di ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="8049a-103">ASP.NET Core Blazor environments</span></span>

> [!NOTE]
> <span data-ttu-id="8049a-104">Questo argomento si applica a Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="8049a-104">This topic applies to Blazor WebAssembly.</span></span> <span data-ttu-id="8049a-105">Per indicazioni generali sulla configurazione dell'app ASP.NET Core, che descrive gli approcci da usare per le Blazor Server app, vedere <xref:fundamentals/environments> .</span><span class="sxs-lookup"><span data-stu-id="8049a-105">For general guidance on ASP.NET Core app configuration, which describes the approaches to use for Blazor Server apps, see <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="8049a-106">Quando si esegue un'app localmente, per impostazione predefinita viene impostato lo sviluppo per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="8049a-106">When running an app locally, the environment defaults to Development.</span></span> <span data-ttu-id="8049a-107">Quando l'app viene pubblicata, per impostazione predefinita viene impostato l'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="8049a-107">When the app is published, the environment defaults to Production.</span></span>

<span data-ttu-id="8049a-108">L'app sul lato client Blazor ( **`Client`** ) di una soluzione ospitata Blazor WebAssembly determina l'ambiente dall' **`Server`** app della soluzione tramite un middleware che comunica l'ambiente al browser.</span><span class="sxs-lookup"><span data-stu-id="8049a-108">The client-side Blazor app (**`Client`**) of a hosted Blazor WebAssembly solution determines the environment from the **`Server`** app of the solution via a middleware that communicates the environment to the browser.</span></span> <span data-ttu-id="8049a-109">L' **`Server`** app aggiunge un'intestazione denominata `blazor-environment` con l'ambiente come valore dell'intestazione.</span><span class="sxs-lookup"><span data-stu-id="8049a-109">The **`Server`** app adds a header named `blazor-environment` with the environment as the value of the header.</span></span> <span data-ttu-id="8049a-110">L' **`Client`** intestazione viene letta dall'app.</span><span class="sxs-lookup"><span data-stu-id="8049a-110">The **`Client`** app reads the header.</span></span> <span data-ttu-id="8049a-111">L' **`Server`** app della soluzione è un'app ASP.NET Core, quindi ulteriori informazioni su come configurare l'ambiente si trovano in <xref:fundamentals/environments> .</span><span class="sxs-lookup"><span data-stu-id="8049a-111">The **`Server`** app of the solution is an ASP.NET Core app, so more information on how to configure the environment is found in <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="8049a-112">Per un'app autonoma Blazor WebAssembly in esecuzione in locale, il server di sviluppo aggiunge l' `blazor-environment` intestazione per specificare l'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="8049a-112">For a standalone Blazor WebAssembly app running locally, the development server adds the `blazor-environment` header to specify the Development environment.</span></span> <span data-ttu-id="8049a-113">Per specificare l'ambiente per altri ambienti host, aggiungere l' `blazor-environment` intestazione.</span><span class="sxs-lookup"><span data-stu-id="8049a-113">To specify the environment for other hosting environments, add the `blazor-environment` header.</span></span>

<span data-ttu-id="8049a-114">Nell'esempio seguente per IIS, l'intestazione personalizzata ( `blazor-environment` ) viene aggiunta al `web.config` file pubblicato.</span><span class="sxs-lookup"><span data-stu-id="8049a-114">In the following example for IIS, the custom header (`blazor-environment`) is added to the published `web.config` file.</span></span> <span data-ttu-id="8049a-115">Il `web.config` file si trova nella `bin/Release/{TARGET FRAMEWORK}/publish` cartella, dove il segnaposto `{TARGET FRAMEWORK}` è il Framework di destinazione:</span><span class="sxs-lookup"><span data-stu-id="8049a-115">The `web.config` file is located in the `bin/Release/{TARGET FRAMEWORK}/publish` folder, where the placeholder `{TARGET FRAMEWORK}` is the target framework:</span></span>

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
> <span data-ttu-id="8049a-116">Per usare un `web.config` file personalizzato per IIS che non viene sovrascritto quando l'app viene pubblicata nella `publish` cartella, vedere <xref:blazor/host-and-deploy/webassembly#use-a-custom-webconfig> .</span><span class="sxs-lookup"><span data-stu-id="8049a-116">To use a custom `web.config` file for IIS that isn't overwritten when the app is published to the `publish` folder, see <xref:blazor/host-and-deploy/webassembly#use-a-custom-webconfig>.</span></span>

<span data-ttu-id="8049a-117">Ottenere l'ambiente dell'app in un componente inserendo <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> e leggendo la <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.Environment> Proprietà.</span><span class="sxs-lookup"><span data-stu-id="8049a-117">Obtain the app's environment in a component by injecting <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> and reading the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.Environment> property.</span></span>

<span data-ttu-id="8049a-118">`Pages/ReadEnvironment.razor`:</span><span class="sxs-lookup"><span data-stu-id="8049a-118">`Pages/ReadEnvironment.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/environments/ReadEnvironment.razor?highlight=3,7)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/environments/ReadEnvironment.razor?highlight=3,7)]

::: moniker-end

<span data-ttu-id="8049a-119">Durante l'avvio, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder> espone l'oggetto <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> tramite la <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.HostEnvironment> proprietà, che consente la logica specifica dell'ambiente nel codice del generatore host.</span><span class="sxs-lookup"><span data-stu-id="8049a-119">During startup, the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder> exposes the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> through the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.HostEnvironment> property, which enables environment-specific logic in host builder code.</span></span>

<span data-ttu-id="8049a-120">In `Program.Main` di `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="8049a-120">In `Program.Main` of `Program.cs`:</span></span>

```csharp
if (builder.HostEnvironment.Environment == "Custom")
{
    ...
};
```

<span data-ttu-id="8049a-121">I metodi di estensione pratici seguenti forniti tramite <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions> consentono di controllare l'ambiente corrente per i nomi di sviluppo, produzione, gestione temporanea e ambiente personalizzato:</span><span class="sxs-lookup"><span data-stu-id="8049a-121">The following convenience extension methods provided through <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions> permit checking the current environment for Development, Production, Staging, and custom environment names:</span></span>

* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsDevelopment%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsProduction%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsStaging%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsEnvironment%2A>

<span data-ttu-id="8049a-122">In `Program.Main` di `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="8049a-122">In `Program.Main` of `Program.cs`:</span></span>

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

<span data-ttu-id="8049a-123">La <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> proprietà può essere utilizzata durante l'avvio quando il <xref:Microsoft.AspNetCore.Components.NavigationManager> servizio non è disponibile.</span><span class="sxs-lookup"><span data-stu-id="8049a-123">The <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> property can be used during startup when the <xref:Microsoft.AspNetCore.Components.NavigationManager> service isn't available.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="8049a-124">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="8049a-124">Additional resources</span></span>

* <xref:fundamentals/environments>
