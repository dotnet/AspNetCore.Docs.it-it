---
title: Modello di opzioni in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare il modello di opzioni per rappresentare gruppi di opzioni correlate nelle app ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/20/2020
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
uid: fundamentals/configuration/options
ms.openlocfilehash: cb78147050ebdafc7de4ad150ae2644689d6efbc
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586215"
---
# <a name="options-pattern-in-aspnet-core"></a>Modello di opzioni in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Kirk Larkin](https://twitter.com/serpent5) e [Rick Anderson](https://twitter.com/RickAndMSFT).

Il modello options usa le classi per fornire accesso fortemente tipizzato ai gruppi di impostazioni correlate. Quando le [impostazioni di configurazione](xref:fundamentals/configuration/index) vengono isolate in base allo scenario in classi separate, l'app aderisce a due importanti principi di progettazione del software:

* Il [principio di separazione dell'interfaccia (ISP) o l'incapsulamento](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): scenari (classi) che dipendono dalle impostazioni di configurazione dipendono solo dalle impostazioni di configurazione che usano.
* [Separazione delle competenze (Separation of Concerns, SoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): le impostazioni di parti diverse dell'app non sono dipendenti o accoppiate l'una con l'altra.

Le opzioni offrono anche un meccanismo per convalidare i dati di configurazione. Per altre informazioni, vedere la sezione [Opzioni di convalida](#options-validation).

In questo argomento vengono fornite informazioni sul modello di opzioni in ASP.NET Core. Per informazioni sull'uso del modello di opzioni nelle app console, vedere [modello di opzioni in .NET](/dotnet/core/extensions/options).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

<a name="optpat"></a>

## <a name="bind-hierarchical-configuration"></a>Associa configurazione gerarchica

[!INCLUDE[](~/includes/bind.md)]

<a name="oi"></a>

## <a name="options-interfaces"></a>Interfacce per le opzioni

<xref:Microsoft.Extensions.Options.IOptions%601>:

* Non ***supporta:***
  * Lettura dei dati di configurazione dopo l'avvio dell'app.
  * [Opzioni denominate](#named)
* Viene registrato come [singleton](xref:fundamentals/dependency-injection#singleton) e può essere inserito in qualsiasi durata del [servizio](xref:fundamentals/dependency-injection#service-lifetimes).

<xref:Microsoft.Extensions.Options.IOptionsSnapshot%601>:

* È utile negli scenari in cui le opzioni devono essere ricalcolate a ogni richiesta. Per altre informazioni, vedere [usare IOptionsSnapshot per leggere i dati aggiornati](#ios).
* Viene registrato come [ambito](xref:fundamentals/dependency-injection#scoped) e pertanto non può essere inserito in un servizio singleton.
* Supporta le [opzioni denominate](#named)

<xref:Microsoft.Extensions.Options.IOptionsMonitor%601>:

* Viene usato per recuperare le opzioni e gestire le notifiche delle opzioni per le `TOptions` istanze.
* Viene registrato come [singleton](xref:fundamentals/dependency-injection#singleton) e può essere inserito in qualsiasi durata del [servizio](xref:fundamentals/dependency-injection#service-lifetimes).
* Supporto:
  * Notifiche di modifica
  * [Opzioni denominate](#named-options-support-with-iconfigurenamedoptions)
  * [Configurazione ricaricabile](#ios)
  * Invalidamento selettivo di opzioni (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)
  
Gli scenari [di post-configurazione](#options-post-configuration) consentono di impostare o modificare le opzioni dopo la <xref:Microsoft.Extensions.Options.IConfigureOptions%601> configurazione.

<xref:Microsoft.Extensions.Options.IOptionsFactory%601> è responsabile della creazione di nuove istanze di opzioni. Include un singolo metodo <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*>. L'implementazione predefinita accetta tutte le interfacce <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> registrate ed esegue tutte le configurazioni, seguite dalla post-configurazione. Fa distinzione tra <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e chiama solo l'interfaccia appropriata.

<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> viene usata da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per memorizzare nella cache le istanze di `TOptions`. <xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalida le istanze delle opzioni nel monitoraggio in modo che il valore venga ricalcolato (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>). I valori possono essere introdotti manualmente con <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>. Il metodo <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> viene usato quando tutte le istanze denominate devono essere ricreate su richiesta.

<a name="ios"></a>

## <a name="use-ioptionssnapshot-to-read-updated-data"></a>Usare IOptionsSnapshot per leggere i dati aggiornati

Usando <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> , le opzioni vengono calcolate una volta per ogni richiesta in caso di accesso e memorizzati nella cache per la durata della richiesta. Le modifiche alla configurazione vengono lette dopo l'avvio dell'app quando si usano i provider di configurazione che supportano la lettura dei valori di configurazione aggiornati.

La differenza tra `IOptionsMonitor` e `IOptionsSnapshot` è che:

* `IOptionsMonitor` è un [servizio singleton](xref:fundamentals/dependency-injection#singleton) che recupera i valori correnti delle opzioni in qualsiasi momento, operazione particolarmente utile nelle dipendenze singleton.
* `IOptionsSnapshot` è un [servizio con ambito](xref:fundamentals/dependency-injection#scoped) e fornisce uno snapshot delle opzioni nel momento in cui `IOptionsSnapshot<T>` viene costruito l'oggetto. Gli snapshot delle opzioni sono progettati per l'uso con dipendenze temporanee e con ambito.

Il codice seguente usa <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> .

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/TestSnap.cshtml.cs?name=snippet)]

Il codice seguente registra un'istanza di configurazione `MyOptions` con cui viene eseguita l'associazione:

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup3.cs?name=snippet_Example2)]

Nel codice precedente, le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app vengono lette.

## <a name="ioptionsmonitor"></a>IOptionsMonitor

Il codice seguente registra un'istanza di configurazione `MyOptions` con cui viene eseguita l'associazione.

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup3.cs?name=snippet_Example2)]

Nell'esempio seguente viene usato <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>:

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/TestMonitor.cshtml.cs?name=snippet)]

Nel codice precedente, per impostazione predefinita, vengono lette le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app.

<a name="named"></a>

## <a name="named-options-support-using-iconfigurenamedoptions"></a>Supporto delle opzioni denominate con IConfigureNamedOptions

Opzioni denominate:

* Sono utili quando più sezioni di configurazione sono associate alle stesse proprietà.
* Distinzione tra maiuscole e minuscole.

Si consideri il seguente *appsettings.json* file:

[!code-json[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/appsettings.NO.json)]

Anziché creare due classi da associare `TopItem:Month` e `TopItem:Year` , per ogni sezione viene usata la classe seguente:

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Models/TopItemSettings.cs)]

Il codice seguente configura le opzioni denominate:

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/StartupNO.cs?name=snippet_Example2)]

Il codice seguente consente di visualizzare le opzioni denominate:

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/TestNO.cshtml.cs?name=snippet)]

