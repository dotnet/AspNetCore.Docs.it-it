---
title: Host Web ASP.NET Core
author: rick-anderson
description: Informazioni sull'host Web in ASP.NET Core, responsabile della gestione dell'avvio e della durata delle app.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/07/2019
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
uid: fundamentals/host/web-host
ms.openlocfilehash: fa9b1941d6dcda30855a4729dfa1cd78f897d9b6
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109975"
---
# <a name="aspnet-core-web-host"></a>Host Web ASP.NET Core

ASP.NET Core le app configurano e avviano un *host*. L'host è responsabile della gestione dell'avvio e della durata delle app. L'host configura almeno un server e una pipeline di elaborazione delle richieste. L'host può anche configurare la registrazione, l'inserimento delle dipendenze e la configurazione.

::: moniker range=">= aspnetcore-3.0"

Questo articolo descrive l'host Web, che rimane disponibile solo per compatibilità con le versioni precedenti. I modelli di ASP.NET Core creano un [host generico .NET](<xref:fundamentals/host/generic-host>), consigliato per tutti i tipi di app.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Questo articolo descrive l'host Web destinato all'hosting di app Web. Per altri tipi di app, usare l'[host generico](xref:fundamentals/host/generic-host).

::: moniker-end

## <a name="set-up-a-host"></a>Configurare un host

Creare un host usando un'istanza di [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder). Questa operazione viene in genere eseguita nel punto di ingresso dell'app, il metodo `Main`.

Nei modelli di progetto `Main` si trova in *Program.cs*. Un'app tipica chiama [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) per avviare la configurazione di un host:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

Il codice che chiama `CreateDefaultBuilder` è incluso in un metodo denominato `CreateWebHostBuilder`, che lo separa dal codice in `Main` che chiama `Run` per l'oggetto generatore. Questa separazione è necessaria se si usano gli [strumenti di Entity Framework Core](/ef/core/miscellaneous/cli/). Gli strumenti si aspettano di trovare un metodo `CreateWebHostBuilder` che possono chiamare in fase di progettazione per configurare l'host senza eseguire l'app. Un'alternativa consiste nell'implementare `IDesignTimeDbContextFactory`. Per altre informazioni, vedere [Creazione DbContext in fase di progettazione](/ef/core/miscellaneous/cli/dbcontext-creation).

`CreateDefaultBuilder` esegue queste attività:

::: moniker range=">= aspnetcore-5.0"
* Configura il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web usando i provider di configurazione dell'host dell'app. Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel/options>.
::: moniker-end
::: moniker range="< aspnetcore-5.0"
* Configura il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web usando i provider di configurazione dell'host dell'app. Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel#kestrel-options>.
::: moniker-end
* Imposta la [radice del contenuto](xref:fundamentals/index#content-root) sul percorso restituito da [Directory. GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory).
* Carica la [configurazione dell'host](#host-configuration-values) da:
  * Le variabili di ambiente con prefisso `ASPNETCORE_` (ad esempio, `ASPNETCORE_ENVIRONMENT`).
  * Argomenti della riga di comando.
* Carica la configurazione dell'app nell'ordine seguente da:
  * *appsettings.json*.
  * *appsettings.{Environment}.json*.
  * [Segreti dell'utente](xref:security/app-secrets) quando l'app viene eseguita nell'ambiente `Development` usando l'assembly di ingresso.
  * Variabili di ambiente.
  * Argomenti della riga di comando.
* Configura la [registrazione](xref:fundamentals/logging/index) per l'output della console e del debug. La registrazione include le regole di [filtro dei log](xref:fundamentals/logging/index#log-filtering) specificate in una sezione di configurazione di registrazione di un oggetto *appsettings.json* o *appSettings. { Environment}. file JSON* .
* Quando è in esecuzione dietro IIS con il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), `CreateDefaultBuilder` Abilita l' [integrazione di IIS](xref:host-and-deploy/iis/index), che configura l'indirizzo di base e la porta dell'app. L'integrazione di IIS configura inoltre l'app per l'[acquisizione degli errori di avvio](#capture-startup-errors). Per le opzioni predefinite di IIS, vedere <xref:host-and-deploy/iis/index#iis-options>.
* Imposta [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) to `true` se l'ambiente dell'app è lo sviluppo. Per ulteriori informazioni, vedere [Convalida dell'ambito](#scope-validation).

La configurazione definita da `CreateDefaultBuilder` può essere sottoposta a override e aumentata tramite [ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration), [ConfigureLogging](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging) e altri metodi e metodi di estensione di [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder). Di seguito sono riportati alcuni esempi:

* [ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration) viene usato per specificare altri elementi `IConfiguration` per l'app. La chiamata a `ConfigureAppConfiguration` seguente consente di aggiungere un delegato per includere la configurazione dell'app nel file *appsettings.xml*. `ConfigureAppConfiguration` può essere chiamato più volte. Si noti che questa configurazione non è valida per l'host (ad esempio, gli URL del server o l'ambiente). Vedere la sezione [Valori di configurazione dell'host](#host-configuration-values).

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((hostingContext, config) =>
        {
            config.AddXmlFile("appsettings.xml", optional: true, reloadOnChange: true);
        })
        ...
    ```

* La chiamata a `ConfigureLogging` seguente consente di aggiungere un delegato per configurare il livello di registrazione minimo ([SetMinimumLevel](/dotnet/api/microsoft.extensions.logging.loggingbuilderextensions.setminimumlevel)) [LogLevel.Warning](/dotnet/api/microsoft.extensions.logging.loglevel). Questa impostazione sostituisce le impostazioni *appsettings.Development.json* ( `LogLevel.Debug` ) e *appsettings.Production.json* ( `LogLevel.Error` ) configurate da `CreateDefaultBuilder` . `ConfigureLogging` può essere chiamato più volte.

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureLogging(logging => 
        {
            logging.SetMinimumLevel(LogLevel.Warning);
        })
        ...
    ```

::: moniker range=">= aspnetcore-2.2"

* La chiamata a `ConfigureKestrel` seguente sostituisce il valore predefinito [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) di 30.000.000 di byte stabilito durante la configurazione del Kestrel da `CreateDefaultBuilder`:

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureKestrel((context, options) =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
    ```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* La chiamata a [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel) seguente sostituisce il valore predefinito [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) di 30.000.000 di byte stabilito durante la configurazione del Kestrel da `CreateDefaultBuilder`:

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .UseKestrel(options =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
    ```

::: moniker-end

La [radice del contenuto](xref:fundamentals/index#content-root) determina la posizione in cui l'host cerca i file dei contenuti, ad esempio i file di visualizzazione MVC. Quando l'app viene avviata dalla cartella radice del progetto, la cartella radice del progetto viene usata come radice del contenuto. Si tratta dell'impostazione predefinita usata in [Visual Studio](https://visualstudio.microsoft.com) e nei [nuovi modelli dotnet](/dotnet/core/tools/dotnet-new).

Per altre informazioni sulla configurazione dell'app, vedere <xref:fundamentals/configuration/index>.

> [!NOTE]
> In alternativa all'uso del metodo `CreateDefaultBuilder` statico, un approccio supportato in ASP.NET Core 2.x consiste nel creare un host da [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder).

Quando si configura un host, è possibile specificare i metodi [Configure](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure) e [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureservices). Se viene specificata una classe `Startup`, la classe deve definire un metodo `Configure`. Per altre informazioni, vedere <xref:fundamentals/startup>. Se vengono effettuate più chiamate a `ConfigureServices`, le chiamate vengono aggiunte l'una all'altra. Più chiamate a `Configure` o `UseStartup` in `WebHostBuilder` sostituiscono le impostazioni precedenti.

## <a name="host-configuration-values"></a>Valori di configurazione dell'host

[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) si basa sugli approcci seguenti per impostare i valori di configurazione dell'host:

* Configurazione del generatore di host, che include le variabili di ambiente nel formato `ASPNETCORE_{configurationKey}`. Ad esempio: `ASPNETCORE_ENVIRONMENT`.
* Le estensioni come [UseContentRoot](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.usecontentroot) e [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) (vedere la sezione [Override della configurazione](#override-configuration)).
* [UseSetting](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.usesetting) e chiave associata. Quando si imposta un valore con `UseSetting`, il valore viene impostato come stringa indipendentemente dal tipo.

L'host usa l'ultima opzione che imposta un valore. Per altre informazioni, vedere [Override della configurazione](#override-configuration) nella prossima sezione.

### <a name="application-key-name"></a>Chiave applicazione (nome)

::: moniker range=">= aspnetcore-3.0"

La `IWebHostEnvironment.ApplicationName` proprietà viene impostata automaticamente quando [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) o [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) viene chiamato durante la costruzione dell'host. Il valore viene impostato sul nome dell'assembly contenente il punto di ingresso dell'app. Per impostare il valore in modo esplicito, usare [WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey):

::: moniker-end

::: moniker range="< aspnetcore-3.0"

La proprietà [IHostingEnvironment.ApplicationName](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.applicationname) viene impostata automaticamente quando si chiama [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) o [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) durante la costruzione dell'host. Il valore viene impostato sul nome dell'assembly contenente il punto di ingresso dell'app. Per impostare il valore in modo esplicito, usare [WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey):

::: moniker-end

**Chiave**: applicationName  
**Tipo**: *stringa*  
**Impostazione predefinita**: il nome dell'assembly contenente il punto di ingresso dell'app.  
**Imposta utilizzando**: `UseSetting`  
**Variabile di ambiente**: `ASPNETCORE_APPLICATIONNAME`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.ApplicationKey, "CustomApplicationName")
```

### <a name="capture-startup-errors"></a>Acquisizione degli errori di avvio

Questa impostazione controlla l'acquisizione degli errori di avvio.

**Chiave**: captureStartupErrors  
**Tipo**: *bool* (`true` o `1`)  
**Impostazione predefinita**: il valore predefinito è `false` a meno che l'app non venga eseguita con Kestrel in IIS, in tal caso il valore predefinito è `true`.  
**Imposta utilizzando**: `CaptureStartupErrors`  
**Variabile di ambiente**: `ASPNETCORE_CAPTURESTARTUPERRORS`

Quando è `false`, gli errori durante l'avvio causano la chiusura dell'host. Quando è `true`, l'host acquisisce le eccezioni durante l'avvio e tenta di avviare il server.

```csharp
WebHost.CreateDefaultBuilder(args)
    .CaptureStartupErrors(true)
```

### <a name="content-root"></a>Radice del contenuto

Questa impostazione determina la posizione in cui ASP.NET Core inizia la ricerca dei file di contenuto.

**Chiave**: contentRoot  
**Tipo**: *stringa*  
**Impostazione predefinita**: il valore predefinito corrisponde alla cartella contenente l'assembly dell'app.  
**Imposta utilizzando**: `UseContentRoot`  
**Variabile di ambiente**: `ASPNETCORE_CONTENTROOT`

La radice del contenuto viene usata anche come percorso di base per la [radice Web](xref:fundamentals/index#web-root). Se il percorso radice del contenuto non esiste, l'host non viene avviato.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\<content-root>")
```

Per altre informazioni, vedere:

* [Nozioni fondamentali: radice del contenuto](xref:fundamentals/index#content-root)
* [Radice Web](#web-root)

### <a name="detailed-errors"></a>Errori dettagliati

Determina l'acquisizione degli errori dettagliati.

**Chiave**: detailedErrors  
**Tipo**: *bool* (`true` o `1`)  
**Valore predefinito**: false  
**Imposta utilizzando**: `UseSetting`  
**Variabile di ambiente**: `ASPNETCORE_DETAILEDERRORS`

Quando è abilitata (o quando l'<a href="#environment">ambiente</a> è impostato su `Development`), l'app acquisisce le eccezioni dettagliate.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
```

### <a name="environment"></a>Ambiente

Imposta l'ambiente dell'app.

**Chiave**: environment  
**Tipo**: *stringa*  
**Impostazione predefinita**: Production  
**Imposta utilizzando**: `UseEnvironment`  
**Variabile di ambiente**: `ASPNETCORE_ENVIRONMENT`

L'ambiente può essere impostato su qualsiasi valore. I valori definiti dal framework includono `Development`, `Staging` e `Production`. Nei valori non viene fatta distinzione tra maiuscole e minuscole. Per impostazione predefinita, l'*ambiente* viene letto dalla variabile di ambiente `ASPNETCORE_ENVIRONMENT`. Quando si usa [Visual Studio](https://visualstudio.microsoft.com), le variabili di ambiente possono essere impostate nel file *launchSettings.json*. Per altre informazioni, vedere <xref:fundamentals/environments>.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseEnvironment(EnvironmentName.Development)
```

### <a name="hosting-startup-assemblies"></a>Assembly di avvio dell'hosting

Imposta gli assembly di avvio dell'hosting dell'app.

**Chiave**: hostingStartupAssemblies  
**Tipo**: *stringa*  
**Impostazione predefinita**: stringa vuota  
**Imposta utilizzando**: `UseSetting`  
**Variabile di ambiente**: `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`

Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da caricare all'avvio.

Sebbene il valore di configurazione predefinito sia una stringa vuota, gli assembly di avvio dell'hosting includono sempre l'assembly dell'app. Quando vengono specificati, gli assembly di avvio dell'hosting vengono aggiunti all'assembly dell'app per essere caricati quando l'app compila i servizi comuni durante l'avvio.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2")
```

### <a name="https-port"></a>Porta HTTPS

Impostare la porta di reindirizzamento HTTPS. Usata per [imporre HTTPS](xref:security/enforcing-ssl).

**Chiave**: https_port **Tipo**: *string*
**Valore predefinito**: non è impostato un valore predefinito.
**Imposta utilizzando**: `UseSetting` 
 **variabile di ambiente**:`ASPNETCORE_HTTPS_PORT`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting("https_port", "8080")
```

### <a name="hosting-startup-exclude-assemblies"></a>Assembly di avvio dell'hosting da escludere

Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da escludere all'avvio.

**Chiave**: hostingStartupExcludeAssemblies  
**Tipo**: *stringa*  
**Impostazione predefinita**: stringa vuota  
**Imposta utilizzando**: `UseSetting`  
**Variabile di ambiente**: `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2")
```

### <a name="prefer-hosting-urls"></a>Preferire gli URL di hosting

Indica se l'host deve eseguire l'ascolto sugli URL configurati con `WebHostBuilder` anziché su quelli configurati con l'implementazione `IServer`.

**Chiave**: preferHostingUrls  
**Tipo**: *bool* (`true` o `1`)  
**Impostazione predefinita**: true  
**Imposta utilizzando**: `PreferHostingUrls`  
**Variabile di ambiente**: `ASPNETCORE_PREFERHOSTINGURLS`

```csharp
WebHost.CreateDefaultBuilder(args)
    .PreferHostingUrls(false)
```

### <a name="prevent-hosting-startup"></a>Impedire l'avvio dell'hosting

Impedisce il caricamento automatico degli assembly di avvio dell'hosting, inclusi gli assembly di avvio dell'hosting configurati dall'assembly dell'app. Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.

**Chiave**: preventHostingStartup  
**Tipo**: *bool* (`true` o `1`)  
**Valore predefinito**: false  
**Imposta utilizzando**: `UseSetting`  
**Variabile di ambiente**: `ASPNETCORE_PREVENTHOSTINGSTARTUP`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.PreventHostingStartupKey, "true")
```

### <a name="server-urls"></a>URL del server

Indica gli indirizzi IP o gli indirizzi host con le porte e protocolli su cui il server deve eseguire l'ascolto per le richieste.

**Chiave**: urls  
**Tipo**: *stringa*  
**Impostazione predefinita**: http://localhost:5000  
**Imposta utilizzando**: `UseUrls`  
**Variabile di ambiente**: `ASPNETCORE_URLS`

Impostare su un elenco di prefissi URL separati da punto e virgola (;) ai quali il server deve rispondere. Ad esempio: `http://localhost:123`. Usare "\*" per indicare che il server deve eseguire l'ascolto per le richieste su tutti gli indirizzi IP o nomi host usando la porta e il protocollo specificati (ad esempio, `http://*:5000`). Il protocollo (`http://` o `https://`) deve essere incluso con ogni URL. I formati supportati variano a seconda del server.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
```

::: moniker range=">= aspnetcore-5.0"
Kestrel ha una propria API di configurazione degli endpoint. Per altre informazioni, vedere <xref:fundamentals/servers/kestrel/endpoints>.
::: moniker-end
::: moniker range="< aspnetcore-5.0"
Kestrel ha una propria API di configurazione degli endpoint. Per altre informazioni, vedere <xref:fundamentals/servers/kestrel#endpoint-configuration>.
::: moniker-end

### <a name="shutdown-timeout"></a>Timeout di arresto

Specifica il tempo di attesa per l'arresto dell'host Web.

**Chiave**: shutdownTimeoutSeconds  
**Tipo**: *int*  
**Valore predefinito**: 5  
**Imposta utilizzando**: `UseShutdownTimeout`  
**Variabile di ambiente**: `ASPNETCORE_SHUTDOWNTIMEOUTSECONDS`

Sebbene la chiave accetti *int* con `UseSetting` (ad esempio, `.UseSetting(WebHostDefaults.ShutdownTimeoutKey, "10")`), il metodo di estensione [UseShutdownTimeout](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout) accetta [TimeSpan](/dotnet/api/system.timespan).

Durante il periodo di timeout, l'hosting:

* Attiva [IApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.applicationstopping).
* Tenta di arrestare i servizi ospitati, registrando eventuali errori per i servizi che non si interrompono.

Se il periodo di timeout scade prima che siano stati arrestati tutti i servizi ospitati, gli eventuali servizi attivi rimanenti vengono interrotti quando l'app viene arrestata. I servizi si arrestano anche se non hanno completato l'elaborazione. Se l'arresto dei servizi richiede più tempo, aumentare il valore di timeout.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseShutdownTimeout(TimeSpan.FromSeconds(10))
```

### <a name="startup-assembly"></a>Assembly di avvio

Determina l'assembly per la ricerca della classe `Startup`.

**Chiave**: startupAssembly  
**Tipo**: *stringa*  
**Impostazione predefinita**: assembly dell'app  
**Imposta utilizzando**: `UseStartup`  
**Variabile di ambiente**: `ASPNETCORE_STARTUPASSEMBLY`

È possibile fare riferimento all'assembly per nome (`string`) o per tipo (`TStartup`). Se vengono chiamati più metodi `UseStartup`, l'ultimo metodo ha la precedenza.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup("StartupAssemblyName")
```

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup<TStartup>()
```

### <a name="web-root"></a>Radice Web

Imposta il percorso relativo degli asset statici dell'app.

**Chiave**: webroot  
**Tipo**: *stringa*  
**Impostazione predefinita**: il valore predefinito è `wwwroot` . Il percorso di *{radice del contenuto}/wwwroot* deve esistere. Se il percorso non esiste, viene usato un provider di file no-op.  
**Imposta utilizzando**: `UseWebRoot`  
**Variabile di ambiente**: `ASPNETCORE_WEBROOT`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseWebRoot("public")
```

Per altre informazioni, vedere:

* [Nozioni fondamentali: radice Web](xref:fundamentals/index#web-root)
* [Radice del contenuto](#content-root)

## <a name="override-configuration"></a>Override della configurazione

Usare la [Configurazione](xref:fundamentals/configuration/index) per configurare l'host Web. Nell'esempio seguente la configurazione dell'host è specificata facoltativamente nel file *hostsettings.json*. Qualsiasi configurazione caricata dal file *hostsettings.json* può essere sostituita dagli argomenti della riga di comando. La configurazione compilata (in `config`) viene usata per configurare l'host con [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration). La configurazione di `IWebHostBuilder` viene aggiunta alla configurazione dell'app, ma non il contrario: &mdash;`ConfigureAppConfiguration` non influenza la configurazione di `IWebHostBuilder`.

Esecuzione dell'override della configurazione specificata da `UseUrls` con prima la configurazione *hostsettings.json* e quindi con la configurazione dell'argomento della riga di comando:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hostsettings.json", optional: true)
            .AddCommandLine(args)
            .Build();

        return WebHost.CreateDefaultBuilder(args)
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            });
    }
}
```

*hostsettings.json*:

```json
{
    urls: "http://*:5005"
}
```

> [!NOTE]
> [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) copia solo le chiavi dall'oggetto fornito `IConfiguration` alla configurazione del generatore host. Pertanto, l'impostazione di `reloadOnChange: true` per i file JSON, INI e XML non ha alcun effetto.

Per specificare l'host eseguito in un URL specifico, il valore desiderato può essere passato da un prompt dei comandi durante l'esecuzione di [dotnet run](/dotnet/core/tools/dotnet-run). L'argomento della riga di comando esegue l'override del valore `urls` dal file *hostsettings.json*, mentre il server esegue l'ascolto sulla porta 8080:

```dotnetcli
dotnet run --urls "http://*:8080"
```

## <a name="manage-the-host"></a>Gestire l'host

**Esegui**

Il metodo `Run` avvia l'app Web e blocca il thread di chiamata fino all'arresto dell'host:

```csharp
host.Run();
```

**Inizia**

Eseguire l'host in modo non bloccante chiamandone il metodo `Start`:

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

Se viene passato un elenco di URL al metodo `Start`, viene eseguito l'ascolto sugli URL specificati:

```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

L'app può inizializzare e avviare un nuovo host usando i valori predefiniti preconfigurati di `CreateDefaultBuilder` con un metodo pratico statico. Questi metodi avviano il server senza l'output della console e con l'attesa [WaitForShutdown](/dotnet/api/microsoft.aspnetcore.hosting.webhostextensions.waitforshutdown) di un'interruzione (Ctrl-C/SIGINT o SIGTERM):

**Start(RequestDelegate app)**

Start con `RequestDelegate`:

```csharp
using (var host = WebHost.Start(app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Creare una richiesta nel browser a `http://localhost:5000` per ricevere la risposta "Hello World!" `WaitForShutdown` rimane bloccato fino a quando non viene eseguita un'interruzione (Ctrl-C/SIGINT o SIGTERM). L'app visualizza il messaggio `Console.WriteLine` e attende la pressione di un tasto per chiudersi.

**Start(string url, RequestDelegate app)**

Start con un URL e `RequestDelegate`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Produce lo stesso risultato di **Start(app RequestDelegate)**, ad eccezione del fatto che l'app risponde su `http://localhost:8080`.

**Start(Action\<IRouteBuilder> routeBuilder)**

Usare un'istanza di `IRouteBuilder` ([Microsoft.AspNetCore.Routing](https://www.nuget.org/packages/Microsoft.AspNetCore.Routing/)) per usare il middleware di routing:

```csharp
using (var host = WebHost.Start(router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Usare le richieste del browser seguenti con l'esempio:

| Richiesta                                    | Risposta                                 |
| ------------------------------------------ | ---------------------------------------- |
| `http://localhost:5000/hello/Martin`       | Hello, Martin!                           |
| `http://localhost:5000/buenosdias/Catrina` | Buenos dias, Catrina!                    |
| `http://localhost:5000/throw/ooops!`       | Genera un'eccezione con la stringa "ooops!" |
| `http://localhost:5000/throw`              | Genera un'eccezione con la stringa "Uh oh!" |
| `http://localhost:5000/Sante/Kevin`        | Sante, Kevin!                            |
| `http://localhost:5000`                    | Hello World!                             |

`WaitForShutdown` rimane bloccato fino a quando non viene eseguita un'interruzione (Ctrl-C/SIGINT o SIGTERM). L'app visualizza il messaggio `Console.WriteLine` e attende la pressione di un tasto per chiudersi.

**Start(string url, Action\<IRouteBuilder> routeBuilder)**

Usare un URL e un'istanza di `IRouteBuilder`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

Produce lo stesso risultato di **Start(Action\<IRouteBuilder> routeBuilder)**, ad eccezione del fatto che l'app risponde in `http://localhost:8080`.

**StartWith(Action\<IApplicationBuilder> app)**

Specificare un delegato per configurare `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith(app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

Creare una richiesta nel browser a `http://localhost:5000` per ricevere la risposta "Hello World!" `WaitForShutdown` rimane bloccato fino a quando non viene eseguita un'interruzione (Ctrl-C/SIGINT o SIGTERM). L'app visualizza il messaggio `Console.WriteLine` e attende la pressione di un tasto per chiudersi.

**StartWith(string url, Action\<IApplicationBuilder> app)**

Specificare un URL e un delegato per configurare `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith("http://localhost:8080", app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

Produce lo stesso risultato di **StartWith(Action\<IApplicationBuilder> app)**, ad eccezione del fatto che l'app risponde su `http://localhost:8080`.

::: moniker range=">= aspnetcore-3.0"

## <a name="iwebhostenvironment-interface"></a>Interfaccia IWebHostEnvironment

L' `IWebHostEnvironment` interfaccia fornisce informazioni sull'ambiente di hosting Web dell'app. Usare l'[inserimento di un costruttore](xref:fundamentals/dependency-injection) per ottenere `IWebHostEnvironment` per poterne usare le proprietà e i metodi di estensione:

```csharp
public class CustomFileReader
{
    private readonly IWebHostEnvironment _env;

    public CustomFileReader(IWebHostEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

È possibile usare un [approccio basato su convenzione](xref:fundamentals/environments#environment-based-startup-class-and-methods) per configurare l'app all'avvio in base all'ambiente. In alternativa, inserire `IWebHostEnvironment` nel costruttore `Startup` per l'utilizzo in `ConfigureServices`:

```csharp
public class Startup
{
    public Startup(IWebHostEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IWebHostEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> Oltre al metodo di estensione `IsDevelopment`, `IWebHostEnvironment` offre i metodi `IsStaging`, `IsProduction` e `IsEnvironment(string environmentName)`. Per altre informazioni, vedere <xref:fundamentals/environments>.

Il servizio `IWebHostEnvironment` può anche essere inserito direttamente nel metodo `Configure` per configurare la pipeline di elaborazione:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the Developer Exception Page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

`IWebHostEnvironment` può essere inserito nel metodo `Invoke` durante la creazione di [middleware](xref:fundamentals/middleware/write) personalizzato:

```csharp
public async Task Invoke(HttpContext context, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="ihostingenvironment-interface"></a>Interfaccia IHostingEnvironment

L'[interfaccia IHostingEnvironment](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment) fornisce informazioni sull'ambiente di hosting Web dell'app. Usare l'[inserimento di un costruttore](xref:fundamentals/dependency-injection) per ottenere `IHostingEnvironment` per poterne usare le proprietà e i metodi di estensione:

```csharp
public class CustomFileReader
{
    private readonly IHostingEnvironment _env;

    public CustomFileReader(IHostingEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

È possibile usare un [approccio basato su convenzione](xref:fundamentals/environments#environment-based-startup-class-and-methods) per configurare l'app all'avvio in base all'ambiente. In alternativa, inserire `IHostingEnvironment` nel costruttore `Startup` per l'utilizzo in `ConfigureServices`:

```csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IHostingEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> Oltre al metodo di estensione `IsDevelopment`, `IHostingEnvironment` offre i metodi `IsStaging`, `IsProduction` e `IsEnvironment(string environmentName)`. Per altre informazioni, vedere <xref:fundamentals/environments>.

Il servizio `IHostingEnvironment` può anche essere inserito direttamente nel metodo `Configure` per configurare la pipeline di elaborazione:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the Developer Exception Page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

`IHostingEnvironment` può essere inserito nel metodo `Invoke` durante la creazione di [middleware](xref:fundamentals/middleware/write) personalizzato:

```csharp
public async Task Invoke(HttpContext context, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="ihostapplicationlifetime-interface"></a>Interfaccia IHostApplicationLifetime

`IHostApplicationLifetime` consente attività post-avvio e di arresto. Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi `Action` che definiscono gli eventi di avvio e arresto.

| Token di annullamento    | Attivato quando&#8230; |
| --------------------- | --------------------- |
| `ApplicationStarted`  | L'host è stato completamente avviato. |
| `ApplicationStopped`  | L'host sta completando un arresto normale. Tutte le richieste devono essere elaborate. L'arresto è bloccato fino al completamento di questo evento. |
| `ApplicationStopping` | L'host sta eseguendo un arresto normale. È possibile che le richieste siano ancora in esecuzione. L'arresto è bloccato fino al completamento di questo evento. |

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IHostApplicationLifetime appLifetime)
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

`StopApplication` richiede la chiusura dell'applicazione corrente. La classe seguente usa `StopApplication` per arrestare normalmente un'app quando viene chiamato il metodo `Shutdown` della classe:

```csharp
public class MyClass
{
    private readonly IHostApplicationLifetime _appLifetime;

    public MyClass(IHostApplicationLifetime appLifetime)
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

::: moniker range="< aspnetcore-3.0"

## <a name="iapplicationlifetime-interface"></a>Interfaccia IApplicationLifetime

[IApplicationLifetime](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime) consente attività post-avvio e di arresto. Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi `Action` che definiscono gli eventi di avvio e arresto.

| Token di annullamento    | Attivato quando&#8230; |
| --------------------- | --------------------- |
| [ApplicationStarted](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstarted) | L'host è stato completamente avviato. |
| [ApplicationStopped](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopped) | L'host sta completando un arresto normale. Tutte le richieste devono essere elaborate. L'arresto è bloccato fino al completamento di questo evento. |
| [ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopping) | L'host sta eseguendo un arresto normale. È possibile che le richieste siano ancora in esecuzione. L'arresto è bloccato fino al completamento di questo evento. |

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IApplicationLifetime appLifetime)
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

[StopApplication](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.stopapplication) richiede la chiusura dell'applicazione corrente. La classe seguente usa `StopApplication` per arrestare normalmente un'app quando viene chiamato il metodo `Shutdown` della classe:

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

## <a name="scope-validation"></a>Convalida dell'ambito

[CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) imposta [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) su `true` se l'ambiente dell'app è lo sviluppo.

Quando `ValidateScopes` è impostato su `true`, il provider di servizi predefinito esegue dei controlli per verificare che:

* I servizi con ambito non vengano risolti direttamente o indirettamente dal provider di servizi radice.
* I servizi con ambito non vengano inseriti direttamente o indirettamente in singleton.

Il provider di servizi radice venga creato con la chiamata di [BuildServiceProvider](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider). La durata del provider di servizi radice corrisponde alla durata dell'app/server quando il provider viene avviato con l'app e viene eliminato alla chiusura dell'app.

I servizi con ambito vengono eliminati dal contenitore che li ha creati. Se un servizio con ambito viene creato nel contenitore radice, la durata del servizio viene di fatto convertita in singleton, perché il servizio viene eliminato solo dal contenitore radice alla chiusura dell'app/server. La convalida degli ambiti servizio rileva queste situazioni nel contesto della chiamata di `BuildServiceProvider`.

Per convalidare sempre gli ambiti, inclusi quelli dell'ambiente di produzione, configurare [ServiceProviderOptions](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions) con [UseDefaultServiceProvider](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usedefaultserviceprovider) nel generatore di host:

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseDefaultServiceProvider((context, options) => {
        options.ValidateScopes = true;
    })
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/linux-nginx>
* <xref:host-and-deploy/linux-apache>
* <xref:host-and-deploy/windows-service>
