---
title: Associazione di modelli personalizzata in ASP.NET Core
author: ardalis
description: Informazioni su come l'associazione di modelli consente alle azioni del controller di funzionare direttamente con i tipi di modello in ASP.NET Core.
ms.author: riande
ms.date: 01/06/2020
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
uid: mvc/advanced/custom-model-binding
ms.openlocfilehash: b674d3a034124a26e2e76f2a7f0220e3f0ecea56
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588770"
---
# <a name="custom-model-binding-in-aspnet-core"></a>Associazione di modelli personalizzata in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Steve Smith](https://ardalis.com/) e [Kirk Larkin](https://twitter.com/serpent5)

Mediante l'associazione di modelli le azioni dei controller possono operare direttamente con i tipi di modello (passati come argomenti dei metodi), anziché con le richieste HTTP. Il mapping tra i dati delle richieste in ingresso e i modelli applicativi è gestito dagli strumenti di associazione di modelli. Gli sviluppatori possono estendere la funzionalità di associazione di modelli predefinita implementando strumenti di associazione di modelli personalizzati (anche se in genere non è necessario creare un provider personalizzato).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="default-model-binder-limitations"></a>Limiti degli strumenti di associazione di modelli predefiniti

Gli strumenti di associazione di modelli predefiniti supportano la maggior parte dei tipi di dati .NET Core comuni e soddisfano le esigenze della maggior parte degli sviluppatori. Tali strumenti associano direttamente l'input di testo della richiesta ai tipi di modello. Può essere necessario trasformare l'input prima dell'associazione. Un esempio è il caso in cui è presente una chiave che può essere usata per cercare i dati del modello. È possibile usare uno strumento di associazione di modelli personalizzato per il recupero di dati in base alla chiave.

## <a name="model-binding-review"></a>Analisi dell'associazione di modelli

L'associazione di modelli usa definizioni specifiche per i tipi sui quali opera. Un *tipo semplice* viene convertito da una stringa singola dell'input. Un *tipo complesso* viene convertito da più valori di input. Il framework determina la differenza in base all'esistenza di un elemento `TypeConverter`. È consigliabile creare un convertitore di tipi se è presente un mapping `string` -> `SomeType` semplice che non richiede risorse esterne.

Prima di creare uno strumento di associazione di modelli personalizzato, è utile esaminare le modalità di implementazione degli strumenti di associazione di modelli esistenti. Si consideri l'oggetto <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> che può essere usato per convertire le stringhe con codifica Base64 in matrici di byte. Le matrici di byte vengono spesso archiviate come file o campi BLOB del database.

### <a name="working-with-the-bytearraymodelbinder"></a>Uso di ByteArrayModelBinder

Le stringhe con codifica Base64 possono essere usate per rappresentare i dati binari. Un'immagine, ad esempio, può essere codificata come stringa. Nell'esempio è inclusa un'immagine come stringa con codifica Base64 in [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/3.x/CustomModelBindingSample/Base64String.txt).