Tutte le opzioni sono istanze denominate. <xref:Microsoft.Extensions.Options.IConfigureOptions%601> le istanze vengono considerate come destinazione dell' `Options.DefaultName` istanza, ovvero `string.Empty` . <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> implementa anche <xref:Microsoft.Extensions.Options.IConfigureOptions%601>. L'implementazione predefinita di <xref:Microsoft.Extensions.Options.IOptionsFactory%601> include la logica per usarle in modo appropriato. L' `null` opzione denominata viene utilizzata per fare riferimento a tutte le istanze denominate anziché a un'istanza denominata specifica. <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> e <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> usano questa convenzione.

## <a name="optionsbuilder-api"></a>API OptionsBuilder

<xref:Microsoft.Extensions.Options.OptionsBuilder%601> viene usata per configurare le istanze di `TOptions`. `OptionsBuilder` semplifica la creazione di opzioni denominate perché è costituita da un singolo parametro nella chiamata iniziale ad `AddOptions<TOptions>(string optionsName)` invece di essere visualizzata in tutte le chiamate successive. La convalida delle opzioni e gli overload `ConfigureOptions` che accettano le dipendenze dei servizi sono disponibili solo tramite `OptionsBuilder`.

`OptionsBuilder` viene usato nella sezione [convalida opzioni](#val) .

## <a name="use-di-services-to-configure-options"></a>Usare i servizi di inserimento delle dipendenze per configurare le opzioni

È possibile accedere ai servizi dall'inserimento delle dipendenze durante la configurazione delle opzioni in due modi:

* Passare un delegato di configurazione da [configurare](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) in [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1). `OptionsBuilder<TOptions>` fornisce overload di [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) che consentono l'uso di un massimo di cinque servizi per configurare le opzioni:

  ```csharp
  services.AddOptions<MyOptions>("optionalName")
      .Configure<Service1, Service2, Service3, Service4, Service5>(
          (o, s, s2, s3, s4, s5) => 
              o.Property = DoSomethingWith(s, s2, s3, s4, s5));
  ```

* Creare un tipo che implementi <xref:Microsoft.Extensions.Options.IConfigureOptions%601> o <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e registri il tipo come servizio.

È consigliabile passare un delegato di configurazione a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) perché la creazione di un servizio è più complessa. La creazione di un tipo è equivalente a quella eseguita dal framework quando si chiama [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*). La chiamata a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registra un'interfaccia <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> generica temporanea, che ha un costruttore che accetta i tipi di servizi generici specificati. 

<a name="val"></a>

## <a name="options-validation"></a>Convalida delle opzioni

La convalida delle opzioni consente di convalidare i valori dell'opzione.

Si consideri il seguente *appsettings.json* file:

[!code-json[](~/fundamentals/configuration/options/samples/3.x/OptionsValidationSample/appsettings.Dev2.json)]

La classe seguente viene associata alla `"MyConfig"` sezione di configurazione e applica due `DataAnnotations` regole:

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Configuration/MyConfigOptions.cs?name=snippet)]

Il codice seguente:

* Chiama <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions%2A> per ottenere un [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1) associato alla `MyConfigOptions` classe.
* Chiama <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations%2A> per abilitare la convalida usando `DataAnnotations` .

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Startup.cs?name=snippet)]

Il `ValidateDataAnnotations` metodo di estensione è definito nel pacchetto NuGet [Microsoft. Extensions. Options. DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) . Per le app Web che usano l' `Microsoft.NET.Sdk.Web` SDK, a questo pacchetto viene fatto riferimento in modo implicito dal Framework condiviso.

Nel codice seguente vengono visualizzati i valori di configurazione o gli errori di convalida:

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Controllers/HomeController.cs?name=snippet)]

Il codice seguente applica una regola di convalida più complessa usando un delegato:

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Startup2.cs?name=snippet)]

### <a name="ivalidateoptions-for-complex-validation"></a>IValidateOptions per la convalida complessa

La classe seguente implementa <xref:Microsoft.Extensions.Options.IValidateOptions`1> :

[!code-csharp[](options/samples/3.x/OptionsValidationSample/Configuration/MyConfigValidation.cs?name=snippet)]

`IValidateOptions` consente di trasferire il codice di convalida da `StartUp` e in una classe.

Utilizzando il codice precedente, la convalida è abilitata in `Startup.ConfigureServices` con il codice seguente:

[!code-csharp[](options/samples/3.x/OptionsValidationSample/StartupValidation.cs?name=snippet)]

<!-- The following comment doesn't seem that useful 
Options validation doesn't guard against options modifications after the options instance is created. For example, `IOptionsSnapshot` options are created and validated once per request when the options are first accessed. The `IOptionsSnapshot` options aren't validated again on subsequent access attempts *for the same request*.

The `Validate` method accepts a `Func<TOptions, bool>`. To fully customize validation, implement `IValidateOptions<TOptions>`, which allows:

* Validation of multiple options types: `class ValidateTwo : IValidateOptions<Option1>, IValidationOptions<Option2>`
* Validation that depends on another option type: `public DependsOnAnotherOptionValidator(IOptionsMonitor<AnotherOption> options)`

`IValidateOptions` validates:

* A specific named options instance.
* All options when `name` is `null`.

Return a `ValidateOptionsResult` from your implementation of the interface:

```csharp
public interface IValidateOptions<TOptions> where TOptions : class
{
    ValidateOptionsResult Validate(string name, TOptions options);
}
```

Data Annotation-based validation is available from the [Microsoft.Extensions.Options.DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) package by calling the <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations*> method on `OptionsBuilder<TOptions>`. `Microsoft.Extensions.Options.DataAnnotations` is implicitly referenced in ASP.NET Core apps.

-->

## <a name="options-post-configuration"></a>Post-configurazione delle opzioni

Impostare la post-configurazione con <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>. La post-configurazione viene eseguita dopo il completamento della configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>:

```csharp
services.PostConfigure<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> è disponibile per la post-configurazione delle opzioni denominate:

```csharp
services.PostConfigure<MyOptions>("named_options_1", myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

Usare <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> per la post-configurazione di tutte le istanze di configurazione:

```csharp
services.PostConfigureAll<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

## <a name="accessing-options-during-startup"></a>Accesso alle opzioni durante l'avvio

<xref:Microsoft.Extensions.Options.IOptions%601> e <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> possono essere usate in `Startup.Configure`, perché i servizi vengono compilati prima dell'esecuzione del metodo `Configure`.

```csharp
public void Configure(IApplicationBuilder app, 
    IOptionsMonitor<MyOptions> optionsAccessor)
{
    var option1 = optionsAccessor.CurrentValue.Option1;
}
```

