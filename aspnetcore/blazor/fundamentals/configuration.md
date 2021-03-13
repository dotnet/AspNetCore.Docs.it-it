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
ms.openlocfilehash: fd2a9784e92b132d679b2065b37cb37b7dd89cde
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413405"
---
# <a name="aspnet-core-blazor-configuration"></a>Configurazione di ASP.NET Core Blazor

> [!NOTE]
> Questo argomento si applica a Blazor WebAssembly . Per indicazioni generali sulla configurazione dell'app ASP.NET Core, vedere <xref:fundamentals/configuration/index> .

Blazor WebAssembly carica la configurazione dai file di impostazioni dell'app seguenti per impostazione predefinita:

* `wwwroot/appsettings.json`.
* `wwwroot/appsettings.{ENVIRONMENT}.json`, dove il `{ENVIRONMENT}` segnaposto è l' [ambiente di runtime](xref:fundamentals/environments)dell'app.

Altri provider di configurazione registrati dall'app possono anche fornire la configurazione, ma non tutti i provider o le funzionalità del provider sono appropriate per le Blazor WebAssembly app:

* [Provider di configurazione Azure Key Vault](xref:security/key-vault-configuration): il provider non è supportato per l'identità gestita e l'ID applicazione (ID client) con scenari di segreto client. L'ID applicazione con un segreto client non è consigliato per nessuna app ASP.NET Core, in particolare per le Blazor WebAssembly app perché il segreto client non può essere protetto sul lato client per accedere al servizio Azure Key Vault.
* [Provider di configurazione app Azure](/azure/azure-app-configuration/quickstart-aspnet-core-app): il provider non è appropriato per le Blazor WebAssembly app perché le Blazor WebAssembly app non vengono eseguite in un server in Azure.

> [!WARNING]
> La configurazione in un' Blazor WebAssembly app è visibile agli utenti. **Non archiviare i segreti delle app, le credenziali o altri dati sensibili nella configurazione di un' Blazor WebAssembly app.**

Per ulteriori informazioni sui provider di configurazione, vedere <xref:fundamentals/configuration/index> .

## <a name="app-settings-configuration"></a>Configurazione impostazioni app

La configurazione nei file di impostazioni dell'app viene caricata per impostazione predefinita. Nell'esempio seguente un valore di configurazione dell'interfaccia utente viene archiviato in un file di impostazioni dell'app e caricato Blazor automaticamente dal Framework. Il valore viene letto da un componente.

`wwwroot/appsettings.json`:

```json
{
  "h1FontSize": "50px"
}
```

Inserire un' <xref:Microsoft.Extensions.Configuration.IConfiguration> istanza in un componente per accedere ai dati di configurazione.

`Pages/ConfigurationExample.razor`:

```razor
@page "/configuration-example"
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<h1 style="font-size:@Configuration["h1FontSize"]">
    Configuration example
</h1>
```

Per leggere altri file di configurazione dalla `wwwroot` cartella alla configurazione, usare un <xref:System.Net.Http.HttpClient> per ottenere il contenuto del file. Nell'esempio seguente viene letto un file di configurazione ( `cars.json` ) nella configurazione dell'app.

`wwwroot/cars.json`:

```json
{
    "size": "tiny"
}
```

Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> a `Program.cs` :

```csharp
using Microsoft.Extensions.Configuration;
```

In `Program.Main` di `Program.cs` modificare la registrazione del <xref:System.Net.Http.HttpClient> servizio esistente per utilizzare il client di per leggere il file:

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

## <a name="memory-configuration-source"></a>Origine configurazione memoria

Nell'esempio seguente viene usato un <xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationSource> in `Program.Main` per fornire la configurazione aggiuntiva.

Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.Configuration.Memory?displayProperty=fullName> a `Program.cs` :

```csharp
using Microsoft.Extensions.Configuration.Memory;
```

In `Program.Main` di `Program.cs` :

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

Inserire un' <xref:Microsoft.Extensions.Configuration.IConfiguration> istanza in un componente per accedere ai dati di configurazione.

`Pages/MemoryConfig.razor`:

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

Ottenere una sezione della configurazione nel codice C# con <xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection%2A?displayProperty=nameWithType> . Nell'esempio seguente viene ottenuta la `wheels` sezione per la configurazione nell'esempio precedente:

```razor
@code {
    protected override void OnInitialized()
    {
        var wheelsSection = Configuration.GetSection("wheels");

        ...
    }
}
```

## <a name="authentication-configuration"></a>Configurazione dell'autenticazione

Fornire la configurazione di autenticazione in un file di impostazioni dell'app.

`wwwroot/appsettings.json`:

```json
{
  "Local": {
    "Authority": "{AUTHORITY}",
    "ClientId": "{CLIENT ID}"
  }
}
```

Caricare la configurazione per un Identity provider con <xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind%2A?displayProperty=nameWithType> in `Program.Main` . Nell'esempio seguente viene caricata la configurazione per un [provider OIDC](xref:blazor/security/webassembly/standalone-with-authentication-library).

`Program.cs`:

```csharp
builder.Services.AddOidcAuthentication(options =>
    builder.Configuration.Bind("Local", options.ProviderOptions));
```

## <a name="logging-configuration"></a>Configurazione della registrazione

Aggiungere un riferimento al pacchetto per [`Microsoft.Extensions.Logging.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Configuration) al file di progetto dell'app:

```xml
<PackageReference Include="Microsoft.Extensions.Logging.Configuration" Version="{VERION}" />
```

Nell'esempio precedente, il `{VERSION}` segnaposto è la versione del pacchetto. Le versioni del pacchetto si trovano in [NuGet.org](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Configuration).

Nel file di impostazioni dell'app fornire la configurazione di registrazione. La configurazione di registrazione viene caricata in `Program.Main` .

`wwwroot/appsettings.json`:

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

Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.Logging?displayProperty=fullName> a `Program.cs` :

```csharp
using Microsoft.Extensions.Logging;
```

In `Program.Main` di `Program.cs` :

```csharp
builder.Logging.AddConfiguration(
    builder.Configuration.GetSection("Logging"));
```

## <a name="host-builder-configuration"></a>Configurazione del generatore host

Leggere la configurazione del generatore host da <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.Configuration?displayProperty=nameWithType> in `Program.Main` .

In `Program.Main` di `Program.cs` :

```csharp
var hostname = builder.Configuration["HostName"];
```

## <a name="cached-configuration"></a>Configurazione memorizzata nella cache

I file di configurazione vengono memorizzati nella cache per l'uso offline. Con [le applicazioni Web progressive (PWA)](xref:blazor/progressive-web-app)è possibile aggiornare solo i file di configurazione quando si crea una nuova distribuzione. La modifica dei file di configurazione tra le distribuzioni non ha effetto perché:

* Gli utenti dispongono di versioni memorizzate nella cache dei file che continuano a usare.
* I `service-worker.js` file e di PWA `service-worker-assets.js` devono essere ricompilati durante la compilazione, che segnalano all'app la prossima visita online dell'utente che l'app è stata ridistribuita.

Per ulteriori informazioni sul modo in cui gli aggiornamenti in background vengono gestiti da PWA, vedere <xref:blazor/progressive-web-app#background-updates> .
