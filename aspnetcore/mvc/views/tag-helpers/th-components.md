---
title: Componenti helper tag in ASP.NET Core
author: scottaddie
description: Informazioni sui componenti helper tag e su come usarli in ASP.NET Core.
monikerRange: '>= aspnetcore-2.0'
ms.author: scaddie
ms.date: 06/12/2019
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
uid: mvc/views/tag-helpers/th-components
ms.openlocfilehash: fb0bda0cf8d225df4c58ae43f81ed0dce10c1adc
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587151"
---
# <a name="tag-helper-components-in-aspnet-core"></a>Componenti helper tag in ASP.NET Core

Di [Scott Addie](https://twitter.com/Scott_Addie) e [Fiyaz Bin Hasan](https://github.com/fiyazbinhasan)

Un componente helper tag è un helper tag che consente di modificare o aggiungere elementi HTML da codice lato server in modo condizionale. Questa funzionalità è disponibile in ASP.NET Core 2.0 o versioni successive.

ASP.NET Core include due componenti helper tag predefiniti: `head` e `body`. Si trovano nello <xref:Microsoft.AspNetCore.Mvc.Razor.TagHelpers> spazio dei nomi e possono essere usati sia in MVC che in Razor pagine. I componenti helper tag non richiedono la registrazione con l'app in *_ViewImports.cshtml*.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/th-components/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="use-cases"></a>Casi d'uso

Due casi d'uso comuni dei componenti helper tag sono:

1. [Inserimento di un `<link>` nell'elemento `<head>`.](#inject-into-html-head-element)
1. [Inserimento di un `<script>` nell'elemento `<body>`.](#inject-into-html-body-element)

Le sezioni seguenti descrivono questi casi d'uso.

### <a name="inject-into-html-head-element"></a>Inserire in un elemento HTML head

All'interno dell'elemento HTML `<head>` i file CSS vengono generalmente importati con l'elemento HTML `<link>`. Il codice seguente inserisce un elemento `<link>` nell'elemento `<head>` usando il componente helper tag `head`:

[!code-csharp[](th-components/samples/RazorPagesSample/TagHelpers/AddressStyleTagHelperComponent.cs)]

Nel codice precedente:

* `AddressStyleTagHelperComponent` implementa <xref:Microsoft.AspNetCore.Razor.TagHelpers.TagHelperComponent>. L'astrazione:
  * Consente l'inizializzazione della classe con un <xref:Microsoft.AspNetCore.Razor.TagHelpers.TagHelperContext>.
  * Abilita l'uso dei componenti helper tag per l'aggiunta o la modifica di elementi HTML.
* La proprietà <xref:Microsoft.AspNetCore.Razor.TagHelpers.TagHelperComponent.Order*> definisce l'ordine in cui viene eseguito il rendering dei componenti. `Order` è necessaria in presenza di più utilizzi dei componenti helper tag in un'app.
* <xref:Microsoft.AspNetCore.Razor.TagHelpers.TagHelperComponent.ProcessAsync*> confronta il valore della proprietà <xref:Microsoft.AspNetCore.Razor.TagHelpers.TagHelperContext.TagName*> del contesto di esecuzione con `head`. Se il confronto restituisce true, il contenuto del campo `_style` viene inserito nell'elemento HTML `<head>`.

### <a name="inject-into-html-body-element"></a>Inserire in un elemento HTML body

Il componente helper tag `body` può inserire un elemento `<script>` nell'elemento `<body>`. Questa tecnica è illustrata nel codice riportato di seguito:

[!code-csharp[](th-components/samples/RazorPagesSample/TagHelpers/AddressScriptTagHelperComponent.cs)]

Per archiviare l'elemento `<script>` viene usato un file HTML separato, che rende il codice più lineare e gestibile. Il codice riportato sopra legge il contenuto di *TagHelpers/Templates/AddressToolTipScript.html* e vi accoda l'output dell'helper tag. Il file *AddressToolTipScript.html* include il markup seguente:

[!code-html[](th-components/samples/RazorPagesSample/TagHelpers/Templates/AddressToolTipScript.html)]

Il codice precedente associa un [widget tooltip Boostrap](https://getbootstrap.com/docs/3.3/javascript/#tooltips) a qualsiasi elemento `<address>` che include un attributo `printable`. L'effetto è visibile quando si posiziona il puntatore del mouse sull'elemento.

## <a name="register-a-component"></a>Registrare un componente

Un componente helper tag deve essere aggiunto alla raccolta di componenti helper tag. È possibile farlo in tre modi:

* [Registrazione tramite un contenitore di servizi](#registration-via-services-container)
* [Registrazione tramite Razor file](#registration-via-razor-file)
* [Registrazione tramite un modello di pagina o un controller](#registration-via-page-model-or-controller)

### <a name="registration-via-services-container"></a>Registrazione tramite un contenitore di servizi

Se la classe di componenti helper tag non è gestita con <xref:Microsoft.AspNetCore.Mvc.Razor.TagHelpers.ITagHelperComponentManager>, deve essere registrata con il sistema di [inserimento delle dipendenze](xref:fundamentals/dependency-injection). Il codice `Startup.ConfigureServices` seguente registra le classi `AddressStyleTagHelperComponent` e `AddressScriptTagHelperComponent` con una [durata temporanea](xref:fundamentals/dependency-injection#lifetime-and-registration-options):

[!code-csharp[](th-components/samples/RazorPagesSample/Startup.cs?name=snippet_ConfigureServices&highlight=12-15)]

### <a name="registration-via-razor-file"></a>Registrazione tramite Razor file

Se il componente helper tag non è registrato con DI, può essere registrato da una Razor pagina di pagine o da una visualizzazione MVC. Questa tecnica viene usata per controllare il markup inserito e l'ordine di esecuzione dei componenti da un Razor file.

`ITagHelperComponentManager` viene usato per aggiungere componenti helper tag o per rimuoverli dall'app. Il codice seguente illustra questa tecnica con `AddressTagHelperComponent`:

[!code-cshtml[](th-components/samples/RazorPagesSample/Pages/Contact.cshtml?name=snippet_ITagHelperComponentManager)]

Nel codice precedente:

* La direttiva `@inject` fornisce un'istanza di `ITagHelperComponentManager`. L'istanza viene assegnata a una variabile denominata `manager` per accedere a downstream nel Razor file.
* Un'istanza di `AddressTagHelperComponent` viene aggiunta alla raccolta di componenti helper tag dell'app.

`AddressTagHelperComponent` viene modificato per contenere un costruttore che accetta i parametri `markup` e `order`:

[!code-csharp[](th-components/samples/RazorPagesSample/TagHelpers/AddressTagHelperComponent.cs?name=snippet_Constructor)]

Il parametro `markup` specificato viene usato in `ProcessAsync` nel modo seguente:

[!code-csharp[](th-components/samples/RazorPagesSample/TagHelpers/AddressTagHelperComponent.cs?name=snippet_ProcessAsync&highlight=10-11)]

### <a name="registration-via-page-model-or-controller"></a>Registrazione tramite un modello di pagina o un controller

Se il componente helper tag non è registrato con DI, può essere registrato da un Razor modello di pagina delle pagine o da un controller MVC. Questa tecnica è utile per separare la logica C# dai Razor file.

Per accedere a un'istanza di `ITagHelperComponentManager` viene usato l'inserimento del costruttore. Il componente helper tag deve viene aggiunto alla raccolta di componenti helper tag dell'istanza. Il Razor modello di pagina delle pagine seguente illustra questa tecnica con `AddressTagHelperComponent` :

[!code-csharp[](th-components/samples/RazorPagesSample/Pages/Index.cshtml.cs?name=snippet_IndexModelClass)]

Nel codice precedente:

* Per accedere a un'istanza di `ITagHelperComponentManager` viene usato l'inserimento del costruttore.
* Un'istanza di `AddressTagHelperComponent` viene aggiunta alla raccolta di componenti helper tag dell'app.

## <a name="create-a-component"></a>Creare un componente

Per creare un componente helper tag personalizzato:

* Creare una classe pubblica che deriva da <xref:Microsoft.AspNetCore.Mvc.Razor.TagHelpers.TagHelperComponentTagHelper>.
* Applicare un [`[HtmlTargetElement]`](xref:Microsoft.AspNetCore.Razor.TagHelpers.HtmlTargetElementAttribute) attributo alla classe. Specificare il nome dell'elemento HTML di destinazione.
* *Facoltativo*: applicare un [`[EditorBrowsable(EditorBrowsableState.Never)]`](xref:System.ComponentModel.EditorBrowsableAttribute) attributo alla classe per disattivare la visualizzazione del tipo in IntelliSense.

Il codice seguente crea un componente helper tag personalizzato che fa riferimento all'elemento HTML `<address>`:

[!code-csharp[](th-components/samples/RazorPagesSample/TagHelpers/AddressTagHelperComponentTagHelper.cs)]

Usare il componente helper tag personalizzato `address` per inserire markup HTML nel modo seguente:

```csharp
public class AddressTagHelperComponent : TagHelperComponent
{
    private readonly string _printableButton =
        "<button type='button' class='btn btn-info' onclick=\"window.open(" +
        "'https://binged.it/2AXRRYw')\">" +
        "<span class='glyphicon glyphicon-road' aria-hidden='true'></span>" +
        "</button>";

    public override int Order => 3;

    public override async Task ProcessAsync(TagHelperContext context,
                                            TagHelperOutput output)
    {
        if (string.Equals(context.TagName, "address",
                StringComparison.OrdinalIgnoreCase) &&
            output.Attributes.ContainsName("printable"))
        {
            var content = await output.GetChildContentAsync();
            output.Content.SetHtmlContent(
                $"<div>{content.GetContent()}</div>{_printableButton}");
        }
    }
}
```

Il metodo `ProcessAsync` precedente inserisce l'HTML fornito a <xref:Microsoft.AspNetCore.Razor.TagHelpers.TagHelperContent.SetHtmlContent*> nell'elemento `<address>` corrispondente. L'inserimento si verifica quando:

* Il valore della proprietà `TagName` del contesto di esecuzione è uguale a `address`.
* L'elemento `<address>` corrispondente ha un attributo `printable`.

Ad esempio, l'istruzione `if` restituisce true durante l'elaborazione dell'elemento `<address>` seguente:

[!code-cshtml[](th-components/samples/RazorPagesSample/Pages/Contact.cshtml?name=snippet_AddressPrintable)]

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/dependency-injection>
* <xref:mvc/views/dependency-injection>
* <xref:mvc/views/tag-helpers/builtin-th/Index>