Non usare <xref:Microsoft.Extensions.Options.IOptions%601> oppure <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`. Le opzioni potrebbero avere uno stato incoerente a causa dell'ordinamento delle registrazioni dei servizi.

## <a name="optionsconfigurationextensions-nuget-package"></a>Options.Configpacchetto NuGet urationExtensions

Il pacchetto [ urationExtensionsMicrosoft.Extensions.Options.Config](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/) viene fatto riferimento in modo implicito nelle app ASP.NET Core.

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Il modello di opzioni usa le classi per rappresentare i gruppi di impostazioni correlate. Quando le [impostazioni di configurazione](xref:fundamentals/configuration/index) vengono isolate in base allo scenario in classi separate, l'app aderisce a due importanti principi di progettazione del software:

* Il [principio di separazione dell'interfaccia (ISP) o l'incapsulamento](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): scenari (classi) che dipendono dalle impostazioni di configurazione dipendono solo dalle impostazioni di configurazione che usano.
* [Separazione delle competenze (Separation of Concerns, SoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): le impostazioni di parti diverse dell'app non sono dipendenti o accoppiate l'una con l'altra.

Le opzioni offrono anche un meccanismo per convalidare i dati di configurazione. Per altre informazioni, vedere la sezione [Opzioni di convalida](#options-validation).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) oppure aggiungere un riferimento al pacchetto [Microsoft.Extensions.Options.ConfigurationExtensions](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/).

## <a name="options-interfaces"></a>Interfacce per le opzioni

<xref:Microsoft.Extensions.Options.IOptionsMonitor%601> consente di recuperare le opzioni e gestire le notifiche di opzioni per le istanze `TOptions`. <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> supporta gli scenari seguenti:

* Notifiche di modifica
* [Opzioni denominate](#named-options-support-with-iconfigurenamedoptions)
* [Configurazione ricaricabile](#reload-configuration-data-with-ioptionssnapshot)
* Invalidamento selettivo di opzioni (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)

Gli scenari di [post-configurazione](#options-post-configuration) consentono di impostare o modificare le opzioni dopo la configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.

<xref:Microsoft.Extensions.Options.IOptionsFactory%601> è responsabile della creazione di nuove istanze di opzioni. Include un singolo metodo <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*>. L'implementazione predefinita accetta tutte le interfacce <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> registrate ed esegue tutte le configurazioni, seguite dalla post-configurazione. Fa distinzione tra <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e chiama solo l'interfaccia appropriata.

<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> viene usata da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per memorizzare nella cache le istanze di `TOptions`. <xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalida le istanze delle opzioni nel monitoraggio in modo che il valore venga ricalcolato (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>). I valori possono essere introdotti manualmente con <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>. Il metodo <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> viene usato quando tutte le istanze denominate devono essere ricreate su richiesta.

<xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> è utile negli scenari in cui le opzioni devono essere ricalcolate a ogni richiesta. Per altre informazioni, vedere la sezione [Ricaricare i dati di configurazione con IOptionsSnapshot](#reload-configuration-data-with-ioptionssnapshot).

<xref:Microsoft.Extensions.Options.IOptions%601> può essere usata a supporto delle opzioni. Tuttavia <xref:Microsoft.Extensions.Options.IOptions%601> non supporta gli scenari precedenti di <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>. È possibile continuare a usare <xref:Microsoft.Extensions.Options.IOptions%601> in framework e librerie esistenti che già usano l'interfaccia <xref:Microsoft.Extensions.Options.IOptions%601> e non richiedono gli scenari forniti da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.

## <a name="general-options-configuration"></a>Configurazione delle opzioni generali

La configurazione delle opzioni generali è illustrata nell'Esempio 1 nell'app di esempio.

Una classe di opzioni deve essere non astratta con un costruttore pubblico senza parametri. La classe seguente, `MyOptions`, ha due proprietà, `Option1` e `Option2`. Sebbene l'impostazione dei valori predefiniti sia facoltativa, il costruttore della classe nell'esempio seguente imposta il valore predefinito di `Option1`. `Option2` ha un valore impostato tramite l'inizializzazione diretta della proprietà (*Models/MyOptions.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptions.cs?name=snippet1)]

La classe `MyOptions` viene aggiunta al contenitore di servizi con <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> e associata alla configurazione:

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example1)]

Il modello di pagina seguente usa l'[inserimento delle dipendenze del costruttore](xref:mvc/controllers/dependency-injection) con <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per accedere alle impostazioni (*Pages/Index.cshtml.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example1)]

Il file dell'esempio *appsettings.json* specifica i valori per `option1` e `option2` :

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=2-3)]

Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:

```html
option1 = value1_from_json, option2 = -1
```

> [!NOTE]
> Se si usa un <xref:System.Configuration.ConfigurationBuilder> personalizzato per caricare la configurazione delle opzioni da un file di impostazioni, verificare che il percorso base sia impostato correttamente:
>
> ```csharp
> var configBuilder = new ConfigurationBuilder()
>    .SetBasePath(Directory.GetCurrentDirectory())
>    .AddJsonFile("appsettings.json", optional: true);
> var config = configBuilder.Build();
>
> services.Configure<MyOptions>(config);
> ```
>
> L'impostazione esplicita del percorso base non è necessaria se si carica la configurazione delle opzioni dal file di impostazioni tramite <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.

## <a name="configure-simple-options-with-a-delegate"></a>Configurare opzioni semplici con un delegato

La configurazione di opzioni semplici con un delegato è illustrata nell'Esempio 2 dell'app di esempio.

Usare un delegato per impostare i valori delle opzioni. L'app di esempio usa la classe `MyOptionsWithDelegateConfig` (*Models/MyOptionsWithDelegateConfig.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptionsWithDelegateConfig.cs?name=snippet1)]

Nel codice seguente viene aggiunto un secondo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi. Viene usato un delegato per configurare l'associazione a `MyOptionsWithDelegateConfig`:

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example2)]

*Index.cshtml.cs*:

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3,9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example2)]

È possibile aggiungere più provider di configurazione. I provider di configurazione sono disponibili dai pacchetti NuGet e vengono applicati nell'ordine in cui sono registrati. Per altre informazioni, vedere <xref:fundamentals/configuration/index>.

Ogni chiamata a <xref:Microsoft.Extensions.Options.IConfigureOptions%601.Configure*> aggiunge un servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi. Nell'esempio precedente, i valori di `Option1` e `Option2` sono entrambi specificati in *appsettings.json* , ma i valori di `Option1` e `Option2` vengono sottoposti a override dal delegato configurato.

Quando sono abilitati più servizi di configurazione, l'origine di configurazione più recente specificata *ha priorità* e imposta il valore di configurazione. Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:

```html
delegate_option1 = value1_configured_by_delegate, delegate_option2 = 500
```

## <a name="suboptions-configuration"></a>Configurazione delle opzioni secondarie

La configurazione delle opzioni secondarie è illustrata nell'Esempio 3 dell'app di esempio.

Le app devono creare classi di opzioni che riguardano gruppi di scenari (classi) specifici nell'app. Le parti dell'app che richiedono valori di configurazione devono avere accesso solo ai valori di configurazione necessari.

Durante l'associazione delle opzioni alla configurazione, ogni proprietà del tipo di opzioni viene associata a una chiave di configurazione con formato `property[:sub-property:]`. Ad esempio, la `MyOptions.Option1` proprietà è associata alla chiave `Option1` , che viene letta dalla `option1` Proprietà in *appsettings.json* .

Nel codice seguente viene aggiunto un terzo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi. Viene associato `MySubOptions` alla sezione `subsection` del *appsettings.json* file:

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example3)]

Il `GetSection` metodo richiede lo <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> spazio dei nomi.

Il file dell'esempio *appsettings.json* definisce un `subsection` membro con chiavi per `suboption1` e `suboption2` :

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=4-7)]

La classe `MySubOptions` definisce le proprietà `SubOption1` e `SubOption2` per contenere i valori delle opzioni (*Models/MySubOptions.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MySubOptions.cs?name=snippet1)]

Il metodo `OnGet` del modello di pagina restituisce una stringa con i valori delle opzioni (*Pages/Index.cshtml.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=4,10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example3)]

Quando viene eseguita l'app, il metodo `OnGet` restituisce una stringa che mostra i valori delle classi delle opzioni secondarie:

```html
subOption1 = subvalue1_from_json, subOption2 = 200
```

## <a name="options-injection"></a>Inserimento di opzioni

L'inserimento di opzioni viene illustrato come esempio 4 nell'app di esempio.

Inserisci <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in:

* Una Razor pagina o una visualizzazione MVC con la [`@inject`](xref:mvc/views/razor#inject) Razor direttiva.
* Modello di pagina o di visualizzazione.

L'esempio seguente dall'app di esempio inserisce <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> un modello di pagina (*pages/index. cshtml. cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example4)]

L'app di esempio mostra come inserire `IOptionsMonitor<MyOptions>` con una direttiva `@inject`:

[!code-cshtml[](options/samples/2.x/OptionsSample/Pages/Index.cshtml?range=1-10&highlight=4)]

Quando l'app viene eseguita, i valori delle opzioni sono visibili nella pagina di cui è stato eseguito il rendering:

![I valori delle opzioni Option1: value1_from_json e Option2: -1 vengono caricati dal modello e tramite inserimento nella visualizzazione.](options/_static/view.png)

## <a name="reload-configuration-data-with-ioptionssnapshot"></a>Ricaricare i dati di configurazione con IOptionsSnapshot

Il ricaricamento dei dati di configurazione con <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> viene illustrato nell'esempio 5 nell'app di esempio.

Usando <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> , le opzioni vengono calcolate una volta per ogni richiesta in caso di accesso e memorizzati nella cache per la durata della richiesta.

La differenza tra `IOptionsMonitor` e `IOptionsSnapshot` è che:

* `IOptionsMonitor` è un [servizio singleton](xref:fundamentals/dependency-injection#singleton) che recupera i valori correnti delle opzioni in qualsiasi momento, operazione particolarmente utile nelle dipendenze singleton.
* `IOptionsSnapshot` è un [servizio con ambito](xref:fundamentals/dependency-injection#scoped) e fornisce uno snapshot delle opzioni nel momento in cui `IOptionsSnapshot<T>` viene costruito l'oggetto. Gli snapshot delle opzioni sono progettati per l'uso con dipendenze temporanee e con ambito.

Nell'esempio seguente viene illustrato come viene creato un nuovo oggetto <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> dopo *appsettings.json* le modifiche (*pages/index. cshtml. cs*). Più richieste al server restituiscono valori costanti forniti dal *appsettings.json* file fino a quando il file non viene modificato e la configurazione viene ricaricata.

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=12)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=5,11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example5)]

Nell'immagine seguente vengono illustrati `option1` i `option2` valori iniziali e caricati dal *appsettings.json* file:

```html
snapshot option1 = value1_from_json, snapshot option2 = -1
```

Modificare i valori nel *appsettings.json* file in `value1_from_json UPDATED` e `200` . Salvare il file *appsettings.json*. Aggiornare il browser per visualizzare i valori delle opzioni aggiornati:

```html
snapshot option1 = value1_from_json UPDATED, snapshot option2 = 200
```

## <a name="named-options-support-with-iconfigurenamedoptions"></a>Supporto delle opzioni denominate con IConfigureNamedOptions

Il supporto delle opzioni denominate con <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> è illustrato nell'Esempio 6 nell'app di esempio.

Il supporto delle opzioni denominate consente all'app di distinguere le configurazioni delle opzioni denominate. Nell'app di esempio le opzioni denominate sono dichiarate con [OptionsServiceCollectionExtensions.ConfigUre](xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure*), che chiama [ConfigureNamedOptions \<TOptions> . Configurare](xref:Microsoft.Extensions.Options.ConfigureNamedOptions`1.Configure*) il metodo di estensione. Le opzioni denominate fanno distinzione maiuscole/minuscole

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example6)]

