---
title: Controlli di integrità in ASP.NET Core
author: rick-anderson
description: Informazioni su come configurare i controlli di integrità per l'infrastruttura ASP.NET Core, ad esempio database e app.
monikerRange: '>= aspnetcore-2.2'
ms.author: riande
ms.custom: mvc
ms.date: 06/22/2020
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
uid: host-and-deploy/health-checks
ms.openlocfilehash: 272f1f098ca90434f26d6c057859a00b5519602e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588906"
---
# <a name="health-checks-in-aspnet-core"></a>Controlli di integrità in ASP.NET Core

Di [Glenn Condron](https://github.com/glennc)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core offre middleware e librerie di controlli di integrità per segnalare l'integrità dei componenti dell'infrastruttura di app.

I controlli di integrità vengono esposti da un'app come endpoint HTTP. È possibile configurare endpoint dei controlli di integrità per svariati scenari di monitoraggio in tempo reale:

* I probe di integrità possono essere usati dagli agenti di orchestrazione e dai servizi di bilanciamento del carico per controllare lo stato di un'app. Ad esempio, un agente di orchestrazione potrebbe rispondere a controllo di integrità non superato arrestando una distribuzione in sequenza o riavviando un contenitore. Un servizio di bilanciamento del carico potrebbe reagire a un'app non integra trasferendo il traffico dall'istanza con errori a un'istanza integra.
* È possibile monitorare lo stato di integrità di memoria, disco e altre risorse fisiche del server.
* I controlli di integrità possono testare le dipendenze di un'app, ad esempio i database e gli endpoint di servizio esterni, per verificare la disponibilità e il normale funzionamento.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/health-checks/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

L'app di esempio include esempi degli scenari descritti in questo argomento. Per eseguire l'app di esempio per un determinato scenario, usare il comando [dotnet run](/dotnet/core/tools/dotnet-run) dalla cartella del progetto in una shell dei comandi. Per informazioni dettagliate su come usare l'app di esempio, vedere il file *README.md* dell'app di esempio e le descrizioni degli scenari in questo argomento.

## <a name="prerequisites"></a>Prerequisiti

I controlli di integrità vengono in genere usati con un servizio di monitoraggio esterno o un agente di orchestrazione per controllare lo stato di un'app. Prima di aggiungere i controlli di integrità a un'app, decidere quale sistema di monitoraggio usare. Il sistema di monitoraggio determina quali tipi di controlli di integrità creare e come configurare i relativi endpoint.

Si fa riferimento al pacchetto [Microsoft. AspNetCore. Diagnostics. HealthChecks](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks) in modo implicito per le app ASP.NET Core. Per eseguire i controlli di integrità usando Entity Framework Core, aggiungere un riferimento al pacchetto [Microsoft. Extensions. Diagnostics. HealthChecks. EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore) .

L'app di esempio include il codice di avvio per illustrare i controlli di integrità per diversi scenari. Lo scenario di [probe del database](#database-probe) controlla l'integrità di una connessione di database usando [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks). Lo scenario [Probe DbContext](#entity-framework-core-dbcontext-probe) verifica un database usando un elemento `DbContext` di EF Core. Per esplorare gli scenari relativi ai database, l'app di esempio:

* Crea un database e fornisce la relativa stringa di connessione nel *appsettings.json* file.
* Dispone dei riferimenti al pacchetto seguenti nel file di progetto:
  * [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/)
  * [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/)

> [!NOTE]
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

Un altro scenario relativo ai controlli di integrità illustra come filtrare i controlli di integrità per una porta di gestione. L'app di esempio richiede di creare un file *Properties/launchSettings.json* che include l'URL di gestione e la porta di gestione. Per altre informazioni, vedere la sezione [Filtrare in base alla porta](#filter-by-port).

## <a name="basic-health-probe"></a>Probe di integrità di base

Per molte app, una configurazione del probe di integrità di base, che segnala la disponibilità dell'app per elaborare le richieste (*attività*), è sufficiente per individuare lo stato dell'app.

La configurazione di base registra i servizi di controllo integrità e chiama il middleware dei controlli di integrità per rispondere a un endpoint URL con una risposta di integrità. Per impostazione predefinita, non vengono registrati controlli di integrità specifici per testare particolari dipendenze o sottosistemi. L'app viene considerata integra se riesce a rispondere all'URL dell'endpoint di integrità. Il writer di risposta predefinito scrive lo stato (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) come risposta di testo non crittografato per il client, indicando uno stato [HealthStatus.Healthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus), [HealthStatus.Degraded](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) o [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus).

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. Creare un endpoint di controllo integrità chiamando `MapHealthChecks` `Startup.Configure` .

Nell'app di esempio l'endpoint di controllo di integrità viene creato in `/health` (*BasicStartup.cs*):

```csharp
public class BasicStartup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddHealthChecks();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHealthChecks("/health");
        });
    }
}
```

Per eseguire lo scenario di configurazione di base usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario basic
```

### <a name="docker-example"></a>Esempio Docker

[Docker](xref:host-and-deploy/docker/index) offre una direttiva `HEALTHCHECK` predefinita che può essere usata per controllare lo stato di un'app che usa la configurazione dei controlli di integrità di base:

```
HEALTHCHECK CMD curl --fail http://localhost:5000/health || exit
```

## <a name="create-health-checks"></a>Creare controlli di integrità

I controlli di integrità vengono creati implementando l'interfaccia <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck>. Il metodo <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> restituisce <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> che indica l'integrità come `Healthy`, `Degraded` o `Unhealthy`. Il risultato viene scritto come risposta di testo non crittografato con un codice di stato configurabile. La configurazione viene descritta nella sezione [Opzioni dei controlli di integrità](#health-check-options). <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> può anche restituire coppie chiave-valore facoltative.

Nella `ExampleHealthCheck` classe seguente viene illustrato il layout di un controllo di integrità. La logica dei controlli di integrità viene inserita nel `CheckHealthAsync` metodo. Nell'esempio seguente viene impostata una variabile fittizia, `healthCheckResultHealthy` , su `true` . Se il valore di `healthCheckResultHealthy` è impostato su `false` , viene restituito lo stato [HealthCheckResult. unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult.Unhealthy*) .

```csharp
public class ExampleHealthCheck : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        var healthCheckResultHealthy = true;

        if (healthCheckResultHealthy)
        {
            return Task.FromResult(
                HealthCheckResult.Healthy("A healthy result."));
        }

        return Task.FromResult(
            HealthCheckResult.Unhealthy("An unhealthy result."));
    }
}
```

## <a name="register-health-check-services"></a>Registrare i servizi di controllo dell'integrità

Il `ExampleHealthCheck` tipo viene aggiunto ai servizi di controllo integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> in `Startup.ConfigureServices` :

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>("example_health_check");
```

L'overload <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> illustrato nell'esempio seguente imposta lo stato di errore (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) per segnalare quando il controllo di integrità indica un errore. Se lo stato di errore è impostato su `null` (impostazione predefinita), viene segnalato [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus). Questo overload è uno scenario utile per gli autori di librerie, in cui lo stato di errore indicato dalla libreria viene applicato dall'app quando si verifica un errore di controllo dell'integrità se l'implementazione del controllo dell'integrità rispetta l'impostazione.

Si possono usare *tag* per filtrare i controlli di integrità, come illustrato più avanti nella sezione [Filtrare i controlli di integrità](#filter-health-checks).

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>(
        "example_health_check",
        failureStatus: HealthStatus.Degraded,
        tags: new[] { "example" });
```

<xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> può anche eseguire una funzione lambda. Nell'esempio seguente il nome del controllo di integrità viene specificato come `Example` e il controllo restituisce sempre uno stato integro:

```csharp
services.AddHealthChecks()
    .AddCheck("Example", () =>
        HealthCheckResult.Healthy("Example is OK!"), tags: new[] { "example" });
```

Chiamare <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddTypeActivatedCheck*> per passare argomenti a un'implementazione del controllo integrità. Nell'esempio seguente `TestHealthCheckWithArgs` accetta un numero intero e una stringa da usare quando <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> viene chiamato il metodo:

```csharp
private class TestHealthCheckWithArgs : IHealthCheck
{
    public TestHealthCheckWithArgs(int i, string s)
    {
        I = i;
        S = s;
    }

    public int I { get; set; }

    public string S { get; set; }

    public Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        ...
    }
}
```

`TestHealthCheckWithArgs` viene registrato chiamando `AddTypeActivatedCheck` con il valore integer e la stringa passati all'implementazione:

```csharp
services.AddHealthChecks()
    .AddTypeActivatedCheck<TestHealthCheckWithArgs>(
        "test", 
        failureStatus: HealthStatus.Degraded, 
        tags: new[] { "example" }, 
        args: new object[] { 5, "string" });
```

## <a name="use-health-checks-routing"></a>Usare i controlli di integrità routing

In `Startup.Configure` , chiamare `MapHealthChecks` sul generatore di endpoint con l'URL dell'endpoint o il percorso relativo:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
});
```

### <a name="require-host"></a>Richiedi host

Chiamare `RequireHost` per specificare uno o più host consentiti per l'endpoint di controllo integrità. Gli host devono essere Unicode anziché Punycode e possono includere una porta. Se non viene specificata una raccolta, viene accettato qualsiasi host.

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health").RequireHost("www.contoso.com:5001");
});
```

Per altre informazioni, vedere la sezione [Filtrare in base alla porta](#filter-by-port).

### <a name="require-authorization"></a>Richiedi autorizzazione

Chiamare `RequireAuthorization` per eseguire il middleware di autorizzazione sull'endpoint della richiesta di controllo integrità. Un `RequireAuthorization` Overload accetta uno o più criteri di autorizzazione. Se un criterio non viene specificato, vengono usati i criteri di autorizzazione predefiniti.

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health").RequireAuthorization();
});
```

