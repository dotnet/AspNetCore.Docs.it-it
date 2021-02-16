---
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
ms.openlocfilehash: 30baab0649268f4abf0dbd6c99dfeef3f43d0054
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536298"
---
Il modo migliore per leggere i valori di configurazione correlati consiste nell'usare il [modello di opzioni](xref:fundamentals/configuration/options). Ad esempio, per leggere i valori di configurazione seguenti:

```json
  "Position": {
    "Title": "Editor",
    "Name": "Joe Smith"
  }
```

Creare la `PositionOptions` classe seguente:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Options/PositionOptions.cs?name=snippet)]

Classe Options:

* Deve essere non astratto con un costruttore pubblico senza parametri.
* Tutte le proprietà di lettura/scrittura pubbliche del tipo sono associate.
* I campi ***non*** sono associati. Nel codice precedente `Position` non è associato. La `Position` proprietà viene usata in modo che la stringa `"Position"` non debba essere hardcoded nell'app quando si associa la classe a un provider di configurazione.

Il codice seguente:

* Chiama [ConfigurationBinder. Bind](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*) per associare la `PositionOptions` classe alla `Position` sezione.
* Consente di visualizzare i `Position` dati di configurazione.

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test22.cshtml.cs?name=snippet)]

Nel codice precedente, per impostazione predefinita, vengono lette le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app.

[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) associa e restituisce il tipo specificato. `ConfigurationBinder.Get<T>` può essere più pratico rispetto all'uso di `ConfigurationBinder.Bind` . Il codice seguente illustra come usare `ConfigurationBinder.Get<T>` con la `PositionOptions` classe:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test21.cshtml.cs?name=snippet)]

Nel codice precedente, per impostazione predefinita, vengono lette le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app.

Un approccio alternativo quando si usa il **modello * Options** _ consiste nel associare la `Position` sezione e aggiungerla al [contenitore del servizio di inserimento delle dipendenze](xref:fundamentals/dependency-injection). Nel codice seguente, `PositionOptions` viene aggiunto al contenitore del servizio con <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure_> e associato alla configurazione:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup.cs?name=snippet)]

Utilizzando il codice precedente, il codice seguente legge le opzioni di posizione:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test2.cshtml.cs?name=snippet)]

Nel codice precedente, le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app ***non*** vengono lette. Per leggere le modifiche dopo l'avvio dell'app, usare [IOptionsSnapshot](xref:fundamentals/configuration/options#ios).