L'app di esempio accede alle opzioni denominate con <xref:Microsoft.Extensions.Options.IOptionsSnapshot`1.Get*> (*Pages/Index.cshtml.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=13-14)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=6,12-13)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example6)]

Eseguendo l'app di esempio, vengono restituite le opzioni denominate:

```html
named_options_1: option1 = value1_from_json, option2 = -1
named_options_2: option1 = named_options_2_value1_from_action, option2 = 5
```

`named_options_1` i valori vengono forniti dalla configurazione, che vengono caricati dal *appsettings.json* file. I valori `named_options_2` sono specificati da:

* Delegato `named_options_2` in `ConfigureServices` per `Option1`.
* Valore predefinito per `Option2` specificato dalla classe `MyOptions`.

## <a name="configure-all-options-with-the-configureall-method"></a>Configurare tutte le opzioni con il metodo ConfigureAll

Configurare tutte le istanze delle opzioni con il metodo <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*>. Il codice seguente configura `Option1` per tutte le istanze di configurazione con un valore comune. Aggiungere manualmente il codice seguente al metodo `Startup.ConfigureServices`:

```csharp
services.ConfigureAll<MyOptions>(myOptions => 
{
    myOptions.Option1 = "ConfigureAll replacement value";
});
```

L'esecuzione dell'app di esempio dopo l'aggiunta del codice produce il risultato seguente:

