---
title: Configurazione di ASP.NET Core Blazor
author: guardrex
description: Informazioni sulla configurazione delle Blazor app, incluse le impostazioni dell'app, l'autenticazione e la configurazione della registrazione.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/10/2020
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
uid: blazor/fundamentals/configuration
ms.openlocfilehash: 48d78f40e9254bac182ffbc534550157664bcc5b
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100106934"
---
# <a name="aspnet-core-blazor-configuration"></a><span data-ttu-id="bf277-103">Configurazione di ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="bf277-103">ASP.NET Core Blazor configuration</span></span>

> [!NOTE]
> <span data-ttu-id="bf277-104">Questo argomento si applica a Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="bf277-104">This topic applies to Blazor WebAssembly.</span></span> <span data-ttu-id="bf277-105">Per indicazioni generali sulla configurazione dell'app ASP.NET Core, vedere <xref:fundamentals/configuration/index> .</span><span class="sxs-lookup"><span data-stu-id="bf277-105">For general guidance on ASP.NET Core app configuration, see <xref:fundamentals/configuration/index>.</span></span>

<span data-ttu-id="bf277-106">Blazor WebAssembly carica la configurazione dai file di impostazioni dell'app seguenti per impostazione predefinita:</span><span class="sxs-lookup"><span data-stu-id="bf277-106">Blazor WebAssembly loads configuration from the following app settings files by default:</span></span>

* <span data-ttu-id="bf277-107">`wwwroot/appsettings.json`.</span><span class="sxs-lookup"><span data-stu-id="bf277-107">`wwwroot/appsettings.json`.</span></span>
* <span data-ttu-id="bf277-108">`wwwroot/appsettings.{ENVIRONMENT}.json`, dove il `{ENVIRONMENT}` segnaposto è l' [ambiente di runtime](xref:fundamentals/environments)dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf277-108">`wwwroot/appsettings.{ENVIRONMENT}.json`, where the `{ENVIRONMENT}` placeholder is the app's [runtime environment](xref:fundamentals/environments).</span></span>

<span data-ttu-id="bf277-109">Altri provider di configurazione registrati dall'app possono anche fornire la configurazione, ma non tutti i provider o le funzionalità del provider sono appropriate per le Blazor WebAssembly app:</span><span class="sxs-lookup"><span data-stu-id="bf277-109">Other configuration providers registered by the app can also provide configuration, but not all providers or provider features are appropriate for Blazor WebAssembly apps:</span></span>

* <span data-ttu-id="bf277-110">[Provider di configurazione Azure Key Vault](xref:security/key-vault-configuration): il provider non è supportato per l'identità gestita e l'ID applicazione (ID client) con scenari di segreto client.</span><span class="sxs-lookup"><span data-stu-id="bf277-110">[Azure Key Vault configuration provider](xref:security/key-vault-configuration): The provider isn't supported for managed identity and application ID (client ID) with client secret scenarios.</span></span> <span data-ttu-id="bf277-111">L'ID applicazione con un segreto client non è consigliato per nessuna app ASP.NET Core, in particolare per le Blazor WebAssembly app perché il segreto client non può essere protetto sul lato client per accedere al servizio Azure Key Vault.</span><span class="sxs-lookup"><span data-stu-id="bf277-111">Application ID with a client secret isn't recommended for any ASP.NET Core app, especially Blazor WebAssembly apps because the client secret can't be secured client-side to access the Azure Key Vault service.</span></span>
* <span data-ttu-id="bf277-112">[Provider di configurazione app Azure](/azure/azure-app-configuration/quickstart-aspnet-core-app): il provider non è appropriato per le Blazor WebAssembly app perché le Blazor WebAssembly app non vengono eseguite in un server in Azure.</span><span class="sxs-lookup"><span data-stu-id="bf277-112">[Azure App configuration provider](/azure/azure-app-configuration/quickstart-aspnet-core-app): The provider isn't appropriate for Blazor WebAssembly apps because Blazor WebAssembly apps don't run on a server in Azure.</span></span>

> [!WARNING]
> <span data-ttu-id="bf277-113">La configurazione in un' Blazor WebAssembly app è visibile agli utenti.</span><span class="sxs-lookup"><span data-stu-id="bf277-113">Configuration in a Blazor WebAssembly app is visible to users.</span></span> <span data-ttu-id="bf277-114">**Non archiviare i segreti delle app, le credenziali o altri dati sensibili nella configurazione di un' Blazor WebAssembly app.**</span><span class="sxs-lookup"><span data-stu-id="bf277-114">**Don't store app secrets, credentials, or any other sensitive data in the configuration of a Blazor WebAssembly app.**</span></span>

