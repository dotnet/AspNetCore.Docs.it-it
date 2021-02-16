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
<span data-ttu-id="79fe9-101">Il modo migliore per leggere i valori di configurazione correlati consiste nell'usare il [modello di opzioni](xref:fundamentals/configuration/options).</span><span class="sxs-lookup"><span data-stu-id="79fe9-101">The preferred way to read related configuration values is using the [options pattern](xref:fundamentals/configuration/options).</span></span> <span data-ttu-id="79fe9-102">Ad esempio, per leggere i valori di configurazione seguenti:</span><span class="sxs-lookup"><span data-stu-id="79fe9-102">For example, to read the following configuration values:</span></span>

```json
  "Position": {
    "Title": "Editor",
    "Name": "Joe Smith"
  }
```

<span data-ttu-id="79fe9-103">Creare la `PositionOptions` classe seguente:</span><span class="sxs-lookup"><span data-stu-id="79fe9-103">Create the following `PositionOptions` class:</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Options/PositionOptions.cs?name=snippet)]

<span data-ttu-id="79fe9-104">Classe Options:</span><span class="sxs-lookup"><span data-stu-id="79fe9-104">An options class:</span></span>

* <span data-ttu-id="79fe9-105">Deve essere non astratto con un costruttore pubblico senza parametri.</span><span class="sxs-lookup"><span data-stu-id="79fe9-105">Must be non-abstract with a public parameterless constructor.</span></span>
* <span data-ttu-id="79fe9-106">Tutte le proprietà di lettura/scrittura pubbliche del tipo sono associate.</span><span class="sxs-lookup"><span data-stu-id="79fe9-106">All public read-write properties of the type are bound.</span></span>
* <span data-ttu-id="79fe9-107">I campi ***non*** sono associati.</span><span class="sxs-lookup"><span data-stu-id="79fe9-107">Fields are ***not*** bound.</span></span> <span data-ttu-id="79fe9-108">Nel codice precedente `Position` non è associato.</span><span class="sxs-lookup"><span data-stu-id="79fe9-108">In the preceding code, `Position` is not bound.</span></span> <span data-ttu-id="79fe9-109">La `Position` proprietà viene usata in modo che la stringa `"Position"` non debba essere hardcoded nell'app quando si associa la classe a un provider di configurazione.</span><span class="sxs-lookup"><span data-stu-id="79fe9-109">The `Position` property is used so the string `"Position"` doesn't need to be hard coded in the app when binding the class to a configuration provider.</span></span>

<span data-ttu-id="79fe9-110">Il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="79fe9-110">The following code:</span></span>

* <span data-ttu-id="79fe9-111">Chiama [ConfigurationBinder. Bind](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*) per associare la `PositionOptions` classe alla `Position` sezione.</span><span class="sxs-lookup"><span data-stu-id="79fe9-111">Calls [ConfigurationBinder.Bind](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*) to bind the `PositionOptions` class to the `Position` section.</span></span>
* <span data-ttu-id="79fe9-112">Consente di visualizzare i `Position` dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="79fe9-112">Displays the `Position` configuration data.</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test22.cshtml.cs?name=snippet)]

<span data-ttu-id="79fe9-113">Nel codice precedente, per impostazione predefinita, vengono lette le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="79fe9-113">In the preceding code, by default, changes to the JSON configuration file after the app has started are read.</span></span>

<span data-ttu-id="79fe9-114">[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) associa e restituisce il tipo specificato.</span><span class="sxs-lookup"><span data-stu-id="79fe9-114">[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) binds and returns the specified type.</span></span> <span data-ttu-id="79fe9-115">`ConfigurationBinder.Get<T>` può essere più pratico rispetto all'uso di `ConfigurationBinder.Bind` .</span><span class="sxs-lookup"><span data-stu-id="79fe9-115">`ConfigurationBinder.Get<T>` may be more convenient than using `ConfigurationBinder.Bind`.</span></span> <span data-ttu-id="79fe9-116">Il codice seguente illustra come usare `ConfigurationBinder.Get<T>` con la `PositionOptions` classe:</span><span class="sxs-lookup"><span data-stu-id="79fe9-116">The following code shows how to use `ConfigurationBinder.Get<T>` with the `PositionOptions` class:</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test21.cshtml.cs?name=snippet)]

<span data-ttu-id="79fe9-117">Nel codice precedente, per impostazione predefinita, vengono lette le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="79fe9-117">In the preceding code, by default, changes to the JSON configuration file after the app has started are read.</span></span>

<span data-ttu-id="79fe9-118">Un approccio alternativo quando si usa il **modello \* Options** _ consiste nel associare la `Position` sezione e aggiungerla al [contenitore del servizio di inserimento delle dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="79fe9-118">An alternative approach when using the \***options pattern** _ is to bind the `Position` section and add it to the [dependency injection service container](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="79fe9-119">Nel codice seguente, `PositionOptions` viene aggiunto al contenitore del servizio con <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure_> e associato alla configurazione:</span><span class="sxs-lookup"><span data-stu-id="79fe9-119">In the following code, `PositionOptions` is added to the service container with <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure_> and bound to configuration:</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup.cs?name=snippet)]

<span data-ttu-id="79fe9-120">Utilizzando il codice precedente, il codice seguente legge le opzioni di posizione:</span><span class="sxs-lookup"><span data-stu-id="79fe9-120">Using the preceding code, the following code reads the position options:</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Pages/Test2.cshtml.cs?name=snippet)]

<span data-ttu-id="79fe9-121">Nel codice precedente, le modifiche apportate al file di configurazione JSON dopo l'avvio dell'app ***non*** vengono lette.</span><span class="sxs-lookup"><span data-stu-id="79fe9-121">In the preceding code, changes to the JSON configuration file after the app has started are ***not*** read.</span></span> <span data-ttu-id="79fe9-122">Per leggere le modifiche dopo l'avvio dell'app, usare [IOptionsSnapshot](xref:fundamentals/configuration/options#ios).</span><span class="sxs-lookup"><span data-stu-id="79fe9-122">To read changes after the app has started, use [IOptionsSnapshot](xref:fundamentals/configuration/options#ios).</span></span>