ASP.NET Core MVC può accettare una stringa con codifica base64 e usare un oggetto `ByteArrayModelBinder` per convertirla in una matrice di byte. <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider>Esegue il mapping degli `byte[]` argomenti a `ByteArrayModelBinder` :

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        var loggerFactory = context.Services.GetRequiredService<ILoggerFactory>();
        return new ByteArrayModelBinder(loggerFactory);
    }

    return null;
}
```

Quando si crea un gestore di associazione di modelli personalizzato, è possibile implementare un proprio `IModelBinderProvider` tipo o usare <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute> .

L'esempio indica come usare `ByteArrayModelBinder` per convertire una stringa con codifica base64 in un `byte[]` e salvare il risultato in un file:

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_Post)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

È possibile pubblicare (POST) una stringa con codifica base64 in questo metodo API usando uno strumento come [Postman](https://www.getpostman.com/):

![Postman](custom-model-binding/images/postman.png "Postman")

Se lo strumento di associazione riesce ad associare i dati della richiesta a proprietà o argomenti denominati in modo appropriato, l'associazione di modelli ha esito positivo. L'esempio seguente indica come usare `ByteArrayModelBinder` con un modello di visualizzazione:

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_SaveProfile&highlight=2)]

## <a name="custom-model-binder-sample"></a>Esempio di strumento di associazione di modelli personalizzato

In questa sezione viene implementato uno strumento di associazione di modelli personalizzato che:

- Converte i dati della richiesta in ingresso in argomenti chiave fortemente tipizzati.
- Usa Entity Framework Core per recuperare l'entità associata.
- Passa l'entità associata come argomento al metodo di azione.

L'esempio seguente usa l'attributo `ModelBinder` nel modello `Author`:

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

Nel codice precedente l'attributo `ModelBinder` specifica il tipo di `IModelBinder` da usare per associare i parametri di azione `Author`.

La classe `AuthorEntityBinder` seguente associa un parametro `Author` recuperando l'entità da un'origine dati tramite Entity Framework Core e un `authorId`:

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=snippet_Class)]

> [!NOTE]
> La classe `AuthorEntityBinder` precedente è destinata a illustrare uno strumento di associazione di modelli personalizzato. La classe non ha lo scopo di illustrare procedure consigliate per uno scenario di ricerca. Per la ricerca, associare l'elemento `authorId` ed eseguire query nel database in un metodo di azione. Questo approccio separa gli errori di associazione di modelli dai casi `NotFound`.

L'esempio di codice seguente indica come usare `AuthorEntityBinder` in un metodo di azione:

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_Get&highlight=2)]

L'attributo `ModelBinder` può essere usato per applicare gli elementi `AuthorEntityBinder` ai parametri che non usano convenzioni predefinite:

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_GetById&highlight=2)]

In questo esempio, dato che il nome dell'argomento non è il valore `authorId` predefinito, viene specificato nel parametro usando l'attributo `ModelBinder`. Il controller e il metodo di azione sono semplificati rispetto alla ricerca dell'entità nel metodo di azione. La logica per recuperare l'autore usando Entity Framework Core viene spostata nello strumento di associazione di modelli. Ciò può rappresentare una semplificazione notevole in presenza di vari metodi associati al modello `Author`.

È possibile applicare l'attributo `ModelBinder` a singole proprietà del modello (ad esempio a ViewModel) o a parametri del metodo di azione per specificare un determinato strumento di associazione di modelli o nome di modello solo per quel determinato tipo o azione.

### <a name="implementing-a-modelbinderprovider"></a>Implementazione di un elemento ModelBinderProvider

Anziché applicare un attributo, è possibile implementare `IModelBinderProvider`. Gli strumenti di associazione del framework incorporati vengono implementati in questo modo. Quando si specifica il tipo sul quale opera lo strumento di associazione, si indica il tipo di argomento prodotto dallo strumento, **non** l'input accettato dallo strumento. Il provider strumento di associazione seguente funziona con l'elemento `AuthorEntityBinder`. Quando viene aggiunto alla raccolta di provider di MVC, non è necessario usare l'attributo `ModelBinder` sui parametri tipizzati `Author` o `Author`.

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> Nota: il codice precedente restituisce un elemento `BinderTypeModelBinder`. `BinderTypeModelBinder` funziona come factory per gli strumenti di associazione di modelli e implementa la funzionalità DI (Dependency Injection, Inserimento di dipendenze). `AuthorEntityBinder` richiede DI (Dependency Injection, Inserimento di dipendenze) per l'accesso a EF Core. Usare `BinderTypeModelBinder` se lo strumento di associazione di modelli richiede servizi da DI (Dependency Injection, Inserimento di dipendenze).

Per usare un provider dello strumento di associazione di modelli personalizzato, aggiungerlo in `ConfigureServices`:

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-8)]

Durante la valutazione degli strumenti di associazione di modelli, la raccolta di provider viene esaminata in ordine. Viene utilizzato il primo provider che restituisce uno strumento di associazione che corrisponde al modello di input. L'aggiunta del provider alla fine della raccolta può pertanto comportare la chiamata di uno strumento di associazione di modelli predefinito prima che il Binder personalizzato abbia la possibilità. In questo esempio, il provider personalizzato viene aggiunto all'inizio della raccolta per assicurarsi che venga sempre usato per gli `Author` argomenti dell'azione.

### <a name="polymorphic-model-binding"></a>Associazione di modelli polimorfici

L'associazione a modelli diversi di tipi derivati è nota come associazione di modelli polimorfici. L'associazione di modelli personalizzati polimorfici è obbligatoria quando il valore della richiesta deve essere associato al tipo di modello derivato specifico. Associazione di modelli polimorfici:

* Non è tipico per un'API REST progettata per interagire con tutti i linguaggi.
* Rende difficile la motivazione dei modelli associati.

Tuttavia, se un'app richiede l'associazione di modelli polimorfici, un'implementazione potrebbe avere un aspetto simile al codice seguente:

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a>Suggerimenti e procedure ottimali

Gli strumenti di associazione di modelli personalizzati:

- Non devono provare a impostare codici di stato o restituire risultati (ad esempio 404 Non trovato). Se si verifica un errore nell'associazione di modelli, l'errore deve essere gestito da un [filtro azioni](xref:mvc/controllers/filters) o da logica inclusa nel metodo di azione.
- Sono particolarmente utili per eliminare codice ripetitivo e problemi di montaggio incrociato dai metodi di azione.
- In genere non devono essere usati per convertire una stringa in un tipo personalizzato. Un elemento <xref:System.ComponentModel.TypeConverter> rappresenta solitamente una scelta migliore.

::: moniker-end
::: moniker range="< aspnetcore-3.0"

Di [Steve Smith](https://ardalis.com/)

Mediante l'associazione di modelli le azioni dei controller possono operare direttamente con i tipi di modello (passati come argomenti dei metodi), anziché con le richieste HTTP. Il mapping tra i dati delle richieste in ingresso e i modelli applicativi è gestito dagli strumenti di associazione di modelli. Gli sviluppatori possono estendere la funzionalità di associazione di modelli predefinita implementando strumenti di associazione di modelli personalizzati (anche se in genere non è necessario creare un provider personalizzato).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="default-model-binder-limitations"></a>Limiti degli strumenti di associazione di modelli predefiniti

Gli strumenti di associazione di modelli predefiniti supportano la maggior parte dei tipi di dati .NET Core comuni e soddisfano le esigenze della maggior parte degli sviluppatori. Tali strumenti associano direttamente l'input di testo della richiesta ai tipi di modello. Può essere necessario trasformare l'input prima dell'associazione. Un esempio è il caso in cui è presente una chiave che può essere usata per cercare i dati del modello. È possibile usare uno strumento di associazione di modelli personalizzato per il recupero di dati in base alla chiave.

## <a name="model-binding-review"></a>Analisi dell'associazione di modelli

L'associazione di modelli usa definizioni specifiche per i tipi sui quali opera. Un *tipo semplice* viene convertito da una stringa singola dell'input. Un *tipo complesso* viene convertito da più valori di input. Il framework determina la differenza in base all'esistenza di un elemento `TypeConverter`. È consigliabile creare un convertitore di tipi se è presente un mapping `string` -> `SomeType` semplice che non richiede risorse esterne.

Prima di creare uno strumento di associazione di modelli personalizzato, è utile esaminare le modalità di implementazione degli strumenti di associazione di modelli esistenti. Si consideri l'oggetto <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> che può essere usato per convertire le stringhe con codifica Base64 in matrici di byte. Le matrici di byte vengono spesso archiviate come file o campi BLOB del database.

### <a name="working-with-the-bytearraymodelbinder"></a>Uso di ByteArrayModelBinder

Le stringhe con codifica Base64 possono essere usate per rappresentare i dati binari. Un'immagine, ad esempio, può essere codificata come stringa. Nell'esempio è inclusa un'immagine come stringa con codifica Base64 in [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/2.x/CustomModelBindingSample/Base64String.txt).

ASP.NET Core MVC può accettare una stringa con codifica base64 e usare un oggetto `ByteArrayModelBinder` per convertirla in una matrice di byte. <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider>Esegue il mapping degli `byte[]` argomenti a `ByteArrayModelBinder` :

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        return new ByteArrayModelBinder();
    }

    return null;
}
```