<span data-ttu-id="bf277-115">Per ulteriori informazioni sui provider di configurazione, vedere <xref:fundamentals/configuration/index> .</span><span class="sxs-lookup"><span data-stu-id="bf277-115">For more information on configuration providers, see <xref:fundamentals/configuration/index>.</span></span>

## <a name="app-settings-configuration"></a><span data-ttu-id="bf277-116">Configurazione impostazioni app</span><span class="sxs-lookup"><span data-stu-id="bf277-116">App settings configuration</span></span>

<span data-ttu-id="bf277-117">La configurazione nei file di impostazioni dell'app viene caricata per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="bf277-117">Configuration in app settings files are loaded by default.</span></span> <span data-ttu-id="bf277-118">Nell'esempio seguente un valore di configurazione dell'interfaccia utente viene archiviato in un file di impostazioni dell'app e caricato Blazor automaticamente dal Framework.</span><span class="sxs-lookup"><span data-stu-id="bf277-118">In the following example, a UI configuration value is stored in an app settings file and loaded by the Blazor framework automatically.</span></span> <span data-ttu-id="bf277-119">Il valore viene letto da un componente.</span><span class="sxs-lookup"><span data-stu-id="bf277-119">The value is read by a component.</span></span>

<span data-ttu-id="bf277-120">`wwwroot/appsettings.json`:</span><span class="sxs-lookup"><span data-stu-id="bf277-120">`wwwroot/appsettings.json`:</span></span>

```json
{
  "h1FontSize": "50px"
}
```

<span data-ttu-id="bf277-121">Inserire un' <xref:Microsoft.Extensions.Configuration.IConfiguration> istanza in un componente per accedere ai dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="bf277-121">Inject an <xref:Microsoft.Extensions.Configuration.IConfiguration> instance into a component to access the configuration data.</span></span>

<span data-ttu-id="bf277-122">`Pages/ConfigurationExample.razor`:</span><span class="sxs-lookup"><span data-stu-id="bf277-122">`Pages/ConfigurationExample.razor`:</span></span>

```razor
@page "/configuration-example"
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<h1 style="font-size:@Configuration["h1FontSize"]">
    Configuration example
</h1>
```

<span data-ttu-id="bf277-123">Per leggere altri file di configurazione dalla `wwwroot` cartella alla configurazione, usare un <xref:System.Net.Http.HttpClient> per ottenere il contenuto del file.</span><span class="sxs-lookup"><span data-stu-id="bf277-123">To read other configuration files from the `wwwroot` folder into configuration, use an <xref:System.Net.Http.HttpClient> to obtain the file's content.</span></span> <span data-ttu-id="bf277-124">Nell'esempio seguente viene letto un file di configurazione ( `cars.json` ) nella configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf277-124">The following example reads a configuration file (`cars.json`) into the app's configuration.</span></span>

<span data-ttu-id="bf277-125">`wwwroot/cars.json`:</span><span class="sxs-lookup"><span data-stu-id="bf277-125">`wwwroot/cars.json`:</span></span>

```json
{
    "size": "tiny"
}
```

<span data-ttu-id="bf277-126">Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> a `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="bf277-126">Add the namespace for <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> to `Program.cs`:</span></span>

```csharp
using Microsoft.Extensions.Configuration;
```

<span data-ttu-id="bf277-127">In `Program.Main` di `Program.cs` modificare la registrazione del <xref:System.Net.Http.HttpClient> servizio esistente per utilizzare il client di per leggere il file:</span><span class="sxs-lookup"><span data-stu-id="bf277-127">In `Program.Main` of `Program.cs`, modify the existing <xref:System.Net.Http.HttpClient> service registration to use the client to read the file:</span></span>

```csharp
var http = new HttpClient()
{
    BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
};

builder.Services.AddScoped(sp => http);

using var response = await http.GetAsync("cars.json");
using var stream = await response.Content.ReadAsStreamAsync();

builder.Configuration.AddJsonStream(stream);
```

## <a name="memory-configuration-source"></a><span data-ttu-id="bf277-128">Origine configurazione memoria</span><span class="sxs-lookup"><span data-stu-id="bf277-128">Memory Configuration Source</span></span>

<span data-ttu-id="bf277-129">Nell'esempio seguente viene usato un <xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationSource> in `Program.Main` per fornire la configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="bf277-129">The following example uses a <xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationSource> in `Program.Main` to supply additional configuration.</span></span>

<span data-ttu-id="bf277-130">Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.Configuration.Memory?displayProperty=fullName> a `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="bf277-130">Add the namespace for <xref:Microsoft.Extensions.Configuration.Memory?displayProperty=fullName> to `Program.cs`:</span></span>

