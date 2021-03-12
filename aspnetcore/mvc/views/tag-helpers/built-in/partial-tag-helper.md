---
title: Helper tag Partial in ASP.NET Core
author: scottaddie
description: Informazioni sull'helper tag Partial di ASP.NET Core e sul ruolo dei singoli attributi dell'helper nel rendering di una visualizzazione parziale.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 04/06/2019
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
uid: mvc/views/tag-helpers/builtin-th/partial-tag-helper
ms.openlocfilehash: 184901ad0bb6188ed908d41dabf2433c5ca7c1ce
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587164"
---
# <a name="partial-tag-helper-in-aspnet-core"></a>Helper tag Partial in ASP.NET Core

Di [Scott Addie](https://github.com/scottaddie)

Per una panoramica degli helper per tag, vedere <xref:mvc/views/tag-helpers/intro>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/built-in/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="overview"></a>Panoramica

L'helper per tag parziale viene usato per il rendering di una [visualizzazione parziale](xref:mvc/views/partial) in Razor pagine e app MVC. Tenere presente che:

* Richiede ASP.NET Core 2.1 o versione successiva.
* Rappresenta un'alternativa alla [sintassi helper HTML](xref:mvc/views/partial#reference-a-partial-view).
* Esegue il rendering della visualizzazione parziale in modo asincrono.

Le opzioni helper HTML per il rendering di una visualizzazione parziale includono:

* [`@await Html.PartialAsync`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partialasync)
* [`@await Html.RenderPartialAsync`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartialasync)
* [`@Html.Partial`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partial)
* [`@Html.RenderPartial`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartial)

Il modello *Product* è usato negli esempi di questo documento:

[!code-csharp[](samples/TagHelpersBuiltIn/Models/Product.cs)]

Segue un inventario degli attributi dell'helper tag Partial.

## <a name="name"></a>name

L'attributo `name` è obbligatorio. Indica il nome o il percorso della visualizzazione parziale di cui eseguire il rendering. Quando viene offerto un nome della visualizzazione parziale, viene avviato il processo di [individuazione delle visualizzazioni](xref:mvc/views/overview#view-discovery). Tale processo viene ignorato quando viene offerto un percorso esplicito. Per tutti i valori `name` accettabili, vedere [Individuazione delle visualizzazioni parziali](xref:mvc/views/partial#partial-view-discovery).

Il markup seguente usa un percorso esplicito, che indica che *_ProductPartial.cshtml* deve essere caricato dalla cartella *Shared*. Mediante l'attributo [for](#for) un modello viene passato alla visualizzazione parziale per l'associazione.

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_Name)]

## <a name="for"></a>per

L'attributo `for` assegna una [ModelExpression](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.modelexpression) da valutare rispetto al modello corrente. Un elemento `ModelExpression` deduce la sintassi `@Model.`. Ad esempio `for="Product"` può essere usato invece di `for="@Model.Product"`. Questo comportamento di inferenza predefinito può essere ignorato se si usa il simbolo `@` per definire un'espressione inline.

Il markup seguente carica *_ProductPartial.cshtml*:

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_For)]

La visualizzazione parziale è correlata alla proprietà `Product` del modello di pagina associato:

[!code-csharp[](samples/TagHelpersBuiltIn/Pages/Product.cshtml.cs?highlight=8)]

## <a name="model"></a>model

L'attributo `model` assegna un'istanza del modello da passare alla visualizzazione parziale. L'attributo `model` non può essere usato con l'attributo [for](#for).

Nel markup seguente viene creata un'istanza di un nuovo oggetto `Product` che viene quindi passata all'attributo `model` per il binding:

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_Model)]

## <a name="view-data"></a>view-data

L'attributo `view-data` assegna un [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) da passare alla visualizzazione parziale. Il markup seguente rende l'intera raccolta ViewData accessibile alla visualizzazione parziale:

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_ViewData&highlight=5-)]

Nel codice precedente, il valore della chiave `IsNumberReadOnly` è impostato su `true` e aggiunto alla raccolta ViewData. Di conseguenza l'elemento `ViewData["IsNumberReadOnly"]` viene reso accessibile all'interno della visualizzazione parziale seguente:

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Shared/_ProductViewDataPartial.cshtml?highlight=5)]

In questo esempio il valore di `ViewData["IsNumberReadOnly"]` determina se il campo *Number* viene visualizzato come campo di sola lettura.

## <a name="migrate-from-an-html-helper"></a>Eseguire la migrazione da un helper HTML

Osservare l'esempio di helper HTML asincrono seguente. Viene eseguita l'iterazione e visualizzata una raccolta di prodotti. Per il primo parametro del metodo `PartialAsync`, viene caricata la visualizzazione parziale *_ProductPartial.cshtml*. Un'istanza del modello `Product` viene passata alla visualizzazione parziale per l'associazione.

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Products.cshtml?name=snippet_HtmlHelper&highlight=3)]

L'helper tag Partial seguente consente di ottenere lo stesso comportamento asincrono dell'helper HTML `PartialAsync`. All'attributo `model` viene assegnata un'istanza del modello `Product` per l'associazione alla visualizzazione parziale.

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Products.cshtml?name=snippet_TagHelper&highlight=3)]

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/views/partial>
* <xref:mvc/views/overview#weakly-typed-data-viewdata-viewdata-attribute-and-viewbag>