```html
named_options_1: option1 = ConfigureAll replacement value, option2 = -1
named_options_2: option1 = ConfigureAll replacement value, option2 = 5
```

> [!NOTE]
> Tutte le opzioni sono istanze denominate. Le istanze di <xref:Microsoft.Extensions.Options.IConfigureOptions%601> esistenti sono trattate come se fossero indirizzate all'istanza di `Options.DefaultName`, ovvero `string.Empty`. <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> implementa anche <xref:Microsoft.Extensions.Options.IConfigureOptions%601>. L'implementazione predefinita di <xref:Microsoft.Extensions.Options.IOptionsFactory%601> include la logica per usarle in modo appropriato. L'opzione denominata `null` viene usata per avere come destinazione tutte le istanze denominate anziché un'istanza specifica denominata (questa convenzione è usata da <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> e <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*>).

## <a name="optionsbuilder-api"></a>API OptionsBuilder

<xref:Microsoft.Extensions.Options.OptionsBuilder%601> viene usata per configurare le istanze di `TOptions`. `OptionsBuilder` semplifica la creazione di opzioni denominate perché è costituita da un singolo parametro nella chiamata iniziale ad `AddOptions<TOptions>(string optionsName)` invece di essere visualizzata in tutte le chiamate successive. La convalida delle opzioni e gli overload `ConfigureOptions` che accettano le dipendenze dei servizi sono disponibili solo tramite `OptionsBuilder`.

```csharp
// Options.DefaultName = "" is used.
services.AddOptions<MyOptions>().Configure(o => o.Property = "default");

services.AddOptions<MyOptions>("optionalName")
    .Configure(o => o.Property = "named");
```

## <a name="use-di-services-to-configure-options"></a>Usare i servizi di inserimento delle dipendenze per configurare le opzioni

È possibile accedere ad altri servizi dall'inserimento delle dipendenze durante la configurazione delle opzioni in due modi:

* Passare un delegato di configurazione da [configurare](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) in [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1). [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1) fornisce overload di [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) che consentono di usare fino a cinque servizi per configurare le opzioni:

  ```csharp
  services.AddOptions<MyOptions>("optionalName")
      .Configure<Service1, Service2, Service3, Service4, Service5>(
          (o, s, s2, s3, s4, s5) => 
              o.Property = DoSomethingWith(s, s2, s3, s4, s5));
  ```

* Creare un tipo personalizzato che implementa <xref:Microsoft.Extensions.Options.IConfigureOptions%601> o <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e registrare il tipo come servizio.

È consigliabile passare un delegato di configurazione a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) perché la creazione di un servizio è più complessa. La creazione di un tipo personalizzato equivale alle operazioni che il framework esegue automaticamente quando si usa [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*). La chiamata a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registra un'interfaccia <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> generica temporanea, che ha un costruttore che accetta i tipi di servizi generici specificati. 

## <a name="options-validation"></a>Convalida delle opzioni

La convalida le opzioni consente di convalidare le opzioni quando vengono configurate. Chiamare `Validate` con un metodo di convalida che restituisce `true` se le opzioni sono valide e `false` se non sono valide:

```csharp
// Registration
services.AddOptions<MyOptions>("optionalOptionsName")
    .Configure(o => { }) // Configure the options
    .Validate(o => YourValidationShouldReturnTrueIfValid(o), 
        "custom error");

// Consumption
var monitor = services.BuildServiceProvider()
    .GetService<IOptionsMonitor<MyOptions>>();
  
try
{
    var options = monitor.Get("optionalOptionsName");
}
catch (OptionsValidationException e) 
{
   // e.OptionsName returns "optionalOptionsName"
   // e.OptionsType returns typeof(MyOptions)
   // e.Failures returns a list of errors, which would contain 
   //     "custom error"
}
```

L'esempio precedente imposta l'istanza di opzioni denominata su `optionalOptionsName`. L'istanza di opzioni predefinita è `Options.DefaultName`.

La convalida viene eseguita quando viene creata l'istanza di opzioni. Si garantisce che un'istanza di opzioni superi la convalida la prima volta che si accede.

> [!IMPORTANT]
> La convalida delle opzioni non protegge le modifiche delle opzioni dopo la creazione dell'istanza di Options. Le opzioni, ad esempio, `IOptionsSnapshot` vengono create e convalidate una volta per ogni richiesta quando si accede per la prima volta alle opzioni. Le `IOptionsSnapshot` Opzioni non vengono convalidate nuovamente ai tentativi *di accesso successivi per la stessa richiesta*.

Il metodo `Validate` accetta `Func<TOptions, bool>`. Per personalizzare completamente la convalida, implementare `IValidateOptions<TOptions>`, che consente:

* La convalida di più tipi di opzioni: `class ValidateTwo : IValidateOptions<Option1>, IValidationOptions<Option2>`
* La convalida che dipende da un altro tipo di opzione: `public DependsOnAnotherOptionValidator(IOptionsMonitor<AnotherOption> options)`

`IValidateOptions` convalida:

* Un'istanza di opzioni denominata specifica.
* Tutte le opzioni quando `name` è `null`.

Restituire `ValidateOptionsResult` dall'implementazione dell'interfaccia:

```csharp
public interface IValidateOptions<TOptions> where TOptions : class
{
    ValidateOptionsResult Validate(string name, TOptions options);
}
```

La convalida basata sull'annotazione dei dati è disponibile dal pacchetto [Microsoft. Extensions. Options. DataAnnotations](https://www.nuget.org/packages/Microsoft.Extensions.Options.DataAnnotations) chiamando il <xref:Microsoft.Extensions.DependencyInjection.OptionsBuilderDataAnnotationsExtensions.ValidateDataAnnotations*> metodo su `OptionsBuilder<TOptions>` . `Microsoft.Extensions.Options.DataAnnotations` è incluso nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).

```csharp
using Microsoft.Extensions.DependencyInjection;

private class AnnotatedOptions
{
    [Required]
    public string Required { get; set; }

    [StringLength(5, ErrorMessage = "Too long.")]
    public string StringLength { get; set; }

    [Range(-5, 5, ErrorMessage = "Out of range.")]
    public int IntRange { get; set; }
}

[Fact]
public void CanValidateDataAnnotations()
{
    var services = new ServiceCollection();
    services.AddOptions<AnnotatedOptions>()
        .Configure(o =>
        {
            o.StringLength = "111111";
            o.IntRange = 10;
            o.Custom = "nowhere";
        })
        .ValidateDataAnnotations();

    var sp = services.BuildServiceProvider();

    var error = Assert.Throws<OptionsValidationException>(() => 
        sp.GetRequiredService<IOptionsMonitor<AnnotatedOptions>>().CurrentValue);
    ValidateFailure<AnnotatedOptions>(error, Options.DefaultName, 1,
        "DataAnnotation validation failed for members Required " +
            "with the error 'The Required field is required.'.",
        "DataAnnotation validation failed for members StringLength " +
            "with the error 'Too long.'.",
        "DataAnnotation validation failed for members IntRange " +
            "with the error 'Out of range.'.");
}
```

