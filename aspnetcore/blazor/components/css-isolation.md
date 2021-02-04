---
title: BlazorIsolamento ASP.NET Core CSS
author: daveabrock
description: Scopri in che modo l'isolamento CSS ti permette di definire l'ambito di CSS per i tuoi componenti, semplificando i tuoi CSS ed evitando conflitti con altri componenti o librerie.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/20/2020
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
uid: blazor/components/css-isolation
ms.openlocfilehash: 0748f606314963788e6733ca2ae2ca2123d839b3
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99529982"
---
# <a name="aspnet-core-blazor-css-isolation"></a><span data-ttu-id="0bdc7-103">BlazorIsolamento ASP.NET Core CSS</span><span class="sxs-lookup"><span data-stu-id="0bdc7-103">ASP.NET Core Blazor CSS isolation</span></span>

<span data-ttu-id="0bdc7-104">Di [Dave Brock](https://twitter.com/daveabrock)</span><span class="sxs-lookup"><span data-stu-id="0bdc7-104">By [Dave Brock](https://twitter.com/daveabrock)</span></span>

<span data-ttu-id="0bdc7-105">L'isolamento CSS semplifica l'impronta CSS di un'app impedendo le dipendenze sugli stili globali e consente di evitare conflitti di stile tra componenti e librerie.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-105">CSS isolation simplifies an app's CSS footprint by preventing dependencies on global styles and helps to avoid styling conflicts among components and libraries.</span></span>

## <a name="enable-css-isolation"></a><span data-ttu-id="0bdc7-106">Abilita isolamento CSS</span><span class="sxs-lookup"><span data-stu-id="0bdc7-106">Enable CSS isolation</span></span> 

<span data-ttu-id="0bdc7-107">Per definire stili specifici del componente, creare un `.razor.css` file corrispondente al nome del `.razor` file per il componente nella stessa cartella.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-107">To define component-specific styles, create a `.razor.css` file matching the name of the `.razor` file for the component in the same folder.</span></span> <span data-ttu-id="0bdc7-108">Il `.razor.css` file è un *file CSS con ambito*.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-108">The `.razor.css` file is a *scoped CSS file*.</span></span> 

<span data-ttu-id="0bdc7-109">Per un `Example` componente in un `Example.razor` file, creare un file insieme al componente denominato `Example.razor.css` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-109">For an `Example` component in an `Example.razor` file, create a file alongside the component named `Example.razor.css`.</span></span> <span data-ttu-id="0bdc7-110">Il `Example.razor.css` file deve risiedere nella stessa cartella del `Example` componente ( `Example.razor` ).</span><span class="sxs-lookup"><span data-stu-id="0bdc7-110">The `Example.razor.css` file must reside in the same folder as the `Example` component (`Example.razor`).</span></span> <span data-ttu-id="0bdc7-111">Il `Example` nome di base del file **non** fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-111">The "`Example`" base name of the file is **not** case-sensitive.</span></span>

<span data-ttu-id="0bdc7-112">`Pages/Example.razor`:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-112">`Pages/Example.razor`:</span></span>

```razor
@page "/example"

<h1>Scoped CSS Example</h1>
```

<span data-ttu-id="0bdc7-113">`Pages/Example.razor.css`:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-113">`Pages/Example.razor.css`:</span></span>

```css
h1 { 
    color: brown;
    font-family: Tahoma, Geneva, Verdana, sans-serif;
}
```

<span data-ttu-id="0bdc7-114">**Gli stili definiti in `Example.razor.css` vengono applicati solo all'output del componente di cui è stato eseguito il rendering `Example` .**</span><span class="sxs-lookup"><span data-stu-id="0bdc7-114">**The styles defined in `Example.razor.css` are only applied to the rendered output of the `Example` component.**</span></span> <span data-ttu-id="0bdc7-115">L'isolamento CSS viene applicato agli elementi HTML nel Razor file corrispondente.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-115">CSS isolation is applied to HTML elements in the matching Razor file.</span></span> <span data-ttu-id="0bdc7-116">Qualsiasi `h1` dichiarazione CSS definita altrove nell'app non è in conflitto con gli `Example` stili del componente.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-116">Any `h1` CSS declarations defined elsewhere in the app don't conflict with the `Example` component's styles.</span></span>

> [!NOTE]
> <span data-ttu-id="0bdc7-117">Per garantire l'isolamento dello stile quando si verifica la creazione di bundle, l'importazione di CSS nei Razor blocchi di codice non è supportata.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-117">In order to guarantee style isolation when bundling occurs, importing CSS in Razor code blocks isn't supported.</span></span>

## <a name="css-isolation-bundling"></a><span data-ttu-id="0bdc7-118">Aggregazione di isolamento CSS</span><span class="sxs-lookup"><span data-stu-id="0bdc7-118">CSS isolation bundling</span></span>

<span data-ttu-id="0bdc7-119">L'isolamento CSS si verifica in fase di compilazione.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-119">CSS isolation occurs at build time.</span></span> <span data-ttu-id="0bdc7-120">Durante questo processo, Blazor riscrive i selettori CSS in modo che corrispondano al markup sottoposto a rendering dal componente.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-120">During this process, Blazor rewrites CSS selectors to match markup rendered by the component.</span></span> <span data-ttu-id="0bdc7-121">Questi stili CSS riscritti sono aggregati e prodotti come asset statico in `{PROJECT NAME}.styles.css` , dove il segnaposto `{PROJECT NAME}` è il nome del pacchetto o del prodotto a cui si fa riferimento.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-121">These rewritten CSS styles are bundled and produced as a static asset at `{PROJECT NAME}.styles.css`, where the placeholder `{PROJECT NAME}` is the referenced package or product name.</span></span>

<span data-ttu-id="0bdc7-122">Per impostazione predefinita, a questi file statici viene fatto riferimento dal percorso radice dell'app.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-122">These static files are referenced from the root path of the app by default.</span></span> <span data-ttu-id="0bdc7-123">Nell'app, fare riferimento al file in bundle controllando il riferimento all'interno del `<head>` tag del codice HTML generato:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-123">In the app, reference the bundled file by inspecting the reference inside the `<head>` tag of the generated HTML:</span></span>

```html
<link href="ProjectName.styles.css" rel="stylesheet">
```

<span data-ttu-id="0bdc7-124">All'interno del file in bundle, ogni componente è associato a un identificatore di ambito.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-124">Within the bundled file, each component is associated with a scope identifier.</span></span> <span data-ttu-id="0bdc7-125">Per ogni componente con stile, viene aggiunto un attributo HTML con il formato `b-<10-character-string>` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-125">For each styled component, an HTML attribute is appended with the format `b-<10-character-string>`.</span></span> <span data-ttu-id="0bdc7-126">L'identificatore è univoco e diverso per ogni app.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-126">The identifier is unique and different for each app.</span></span> <span data-ttu-id="0bdc7-127">Nel componente sottoposto a rendering `Counter` Blazor Accoda un identificatore di ambito all' `h1` elemento:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-127">In the rendered `Counter` component, Blazor appends a scope identifier to the `h1` element:</span></span>

```html
<h1 b-3xxtam6d07>
```

<span data-ttu-id="0bdc7-128">Il `ProjectName.styles.css` file usa l'identificatore di ambito per raggruppare una dichiarazione di stile con il relativo componente.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-128">The `ProjectName.styles.css` file uses the scope identifier to group a style declaration with its component.</span></span> <span data-ttu-id="0bdc7-129">Nell'esempio seguente viene fornito lo stile per l' `<h1>` elemento precedente:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-129">The following example provides the style for the preceding `<h1>` element:</span></span>

```css
/* /Pages/Counter.razor.rz.scp.css */
h1[b-3xxtam6d07] {
    color: brown;
}
```

<span data-ttu-id="0bdc7-130">In fase di compilazione, viene creato un bundle di progetto con la convenzione `{STATIC WEB ASSETS BASE PATH}/Project.lib.scp.css` , dove il segnaposto `{STATIC WEB ASSETS BASE PATH}` è il percorso di base degli asset Web statici.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-130">At build time, a project bundle is created with the convention `{STATIC WEB ASSETS BASE PATH}/Project.lib.scp.css`, where the placeholder `{STATIC WEB ASSETS BASE PATH}` is the static web assets base path.</span></span>

<span data-ttu-id="0bdc7-131">Se vengono usati altri progetti, ad esempio pacchetti NuGet o [ Razor librerie di classi](xref:blazor/components/class-libraries), il file in bundle:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-131">If other projects are utilized, such as NuGet packages or [Razor class libraries](xref:blazor/components/class-libraries), the bundled file:</span></span>

* <span data-ttu-id="0bdc7-132">Fa riferimento agli stili utilizzando le importazioni CSS.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-132">References the styles using CSS imports.</span></span>
* <span data-ttu-id="0bdc7-133">Non viene pubblicato come asset Web statico dell'app che utilizza gli stili.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-133">Isn't published as a static web asset of the app that consumes the styles.</span></span>

## <a name="child-component-support"></a><span data-ttu-id="0bdc7-134">Supporto componenti figlio</span><span class="sxs-lookup"><span data-stu-id="0bdc7-134">Child component support</span></span>

<span data-ttu-id="0bdc7-135">Per impostazione predefinita, l'isolamento CSS si applica solo al componente associato al formato `{COMPONENT NAME}.razor.css` , in cui il segnaposto `{COMPONENT NAME}` è in genere il nome del componente.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-135">By default, CSS isolation only applies to the component you associate with the format `{COMPONENT NAME}.razor.css`, where the placeholder `{COMPONENT NAME}` is usually the component name.</span></span> <span data-ttu-id="0bdc7-136">Per applicare le modifiche a un componente figlio, usare il `::deep` combinatore con tutti gli elementi discendenti nel file del componente padre `.razor.css` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-136">To apply changes to a child component, use the `::deep` combinator to any descendant elements in the parent component's `.razor.css` file.</span></span> <span data-ttu-id="0bdc7-137">Il `::deep` combinatore Seleziona elementi *discendenti* dell'identificatore di ambito generato da un elemento.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-137">The `::deep` combinator selects elements that are *descendants* of an element's generated scope identifier.</span></span> 

<span data-ttu-id="0bdc7-138">Nell'esempio seguente viene illustrato un componente padre denominato `Parent` con un componente figlio denominato `Child` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-138">The following example shows a parent component called `Parent` with a child component called `Child`.</span></span>

<span data-ttu-id="0bdc7-139">`Pages/Parent.razor`:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-139">`Pages/Parent.razor`:</span></span>

```razor
@page "/parent"

<div>
    <h1>Parent component</h1>

    <Child />
</div>
```

<span data-ttu-id="0bdc7-140">`Shared/Child.razor`:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-140">`Shared/Child.razor`:</span></span>

```razor
<h1>Child Component</h1>
```

<span data-ttu-id="0bdc7-141">Aggiornare la `h1` dichiarazione in `Parent.razor.css` con il `::deep` combinatore per indicare che la `h1` dichiarazione di stile deve essere applicata al componente padre e ai relativi elementi figlio.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-141">Update the `h1` declaration in `Parent.razor.css` with the `::deep` combinator to signify the `h1` style declaration must apply to the parent component and its children.</span></span>

<span data-ttu-id="0bdc7-142">`Pages/Parent.razor.css`:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-142">`Pages/Parent.razor.css`:</span></span>

```css
::deep h1 { 
    color: red;
}
```

<span data-ttu-id="0bdc7-143">Lo `h1` stile è ora applicabile ai `Parent` `Child` componenti e senza la necessità di creare un file CSS con ambito separato per il componente figlio.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-143">The `h1` style now applies to the `Parent` and `Child` components without the need to create a separate scoped CSS file for the child component.</span></span>

<span data-ttu-id="0bdc7-144">Il `::deep` combinatore funziona solo con gli elementi discendenti.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-144">The `::deep` combinator only works with descendant elements.</span></span> <span data-ttu-id="0bdc7-145">Il markup seguente applica gli `h1` stili ai componenti come previsto.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-145">The following markup applies the `h1` styles to components as expected.</span></span> <span data-ttu-id="0bdc7-146">L'identificatore di ambito del componente padre viene applicato all' `div` elemento, in modo che il browser sappia ereditare gli stili dal componente padre.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-146">The parent component's scope identifier is applied to the `div` element, so the browser knows to inherit styles from the parent component.</span></span>

<span data-ttu-id="0bdc7-147">`Pages/Parent.razor`:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-147">`Pages/Parent.razor`:</span></span>

```razor
<div>
    <h1>Parent</h1>

    <Child />
</div>
```

<span data-ttu-id="0bdc7-148">Tuttavia, escludendo l' `div` elemento viene rimossa la relazione discendente.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-148">However, excluding the `div` element removes the descendant relationship.</span></span> <span data-ttu-id="0bdc7-149">Nell'esempio seguente lo stile **non** viene applicato al componente figlio.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-149">In the following example, the style is **not** applied to the child component.</span></span>

<span data-ttu-id="0bdc7-150">`Pages/Parent.razor`:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-150">`Pages/Parent.razor`:</span></span>

```razor
<h1>Parent</h1>

<Child />
```

## <a name="css-preprocessor-support"></a><span data-ttu-id="0bdc7-151">Supporto per il preprocessore CSS</span><span class="sxs-lookup"><span data-stu-id="0bdc7-151">CSS preprocessor support</span></span>

<span data-ttu-id="0bdc7-152">I preprocessori CSS sono utili per migliorare lo sviluppo CSS utilizzando funzionalità quali variabili, nidificazione, moduli, mixin ed ereditarietà.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-152">CSS preprocessors are useful for improving CSS development by utilizing features such as variables, nesting, modules, mixins, and inheritance.</span></span> <span data-ttu-id="0bdc7-153">Sebbene l'isolamento CSS non supporti in modo nativo i preprocessori CSS come Sass o less, l'integrazione dei preprocessori CSS è facile purché la compilazione del preprocessore venga eseguita prima che Blazor riscriva i selettori CSS durante il processo di compilazione.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-153">While CSS isolation doesn't natively support CSS preprocessors such as Sass or Less, integrating CSS preprocessors is seamless as long as preprocessor compilation occurs before Blazor rewrites the CSS selectors during the build process.</span></span> <span data-ttu-id="0bdc7-154">Usando Visual Studio, ad esempio, configurare la compilazione del preprocessore esistente come attività **prima della compilazione** in Visual Studio Task Runner Explorer.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-154">Using Visual Studio for example, configure existing preprocessor compilation as a **Before Build** task in the Visual Studio Task Runner Explorer.</span></span>

<span data-ttu-id="0bdc7-155">Molti pacchetti NuGet di terze parti, ad esempio [delegate. SassBuilder](https://www.nuget.org/packages/Delegate.SassBuilder), possono compilare i file Sass/scss all'inizio del processo di compilazione prima che si verifichi l'isolamento CSS e non è necessaria alcuna configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-155">Many third-party NuGet packages, such as [Delegate.SassBuilder](https://www.nuget.org/packages/Delegate.SassBuilder), can compile SASS/SCSS files at the beginning of the build process before CSS isolation occurs, and no additional configuration is required.</span></span>

## <a name="css-isolation-configuration"></a><span data-ttu-id="0bdc7-156">Configurazione dell'isolamento CSS</span><span class="sxs-lookup"><span data-stu-id="0bdc7-156">CSS isolation configuration</span></span>

<span data-ttu-id="0bdc7-157">L'isolamento CSS è progettato per l'uso predefinito, ma fornisce la configurazione per alcuni scenari avanzati, ad esempio quando sono presenti dipendenze da strumenti o flussi di lavoro esistenti.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-157">CSS isolation is designed to work out-of-the-box but provides configuration for some advanced scenarios, such as when there are dependencies on existing tools or workflows.</span></span>

### <a name="customize-scope-identifier-format"></a><span data-ttu-id="0bdc7-158">Personalizzare il formato dell'identificatore dell'ambito</span><span class="sxs-lookup"><span data-stu-id="0bdc7-158">Customize scope identifier format</span></span>

<span data-ttu-id="0bdc7-159">Per impostazione predefinita, gli identificatori di ambito usano il formato `b-<10-character-string>` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-159">By default, scope identifiers use the format `b-<10-character-string>`.</span></span> <span data-ttu-id="0bdc7-160">Per personalizzare il formato dell'identificatore di ambito, aggiornare il file di progetto in un modello desiderato:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-160">To customize the scope identifier format, update the project file to a desired pattern:</span></span>

```xml
<ItemGroup>
  <None Update="Pages/Example.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

<span data-ttu-id="0bdc7-161">Nell'esempio precedente, il CSS generato per `Example.Razor.css` modifica l'identificatore di ambito da `b-<10-character-string>` a `my-custom-scope-identifier` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-161">In the preceding example, the CSS generated for `Example.Razor.css` changes its scope identifier from `b-<10-character-string>` to `my-custom-scope-identifier`.</span></span>

<span data-ttu-id="0bdc7-162">Usare gli identificatori di ambito per ottenere l'ereditarietà con file CSS con ambito.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-162">Use scope identifiers to achieve inheritance with scoped CSS files.</span></span> <span data-ttu-id="0bdc7-163">Nell'esempio di file di progetto seguente un `BaseComponent.razor.css` file contiene stili comuni tra i componenti.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-163">In the following project file example, a `BaseComponent.razor.css` file contains common styles across components.</span></span> <span data-ttu-id="0bdc7-164">Un `DerivedComponent.razor.css` file eredita questi stili.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-164">A `DerivedComponent.razor.css` file inherits these styles.</span></span>

```xml
<ItemGroup>
  <None Update="Pages/BaseComponent.razor.css" CssScope="my-custom-scope-identifier" />
  <None Update="Pages/DerivedComponent.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

<span data-ttu-id="0bdc7-165">Usare l'operatore jolly ( `*` ) per condividere gli identificatori di ambito tra più file:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-165">Use the wildcard (`*`) operator to share scope identifiers across multiple files:</span></span>

```xml
<ItemGroup>
  <None Update="Pages/*.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

### <a name="change-base-path-for-static-web-assets"></a><span data-ttu-id="0bdc7-166">Modificare il percorso di base per gli asset Web statici</span><span class="sxs-lookup"><span data-stu-id="0bdc7-166">Change base path for static web assets</span></span>

<span data-ttu-id="0bdc7-167">Il `scoped.styles.css` file viene generato alla radice dell'app.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-167">The `scoped.styles.css` file is generated at the root of the app.</span></span> <span data-ttu-id="0bdc7-168">Nel file di progetto, utilizzare la `StaticWebAssetBasePath` proprietà per modificare il percorso predefinito.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-168">In the project file, use the `StaticWebAssetBasePath` property to change the default path.</span></span> <span data-ttu-id="0bdc7-169">L'esempio seguente posiziona il `scoped.styles.css` file e il resto degli asset dell'app nel `_content` percorso:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-169">The following example places the `scoped.styles.css` file, and the rest of the app's assets, at the `_content` path:</span></span>

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>_content/$(PackageId)</StaticWebAssetBasePath>
</PropertyGroup>
```

### <a name="disable-automatic-bundling"></a><span data-ttu-id="0bdc7-170">Disabilitare la creazione automatica di bundle</span><span class="sxs-lookup"><span data-stu-id="0bdc7-170">Disable automatic bundling</span></span>

<span data-ttu-id="0bdc7-171">Per rifiutare esplicitamente la modalità di Blazor pubblicazione e caricamento dei file con ambito in fase di esecuzione, utilizzare la `DisableScopedCssBundling` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-171">To opt out of how Blazor publishes and loads scoped files at runtime, use the `DisableScopedCssBundling` property.</span></span> <span data-ttu-id="0bdc7-172">Quando si usa questa proprietà, altri strumenti o processi sono responsabili dell'acquisizione dei file CSS isolati dalla `obj` Directory e della pubblicazione e del caricamento in fase di esecuzione:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-172">When using this property, it means other tools or processes are responsible for taking the isolated CSS files from the `obj` directory and publishing and loading them at runtime:</span></span>

```xml
<PropertyGroup>
  <DisableScopedCssBundling>true</DisableScopedCssBundling>
</PropertyGroup>
```

## <a name="razor-class-library-rcl-support"></a><span data-ttu-id="0bdc7-173">Razor supporto della libreria di classi (RCL)</span><span class="sxs-lookup"><span data-stu-id="0bdc7-173">Razor class library (RCL) support</span></span>

<span data-ttu-id="0bdc7-174">Quando una [ Razor libreria di classi (RCL)](xref:razor-pages/ui-class) fornisce stili isolati, l' `<link>` attributo del tag `href` punta a `{STATIC WEB ASSET BASE PATH}/{ASSEMBLY NAME}.bundle.scp.css` , dove i segnaposto sono:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-174">When a [Razor class library (RCL)](xref:razor-pages/ui-class) provides isolated styles, the `<link>` tag's `href` attribute points to `{STATIC WEB ASSET BASE PATH}/{ASSEMBLY NAME}.bundle.scp.css`, where the placeholders are:</span></span>

* <span data-ttu-id="0bdc7-175">`{STATIC WEB ASSET BASE PATH}`: Percorso di base dell'asset Web statico.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-175">`{STATIC WEB ASSET BASE PATH}`: The static web asset base path.</span></span>
* <span data-ttu-id="0bdc7-176">`{ASSEMBLY NAME}`: Nome dell'assembly della libreria di classi.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-176">`{ASSEMBLY NAME}`: The class library's assembly name.</span></span>

<span data-ttu-id="0bdc7-177">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-177">In the following example:</span></span>

* <span data-ttu-id="0bdc7-178">Il percorso di base dell'asset Web statico è `_content/ClassLib` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-178">The static web asset base path is `_content/ClassLib`.</span></span>
* <span data-ttu-id="0bdc7-179">Il nome dell'assembly della libreria di classi è `ClassLib` .</span><span class="sxs-lookup"><span data-stu-id="0bdc7-179">The class library's assembly name is `ClassLib`.</span></span>

<span data-ttu-id="0bdc7-180">`wwwroot/index.html` ( Blazor WebAssembly ) o `Pages_Host.cshtml` (Blazor Server):</span><span class="sxs-lookup"><span data-stu-id="0bdc7-180">`wwwroot/index.html` (Blazor WebAssembly) or `Pages_Host.cshtml` (Blazor Server):</span></span>

```html
<link href="_content/ClassLib/ClassLib.bundle.scp.css" rel="stylesheet">
```

<span data-ttu-id="0bdc7-181">Per ulteriori informazioni sulle librerie di componenti e RCL, vedere:</span><span class="sxs-lookup"><span data-stu-id="0bdc7-181">For more information on RCLs and component libraries, see:</span></span>

* <xref:razor-pages/ui-class>
* <span data-ttu-id="0bdc7-182"><xref:blazor/components/class-libraries>.</span><span class="sxs-lookup"><span data-stu-id="0bdc7-182"><xref:blazor/components/class-libraries>.</span></span>
