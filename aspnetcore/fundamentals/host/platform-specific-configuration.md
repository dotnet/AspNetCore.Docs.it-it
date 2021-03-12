---
title: Usare assembly di avvio dell'hosting in ASP.NET Core
author: rick-anderson
description: Informazioni su come migliorare un'app ASP.NET Core da un assembly esterno con un'implementazione IHostingStartup.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 09/26/2019
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
uid: fundamentals/configuration/platform-specific-configuration
ms.openlocfilehash: b39215a70f990afeb7d3fe0a62981113b154354e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588243"
---
# <a name="use-hosting-startup-assemblies-in-aspnet-core"></a>Usare assembly di avvio dell'hosting in ASP.NET Core

Di [Pavel Krymets](https://github.com/pakrym)

::: moniker range=">= aspnetcore-3.0"

Un' <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> implementazione di (host Startup) aggiunge miglioramenti a un'app all'avvio da un assembly esterno. Una libreria esterna può ad esempio usare un'implementazione di avvio dell'hosting per offrire servizi o provider di configurazione aggiuntivi a un'app.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="hostingstartup-attribute"></a>Attributo HostingStartup

Un attributo [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) indica la presenza di un assembly di avvio dell'hosting da attivare in fase di runtime.

L'assembly di ingresso o l'assembly contenente la classe `Startup` viene analizzato automaticamente per la ricerca dell'attributo `HostingStartup`. L'elenco di assembly in cui cercare gli attributi `HostingStartup` viene caricato in fase di runtime dalla configurazione in [WebHostDefaults.HostingStartupAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupAssembliesKey). L'elenco di assembly da escludere dalla ricerca viene caricato da [WebHostDefaults.HostingStartupExcludeAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupExcludeAssembliesKey).

Nell'esempio seguente lo spazio dei nomi dell'assembly di avvio dell'hosting è `StartupEnhancement`. La classe che contiene il codice di avvio dell'hosting è `StartupEnhancementHostingStartup`:

[!code-csharp[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.cs?name=snippet1)]

L'attributo `HostingStartup` si trova in genere nel file della classe di implementazione `IHostingStartup` dell'assembly di avvio dell'hosting.

## <a name="discover-loaded-hosting-startup-assemblies"></a>Individuare gli assembly di avvio dell'hosting caricati

Per individuare gli assembly di avvio dell'hosting caricati, abilitare la registrazione e controllare i log dell'app. Gli errori che si verificano durante il caricamento degli assembly vengono registrati. Gli assembly di avvio dell'hosting caricati vengono registrati a livello di debug e tutti gli errori vengono registrati.

## <a name="disable-automatic-loading-of-hosting-startup-assemblies"></a>Disabilitare il caricamento automatico di assembly di avvio dell'hosting

Per disabilitare il caricamento automatico degli assembly di avvio dell'hosting, usare uno degli approcci seguenti:

* Per evitare il caricamento di tutti gli assembly di avvio dell'hosting, impostare uno degli elementi seguenti su `true` o `1`:

  * Impedisci l'impostazione di configurazione host di avvio:

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseSetting(
                        WebHostDefaults.PreventHostingStartupKey, "true")
                    .UseStartup<Startup>();
            });
    ```

  * La variabile di ambiente `ASPNETCORE_PREVENTHOSTINGSTARTUP`.

* Per impedire il caricamento di specifici assembly di avvio dell'hosting, impostare uno degli elementi seguenti su una stringa delimitata da punti e virgola di assembly di avvio dell'hosting da escludere all'avvio:

  * Impostazione configurazione host di esclusione dell'avvio dell'hosting:

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseSetting(
                        WebHostDefaults.HostingStartupExcludeAssembliesKey, 
                        "{ASSEMBLY1;ASSEMBLY2; ...}")
                    .UseStartup<Startup>();
            });
    ```

  * La variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES`.

Se l'impostazione di configurazione host e la variabile di ambiente sono entrambe impostate, l'impostazione host controlla il comportamento.

La disabilitazione degli assembly di avvio dell'hosting tramite l'impostazione host o la variabile di ambiente ne determina la disabilitazione globale e la possibile disabilitazione di diverse caratteristiche di un'app.

## <a name="project"></a>Project

Creare l'avvio dell'hosting con uno dei tipi di progetto seguenti:

* [Libreria di classi](#class-library)
* [App console senza un punto di ingresso](#console-app-without-an-entry-point)

### <a name="class-library"></a>Libreria di classi

È possibile includere un miglioramento di avvio dell'hosting in una libreria di classi. La libreria contiene un attributo `HostingStartup`.

Il [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) include un' Razor app Pages, *HostingStartupApp* e una libreria di classi, *HostingStartupLibrary*. La libreria di classi:

* Contiene una classe di avvio dell'hosting, `ServiceKeyInjection`, che implementa `IHostingStartup`. `ServiceKeyInjection` aggiunge una coppia di stringhe di servizio alla configurazione dell'app usando il provider di configurazione in memoria ([AddInMemoryCollection](xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*)).
* Include un attributo `HostingStartup` che identifica lo spazio dei nomi e la classe di avvio dell'hosting.

Il `ServiceKeyInjection` metodo della classe <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> Usa un <xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> per aggiungere miglioramenti a un'app.

*HostingStartupLibrary/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupLibrary/ServiceKeyInjection.cs?name=snippet1)]

La pagina Indice dell'app legge ed esegue il rendering dei valori di configurazione per le due chiavi impostate dall'assembly di avvio dell'hosting della libreria di classi:

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=5-6,11-12)]

Il [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) include anche un progetto di pacchetto NuGet che offre un avvio dell'hosting separato, *HostingStartupPackage*. Il pacchetto ha le stesse caratteristiche della libreria di classi descritta in precedenza. Il pacchetto:

* Contiene una classe di avvio dell'hosting, `ServiceKeyInjection`, che implementa `IHostingStartup`. `ServiceKeyInjection` aggiunge una coppia di stringhe di servizio alla configurazione dell'app.
* Include un attributo `HostingStartup`.

*HostingStartupPackage/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupPackage/ServiceKeyInjection.cs?name=snippet1)]

La pagina Indice dell'app legge ed esegue il rendering dei valori di configurazione per le due chiavi impostate dall'assembly di avvio dell'hosting del pacchetto:

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/3.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=7-8,13-14)]

### <a name="console-app-without-an-entry-point"></a>App console senza un punto di ingresso

*Questo approccio è disponibile solo per le app .NET Core, non .NET Framework.*

Un miglioramento di avvio dell'hosting dinamico che non richiede un riferimento in fase di compilazione per l'attivazione può essere fornito in un'app console senza un punto di ingresso che contiene un attributo `HostingStartup`. La pubblicazione dell'app console produce un assembly di avvio dell'hosting che può essere utilizzato dall'archivio di runtime.

In questo processo viene usata un'app console senza un punto di ingresso perché:

* È necessario un file di dipendenze per utilizzare l'avvio dell'hosting nell'assembly di avvio dell'hosting. Un file di dipendenze è una risorsa di app eseguibile prodotta dalla pubblicazione di un'app, non una libreria.
* Una libreria non può essere aggiunta direttamente all'[archivio dei pacchetti di runtime](/dotnet/core/deploying/runtime-store) che richiede un progetto eseguibile indirizzato al runtime condiviso.

Durante la creazione di un avvio dell'hosting dinamico:

* Viene creato un assembly di avvio dell'hosting dall'app console senza un punto di ingresso che:
  * Include una classe che contiene l'implementazione `IHostingStartup`.
  * Include un attributo [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) per identificare la classe di implementazione `IHostingStartup`.
* L'app console viene pubblicata per ottenere le dipendenze dell'avvio dell'hosting. Una conseguenza della pubblicazione dell'app console è che le dipendenze non usate vengono eliminate dal file di dipendenze.
* Il file delle dipendenze viene modificato per impostare il percorso di runtime dell'assembly di avvio dell'hosting.
* L'assembly di avvio dell'hosting e il relativo file di dipendenze vengono posizionati nell'archivio dei pacchetti di runtime. Per individuare l'assembly di avvio dell'hosting e il relativo file di dipendenze, questi elementi sono elencati in una coppia di variabili di ambiente.

L'app console fa riferimento al pacchetto [Microsoft.AspNetCore.Hosting.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.Abstractions/):

[!code-xml[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.csproj)]

Un attributo [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) identifica una classe come implementazione di `IHostingStartup` per il caricamento e l'esecuzione durante la compilazione dell'oggetto <xref:Microsoft.AspNetCore.Hosting.IWebHost> . Nell'esempio seguente, lo spazio dei nomi è `StartupEnhancement` e la classe è `StartupEnhancementHostingStartup`:

[!code-csharp[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.cs?name=snippet1)]

Una classe implementa `IHostingStartup`. Il metodo della classe <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> Usa un <xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> per aggiungere miglioramenti a un'app. Nell'assembly di avvio dell'hosting `IHostingStartup.Configure` viene chiamato dal runtime prima di `Startup.Configure` nel codice utente, il che consente al codice utente di sovrascrivere qualsiasi configurazione fornita dall'assembly di avvio dell'hosting.

[!code-csharp[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement.cs?name=snippet2&highlight=3,5)]

Quando si compila un progetto `IHostingStartup`, il file delle dipendenze (*.deps.json*) imposta il percorso di `runtime` dell'assembly sulla cartella *bin*:

[!code-json[](platform-specific-configuration/samples-snapshot/3.x/StartupEnhancement1.deps.json?range=2-13&highlight=8)]

Viene visualizzata solo una parte del file. Il nome dell'assembly nell'esempio è `StartupEnhancement`.

## <a name="configuration-provided-by-the-hosting-startup"></a>Configurazione fornita dall'avvio dell'hosting

Esistono due approcci per la gestione di configurazione, a seconda che si voglia dare la precedenza alla configurazione d avvio dell'hosting o alla configurazione dell'app:

1. Fornire la configurazione all'app usando <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> per caricare la configurazione dopo l'esecuzione dei delegati <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> dell'app. La configurazione di avvio dell'hosting ha la priorità rispetto alla configurazione dell'app con questo approccio.
1. Fornire la configurazione all'app usando <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseConfiguration*> per caricare la configurazione prima dell'esecuzione dei delegati <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> dell'app. I valori di configurazione dell'app hanno la priorità rispetto a quelli forniti dall'avvio dell'hosting usando questo approccio.

```csharp
public class ConfigurationInjection : IHostingStartup
{
    public void Configure(IWebHostBuilder builder)
    {
        Dictionary<string, string> dict;

        builder.ConfigureAppConfiguration(config =>
        {
            dict = new Dictionary<string, string>
            {
                {"ConfigurationKey1", 
                    "From IHostingStartup: Higher priority " +
                    "than the app's configuration."},
            };

            config.AddInMemoryCollection(dict);
        });

        dict = new Dictionary<string, string>
        {
            {"ConfigurationKey2", 
                "From IHostingStartup: Lower priority " +
                "than the app's configuration."},
        };

        var builtConfig = new ConfigurationBuilder()
            .AddInMemoryCollection(dict)
            .Build();

        builder.UseConfiguration(builtConfig);
    }
}
```

## <a name="specify-the-hosting-startup-assembly"></a>Specificare l'assembly di avvio dell'hosting

Per un avvio dell'hosting fornito da una libreria di classi o da un'app console, specificare il nome dell'assembly di avvio dell'hosting nella variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`. La variabile di ambiente è un elenco di assembly delimitato da punto e virgola.