Quando si crea un gestore di associazione di modelli personalizzato, è possibile implementare un proprio `IModelBinderProvider` tipo o usare <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute> .

L'esempio indica come usare `ByteArrayModelBinder` per convertire una stringa con codifica base64 in un `byte[]` e salvare il risultato in un file:

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post1)]

È possibile pubblicare (POST) una stringa con codifica base64 in questo metodo API usando uno strumento come [Postman](https://www.getpostman.com/):

![Postman](custom-model-binding/images/postman.png "Postman")

Se lo strumento di associazione riesce ad associare i dati della richiesta a proprietà o argomenti denominati in modo appropriato, l'associazione di modelli ha esito positivo. L'esempio seguente indica come usare `ByteArrayModelBinder` con un modello di visualizzazione:

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post2&highlight=2)]

## <a name="custom-model-binder-sample"></a>Esempio di strumento di associazione di modelli personalizzato

In questa sezione viene implementato uno strumento di associazione di modelli personalizzato che:

- Converte i dati della richiesta in ingresso in argomenti chiave fortemente tipizzati.
- Usa Entity Framework Core per recuperare l'entità associata.
- Passa l'entità associata come argomento al metodo di azione.

L'esempio seguente usa l'attributo `ModelBinder` nel modello `Author`:

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

Nel codice precedente l'attributo `ModelBinder` specifica il tipo di `IModelBinder` da usare per associare i parametri di azione `Author`.

La classe `AuthorEntityBinder` seguente associa un parametro `Author` recuperando l'entità da un'origine dati tramite Entity Framework Core e un `authorId`:

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=demo)]

