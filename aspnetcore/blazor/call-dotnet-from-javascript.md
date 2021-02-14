---
title: Chiamare i metodi .NET da funzioni JavaScript in ASP.NET Core Blazor
author: guardrex
description: Informazioni su come richiamare i metodi .NET dalle funzioni JavaScript nelle Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 08/12/2020
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
uid: blazor/call-dotnet-from-javascript
ms.openlocfilehash: 45ddcc9e006df2c5e86a7859efc76882b269a496
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280396"
---
# <a name="call-net-methods-from-javascript-functions-in-aspnet-core-blazor"></a><span data-ttu-id="262de-103">Chiamare i metodi .NET da funzioni JavaScript in ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="262de-103">Call .NET methods from JavaScript functions in ASP.NET Core Blazor</span></span>

<span data-ttu-id="262de-104">Un' Blazor app può richiamare funzioni JavaScript da metodi .NET e metodi .NET da funzioni JavaScript.</span><span class="sxs-lookup"><span data-stu-id="262de-104">A Blazor app can invoke JavaScript functions from .NET methods and .NET methods from JavaScript functions.</span></span> <span data-ttu-id="262de-105">Questi scenari sono detti *interoperabilità JavaScript (interoperabilità* *JS*).</span><span class="sxs-lookup"><span data-stu-id="262de-105">These scenarios are called *JavaScript interoperability* (*JS interop*).</span></span>

<span data-ttu-id="262de-106">Questo articolo descrive come richiamare i metodi .NET da JavaScript.</span><span class="sxs-lookup"><span data-stu-id="262de-106">This article covers invoking .NET methods from JavaScript.</span></span> <span data-ttu-id="262de-107">Per informazioni su come chiamare funzioni JavaScript da .NET, vedere <xref:blazor/call-javascript-from-dotnet> .</span><span class="sxs-lookup"><span data-stu-id="262de-107">For information on how to call JavaScript functions from .NET, see <xref:blazor/call-javascript-from-dotnet>.</span></span>

<span data-ttu-id="262de-108">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="262de-108">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

> [!NOTE]
> <span data-ttu-id="262de-109">Aggiungere i file JS ( `<script>` tag) prima del `</body>` tag di chiusura nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ).</span><span class="sxs-lookup"><span data-stu-id="262de-109">Add JS files (`<script>` tags) before the closing `</body>` tag in the `wwwroot/index.html` file (Blazor WebAssembly) or `Pages/_Host.cshtml` file (Blazor Server).</span></span> <span data-ttu-id="262de-110">Assicurarsi che i file JS con i metodi di interoperabilità JS siano inclusi prima dei Blazor file di framework JS.</span><span class="sxs-lookup"><span data-stu-id="262de-110">Ensure that JS files with JS interop methods are included before Blazor framework JS files.</span></span>

## <a name="static-net-method-call"></a><span data-ttu-id="262de-111">Chiamata al metodo .NET statico</span><span class="sxs-lookup"><span data-stu-id="262de-111">Static .NET method call</span></span>

<span data-ttu-id="262de-112">Per richiamare un metodo .NET statico da JavaScript, usare le `DotNet.invokeMethod` `DotNet.invokeMethodAsync` funzioni o.</span><span class="sxs-lookup"><span data-stu-id="262de-112">To invoke a static .NET method from JavaScript, use the `DotNet.invokeMethod` or `DotNet.invokeMethodAsync` functions.</span></span> <span data-ttu-id="262de-113">Passare l'identificatore del metodo statico che si desidera chiamare, il nome dell'assembly che contiene la funzione e gli eventuali argomenti.</span><span class="sxs-lookup"><span data-stu-id="262de-113">Pass in the identifier of the static method you wish to call, the name of the assembly containing the function, and any arguments.</span></span> <span data-ttu-id="262de-114">Per supportare gli scenari è preferibile la versione asincrona Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="262de-114">The asynchronous version is preferred to support Blazor Server scenarios.</span></span> <span data-ttu-id="262de-115">Il metodo .NET deve essere pubblico, statico e avere l' [ `[JSInvokable]` attributo](xref:Microsoft.JSInterop.JSInvokableAttribute).</span><span class="sxs-lookup"><span data-stu-id="262de-115">The .NET method must be public, static, and have the [`[JSInvokable]` attribute](xref:Microsoft.JSInterop.JSInvokableAttribute).</span></span> <span data-ttu-id="262de-116">La chiamata ai metodi generici aperti non è attualmente supportata.</span><span class="sxs-lookup"><span data-stu-id="262de-116">Calling open generic methods isn't currently supported.</span></span>

<span data-ttu-id="262de-117">L'app di esempio include un metodo C# per restituire una `int` matrice.</span><span class="sxs-lookup"><span data-stu-id="262de-117">The sample app includes a C# method to return an `int` array.</span></span> <span data-ttu-id="262de-118">L' [ `[JSInvokable]` attributo](xref:Microsoft.JSInterop.JSInvokableAttribute) viene applicato al metodo.</span><span class="sxs-lookup"><span data-stu-id="262de-118">The [`[JSInvokable]` attribute](xref:Microsoft.JSInterop.JSInvokableAttribute) is applied to the method.</span></span>