### <a name="enable-cross-origin-requests-cors"></a>Abilitare richieste tra le origini (CORS)

Sebbene i controlli di integrità eseguiti manualmente da un browser non siano uno scenario di utilizzo comune, il middleware CORS può essere abilitato chiamando `RequireCors` sugli endpoint di controllo dell'integrità. Un `RequireCors` Overload accetta un delegato del generatore di criteri CORS ( `CorsPolicyBuilder` ) o un nome di criterio. Se non viene specificato alcun criterio, viene usato il criterio CORS predefinito. Per altre informazioni, vedere <xref:security/cors>.

## <a name="health-check-options"></a>Opzioni dei controlli di integrità

<xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions> consente di personalizzare il comportamento dei controlli di integrità:

* [Filtrare i controlli di integrità](#filter-health-checks)
* [Personalizzare il codice di stato HTTP](#customize-the-http-status-code)
* [Eliminare le intestazioni della cache](#suppress-cache-headers)
* [Personalizzare l'output](#customize-output)

### <a name="filter-health-checks"></a>Filtrare i controlli di integrità

Per impostazione predefinita, il middleware controlli integrità esegue tutti i controlli di integrità registrati. Per eseguire un subset dei controlli di integrità, specificare una funzione che restituisce un valore booleano all'opzione <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate>. Nell'esempio seguente il controllo di integrità `Bar` viene filtrato in base al tag (`bar_tag`) nell'istruzione condizionale della funzione, dove `true` viene restituito solo se la proprietà <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.Tags> del controllo di integrità corrisponde a `foo_tag` o a `baz_tag`:

In `Startup.ConfigureServices`:

```csharp
services.AddHealthChecks()
    .AddCheck("Foo", () =>
        HealthCheckResult.Healthy("Foo is OK!"), tags: new[] { "foo_tag" })
    .AddCheck("Bar", () =>
        HealthCheckResult.Unhealthy("Bar is unhealthy!"), tags: new[] { "bar_tag" })
    .AddCheck("Baz", () =>
        HealthCheckResult.Healthy("Baz is OK!"), tags: new[] { "baz_tag" });
```

In `Startup.Configure` il `Predicate` controllo integrità della barra è filtrato. Solo foo e Baz Execute.:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("foo_tag") ||
            check.Tags.Contains("baz_tag")
    });
});
```

### <a name="customize-the-http-status-code"></a>Personalizzare il codice di stato HTTP

Usare <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResultStatusCodes> per personalizzare il mapping dello stato di integrità dei codici di stato HTTP. Le assegnazioni <xref:Microsoft.AspNetCore.Http.StatusCodes> seguenti sono i valori predefiniti usati dal middleware. Modificare i valori dei codici di stato in base ai requisiti.

In `Startup.Configure`:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        ResultStatusCodes =
        {
            [HealthStatus.Healthy] = StatusCodes.Status200OK,
            [HealthStatus.Degraded] = StatusCodes.Status200OK,
            [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
        }
    });
});
```

### <a name="suppress-cache-headers"></a>Eliminare le intestazioni della cache

<xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.AllowCachingResponses> Controlla se il middleware dei controlli di integrità aggiunge intestazioni HTTP a una risposta Probe per evitare la memorizzazione nella cache delle risposte. Se il valore è `false` (impostazione predefinita), il middleware imposta le intestazioni `Cache-Control`, `Expires` e `Pragma` o ne esegue l'override per impedire la memorizzazione della risposta nella cache. Se il valore è `true`, il middleware non modifica le intestazioni della risposta nella cache.

In `Startup.Configure`:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        AllowCachingResponses = false
    });
});
```

### <a name="customize-output"></a>Personalizzare l'output

In `Startup.Configure` impostare l'opzione [HealthCheckOptions. ResponseWriter](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter) su un delegato per la scrittura della risposta:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        ResponseWriter = WriteResponse
    });
});
```

Il delegato predefinito scrive una risposta in testo non crittografato minima con il valore di stringa [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status). I seguenti delegati personalizzati restituiscono una risposta JSON personalizzata.

Il primo esempio dell'app di esempio illustra come usare <xref:System.Text.Json?displayProperty=fullName> :

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_WriteResponse_SystemTextJson)]

Nel secondo esempio viene illustrato come utilizzare [Newtonsoft.Jsin](https://www.nuget.org/packages/Newtonsoft.Json/):

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_WriteResponse_NewtonSoftJson)]

Nell'app di esempio impostare come commento la `SYSTEM_TEXT_JSON` [direttiva per il preprocessore](xref:index#preprocessor-directives-in-sample-code) in *CustomWriterStartup.cs* per abilitare la `Newtonsoft.Json` versione di `WriteResponse` .

L'API controlli di integrità non fornisce supporto incorporato per i formati restituiti JSON complessi, perché il formato è specifico del sistema di monitoraggio scelto. Personalizzare la risposta negli esempi precedenti in base alle esigenze. Per altre informazioni sulla serializzazione JSON con `System.Text.Json` , vedere [come serializzare e deserializzare JSON in .NET](/dotnet/standard/serialization/system-text-json-how-to).

## <a name="database-probe"></a>Probe del database

Un controllo di integrità può specificare una query di database da eseguire come test booleano per indicare se il database sta rispondendo normalmente.

L'app di esempio usa [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks), una libreria di controlli di integrità per app ASP.NET Core, per eseguire un controllo di integrità su un database di SQL Server. `AspNetCore.Diagnostics.HealthChecks` esegue una query `SELECT 1` sul database per confermare che la connessione al database sia integra.

> [!WARNING]
> Quando si controlla una connessione di database con una query, scegliere una query che viene restituita rapidamente. Dall'approccio alla query possono derivare il sovraccarico del database e il degrado delle prestazioni. Nella maggior parte dei casi, l'esecuzione di una query di test non è necessaria. È sufficiente stabilire una connessione al database. Se invece è necessario eseguire una query, scegliere una semplice query SELECT, ad esempio `SELECT 1`.

Includere un riferimento al pacchetto [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/).

Specificare una stringa di connessione al database valida nel *appsettings.json* file dell'app di esempio. L'app usa una database di SQL Server denominato `HealthCheckSample`:

[!code-json[](health-checks/samples/3.x/HealthChecksSample/appsettings.json?highlight=3)]

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. L'app di esempio chiama il metodo `AddSqlServer` di con la stringa di connessione del database (*DbHealthStartup.cs*):

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/DbHealthStartup.cs?name=snippet_ConfigureServices)]

Un endpoint di controllo dell'integrità viene creato chiamando `MapHealthChecks` in `Startup.Configure` :

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
}
```

Per eseguire lo scenario di probe del database di base usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario db
```

> [!NOTE]
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

## <a name="entity-framework-core-dbcontext-probe"></a>Probe DbContext di Entity Framework Core

Il controllo `DbContext` verifica che l'app possa comunicare con il database configurato per un `DbContext` di EF Core. Il controllo `DbContext` è supportato nelle app che:

* Usano [Entity Framework (EF) Core](/ef/core/).
* Includono un riferimento del pacchetto a [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/).

`AddDbContextCheck<TContext>` registra un controllo di integrità per un `DbContext`. `DbContext` viene reso disponibile come `TContext` al metodo. È disponibile un overload per configurare lo stato di errore, i tag e una query di test personalizzata.

Per impostazione predefinita:

* `DbContextHealthCheck` chiama il metodo `CanConnectAsync` di EF Core. È possibile determinare l'operazione da eseguire quando si controlla l'integrità usando gli overload del metodo `AddDbContextCheck`.
* Il nome del controllo di integrità è il nome del tipo `TContext`.

Nell'app di esempio, `AppDbContext` viene fornito a `AddDbContextCheck` e registrato come servizio in `Startup.ConfigureServices` (*DbContextHealthStartup.cs*):

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/DbContextHealthStartup.cs?name=snippet_ConfigureServices)]

Un endpoint di controllo dell'integrità viene creato chiamando `MapHealthChecks` in `Startup.Configure` :

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
}
```

Per eseguire lo scenario di probe `DbContext` usando l'app di esempio, verificare che il database specificato dalla stringa di connessione non esista nell'istanza di SQL Server. Se il database esiste, eliminarlo.

Eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario dbcontext
```

Dopo che l'app è in esecuzione, controllare lo stato di integrità effettuando una richiesta all'endpoint `/health` in un browser. Il database e `AppDbContext` non esistono, quindi l'app fornisce la risposta seguente:

```
Unhealthy
```

Attivare l'app di esempio per creare il database. Effettuare una richiesta a `/createdatabase`. L'app risponde:

```
Creating the database...
Done!
Navigate to /health to see the health status.
```

Effettuare una richiesta all'endpoint `/health`. Il database e il contesto esistono, quindi l'app risponde:

```
Healthy
```

Attivare l'app di esempio per eliminare il database. Effettuare una richiesta a `/deletedatabase`. L'app risponde:

```
Deleting the database...
Done!
Navigate to /health to see the health status.
```

Effettuare una richiesta all'endpoint `/health`. L'app fornisce una risposta non integra:

```
Unhealthy
```

## <a name="separate-readiness-and-liveness-probes"></a>Separare i probe di idoneità e di attività

In alcuni scenari di hosting vengono usati due controlli di integrità che distinguono due stati dell'app:

* *Conformità* indica se l'app è in esecuzione normalmente ma non è pronta a ricevere le richieste.
* *Liveity* indica se un'app si è arrestata in modo anomalo e deve essere riavviata.

Si consideri l'esempio seguente: un'app deve scaricare un file di configurazione di grandi dimensioni prima che sia pronto per elaborare le richieste. Non si vuole riavviare l'app se il download iniziale non riesce perché l'app può riprovare a scaricare il file più volte. Viene usato un *Probe di liveity* per descrivere la durata del processo e non vengono eseguiti controlli aggiuntivi. Si vuole anche impedire che le richieste vengano inviate all'app prima che il download del file di configurazione abbia esito positivo. Viene usato un *Probe di conformità* per indicare uno stato "non pronto" fino a quando il download non riesce e l'app è pronta a ricevere le richieste.

L'app di esempio contiene un controllo di integrità per segnalare il completamento dell'attività di avvio con esecuzione prolungata in un [servizio ospitato](xref:fundamentals/host/hosted-services). `StartupHostedServiceHealthCheck` espone una proprietà, `StartupTaskCompleted`, che il servizio ospitato può impostare su `true` al termine dell'attività con esecuzione prolungata (*StartupHostedServiceHealthCheck.cs*):

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/StartupHostedServiceHealthCheck.cs?name=snippet1&highlight=7-11)]

L'attività in background con esecuzione prolungata viene avviata da un [servizio ospitato](xref:fundamentals/host/hosted-services) (*Services/StartupHostedService*). Al termine dell'attività, `StartupHostedServiceHealthCheck.StartupTaskCompleted` viene impostata su `true`:

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/Services/StartupHostedService.cs?name=snippet1&highlight=18-20)]

Il controllo di integrità viene registrato con <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> in `Startup.ConfigureServices` insieme al servizio ospitato. Poiché il servizio ospitato deve impostare la proprietà sul controllo di integrità, il controllo di integrità viene anche registrato nel contenitore del servizio (*LivenessProbeStartup.cs*):

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices)]

Un endpoint di controllo dell'integrità viene creato chiamando `MapHealthChecks` in `Startup.Configure` . Nell'app di esempio gli endpoint di controllo integrità vengono creati in:

* `/health/ready` per il controllo della conformità. Il controllo di idoneità filtra i controlli di integrità per il controllo di integrità con il tag `ready`.
* `/health/live` per il controllo dell'anima. Il controllo della durata filtra l'oggetto `StartupHostedServiceHealthCheck` restituendo `false` in [HealthCheckOptions. Predicate](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate) (per altre informazioni, vedere [filtrare i controlli di integrità](#filter-health-checks))

Nell'esempio di codice seguente:

* Il controllo di conformità utilizza tutti i controlli registrati con il tag "Ready".
* `Predicate`Esclude tutti i controlli e restituisce 200-OK.

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health/ready", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("ready"),
    });

    endpoints.MapHealthChecks("/health/live", new HealthCheckOptions()
    {
        Predicate = (_) => false
    });
}
```

Per eseguire lo scenario di configurazione di idoneità/attività usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario liveness
```

In a browser visitare più volte `/health/ready` per 15 secondi. Il controllo di integrità segnala *Unhealthy* per i primi 15 secondi. Dopo 15 secondi, l'endpoint segnala *Healthy*, che indica il completamento dell'attività con esecuzione prolungata da parte del servizio ospitato.

Questo esempio crea anche un server di pubblicazione dei controlli di integrità (implementazione di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>) che esegue il primo controllo di conformità con un ritardo di due secondi. Per altre informazioni, vedere la sezione [Server di pubblicazione dei controlli di integrità](#health-check-publisher).

### <a name="kubernetes-example"></a>Esempio di Kubernetes

L'uso di controlli di idoneità e attività separati è utile in un ambiente come [Kubernetes](https://kubernetes.io/). In Kubernetes un'app potrebbe dover eseguire un'operazione di avvio con esecuzione prolungata prima di accettare le richieste, ad esempio un test della disponibilità del database sottostante. L'uso di controlli separati consente all'agente di orchestrazione di distinguere se l'app è funzionante, ma non è ancora pronta o se l'app non è stata avviata. Per altre informazioni sui probe di idoneità e di attività in Kubernetes, vedere [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) (Configurare i probe di attività e di idoneità in Kubernetes) nella documentazione di Kubernetes.

L'esempio seguente illustra la configurazione di un probe di idoneità di Kubernetes:

```yml
spec:
  template:
  spec:
    readinessProbe:
      # an http probe
      httpGet:
        path: /health/ready
        port: 80
      # length of time to wait for a pod to initialize
      # after pod startup, before applying health checking
      initialDelaySeconds: 30
      timeoutSeconds: 1
    ports:
      - containerPort: 80
```

## <a name="metric-based-probe-with-a-custom-response-writer"></a>Probe basato sulle metriche con un writer di risposta personalizzata

L'app di esempio illustra un controllo di integrità della memoria con un writer di risposta personalizzata.

`MemoryHealthCheck` segnala uno stato danneggiato se l'app usa più di una determinata soglia di memoria (1 GB nell'app di esempio). <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> include informazioni sul Garbage Collector (GC) per l'app (*MemoryHealthCheck.cs*):

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/MemoryHealthCheck.cs?name=snippet1)]

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. Invece di abilitare il controllo di integrità passandolo ad <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, `MemoryHealthCheck` viene registrato come servizio. Tutti i servizi registrati da <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> sono disponibili per il middleware e i servizi di controllo dell'integrità. È consigliabile registrare i servizi di controllo dell'integrità come servizi Singleton.

In *CustomWriterStartup.cs* dell'app di esempio:

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_ConfigureServices&highlight=4)]

Un endpoint di controllo dell'integrità viene creato chiamando `MapHealthChecks` in `Startup.Configure` . Un `WriteResponse` delegato viene fornito alla proprietà <Microsoft. AspNetCore. Diagnostics. HealthChecks. HealthCheckOptions. ResponseWriter> per restituire una risposta JSON personalizzata quando viene eseguito il controllo integrità:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        ResponseWriter = WriteResponse
    });
}
```

Il `WriteResponse` delegato formatta `CompositeHealthCheckResult` in un oggetto JSON e restituisce l'output JSON per la risposta del controllo integrità. Per ulteriori informazioni, vedere la sezione [personalizzare l'output](#customize-output) .

Per eseguire lo scenario di probe basato sulle metriche con il writer di risposta personalizzata usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario writer
```

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) include scenari di controllo dell'integrità basati su metriche, tra cui i controlli dell'archiviazione su disco e dell'attività dei valori massimi.
>
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

## <a name="filter-by-port"></a>Filtrare in base alla porta