L'attributo `HostingStartup` viene cercato solo negli assembly di avvio dell'hosting. Per l'app di esempio *HostingStartupApp*, per individuare gli avvii dell'hosting descritti in precedenza, la variabile di ambiente viene impostata sul valore seguente:

```
HostingStartupLibrary;HostingStartupPackage;StartupDiagnostics
```

È inoltre possibile impostare un assembly di avvio dell'hosting utilizzando l'impostazione di configurazione host di assembly di avvio host:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseSetting(
                    WebHostDefaults.HostingStartupAssembliesKey, 
                    "{ASSEMBLY1;ASSEMBLY2; ...}")
                .UseStartup<Startup>();
        });
```

Quando sono presenti più montaggi di avvio host, i relativi <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> metodi vengono eseguiti nell'ordine in cui sono elencati gli assembly.

## <a name="activation"></a>Activation

Le opzioni di attivazione dell'avvio dell'hosting sono:

* [Archivio di runtime](#runtime-store): l'attivazione non richiede un riferimento in fase di compilazione per l'attivazione. L'app di esempio inserisce l'assembly di avvio dell'hosting e i file di dipendenze in una cartella *deployment* per facilitare la distribuzione dell'avvio dell'hosting in un ambiente con più computer. La cartella *deployment* include anche uno script PowerShell che crea o modifica le variabili di ambiente nel sistema di distribuzione per abilitare l'avvio dell'hosting.
* Riferimento in fase di compilazione richiesto per l'attivazione
  * [Pacchetto NuGet](#nuget-package)
  * [Cartella bin del progetto](#project-bin-folder)

### <a name="runtime-store"></a>Archivio di runtime

L'implementazione dell'avvio dell'hosting viene inserita nell'[archivio di runtime](/dotnet/core/deploying/runtime-store). L'app migliorata non richiede un riferimento in fase di compilazione all'assembly.

Dopo la compilazione dell'avvio dell'hosting, viene generato un archivio di runtime usando il file di progetto del manifesto e il comando [dotnet store](/dotnet/core/tools/dotnet-store).

```dotnetcli
dotnet store --manifest {MANIFEST FILE} --runtime {RUNTIME IDENTIFIER} --output {OUTPUT LOCATION} --skip-optimization
```

Nell'app di esempio (progetto *RuntimeStore*) viene usato il comando seguente:

```dotnetcli
dotnet store --manifest store.manifest.csproj --runtime win7-x64 --output ./deployment/store --skip-optimization
```

Per consentire al runtime di individuare l'archivio di runtime, il percorso dell'archivio di runtime viene aggiunto alla variabile di ambiente `DOTNET_SHARED_STORE`.

**Modificare e inserire il file di dipendenze dell'avvio dell'hosting**

Per attivare il miglioramento senza un riferimento al pacchetto per il miglioramento, specificare le dipendenze aggiuntive per il runtime con `additionalDeps`. `additionalDeps` consente di:

* Estendere il grafo della libreria dell'app fornendo un set di file *.deps.json* aggiuntivi da unire al file *.deps.json* proprio dell'app all'avvio.
* Rendere l'assembly di avvio dell'hosting individuabile e caricabile.

L'approccio consigliato per la generazione del file delle dipendenze aggiuntive è:

 1. Eseguire `dotnet publish` sul file manifesto dell'archivio di runtime indicato nella sezione precedente.
 1. Rimuovere il riferimento al manifesto dalle librerie e la `runtime` sezione del *.deps.jsrisultante sul* file.

Nel progetto di esempio la proprietà `store.manifest/1.0.0` viene rimossa da `targets` e dalla sezione `libraries`:

```json
{
  "runtimeTarget": {
    "name": ".NETCoreApp,Version=v3.0",
    "signature": ""
  },
  "compilationOptions": {},
  "targets": {
    ".NETCoreApp,Version=v3.0": {
      "store.manifest/1.0.0": {
        "dependencies": {
          "StartupDiagnostics": "1.0.0"
        },
        "runtime": {
          "store.manifest.dll": {}
        }
      },
      "StartupDiagnostics/1.0.0": {
        "runtime": {
          "lib/netcoreapp3.0/StartupDiagnostics.dll": {
            "assemblyVersion": "1.0.0.0",
            "fileVersion": "1.0.0.0"
          }
        }
      }
    }
  },
  "libraries": {
    "store.manifest/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "StartupDiagnostics/1.0.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-xrhzuNSyM5/f4ZswhooJ9dmIYLP64wMnqUJSyTKVDKDVj5T+qtzypl8JmM/aFJLLpYrf0FYpVWvGujd7/FfMEw==",
      "path": "startupdiagnostics/1.0.0",
      "hashPath": "startupdiagnostics.1.0.0.nupkg.sha512"
    }
  }
}
```

Posizionare il file *.deps.json* nel percorso seguente:

```
{ADDITIONAL DEPENDENCIES PATH}/shared/{SHARED FRAMEWORK NAME}/{SHARED FRAMEWORK VERSION}/{ENHANCEMENT ASSEMBLY NAME}.deps.json
```

* `{ADDITIONAL DEPENDENCIES PATH}`: Percorso aggiunto alla `DOTNET_ADDITIONAL_DEPS` variabile di ambiente.
* `{SHARED FRAMEWORK NAME}`: Framework condiviso necessario per questo file di dipendenze aggiuntivo.
* `{SHARED FRAMEWORK VERSION}`: Versione minima del Framework condiviso.
* `{ENHANCEMENT ASSEMBLY NAME}`: Nome dell'assembly della funzionalità avanzata.

Nell'app di esempio (progetto *RuntimeStore*), il file di dipendenze aggiuntive viene posizionato nel percorso seguente:

```
deployment/additionalDeps/shared/Microsoft.AspNetCore.App/3.0.0/StartupDiagnostics.deps.json
```

Affinché il runtime possa individuare il percorso dell'archivio di runtime, il percorso del file di dipendenze aggiuntive viene aggiunto alla variabile di ambiente `DOTNET_ADDITIONAL_DEPS`.

Nell'app di esempio (progetto *RuntimeStore*), la creazione dell'archivio di runtime e la generazione del file di dipendenze aggiuntive vengono eseguite tramite uno script di [PowerShell](/powershell/scripting/powershell-scripting).

Per esempi che illustrano come impostare le variabili di ambiente per diversi sistemi operativi, vedere [Usare più ambienti](xref:fundamentals/environments).

**Distribuzione**

Per facilitare la distribuzione di un avvio dell'hosting in un ambiente con più computer, l'app di esempio crea una cartella *deplyment* nell'output pubblicato che contiene:

* L'archivio di runtime dell'avvio dell'hosting.
* Il file di dipendenze dell'avvio dell'hosting.
* Uno script di PowerShell che crea o modifica `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`, `DOTNET_SHARED_STORE` e `DOTNET_ADDITIONAL_DEPS` per supportare l'attivazione dell'avvio dell'hosting. Eseguire lo script da un prompt dei comandi di PowerShell amministrativo del sistema di distribuzione.

### <a name="nuget-package"></a>Pacchetto NuGet

È possibile includere un miglioramento di avvio dell'hosting in un pacchetto NuGet. Il pacchetto include un attributo `HostingStartup`. I tipi di avvio dell'hosting offerti dal pacchetto vengono resi disponibili all'app usando uno degli approcci seguenti:

* Il file di progetto dell'app migliorata crea un riferimento al pacchetto per l'avvio dell'hosting nel file di progetto dell'app (un riferimento in fase di compilazione). Dopo aver inserito il riferimento in fase di compilazione, l'assembly di avvio dell'hosting e tutte le relative dipendenze vengono incorporate nel file delle dipendenze dell'app (*.deps.json*). Questo approccio si applica a un pacchetto di assembly di avvio dell'hosting pubblicato in [nuget.org](https://www.nuget.org/).
* Il file di dipendenze dell'avvio dell'hosting viene reso disponibile all'app migliorata come descritto nella sezione [Archivio di runtime](#runtime-store) (senza un riferimento in fase di compilazione).

Per altre informazioni su pacchetti NuGet e l'archivio di runtime, vedere gli argomenti seguenti:

* [Come creare un pacchetto NuGet con strumenti multipiattaforma](/dotnet/core/deploying/creating-nuget-packages)
* [Pubblicazione di pacchetti](/nuget/create-packages/publish-a-package)
* [Archivio pacchetti di runtime](/dotnet/core/deploying/runtime-store)

### <a name="project-bin-folder"></a>Cartella bin del progetto

È possibile inserire un avvio dell'hosting tramite un assembly distribuito tramite *bin* nell'app migliorata. I tipi di avvio dell'hosting offerti dall'assembly vengono resi disponibili all'app usando uno degli approcci seguenti:

* Il file di progetto dell'app migliorata crea un riferimento all'assembly nell'avvio dell'hosting startup (riferimento in fase di compilazione). Dopo aver inserito il riferimento in fase di compilazione, l'assembly di avvio dell'hosting e tutte le relative dipendenze vengono incorporate nel file delle dipendenze dell'app (*.deps.json*). Questo approccio si applica quando lo scenario di distribuzione richiede la creazione di un riferimento in fase di compilazione all'assembly dell'avvio dell'hosting (file *DLL*) e lo spostamento dell'assembly in uno dei due elementi seguenti:
  * Il progetto che lo utilizza.
  * Una posizione accessibile dal progetto che lo utilizza.
* Il file di dipendenze dell'avvio dell'hosting viene reso disponibile all'app migliorata come descritto nella sezione [Archivio di runtime](#runtime-store) (senza un riferimento in fase di compilazione).
* Quando la destinazione è .NET Framework, l'assembly può essere caricato nel contesto di caricamento predefinito, che in .NET Framework significa che l'assembly si trova in una delle posizioni seguenti:
  * Percorso di base dell'applicazione: la cartella *bin* in cui si trova l'eseguibile dell'app (con *estensione exe*).
  * Global assembly cache (GAC): la GAC archivia gli assembly condivisi da diverse app .NET Framework. Per ulteriori informazioni, vedere [procedura: installare un assembly nella global assembly cache](/dotnet/framework/app-domains/how-to-install-an-assembly-into-the-gac) nella documentazione di .NET Framework.

## <a name="sample-code"></a>Codice di esempio

Il [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) ([come eseguire il download](xref:index#how-to-download-a-sample)) illustra gli scenari di implementazione dell'avvio dell'hosting:

* Due assembly di avvio dell'hosting (librerie di classi) impostano ognuno una coppia chiave-valore di configurazione in memoria:
  * Pacchetto NuGet (*HostingStartupPackage*)
  * Libreria di classi (*HostingStartupLibrary*)
* L'avvio dell'hosting viene attivato da un assembly distribuito tramite archivio di runtime (*StartupDiagnostics*). All'avvio l'assembly aggiunge all'app due middleware che offrono informazioni di diagnostica su:
  * Servizi registrati
  * Indirizzo (schema, host, base del percorso, percorso, stringa di query)
  * Connessione (IP remoto, porta remota, IP locale, porta locale, certificato client)
  * Intestazioni della richiesta
  * Variabili di ambiente

Per eseguire l'esempio:

**Attivazione da un pacchetto NuGet**

1. Compilare il pacchetto *HostingStartupPackage* con il comando [dotnet pack](/dotnet/core/tools/dotnet-pack).
1. Aggiungere il nome dell'assembly del pacchetto *HostingStartupPackage* alla variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`.
1. Compila ed esegui l'app. L'app migliorata include un riferimento al pacchetto (riferimento in fase di compilazione). `<PropertyGroup>` nel file di progetto dell'app specifica l'output del progetto del pacchetto (*../HostingStartupPackage/bin/Debug*) come origine del pacchetto. Ciò consente all'app di usare il pacchetto senza caricare il pacchetto in [NuGet.org](https://www.nuget.org/). Per ulteriori informazioni, vedere le note nel file di progetto di HostingStartupApp.

   ```xml
   <PropertyGroup>
     <RestoreSources>$(RestoreSources);https://api.nuget.org/v3/index.json;../HostingStartupPackage/bin/Debug</RestoreSources>
   </PropertyGroup>
   ```