La convalida eager (fallimento e risposta immediata agli errori all'avvio) è pianificata per una versione futura.

## <a name="options-post-configuration"></a>Post-configurazione delle opzioni

Impostare la post-configurazione con <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>. La post-configurazione viene eseguita dopo il completamento della configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>:

```csharp
services.PostConfigure<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> è disponibile per la post-configurazione delle opzioni denominate:

```csharp
services.PostConfigure<MyOptions>("named_options_1", myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

Usare <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> per la post-configurazione di tutte le istanze di configurazione:

```csharp
services.PostConfigureAll<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

## <a name="accessing-options-during-startup"></a>Accesso alle opzioni durante l'avvio

<xref:Microsoft.Extensions.Options.IOptions%601> e <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> possono essere usate in `Startup.Configure`, perché i servizi vengono compilati prima dell'esecuzione del metodo `Configure`.

```csharp
public void Configure(IApplicationBuilder app, IOptionsMonitor<MyOptions> optionsAccessor)
{
    var option1 = optionsAccessor.CurrentValue.Option1;
}
```

Non usare <xref:Microsoft.Extensions.Options.IOptions%601> oppure <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`. Le opzioni potrebbero avere uno stato incoerente a causa dell'ordinamento delle registrazioni dei servizi.

::: moniker-end

::: moniker range="= aspnetcore-2.1"

Il modello di opzioni usa le classi per rappresentare i gruppi di impostazioni correlate. Quando le [impostazioni di configurazione](xref:fundamentals/configuration/index) vengono isolate in base allo scenario in classi separate, l'app aderisce a due importanti principi di progettazione del software:

* Il [principio di separazione dell'interfaccia (ISP) o l'incapsulamento](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#encapsulation): scenari (classi) che dipendono dalle impostazioni di configurazione dipendono solo dalle impostazioni di configurazione che usano.
* [Separazione delle competenze (Separation of Concerns, SoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns): le impostazioni di parti diverse dell'app non sono dipendenti o accoppiate l'una con l'altra.

Le opzioni offrono anche un meccanismo per convalidare i dati di configurazione. Per altre informazioni, vedere la sezione [Opzioni di convalida](#options-validation).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/configuration/options/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) oppure aggiungere un riferimento al pacchetto [Microsoft.Extensions.Options.ConfigurationExtensions](https://www.nuget.org/packages/Microsoft.Extensions.Options.ConfigurationExtensions/).

## <a name="options-interfaces"></a>Interfacce per le opzioni

<xref:Microsoft.Extensions.Options.IOptionsMonitor%601> consente di recuperare le opzioni e gestire le notifiche di opzioni per le istanze `TOptions`. <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> supporta gli scenari seguenti:

* Notifiche di modifica
* [Opzioni denominate](#named-options-support-with-iconfigurenamedoptions)
* [Configurazione ricaricabile](#reload-configuration-data-with-ioptionssnapshot)
* Invalidamento selettivo di opzioni (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601>)

Gli scenari di [post-configurazione](#options-post-configuration) consentono di impostare o modificare le opzioni dopo la configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>.

<xref:Microsoft.Extensions.Options.IOptionsFactory%601> è responsabile della creazione di nuove istanze di opzioni. Include un singolo metodo <xref:Microsoft.Extensions.Options.IOptionsFactory`1.Create*>. L'implementazione predefinita accetta tutte le interfacce <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601> registrate ed esegue tutte le configurazioni, seguite dalla post-configurazione. Fa distinzione tra <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e <xref:Microsoft.Extensions.Options.IConfigureOptions%601> e chiama solo l'interfaccia appropriata.

<xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> viene usata da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per memorizzare nella cache le istanze di `TOptions`. <xref:Microsoft.Extensions.Options.IOptionsMonitorCache%601> invalida le istanze delle opzioni nel monitoraggio in modo che il valore venga ricalcolato (<xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryRemove*>). I valori possono essere introdotti manualmente con <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.TryAdd*>. Il metodo <xref:Microsoft.Extensions.Options.IOptionsMonitorCache`1.Clear*> viene usato quando tutte le istanze denominate devono essere ricreate su richiesta.

<xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> è utile negli scenari in cui le opzioni devono essere ricalcolate a ogni richiesta. Per altre informazioni, vedere la sezione [Ricaricare i dati di configurazione con IOptionsSnapshot](#reload-configuration-data-with-ioptionssnapshot).

<xref:Microsoft.Extensions.Options.IOptions%601> può essere usata a supporto delle opzioni. Tuttavia <xref:Microsoft.Extensions.Options.IOptions%601> non supporta gli scenari precedenti di <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>. È possibile continuare a usare <xref:Microsoft.Extensions.Options.IOptions%601> in framework e librerie esistenti che già usano l'interfaccia <xref:Microsoft.Extensions.Options.IOptions%601> e non richiedono gli scenari forniti da <xref:Microsoft.Extensions.Options.IOptionsMonitor%601>.

## <a name="general-options-configuration"></a>Configurazione delle opzioni generali

La configurazione delle opzioni generali è illustrata nell'Esempio 1 nell'app di esempio.

Una classe di opzioni deve essere non astratta con un costruttore pubblico senza parametri. La classe seguente, `MyOptions`, ha due proprietà, `Option1` e `Option2`. Sebbene l'impostazione dei valori predefiniti sia facoltativa, il costruttore della classe nell'esempio seguente imposta il valore predefinito di `Option1`. `Option2` ha un valore impostato tramite l'inizializzazione diretta della proprietà (*Models/MyOptions.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptions.cs?name=snippet1)]

La classe `MyOptions` viene aggiunta al contenitore di servizi con <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> e associata alla configurazione:

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example1)]

Il modello di pagina seguente usa l'[inserimento delle dipendenze del costruttore](xref:mvc/controllers/dependency-injection) con <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> per accedere alle impostazioni (*Pages/Index.cshtml.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example1)]

Il file dell'esempio *appsettings.json* specifica i valori per `option1` e `option2` :

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=2-3)]

Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:

```html
option1 = value1_from_json, option2 = -1
```

> [!NOTE]
> Se si usa un <xref:System.Configuration.ConfigurationBuilder> personalizzato per caricare la configurazione delle opzioni da un file di impostazioni, verificare che il percorso base sia impostato correttamente:
>
> ```csharp
> var configBuilder = new ConfigurationBuilder()
>    .SetBasePath(Directory.GetCurrentDirectory())
>    .AddJsonFile("appsettings.json", optional: true);
> var config = configBuilder.Build();
>
> services.Configure<MyOptions>(config);
> ```
>
> L'impostazione esplicita del percorso base non è necessaria se si carica la configurazione delle opzioni dal file di impostazioni tramite <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.

## <a name="configure-simple-options-with-a-delegate"></a>Configurare opzioni semplici con un delegato

La configurazione di opzioni semplici con un delegato è illustrata nell'Esempio 2 dell'app di esempio.

Usare un delegato per impostare i valori delle opzioni. L'app di esempio usa la classe `MyOptionsWithDelegateConfig` (*Models/MyOptionsWithDelegateConfig.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MyOptionsWithDelegateConfig.cs?name=snippet1)]

Nel codice seguente viene aggiunto un secondo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi. Viene usato un delegato per configurare l'associazione a `MyOptionsWithDelegateConfig`:

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example2)]

*Index.cshtml.cs*:

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3,9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example2)]

È possibile aggiungere più provider di configurazione. I provider di configurazione sono disponibili dai pacchetti NuGet e vengono applicati nell'ordine in cui sono registrati. Per altre informazioni, vedere <xref:fundamentals/configuration/index>.

Ogni chiamata a <xref:Microsoft.Extensions.Options.IConfigureOptions%601.Configure*> aggiunge un servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi. Nell'esempio precedente, i valori di `Option1` e `Option2` sono entrambi specificati in *appsettings.json* , ma i valori di `Option1` e `Option2` vengono sottoposti a override dal delegato configurato.

Quando sono abilitati più servizi di configurazione, l'origine di configurazione più recente specificata *ha priorità* e imposta il valore di configurazione. Quando viene eseguita l'app, il metodo `OnGet` del modello di pagina restituisce una stringa che mostra i valori delle classi delle opzioni:

```html
delegate_option1 = value1_configured_by_delegate, delegate_option2 = 500
```

## <a name="suboptions-configuration"></a>Configurazione delle opzioni secondarie

La configurazione delle opzioni secondarie è illustrata nell'Esempio 3 dell'app di esempio.

Le app devono creare classi di opzioni che riguardano gruppi di scenari (classi) specifici nell'app. Le parti dell'app che richiedono valori di configurazione devono avere accesso solo ai valori di configurazione necessari.

Durante l'associazione delle opzioni alla configurazione, ogni proprietà del tipo di opzioni viene associata a una chiave di configurazione con formato `property[:sub-property:]`. Ad esempio, la `MyOptions.Option1` proprietà è associata alla chiave `Option1` , che viene letta dalla `option1` Proprietà in *appsettings.json* .

Nel codice seguente viene aggiunto un terzo servizio <xref:Microsoft.Extensions.Options.IConfigureOptions%601> al contenitore di servizi. Viene associato `MySubOptions` alla sezione `subsection` del *appsettings.json* file:

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example3)]