Chiamare `RequireHost` `MapHealthChecks` con un modello di URL che specifica una porta per limitare le richieste di controllo integrità alla porta specificata. Questa chiamata viene in genere usata nell'ambiente di un contenitore per esporre una porta per il monitoraggio dei servizi.

L'app di esempio configura la porta usando il [provider di configurazione della variabile di ambiente](xref:fundamentals/configuration/index#environment-variables). La porta viene impostata nel file *launchSettings.json* e passata al provider di configurazione tramite una variabile di ambiente. È anche necessario configurare il server per l'ascolto di richieste sulla porta di gestione.

Per usare l'app di esempio per illustrare la configurazione della porta di gestione, creare il file *launchSettings.json* in una cartella *Properties*.

Le seguenti *Proprietà/launchSettings.js* nel file nell'app di esempio non sono incluse nei file di progetto dell'app di esempio e devono essere create manualmente:

```json
{
  "profiles": {
    "SampleApp": {
      "commandName": "Project",
      "commandLineArgs": "",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_URLS": "http://localhost:5000/;http://localhost:5001/",
        "ASPNETCORE_MANAGEMENTPORT": "5001"
      },
      "applicationUrl": "http://localhost:5000/"
    }
  }
}
```

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. Creare un endpoint di controllo integrità chiamando `MapHealthChecks` `Startup.Configure` .

Nell'app di esempio, una chiamata a `RequireHost` sull'endpoint in `Startup.Configure` specifica la porta di gestione dalla configurazione:

```csharp
endpoints.MapHealthChecks("/health")
    .RequireHost($"*:{Configuration["ManagementPort"]}");
```

Gli endpoint vengono creati nell'app di esempio in `Startup.Configure` . Nell'esempio di codice seguente:

* Il controllo di conformità utilizza tutti i controlli registrati con il tag "Ready".
* `Predicate`Esclude tutti i controlli e restituisce 200-OK.

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health/ready", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("ready"),
    });

    endpoints.MapHealthChecks("/health/live", new HealthCheckOptions()
    {
        Predicate = (_) => false
    });
}
```

> [!NOTE]
> È possibile evitare di creare il *launchSettings.js* nel file nell'app di esempio impostando la porta di gestione in modo esplicito nel codice. In *Program.cs* , in cui <xref:Microsoft.Extensions.Hosting.HostBuilder> viene creato l'oggetto, aggiungere una chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenAnyIP*> e fornire l'endpoint della porta di gestione dell'app. In `Configure` di *ManagementPortStartup.cs* specificare la porta di gestione con `RequireHost` :
>
> *Program.cs*:
>
> ```csharp
> return new HostBuilder()
>     .ConfigureWebHostDefaults(webBuilder =>
>     {
>         webBuilder.UseKestrel()
>             .ConfigureKestrel(serverOptions =>
>             {
>                 serverOptions.ListenAnyIP(5001);
>             })
>             .UseStartup(startupType);
>     })
>     .Build();
> ```
>
> *ManagementPortStartup.cs*:
>
> ```csharp
> app.UseEndpoints(endpoints =>
> {
>     endpoints.MapHealthChecks("/health").RequireHost("*:5001");
> });
> ```

Per eseguire lo scenario di configurazione della porta di gestione usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario port
```

## <a name="distribute-a-health-check-library"></a>Distribuire una libreria di controllo di integrità

Per distribuire un controllo di integrità come libreria:

1. Scrivere un controllo di integrità che implementa l'interfaccia <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> come classe autonoma. La classe può basarsi su [inserimento delle dipendenze](xref:fundamentals/dependency-injection), attivazione del tipo e [opzioni denominate](xref:fundamentals/configuration/options) per accedere ai dati di configurazione.

   Nella logica dei controlli di integrità di `CheckHealthAsync` :

   * `data1` e `data2` vengono usati nel metodo per eseguire la logica del controllo di integrità del probe.
   * `AccessViolationException` viene gestito.

   Quando <xref:System.AccessViolationException> si verifica un <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.FailureStatus> errore, viene restituito con <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> per consentire agli utenti di configurare lo stato di errore dei controlli di integrità.

   ```csharp
   using System;
   using System.Threading;
   using System.Threading.Tasks;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   namespace SampleApp
   {
       public class ExampleHealthCheck : IHealthCheck
       {
           private readonly string _data1;
           private readonly int? _data2;

           public ExampleHealthCheck(string data1, int? data2)
           {
               _data1 = data1 ?? throw new ArgumentNullException(nameof(data1));
               _data2 = data2 ?? throw new ArgumentNullException(nameof(data2));
           }

           public async Task<HealthCheckResult> CheckHealthAsync(
               HealthCheckContext context, CancellationToken cancellationToken)
           {
               try
               {
                   return HealthCheckResult.Healthy();
               }
               catch (AccessViolationException ex)
               {
                   return new HealthCheckResult(
                       context.Registration.FailureStatus,
                       description: "An access violation occurred during the check.",
                       exception: ex,
                       data: null);
               }
           }
       }
   }
   ```

1. Scrivere un metodo di estensione con i parametri che l'app chiama nel metodo `Startup.Configure`. Nell'esempio seguente si presuma la firma del metodo di controllo dell'integrità seguente:

   ```csharp
   ExampleHealthCheck(string, string, int )
   ```

   La firma precedente indica che `ExampleHealthCheck` richiede dati aggiuntivi per elaborare la logica del probe di controllo dell'integrità. I dati vengono forniti al delegato usato per creare l'istanza del controllo di integrità quando il controllo di integrità viene registrato con un metodo di estensione. Nell'esempio seguente il chiamante specifica facoltativamente:

   * nome del controllo integrità (`name`). Se `null`, viene usato `example_health_check`.
   * punto dati stringa per il controllo di integrità (`data1`).
   * punto dati Integer per il controllo di integrità (`data2`). Se `null`, viene usato `1`.
   * stato dell'errore (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>). Il valore predefinito è `null`. Se `null`, viene segnalato [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) per uno stato di errore.
   * tag (`IEnumerable<string>`).

   ```csharp
   using System.Collections.Generic;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   public static class ExampleHealthCheckBuilderExtensions
   {
       const string DefaultName = "example_health_check";

       public static IHealthChecksBuilder AddExampleHealthCheck(
           this IHealthChecksBuilder builder,
           string name = default,
           string data1,
           int data2 = 1,
           HealthStatus? failureStatus = default,
           IEnumerable<string> tags = default)
       {
           return builder.Add(new HealthCheckRegistration(
               name ?? DefaultName,
               sp => new ExampleHealthCheck(data1, data2),
               failureStatus,
               tags));
       }
   }
   ```

## <a name="health-check-publisher"></a>Server di pubblicazione dei controlli di integrità

Quando <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> viene aggiunto al contenitore del servizio, il sistema di controllo dell'integrità esegue periodicamente i controlli di integrità e chiama `PublishAsync` con il risultato. Ciò è utile in uno scenario relativo a un sistema di monitoraggio dell'integrità basato su push, che prevede che ogni processo chiami periodicamente il sistema di monitoraggio per determinare l'integrità.

L'interfaccia <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> ha un solo metodo:

```csharp
Task PublishAsync(HealthReport report, CancellationToken cancellationToken);
```

<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions> consente di impostare:

* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>: Ritardo iniziale applicato dopo l'avvio dell'app prima dell'esecuzione di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> istanze. Il ritardo viene applicato una sola volta all'avvio e non si applica alle iterazioni successive. Il valore predefinito è cinque secondi.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period>: Periodo di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> esecuzione. Il valore predefinito è 30 secondi.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate>: Se <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> è `null` (impostazione predefinita), il servizio Publisher verifica integrità esegue tutti i controlli di integrità registrati. Per eseguire un subset dei controlli di integrità, specificare una funzione che filtra il set di controlli. Il predicato viene valutato per ogni periodo.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Timeout>: Timeout per l'esecuzione dei controlli di integrità per tutte le <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> istanze. Usare <xref:System.Threading.Timeout.InfiniteTimeSpan> per l'esecuzione senza un timeout. Il valore predefinito è 30 secondi.

Nell'app di esempio, `ReadinessPublisher` è un'implementazione di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. Lo stato del controllo di integrità viene registrato per ogni controllo a livello di log di:

* Information ( <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> ) se lo stato dei controlli di integrità è <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Healthy> .
* Error ( <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogError*> ) se lo stato è <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Degraded> o <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Unhealthy> .

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/ReadinessPublisher.cs?name=snippet_ReadinessPublisher&highlight=18-27)]