1. Si noti che i valori di chiave di configurazione del servizio di cui è stato eseguito il rendering tramite la pagina Indice corrispondono ai valori impostati dal metodo `ServiceKeyInjection.Configure` del pacchetto.

Se si apportano modifiche al progetto *HostingStartupPackage* e lo si ricompila, cancellare le cache del pacchetto NuGet locale per assicurarsi che *HostingStartupApp* riceva il pacchetto aggiornato e non un pacchetto non aggiornato proveniente dalla cache locale. Per cancellare le cache NuGet locali, eseguire il comando [dotnet nuget locals](/dotnet/core/tools/dotnet-nuget-locals) seguente:

```dotnetcli
dotnet nuget locals all --clear
```

**Attivazione da una libreria di classi**

1. Compilare la libreria di classi *HostingStartupLibrary* con il comando [dotnet build](/dotnet/core/tools/dotnet-build).
1. Aggiungere il nome dell'assembly della libreria di classi di *HostingStartupLibrary* alla variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`.
1. Distribuire tramite *bin* l'assembly della libreria di classi all'app copiando il file *HostingStartupLibrary.dll* dall'output compilato della libreria di classi alla cartella *bin/Debug* dell'app.
1. Compila ed esegui l'app. `<ItemGroup>`Nel file di progetto dell'app fa riferimento all'assembly della libreria di classi (*.\bin\Debug\netcoreapp3.0\HostingStartupLibrary.dll*) (riferimento in fase di compilazione). Per altre informazioni, vedere le note nel file di progetto di HostingStartupApp.

   ```xml
   <ItemGroup>
     <Reference Include=".\\bin\\Debug\\netcoreapp3.0\\HostingStartupLibrary.dll">
       <HintPath>.\bin\Debug\netcoreapp3.0\HostingStartupLibrary.dll</HintPath>
       <SpecificVersion>False</SpecificVersion> 
     </Reference>
   </ItemGroup>
   ```