```csharp
using Microsoft.Extensions.Configuration.Memory;
```

<span data-ttu-id="bf277-131">In `Program.Main` di `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="bf277-131">In `Program.Main` of `Program.cs`:</span></span>

```csharp
var vehicleData = new Dictionary<string, string>()
{
    { "color", "blue" },
    { "type", "car" },
    { "wheels:count", "3" },
    { "wheels:brand", "Blazin" },
    { "wheels:brand:type", "rally" },
    { "wheels:year", "2008" },
};

var memoryConfig = new MemoryConfigurationSource { InitialData = vehicleData };

builder.Configuration.Add(memoryConfig);
```

<span data-ttu-id="bf277-132">Inserire un' <xref:Microsoft.Extensions.Configuration.IConfiguration> istanza in un componente per accedere ai dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="bf277-132">Inject an <xref:Microsoft.Extensions.Configuration.IConfiguration> instance into a component to access the configuration data.</span></span>

<span data-ttu-id="bf277-133">`Pages/MemoryConfig.razor`:</span><span class="sxs-lookup"><span data-stu-id="bf277-133">`Pages/MemoryConfig.razor`:</span></span>

```razor
@page "/memory-config"
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<h1>Memory configuration example</h1>

<h2>General specifications</h2>

<ul>
    <li>Color: @Configuration["color"]</li>
    <li>Type: @Configuration["type"]</li>
</ul>

<h2>Wheels</h2>

<ul>
    <li>Count: @Configuration["wheels:count"]</li>
    <li>Brand: @Configuration["wheels:brand"]</li>
    <li>Type: @Configuration["wheels:brand:type"]</li>
    <li>Year: @Configuration["wheels:year"]</li>
</ul>
```

<span data-ttu-id="bf277-134">Ottenere una sezione della configurazione nel codice C# con <xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="bf277-134">Obtain a section of the configuration in C# code with <xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="bf277-135">Nell'esempio seguente viene ottenuta la `wheels` sezione per la configurazione nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="bf277-135">The following example obtains the `wheels` section for the configuration in the preceding example:</span></span>

```razor
@code {
    protected override void OnInitialized()
    {
        var wheelsSection = Configuration.GetSection("wheels");

        ...
    }
}
```

## <a name="authentication-configuration"></a><span data-ttu-id="bf277-136">Configurazione dell'autenticazione</span><span class="sxs-lookup"><span data-stu-id="bf277-136">Authentication configuration</span></span>

<span data-ttu-id="bf277-137">Fornire la configurazione di autenticazione in un file di impostazioni dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf277-137">Provide authentication configuration in an app settings file.</span></span>

<span data-ttu-id="bf277-138">`wwwroot/appsettings.json`:</span><span class="sxs-lookup"><span data-stu-id="bf277-138">`wwwroot/appsettings.json`:</span></span>

```json
{
  "Local": {
    "Authority": "{AUTHORITY}",
    "ClientId": "{CLIENT ID}"
  }
}
```

<span data-ttu-id="bf277-139">Caricare la configurazione per un Identity provider con <xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind%2A?displayProperty=nameWithType> in `Program.Main` .</span><span class="sxs-lookup"><span data-stu-id="bf277-139">Load the configuration for an Identity provider with <xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind%2A?displayProperty=nameWithType> in `Program.Main`.</span></span> <span data-ttu-id="bf277-140">Nell'esempio seguente viene caricata la configurazione per un [provider OIDC](xref:blazor/security/webassembly/standalone-with-authentication-library).</span><span class="sxs-lookup"><span data-stu-id="bf277-140">The following example loads configuration for an [OIDC provider](xref:blazor/security/webassembly/standalone-with-authentication-library).</span></span>

<span data-ttu-id="bf277-141">`Program.cs`:</span><span class="sxs-lookup"><span data-stu-id="bf277-141">`Program.cs`:</span></span>

```csharp
builder.Services.AddOidcAuthentication(options =>
    builder.Configuration.Bind("Local", options.ProviderOptions));