Nell'esempio di `LivenessProbeStartup` dell'app di esempio, il controllo di conformità `StartupHostedService` ha un ritardo di avvio di due secondi e il controllo viene eseguito ogni 30 secondi. Per attivare l'implementazione <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>, l'esempio registra `ReadinessPublisher` come servizio singleton nel contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection):

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices)]

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) include i server di pubblicazione per diversi sistemi, tra cui [Application Insights](/azure/application-insights/app-insights-overview).
>
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

## <a name="restrict-health-checks-with-mapwhen"></a>Limitare i controlli integrità con MapWhen

Usare <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> per creare un ramo condizionale della pipeline delle richieste per gli endpoint di controllo integrità.

Nell'esempio seguente, `MapWhen` la pipeline della richiesta viene diramata per attivare il middleware dei controlli di integrità se viene ricevuta una richiesta GET per l' `api/HealthCheck` endpoint:

```csharp
app.MapWhen(
    context => context.Request.Method == HttpMethod.Get.Method && 
        context.Request.Path.StartsWith("/api/HealthCheck"),
    builder => builder.UseHealthChecks());

app.UseEndpoints(endpoints =>
{
    endpoints.MapRazorPages();
});
```

Per altre informazioni, vedere <xref:fundamentals/middleware/index#branch-the-middleware-pipeline>.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core offre middleware e librerie di controlli di integrità per segnalare l'integrità dei componenti dell'infrastruttura di app.

I controlli di integrità vengono esposti da un'app come endpoint HTTP. È possibile configurare endpoint dei controlli di integrità per svariati scenari di monitoraggio in tempo reale:

* I probe di integrità possono essere usati dagli agenti di orchestrazione e dai servizi di bilanciamento del carico per controllare lo stato di un'app. Ad esempio, un agente di orchestrazione potrebbe rispondere a controllo di integrità non superato arrestando una distribuzione in sequenza o riavviando un contenitore. Un servizio di bilanciamento del carico potrebbe reagire a un'app non integra trasferendo il traffico dall'istanza con errori a un'istanza integra.
* È possibile monitorare lo stato di integrità di memoria, disco e altre risorse fisiche del server.
* I controlli di integrità possono testare le dipendenze di un'app, ad esempio i database e gli endpoint di servizio esterni, per verificare la disponibilità e il normale funzionamento.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/health-checks/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

L'app di esempio include esempi degli scenari descritti in questo argomento. Per eseguire l'app di esempio per un determinato scenario, usare il comando [dotnet run](/dotnet/core/tools/dotnet-run) dalla cartella del progetto in una shell dei comandi. Per informazioni dettagliate su come usare l'app di esempio, vedere il file *README.md* dell'app di esempio e le descrizioni degli scenari in questo argomento.

## <a name="prerequisites"></a>Prerequisiti

I controlli di integrità vengono in genere usati con un servizio di monitoraggio esterno o un agente di orchestrazione per controllare lo stato di un'app. Prima di aggiungere i controlli di integrità a un'app, decidere quale sistema di monitoraggio usare. Il sistema di monitoraggio determina quali tipi di controlli di integrità creare e come configurare i relativi endpoint.

Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) oppure aggiungere un riferimento al pacchetto [Microsoft.AspNetCore.Diagnostics.HealthChecks](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks).

L'app di esempio include il codice di avvio per illustrare i controlli di integrità per diversi scenari. Lo scenario di [probe del database](#database-probe) controlla l'integrità di una connessione di database usando [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks). Lo scenario [Probe DbContext](#entity-framework-core-dbcontext-probe) verifica un database usando un elemento `DbContext` di EF Core. Per esplorare gli scenari relativi ai database, l'app di esempio:

* Crea un database e fornisce la relativa stringa di connessione nel *appsettings.json* file.
* Dispone dei riferimenti al pacchetto seguenti nel file di progetto:
  * [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/)
  * [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/)

> [!NOTE]
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

Un altro scenario relativo ai controlli di integrità illustra come filtrare i controlli di integrità per una porta di gestione. L'app di esempio richiede di creare un file *Properties/launchSettings.json* che include l'URL di gestione e la porta di gestione. Per altre informazioni, vedere la sezione [Filtrare in base alla porta](#filter-by-port).

## <a name="basic-health-probe"></a>Probe di integrità di base

Per molte app, una configurazione del probe di integrità di base, che segnala la disponibilità dell'app per elaborare le richieste (*attività*), è sufficiente per individuare lo stato dell'app.

La configurazione di base registra i servizi di controllo integrità e chiama il middleware dei controlli di integrità per rispondere a un endpoint URL con una risposta di integrità. Per impostazione predefinita, non vengono registrati controlli di integrità specifici per testare particolari dipendenze o sottosistemi. L'app viene considerata integra se riesce a rispondere all'URL dell'endpoint di integrità. Il writer di risposta predefinito scrive lo stato (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) come risposta di testo non crittografato per il client, indicando uno stato [HealthStatus.Healthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus), [HealthStatus.Degraded](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) o [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus).

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. Aggiungere un endpoint per il middleware dei controlli di integrità con <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> nella pipeline di elaborazione della richiesta di `Startup.Configure` .

Nell'app di esempio l'endpoint di controllo di integrità viene creato in `/health` (*BasicStartup.cs*):

```csharp
public class BasicStartup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddHealthChecks();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseHealthChecks("/health");
    }
}
```

Per eseguire lo scenario di configurazione di base usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario basic
```

### <a name="docker-example"></a>Esempio Docker

[Docker](xref:host-and-deploy/docker/index) offre una direttiva `HEALTHCHECK` predefinita che può essere usata per controllare lo stato di un'app che usa la configurazione dei controlli di integrità di base:

```
HEALTHCHECK CMD curl --fail http://localhost:5000/health || exit
```

## <a name="create-health-checks"></a>Creare controlli di integrità

I controlli di integrità vengono creati implementando l'interfaccia <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck>. Il metodo <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> restituisce <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> che indica l'integrità come `Healthy`, `Degraded` o `Unhealthy`. Il risultato viene scritto come risposta di testo non crittografato con un codice di stato configurabile. La configurazione viene descritta nella sezione [Opzioni dei controlli di integrità](#health-check-options). <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> può anche restituire coppie chiave-valore facoltative.

### <a name="example-health-check"></a>Controllo di integrità di esempio

Nella `ExampleHealthCheck` classe seguente viene illustrato il layout di un controllo di integrità. La logica dei controlli di integrità viene inserita nel `CheckHealthAsync` metodo. Nell'esempio seguente viene impostata una variabile fittizia, `healthCheckResultHealthy` , su `true` . Se il valore di `healthCheckResultHealthy` è impostato su `false` , viene restituito lo stato [HealthCheckResult. unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult.Unhealthy*) .

```csharp
public class ExampleHealthCheck : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        var healthCheckResultHealthy = true;

        if (healthCheckResultHealthy)
        {
            return Task.FromResult(
                HealthCheckResult.Healthy("The check indicates a healthy result."));
        }

        return Task.FromResult(
            HealthCheckResult.Unhealthy("The check indicates an unhealthy result."));
    }
}
```

### <a name="register-health-check-services"></a>Registrare i servizi di controllo dell'integrità

Il `ExampleHealthCheck` tipo viene aggiunto ai servizi di controllo integrità in `Startup.ConfigureServices` con <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> :

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>("example_health_check");
```

L'overload <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> illustrato nell'esempio seguente imposta lo stato di errore (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) per segnalare quando il controllo di integrità indica un errore. Se lo stato di errore è impostato su `null` (impostazione predefinita), viene segnalato [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus). Questo overload è uno scenario utile per gli autori di librerie, in cui lo stato di errore indicato dalla libreria viene applicato dall'app quando si verifica un errore di controllo dell'integrità se l'implementazione del controllo dell'integrità rispetta l'impostazione.

