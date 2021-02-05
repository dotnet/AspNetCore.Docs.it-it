<span data-ttu-id="82d30-101">Mentre un'app del server blazer è prerendering, alcune azioni, ad esempio la chiamata a JavaScript, non sono possibili perché non è stata stabilita una connessione con il browser.</span><span class="sxs-lookup"><span data-stu-id="82d30-101">While a Blazor Server app is prerendering, certain actions, such as calling into JavaScript, aren't possible because a connection with the browser hasn't been established.</span></span> <span data-ttu-id="82d30-102">I componenti potrebbero dover eseguire il rendering in modo diverso quando ne viene eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="82d30-102">Components may need to render differently when prerendered.</span></span>

<span data-ttu-id="82d30-103">Per ritardare le chiamate di interoperabilità JavaScript finché non viene stabilita la connessione con il browser, è possibile usare l'evento del ciclo di vita del [componente OnAfterRenderAsync](xref:blazor/components/lifecycle#after-component-render).</span><span class="sxs-lookup"><span data-stu-id="82d30-103">To delay JavaScript interop calls until after the connection with the browser is established, you can use the [OnAfterRenderAsync component lifecycle event](xref:blazor/components/lifecycle#after-component-render).</span></span> <span data-ttu-id="82d30-104">Questo evento viene chiamato solo dopo che viene eseguito il rendering completo dell'app e viene stabilita la connessione client.</span><span class="sxs-lookup"><span data-stu-id="82d30-104">This event is only called after the app is fully rendered and the client connection is established.</span></span>

```cshtml
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<div @ref="divElement">Text during render</div>

@code {
    private ElementReference divElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await JSRuntime.InvokeVoidAsync(
                "setElementText", divElement, "Text after render");
        }
    }
}
```

<span data-ttu-id="82d30-105">Per il codice di esempio precedente, fornire una `setElementText` funzione JavaScript all'interno dell' `<head>` elemento di `wwwroot/index.html` (webassembly Blazer) o `Pages/_Host.cshtml` (server Blazer).</span><span class="sxs-lookup"><span data-stu-id="82d30-105">For the preceding example code, provide a `setElementText` JavaScript function inside the `<head>` element of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server).</span></span> <span data-ttu-id="82d30-106">La funzione viene chiamata con <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> e non restituisce un valore:</span><span class="sxs-lookup"><span data-stu-id="82d30-106">The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> and doesn't return a value:</span></span>

```html
<script>
  window.setElementText = (element, text) => element.innerText = text;
</script>
```

> [!WARNING]
> <span data-ttu-id="82d30-107">Nell'esempio precedente la Document Object Model (DOM) viene modificata direttamente a scopo dimostrativo.</span><span class="sxs-lookup"><span data-stu-id="82d30-107">The preceding example modifies the Document Object Model (DOM) directly for demonstration purposes only.</span></span> <span data-ttu-id="82d30-108">La modifica diretta del DOM con JavaScript non è consigliata nella maggior parte degli scenari perché JavaScript può interferire con il rilevamento delle modifiche di Blazer.</span><span class="sxs-lookup"><span data-stu-id="82d30-108">Directly modifying the DOM with JavaScript isn't recommended in most scenarios because JavaScript can interfere with Blazor's change tracking.</span></span>

<span data-ttu-id="82d30-109">Nel componente seguente viene illustrato come utilizzare l'interoperabilità JavaScript come parte della logica di inizializzazione di un componente in modo che sia compatibile con il prerendering.</span><span class="sxs-lookup"><span data-stu-id="82d30-109">The following component demonstrates how to use JavaScript interop as part of a component's initialization logic in a way that's compatible with prerendering.</span></span> <span data-ttu-id="82d30-110">Il componente indica che è possibile attivare un aggiornamento di rendering dall'interno di <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="82d30-110">The component shows that it's possible to trigger a rendering update from inside <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>.</span></span> <span data-ttu-id="82d30-111">Lo sviluppatore deve evitare di creare un ciclo infinito in questo scenario.</span><span class="sxs-lookup"><span data-stu-id="82d30-111">The developer must avoid creating an infinite loop in this scenario.</span></span>