```

## <a name="logging-configuration"></a><span data-ttu-id="bf277-142">Configurazione della registrazione</span><span class="sxs-lookup"><span data-stu-id="bf277-142">Logging configuration</span></span>

<span data-ttu-id="bf277-143">Aggiungere un riferimento al pacchetto per [`Microsoft.Extensions.Logging.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Configuration) al file di progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf277-143">Add a package reference for [`Microsoft.Extensions.Logging.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Configuration) to the app's project file.</span></span> <span data-ttu-id="bf277-144">Nel file di impostazioni dell'app fornire la configurazione di registrazione.</span><span class="sxs-lookup"><span data-stu-id="bf277-144">In the app settings file, provide logging configuration.</span></span> <span data-ttu-id="bf277-145">La configurazione di registrazione viene caricata in `Program.Main` .</span><span class="sxs-lookup"><span data-stu-id="bf277-145">The logging configuration is loaded in `Program.Main`.</span></span>

<span data-ttu-id="bf277-146">`wwwroot/appsettings.json`:</span><span class="sxs-lookup"><span data-stu-id="bf277-146">`wwwroot/appsettings.json`:</span></span>

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}
```

<span data-ttu-id="bf277-147">Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.Logging?displayProperty=fullName> a `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="bf277-147">Add the namespace for <xref:Microsoft.Extensions.Logging?displayProperty=fullName> to `Program.cs`:</span></span>

```csharp
using Microsoft.Extensions.Logging;
```

<span data-ttu-id="bf277-148">In `Program.Main` di `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="bf277-148">In `Program.Main` of `Program.cs`:</span></span>

```csharp
builder.Logging.AddConfiguration(
    builder.Configuration.GetSection("Logging"));
```

## <a name="host-builder-configuration"></a><span data-ttu-id="bf277-149">Configurazione del generatore host</span><span class="sxs-lookup"><span data-stu-id="bf277-149">Host builder configuration</span></span>

<span data-ttu-id="bf277-150">Leggere la configurazione del generatore host da <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.Configuration?displayProperty=nameWithType> in `Program.Main` .</span><span class="sxs-lookup"><span data-stu-id="bf277-150">Read host builder configuration from <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.Configuration?displayProperty=nameWithType> in `Program.Main`.</span></span>

<span data-ttu-id="bf277-151">In `Program.Main` di `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="bf277-151">In `Program.Main` of `Program.cs`:</span></span>

```csharp
var hostname = builder.Configuration["HostName"];
```

## <a name="cached-configuration"></a><span data-ttu-id="bf277-152">Configurazione memorizzata nella cache</span><span class="sxs-lookup"><span data-stu-id="bf277-152">Cached configuration</span></span>

<span data-ttu-id="bf277-153">I file di configurazione vengono memorizzati nella cache per l'uso offline.</span><span class="sxs-lookup"><span data-stu-id="bf277-153">Configuration files are cached for offline use.</span></span> <span data-ttu-id="bf277-154">Con [le applicazioni Web progressive (PWA)](xref:blazor/progressive-web-app)è possibile aggiornare solo i file di configurazione quando si crea una nuova distribuzione.</span><span class="sxs-lookup"><span data-stu-id="bf277-154">With [Progressive Web Applications (PWAs)](xref:blazor/progressive-web-app), you can only update configuration files when creating a new deployment.</span></span> <span data-ttu-id="bf277-155">La modifica dei file di configurazione tra le distribuzioni non ha effetto perché:</span><span class="sxs-lookup"><span data-stu-id="bf277-155">Editing configuration files between deployments has no effect because:</span></span>

* <span data-ttu-id="bf277-156">Gli utenti dispongono di versioni memorizzate nella cache dei file che continuano a usare.</span><span class="sxs-lookup"><span data-stu-id="bf277-156">Users have cached versions of the files that they continue to use.</span></span>
* <span data-ttu-id="bf277-157">I `service-worker.js` file e di PWA `service-worker-assets.js` devono essere ricompilati durante la compilazione, che segnalano all'app la prossima visita online dell'utente che l'app è stata ridistribuita.</span><span class="sxs-lookup"><span data-stu-id="bf277-157">The PWA's `service-worker.js` and `service-worker-assets.js` files must be rebuilt on compilation, which signal to the app on the user's next online visit that the app has been redeployed.</span></span>

<span data-ttu-id="bf277-158">Per ulteriori informazioni sul modo in cui gli aggiornamenti in background vengono gestiti da PWA, vedere <xref:blazor/progressive-web-app#background-updates> .</span><span class="sxs-lookup"><span data-stu-id="bf277-158">For more information on how background updates are handled by PWAs, see <xref:blazor/progressive-web-app#background-updates>.</span></span>