1. Si noti che i valori di chiave di configurazione del servizio di cui è stato eseguito il rendering tramite la pagina Indice corrispondono ai valori impostati dal metodo `ServiceKeyInjection.Configure` della libreria di classi.

**Attivazione da un assembly distribuito tramite l'archivio di runtime**

1. Il progetto *StartupDiagnostics* usa [PowerShell](/powershell/scripting/powershell-scripting) per modificare il relativo file *StartupDiagnostics.deps.json*. PowerShell viene installato per impostazione predefinita in Windows a partire da Windows 7 SP1 e Windows Server 2008 R2 SP1. Per ottenere PowerShell su altre piattaforme, vedere [installazione di diverse versioni di PowerShell](/powershell/scripting/install/installing-powershell).
1. Eseguire lo script *build.ps1* nella cartella *RuntimeStore*. Lo script:
   * Genera il `StartupDiagnostics` pacchetto nella cartella *obj\packages* .
   * Genera l'archivio di runtime per `StartupDiagnostics` nella cartella *store*. Il comando `dotnet store` nello script usa l' [identificatore di runtime (RID)](/dotnet/core/rid-catalog)`win7-x64` per un avvio dell'hosting distribuito su Windows. Quando si specifica l'avvio dell'hosting per un runtime diverso, immettere il RID corretto nella riga 37 dello script. L'archivio di runtime di verrà `StartupDiagnostics` spostato in un secondo momento nell'archivio di runtime dell'utente o del sistema nel computer in cui verrà utilizzato l'assembly. Il percorso di installazione dell'archivio di runtime utente per l' `StartupDiagnostics` assembly è *. dotnet/Store/x64/netcoreapp 3.0/startupdiagnostics/1.0.0/lib/netcoreapp 3.0/StartupDiagnostics.dll*.
   * Genera `additionalDeps` per `StartupDiagnostics` nella cartella *additionalDeps* . Le dipendenze aggiuntive verranno spostate in un secondo momento alle dipendenze aggiuntive del sistema o dell'utente. Il `StartupDiagnostics` percorso di installazione delle dipendenze aggiuntive dell'utente è *. dotnet/x64/AdditionalDeps/StartupDiagnostics/Shared/Microsoft. NETCore. app/3.0.0/StartupDiagnostics.deps.json*.
   * Posiziona il file *deploy.ps1* nella cartella *deployment*.