Si possono usare *tag* per filtrare i controlli di integrità, come illustrato più avanti nella sezione [Filtrare i controlli di integrità](#filter-health-checks).

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>(
        "example_health_check",
        failureStatus: HealthStatus.Degraded,
        tags: new[] { "example" });
```

<xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> può anche eseguire una funzione lambda. Nell'esempio seguente `Startup.ConfigureServices` il nome del controllo integrità viene specificato come `Example` e il controllo restituisce sempre uno stato integro:

```csharp
services.AddHealthChecks()
    .AddCheck("Example", () =>
        HealthCheckResult.Healthy("Example is OK!"), tags: new[] { "example" });
```

### <a name="use-health-checks-middleware"></a>Usare il middleware per i controlli di integrità

In `Startup.Configure` chiamare <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> nella pipeline di elaborazione con l'URL dell'endpoint o il percorso relativo:

```csharp
app.UseHealthChecks("/health");
```

Se i controlli di integrità devono essere in ascolto su una porta specifica, usare un overload di <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> per impostare la porta, come illustrato più avanti nella sezione [Filtrare in base alla porta](#filter-by-port):

```csharp
app.UseHealthChecks("/health", port: 8000);
```

## <a name="health-check-options"></a>Opzioni dei controlli di integrità

<xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions> consente di personalizzare il comportamento dei controlli di integrità:

* [Filtrare i controlli di integrità](#filter-health-checks)
* [Personalizzare il codice di stato HTTP](#customize-the-http-status-code)
* [Eliminare le intestazioni della cache](#suppress-cache-headers)
* [Personalizzare l'output](#customize-output)

### <a name="filter-health-checks"></a>Filtrare i controlli di integrità

Per impostazione predefinita, il middleware controlli integrità esegue tutti i controlli di integrità registrati. Per eseguire un subset dei controlli di integrità, specificare una funzione che restituisce un valore booleano all'opzione <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate>. Nell'esempio seguente il controllo di integrità `Bar` viene filtrato in base al tag (`bar_tag`) nell'istruzione condizionale della funzione, dove `true` viene restituito solo se la proprietà <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.Tags> del controllo di integrità corrisponde a `foo_tag` o a `baz_tag`:

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;

public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddCheck("Foo", () =>
            HealthCheckResult.Healthy("Foo is OK!"), tags: new[] { "foo_tag" })
        .AddCheck("Bar", () =>
            HealthCheckResult.Unhealthy("Bar is unhealthy!"), 
                tags: new[] { "bar_tag" })
        .AddCheck("Baz", () =>
            HealthCheckResult.Healthy("Baz is OK!"), tags: new[] { "baz_tag" });
}

public void Configure(IApplicationBuilder app)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("foo_tag") ||
            check.Tags.Contains("baz_tag")
    });
}
```

### <a name="customize-the-http-status-code"></a>Personalizzare il codice di stato HTTP

Usare <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResultStatusCodes> per personalizzare il mapping dello stato di integrità dei codici di stato HTTP. Le assegnazioni <xref:Microsoft.AspNetCore.Http.StatusCodes> seguenti sono i valori predefiniti usati dal middleware. Modificare i valori dei codici di stato in base ai requisiti.

In `Startup.Configure`:

```csharp
//using Microsoft.AspNetCore.Diagnostics.HealthChecks;
//using Microsoft.Extensions.Diagnostics.HealthChecks;

app.UseHealthChecks("/health", new HealthCheckOptions()
{
    ResultStatusCodes =
    {
        [HealthStatus.Healthy] = StatusCodes.Status200OK,
        [HealthStatus.Degraded] = StatusCodes.Status200OK,
        [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
    }
});
```

### <a name="suppress-cache-headers"></a>Eliminare le intestazioni della cache

<xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.AllowCachingResponses> Controlla se il middleware dei controlli di integrità aggiunge intestazioni HTTP a una risposta Probe per evitare la memorizzazione nella cache delle risposte. Se il valore è `false` (impostazione predefinita), il middleware imposta le intestazioni `Cache-Control`, `Expires` e `Pragma` o ne esegue l'override per impedire la memorizzazione della risposta nella cache. Se il valore è `true`, il middleware non modifica le intestazioni della risposta nella cache.

In `Startup.Configure`:

```csharp
//using Microsoft.AspNetCore.Diagnostics.HealthChecks;
//using Microsoft.Extensions.Diagnostics.HealthChecks;

app.UseHealthChecks("/health", new HealthCheckOptions()
{
    AllowCachingResponses = false
});
```

### <a name="customize-output"></a>Personalizzare l'output

L'opzione <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter> ottiene o imposta un delegato usato per scrivere la risposta. Il delegato predefinito scrive una risposta in testo non crittografato minima con il valore di stringa [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).

In `Startup.Configure`:

```csharp
// using Microsoft.AspNetCore.Diagnostics.HealthChecks;
// using Microsoft.Extensions.Diagnostics.HealthChecks;

app.UseHealthChecks("/health", new HealthCheckOptions()
{
    ResponseWriter = WriteResponse
});
```

Il delegato predefinito scrive una risposta in testo non crittografato minima con il valore di stringa [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status). Il delegato personalizzato seguente, `WriteResponse` , restituisce una risposta JSON personalizzata:

```csharp
private static Task WriteResponse(HttpContext httpContext, HealthReport result)
{
    httpContext.Response.ContentType = "application/json";

    var json = new JObject(
        new JProperty("status", result.Status.ToString()),
        new JProperty("results", new JObject(result.Entries.Select(pair =>
            new JProperty(pair.Key, new JObject(
                new JProperty("status", pair.Value.Status.ToString()),
                new JProperty("description", pair.Value.Description),
                new JProperty("data", new JObject(pair.Value.Data.Select(
                    p => new JProperty(p.Key, p.Value))))))))));
    return httpContext.Response.WriteAsync(
        json.ToString(Formatting.Indented));
}
```

Il sistema di controlli di integrità non fornisce il supporto incorporato per i formati restituiti JSON complessi, perché il formato è specifico del sistema di monitoraggio scelto. È possibile personalizzare nell' `JObject` esempio precedente, se necessario, per soddisfare le proprie esigenze.

## <a name="database-probe"></a>Probe del database

Un controllo di integrità può specificare una query di database da eseguire come test booleano per indicare se il database sta rispondendo normalmente.

L'app di esempio usa [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks), una libreria di controlli di integrità per app ASP.NET Core, per eseguire un controllo di integrità su un database di SQL Server. `AspNetCore.Diagnostics.HealthChecks` esegue una query `SELECT 1` sul database per confermare che la connessione al database sia integra.

> [!WARNING]
> Quando si controlla una connessione di database con una query, scegliere una query che viene restituita rapidamente. Dall'approccio alla query possono derivare il sovraccarico del database e il degrado delle prestazioni. Nella maggior parte dei casi, l'esecuzione di una query di test non è necessaria. È sufficiente stabilire una connessione al database. Se invece è necessario eseguire una query, scegliere una semplice query SELECT, ad esempio `SELECT 1`.

Includere un riferimento al pacchetto [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/).

Specificare una stringa di connessione al database valida nel *appsettings.json* file dell'app di esempio. L'app usa una database di SQL Server denominato `HealthCheckSample`:

[!code-json[](health-checks/samples/2.x/HealthChecksSample/appsettings.json?highlight=3)]

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. L'app di esempio chiama il metodo `AddSqlServer` di con la stringa di connessione del database (*DbHealthStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbHealthStartup.cs?name=snippet_ConfigureServices)]

Chiamare il middleware verifica integrità nella pipeline di elaborazione delle app in `Startup.Configure` :

```csharp
app.UseHealthChecks("/health");
```

Per eseguire lo scenario di probe del database di base usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario db
```

> [!NOTE]
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

## <a name="entity-framework-core-dbcontext-probe"></a>Probe DbContext di Entity Framework Core

Il controllo `DbContext` verifica che l'app possa comunicare con il database configurato per un `DbContext` di EF Core. Il controllo `DbContext` è supportato nelle app che:

* Usano [Entity Framework (EF) Core](/ef/core/).
* Includono un riferimento del pacchetto a [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/).

`AddDbContextCheck<TContext>` registra un controllo di integrità per un `DbContext`. `DbContext` viene reso disponibile come `TContext` al metodo. È disponibile un overload per configurare lo stato di errore, i tag e una query di test personalizzata.

Per impostazione predefinita:

* `DbContextHealthCheck` chiama il metodo `CanConnectAsync` di EF Core. È possibile determinare l'operazione da eseguire quando si controlla l'integrità usando gli overload del metodo `AddDbContextCheck`.
* Il nome del controllo di integrità è il nome del tipo `TContext`.

Nell'app di esempio, `AppDbContext` viene fornito a `AddDbContextCheck` e registrato come servizio in `Startup.ConfigureServices` (*DbContextHealthStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbContextHealthStartup.cs?name=snippet_ConfigureServices)]

Nell'app di esempio, `UseHealthChecks` aggiunge il middleware dei controlli di integrità in `Startup.Configure` .

```csharp
app.UseHealthChecks("/health");
```

Per eseguire lo scenario di probe `DbContext` usando l'app di esempio, verificare che il database specificato dalla stringa di connessione non esista nell'istanza di SQL Server. Se il database esiste, eliminarlo.

Eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario dbcontext
```

Dopo che l'app è in esecuzione, controllare lo stato di integrità effettuando una richiesta all'endpoint `/health` in un browser. Il database e `AppDbContext` non esistono, quindi l'app fornisce la risposta seguente:

```
Unhealthy
```

Attivare l'app di esempio per creare il database. Effettuare una richiesta a `/createdatabase`. L'app risponde:

```
Creating the database...
Done!
Navigate to /health to see the health status.
```

Effettuare una richiesta all'endpoint `/health`. Il database e il contesto esistono, quindi l'app risponde:

```
Healthy
```

Attivare l'app di esempio per eliminare il database. Effettuare una richiesta a `/deletedatabase`. L'app risponde:

```
Deleting the database...
Done!
Navigate to /health to see the health status.
```

Effettuare una richiesta all'endpoint `/health`. L'app fornisce una risposta non integra:

```
Unhealthy
```

## <a name="separate-readiness-and-liveness-probes"></a>Separare i probe di idoneità e di attività

In alcuni scenari di hosting vengono usati due controlli di integrità che distinguono due stati dell'app:

* *Conformità* indica se l'app è in esecuzione normalmente ma non è pronta a ricevere le richieste.
* *Liveity* indica se un'app si è arrestata in modo anomalo e deve essere riavviata.

Si consideri l'esempio seguente: un'app deve scaricare un file di configurazione di grandi dimensioni prima che sia pronto per elaborare le richieste. Non si vuole riavviare l'app se il download iniziale non riesce perché l'app può riprovare a scaricare il file più volte. Viene usato un *Probe di liveity* per descrivere la durata del processo e non vengono eseguiti controlli aggiuntivi. Si vuole anche impedire che le richieste vengano inviate all'app prima che il download del file di configurazione abbia esito positivo. Viene usato un *Probe di conformità* per indicare uno stato "non pronto" fino a quando il download non riesce e l'app è pronta a ricevere le richieste.

L'app di esempio contiene un controllo di integrità per segnalare il completamento dell'attività di avvio con esecuzione prolungata in un [servizio ospitato](xref:fundamentals/host/hosted-services). `StartupHostedServiceHealthCheck` espone una proprietà, `StartupTaskCompleted`, che il servizio ospitato può impostare su `true` al termine dell'attività con esecuzione prolungata (*StartupHostedServiceHealthCheck.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/StartupHostedServiceHealthCheck.cs?name=snippet1&highlight=7-11)]

L'attività in background con esecuzione prolungata viene avviata da un [servizio ospitato](xref:fundamentals/host/hosted-services) (*Services/StartupHostedService*). Al termine dell'attività, `StartupHostedServiceHealthCheck.StartupTaskCompleted` viene impostata su `true`:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/Services/StartupHostedService.cs?name=snippet1&highlight=18-20)]