> [!NOTE]
> La classe `AuthorEntityBinder` precedente è destinata a illustrare uno strumento di associazione di modelli personalizzato. La classe non ha lo scopo di illustrare procedure consigliate per uno scenario di ricerca. Per la ricerca, associare l'elemento `authorId` ed eseguire query nel database in un metodo di azione. Questo approccio separa gli errori di associazione di modelli dai casi `NotFound`.

L'esempio di codice seguente indica come usare `AuthorEntityBinder` in un metodo di azione:

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo2&highlight=2)]

L'attributo `ModelBinder` può essere usato per applicare gli elementi `AuthorEntityBinder` ai parametri che non usano convenzioni predefinite:

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo1&highlight=2)]

In questo esempio, dato che il nome dell'argomento non è il valore `authorId` predefinito, viene specificato nel parametro usando l'attributo `ModelBinder`. Il controller e il metodo di azione sono semplificati rispetto alla ricerca dell'entità nel metodo di azione. La logica per recuperare l'autore usando Entity Framework Core viene spostata nello strumento di associazione di modelli. Ciò può rappresentare una semplificazione notevole in presenza di vari metodi associati al modello `Author`.

È possibile applicare l'attributo `ModelBinder` a singole proprietà del modello (ad esempio a ViewModel) o a parametri del metodo di azione per specificare un determinato strumento di associazione di modelli o nome di modello solo per quel determinato tipo o azione.

### <a name="implementing-a-modelbinderprovider"></a>Implementazione di un elemento ModelBinderProvider

Anziché applicare un attributo, è possibile implementare `IModelBinderProvider`. Gli strumenti di associazione del framework incorporati vengono implementati in questo modo. Quando si specifica il tipo sul quale opera lo strumento di associazione, si indica il tipo di argomento prodotto dallo strumento, **non** l'input accettato dallo strumento. Il provider strumento di associazione seguente funziona con l'elemento `AuthorEntityBinder`. Quando viene aggiunto alla raccolta di provider di MVC, non è necessario usare l'attributo `ModelBinder` sui parametri tipizzati `Author` o `Author`.

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> Nota: il codice precedente restituisce un elemento `BinderTypeModelBinder`. `BinderTypeModelBinder` funziona come factory per gli strumenti di associazione di modelli e implementa la funzionalità DI (Dependency Injection, Inserimento di dipendenze). `AuthorEntityBinder` richiede DI (Dependency Injection, Inserimento di dipendenze) per l'accesso a EF Core. Usare `BinderTypeModelBinder` se lo strumento di associazione di modelli richiede servizi da DI (Dependency Injection, Inserimento di dipendenze).

Per usare un provider dello strumento di associazione di modelli personalizzato, aggiungerlo in `ConfigureServices`:

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-10)]

Durante la valutazione degli strumenti di associazione di modelli, la raccolta di provider viene esaminata in ordine. Viene usato il primo provider che restituisce uno strumento di associazione. Se il provider personalizzato viene aggiunto alla fine della raccolta, è possibile che uno strumento di associazione di modelli incorporato venga chiamato prima dello strumento di associazione di modelli personalizzato. In questo esempio il provider personalizzato viene aggiunto all'inizio della raccolta, per garantire che venga usato per gli argomenti dell'azione `Author`.

### <a name="polymorphic-model-binding"></a>Associazione di modelli polimorfici

L'associazione a modelli diversi di tipi derivati è nota come associazione di modelli polimorfici. L'associazione di modelli personalizzati polimorfici è obbligatoria quando il valore della richiesta deve essere associato al tipo di modello derivato specifico. Associazione di modelli polimorfici:

* Non è tipico per un'API REST progettata per interagire con tutti i linguaggi.
* Rende difficile la motivazione dei modelli associati.

Tuttavia, se un'app richiede l'associazione di modelli polimorfici, un'implementazione potrebbe avere un aspetto simile al codice seguente:

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a>Suggerimenti e procedure ottimali

Gli strumenti di associazione di modelli personalizzati:

- Non devono provare a impostare codici di stato o restituire risultati (ad esempio 404 Non trovato). Se si verifica un errore nell'associazione di modelli, l'errore deve essere gestito da un [filtro azioni](xref:mvc/controllers/filters) o da logica inclusa nel metodo di azione.
- Sono particolarmente utili per eliminare codice ripetitivo e problemi di montaggio incrociato dai metodi di azione.
- In genere non devono essere usati per convertire una stringa in un tipo personalizzato. Un elemento <xref:System.ComponentModel.TypeConverter> rappresenta solitamente una scelta migliore.

::: moniker-end