1. Eseguire lo script *deploy.ps1* nella cartella *deployment*. Lo script aggiunge:
   * `StartupDiagnostics` alla variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`.
   * Percorso delle dipendenze di avvio dell'hosting (nella cartella di *distribuzione* del progetto RuntimeStore) alla `DOTNET_ADDITIONAL_DEPS` variabile di ambiente.
   * Percorso dell'archivio di runtime (nella cartella di *distribuzione* del progetto RuntimeStore) alla `DOTNET_SHARED_STORE` variabile di ambiente.
1. Eseguire l'app di esempio.
1. Richiedere l'endpoint `/services` per visualizzare i servizi registrati dell'app. Richiedere l'endpoint `/diag` per visualizzare le informazioni di diagnostica.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Un' <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> implementazione di (host Startup) aggiunge miglioramenti a un'app all'avvio da un assembly esterno. Una libreria esterna può ad esempio usare un'implementazione di avvio dell'hosting per offrire servizi o provider di configurazione aggiuntivi a un'app.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="hostingstartup-attribute"></a>Attributo HostingStartup

Un attributo [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) indica la presenza di un assembly di avvio dell'hosting da attivare in fase di runtime.

L'assembly di ingresso o l'assembly contenente la classe `Startup` viene analizzato automaticamente per la ricerca dell'attributo `HostingStartup`. L'elenco di assembly in cui cercare gli attributi `HostingStartup` viene caricato in fase di runtime dalla configurazione in [WebHostDefaults.HostingStartupAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupAssembliesKey). L'elenco di assembly da escludere dalla ricerca viene caricato da [WebHostDefaults.HostingStartupExcludeAssembliesKey](xref:Microsoft.AspNetCore.Hosting.WebHostDefaults.HostingStartupExcludeAssembliesKey). Per altre informazioni, vedere [Host Web: Assembly di avvio dell'hosting](xref:fundamentals/host/web-host#hosting-startup-assemblies) e [Host Web: Assembly di avvio dell'hosting da escludere](xref:fundamentals/host/web-host#hosting-startup-exclude-assemblies).

Nell'esempio seguente lo spazio dei nomi dell'assembly di avvio dell'hosting è `StartupEnhancement`. La classe che contiene il codice di avvio dell'hosting è `StartupEnhancementHostingStartup`:

[!code-csharp[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.cs?name=snippet1)]

L'attributo `HostingStartup` si trova in genere nel file della classe di implementazione `IHostingStartup` dell'assembly di avvio dell'hosting.

## <a name="discover-loaded-hosting-startup-assemblies"></a>Individuare gli assembly di avvio dell'hosting caricati

Per individuare gli assembly di avvio dell'hosting caricati, abilitare la registrazione e controllare i log dell'app. Gli errori che si verificano durante il caricamento degli assembly vengono registrati. Gli assembly di avvio dell'hosting caricati vengono registrati a livello di debug e tutti gli errori vengono registrati.

## <a name="disable-automatic-loading-of-hosting-startup-assemblies"></a>Disabilitare il caricamento automatico di assembly di avvio dell'hosting

Per disabilitare il caricamento automatico degli assembly di avvio dell'hosting, usare uno degli approcci seguenti:

* Per evitare il caricamento di tutti gli assembly di avvio dell'hosting, impostare uno degli elementi seguenti su `true` o `1`:
  * Impostazione di configurazione host per [impedire l'avvio dell'hosting](xref:fundamentals/host/web-host#prevent-hosting-startup).
  * La variabile di ambiente `ASPNETCORE_PREVENTHOSTINGSTARTUP`.
* Per impedire il caricamento di specifici assembly di avvio dell'hosting, impostare uno degli elementi seguenti su una stringa delimitata da punti e virgola di assembly di avvio dell'hosting da escludere all'avvio:
  * Impostazione di configurazione host per [assembly di avvio dell'hosting da escludere](xref:fundamentals/host/web-host#hosting-startup-exclude-assemblies).
  * La variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES`.

Se l'impostazione di configurazione host e la variabile di ambiente sono entrambe impostate, l'impostazione host controlla il comportamento.

La disabilitazione degli assembly di avvio dell'hosting tramite l'impostazione host o la variabile di ambiente ne determina la disabilitazione globale e la possibile disabilitazione di diverse caratteristiche di un'app.

## <a name="project"></a>Project

Creare l'avvio dell'hosting con uno dei tipi di progetto seguenti:

* [Libreria di classi](#class-library)
* [App console senza un punto di ingresso](#console-app-without-an-entry-point)

### <a name="class-library"></a>Libreria di classi

È possibile includere un miglioramento di avvio dell'hosting in una libreria di classi. La libreria contiene un attributo `HostingStartup`.

Il [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) include un' Razor app Pages, *HostingStartupApp* e una libreria di classi, *HostingStartupLibrary*. La libreria di classi:

* Contiene una classe di avvio dell'hosting, `ServiceKeyInjection`, che implementa `IHostingStartup`. `ServiceKeyInjection` aggiunge una coppia di stringhe di servizio alla configurazione dell'app usando il provider di configurazione in memoria ([AddInMemoryCollection](xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*)).
* Include un attributo `HostingStartup` che identifica lo spazio dei nomi e la classe di avvio dell'hosting.

Il `ServiceKeyInjection` metodo della classe <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> Usa un <xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> per aggiungere miglioramenti a un'app.

*HostingStartupLibrary/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupLibrary/ServiceKeyInjection.cs?name=snippet1)]

La pagina Indice dell'app legge ed esegue il rendering dei valori di configurazione per le due chiavi impostate dall'assembly di avvio dell'hosting della libreria di classi:

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=5-6,11-12)]

Il [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) include anche un progetto di pacchetto NuGet che offre un avvio dell'hosting separato, *HostingStartupPackage*. Il pacchetto ha le stesse caratteristiche della libreria di classi descritta in precedenza. Il pacchetto:

* Contiene una classe di avvio dell'hosting, `ServiceKeyInjection`, che implementa `IHostingStartup`. `ServiceKeyInjection` aggiunge una coppia di stringhe di servizio alla configurazione dell'app.
* Include un attributo `HostingStartup`.

*HostingStartupPackage/ServiceKeyInjection.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupPackage/ServiceKeyInjection.cs?name=snippet1)]

La pagina Indice dell'app legge ed esegue il rendering dei valori di configurazione per le due chiavi impostate dall'assembly di avvio dell'hosting del pacchetto:

*HostingStartupApp/Pages/Index.cshtml.cs*:

[!code-csharp[](platform-specific-configuration/samples/2.x/HostingStartupApp/Pages/Index.cshtml.cs?name=snippet1&highlight=7-8,13-14)]

### <a name="console-app-without-an-entry-point"></a>App console senza un punto di ingresso

*Questo approccio è disponibile solo per le app .NET Core, non .NET Framework.*

Un miglioramento di avvio dell'hosting dinamico che non richiede un riferimento in fase di compilazione per l'attivazione può essere fornito in un'app console senza un punto di ingresso che contiene un attributo `HostingStartup`. La pubblicazione dell'app console produce un assembly di avvio dell'hosting che può essere utilizzato dall'archivio di runtime.

In questo processo viene usata un'app console senza un punto di ingresso perché:

* È necessario un file di dipendenze per utilizzare l'avvio dell'hosting nell'assembly di avvio dell'hosting. Un file di dipendenze è una risorsa di app eseguibile prodotta dalla pubblicazione di un'app, non una libreria.
* Una libreria non può essere aggiunta direttamente all'[archivio dei pacchetti di runtime](/dotnet/core/deploying/runtime-store) che richiede un progetto eseguibile indirizzato al runtime condiviso.

Durante la creazione di un avvio dell'hosting dinamico:

* Viene creato un assembly di avvio dell'hosting dall'app console senza un punto di ingresso che:
  * Include una classe che contiene l'implementazione `IHostingStartup`.
  * Include un attributo [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) per identificare la classe di implementazione `IHostingStartup`.
* L'app console viene pubblicata per ottenere le dipendenze dell'avvio dell'hosting. Una conseguenza della pubblicazione dell'app console è che le dipendenze non usate vengono eliminate dal file di dipendenze.
* Il file delle dipendenze viene modificato per impostare il percorso di runtime dell'assembly di avvio dell'hosting.
* L'assembly di avvio dell'hosting e il relativo file di dipendenze vengono posizionati nell'archivio dei pacchetti di runtime. Per individuare l'assembly di avvio dell'hosting e il relativo file di dipendenze, questi elementi sono elencati in una coppia di variabili di ambiente.

L'app console fa riferimento al pacchetto [Microsoft.AspNetCore.Hosting.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.Abstractions/):

[!code-xml[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.csproj)]

Un attributo [HostingStartup](xref:Microsoft.AspNetCore.Hosting.HostingStartupAttribute) identifica una classe come implementazione di `IHostingStartup` per il caricamento e l'esecuzione durante la compilazione dell'oggetto <xref:Microsoft.AspNetCore.Hosting.IWebHost> . Nell'esempio seguente, lo spazio dei nomi è `StartupEnhancement` e la classe è `StartupEnhancementHostingStartup`:

[!code-csharp[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.cs?name=snippet1)]

Una classe implementa `IHostingStartup`. Il metodo della classe <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> Usa un <xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> per aggiungere miglioramenti a un'app. Nell'assembly di avvio dell'hosting `IHostingStartup.Configure` viene chiamato dal runtime prima di `Startup.Configure` nel codice utente, il che consente al codice utente di sovrascrivere qualsiasi configurazione fornita dall'assembly di avvio dell'hosting.

[!code-csharp[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement.cs?name=snippet2&highlight=3,5)]

Quando si compila un progetto `IHostingStartup`, il file delle dipendenze (*.deps.json*) imposta il percorso di `runtime` dell'assembly sulla cartella *bin*:

[!code-json[](platform-specific-configuration/samples-snapshot/2.x/StartupEnhancement1.deps.json?range=2-13&highlight=8)]

Viene visualizzata solo una parte del file. Il nome dell'assembly nell'esempio è `StartupEnhancement`.

## <a name="configuration-provided-by-the-hosting-startup"></a>Configurazione fornita dall'avvio dell'hosting

Esistono due approcci per la gestione di configurazione, a seconda che si voglia dare la precedenza alla configurazione d avvio dell'hosting o alla configurazione dell'app:

1. Fornire la configurazione all'app usando <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> per caricare la configurazione dopo l'esecuzione dei delegati <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> dell'app. La configurazione di avvio dell'hosting ha la priorità rispetto alla configurazione dell'app con questo approccio.
1. Fornire la configurazione all'app usando <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseConfiguration*> per caricare la configurazione prima dell'esecuzione dei delegati <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureAppConfiguration*> dell'app. I valori di configurazione dell'app hanno la priorità rispetto a quelli forniti dall'avvio dell'hosting usando questo approccio.

```csharp
public class ConfigurationInjection : IHostingStartup
{
    public void Configure(IWebHostBuilder builder)
    {
        Dictionary<string, string> dict;

        builder.ConfigureAppConfiguration(config =>
        {
            dict = new Dictionary<string, string>
            {
                {"ConfigurationKey1", 
                    "From IHostingStartup: Higher priority " +
                    "than the app's configuration."},
            };

            config.AddInMemoryCollection(dict);
        });

        dict = new Dictionary<string, string>
        {
            {"ConfigurationKey2", 
                "From IHostingStartup: Lower priority " +
                "than the app's configuration."},
        };

        var builtConfig = new ConfigurationBuilder()
            .AddInMemoryCollection(dict)
            .Build();

        builder.UseConfiguration(builtConfig);
    }
}
```

## <a name="specify-the-hosting-startup-assembly"></a>Specificare l'assembly di avvio dell'hosting