Il controllo di integrità viene registrato con <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> in `Startup.ConfigureServices` insieme al servizio ospitato. Poiché il servizio ospitato deve impostare la proprietà sul controllo di integrità, il controllo di integrità viene anche registrato nel contenitore del servizio (*LivenessProbeStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices)]

Chiamare il middleware verifica integrità nella pipeline di elaborazione delle app in `Startup.Configure` . Nell'app di esempio gli endpoint dei controlli di integrità vengono creati in `/health/ready` per il controllo di idoneità e in `/health/live` per il controllo di attività. Il controllo di idoneità filtra i controlli di integrità per il controllo di integrità con il tag `ready`. Il controllo di attività filtra `StartupHostedServiceHealthCheck` restituendo `false` in [HealthCheckOptions.Predicate](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate) (per altre informazioni, vedere [Filtrare i controlli di integrità](#filter-health-checks)):

```csharp
app.UseHealthChecks("/health/ready", new HealthCheckOptions()
{
    Predicate = (check) => check.Tags.Contains("ready"), 
});

app.UseHealthChecks("/health/live", new HealthCheckOptions()
{
    Predicate = (_) => false
});
```

Per eseguire lo scenario di configurazione di idoneità/attività usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario liveness
```

In a browser visitare più volte `/health/ready` per 15 secondi. Il controllo di integrità segnala *Unhealthy* per i primi 15 secondi. Dopo 15 secondi, l'endpoint segnala *Healthy*, che indica il completamento dell'attività con esecuzione prolungata da parte del servizio ospitato.

Questo esempio crea anche un server di pubblicazione dei controlli di integrità (implementazione di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>) che esegue il primo controllo di conformità con un ritardo di due secondi. Per altre informazioni, vedere la sezione [Server di pubblicazione dei controlli di integrità](#health-check-publisher).

### <a name="kubernetes-example"></a>Esempio di Kubernetes

L'uso di controlli di idoneità e attività separati è utile in un ambiente come [Kubernetes](https://kubernetes.io/). In Kubernetes un'app potrebbe dover eseguire un'operazione di avvio con esecuzione prolungata prima di accettare le richieste, ad esempio un test della disponibilità del database sottostante. L'uso di controlli separati consente all'agente di orchestrazione di distinguere se l'app è funzionante, ma non è ancora pronta o se l'app non è stata avviata. Per altre informazioni sui probe di idoneità e di attività in Kubernetes, vedere [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) (Configurare i probe di attività e di idoneità in Kubernetes) nella documentazione di Kubernetes.

L'esempio seguente illustra la configurazione di un probe di idoneità di Kubernetes:

```
spec:
  template:
  spec:
    readinessProbe:
      # an http probe
      httpGet:
        path: /health/ready
        port: 80
      # length of time to wait for a pod to initialize
      # after pod startup, before applying health checking
      initialDelaySeconds: 30
      timeoutSeconds: 1
    ports:
      - containerPort: 80
```

## <a name="metric-based-probe-with-a-custom-response-writer"></a>Probe basato sulle metriche con un writer di risposta personalizzata

L'app di esempio illustra un controllo di integrità della memoria con un writer di risposta personalizzata.

`MemoryHealthCheck` segnala uno stato di tipo non integro se l'applicazione usa più di una determinata soglia di memoria (1 GB nell'app di esempio). <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> include informazioni sul Garbage Collector (GC) per l'app (*MemoryHealthCheck.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/MemoryHealthCheck.cs?name=snippet1)]

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. Invece di abilitare il controllo di integrità passandolo ad <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, `MemoryHealthCheck` viene registrato come servizio. Tutti i servizi registrati da <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> sono disponibili per il middleware e i servizi di controllo dell'integrità. È consigliabile registrare i servizi di controllo dell'integrità come servizi Singleton.

Nell'app di esempio (*CustomWriterStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_ConfigureServices&highlight=4)]

Chiamare il middleware verifica integrità nella pipeline di elaborazione delle app in `Startup.Configure` . Viene fornito un delegato `WriteResponse` alla proprietà `ResponseWriter` per visualizzare una risposta JSON personalizzata quando il controllo di integrità viene eseguito:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        // This custom writer formats the detailed status as JSON.
        ResponseWriter = WriteResponse
    });
}
```

Il metodo `WriteResponse` formatta `CompositeHealthCheckResult` in un oggetto JSON e genera l'output JSON per la risposta del controllo di integrità:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_WriteResponse)]

Per eseguire lo scenario di probe basato sulle metriche con il writer di risposta personalizzata usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario writer
```

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) include scenari di controllo dell'integrità basati su metriche, tra cui i controlli dell'archiviazione su disco e dell'attività dei valori massimi.
>
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

## <a name="filter-by-port"></a>Filtrare in base alla porta

La chiamata a <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> con una porta limita le richieste di controllo dell'integrità alla porta specificata. Questa chiamata viene in genere usata nell'ambiente di un contenitore per esporre una porta per il monitoraggio dei servizi.

L'app di esempio configura la porta usando il [provider di configurazione della variabile di ambiente](xref:fundamentals/configuration/index#environment-variables-configuration-provider). La porta viene impostata nel file *launchSettings.json* e passata al provider di configurazione tramite una variabile di ambiente. È anche necessario configurare il server per l'ascolto di richieste sulla porta di gestione.

Per usare l'app di esempio per illustrare la configurazione della porta di gestione, creare il file *launchSettings.json* in una cartella *Properties*.

Le seguenti *Proprietà/launchSettings.js* nel file nell'app di esempio non sono incluse nei file di progetto dell'app di esempio e devono essere create manualmente:

```json
{
  "profiles": {
    "SampleApp": {
      "commandName": "Project",
      "commandLineArgs": "",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_URLS": "http://localhost:5000/;http://localhost:5001/",
        "ASPNETCORE_MANAGEMENTPORT": "5001"
      },
      "applicationUrl": "http://localhost:5000/"
    }
  }
}
```

Registrare i servizi dei controlli di integrità con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`. La chiamata a <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> specifica la porta di gestione (*ManagementPortStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/ManagementPortStartup.cs?name=snippet1&highlight=17)]

