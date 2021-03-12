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
# <a name="partial-tag-helper-in-aspnet-core"></a><span data-ttu-id="a9dc5-103">Helper tag Partial in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="a9dc5-103">Partial Tag Helper in ASP.NET Core</span></span>

<span data-ttu-id="a9dc5-104">Di [Scott Addie](https://github.com/scottaddie)</span><span class="sxs-lookup"><span data-stu-id="a9dc5-104">By [Scott Addie](https://github.com/scottaddie)</span></span>

<span data-ttu-id="a9dc5-105">Per una panoramica degli helper per tag, vedere <xref:mvc/views/tag-helpers/intro>.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-105">For an overview of Tag Helpers, see <xref:mvc/views/tag-helpers/intro>.</span></span>

<span data-ttu-id="a9dc5-106">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/built-in/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="a9dc5-106">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/built-in/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="overview"></a><span data-ttu-id="a9dc5-107">Panoramica</span><span class="sxs-lookup"><span data-stu-id="a9dc5-107">Overview</span></span>

<span data-ttu-id="a9dc5-108">L'helper per tag parziale viene usato per il rendering di una [visualizzazione parziale](xref:mvc/views/partial) in Razor pagine e app MVC.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-108">The Partial Tag Helper is used for rendering a [partial view](xref:mvc/views/partial) in Razor Pages and MVC apps.</span></span> <span data-ttu-id="a9dc5-109">Tenere presente che:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-109">Consider that it:</span></span>

* <span data-ttu-id="a9dc5-110">Richiede ASP.NET Core 2.1 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-110">Requires ASP.NET Core 2.1 or later.</span></span>
* <span data-ttu-id="a9dc5-111">Rappresenta un'alternativa alla [sintassi helper HTML](xref:mvc/views/partial#reference-a-partial-view).</span><span class="sxs-lookup"><span data-stu-id="a9dc5-111">Is an alternative to [HTML Helper syntax](xref:mvc/views/partial#reference-a-partial-view).</span></span>
* <span data-ttu-id="a9dc5-112">Esegue il rendering della visualizzazione parziale in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-112">Renders the partial view asynchronously.</span></span>

<span data-ttu-id="a9dc5-113">Le opzioni helper HTML per il rendering di una visualizzazione parziale includono:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-113">The HTML Helper options for rendering a partial view include:</span></span>

* [`@await Html.PartialAsync`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partialasync)
* [`@await Html.RenderPartialAsync`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartialasync)
* [`@Html.Partial`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partial)
* [`@Html.RenderPartial`](/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartial)

<span data-ttu-id="a9dc5-114">Il modello *Product* è usato negli esempi di questo documento:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-114">The *Product* model is used in samples throughout this document:</span></span>

[!code-csharp[](samples/TagHelpersBuiltIn/Models/Product.cs)]

<span data-ttu-id="a9dc5-115">Segue un inventario degli attributi dell'helper tag Partial.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-115">An inventory of the Partial Tag Helper attributes follows.</span></span>

## <a name="name"></a><span data-ttu-id="a9dc5-116">name</span><span class="sxs-lookup"><span data-stu-id="a9dc5-116">name</span></span>

<span data-ttu-id="a9dc5-117">L'attributo `name` è obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-117">The `name` attribute is required.</span></span> <span data-ttu-id="a9dc5-118">Indica il nome o il percorso della visualizzazione parziale di cui eseguire il rendering.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-118">It indicates the name or the path of the partial view to be rendered.</span></span> <span data-ttu-id="a9dc5-119">Quando viene offerto un nome della visualizzazione parziale, viene avviato il processo di [individuazione delle visualizzazioni](xref:mvc/views/overview#view-discovery).</span><span class="sxs-lookup"><span data-stu-id="a9dc5-119">When a partial view name is provided, the [view discovery](xref:mvc/views/overview#view-discovery) process is initiated.</span></span> <span data-ttu-id="a9dc5-120">Tale processo viene ignorato quando viene offerto un percorso esplicito.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-120">That process is bypassed when an explicit path is provided.</span></span> <span data-ttu-id="a9dc5-121">Per tutti i valori `name` accettabili, vedere [Individuazione delle visualizzazioni parziali](xref:mvc/views/partial#partial-view-discovery).</span><span class="sxs-lookup"><span data-stu-id="a9dc5-121">For all acceptable `name` values, see [Partial view discovery](xref:mvc/views/partial#partial-view-discovery).</span></span>

<span data-ttu-id="a9dc5-122">Il markup seguente usa un percorso esplicito, che indica che *_ProductPartial.cshtml* deve essere caricato dalla cartella *Shared*.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-122">The following markup uses an explicit path, indicating that *_ProductPartial.cshtml* is to be loaded from the *Shared* folder.</span></span> <span data-ttu-id="a9dc5-123">Mediante l'attributo [for](#for) un modello viene passato alla visualizzazione parziale per l'associazione.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-123">Using the [for](#for) attribute, a model is passed to the partial view for binding.</span></span>

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_Name)]

## <a name="for"></a><span data-ttu-id="a9dc5-124">per</span><span class="sxs-lookup"><span data-stu-id="a9dc5-124">for</span></span>

<span data-ttu-id="a9dc5-125">L'attributo `for` assegna una [ModelExpression](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.modelexpression) da valutare rispetto al modello corrente.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-125">The `for` attribute assigns a [ModelExpression](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.modelexpression) to be evaluated against the current model.</span></span> <span data-ttu-id="a9dc5-126">Un elemento `ModelExpression` deduce la sintassi `@Model.`.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-126">A `ModelExpression` infers the `@Model.` syntax.</span></span> <span data-ttu-id="a9dc5-127">Ad esempio `for="Product"` può essere usato invece di `for="@Model.Product"`.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-127">For example, `for="Product"` can be used instead of `for="@Model.Product"`.</span></span> <span data-ttu-id="a9dc5-128">Questo comportamento di inferenza predefinito può essere ignorato se si usa il simbolo `@` per definire un'espressione inline.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-128">This default inference behavior is overridden by using the `@` symbol to define an inline expression.</span></span>