Per un avvio dell'hosting fornito da una libreria di classi o da un'app console, specificare il nome dell'assembly di avvio dell'hosting nella variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`. La variabile di ambiente è un elenco di assembly delimitato da punto e virgola.

L'attributo `HostingStartup` viene cercato solo negli assembly di avvio dell'hosting. Per l'app di esempio *HostingStartupApp*, per individuare gli avvii dell'hosting descritti in precedenza, la variabile di ambiente viene impostata sul valore seguente:

```
HostingStartupLibrary;HostingStartupPackage;StartupDiagnostics
```

È possibile impostare l'assembly di avvio dell'hosting anche tramite l'impostazione di configurazione host [Assembly di avvio dell'hosting](xref:fundamentals/host/web-host#hosting-startup-assemblies).

Quando sono presenti più montaggi di avvio host, i relativi <xref:Microsoft.AspNetCore.Hosting.IHostingStartup.Configure*> metodi vengono eseguiti nell'ordine in cui sono elencati gli assembly.

## <a name="activation"></a>Activation

Le opzioni di attivazione dell'avvio dell'hosting sono:

* [Archivio di runtime](#runtime-store): l'attivazione non richiede un riferimento in fase di compilazione per l'attivazione. L'app di esempio inserisce l'assembly di avvio dell'hosting e i file di dipendenze in una cartella *deployment* per facilitare la distribuzione dell'avvio dell'hosting in un ambiente con più computer. La cartella *deployment* include anche uno script PowerShell che crea o modifica le variabili di ambiente nel sistema di distribuzione per abilitare l'avvio dell'hosting.
* Riferimento in fase di compilazione richiesto per l'attivazione
  * [Pacchetto NuGet](#nuget-package)
  * [Cartella bin del progetto](#project-bin-folder)

### <a name="runtime-store"></a>Archivio di runtime

L'implementazione dell'avvio dell'hosting viene inserita nell'[archivio di runtime](/dotnet/core/deploying/runtime-store). L'app migliorata non richiede un riferimento in fase di compilazione all'assembly.

Dopo la compilazione dell'avvio dell'hosting, viene generato un archivio di runtime usando il file di progetto del manifesto e il comando [dotnet store](/dotnet/core/tools/dotnet-store).

```dotnetcli
dotnet store --manifest {MANIFEST FILE} --runtime {RUNTIME IDENTIFIER} --output {OUTPUT LOCATION} --skip-optimization
```

Nell'app di esempio (progetto *RuntimeStore*) viene usato il comando seguente:

```dotnetcli
dotnet store --manifest store.manifest.csproj --runtime win7-x64 --output ./deployment/store --skip-optimization
```

Per consentire al runtime di individuare l'archivio di runtime, il percorso dell'archivio di runtime viene aggiunto alla variabile di ambiente `DOTNET_SHARED_STORE`.

**Modificare e inserire il file di dipendenze dell'avvio dell'hosting**

Per attivare il miglioramento senza un riferimento al pacchetto per il miglioramento, specificare le dipendenze aggiuntive per il runtime con `additionalDeps`. `additionalDeps` consente di:

* Estendere il grafo della libreria dell'app fornendo un set di file *.deps.json* aggiuntivi da unire al file *.deps.json* proprio dell'app all'avvio.
* Rendere l'assembly di avvio dell'hosting individuabile e caricabile.

L'approccio consigliato per la generazione del file delle dipendenze aggiuntive è:

 1. Eseguire `dotnet publish` sul file manifesto dell'archivio di runtime indicato nella sezione precedente.
 1. Rimuovere il riferimento al manifesto dalle librerie e la `runtime` sezione del *.deps.jsrisultante sul* file.

Nel progetto di esempio la proprietà `store.manifest/1.0.0` viene rimossa da `targets` e dalla sezione `libraries`:

```json
{
  "runtimeTarget": {
    "name": ".NETCoreApp,Version=v2.1",
    "signature": "4ea77c7b75ad1895ae1ea65e6ba2399010514f99"
  },
  "compilationOptions": {},
  "targets": {
    ".NETCoreApp,Version=v2.1": {
      "store.manifest/1.0.0": {
        "dependencies": {
          "StartupDiagnostics": "1.0.0"
        },
        "runtime": {
          "store.manifest.dll": {}
        }
      },
      "StartupDiagnostics/1.0.0": {
        "runtime": {
          "lib/netcoreapp2.1/StartupDiagnostics.dll": {
            "assemblyVersion": "1.0.0.0",
            "fileVersion": "1.0.0.0"
          }
        }
      }
    }
  },
  "libraries": {
    "store.manifest/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "StartupDiagnostics/1.0.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-oiQr60vBQW7+nBTmgKLSldj06WNLRTdhOZpAdEbCuapoZ+M2DJH2uQbRLvFT8EGAAv4TAKzNtcztpx5YOgBXQQ==",
      "path": "startupdiagnostics/1.0.0",
      "hashPath": "startupdiagnostics.1.0.0.nupkg.sha512"
    }
  }
}
```

Posizionare il file *.deps.json* nel percorso seguente:

```
{ADDITIONAL DEPENDENCIES PATH}/shared/{SHARED FRAMEWORK NAME}/{SHARED FRAMEWORK VERSION}/{ENHANCEMENT ASSEMBLY NAME}.deps.json
```

* `{ADDITIONAL DEPENDENCIES PATH}`: Percorso aggiunto alla `DOTNET_ADDITIONAL_DEPS` variabile di ambiente.
* `{SHARED FRAMEWORK NAME}`: Framework condiviso necessario per questo file di dipendenze aggiuntivo.
* `{SHARED FRAMEWORK VERSION}`: Versione minima del Framework condiviso.
* `{ENHANCEMENT ASSEMBLY NAME}`: Nome dell'assembly della funzionalità avanzata.

Nell'app di esempio (progetto *RuntimeStore*), il file di dipendenze aggiuntive viene posizionato nel percorso seguente:

```
deployment/additionalDeps/shared/Microsoft.AspNetCore.App/2.1.0/StartupDiagnostics.deps.json
```

Affinché il runtime possa individuare il percorso dell'archivio di runtime, il percorso del file di dipendenze aggiuntive viene aggiunto alla variabile di ambiente `DOTNET_ADDITIONAL_DEPS`.

Nell'app di esempio (progetto *RuntimeStore*), la creazione dell'archivio di runtime e la generazione del file di dipendenze aggiuntive vengono eseguite tramite uno script di [PowerShell](/powershell/scripting/powershell-scripting).

Per esempi che illustrano come impostare le variabili di ambiente per diversi sistemi operativi, vedere [Usare più ambienti](xref:fundamentals/environments).

**Distribuzione**

Per facilitare la distribuzione di un avvio dell'hosting in un ambiente con più computer, l'app di esempio crea una cartella *deplyment* nell'output pubblicato che contiene:

* L'archivio di runtime dell'avvio dell'hosting.
* Il file di dipendenze dell'avvio dell'hosting.
* Uno script di PowerShell che crea o modifica `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`, `DOTNET_SHARED_STORE` e `DOTNET_ADDITIONAL_DEPS` per supportare l'attivazione dell'avvio dell'hosting. Eseguire lo script da un prompt dei comandi di PowerShell amministrativo del sistema di distribuzione.

### <a name="nuget-package"></a>Pacchetto NuGet

È possibile includere un miglioramento di avvio dell'hosting in un pacchetto NuGet. Il pacchetto include un attributo `HostingStartup`. I tipi di avvio dell'hosting offerti dal pacchetto vengono resi disponibili all'app usando uno degli approcci seguenti:

* Il file di progetto dell'app migliorata crea un riferimento al pacchetto per l'avvio dell'hosting nel file di progetto dell'app (un riferimento in fase di compilazione). Dopo aver inserito il riferimento in fase di compilazione, l'assembly di avvio dell'hosting e tutte le relative dipendenze vengono incorporate nel file delle dipendenze dell'app (*.deps.json*). Questo approccio si applica a un pacchetto di assembly di avvio dell'hosting pubblicato in [nuget.org](https://www.nuget.org/).
* Il file di dipendenze dell'avvio dell'hosting viene reso disponibile all'app migliorata come descritto nella sezione [Archivio di runtime](#runtime-store) (senza un riferimento in fase di compilazione).

Per altre informazioni su pacchetti NuGet e l'archivio di runtime, vedere gli argomenti seguenti:

* [Come creare un pacchetto NuGet con strumenti multipiattaforma](/dotnet/core/deploying/creating-nuget-packages)
* [Pubblicazione di pacchetti](/nuget/create-packages/publish-a-package)
* [Archivio pacchetti di runtime](/dotnet/core/deploying/runtime-store)

### <a name="project-bin-folder"></a>Cartella bin del progetto

È possibile inserire un avvio dell'hosting tramite un assembly distribuito tramite *bin* nell'app migliorata. I tipi di avvio dell'hosting offerti dall'assembly vengono resi disponibili all'app usando uno degli approcci seguenti:

* Il file di progetto dell'app migliorata crea un riferimento all'assembly nell'avvio dell'hosting startup (riferimento in fase di compilazione). Dopo aver inserito il riferimento in fase di compilazione, l'assembly di avvio dell'hosting e tutte le relative dipendenze vengono incorporate nel file delle dipendenze dell'app (*.deps.json*). Questo approccio si applica quando lo scenario di distribuzione richiede la creazione di un riferimento in fase di compilazione all'assembly dell'avvio dell'hosting (file *DLL*) e lo spostamento dell'assembly in uno dei due elementi seguenti:
  * Il progetto che lo utilizza.
  * Una posizione accessibile dal progetto che lo utilizza.
* Il file di dipendenze dell'avvio dell'hosting viene reso disponibile all'app migliorata come descritto nella sezione [Archivio di runtime](#runtime-store) (senza un riferimento in fase di compilazione).
* Quando la destinazione è .NET Framework, l'assembly può essere caricato nel contesto di caricamento predefinito, che in .NET Framework significa che l'assembly si trova in una delle posizioni seguenti:
  * Percorso di base dell'applicazione: la cartella *bin* in cui si trova l'eseguibile dell'app (con *estensione exe*).
  * Global assembly cache (GAC): la GAC archivia gli assembly condivisi da diverse app .NET Framework. Per ulteriori informazioni, vedere [procedura: installare un assembly nella global assembly cache](/dotnet/framework/app-domains/how-to-install-an-assembly-into-the-gac) nella documentazione di .NET Framework.

## <a name="sample-code"></a>Codice di esempio

Il [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/platform-specific-configuration/samples/) ([come eseguire il download](xref:index#how-to-download-a-sample)) illustra gli scenari di implementazione dell'avvio dell'hosting:

* Due assembly di avvio dell'hosting (librerie di classi) impostano ognuno una coppia chiave-valore di configurazione in memoria:
  * Pacchetto NuGet (*HostingStartupPackage*)
  * Libreria di classi (*HostingStartupLibrary*)
* L'avvio dell'hosting viene attivato da un assembly distribuito tramite archivio di runtime (*StartupDiagnostics*). All'avvio l'assembly aggiunge all'app due middleware che offrono informazioni di diagnostica su:
  * Servizi registrati
  * Indirizzo (schema, host, base del percorso, percorso, stringa di query)
  * Connessione (IP remoto, porta remota, IP locale, porta locale, certificato client)
  * Intestazioni della richiesta
  * Variabili di ambiente

Per eseguire l'esempio:

**Attivazione da un pacchetto NuGet**

1. Compilare il pacchetto *HostingStartupPackage* con il comando [dotnet pack](/dotnet/core/tools/dotnet-pack).
1. Aggiungere il nome dell'assembly del pacchetto *HostingStartupPackage* alla variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`.
1. Compila ed esegui l'app. L'app migliorata include un riferimento al pacchetto (riferimento in fase di compilazione). `<PropertyGroup>` nel file di progetto dell'app specifica l'output del progetto del pacchetto (*../HostingStartupPackage/bin/Debug*) come origine del pacchetto. Ciò consente all'app di usare il pacchetto senza caricare il pacchetto in [NuGet.org](https://www.nuget.org/). Per ulteriori informazioni, vedere le note nel file di progetto di HostingStartupApp.

   ```xml
   <PropertyGroup>
     <RestoreSources>$(RestoreSources);https://api.nuget.org/v3/index.json;../HostingStartupPackage/bin/Debug</RestoreSources>
   </PropertyGroup>
   ```