> [!NOTE]
> È possibile evitare di creare il file *launchSettings.json* nell'app di esempio impostando gli URL e la porta di gestione in modo esplicito nel codice. In *Program.cs*, dove viene creato <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder>, aggiungere una chiamata a <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*> e fornire l'endpoint della porta di gestione e l'endpoint di risposta normale dell'app. In *ManagementPortStartup.cs*, dove viene chiamato <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*>, specificare la porta di gestione in modo esplicito.
>
> *Program.cs*:
>
> ```csharp
> return new WebHostBuilder()
>     .UseConfiguration(config)
>     .UseUrls("http://localhost:5000/;http://localhost:5001/")
>     .ConfigureLogging(builder =>
>     {
>         builder.SetMinimumLevel(LogLevel.Trace);
>         builder.AddConfiguration(config);
>         builder.AddConsole();
>     })
>     .UseKestrel()
>     .UseStartup(startupType)
>     .Build();
> ```
>
> *ManagementPortStartup.cs*:
>
> ```csharp
> app.UseHealthChecks("/health", port: 5001);
> ```

Per eseguire lo scenario di configurazione della porta di gestione usando l'app di esempio, eseguire il comando seguente dalla cartella del progetto in una shell dei comandi:

```dotnetcli
dotnet run --scenario port
```

## <a name="distribute-a-health-check-library"></a>Distribuire una libreria di controllo di integrità

Per distribuire un controllo di integrità come libreria:

1. Scrivere un controllo di integrità che implementa l'interfaccia <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> come classe autonoma. La classe può basarsi su [inserimento delle dipendenze](xref:fundamentals/dependency-injection), attivazione del tipo e [opzioni denominate](xref:fundamentals/configuration/options) per accedere ai dati di configurazione.

   Nella logica dei controlli di integrità di `CheckHealthAsync` :

   * `data1` e `data2` vengono usati nel metodo per eseguire la logica del controllo di integrità del probe.
   * `AccessViolationException` viene gestito.

   Quando <xref:System.AccessViolationException> si verifica un <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.FailureStatus> errore, viene restituito con <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> per consentire agli utenti di configurare lo stato di errore dei controlli di integrità.

   ```csharp
   using System;
   using System.Threading;
   using System.Threading.Tasks;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   public class ExampleHealthCheck : IHealthCheck
   {
       private readonly string _data1;
       private readonly int? _data2;

       public ExampleHealthCheck(string data1, int? data2)
       {
           _data1 = data1 ?? throw new ArgumentNullException(nameof(data1));
           _data2 = data2 ?? throw new ArgumentNullException(nameof(data2));
       }

       public async Task<HealthCheckResult> CheckHealthAsync(
           HealthCheckContext context, CancellationToken cancellationToken)
       {
           try
           {
               return HealthCheckResult.Healthy();
           }
           catch (AccessViolationException ex)
           {
               return new HealthCheckResult(
                   context.Registration.FailureStatus,
                   description: "An access violation occurred during the check.",
                   exception: ex,
                   data: null);
           }
       }
   }
   ```

1. Scrivere un metodo di estensione con i parametri che l'app chiama nel metodo `Startup.Configure`. Nell'esempio seguente si presuma la firma del metodo di controllo dell'integrità seguente:

   ```csharp
   ExampleHealthCheck(string, string, int )
   ```

   La firma precedente indica che `ExampleHealthCheck` richiede dati aggiuntivi per elaborare la logica del probe di controllo dell'integrità. I dati vengono forniti al delegato usato per creare l'istanza del controllo di integrità quando il controllo di integrità viene registrato con un metodo di estensione. Nell'esempio seguente il chiamante specifica facoltativamente:

   * nome del controllo integrità (`name`). Se `null`, viene usato `example_health_check`.
   * punto dati stringa per il controllo di integrità (`data1`).
   * punto dati Integer per il controllo di integrità (`data2`). Se `null`, viene usato `1`.
   * stato dell'errore (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>). Il valore predefinito è `null`. Se `null`, viene segnalato [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) per uno stato di errore.
   * tag (`IEnumerable<string>`).

   ```csharp
   using System.Collections.Generic;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   public static class ExampleHealthCheckBuilderExtensions
   {
       const string DefaultName = "example_health_check";

       public static IHealthChecksBuilder AddExampleHealthCheck(
           this IHealthChecksBuilder builder,
           string name = default,
           string data1,
           int data2 = 1,
           HealthStatus? failureStatus = default,
           IEnumerable<string> tags = default)
       {
           return builder.Add(new HealthCheckRegistration(
               name ?? DefaultName,
               sp => new ExampleHealthCheck(data1, data2),
               failureStatus,
               tags));
       }
   }
   ```

## <a name="health-check-publisher"></a>Server di pubblicazione dei controlli di integrità

Quando <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> viene aggiunto al contenitore del servizio, il sistema di controllo dell'integrità esegue periodicamente i controlli di integrità e chiama `PublishAsync` con il risultato. Ciò è utile in uno scenario relativo a un sistema di monitoraggio dell'integrità basato su push, che prevede che ogni processo chiami periodicamente il sistema di monitoraggio per determinare l'integrità.

L'interfaccia <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> ha un solo metodo:

```csharp
Task PublishAsync(HealthReport report, CancellationToken cancellationToken);
```

<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions> consente di impostare:

* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>: Ritardo iniziale applicato dopo l'avvio dell'app prima dell'esecuzione di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> istanze. Il ritardo viene applicato una sola volta all'avvio e non si applica alle iterazioni successive. Il valore predefinito è cinque secondi.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period>: Periodo di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> esecuzione. Il valore predefinito è 30 secondi.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate>: Se <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> è `null` (impostazione predefinita), il servizio Publisher verifica integrità esegue tutti i controlli di integrità registrati. Per eseguire un subset dei controlli di integrità, specificare una funzione che filtra il set di controlli. Il predicato viene valutato per ogni periodo.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Timeout>: Timeout per l'esecuzione dei controlli di integrità per tutte le <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> istanze. Usare <xref:System.Threading.Timeout.InfiniteTimeSpan> per l'esecuzione senza un timeout. Il valore predefinito è 30 secondi.

> [!WARNING]
> Nella versione 2.2 di ASP.NET Core, l'impostazione <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period> non viene rispettata dall'implementazione <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. Viene impostato il valore <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>. Questo problema è stato risolto in ASP.NET Core 3,0.

Nell'app di esempio, `ReadinessPublisher` è un'implementazione di <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. Lo stato di controllo integrità viene registrato per ogni controllo come:

* Information ( <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> ) se lo stato dei controlli di integrità è <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Healthy> .
* Error ( <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogError*> ) se lo stato è <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Degraded> o <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Unhealthy> .

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/ReadinessPublisher.cs?name=snippet_ReadinessPublisher&highlight=18-27)]

Nell'esempio di `LivenessProbeStartup` dell'app di esempio, il controllo di conformità `StartupHostedService` ha un ritardo di avvio di due secondi e il controllo viene eseguito ogni 30 secondi. Per attivare l'implementazione <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>, l'esempio registra `ReadinessPublisher` come servizio singleton nel contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices&highlight=12-17,28)]

> [!NOTE]
> La soluzione alternativa seguente consente di aggiungere un'istanza <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> al contenitore del servizio quando uno o più servizi ospitati sono già stati aggiunti all'app. Questa soluzione alternativa non è necessaria in ASP.NET Core 3,0.
>
> ```csharp
> private const string HealthCheckServiceAssembly =
>     "Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherHostedService";
>
> services.TryAddEnumerable(
>     ServiceDescriptor.Singleton(typeof(IHostedService),
>         typeof(HealthCheckPublisherOptions).Assembly
>             .GetType(HealthCheckServiceAssembly)));
> ```

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) include i server di pubblicazione per diversi sistemi, tra cui [Application Insights](/azure/application-insights/app-insights-overview).
>
> [AspNetCore. Diagnostics. HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) non è gestito né supportato da Microsoft.

## <a name="restrict-health-checks-with-mapwhen"></a>Limitare i controlli integrità con MapWhen

Usare <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> per creare un ramo condizionale della pipeline delle richieste per gli endpoint di controllo integrità.

Nell'esempio seguente, `MapWhen` la pipeline della richiesta viene diramata per attivare il middleware dei controlli di integrità se viene ricevuta una richiesta GET per l' `api/HealthCheck` endpoint:

```csharp
app.MapWhen(
    context => context.Request.Method == HttpMethod.Get.Method && 
        context.Request.Path.StartsWith("/api/HealthCheck"),
    builder => builder.UseHealthChecks());

app.UseMvc();
```

Per altre informazioni, vedere <xref:fundamentals/middleware/index#use-run-and-map>.

::: moniker-end
