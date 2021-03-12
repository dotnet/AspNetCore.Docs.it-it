---
title: Formattatori personalizzati nell'API Web ASP.NET Core
author: rick-anderson
description: Informazioni su come creare e usare i formattatori personalizzati nelle API Web ASP.NET Core.
ms.author: riande
ms.date: 06/25/2020
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
uid: web-api/advanced/custom-formatters
ms.openlocfilehash: 91c9c6513d7c8df671e283508ecc276768d79539
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587827"
---
# <a name="custom-formatters-in-aspnet-core-web-api"></a>Formattatori personalizzati nell'API Web ASP.NET Core

Di [Kirk Larkin](https://twitter.com/serpent5) e [Tom Dykstra](https://github.com/tdykstra).

ASP.NET Core MVC supporta lo scambio di dati in API Web mediante formattatori di input e di output. I formattatori di input vengono utilizzati dall' [associazione di modelli](xref:mvc/models/model-binding). I formattatori di output vengono utilizzati per [formattare le risposte](xref:web-api/advanced/formatting).

Il Framework fornisce formattatori di input e output predefiniti per JSON e XML. Fornisce un formattatore di output incorporato per il testo normale, ma non fornisce un formattatore di input per testo normale.

In questo articolo viene illustrato come aggiungere supporto per altri formati creando formattatori personalizzati. Per un esempio di formattatore di input di testo normale personalizzato, vedere [TextPlainInputFormatter](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.Formatters/TextPlainInputFormatter.cs) su GitHub.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="when-to-use-custom-formatters"></a>Quando usare i formattatori personalizzati

Usare un formattatore personalizzato per aggiungere il supporto per un tipo di contenuto non gestito dai formattatori predefiniti.

## <a name="overview-of-how-to-use-a-custom-formatter"></a>Panoramica sull'uso di un formattatore personalizzato

Per creare un formattatore personalizzato:

* Per la serializzazione dei dati inviati al client, creare una classe formattatore di output.
* Per la deserializzazione dei dati ricevuti dal client, creare una classe formatter di input.
* Aggiungere istanze delle classi formatter alle `InputFormatters` raccolte e `OutputFormatters` in <xref:Microsoft.AspNetCore.Mvc.MvcOptions> .

## <a name="how-to-create-a-custom-formatter-class"></a>Come creare una classe formattatore personalizzato

Per creare un formattatore:

* Derivare la classe dalla classe di base appropriata. L'app di esempio deriva da <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> e <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> .
* Specificare i formati di testo multimediali validi e le codifiche nel costruttore.
* Eseguire l'override dei metodi <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.CanReadType%2A> e <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter.CanWriteType%2A>.
* Eseguire l'override dei metodi <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.ReadRequestBodyAsync%2A> e `WriteResponseBodyAsync`.

Nel codice seguente viene illustrata la `VcardOutputFormatter` classe dell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples):

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_Class)]
  
### <a name="derive-from-the-appropriate-base-class"></a>Derivare dalla classe di base appropriata

Per i tipi di supporti di testo (ad esempio, vCard), derivare dalla <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> classe di base o.

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ClassDeclaration)]

Per i tipi binari, derivare <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter> dalla <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter> classe di base o.

### <a name="specify-valid-media-types-and-encodings"></a>Specificare i formati multimediali validi e le codifiche

Nel costruttore specificare i formati multimediali validi e le codifiche aggiungendo alle raccolte `SupportedMediaTypes` e `SupportedEncodings`.

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ctor)]

Una classe formatter **non** può utilizzare l'inserimento del costruttore per le relative dipendenze. Ad esempio, `ILogger<VcardOutputFormatter>` non può essere aggiunto come parametro al costruttore. Per accedere ai servizi, usare l'oggetto di contesto che viene passato ai metodi. Un esempio di codice in questo articolo e nell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples) illustra come eseguire questa operazione.

### <a name="override-canreadtype-and-canwritetype"></a>Eseguire l'override di CanReadType e CanWriteType

Specificare il tipo in cui eseguire la deserializzazione o serializzare eseguendo l'override dei `CanReadType` `CanWriteType` metodi o. Ad esempio, la creazione di testo vCard da un `Contact` tipo e viceversa.

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_CanWriteType)]

#### <a name="the-canwriteresult-method"></a>Metodo CanWriteResult

In alcuni scenari `CanWriteResult` è necessario eseguire l'override di anziché `CanWriteType` . Usare `CanWriteResult` se vengono soddisfatte le condizioni seguenti:

* Il metodo di azione restituisce una classe di modello.
* Sono disponibili classi derivate che possono essere restituite in fase di esecuzione.
* La classe derivata restituita dall'azione deve essere nota in fase di esecuzione.

Si supponga, ad esempio, che il metodo di azione:

* La firma restituisce un `Person` tipo.
* Può restituire un `Student` `Instructor` tipo o che deriva da `Person` . 

Affinché il formattatore gestisca solo `Student` oggetti, controllare il tipo di <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatterCanWriteContext.Object> nell'oggetto di contesto fornito al `CanWriteResult` metodo. Quando il metodo di azione restituisce `IActionResult` :

* Non è necessario usare `CanWriteResult` .
* Il `CanWriteType` metodo riceve il tipo di Runtime.

<a id="read-write"></a>

### <a name="override-readrequestbodyasync-and-writeresponsebodyasync"></a>Eseguire l'override di ReadRequestBodyAsync e WriteResponseBodyAsync

La deserializzazione o la serializzazione viene eseguita in `ReadRequestBodyAsync` o `WriteResponseBodyAsync` . Nell'esempio seguente viene illustrato come ottenere servizi dal contenitore di inserimento delle dipendenze. Non è possibile ottenere i servizi dai parametri del costruttore.

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_WriteResponseBodyAsync)]

## <a name="how-to-configure-mvc-to-use-a-custom-formatter"></a>Come configurare MVC per usare un formattatore personalizzato

Per usare un formattatore personalizzato, aggiungere un'istanza della classe formattatore alla raccolta `InputFormatters` o `OutputFormatters`.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/2.x/CustomFormattersSample/Startup.cs?name=mvcoptions&highlight=3-4)]

::: moniker-end

I formattatori vengono valutati nell'ordine d'inserimento. Il primo ha la precedenza.

## <a name="the-complete-vcardinputformatter-class"></a>Classe completa `VcardInputFormatter`

Nel codice seguente viene illustrata la `VcardInputFormatter` classe dell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples):

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardInputFormatter.cs?name=snippet_Class)]

## <a name="test-the-app"></a>Testare l'app

[Eseguire l'app di esempio per questo articolo](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples), che implementa i formattatori di input e output vCard di base. L'app legge e scrive le vCard simili alle seguenti:

```
BEGIN:VCARD
VERSION:2.1
N:Davolio;Nancy
FN:Nancy Davolio
END:VCARD
```

Per visualizzare l'output della vCard, eseguire l'app e inviare una richiesta GET con l'intestazione Accept `text/vcard` a `https://localhost:5001/api/contacts` .

Per aggiungere un vCard alla raccolta di contatti in memoria:

* Inviare una `Post` richiesta a `/api/contacts` con uno strumento come il post.
* Impostare l'intestazione `Content-Type` su `text/vcard`.
* Imposta il `vCard` testo nel corpo, formattato come l'esempio precedente.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:web-api/advanced/formatting>
* <xref:grpc/dotnet-grpc>