<span data-ttu-id="262de-119">`Pages/JsInterop.razor`:</span><span class="sxs-lookup"><span data-stu-id="262de-119">`Pages/JsInterop.razor`:</span></span>

```razor
<button type="button" class="btn btn-primary"
        onclick="exampleJsFunctions.returnArrayAsyncJs()">
    Trigger .NET static method ReturnArrayAsync
</button>

@code {
    [JSInvokable]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

<span data-ttu-id="262de-120">JavaScript servito al client richiama il metodo C# .NET.</span><span class="sxs-lookup"><span data-stu-id="262de-120">JavaScript served to the client invokes the C# .NET method.</span></span>

<span data-ttu-id="262de-121">`wwwroot/exampleJsInterop.js`:</span><span class="sxs-lookup"><span data-stu-id="262de-121">`wwwroot/exampleJsInterop.js`:</span></span>

[!code-javascript[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=8-14)]

<span data-ttu-id="262de-122">Quando **`Trigger .NET static method ReturnArrayAsync`** si seleziona il pulsante, esaminare l'output della console negli strumenti di sviluppo Web del browser.</span><span class="sxs-lookup"><span data-stu-id="262de-122">When the **`Trigger .NET static method ReturnArrayAsync`** button is selected, examine the console output in the browser's web developer tools.</span></span>

<span data-ttu-id="262de-123">L'output della console è:</span><span class="sxs-lookup"><span data-stu-id="262de-123">The console output is:</span></span>

```console
Array(4) [ 1, 2, 3, 4 ]
```

<span data-ttu-id="262de-124">Il quarto valore della matrice viene inserito nella matrice ( `data.push(4);` ) restituito da `ReturnArrayAsync` .</span><span class="sxs-lookup"><span data-stu-id="262de-124">The fourth array value is pushed to the array (`data.push(4);`) returned by `ReturnArrayAsync`.</span></span>

<span data-ttu-id="262de-125">Per impostazione predefinita, l'identificatore del metodo è il nome del metodo, ma è possibile specificare un identificatore diverso usando il costruttore dell' [ `[JSInvokable]` attributo](xref:Microsoft.JSInterop.JSInvokableAttribute) :</span><span class="sxs-lookup"><span data-stu-id="262de-125">By default, the method identifier is the method name, but you can specify a different identifier using the [`[JSInvokable]` attribute](xref:Microsoft.JSInterop.JSInvokableAttribute) constructor:</span></span>

```csharp
@code {
    [JSInvokable("DifferentMethodName")]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

<span data-ttu-id="262de-126">Nel file JavaScript sul lato client:</span><span class="sxs-lookup"><span data-stu-id="262de-126">In the client-side JavaScript file:</span></span>

```javascript
returnArrayAsyncJs: function () {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'DifferentMethodName')
    .then(data => {
      data.push(4);
      console.log(data);
    });
}
```

<span data-ttu-id="262de-127">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="262de-127">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

## <a name="instance-method-call"></a><span data-ttu-id="262de-128">Chiamata al metodo di istanza</span><span class="sxs-lookup"><span data-stu-id="262de-128">Instance method call</span></span>

<span data-ttu-id="262de-129">È anche possibile chiamare i metodi di istanza .NET da JavaScript.</span><span class="sxs-lookup"><span data-stu-id="262de-129">You can also call .NET instance methods from JavaScript.</span></span> <span data-ttu-id="262de-130">Per richiamare un metodo di istanza .NET da JavaScript:</span><span class="sxs-lookup"><span data-stu-id="262de-130">To invoke a .NET instance method from JavaScript:</span></span>

* <span data-ttu-id="262de-131">Passa l'istanza .NET per riferimento a JavaScript:</span><span class="sxs-lookup"><span data-stu-id="262de-131">Pass the .NET instance by reference to JavaScript:</span></span>
  * <span data-ttu-id="262de-132">Eseguire una chiamata statica a <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="262de-132">Make a static call to <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A?displayProperty=nameWithType>.</span></span>
  * <span data-ttu-id="262de-133">Eseguire il wrapping dell'istanza in un' <xref:Microsoft.JSInterop.DotNetObjectReference> istanza di e chiamare <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A> sull' <xref:Microsoft.JSInterop.DotNetObjectReference> istanza.</span><span class="sxs-lookup"><span data-stu-id="262de-133">Wrap the instance in a <xref:Microsoft.JSInterop.DotNetObjectReference> instance and call <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A> on the <xref:Microsoft.JSInterop.DotNetObjectReference> instance.</span></span> <span data-ttu-id="262de-134">Eliminazione di <xref:Microsoft.JSInterop.DotNetObjectReference> oggetti (un esempio viene visualizzato più avanti in questa sezione).</span><span class="sxs-lookup"><span data-stu-id="262de-134">Dispose of <xref:Microsoft.JSInterop.DotNetObjectReference> objects (an example appears later in this section).</span></span>
* <span data-ttu-id="262de-135">Richiamare i metodi di istanza .NET sull'istanza usando `invokeMethod` le `invokeMethodAsync` funzioni o.</span><span class="sxs-lookup"><span data-stu-id="262de-135">Invoke .NET instance methods on the instance using the `invokeMethod` or `invokeMethodAsync` functions.</span></span> <span data-ttu-id="262de-136">L'istanza di .NET può essere passata anche come argomento quando si richiamano altri metodi .NET da JavaScript.</span><span class="sxs-lookup"><span data-stu-id="262de-136">The .NET instance can also be passed as an argument when invoking other .NET methods from JavaScript.</span></span>

> [!NOTE]
> <span data-ttu-id="262de-137">L'app di esempio consente di registrare i messaggi nella console lato client.</span><span class="sxs-lookup"><span data-stu-id="262de-137">The sample app logs messages to the client-side console.</span></span> <span data-ttu-id="262de-138">Per gli esempi seguenti illustrati dall'app di esempio, esaminare l'output della console del browser negli strumenti di sviluppo del browser.</span><span class="sxs-lookup"><span data-stu-id="262de-138">For the following examples demonstrated by the sample app, examine the browser's console output in the browser's developer tools.</span></span>

<span data-ttu-id="262de-139">Quando il **`Trigger .NET instance method HelloHelper.SayHello`** pulsante è selezionato, `ExampleJsInterop.CallHelloHelperSayHello` viene chiamato e passa un nome `Blazor` al metodo.</span><span class="sxs-lookup"><span data-stu-id="262de-139">When the **`Trigger .NET instance method HelloHelper.SayHello`** button is selected, `ExampleJsInterop.CallHelloHelperSayHello` is called and passes a name, `Blazor`, to the method.</span></span>

<span data-ttu-id="262de-140">`Pages/JsInterop.razor`:</span><span class="sxs-lookup"><span data-stu-id="262de-140">`Pages/JsInterop.razor`:</span></span>

```razor
<button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
    Trigger .NET instance method HelloHelper.SayHello
</button>

@code {
    public async Task TriggerNetInstanceMethod()
    {
        var exampleJsInterop = new ExampleJsInterop(JS);
        await exampleJsInterop.CallHelloHelperSayHello("Blazor");
    }
}
```

<span data-ttu-id="262de-141">`CallHelloHelperSayHello` richiama la funzione JavaScript `sayHello` con una nuova istanza di `HelloHelper` .</span><span class="sxs-lookup"><span data-stu-id="262de-141">`CallHelloHelperSayHello` invokes the JavaScript function `sayHello` with a new instance of `HelloHelper`.</span></span>

<span data-ttu-id="262de-142">`JsInteropClasses/ExampleJsInterop.cs`:</span><span class="sxs-lookup"><span data-stu-id="262de-142">`JsInteropClasses/ExampleJsInterop.cs`:</span></span>

[!code-csharp[](./common/samples/5.x/BlazorWebAssemblySample/JsInteropClasses/ExampleJsInterop.cs?name=snippet1&highlight=11-18)]

<span data-ttu-id="262de-143">`wwwroot/exampleJsInterop.js`:</span><span class="sxs-lookup"><span data-stu-id="262de-143">`wwwroot/exampleJsInterop.js`:</span></span>

[!code-javascript[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=15-18)]

<span data-ttu-id="262de-144">Il nome viene passato al `HelloHelper` costruttore, che imposta la `HelloHelper.Name` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="262de-144">The name is passed to `HelloHelper`'s constructor, which sets the `HelloHelper.Name` property.</span></span> <span data-ttu-id="262de-145">Quando viene eseguita la funzione JavaScript `sayHello` , `HelloHelper.SayHello` restituisce il `Hello, {Name}!` messaggio, che viene scritto nella console dalla funzione JavaScript.</span><span class="sxs-lookup"><span data-stu-id="262de-145">When the JavaScript function `sayHello` is executed, `HelloHelper.SayHello` returns the `Hello, {Name}!` message, which is written to the console by the JavaScript function.</span></span>

<span data-ttu-id="262de-146">`JsInteropClasses/HelloHelper.cs`:</span><span class="sxs-lookup"><span data-stu-id="262de-146">`JsInteropClasses/HelloHelper.cs`:</span></span>

[!code-csharp[](./common/samples/5.x/BlazorWebAssemblySample/JsInteropClasses/HelloHelper.cs?name=snippet1&highlight=5,10-11)]

<span data-ttu-id="262de-147">Output della console negli strumenti di sviluppo Web del browser:</span><span class="sxs-lookup"><span data-stu-id="262de-147">Console output in the browser's web developer tools:</span></span>

```console
Hello, Blazor!
```

<span data-ttu-id="262de-148">Per evitare una perdita di memoria e consentire Garbage Collection su un componente che crea un <xref:Microsoft.JSInterop.DotNetObjectReference> , adottare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="262de-148">To avoid a memory leak and allow garbage collection on a component that creates a <xref:Microsoft.JSInterop.DotNetObjectReference>, adopt one of the following approaches:</span></span>

* <span data-ttu-id="262de-149">Eliminare l'oggetto nella classe che ha creato l' <xref:Microsoft.JSInterop.DotNetObjectReference> istanza:</span><span class="sxs-lookup"><span data-stu-id="262de-149">Dispose of the object in the class that created the <xref:Microsoft.JSInterop.DotNetObjectReference> instance:</span></span>

  ```csharp
  public class ExampleJsInterop : IDisposable
  {
      private readonly IJSRuntime js;
      private DotNetObjectReference<HelloHelper> objRef;

      public ExampleJsInterop(IJSRuntime js)
      {
          this.js = js;
      }

      public ValueTask<string> CallHelloHelperSayHello(string name)
      {
          objRef = DotNetObjectReference.Create(new HelloHelper(name));

          return js.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```

  <span data-ttu-id="262de-150">Il modello precedente illustrato nella `ExampleJsInterop` classe può anche essere implementato in un componente:</span><span class="sxs-lookup"><span data-stu-id="262de-150">The preceding pattern shown in the `ExampleJsInterop` class can also be implemented in a component:</span></span>

  ```razor
  @page "/JSInteropComponent"
  @using {APP ASSEMBLY}.JsInteropClasses
  @implements IDisposable
  @inject IJSRuntime JS

  <h1>JavaScript Interop</h1>

  <button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
      Trigger .NET instance method HelloHelper.SayHello
  </button>

  @code {
      private DotNetObjectReference<HelloHelper> objRef;

      public async Task TriggerNetInstanceMethod()
      {
          objRef = DotNetObjectReference.Create(new HelloHelper("Blazor"));

          await JS.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```
  
  <span data-ttu-id="262de-151">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="262de-151">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

* <span data-ttu-id="262de-152">Quando il componente o la classe non Elimina <xref:Microsoft.JSInterop.DotNetObjectReference> , eliminare l'oggetto nel client chiamando `.dispose()` :</span><span class="sxs-lookup"><span data-stu-id="262de-152">When the component or class doesn't dispose of the <xref:Microsoft.JSInterop.DotNetObjectReference>, dispose of the object on the client by calling `.dispose()`:</span></span>

  ```javascript
  window.myFunction = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'MyMethod');
    dotnetHelper.dispose();
  }
  ```

## <a name="component-instance-method-call"></a><span data-ttu-id="262de-153">Chiamata al metodo dell'istanza del componente</span><span class="sxs-lookup"><span data-stu-id="262de-153">Component instance method call</span></span>

<span data-ttu-id="262de-154">Per richiamare i metodi .NET di un componente:</span><span class="sxs-lookup"><span data-stu-id="262de-154">To invoke a component's .NET methods:</span></span>

* <span data-ttu-id="262de-155">Utilizzare la `invokeMethod` `invokeMethodAsync` funzione o per eseguire una chiamata al metodo statico al componente.</span><span class="sxs-lookup"><span data-stu-id="262de-155">Use the `invokeMethod` or `invokeMethodAsync` function to make a static method call to the component.</span></span>
* <span data-ttu-id="262de-156">Il metodo statico del componente esegue il wrapping della chiamata al metodo di istanza come oggetto richiamato <xref:System.Action> .</span><span class="sxs-lookup"><span data-stu-id="262de-156">The component's static method wraps the call to its instance method as an invoked <xref:System.Action>.</span></span>

> [!NOTE]
> <span data-ttu-id="262de-157">Per Blazor Server le app, in cui più utenti possono usare contemporaneamente lo stesso componente, usare una classe helper per richiamare i metodi di istanza.</span><span class="sxs-lookup"><span data-stu-id="262de-157">For Blazor Server apps, where several users might be concurrently using the same component, use a helper class to invoke instance methods.</span></span>
>
> <span data-ttu-id="262de-158">Per ulteriori informazioni, vedere la sezione relativa alla [classe helper del metodo di istanza Component](#component-instance-method-helper-class) .</span><span class="sxs-lookup"><span data-stu-id="262de-158">For more information, see the [Component instance method helper class](#component-instance-method-helper-class) section.</span></span>

<span data-ttu-id="262de-159">Nel codice JavaScript lato client:</span><span class="sxs-lookup"><span data-stu-id="262de-159">In the client-side JavaScript:</span></span>

```javascript
function updateMessageCallerJS() {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
}
```

<span data-ttu-id="262de-160">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="262de-160">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

<span data-ttu-id="262de-161">`Pages/JSInteropComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="262de-161">`Pages/JSInteropComponent.razor`:</span></span>

```razor
@page "/JSInteropComponent"

<p>
    Message: @message
</p>

<p>
    <button onclick="updateMessageCallerJS()">Call JS Method</button>
</p>

@code {
    private static Action action;
    private string message = "Select the button.";

    protected override void OnInitialized()
    {
        action = UpdateMessage;
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        StateHasChanged();
    }

    [JSInvokable]
    public static void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

<span data-ttu-id="262de-162">Per passare argomenti al metodo di istanza:</span><span class="sxs-lookup"><span data-stu-id="262de-162">To pass arguments to the instance method:</span></span>

* <span data-ttu-id="262de-163">Aggiungere parametri alla chiamata del metodo JS.</span><span class="sxs-lookup"><span data-stu-id="262de-163">Add parameters to the JS method invocation.</span></span> <span data-ttu-id="262de-164">Nell'esempio seguente un nome viene passato al metodo.</span><span class="sxs-lookup"><span data-stu-id="262de-164">In the following example, a name is passed to the method.</span></span> <span data-ttu-id="262de-165">È possibile aggiungere altri parametri all'elenco in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="262de-165">Additional parameters can be added to the list as needed.</span></span>

  ```javascript
  function updateMessageCallerJS(name) {
    DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller', name);
  }
  ```
  
  <span data-ttu-id="262de-166">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="262de-166">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

* <span data-ttu-id="262de-167">Fornire i tipi corretti a <xref:System.Action> per i parametri.</span><span class="sxs-lookup"><span data-stu-id="262de-167">Provide the correct types to the <xref:System.Action> for the parameters.</span></span> <span data-ttu-id="262de-168">Fornire l'elenco di parametri ai metodi di C#.</span><span class="sxs-lookup"><span data-stu-id="262de-168">Provide the parameter list to the C# methods.</span></span> <span data-ttu-id="262de-169">Richiamare <xref:System.Action> ( `UpdateMessage` ) con i parametri ( `action.Invoke(name)` ).</span><span class="sxs-lookup"><span data-stu-id="262de-169">Invoke the <xref:System.Action> (`UpdateMessage`) with the parameters (`action.Invoke(name)`).</span></span>

  <span data-ttu-id="262de-170">`Pages/JSInteropComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="262de-170">`Pages/JSInteropComponent.razor`:</span></span>

  ```razor
  @page "/JSInteropComponent"

  <p>
      Message: @message
  </p>

  <p>
      <button onclick="updateMessageCallerJS('Sarah Jane')">
          Call JS Method
      </button>
  </p>

  @code {
      private static Action<string> action;
      private string message = "Select the button.";

      protected override void OnInitialized()
      {
          action = UpdateMessage;
      }

      private void UpdateMessage(string name)
      {
          message = $"{name}, UpdateMessage Called!";
          StateHasChanged();
      }

      [JSInvokable]
      public static void UpdateMessageCaller(string name)
      {
          action.Invoke(name);
      }
  }
  ```

  <span data-ttu-id="262de-171">Output `message` quando viene selezionato il pulsante **chiama metodo JS** :</span><span class="sxs-lookup"><span data-stu-id="262de-171">Output `message` when the **Call JS Method** button is selected:</span></span>

  ```
  Sarah Jane, UpdateMessage Called!
  ```

## <a name="component-instance-method-helper-class"></a><span data-ttu-id="262de-172">Classe helper del metodo di istanza Component</span><span class="sxs-lookup"><span data-stu-id="262de-172">Component instance method helper class</span></span>

<span data-ttu-id="262de-173">La classe helper viene utilizzata per richiamare un metodo di istanza come <xref:System.Action> .</span><span class="sxs-lookup"><span data-stu-id="262de-173">The helper class is used to invoke an instance method as an <xref:System.Action>.</span></span> <span data-ttu-id="262de-174">Le classi helper sono utili nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="262de-174">Helper classes are useful when:</span></span>

* <span data-ttu-id="262de-175">Viene eseguito il rendering di diversi componenti dello stesso tipo nella stessa pagina.</span><span class="sxs-lookup"><span data-stu-id="262de-175">Several components of the same type are rendered on the same page.</span></span>
* <span data-ttu-id="262de-176">Blazor ServerViene usata un'app, in cui più utenti potrebbero usare un componente simultaneamente.</span><span class="sxs-lookup"><span data-stu-id="262de-176">A Blazor Server app is used, where multiple users might be using a component concurrently.</span></span>

<span data-ttu-id="262de-177">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="262de-177">In the following example:</span></span>

* <span data-ttu-id="262de-178">Il `JSInteropExample` componente contiene diversi `ListItem` componenti.</span><span class="sxs-lookup"><span data-stu-id="262de-178">The `JSInteropExample` component contains several `ListItem` components.</span></span>
* <span data-ttu-id="262de-179">Ogni `ListItem` componente è costituito da un messaggio e da un pulsante.</span><span class="sxs-lookup"><span data-stu-id="262de-179">Each `ListItem` component is composed of a message and a button.</span></span>
* <span data-ttu-id="262de-180">Quando `ListItem` viene selezionato un pulsante componente, `ListItem` `UpdateMessage` il metodo modifica il testo dell'elemento dell'elenco e lo nasconde.</span><span class="sxs-lookup"><span data-stu-id="262de-180">When a `ListItem` component button is selected, that `ListItem`'s `UpdateMessage` method changes the list item text and hides the button.</span></span>

<span data-ttu-id="262de-181">`MessageUpdateInvokeHelper.cs`:</span><span class="sxs-lookup"><span data-stu-id="262de-181">`MessageUpdateInvokeHelper.cs`:</span></span>

```csharp
using System;
using Microsoft.JSInterop;

public class MessageUpdateInvokeHelper
{
    private Action action;

    public MessageUpdateInvokeHelper(Action action)
    {
        this.action = action;
    }

    [JSInvokable("{APP ASSEMBLY}")]
    public void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

<span data-ttu-id="262de-182">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="262de-182">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

<span data-ttu-id="262de-183">Nel codice JavaScript lato client:</span><span class="sxs-lookup"><span data-stu-id="262de-183">In the client-side JavaScript:</span></span>

```javascript
window.updateMessageCallerJS = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
    dotnetHelper.dispose();
}
```

<span data-ttu-id="262de-184">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="262de-184">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

<span data-ttu-id="262de-185">`Shared/ListItem.razor`:</span><span class="sxs-lookup"><span data-stu-id="262de-185">`Shared/ListItem.razor`:</span></span>

```razor
@inject IJSRuntime JS

<li>
    @message
    <button @onclick="InteropCall" style="display:@display">InteropCall</button>
</li>

@code {
    private string message = "Select one of these list item buttons.";
    private string display = "inline-block";
    private MessageUpdateInvokeHelper messageUpdateInvokeHelper;

    protected override void OnInitialized()
    {
        messageUpdateInvokeHelper = new MessageUpdateInvokeHelper(UpdateMessage);
    }

    protected async Task InteropCall()
    {
        await JS.InvokeVoidAsync("updateMessageCallerJS",
            DotNetObjectReference.Create(messageUpdateInvokeHelper));
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        display = "none";
        StateHasChanged();
    }
}
```

<span data-ttu-id="262de-186">`Pages/JSInteropExample.razor`:</span><span class="sxs-lookup"><span data-stu-id="262de-186">`Pages/JSInteropExample.razor`:</span></span>

```razor
@page "/JSInteropExample"

<h1>List of components</h1>

<ul>
    <ListItem />
    <ListItem />
    <ListItem />
    <ListItem />
</ul>
```

[!INCLUDE[](~/blazor/includes/share-interop-code.md)]

## <a name="avoid-circular-object-references"></a><span data-ttu-id="262de-187">Evitare riferimenti a oggetti circolari</span><span class="sxs-lookup"><span data-stu-id="262de-187">Avoid circular object references</span></span>

<span data-ttu-id="262de-188">Gli oggetti che contengono riferimenti circolari non possono essere serializzati sul client per uno dei seguenti:</span><span class="sxs-lookup"><span data-stu-id="262de-188">Objects that contain circular references can't be serialized on the client for either:</span></span>

* <span data-ttu-id="262de-189">Chiamate al metodo .NET.</span><span class="sxs-lookup"><span data-stu-id="262de-189">.NET method calls.</span></span>
* <span data-ttu-id="262de-190">Chiamate al metodo JavaScript da C# quando il tipo restituito contiene riferimenti circolari.</span><span class="sxs-lookup"><span data-stu-id="262de-190">JavaScript method calls from C# when the return type has circular references.</span></span>

<span data-ttu-id="262de-191">Per ulteriori informazioni, vedere i seguenti problemi:</span><span class="sxs-lookup"><span data-stu-id="262de-191">For more information, see the following issues:</span></span>

* [<span data-ttu-id="262de-192">I riferimenti circolari non sono supportati, accettano due (DotNet/aspnetcore #20525)</span><span class="sxs-lookup"><span data-stu-id="262de-192">Circular references are not supported, take two (dotnet/aspnetcore #20525)</span></span>](https://github.com/dotnet/aspnetcore/issues/20525)
* [<span data-ttu-id="262de-193">Proposta: aggiungere un meccanismo per gestire i riferimenti circolari durante la serializzazione (DotNet/Runtime #30820)</span><span class="sxs-lookup"><span data-stu-id="262de-193">Proposal: Add mechanism to handle circular references when serializing (dotnet/runtime #30820)</span></span>](https://github.com/dotnet/runtime/issues/30820)

## <a name="size-limits-on-js-interop-calls"></a><span data-ttu-id="262de-194">Limiti delle dimensioni per le chiamate di interoperabilità JS</span><span class="sxs-lookup"><span data-stu-id="262de-194">Size limits on JS interop calls</span></span>

<span data-ttu-id="262de-195">In Blazor WebAssembly il Framework non impone un limite per le dimensioni degli input e degli output di interoperabilità js.</span><span class="sxs-lookup"><span data-stu-id="262de-195">In Blazor WebAssembly, the framework doesn't impose a limit on the size of JS interop inputs and outputs.</span></span>

<span data-ttu-id="262de-196">In Blazor Server le chiamate di interoperabilità JS hanno dimensioni limitate per la dimensione massima dei messaggi in ingresso SignalR consentita per i metodi dell'hub, che viene applicata da <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> (valore predefinito: 32 KB).</span><span class="sxs-lookup"><span data-stu-id="262de-196">In Blazor Server, JS interop calls are limited in size by the maximum incoming SignalR message size permitted for hub methods, which is enforced by <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> (default: 32 KB).</span></span> <span data-ttu-id="262de-197">I messaggi da JS a .NET SignalR maggiori di <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> generano un errore.</span><span class="sxs-lookup"><span data-stu-id="262de-197">JS to .NET SignalR messages larger than <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> throw an error.</span></span> <span data-ttu-id="262de-198">Il Framework non impone un limite per le dimensioni di un SignalR messaggio dall'hub a un client.</span><span class="sxs-lookup"><span data-stu-id="262de-198">The framework doesn't impose a limit on the size of a SignalR message from the hub to a client.</span></span>

<span data-ttu-id="262de-199">Quando SignalR la registrazione non è impostata su [debug](xref:Microsoft.Extensions.Logging.LogLevel) o [Trace](xref:Microsoft.Extensions.Logging.LogLevel), un errore relativo alle dimensioni del messaggio viene visualizzato solo nella console degli strumenti di sviluppo del browser:</span><span class="sxs-lookup"><span data-stu-id="262de-199">When SignalR logging isn't set to [Debug](xref:Microsoft.Extensions.Logging.LogLevel) or [Trace](xref:Microsoft.Extensions.Logging.LogLevel), a message size error only appears in the browser's developer tools console:</span></span>

> <span data-ttu-id="262de-200">Errore: connessione disconnessa con errore "errore: il server ha restituito un errore durante la chiusura: la connessione è stata chiusa con un errore."</span><span class="sxs-lookup"><span data-stu-id="262de-200">Error: Connection disconnected with error 'Error: Server returned an error on close: Connection closed with an error.'.</span></span>

<span data-ttu-id="262de-201">Quando la [ SignalR registrazione lato server](xref:signalr/diagnostics#server-side-logging) è impostata su [debug](xref:Microsoft.Extensions.Logging.LogLevel) o [Trace](xref:Microsoft.Extensions.Logging.LogLevel), la registrazione lato server presenta un <xref:System.IO.InvalidDataException> errore di dimensione del messaggio.</span><span class="sxs-lookup"><span data-stu-id="262de-201">When [SignalR server-side logging](xref:signalr/diagnostics#server-side-logging) is set to [Debug](xref:Microsoft.Extensions.Logging.LogLevel) or [Trace](xref:Microsoft.Extensions.Logging.LogLevel), server-side logging surfaces an <xref:System.IO.InvalidDataException> for a message size error.</span></span>

<span data-ttu-id="262de-202">`appsettings.Development.json`:</span><span class="sxs-lookup"><span data-stu-id="262de-202">`appsettings.Development.json`:</span></span>

```json
{
  "DetailedErrors": true,
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.AspNetCore.SignalR": "Debug"
    }
  }
}
```

> <span data-ttu-id="262de-203">System. IO. InvalidDataException: è stata superata la dimensione massima del messaggio di 32768B.</span><span class="sxs-lookup"><span data-stu-id="262de-203">System.IO.InvalidDataException: The maximum message size of 32768B was exceeded.</span></span> <span data-ttu-id="262de-204">La dimensione del messaggio può essere configurata in AddHubOptions.</span><span class="sxs-lookup"><span data-stu-id="262de-204">The message size can be configured in AddHubOptions.</span></span>

<span data-ttu-id="262de-205">Aumentare il limite impostando <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="262de-205">Increase the limit by setting <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="262de-206">Nell'esempio seguente viene impostata la dimensione massima del messaggio di ricezione su 64 KB (64 \* 1024):</span><span class="sxs-lookup"><span data-stu-id="262de-206">The following example sets the maximum receive message size to 64 KB (64 \* 1024):</span></span>

```csharp
services.AddServerSideBlazor()
   .AddHubOptions(options => options.MaximumReceiveMessageSize = 64 * 1024);
```

<span data-ttu-id="262de-207">L'aumento del SignalR limite delle dimensioni dei messaggi in ingresso comporta il costo della richiesta di più risorse del server ed espone il server a maggiori rischi da parte di utenti malintenzionati.</span><span class="sxs-lookup"><span data-stu-id="262de-207">Increasing the SignalR incoming message size limit comes at the cost of requiring more server resources, and it exposes the server to increased risks from a malicious user.</span></span> <span data-ttu-id="262de-208">Inoltre, la lettura di una grande quantità di contenuto in memoria come stringhe o matrici di byte può comportare anche l'utilizzo di allocazioni che funzionano in modo non corretto con la Garbage Collector, con conseguente penalizzazione delle prestazioni.</span><span class="sxs-lookup"><span data-stu-id="262de-208">Additionally, reading a large amount of content in to memory as strings or byte arrays can also result in allocations that work poorly with the garbage collector, resulting in additional performance penalties.</span></span>

<span data-ttu-id="262de-209">Un'opzione per la lettura di payload di grandi dimensioni consiste nell'inviare il contenuto in blocchi più piccoli ed elaborare il payload come <xref:System.IO.Stream> .</span><span class="sxs-lookup"><span data-stu-id="262de-209">One option for reading large payloads is to send the content in smaller chunks and process the payload as a <xref:System.IO.Stream>.</span></span> <span data-ttu-id="262de-210">Può essere usato durante la lettura di payload JSON di grandi dimensioni o se i dati sono disponibili in JavaScript come byte non elaborati.</span><span class="sxs-lookup"><span data-stu-id="262de-210">This can be used when reading large JSON payloads or if data is available in JavaScript as raw bytes.</span></span> <span data-ttu-id="262de-211">Per un esempio che illustra l'invio di payload binari di grandi dimensioni in Blazor Server che usa tecniche simili al `InputFile` componente, vedere l' [app di esempio binario Submit](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/BinarySubmit).</span><span class="sxs-lookup"><span data-stu-id="262de-211">For an example that demonstrates sending large binary payloads in Blazor Server that uses techniques similar to the `InputFile` component, see the [Binary Submit sample app](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/BinarySubmit).</span></span>

<span data-ttu-id="262de-212">Quando si sviluppa codice che trasferisce una grande quantità di dati tra JavaScript e, tenere presenti le linee guida seguenti Blazor :</span><span class="sxs-lookup"><span data-stu-id="262de-212">Consider the following guidance when developing code that transfers a large amount of data between JavaScript and Blazor:</span></span>

* <span data-ttu-id="262de-213">Sezionare i dati in parti più piccole e inviare i segmenti di dati in sequenza fino a quando non vengono ricevuti tutti i dati dal server.</span><span class="sxs-lookup"><span data-stu-id="262de-213">Slice the data into smaller pieces, and send the data segments sequentially until all of the data is received by the server.</span></span>
* <span data-ttu-id="262de-214">Non allocare oggetti di grandi dimensioni in codice JavaScript e C#.</span><span class="sxs-lookup"><span data-stu-id="262de-214">Don't allocate large objects in JavaScript and C# code.</span></span>
* <span data-ttu-id="262de-215">Non bloccare il thread principale dell'interfaccia utente per periodi prolungati durante l'invio o la ricezione di dati.</span><span class="sxs-lookup"><span data-stu-id="262de-215">Don't block the main UI thread for long periods when sending or receiving data.</span></span>
* <span data-ttu-id="262de-216">Liberare la memoria utilizzata quando il processo viene completato o annullato.</span><span class="sxs-lookup"><span data-stu-id="262de-216">Free any memory consumed when the process is completed or cancelled.</span></span>
* <span data-ttu-id="262de-217">Per motivi di sicurezza, applicare i seguenti requisiti aggiuntivi:</span><span class="sxs-lookup"><span data-stu-id="262de-217">Enforce the following additional requirements for security purposes:</span></span>
  * <span data-ttu-id="262de-218">Dichiarare le dimensioni massime di file o dati che è possibile passare.</span><span class="sxs-lookup"><span data-stu-id="262de-218">Declare the maximum file or data size that can be passed.</span></span>
  * <span data-ttu-id="262de-219">Dichiarare la velocità di caricamento minima dal client al server.</span><span class="sxs-lookup"><span data-stu-id="262de-219">Declare the minimum upload rate from the client to the server.</span></span>
* <span data-ttu-id="262de-220">Dopo la ricezione dei dati da parte del server, i dati possono essere:</span><span class="sxs-lookup"><span data-stu-id="262de-220">After the data is received by the server, the data can be:</span></span>
  * <span data-ttu-id="262de-221">Archiviati temporaneamente in un buffer di memoria fino a quando non vengono raccolti tutti i segmenti.</span><span class="sxs-lookup"><span data-stu-id="262de-221">Temporarily stored in a memory buffer until all of the segments are collected.</span></span>
  * <span data-ttu-id="262de-222">Utilizzato immediatamente.</span><span class="sxs-lookup"><span data-stu-id="262de-222">Consumed immediately.</span></span> <span data-ttu-id="262de-223">Ad esempio, i dati possono essere archiviati immediatamente in un database o scritti su disco durante la ricezione di ogni segmento.</span><span class="sxs-lookup"><span data-stu-id="262de-223">For example, the data can be stored immediately in a database or written to disk as each segment is received.</span></span>

## <a name="js-modules"></a><span data-ttu-id="262de-224">Moduli JS</span><span class="sxs-lookup"><span data-stu-id="262de-224">JS modules</span></span>

<span data-ttu-id="262de-225">Per l'isolamento JS, l'interoperabilità JS funziona con il supporto predefinito del browser per i [moduli ECMAScript (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([specifica ECMAScript](https://tc39.es/ecma262/#sec-modules)).</span><span class="sxs-lookup"><span data-stu-id="262de-225">For JS isolation, JS interop works with the browser's default support for [EcmaScript modules (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([ECMAScript specification](https://tc39.es/ecma262/#sec-modules)).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="262de-226">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="262de-226">Additional resources</span></span>

* <xref:blazor/call-javascript-from-dotnet>
* [<span data-ttu-id="262de-227">`InteropComponent.razor` esempio (repository GitHub DotNet/AspNetCore, ramo di versione 3,1)</span><span class="sxs-lookup"><span data-stu-id="262de-227">`InteropComponent.razor` example (dotnet/AspNetCore GitHub repository, 3.1 release branch)</span></span>](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