1. Si noti che i valori di chiave di configurazione del servizio di cui è stato eseguito il rendering tramite la pagina Indice corrispondono ai valori impostati dal metodo `ServiceKeyInjection.Configure` del pacchetto.

Se si apportano modifiche al progetto *HostingStartupPackage* e lo si ricompila, cancellare le cache del pacchetto NuGet locale per assicurarsi che *HostingStartupApp* riceva il pacchetto aggiornato e non un pacchetto non aggiornato proveniente dalla cache locale. Per cancellare le cache NuGet locali, eseguire il comando [dotnet nuget locals](/dotnet/core/tools/dotnet-nuget-locals) seguente:

```dotnetcli
dotnet nuget locals all --clear
```

**Attivazione da una libreria di classi**

1. Compilare la libreria di classi *HostingStartupLibrary* con il comando [dotnet build](/dotnet/core/tools/dotnet-build).
1. Aggiungere il nome dell'assembly della libreria di classi di *HostingStartupLibrary* alla variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`.
1. Distribuire tramite *bin* l'assembly della libreria di classi all'app copiando il file *HostingStartupLibrary.dll* dall'output compilato della libreria di classi alla cartella *bin/Debug* dell'app.
1. Compila ed esegui l'app. `<ItemGroup>` nel file di progetto dell'app fa riferimento all'assembly della libreria di classi (*.\bin\Debug\netcoreapp2.1\HostingStartupLibrary.dll*) (riferimento in fase di compilazione). Per altre informazioni, vedere le note nel file di progetto di HostingStartupApp.

   ```xml
   <ItemGroup>
     <Reference Include=".\\bin\\Debug\\netcoreapp2.1\\HostingStartupLibrary.dll">
       <HintPath>.\bin\Debug\netcoreapp2.1\HostingStartupLibrary.dll</HintPath>
       <SpecificVersion>False</SpecificVersion>
     </Reference>
   </ItemGroup>
   ```

1. Si noti che i valori di chiave di configurazione del servizio di cui è stato eseguito il rendering tramite la pagina Indice corrispondono ai valori impostati dal metodo `ServiceKeyInjection.Configure` della libreria di classi.

**Attivazione da un assembly distribuito tramite l'archivio di runtime**

1. Il progetto *StartupDiagnostics* usa [PowerShell](/powershell/scripting/powershell-scripting) per modificare il relativo file *StartupDiagnostics.deps.json*. PowerShell viene installato per impostazione predefinita in Windows a partire da Windows 7 SP1 e Windows Server 2008 R2 SP1. Per ottenere PowerShell su altre piattaforme, vedere [installazione di diverse versioni di PowerShell](/powershell/scripting/install/installing-powershell).
1. Eseguire lo script *build.ps1* nella cartella *RuntimeStore*. Lo script:
   * Genera il `StartupDiagnostics` pacchetto nella cartella *obj\packages* .
   * Genera l'archivio di runtime per `StartupDiagnostics` nella cartella *store*. Il comando `dotnet store` nello script usa l' [identificatore di runtime (RID)](/dotnet/core/rid-catalog)`win7-x64` per un avvio dell'hosting distribuito su Windows. Quando si specifica l'avvio dell'hosting per un runtime diverso, immettere il RID corretto nella riga 37 dello script. L'archivio di runtime di verrà `StartupDiagnostics` spostato in un secondo momento nell'archivio di runtime dell'utente o del sistema nel computer in cui verrà utilizzato l'assembly. Il percorso di installazione dell'archivio di runtime utente per l' `StartupDiagnostics` assembly è *. dotnet/Store/x64/netcoreapp 2.2/startupdiagnostics/1.0.0/lib/netcoreapp 2.2/StartupDiagnostics.dll*.
   * Genera `additionalDeps` per `StartupDiagnostics` nella cartella *additionalDeps* . Le dipendenze aggiuntive verranno spostate in un secondo momento alle dipendenze aggiuntive del sistema o dell'utente. Il `StartupDiagnostics` percorso di installazione delle dipendenze aggiuntive dell'utente è *. dotnet/x64/AdditionalDeps/StartupDiagnostics/Shared/Microsoft. NETCore. app/2.2.0/StartupDiagnostics.deps.json*.
   * Posiziona il file *deploy.ps1* nella cartella *deployment*.
1. Eseguire lo script *deploy.ps1* nella cartella *deployment*. Lo script aggiunge:
   * `StartupDiagnostics` alla variabile di ambiente `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`.
   * Percorso delle dipendenze di avvio dell'hosting (nella cartella di *distribuzione* del progetto RuntimeStore) alla `DOTNET_ADDITIONAL_DEPS` variabile di ambiente.
   * Percorso dell'archivio di runtime (nella cartella di *distribuzione* del progetto RuntimeStore) alla `DOTNET_SHARED_STORE` variabile di ambiente.
1. Eseguire l'app di esempio.
1. Richiedere l'endpoint `/services` per visualizzare i servizi registrati dell'app. Richiedere l'endpoint `/diag` per visualizzare le informazioni di diagnostica.

::: moniker-end