<span data-ttu-id="a9dc5-129">Il markup seguente carica *_ProductPartial.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-129">The following markup loads *_ProductPartial.cshtml*:</span></span>

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_For)]

<span data-ttu-id="a9dc5-130">La visualizzazione parziale è correlata alla proprietà `Product` del modello di pagina associato:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-130">The partial view is bound to the associated page model's `Product` property:</span></span>

[!code-csharp[](samples/TagHelpersBuiltIn/Pages/Product.cshtml.cs?highlight=8)]

## <a name="model"></a><span data-ttu-id="a9dc5-131">model</span><span class="sxs-lookup"><span data-stu-id="a9dc5-131">model</span></span>

<span data-ttu-id="a9dc5-132">L'attributo `model` assegna un'istanza del modello da passare alla visualizzazione parziale.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-132">The `model` attribute assigns a model instance to pass to the partial view.</span></span> <span data-ttu-id="a9dc5-133">L'attributo `model` non può essere usato con l'attributo [for](#for).</span><span class="sxs-lookup"><span data-stu-id="a9dc5-133">The `model` attribute can't be used with the [for](#for) attribute.</span></span>

<span data-ttu-id="a9dc5-134">Nel markup seguente viene creata un'istanza di un nuovo oggetto `Product` che viene quindi passata all'attributo `model` per il binding:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-134">In the following markup, a new `Product` object is instantiated and passed to the `model` attribute for binding:</span></span>

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_Model)]

## <a name="view-data"></a><span data-ttu-id="a9dc5-135">view-data</span><span class="sxs-lookup"><span data-stu-id="a9dc5-135">view-data</span></span>

<span data-ttu-id="a9dc5-136">L'attributo `view-data` assegna un [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) da passare alla visualizzazione parziale.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-136">The `view-data` attribute assigns a [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) to pass to the partial view.</span></span> <span data-ttu-id="a9dc5-137">Il markup seguente rende l'intera raccolta ViewData accessibile alla visualizzazione parziale:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-137">The following markup makes the entire ViewData collection accessible to the partial view:</span></span>

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Product.cshtml?name=snippet_ViewData&highlight=5-)]

<span data-ttu-id="a9dc5-138">Nel codice precedente, il valore della chiave `IsNumberReadOnly` è impostato su `true` e aggiunto alla raccolta ViewData.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-138">In the preceding code, the `IsNumberReadOnly` key value is set to `true` and added to the ViewData collection.</span></span> <span data-ttu-id="a9dc5-139">Di conseguenza l'elemento `ViewData["IsNumberReadOnly"]` viene reso accessibile all'interno della visualizzazione parziale seguente:</span><span class="sxs-lookup"><span data-stu-id="a9dc5-139">Consequently, `ViewData["IsNumberReadOnly"]` is made accessible within the following partial view:</span></span>

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Shared/_ProductViewDataPartial.cshtml?highlight=5)]

<span data-ttu-id="a9dc5-140">In questo esempio il valore di `ViewData["IsNumberReadOnly"]` determina se il campo *Number* viene visualizzato come campo di sola lettura.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-140">In this example, the value of `ViewData["IsNumberReadOnly"]` determines whether the *Number* field is displayed as read only.</span></span>

## <a name="migrate-from-an-html-helper"></a><span data-ttu-id="a9dc5-141">Eseguire la migrazione da un helper HTML</span><span class="sxs-lookup"><span data-stu-id="a9dc5-141">Migrate from an HTML Helper</span></span>

<span data-ttu-id="a9dc5-142">Osservare l'esempio di helper HTML asincrono seguente.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-142">Consider the following asynchronous HTML Helper example.</span></span> <span data-ttu-id="a9dc5-143">Viene eseguita l'iterazione e visualizzata una raccolta di prodotti.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-143">A collection of products is iterated and displayed.</span></span> <span data-ttu-id="a9dc5-144">Per il primo parametro del metodo `PartialAsync`, viene caricata la visualizzazione parziale *_ProductPartial.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-144">Per the `PartialAsync` method's first parameter, the *_ProductPartial.cshtml* partial view is loaded.</span></span> <span data-ttu-id="a9dc5-145">Un'istanza del modello `Product` viene passata alla visualizzazione parziale per l'associazione.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-145">An instance of the `Product` model is passed to the partial view for binding.</span></span>

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Products.cshtml?name=snippet_HtmlHelper&highlight=3)]

<span data-ttu-id="a9dc5-146">L'helper tag Partial seguente consente di ottenere lo stesso comportamento asincrono dell'helper HTML `PartialAsync`.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-146">The following Partial Tag Helper achieves the same asynchronous rendering behavior as the `PartialAsync` HTML Helper.</span></span> <span data-ttu-id="a9dc5-147">All'attributo `model` viene assegnata un'istanza del modello `Product` per l'associazione alla visualizzazione parziale.</span><span class="sxs-lookup"><span data-stu-id="a9dc5-147">The `model` attribute is assigned a `Product` model instance for binding to the partial view.</span></span>

[!code-cshtml[](samples/TagHelpersBuiltIn/Pages/Products.cshtml?name=snippet_TagHelper&highlight=3)]

## <a name="additional-resources"></a><span data-ttu-id="a9dc5-148">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a9dc5-148">Additional resources</span></span>

* <xref:mvc/views/partial>
* <xref:mvc/views/overview#weakly-typed-data-viewdata-viewdata-attribute-and-viewbag>