Il `GetSection` metodo richiede lo <xref:Microsoft.Extensions.Configuration?displayProperty=fullName> spazio dei nomi.

Il file dell'esempio *appsettings.json* definisce un `subsection` membro con chiavi per `suboption1` e `suboption2` :

[!code-json[](options/samples/2.x/OptionsSample/appsettings.json?highlight=4-7)]

La classe `MySubOptions` definisce le proprietà `SubOption1` e `SubOption2` per contenere i valori delle opzioni (*Models/MySubOptions.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Models/MySubOptions.cs?name=snippet1)]

Il metodo `OnGet` del modello di pagina restituisce una stringa con i valori delle opzioni (*Pages/Index.cshtml.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=4,10)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example3)]

Quando viene eseguita l'app, il metodo `OnGet` restituisce una stringa che mostra i valori delle classi delle opzioni secondarie:

```html
subOption1 = subvalue1_from_json, subOption2 = 200
```

## <a name="options-provided-by-a-view-model-or-with-direct-view-injection"></a>Opzioni fornite da un modello di visualizzazione o con l'inserimento diretto della visualizzazione

Le opzioni fornite da un modello di visualizzazione o con l'inserimento diretto della visualizzazione sono illustrate nell'Esempio 4 dell'app di esempio.

Le opzioni possono essere rese disponibili in un modello di visualizzazione oppure inserendo <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> direttamente in una visualizzazione (*Pages/Index.cshtml.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=9)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=2,8)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example4)]

L'app di esempio mostra come inserire `IOptionsMonitor<MyOptions>` con una direttiva `@inject`:

[!code-cshtml[](options/samples/2.x/OptionsSample/Pages/Index.cshtml?range=1-10&highlight=4)]

Quando l'app viene eseguita, i valori delle opzioni sono visibili nella pagina di cui è stato eseguito il rendering:

![I valori delle opzioni Option1: value1_from_json e Option2: -1 vengono caricati dal modello e tramite inserimento nella visualizzazione.](options/_static/view.png)

## <a name="reload-configuration-data-with-ioptionssnapshot"></a>Ricaricare i dati di configurazione con IOptionsSnapshot

Il ricaricamento dei dati di configurazione con <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> viene illustrato nell'esempio 5 nell'app di esempio.

<xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> supporta il ricaricamento delle opzioni con un overhead di elaborazione minimo.

Le opzioni vengono calcolate una volta per richiesta quando viene eseguito l'accesso e la memorizzazione nella cache per la durata della richiesta.

Nell'esempio seguente viene illustrato come viene creato un nuovo oggetto <xref:Microsoft.Extensions.Options.IOptionsSnapshot%601> dopo *appsettings.json* le modifiche (*pages/index. cshtml. cs*). Più richieste al server restituiscono valori costanti forniti dal *appsettings.json* file fino a quando il file non viene modificato e la configurazione viene ricaricata.

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=12)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=5,11)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example5)]

Nell'immagine seguente vengono illustrati `option1` i `option2` valori iniziali e caricati dal *appsettings.json* file:

```html
snapshot option1 = value1_from_json, snapshot option2 = -1
```

Modificare i valori nel *appsettings.json* file in `value1_from_json UPDATED` e `200` . Salvare il file *appsettings.json*. Aggiornare il browser per visualizzare i valori delle opzioni aggiornati:

```html
snapshot option1 = value1_from_json UPDATED, snapshot option2 = 200
```

## <a name="named-options-support-with-iconfigurenamedoptions"></a>Supporto delle opzioni denominate con IConfigureNamedOptions

Il supporto delle opzioni denominate con <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> è illustrato nell'Esempio 6 nell'app di esempio.