<span data-ttu-id="82d30-112">Dove <xref:Microsoft.JSInterop.JSRuntime.InvokeAsync%2A?displayProperty=nameWithType> viene chiamato, `ElementRef` viene usato solo in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> e non in nessun metodo del ciclo di vita precedente perché non è presente alcun elemento JavaScript finché non viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="82d30-112">Where <xref:Microsoft.JSInterop.JSRuntime.InvokeAsync%2A?displayProperty=nameWithType> is called, `ElementRef` is only used in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> and not in any earlier lifecycle method because there's no JavaScript element until after the component is rendered.</span></span>

<span data-ttu-id="82d30-113">[StateHasChanged](xref:blazor/components/lifecycle#state-changes) viene chiamato per eseguire nuovamente il rendering del componente con il nuovo stato ottenuto dalla chiamata di interoperabilità JavaScript (per ulteriori informazioni, vedere <xref:blazor/components/rendering> ).</span><span class="sxs-lookup"><span data-stu-id="82d30-113">[StateHasChanged](xref:blazor/components/lifecycle#state-changes) is called to rerender the component with the new state obtained from the JavaScript interop call (for more information, see <xref:blazor/components/rendering>).</span></span> <span data-ttu-id="82d30-114">Il codice non crea un ciclo infinito perché `StateHasChanged` viene chiamato solo quando `infoFromJs` è `null` .</span><span class="sxs-lookup"><span data-stu-id="82d30-114">The code doesn't create an infinite loop because `StateHasChanged` is only called when `infoFromJs` is `null`.</span></span>

```cshtml
@page "/prerendered-interop"
@using Microsoft.AspNetCore.Components
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<p>
    Get value via JS interop call:
    <strong id="val-get-by-interop">@(infoFromJs ?? "No value yet")</strong>
</p>

Set value via JS interop call:
<div id="val-set-by-interop" @ref="divElement"></div>

@code {
    private string infoFromJs;
    private ElementReference divElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender && infoFromJs == null)
        {
            infoFromJs = await JSRuntime.InvokeAsync<string>(
                "setElementText", divElement, "Hello from interop call!");

            StateHasChanged();
        }
    }
}
```

<span data-ttu-id="82d30-115">Per il codice di esempio precedente, fornire una `setElementText` funzione JavaScript all'interno dell' `<head>` elemento di `wwwroot/index.html` (webassembly Blazer) o `Pages/_Host.cshtml` (server Blazer).</span><span class="sxs-lookup"><span data-stu-id="82d30-115">For the preceding example code, provide a `setElementText` JavaScript function inside the `<head>` element of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server).</span></span> <span data-ttu-id="82d30-116">La funzione viene chiamata con <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> e restituisce un valore:</span><span class="sxs-lookup"><span data-stu-id="82d30-116">The function is called with<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> and returns a value:</span></span>

```html
<script>
  window.setElementText = (element, text) => {
    element.innerText = text;
    return text;
  };
</script>
```

> [!WARNING]
> <span data-ttu-id="82d30-117">Nell'esempio precedente la Document Object Model (DOM) viene modificata direttamente a scopo dimostrativo.</span><span class="sxs-lookup"><span data-stu-id="82d30-117">The preceding example modifies the Document Object Model (DOM) directly for demonstration purposes only.</span></span> <span data-ttu-id="82d30-118">La modifica diretta del DOM con JavaScript non è consigliata nella maggior parte degli scenari perché JavaScript può interferire con il rilevamento delle modifiche di Blazer.</span><span class="sxs-lookup"><span data-stu-id="82d30-118">Directly modifying the DOM with JavaScript isn't recommended in most scenarios because JavaScript can interfere with Blazor's change tracking.</span></span>
