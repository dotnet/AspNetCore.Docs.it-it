---
title: Host generico .NET in ASP.NET Core
author: rick-anderson
description: Usare l'host generico di .NET Core nelle app ASP.NET Core.  L'host generico è responsabile della gestione dell'avvio e della durata delle app.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 4/17/2020
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
uid: fundamentals/host/generic-host
ms.openlocfilehash: ceb766cae1f8a1735f5b578a44f0c481927e47a5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586748"
---
# <a name="net-generic-host-in-aspnet-core"></a>Host generico .NET in ASP.NET Core

::: moniker range=">= aspnetcore-5.0"

I modelli di ASP.NET Core creano un host generico .NET Core ( <xref:Microsoft.Extensions.Hosting.HostBuilder> ).

In questo argomento vengono fornite informazioni sull'utilizzo di host generico .NET in ASP.NET Core. Per informazioni sull'uso di un host generico .NET nelle app console, vedere [.NET Generic Host](/dotnet/core/extensions/generic-host).

## <a name="host-definition"></a>Definizione host

Un *host* è un oggetto che incapsula le risorse di un'app, ad esempio:

* Inserimento di dipendenze (DI)
* Registrazione
* Configurazione
* `IHostedService` Implementazioni

Quando viene avviato, un host chiama <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> su ogni implementazione di <xref:Microsoft.Extensions.Hosting.IHostedService> registrata nella raccolta di servizi ospitati del contenitore dei servizi. In un'app Web, una delle implementazioni di `IHostedService` è un servizio web che avvia un'[implementazione del server HTTP](xref:fundamentals/index#servers).

Il motivo principale per cui tutte le risorse interdipendenti dell'app sono incluse in un unico oggetto è la gestione del ciclo di vita, vale a dire il controllo sull'avvio dell'app e sull'arresto normale.

## <a name="set-up-a-host"></a>Configurare un host

L'host è in genere configurato, compilato ed eseguito da codice nella classe `Program`. Il metodo `Main`:

* Chiama un metodo `CreateHostBuilder` per creare e configurare un oggetto generatore.
* Chiamate `Build` e metodi `Run` nell'oggetto generatore.

I modelli Web ASP.NET Core generano il codice seguente per creare un host:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

Il codice seguente crea un carico di lavoro non HTTP con un' `IHostedService` implementazione aggiunta al contenitore di inserimento delle dipendenze.

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

Per un carico di lavoro HTTP, il metodo `Main` è lo stesso ma `CreateHostBuilder` chiama `ConfigureWebHostDefaults`:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

Se l'app usa Entity Framework Core, non modificare il nome o la firma del metodo `CreateHostBuilder`. Gli [strumenti di Entity Framework Core](/ef/core/miscellaneous/cli/) si aspettano di trovare un metodo `CreateHostBuilder` che configura l'host senza eseguire l'app. Per altre informazioni, vedere [Creazione DbContext in fase di progettazione](/ef/core/miscellaneous/cli/dbcontext-creation).

## <a name="default-builder-settings"></a>Impostazioni predefinite del generatore

Il metodo <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*>:

* Imposta la [radice del contenuto](xref:fundamentals/index#content-root) sul percorso restituito da <xref:System.IO.Directory.GetCurrentDirectory*> .
* Carica la configurazione dell'host da:
  * Variabili di ambiente con prefisso `DOTNET_` .
  * Argomenti della riga di comando.
* Carica la configurazione dell'app da:
  * *appsettings.json*.
  * *appsettings.{Environment}.json*.
  * [Segreti dell'utente](xref:security/app-secrets) quando l'app viene eseguita nell'ambiente `Development`.
  * Variabili di ambiente.
  * Argomenti della riga di comando.
* Aggiunge i provider di [log](xref:fundamentals/logging/index) seguenti:
  * Console
  * Debug
  * EventSource
  * EventLog (solo quando è in esecuzione su Windows)
* Abilita la [convalida dell'ambito](xref:fundamentals/dependency-injection#scope-validation) e la [convalida delle dipendenze](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild) quando l'ambiente è lo sviluppo.

Il metodo <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*>:

* Carica la configurazione host dalle variabili di ambiente con prefisso `ASPNETCORE_` .
* Imposta il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web e lo configura usando i provider di configurazione dell'host dell'app. Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel/options>.
* Aggiunge [il middleware dell'applicazione di filtri dell'host](xref:fundamentals/servers/kestrel/host-filtering).
* Aggiunge il [middleware delle intestazioni con inoltri](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) se `ASPNETCORE_FORWARDEDHEADERS_ENABLED` è uguale a `true` .
* Abilita l'integrazione di IIS. Per le opzioni predefinite di IIS, vedere <xref:host-and-deploy/iis/index#iis-options>.

Le sezioni [Impostazioni per tutti i tipi di app](#settings-for-all-app-types) e [Impostazioni per le app Web](#settings-for-web-apps) più avanti in questo articolo mostrano come eseguire l'override delle impostazioni predefinite del generatore.

## <a name="framework-provided-services"></a>Servizi forniti dal framework

I servizi seguenti vengono registrati automaticamente:

* [IHostApplicationLifetime](#ihostapplicationlifetime)
* [IHostLifetime](#ihostlifetime)
* [IHostEnvironment / IWebHostEnvironment](#ihostenvironment)

Per ulteriori informazioni sui servizi forniti dal Framework, vedere <xref:fundamentals/dependency-injection#framework-provided-services> .

## <a name="ihostapplicationlifetime"></a>IHostApplicationLifetime

Inserire il servizio <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (in precedenza `IApplicationLifetime`) in qualsiasi classe per gestire le attività post-avvio e di arresto normale. Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi del gestore dell'evento di avvio e di arresto. L'interfaccia include anche un metodo `StopApplication`.

L'esempio seguente è un' `IHostedService` implementazione di che registra `IHostApplicationLifetime` gli eventi:

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a>IHostLifetime

L'implementazione <xref:Microsoft.Extensions.Hosting.IHostLifetime> controlla quando l'host viene avviato e quando si arresta. Viene usata l'ultima implementazione registrata.

`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` è l'implementazione `IHostLifetime` predefinita. `ConsoleLifetime`:

* Ascolta la <kbd>combinazione di tasti CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM e chiama <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication*> per avviare il processo di arresto.
* Sblocca estensioni come [RunAsync](#runasync) e [WaitForShutdownAsync](#waitforshutdownasync).

## <a name="ihostenvironment"></a>IHostEnvironment

Inserire il <xref:Microsoft.Extensions.Hosting.IHostEnvironment> servizio in una classe per ottenere informazioni sulle impostazioni seguenti:

* [ApplicationName](#applicationname)
* [EnvironmentName](#environmentname)
* [ContentRootPath](#contentroot)

Le app Web implementano l' `IWebHostEnvironment` interfaccia, che eredita `IHostEnvironment` e aggiunge [WebRootPath](#webroot).

## <a name="host-configuration"></a>Configurazione dell'host

La configurazione dell'host viene usata per le proprietà dell'implementazione <xref:Microsoft.Extensions.Hosting.IHostEnvironment>.

La configurazione dell'host è disponibile da [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) all'interno di <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>. Dopo `ConfigureAppConfiguration`, `HostBuilderContext.Configuration` viene sostituito con la configurazione dell'app.

Per aggiungere la configurazione dell'host, chiamare <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> su `IHostBuilder`. `ConfigureHostConfiguration` può essere chiamato più volte e i risultati si sommano. L'host usa l'ultima opzione che imposta un valore in una determinata chiave.

Il provider di variabili di ambiente con prefisso `DOTNET_` e argomenti della riga di comando sono inclusi in `CreateDefaultBuilder` . Per le app Web, viene aggiunto il provider di variabili di ambiente con prefisso `ASPNETCORE_`. Il prefisso viene rimosso quando vengono lette le variabili di ambiente. Ad esempio, il valore della variabile di ambiente per `ASPNETCORE_ENVIRONMENT` diventa il valore di configurazione dell'host per la chiave `environment`.

L'esempio seguente crea la configurazione host:

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a>Configurazione delle app

La configurazione dell'app viene creata chiamando <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> su `IHostBuilder`. `ConfigureAppConfiguration` può essere chiamato più volte e i risultati si sommano. L'app usa l'ultima opzione che imposta un valore in una determinata chiave. 

La configurazione creata da `ConfigureAppConfiguration` è disponibile in [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) per le operazioni successive e come servizio da DI. La configurazione dell'host viene aggiunta anche alla configurazione dell'app.

Per altre informazioni, vedere [Configurazione in ASP.NET Core](xref:fundamentals/configuration/index#configureappconfiguration).

## <a name="settings-for-all-app-types"></a>Impostazioni per tutti i tipi di app

Questa sezione elenca le impostazioni dell'host che si applicano ai carichi di lavoro HTTP e non-HTTP. Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`. Per ulteriori informazioni, vedere la sezione [impostazioni predefinite del generatore](#default-builder-settings) .

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a>ApplicationName

La proprietà [IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) viene impostata dalla configurazione dell'host durante la costruzione dell'host.

**Chiave**: `applicationName`  
**Tipo**: `string`  
**Impostazione predefinita**: il nome dell'assembly che contiene il punto di ingresso dell'app.  
**Variabile di ambiente**: `<PREFIX_>APPLICATIONNAME`

per impostare questo valore usare la variabile di ambiente. 

### <a name="contentroot"></a>ContentRoot

La proprietà [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) determina il punto in cui l'host inizia a cercare i file di contenuto. Se il percorso non esiste, l'host non verrà avviato.

**Chiave**: `contentRoot`  
**Tipo**: `string`  
**Impostazione predefinita**: la cartella in cui risiede l'assembly dell'app.  
**Variabile di ambiente**: `<PREFIX_>CONTENTROOT`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseContentRoot` su `IHostBuilder`:

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

Per altre informazioni, vedere:

* [Nozioni fondamentali: radice del contenuto](xref:fundamentals/index#content-root)
* [WebRoot](#webroot)

### <a name="environmentname"></a>EnvironmentName

La proprietà [IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) può essere impostata su qualsiasi valore. I valori definiti dal framework includono `Development`, `Staging` e `Production`. I valori non fanno distinzione tra maiuscole e minuscole.

**Chiave**: `environment`  
**Tipo**: `string`  
**Impostazione predefinita**: `Production`  
**Variabile di ambiente**: `<PREFIX_>ENVIRONMENT`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseEnvironment` su `IHostBuilder`:

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a>ShutdownTimeout

[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) imposta il timeout per <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>. Il valore predefinito è cinque secondi.  Durante il periodo di timeout, l'host:

* attiva [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping).
* Tenta di arrestare i servizi ospitati, registrando gli errori per i servizi che non si interrompono.

Se il periodo di timeout scade prima che siano stati arrestati tutti i servizi ospitati, gli eventuali servizi attivi rimanenti vengono interrotti quando l'app viene arrestata. I servizi si arrestano anche se non hanno completato l'elaborazione. Se l'arresto dei servizi richiede più tempo, aumentare il valore di timeout.

**Chiave**: `shutdownTimeoutSeconds`  
**Tipo**: `int`  
**Impostazione predefinita**: 5 secondi  
**Variabile di ambiente**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`

Per impostare questo valore, usare la variabile di ambiente o configurare `HostOptions`. Nell'esempio seguente il timeout viene impostato su 20 secondi:

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

### <a name="disable-app-configuration-reload-on-change"></a>Disabilitare il ricaricamento della configurazione dell'app durante la modifica

Per [impostazione predefinita](xref:fundamentals/configuration/index#default), *appsettings.json* e *appSettings. { L'ambiente}. JSON* viene ricaricato quando il file viene modificato. Per disabilitare questo comportamento di ricaricamento in ASP.NET Core 5,0 o versione successiva, impostare la `hostBuilder:reloadConfigOnChange` chiave su `false` .

**Chiave**: `hostBuilder:reloadConfigOnChange`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: `true`  
**Argomento della riga di comando**: `hostBuilder:reloadConfigOnChange`  
**Variabile di ambiente**: `<PREFIX_>hostBuilder:reloadConfigOnChange`

> [!WARNING]
> Il separatore dei due punti ( `:` ) non funziona con le chiavi gerarchiche delle variabili di ambiente in tutte le piattaforme. Per altre informazioni, vedere [variabili di ambiente](xref:fundamentals/configuration/index#environment-variables).

## <a name="settings-for-web-apps"></a>Impostazioni per le app Web

Alcune impostazioni host si applicano solo ai carichi di lavoro HTTP. Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`.

Per queste impostazioni sono disponibili i metodi di estensione su `IWebHostBuilder`. Gli esempi di codice che illustrano come chiamare i metodi di estensione presumono che `webBuilder` sia un'istanza di `IWebHostBuilder`, come nell'esempio seguente:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a>CaptureStartupErrors

Quando è `false`, gli errori durante l'avvio causano la chiusura dell'host. Quando è `true`, l'host acquisisce le eccezioni durante l'avvio e tenta di avviare il server.

**Chiave**: `captureStartupErrors`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: il valore predefinito è `false` a meno che l'app non venga eseguita con Kestrel in IIS, in tal caso il valore predefinito è `true`.  
**Variabile di ambiente**: `<PREFIX_>CAPTURESTARTUPERRORS`

Per impostare questo valore, usare la configurazione o chiamare `CaptureStartupErrors`:

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a>DetailedErrors

Quando è abilitata o quando l'ambiente è `Development`, l'app acquisisce gli errori dettagliati.

**Chiave**: `detailedErrors`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: `false`  
**Variabile di ambiente**: `<PREFIX_>_DETAILEDERRORS`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a>HostingStartupAssemblies

Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da caricare all'avvio. Sebbene il valore di configurazione predefinito sia una stringa vuota, gli assembly di avvio dell'hosting includono sempre l'assembly dell'app. Quando vengono specificati, gli assembly di avvio dell'hosting vengono aggiunti all'assembly dell'app per essere caricati quando l'app compila i servizi comuni durante l'avvio.

**Chiave**: `hostingStartupAssemblies`  
**Tipo**: `string`  
**Impostazione predefinita**: stringa vuota  
**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a>HostingStartupExcludeAssemblies

Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da escludere all'avvio.

**Chiave**: `hostingStartupExcludeAssemblies`  
**Tipo**: `string`  
**Impostazione predefinita**: stringa vuota  
**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a>HTTPS_Port

Porta di reindirizzamento HTTPS. Usata per [imporre HTTPS](xref:security/enforcing-ssl).

**Chiave**: `https_port`  
**Tipo**: `string`  
**Impostazione predefinita**: un valore predefinito non è impostato.  
**Variabile di ambiente**: `<PREFIX_>HTTPS_PORT`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a>PreferHostingUrls

Indica se l'host deve essere in ascolto sugli URL configurati con `IWebHostBuilder` anziché gli URL configurati con l' `IServer` implementazione di.

**Chiave**: `preferHostingUrls`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: `true`  
**Variabile di ambiente**: `<PREFIX_>_PREFERHOSTINGURLS`

Per impostare questo valore, usare la variabile di ambiente o chiamare `PreferHostingUrls`:

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a>PreventHostingStartup

Impedisce il caricamento automatico degli assembly di avvio dell'hosting, inclusi gli assembly di avvio dell'hosting configurati dall'assembly dell'app. Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.

**Chiave**: `preventHostingStartup`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: `false`  
**Variabile di ambiente**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a>StartupAssembly

L'assembly per la ricerca della classe `Startup`.

**Chiave**: `startupAssembly`  
**Tipo**: `string`  
**Impostazione predefinita**: assembly dell'app  
**Variabile di ambiente**: `<PREFIX_>STARTUPASSEMBLY`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseStartup`. `UseStartup` può richiedere un nome di assembly (`string`) o un tipo (`TStartup`). Se vengono chiamati più metodi `UseStartup`, l'ultimo metodo ha la precedenza.

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a>URL

Elenco delimitato da punto e virgola degli indirizzi IP o gli indirizzi host con le porte e protocolli su cui il server deve eseguire l'ascolto per le richieste. Ad esempio: `http://localhost:123`. Usare "\*" per indicare che il server deve eseguire l'ascolto per le richieste su tutti gli indirizzi IP o nomi host usando la porta e il protocollo specificati (ad esempio, `http://*:5000`). Il protocollo (`http://` o `https://`) deve essere incluso con ogni URL. I formati supportati variano a seconda del server.

**Chiave**: `urls`  
**Tipo**: `string`  
**Impostazione predefinita**: `http://localhost:5000` e `https://localhost:5001`  
**Variabile di ambiente**: `<PREFIX_>URLS`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseUrls`:

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

Kestrel ha una propria API di configurazione degli endpoint. Per altre informazioni, vedere <xref:fundamentals/servers/kestrel/endpoints>.

### <a name="webroot"></a>WebRoot

La proprietà [IWebHostEnvironment. WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) determina il percorso relativo degli asset statici dell'app. Se il percorso non esiste, viene usato un provider di file no-op.  

**Chiave**: `webroot`  
**Tipo**: `string`  
**Impostazione predefinita**: il valore predefinito è `wwwroot` . Il percorso di *{radice del contenuto}/wwwroot* deve esistere.  
**Variabile di ambiente**: `<PREFIX_>WEBROOT`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseWebRoot` su `IWebHostBuilder`:

```csharp
webBuilder.UseWebRoot("public");
```

Per altre informazioni, vedere:

* [Nozioni fondamentali: radice Web](xref:fundamentals/index#web-root)
* [ContentRoot](#contentroot)

## <a name="manage-the-host-lifetime"></a>Gestire la durata dell'host

Chiamare metodi sull'implementazione <xref:Microsoft.Extensions.Hosting.IHost> incorporata per avviare e arrestare l'app. Questi metodi influiscono su tutte le implementazioni <xref:Microsoft.Extensions.Hosting.IHostedService> che vengono registrate nel contenitore dei servizi.

### <a name="run"></a>Esegui

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> esegue l'app e blocca il thread di chiamata fino all'arresto dell'host.

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> esegue l'app e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto.

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*>Abilita il supporto della console, compila e avvia l'host e attende la <kbd></kbd> + chiusura di CTRL<kbd>C</kbd>/SIGINT o SIGTERM.

### <a name="start"></a>Inizia

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> avvia l'host in modo sincrono.

### <a name="startasync"></a>StartAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> avvia l'host e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto. 

<xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> viene chiamato all'avvio di `StartAsync`, che attende il completamento prima di continuare. Questo è utile per ritardare l'avvio fino al segnale trasmesso da un evento esterno.

### <a name="stopasync"></a>StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> prova ad arrestare l'host entro il timeout specificato.

### <a name="waitforshutdown"></a>WaitForShutdown

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*>blocca il thread chiamante finché l'arresto non viene attivato da IHostLifetime, ad esempio tramite <kbd>CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM.

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato l'arresto tramite il token specificato, quindi chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.

### <a name="external-control"></a>Controllo esterno

Il controllo diretto della durata dell'host può essere ottenuto usando metodi che possono essere chiamati dall'esterno:

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

I modelli di ASP.NET Core creano un host generico .NET Core ( <xref:Microsoft.Extensions.Hosting.HostBuilder> ).

In questo argomento vengono fornite informazioni sull'utilizzo di host generico .NET in ASP.NET Core. Per informazioni sull'uso di un host generico .NET nelle app console, vedere [.NET Generic Host](/dotnet/core/extensions/generic-host).

## <a name="host-definition"></a>Definizione host

Un *host* è un oggetto che incapsula le risorse di un'app, ad esempio:

* Inserimento di dipendenze (DI)
* Registrazione
* Configurazione
* `IHostedService` Implementazioni

Quando viene avviato, un host chiama <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> su ogni implementazione di <xref:Microsoft.Extensions.Hosting.IHostedService> registrata nella raccolta di servizi ospitati del contenitore dei servizi. In un'app Web, una delle implementazioni di `IHostedService` è un servizio web che avvia un'[implementazione del server HTTP](xref:fundamentals/index#servers).

Il motivo principale per cui tutte le risorse interdipendenti dell'app sono incluse in un unico oggetto è la gestione del ciclo di vita, vale a dire il controllo sull'avvio dell'app e sull'arresto normale.

## <a name="set-up-a-host"></a>Configurare un host

L'host è in genere configurato, compilato ed eseguito da codice nella classe `Program`. Il metodo `Main`:

* Chiama un metodo `CreateHostBuilder` per creare e configurare un oggetto generatore.
* Chiamate `Build` e metodi `Run` nell'oggetto generatore.

I modelli Web ASP.NET Core generano il codice seguente per creare un host generico:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

Il codice seguente crea un host generico utilizzando un carico di lavoro non HTTP. L' `IHostedService` implementazione viene aggiunta al contenitore DI di:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

Per un carico di lavoro HTTP, il metodo `Main` è lo stesso ma `CreateHostBuilder` chiama `ConfigureWebHostDefaults`:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

Il codice precedente viene generato dai modelli di ASP.NET Core.

Se l'app usa Entity Framework Core, non modificare il nome o la firma del metodo `CreateHostBuilder`. Gli [strumenti di Entity Framework Core](/ef/core/miscellaneous/cli/) si aspettano di trovare un metodo `CreateHostBuilder` che configura l'host senza eseguire l'app. Per altre informazioni, vedere [Creazione DbContext in fase di progettazione](/ef/core/miscellaneous/cli/dbcontext-creation).

## <a name="default-builder-settings"></a>Impostazioni predefinite del generatore

Il metodo <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder%2A>:

* Imposta la [radice del contenuto](xref:fundamentals/index#content-root) sul percorso restituito da <xref:System.IO.Directory.GetCurrentDirectory%2A> .
* Carica la configurazione dell'host da:
  * Variabili di ambiente con prefisso `DOTNET_` .
  * Argomenti della riga di comando.
* Carica la configurazione dell'app da:
  * *appsettings.json*.
  * *appsettings.{Environment}.json*.
  * [Segreti dell'utente](xref:security/app-secrets) quando l'app viene eseguita nell'ambiente `Development`.
  * Variabili di ambiente.
  * Argomenti della riga di comando.
* Aggiunge i provider di [log](xref:fundamentals/logging/index) seguenti:
  * Console
  * Debug
  * EventSource
  * EventLog (solo quando è in esecuzione su Windows)
* Abilita la [convalida dell'ambito](xref:fundamentals/dependency-injection#scope-validation) e la [convalida delle dipendenze](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild) quando l'ambiente è lo sviluppo.

Il metodo `ConfigureWebHostDefaults`:

* Carica la configurazione host dalle variabili di ambiente con prefisso `ASPNETCORE_` .
* Imposta il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web e lo configura usando i provider di configurazione dell'host dell'app. Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel#kestrel-options>.
* Aggiunge [il middleware dell'applicazione di filtri dell'host](xref:fundamentals/servers/kestrel#host-filtering).
* Aggiunge il [middleware delle intestazioni con inoltri](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) se `ASPNETCORE_FORWARDEDHEADERS_ENABLED` è uguale a `true` .
* Abilita l'integrazione di IIS. Per le opzioni predefinite di IIS, vedere <xref:host-and-deploy/iis/index#iis-options>.

Le sezioni [Impostazioni per tutti i tipi di app](#settings-for-all-app-types) e [Impostazioni per le app Web](#settings-for-web-apps) più avanti in questo articolo mostrano come eseguire l'override delle impostazioni predefinite del generatore.

## <a name="framework-provided-services"></a>Servizi forniti dal framework

I servizi seguenti vengono registrati automaticamente:

* [IHostApplicationLifetime](#ihostapplicationlifetime)
* [IHostLifetime](#ihostlifetime)
* [IHostEnvironment / IWebHostEnvironment](#ihostenvironment)

Per ulteriori informazioni sui servizi forniti dal Framework, vedere <xref:fundamentals/dependency-injection#framework-provided-services> .

## <a name="ihostapplicationlifetime"></a>IHostApplicationLifetime

Inserire il servizio <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (in precedenza `IApplicationLifetime`) in qualsiasi classe per gestire le attività post-avvio e di arresto normale. Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi del gestore dell'evento di avvio e di arresto. L'interfaccia include anche un metodo `StopApplication`.

L'esempio seguente è un' `IHostedService` implementazione di che registra `IHostApplicationLifetime` gli eventi:

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a>IHostLifetime

L'implementazione <xref:Microsoft.Extensions.Hosting.IHostLifetime> controlla quando l'host viene avviato e quando si arresta. Viene usata l'ultima implementazione registrata.

`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` è l'implementazione `IHostLifetime` predefinita. `ConsoleLifetime`:

* Ascolta la <kbd>combinazione di tasti CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM e chiama <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication%2A> per avviare il processo di arresto.
* Sblocca estensioni come [RunAsync](#runasync) e [WaitForShutdownAsync](#waitforshutdownasync).

## <a name="ihostenvironment"></a>IHostEnvironment

Inserire il <xref:Microsoft.Extensions.Hosting.IHostEnvironment> servizio in una classe per ottenere informazioni sulle impostazioni seguenti:

* [ApplicationName](#applicationname)
* [EnvironmentName](#environmentname)
* [ContentRootPath](#contentroot)

Le app Web implementano l' `IWebHostEnvironment` interfaccia, che eredita `IHostEnvironment` e aggiunge [WebRootPath](#webroot).

## <a name="host-configuration"></a>Configurazione dell'host

La configurazione dell'host viene usata per le proprietà dell'implementazione <xref:Microsoft.Extensions.Hosting.IHostEnvironment>.

La configurazione dell'host è disponibile da [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) all'interno di <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A>. Dopo `ConfigureAppConfiguration`, `HostBuilderContext.Configuration` viene sostituito con la configurazione dell'app.

Per aggiungere la configurazione dell'host, chiamare <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration%2A> su `IHostBuilder`. `ConfigureHostConfiguration` può essere chiamato più volte e i risultati si sommano. L'host usa l'ultima opzione che imposta un valore in una determinata chiave.

Il provider di variabili di ambiente con prefisso `DOTNET_` e argomenti della riga di comando sono inclusi in `CreateDefaultBuilder` . Per le app Web, viene aggiunto il provider di variabili di ambiente con prefisso `ASPNETCORE_`. Il prefisso viene rimosso quando vengono lette le variabili di ambiente. Ad esempio, il valore della variabile di ambiente per `ASPNETCORE_ENVIRONMENT` diventa il valore di configurazione dell'host per la chiave `environment`.

L'esempio seguente crea la configurazione host:

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a>Configurazione delle app

La configurazione dell'app viene creata chiamando <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> su `IHostBuilder`. `ConfigureAppConfiguration` può essere chiamato più volte e i risultati si sommano. L'app usa l'ultima opzione che imposta un valore in una determinata chiave. 

La configurazione creata da `ConfigureAppConfiguration` è disponibile in [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) per le operazioni successive e come servizio da DI. La configurazione dell'host viene aggiunta anche alla configurazione dell'app.

Per altre informazioni, vedere [Configurazione in ASP.NET Core](xref:fundamentals/configuration/index#configureappconfiguration).

## <a name="settings-for-all-app-types"></a>Impostazioni per tutti i tipi di app

Questa sezione elenca le impostazioni dell'host che si applicano ai carichi di lavoro HTTP e non-HTTP. Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`.

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a>ApplicationName

La proprietà [IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) viene impostata dalla configurazione dell'host durante la costruzione dell'host.

**Chiave**: `applicationName`  
**Tipo**: `string`  
**Impostazione predefinita**: il nome dell'assembly che contiene il punto di ingresso dell'app.  
**Variabile di ambiente**: `<PREFIX_>APPLICATIONNAME`

per impostare questo valore usare la variabile di ambiente. 

### <a name="contentroot"></a>ContentRoot

La proprietà [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) determina il punto in cui l'host inizia a cercare i file di contenuto. Se il percorso non esiste, l'host non verrà avviato.

**Chiave**: `contentRoot`  
**Tipo**: `string`  
**Impostazione predefinita**: la cartella in cui risiede l'assembly dell'app.  
**Variabile di ambiente**: `<PREFIX_>CONTENTROOT`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseContentRoot` su `IHostBuilder`:

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

Per altre informazioni, vedere:

* [Nozioni fondamentali: radice del contenuto](xref:fundamentals/index#content-root)
* [WebRoot](#webroot)

### <a name="environmentname"></a>EnvironmentName

La proprietà [IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) può essere impostata su qualsiasi valore. I valori definiti dal framework includono `Development`, `Staging` e `Production`. I valori non fanno distinzione tra maiuscole e minuscole.

**Chiave**: `environment`  
**Tipo**: `string`  
**Impostazione predefinita**: `Production`  
**Variabile di ambiente**: `<PREFIX_>ENVIRONMENT`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseEnvironment` su `IHostBuilder`:

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a>ShutdownTimeout

[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) imposta il timeout per <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>. Il valore predefinito è cinque secondi.  Durante il periodo di timeout, l'host:

* attiva [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping).
* Tenta di arrestare i servizi ospitati, registrando gli errori per i servizi che non si interrompono.

Se il periodo di timeout scade prima che siano stati arrestati tutti i servizi ospitati, gli eventuali servizi attivi rimanenti vengono interrotti quando l'app viene arrestata. I servizi si arrestano anche se non hanno completato l'elaborazione. Se l'arresto dei servizi richiede più tempo, aumentare il valore di timeout.

**Chiave**: `shutdownTimeoutSeconds`  
**Tipo**: `int`  
**Impostazione predefinita**: 5 secondi  
**Variabile di ambiente**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`

Per impostare questo valore, usare la variabile di ambiente o configurare `HostOptions`. Nell'esempio seguente il timeout viene impostato su 20 secondi:

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

## <a name="settings-for-web-apps"></a>Impostazioni per le app Web

Alcune impostazioni host si applicano solo ai carichi di lavoro HTTP. Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`.

Per queste impostazioni sono disponibili i metodi di estensione su `IWebHostBuilder`. Gli esempi di codice che illustrano come chiamare i metodi di estensione presumono che `webBuilder` sia un'istanza di `IWebHostBuilder`, come nell'esempio seguente:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a>CaptureStartupErrors

Quando è `false`, gli errori durante l'avvio causano la chiusura dell'host. Quando è `true`, l'host acquisisce le eccezioni durante l'avvio e tenta di avviare il server.

**Chiave**: `captureStartupErrors`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: il valore predefinito è `false` a meno che l'app non venga eseguita con Kestrel in IIS, in tal caso il valore predefinito è `true`.  
**Variabile di ambiente**: `<PREFIX_>CAPTURESTARTUPERRORS`

Per impostare questo valore, usare la configurazione o chiamare `CaptureStartupErrors`:

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a>DetailedErrors

Quando è abilitata o quando l'ambiente è `Development`, l'app acquisisce gli errori dettagliati.

**Chiave**: `detailedErrors`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: `false`  
**Variabile di ambiente**: `<PREFIX_>_DETAILEDERRORS`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a>HostingStartupAssemblies

Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da caricare all'avvio. Sebbene il valore di configurazione predefinito sia una stringa vuota, gli assembly di avvio dell'hosting includono sempre l'assembly dell'app. Quando vengono specificati, gli assembly di avvio dell'hosting vengono aggiunti all'assembly dell'app per essere caricati quando l'app compila i servizi comuni durante l'avvio.

**Chiave**: `hostingStartupAssemblies`  
**Tipo**: `string`  
**Impostazione predefinita**: stringa vuota  
**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a>HostingStartupExcludeAssemblies

Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da escludere all'avvio.

**Chiave**: `hostingStartupExcludeAssemblies`  
**Tipo**: `string`  
**Impostazione predefinita**: stringa vuota  
**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a>HTTPS_Port

Porta di reindirizzamento HTTPS. Usata per [imporre HTTPS](xref:security/enforcing-ssl).

**Chiave**: `https_port`  
**Tipo**: `string`  
**Impostazione predefinita**: un valore predefinito non è impostato.  
**Variabile di ambiente**: `<PREFIX_>HTTPS_PORT`

Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a>PreferHostingUrls

Indica se l'host deve essere in ascolto sugli URL configurati con `IWebHostBuilder` anziché gli URL configurati con l' `IServer` implementazione di.

**Chiave**: `preferHostingUrls`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: `true`  
**Variabile di ambiente**: `<PREFIX_>_PREFERHOSTINGURLS`

Per impostare questo valore, usare la variabile di ambiente o chiamare `PreferHostingUrls`:

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a>PreventHostingStartup

Impedisce il caricamento automatico degli assembly di avvio dell'hosting, inclusi gli assembly di avvio dell'hosting configurati dall'assembly dell'app. Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.

**Chiave**: `preventHostingStartup`  
**Tipo**: `bool` ( `true` o `1` )  
**Impostazione predefinita**: `false`  
**Variabile di ambiente**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseSetting`:

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a>StartupAssembly

L'assembly per la ricerca della classe `Startup`.

**Chiave**: `startupAssembly`  
**Tipo**: `string`  
**Impostazione predefinita**: assembly dell'app  
**Variabile di ambiente**: `<PREFIX_>STARTUPASSEMBLY`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseStartup`. `UseStartup` può richiedere un nome di assembly (`string`) o un tipo (`TStartup`). Se vengono chiamati più metodi `UseStartup`, l'ultimo metodo ha la precedenza.

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a>URL

Elenco delimitato da punto e virgola degli indirizzi IP o gli indirizzi host con le porte e protocolli su cui il server deve eseguire l'ascolto per le richieste. Ad esempio: `http://localhost:123`. Usare "\*" per indicare che il server deve eseguire l'ascolto per le richieste su tutti gli indirizzi IP o nomi host usando la porta e il protocollo specificati (ad esempio, `http://*:5000`). Il protocollo (`http://` o `https://`) deve essere incluso con ogni URL. I formati supportati variano a seconda del server.

**Chiave**: `urls`  
**Tipo**: `string`  
**Impostazione predefinita**: `http://localhost:5000` e `https://localhost:5001`  
**Variabile di ambiente**: `<PREFIX_>URLS`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseUrls`:

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

Kestrel ha una propria API di configurazione degli endpoint. Per altre informazioni, vedere <xref:fundamentals/servers/kestrel#endpoint-configuration>.

### <a name="webroot"></a>WebRoot

La proprietà [IWebHostEnvironment. WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) determina il percorso relativo degli asset statici dell'app. Se il percorso non esiste, viene usato un provider di file no-op.  

**Chiave**: `webroot`  
**Tipo**: `string`  
**Impostazione predefinita**: il valore predefinito è `wwwroot` . Il percorso di *{radice del contenuto}/wwwroot* deve esistere.  
**Variabile di ambiente**: `<PREFIX_>WEBROOT`

Per impostare questo valore, usare la variabile di ambiente o chiamare `UseWebRoot` su `IWebHostBuilder`:

```csharp
webBuilder.UseWebRoot("public");
```

Per altre informazioni, vedere:

* [Nozioni fondamentali: radice Web](xref:fundamentals/index#web-root)
* [ContentRoot](#contentroot)

## <a name="manage-the-host-lifetime"></a>Gestire la durata dell'host

Chiamare metodi sull'implementazione <xref:Microsoft.Extensions.Hosting.IHost> incorporata per avviare e arrestare l'app. Questi metodi influiscono su tutte le implementazioni <xref:Microsoft.Extensions.Hosting.IHostedService> che vengono registrate nel contenitore dei servizi.

### <a name="run"></a>Esegui

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> esegue l'app e blocca il thread di chiamata fino all'arresto dell'host.

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> esegue l'app e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto.

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*>Abilita il supporto della console, compila e avvia l'host e attende la <kbd></kbd> + chiusura di CTRL<kbd>C</kbd>/SIGINT o SIGTERM.

### <a name="start"></a>Inizia

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> avvia l'host in modo sincrono.

### <a name="startasync"></a>StartAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> avvia l'host e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto. 

<xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> viene chiamato all'avvio di `StartAsync`, che attende il completamento prima di continuare. Questo è utile per ritardare l'avvio fino al segnale trasmesso da un evento esterno.

### <a name="stopasync"></a>StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> prova ad arrestare l'host entro il timeout specificato.

### <a name="waitforshutdown"></a>WaitForShutdown

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*>blocca il thread chiamante finché l'arresto non viene attivato da IHostLifetime, ad esempio tramite <kbd>CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM.

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato l'arresto tramite il token specificato, quindi chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.

### <a name="external-control"></a>Controllo esterno

Il controllo diretto della durata dell'host può essere ottenuto usando metodi che possono essere chiamati dall'esterno:

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Le app ASP.NET Core configurano e avviano un host. L'host è responsabile della gestione dell'avvio e della durata delle app.

Questo articolo illustra l'host generico di ASP.NET Core (<xref:Microsoft.Extensions.Hosting.HostBuilder>), che si usa per le app che non elaborano le richieste HTTP.

Lo scopo dell'host generico è separare la pipeline HTTP dall'API dell'host Web, per abilitare una gamma più ampia di scenari host. La messaggistica, le attività in background e altri carichi di lavoro non HTTP basati sull'host generico traggono vantaggio dalle funzionalità a montaggio incrociato come la configurazione, l'inserimento di dipendenze (DI) e la registrazione.

L'host generico è una nuova funzionalità introdotta in ASP.NET Core 2.1 e non è adatto agli scenari di hosting Web. Per gli scenari di hosting Web usare l'[host Web](xref:fundamentals/host/web-host). L'host generico sostituirà l'host Web in una versione futura, funzionando come API host principale negli scenari sia HTTP che non HTTP.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

Quando si esegue l'app di esempio in [Visual Studio Code](https://code.visualstudio.com/), usare un *terminale esterno o integrato*. Non eseguire l'esempio in un ambiente `internalConsole`.

Per impostare la console in Visual Studio Code:

1. Aprire il file *.vscode/launch.json*.
1. Nella configurazione **.NET Core Launch (console)** trovare la voce **console**. Impostare il valore su `externalTerminal` o `integratedTerminal`.

## <a name="introduction"></a>Introduzione

La libreria host generico è disponibile nello spazio dei nomi <xref:Microsoft.Extensions.Hosting> e viene specificata dal pacchetto [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting/). Il pacchetto `Microsoft.Extensions.Hosting` è incluso nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) (ASP.NET Core 2.1 o versioni successive).

<xref:Microsoft.Extensions.Hosting.IHostedService> è il punto di ingresso per l'esecuzione del codice. Ogni implementazione <xref:Microsoft.Extensions.Hosting.IHostedService> viene eseguita nell'ordine di [registrazione del servizio in ConfigureServices](#configureservices). <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*> viene chiamato su ogni <xref:Microsoft.Extensions.Hosting.IHostedService> quando viene avviato l'host e <xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*> viene chiamato in ordine di registrazione inverso quando l'host viene chiuso normalmente.

## <a name="set-up-a-host"></a>Configurare un host

<xref:Microsoft.Extensions.Hosting.IHostBuilder> è il componente principale che le librerie e le app usano per inizializzare, compilare ed eseguire l'host:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_HostBuilder)]

## <a name="options"></a>Opzioni

<xref:Microsoft.Extensions.Hosting.HostOptions> configura le opzioni per <xref:Microsoft.Extensions.Hosting.IHost>.

### <a name="shutdown-timeout"></a>Timeout di arresto

<xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> imposta il timeout per <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>. Il valore predefinito è cinque secondi.

La seguente configurazione di opzione in `Program.Main` aumenta il timeout predefinito di cinque secondi di chiusura a 20 secondi:

```csharp
var host = new HostBuilder()
    .ConfigureServices((hostContext, services) =>
    {
        services.Configure<HostOptions>(option =>
        {
            option.ShutdownTimeout = System.TimeSpan.FromSeconds(20);
        });
    })
    .Build();
```

## <a name="default-services"></a>Servizi predefiniti

I servizi seguenti vengono registrati durante l'inizializzazione dell'host:

* [Ambiente](xref:fundamentals/environments) ( <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> )
* <xref:Microsoft.Extensions.Hosting.HostBuilderContext>
* [Configurazione](xref:fundamentals/configuration/index) ( <xref:Microsoft.Extensions.Configuration.IConfiguration> )
* <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> (`Microsoft.Extensions.Hosting.Internal.ApplicationLifetime`)
* <xref:Microsoft.Extensions.Hosting.IHostLifetime> (`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime`)
* <xref:Microsoft.Extensions.Hosting.IHost>
* [Opzioni](xref:fundamentals/configuration/options) ( <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions*> )
* [Registrazione](xref:fundamentals/logging/index) ( <xref:Microsoft.Extensions.DependencyInjection.LoggingServiceCollectionExtensions.AddLogging*> )

## <a name="host-configuration"></a>Configurazione dell'host

La configurazione dell'host viene creata tramite:

* Chiamata ai metodi di estensione su <xref:Microsoft.Extensions.Hosting.IHostBuilder> per impostare la [radice del contenuto](#content-root) e l'[ambiente](#environment).
* Lettura della configurazione dai provider di configurazione in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>.

### <a name="extension-methods"></a>Metodi di estensione

### <a name="application-key-name"></a>Chiave applicazione (nome)

La proprietà [IHostingEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ApplicationName*) viene impostata dalla configurazione dell'host durante la costruzione dell'host. Per impostare il valore in modo esplicito, usare [HostDefaults.ApplicationKey](xref:Microsoft.Extensions.Hosting.HostDefaults.ApplicationKey):

**Chiave**: `applicationName`  
**Tipo**: `string`  
**Impostazione predefinita**: il nome dell'assembly contenente il punto di ingresso dell'app.  
**Imposta utilizzando**: `HostBuilderContext.HostingEnvironment.ApplicationName`  
**Variabile di ambiente**: `<PREFIX_>APPLICATIONNAME` ( `<PREFIX_>` è [facoltativo e definito dall'utente](#configurehostconfiguration))

### <a name="content-root"></a>Radice del contenuto

Questa impostazione determina la posizione da cui l'host inizia la ricerca dei file di contenuto.

**Chiave**: `contentRoot`  
**Tipo**: `string`  
**Impostazione predefinita**: il valore predefinito corrisponde alla cartella contenente l'assembly dell'app.  
**Imposta utilizzando**: `UseContentRoot`  
**Variabile di ambiente**: `<PREFIX_>CONTENTROOT` ( `<PREFIX_>` è [facoltativo e definito dall'utente](#configurehostconfiguration))

Se il percorso non esiste, l'host non verrà avviato.

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseContentRoot)]

Per altre informazioni, vedere [nozioni fondamentali: radice del contenuto](xref:fundamentals/index#content-root).

### <a name="environment"></a>Ambiente

Imposta l' [ambiente](xref:fundamentals/environments)dell'app.

**Chiave**: `environment`  
**Tipo**: `string`  
**Impostazione predefinita**: `Production`  
**Imposta utilizzando**: `UseEnvironment`  
**Variabile di ambiente**: `<PREFIX_>ENVIRONMENT` ( `<PREFIX_>` è [facoltativo e definito dall'utente](#configurehostconfiguration))

L'ambiente può essere impostato su qualsiasi valore. I valori definiti dal framework includono `Development`, `Staging` e `Production`. I valori non fanno distinzione tra maiuscole e minuscole.

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseEnvironment)]

### <a name="configurehostconfiguration"></a>ConfigureHostConfiguration

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> usa un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per creare un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfiguration> per l'host. La configurazione dell'host viene usata per inizializzare l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> da usare nel processo di compilazione dell'app.

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> può essere chiamato più volte e i risultati si sommano. L'host usa l'ultima opzione che imposta un valore in una determinata chiave.

Nessun provider è incluso per impostazione predefinita. È necessario specificare in modo esplicito tutti i provider di configurazione richiesti dall'app in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>, inclusi:

* Configurazione dei file (ad esempio, da un file *hostsettings.json*).
* Configurazione delle variabili di ambiente.
* Configurazione degli argomenti della riga di comando.
* Tutti gli altri provider di configurazione necessari.

La configurazione file dell'host viene abilitata specificando il percorso di base dell'app con `SetBasePath` seguito da una chiamata a uno dei [provider di configurazione dei file](xref:fundamentals/configuration/index#file-configuration-provider). L'app di esempio usa un file JSON, *hostsettings.json*, e chiama <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> per utilizzare le impostazioni di configurazione host del file.

Per aggiungere la [configurazione delle variabili di ambiente](xref:fundamentals/configuration/index#environment-variables-configuration-provider) dell'host, chiamare <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> sul generatore di host. `AddEnvironmentVariables` accetta un prefisso facoltativo definito dall'utente. L'app di esempio usa un prefisso di `PREFIX_`. Il prefisso viene rimosso quando vengono lette le variabili di ambiente. Quando viene configurato l'host dell'app di esempio, il valore della variabile di ambiente per `PREFIX_ENVIRONMENT` diventa il valore di configurazione dell'host per la chiave `environment`.

Durante lo sviluppo quando si usa [Visual Studio](https://visualstudio.microsoft.com) o si esegue un'app con `dotnet run`, le variabili di ambiente possono essere impostate nel file *Properties/launchSettings.json*. Durante lo sviluppo in [Visual Studio Code](https://code.visualstudio.com/), le variabili di ambiente possono essere impostate nel file *.vscode/launch.json*. Per altre informazioni, vedere <xref:fundamentals/environments>.

La [configurazione della riga di comando](xref:fundamentals/configuration/index#command-line-configuration-provider) viene aggiunta chiamando <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*>. La configurazione della riga di comando viene aggiunta per ultima per consentire agli argomenti della riga di comando di eseguire l'override della configurazione fornita dai provider di configurazione precedenti.

*hostsettings.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/hostsettings.json)]

È possibile fornire una configurazione aggiuntiva con le chiavi [applicationName](#application-key-name) e [contentRoot](#content-root).

Configurazione di esempio di `HostBuilder` che usa <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureHostConfiguration)]

## <a name="configureappconfiguration"></a>ConfigureAppConfiguration

La configurazione dell'app viene creata chiamando <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> sull'implementazione <xref:Microsoft.Extensions.Hosting.IHostBuilder>. <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> usa un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per creare un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfiguration> per l'app. <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> può essere chiamato più volte e i risultati si sommano. L'app usa l'ultima opzione che imposta un valore in una determinata chiave. La configurazione creata da <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> è disponibile in [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) per le operazioni successive e in <xref:Microsoft.Extensions.Hosting.IHost.Services*>.

La configurazione dell'app riceve automaticamente la configurazione dell'host fornita da [ConfigureHostConfiguration](#configurehostconfiguration).

Configurazione app di esempio che usa <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureAppConfiguration)]

*appsettings.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.json)]

*appsettings.Development.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Development.json)]

*appsettings.Production.json*:

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Production.json)]

Per spostare i file delle impostazioni nella directory di output, specificare i file delle impostazioni come [elementi di progetto MSBuild](/visualstudio/msbuild/common-msbuild-project-items) nel file di progetto. L'app di esempio sposta i file delle impostazioni dell'app JSON e *hostsettings.json* con l'elemento `<Content>` seguente:

```xml
<ItemGroup>
  <Content Include="**\*.json" Exclude="bin\**\*;obj\**\*" 
      CopyToOutputDirectory="PreserveNewest" />
</ItemGroup>
```

> [!NOTE]
> I metodi di estensione di configurazione, ad esempio <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> e <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*>, necessitano di altri pacchetti NuGet, ad esempio [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) e [ Microsoft.Extensions.Configuration.EnvironmentVariables](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.EnvironmentVariables). A meno che l'app non usi il [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), questi pacchetti devono essere aggiunti al progetto oltre al pacchetto [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) di base. Per altre informazioni, vedere <xref:fundamentals/configuration/index>.

## <a name="configureservices"></a>ConfigureServices

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> aggiunge servizi al contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) dell'app. <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> può essere chiamato più volte e i risultati si sommano.

Un servizio ospitato è una classe con logica di attività in background che implementa l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>. Per altre informazioni, vedere <xref:fundamentals/host/hosted-services>.

L'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) usa il metodo di estensione `AddHostedService` per aggiungere all'app un servizio per gli eventi di durata (`LifetimeEventsHostedService`) e un'attività in background con timeout (`TimedHostedService`):

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureServices)]

## <a name="configurelogging"></a>ConfigureLogging

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> aggiunge un delegato per configurare l'interfaccia <xref:Microsoft.Extensions.Logging.ILoggingBuilder> fornita. <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> può essere chiamato più volte e i risultati si sommano.

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureLogging)]

### <a name="useconsolelifetime"></a>UseConsoleLifetime

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*>Ascolta la <kbd>combinazione di tasti CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM e chiama <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> per avviare il processo di arresto. <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> Sblocca le estensioni, ad esempio [RunAsync](#runasync) e [WaitForShutdownAsync](#waitforshutdownasync). `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` è preregistrata come implementazione di durata predefinita. Viene usata l'ultima durata registrata.

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseConsoleLifetime)]

## <a name="container-configuration"></a>Configurazione contenitore

Per supportare l'inserimento di altri contenitori, l'host può accettare un elemento <xref:Microsoft.Extensions.DependencyInjection.IServiceProviderFactory%601>. La disponibilità di una factory non fa parte della registrazione del contenitore DI, ma è una funzionalità intrinseca dell'host usata per creare il contenitore DI concreto. [UseServiceProviderFactory(IServiceProviderFactory&lt;TContainerBuilder&gt;)](xref:Microsoft.Extensions.Hosting.HostBuilder.UseServiceProviderFactory*) esegue l'override della factory predefinita usata per la creazione del provider di servizi dell'app.

La configurazione del contenitore personalizzato è gestita dal metodo <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*>. <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> offre un'esperienza fortemente tipizzata per la configurazione del contenitore sulla base dell'API host sottostante. <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> può essere chiamato più volte e i risultati si sommano.

Creare un contenitore di servizi per l'app:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainer.cs)]

Specificare una factory per il contenitore di servizi:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainerFactory.cs)]

Usare la factory e configurare il contenitore di servizi personalizzato per l'app:

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ContainerConfiguration)]

## <a name="extensibility"></a>Estendibilità

L'estensibilità host viene eseguita con i metodi di estensione su <xref:Microsoft.Extensions.Hosting.IHostBuilder>. L'esempio seguente illustra come un metodo di estensione estende un'implementazione <xref:Microsoft.Extensions.Hosting.IHostBuilder> con l'esempio [TimedHostedService](xref:fundamentals/host/hosted-services#timed-background-tasks) dimostrato in <xref:fundamentals/host/hosted-services>.

```csharp
var host = new HostBuilder()
    .UseHostedService<TimedHostedService>()
    .Build();

await host.StartAsync();
```

Un'app stabilisce il metodo di estensione `UseHostedService` per registrare il servizio ospitato passato in `T`:

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public static class Extensions
{
    public static IHostBuilder UseHostedService<T>(this IHostBuilder hostBuilder)
        where T : class, IHostedService, IDisposable
    {
        return hostBuilder.ConfigureServices(services =>
            services.AddHostedService<T>());
    }
}
```

## <a name="manage-the-host"></a>Gestire l'host

L'implementazione <xref:Microsoft.Extensions.Hosting.IHost> è responsabile dell'avvio e arresto delle implementazioni <xref:Microsoft.Extensions.Hosting.IHostedService> che sono registrate nel contenitore di servizi.

### <a name="run"></a>Esegui

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> esegue l'app e blocca il thread di chiamata fino all'arresto dell'host:

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        host.Run();
    }
}
```

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> esegue l'app e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto:

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        await host.RunAsync();
    }
}
```

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*>Abilita il supporto della console, compila e avvia l'host e attende la <kbd></kbd> + chiusura di CTRL<kbd>C</kbd>/SIGINT o SIGTERM.

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var hostBuilder = new HostBuilder();

        await hostBuilder.RunConsoleAsync();
    }
}
```

### <a name="start-and-stopasync"></a>Start e StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> avvia l'host in modo sincrono.

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> prova ad arrestare l'host entro il timeout specificato.

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            await host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

### <a name="startasync-and-stopasync"></a>StartAsync e StopAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> avvia l'app.

<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> arresta l'app.

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.StopAsync();
        }
    }
}
```

### <a name="waitforshutdown"></a>WaitForShutdown

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*>viene attivato tramite <xref:Microsoft.Extensions.Hosting.IHostLifetime> , ad esempio (in `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` attesa di <kbd>CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM). <xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            host.WaitForShutdown();
        }
    }
}
```

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato l'arresto tramite il token specificato, quindi chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.WaitForShutdownAsync();
        }

    }
}
```

### <a name="external-control"></a>Controllo esterno

Il controllo esterno dell'host può essere ottenuto usando metodi che possono essere chiamati dall'esterno:

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

<xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> viene chiamato all'avvio di <xref:Microsoft.Extensions.Hosting.IHost.StartAsync*>, che attende il completamento prima di continuare. Questo è utile per ritardare l'avvio fino al segnale trasmesso da un evento esterno.

## <a name="ihostingenvironment-interface"></a>Interfaccia IHostingEnvironment

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment> include informazioni sull'ambiente di hosting dell'app. Usare l'[inserimento di un costruttore](xref:fundamentals/dependency-injection) per ottenere <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> per poterne usare le proprietà e i metodi di estensione:

```csharp
public class MyClass
{
    private readonly IHostingEnvironment _env;

    public MyClass(IHostingEnvironment env)
    {
        _env = env;
    }

    public void DoSomething()
    {
        var environmentName = _env.EnvironmentName;
    }
}
```

Per altre informazioni, vedere <xref:fundamentals/environments>.

## <a name="iapplicationlifetime-interface"></a>Interfaccia IApplicationLifetime

<xref:Microsoft.Extensions.Hosting.IApplicationLifetime> consente attività post-avvio e di arresto, incluse le richieste di arresto normale. Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi <xref:System.Action> che definiscono gli eventi di avvio e arresto.

| Token di annullamento | Attivato quando&#8230; |
| ------------------ | --------------------- |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStarted*> | L'host è stato completamente avviato. |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopped*> | L'host sta completando un arresto normale. Tutte le richieste devono essere elaborate. L'arresto è bloccato fino al completamento di questo evento. |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopping*> | L'host sta eseguendo un arresto normale. È possibile che le richieste siano ancora in esecuzione. L'arresto è bloccato fino al completamento di questo evento. |

Inserire tramite costruttore il servizio <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> in qualsiasi classe. L'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) usa l'inserimento di costruttori in una classe `LifetimeEventsHostedService` (implementazione <xref:Microsoft.Extensions.Hosting.IHostedService>) per registrare gli eventi.

*LifetimeEventsHostedService.cs*:

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/LifetimeEventsHostedService.cs?name=snippet1)]

<xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> richiede la chiusura dell'applicazione corrente. La classe seguente usa <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> per arrestare normalmente un'app quando viene chiamato il metodo `Shutdown` della classe:

```csharp
public class MyClass
{
    private readonly IApplicationLifetime _appLifetime;

    public MyClass(IApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public void Shutdown()
    {
        _appLifetime.StopApplication();
    }
}
```

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/host/hosted-services>