Il supporto delle opzioni denominate consente all'app di distinguere le configurazioni delle opzioni denominate. Nell'app di esempio le opzioni denominate sono dichiarate con [OptionsServiceCollectionExtensions.ConfigUre](xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure*), che chiama [ConfigureNamedOptions \<TOptions> . Configurare](xref:Microsoft.Extensions.Options.ConfigureNamedOptions`1.Configure*) il metodo di estensione. Le opzioni denominate fanno distinzione maiuscole/minuscole

[!code-csharp[](options/samples/2.x/OptionsSample/Startup.cs?name=snippet_Example6)]

L'app di esempio accede alle opzioni denominate con <xref:Microsoft.Extensions.Options.IOptionsSnapshot`1.Get*> (*Pages/Index.cshtml.cs*):

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?range=13-14)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet2&highlight=6,12-13)]

[!code-csharp[](options/samples/2.x/OptionsSample/Pages/Index.cshtml.cs?name=snippet_Example6)]

Eseguendo l'app di esempio, vengono restituite le opzioni denominate:

```html
named_options_1: option1 = value1_from_json, option2 = -1
named_options_2: option1 = named_options_2_value1_from_action, option2 = 5
```

`named_options_1` i valori vengono forniti dalla configurazione, che vengono caricati dal *appsettings.json* file. I valori `named_options_2` sono specificati da:

* Delegato `named_options_2` in `ConfigureServices` per `Option1`.
* Valore predefinito per `Option2` specificato dalla classe `MyOptions`.

## <a name="configure-all-options-with-the-configureall-method"></a>Configurare tutte le opzioni con il metodo ConfigureAll

Configurare tutte le istanze delle opzioni con il metodo <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*>. Il codice seguente configura `Option1` per tutte le istanze di configurazione con un valore comune. Aggiungere manualmente il codice seguente al metodo `Startup.ConfigureServices`:

```csharp
services.ConfigureAll<MyOptions>(myOptions => 
{
    myOptions.Option1 = "ConfigureAll replacement value";
});
```

L'esecuzione dell'app di esempio dopo l'aggiunta del codice produce il risultato seguente:

```html
named_options_1: option1 = ConfigureAll replacement value, option2 = -1
named_options_2: option1 = ConfigureAll replacement value, option2 = 5
```

> [!NOTE]
> Tutte le opzioni sono istanze denominate. Le istanze di <xref:Microsoft.Extensions.Options.IConfigureOptions%601> esistenti sono trattate come se fossero indirizzate all'istanza di `Options.DefaultName`, ovvero `string.Empty`. <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> implementa anche <xref:Microsoft.Extensions.Options.IConfigureOptions%601>. L'implementazione predefinita di <xref:Microsoft.Extensions.Options.IOptionsFactory%601> include la logica per usarle in modo appropriato. L'opzione denominata `null` viene usata per avere come destinazione tutte le istanze denominate anziché un'istanza specifica denominata (questa convenzione è usata da <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.ConfigureAll*> e <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*>).

## <a name="optionsbuilder-api"></a>API OptionsBuilder

<xref:Microsoft.Extensions.Options.OptionsBuilder%601> viene usata per configurare le istanze di `TOptions`. `OptionsBuilder` semplifica la creazione di opzioni denominate perché è costituita da un singolo parametro nella chiamata iniziale ad `AddOptions<TOptions>(string optionsName)` invece di essere visualizzata in tutte le chiamate successive. La convalida delle opzioni e gli overload `ConfigureOptions` che accettano le dipendenze dei servizi sono disponibili solo tramite `OptionsBuilder`.

```csharp
// Options.DefaultName = "" is used.
services.AddOptions<MyOptions>().Configure(o => o.Property = "default");

services.AddOptions<MyOptions>("optionalName")
    .Configure(o => o.Property = "named");
```

## <a name="use-di-services-to-configure-options"></a>Usare i servizi di inserimento delle dipendenze per configurare le opzioni

È possibile accedere ad altri servizi dall'inserimento delle dipendenze durante la configurazione delle opzioni in due modi:

* Passare un delegato di configurazione da [configurare](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) in [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1). [OptionsBuilder \<TOptions> ](xref:Microsoft.Extensions.Options.OptionsBuilder`1) fornisce overload di [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) che consentono di usare fino a cinque servizi per configurare le opzioni:

  ```csharp
  services.AddOptions<MyOptions>("optionalName")
      .Configure<Service1, Service2, Service3, Service4, Service5>(
          (o, s, s2, s3, s4, s5) => 
              o.Property = DoSomethingWith(s, s2, s3, s4, s5));
  ```

* Creare un tipo personalizzato che implementa <xref:Microsoft.Extensions.Options.IConfigureOptions%601> o <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> e registrare il tipo come servizio.

È consigliabile passare un delegato di configurazione a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) perché la creazione di un servizio è più complessa. La creazione di un tipo personalizzato equivale alle operazioni che il framework esegue automaticamente quando si usa [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*). La chiamata a [Configure](xref:Microsoft.Extensions.Options.OptionsBuilder`1.Configure*) registra un'interfaccia <xref:Microsoft.Extensions.Options.IConfigureNamedOptions%601> generica temporanea, che ha un costruttore che accetta i tipi di servizi generici specificati. 

## <a name="options-post-configuration"></a>Post-configurazione delle opzioni

Impostare la post-configurazione con <xref:Microsoft.Extensions.Options.IPostConfigureOptions%601>. La post-configurazione viene eseguita dopo il completamento della configurazione di tutte le <xref:Microsoft.Extensions.Options.IConfigureOptions%601>:

```csharp
services.PostConfigure<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

<xref:Microsoft.Extensions.Options.IPostConfigureOptions`1.PostConfigure*> è disponibile per la post-configurazione delle opzioni denominate:

```csharp
services.PostConfigure<MyOptions>("named_options_1", myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

Usare <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.PostConfigureAll*> per la post-configurazione di tutte le istanze di configurazione:

```csharp
services.PostConfigureAll<MyOptions>(myOptions =>
{
    myOptions.Option1 = "post_configured_option1_value";
});
```

## <a name="accessing-options-during-startup"></a>Accesso alle opzioni durante l'avvio

<xref:Microsoft.Extensions.Options.IOptions%601> e <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> possono essere usate in `Startup.Configure`, perché i servizi vengono compilati prima dell'esecuzione del metodo `Configure`.

```csharp
public void Configure(IApplicationBuilder app, IOptionsMonitor<MyOptions> optionsAccessor)
{
    var option1 = optionsAccessor.CurrentValue.Option1;
}
```

Non usare <xref:Microsoft.Extensions.Options.IOptions%601> oppure <xref:Microsoft.Extensions.Options.IOptionsMonitor%601> in `Startup.ConfigureServices`. Le opzioni potrebbero avere uno stato incoerente a causa dell'ordinamento delle registrazioni dei servizi.

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/configuration/index>
