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
ms.openlocfilehash: 64268fc5e9f05bc3c8d912781b5ffdd3610cbf99
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394564"
---
# <a name="call-net-methods-from-javascript-functions-in-aspnet-core-blazor"></a>Chiamare i metodi .NET da funzioni JavaScript in ASP.NET Core Blazor

Un' Blazor app può richiamare funzioni JavaScript da metodi .NET e metodi .NET da funzioni JavaScript. Questi scenari sono detti *interoperabilità JavaScript (interoperabilità* *JS*).

Questo articolo descrive come richiamare i metodi .NET da JavaScript. Per informazioni su come chiamare funzioni JavaScript da .NET, vedere <xref:blazor/call-javascript-from-dotnet> .

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

> [!NOTE]
> Aggiungere i file JS ( `<script>` tag) prima del `</body>` tag di chiusura nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ). Assicurarsi che i file JS con i metodi di interoperabilità JS siano inclusi prima dei Blazor file di framework JS.

## <a name="static-net-method-call"></a>Chiamata al metodo .NET statico

Per richiamare un metodo .NET statico da JavaScript, usare le `DotNet.invokeMethod` `DotNet.invokeMethodAsync` funzioni o. Passare l'identificatore del metodo statico che si desidera chiamare, il nome dell'assembly che contiene la funzione e gli eventuali argomenti. Per supportare gli scenari è preferibile la versione asincrona Blazor Server . Il metodo .NET deve essere pubblico, statico e avere l' [ `[JSInvokable]` attributo](xref:Microsoft.JSInterop.JSInvokableAttribute). La chiamata ai metodi generici aperti non è attualmente supportata.

L'app di esempio include un metodo C# per restituire una `int` matrice. L' [ `[JSInvokable]` attributo](xref:Microsoft.JSInterop.JSInvokableAttribute) viene applicato al metodo.

`Pages/JsInterop.razor`:

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

JavaScript servito al client richiama il metodo C# .NET.

`wwwroot/exampleJsInterop.js`:

[!code-javascript[](~/blazor/common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=8-14)]

Quando **`Trigger .NET static method ReturnArrayAsync`** si seleziona il pulsante, esaminare l'output della console negli strumenti di sviluppo Web del browser.

L'output della console è:

```console
Array(4) [ 1, 2, 3, 4 ]
```

Il quarto valore della matrice viene inserito nella matrice ( `data.push(4);` ) restituito da `ReturnArrayAsync` .

Per impostazione predefinita, l'identificatore del metodo è il nome del metodo, ma è possibile specificare un identificatore diverso usando il costruttore dell' [ `[JSInvokable]` attributo](xref:Microsoft.JSInterop.JSInvokableAttribute) :

```csharp
@code {
    [JSInvokable("DifferentMethodName")]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

Nel file JavaScript sul lato client:

```javascript
returnArrayAsyncJs: function () {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'DifferentMethodName')
    .then(data => {
      data.push(4);
      console.log(data);
    });
}
```

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

## <a name="instance-method-call"></a>Chiamata al metodo di istanza

È anche possibile chiamare i metodi di istanza .NET da JavaScript. Per richiamare un metodo di istanza .NET da JavaScript:

* Passa l'istanza .NET per riferimento a JavaScript:
  * Eseguire una chiamata statica a <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A?displayProperty=nameWithType> .
  * Eseguire il wrapping dell'istanza in un' <xref:Microsoft.JSInterop.DotNetObjectReference> istanza di e chiamare <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A> sull' <xref:Microsoft.JSInterop.DotNetObjectReference> istanza. Eliminazione di <xref:Microsoft.JSInterop.DotNetObjectReference> oggetti (un esempio viene visualizzato più avanti in questa sezione).
* Richiamare i metodi di istanza .NET sull'istanza usando `invokeMethod` le `invokeMethodAsync` funzioni o. L'istanza di .NET può essere passata anche come argomento quando si richiamano altri metodi .NET da JavaScript.

> [!NOTE]
> L'app di esempio consente di registrare i messaggi nella console lato client. Per gli esempi seguenti illustrati dall'app di esempio, esaminare l'output della console del browser negli strumenti di sviluppo del browser.

Quando il **`Trigger .NET instance method HelloHelper.SayHello`** pulsante è selezionato, `ExampleJsInterop.CallHelloHelperSayHello` viene chiamato e passa un nome `Blazor` al metodo.

`Pages/JsInterop.razor`:

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

`CallHelloHelperSayHello` richiama la funzione JavaScript `sayHello` con una nuova istanza di `HelloHelper` .

`JsInteropClasses/ExampleJsInterop.cs`:

[!code-csharp[](~/blazor/common/samples/5.x/BlazorWebAssemblySample/JsInteropClasses/ExampleJsInterop.cs?name=snippet1&highlight=11-18)]

`wwwroot/exampleJsInterop.js`:

[!code-javascript[](~/blazor/common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=15-18)]

Il nome viene passato al `HelloHelper` costruttore, che imposta la `HelloHelper.Name` Proprietà. Quando viene eseguita la funzione JavaScript `sayHello` , `HelloHelper.SayHello` restituisce il `Hello, {Name}!` messaggio, che viene scritto nella console dalla funzione JavaScript.

`JsInteropClasses/HelloHelper.cs`:

[!code-csharp[](~/blazor/common/samples/5.x/BlazorWebAssemblySample/JsInteropClasses/HelloHelper.cs?name=snippet1&highlight=5,10-11)]

Output della console negli strumenti di sviluppo Web del browser:

```console
Hello, Blazor!
```

Per evitare una perdita di memoria e consentire Garbage Collection su un componente che crea un <xref:Microsoft.JSInterop.DotNetObjectReference> , adottare uno degli approcci seguenti:

* Eliminare l'oggetto nella classe che ha creato l' <xref:Microsoft.JSInterop.DotNetObjectReference> istanza:

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

  Il modello precedente illustrato nella `ExampleJsInterop` classe può anche essere implementato in un componente:

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
  
  Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

* Quando il componente o la classe non Elimina <xref:Microsoft.JSInterop.DotNetObjectReference> , eliminare l'oggetto nel client chiamando `.dispose()` :

  ```javascript
  window.myFunction = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'MyMethod');
    dotnetHelper.dispose();
  }
  ```

## <a name="component-instance-method-call"></a>Chiamata al metodo dell'istanza del componente

Per richiamare i metodi .NET di un componente:

* Utilizzare la `invokeMethod` `invokeMethodAsync` funzione o per eseguire una chiamata al metodo statico al componente.
* Il metodo statico del componente esegue il wrapping della chiamata al metodo di istanza come oggetto richiamato <xref:System.Action> .

> [!NOTE]
> Per Blazor Server le app, in cui più utenti possono usare contemporaneamente lo stesso componente, usare una classe helper per richiamare i metodi di istanza.
>
> Per ulteriori informazioni, vedere la sezione relativa alla [classe helper del metodo di istanza Component](#component-instance-method-helper-class) .

Nel codice JavaScript lato client:

```javascript
function updateMessageCallerJS() {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
}
```

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

`Pages/JSInteropComponent.razor`:

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

Per passare argomenti al metodo di istanza:

* Aggiungere parametri alla chiamata del metodo JS. Nell'esempio seguente un nome viene passato al metodo. È possibile aggiungere altri parametri all'elenco in base alle esigenze.

  ```javascript
  function updateMessageCallerJS(name) {
    DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller', name);
  }
  ```
  
  Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

* Fornire i tipi corretti a <xref:System.Action> per i parametri. Fornire l'elenco di parametri ai metodi di C#. Richiamare <xref:System.Action> ( `UpdateMessage` ) con i parametri ( `action.Invoke(name)` ).

  `Pages/JSInteropComponent.razor`:

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

  Output `message` quando viene selezionato il pulsante **chiama metodo JS** :

  ```
  Sarah Jane, UpdateMessage Called!
  ```

## <a name="component-instance-method-helper-class"></a>Classe helper del metodo di istanza Component

La classe helper viene utilizzata per richiamare un metodo di istanza come <xref:System.Action> . Le classi helper sono utili nei casi seguenti:

* Viene eseguito il rendering di diversi componenti dello stesso tipo nella stessa pagina.
* Blazor ServerViene usata un'app, in cui più utenti potrebbero usare un componente simultaneamente.

Nell'esempio seguente:

* Il `JSInteropExample` componente contiene diversi `ListItem` componenti.
* Ogni `ListItem` componente è costituito da un messaggio e da un pulsante.
* Quando `ListItem` viene selezionato un pulsante componente, `ListItem` `UpdateMessage` il metodo modifica il testo dell'elemento dell'elenco e lo nasconde.

`MessageUpdateInvokeHelper.cs`:

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

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

Nel codice JavaScript lato client:

```javascript
window.updateMessageCallerJS = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
    dotnetHelper.dispose();
}
```

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

`Shared/ListItem.razor`:

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

`Pages/JSInteropExample.razor`:

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

## <a name="avoid-circular-object-references"></a>Evitare riferimenti a oggetti circolari

Gli oggetti che contengono riferimenti circolari non possono essere serializzati sul client per uno dei seguenti:

* Chiamate al metodo .NET.
* Chiamate al metodo JavaScript da C# quando il tipo restituito contiene riferimenti circolari.

Per ulteriori informazioni, vedere i seguenti problemi:

* [I riferimenti circolari non sono supportati, accettano due (DotNet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525)
* [Proposta: aggiungere un meccanismo per gestire i riferimenti circolari durante la serializzazione (DotNet/Runtime #30820)](https://github.com/dotnet/runtime/issues/30820)

## <a name="size-limits-on-js-interop-calls"></a>Limiti delle dimensioni per le chiamate di interoperabilità JS

In Blazor WebAssembly il Framework non impone un limite per le dimensioni degli input e degli output di interoperabilità js.

In Blazor Server le chiamate di interoperabilità JS hanno dimensioni limitate per la dimensione massima dei messaggi in ingresso SignalR consentita per i metodi dell'hub, che viene applicata da <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> (valore predefinito: 32 KB). I messaggi da JS a .NET SignalR maggiori di <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> generano un errore. Il Framework non impone un limite per le dimensioni di un SignalR messaggio dall'hub a un client.

Quando SignalR la registrazione non è impostata su [debug](xref:Microsoft.Extensions.Logging.LogLevel) o [Trace](xref:Microsoft.Extensions.Logging.LogLevel), un errore relativo alle dimensioni del messaggio viene visualizzato solo nella console degli strumenti di sviluppo del browser:

> Errore: connessione disconnessa con errore "errore: il server ha restituito un errore durante la chiusura: la connessione è stata chiusa con un errore."

Quando la [ SignalR registrazione lato server](xref:signalr/diagnostics#server-side-logging) è impostata su [debug](xref:Microsoft.Extensions.Logging.LogLevel) o [Trace](xref:Microsoft.Extensions.Logging.LogLevel), la registrazione lato server presenta un <xref:System.IO.InvalidDataException> errore di dimensione del messaggio.

`appsettings.Development.json`:

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

> System. IO. InvalidDataException: è stata superata la dimensione massima del messaggio di 32768B. La dimensione del messaggio può essere configurata in AddHubOptions.

Aumentare il limite impostando <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> in `Startup.ConfigureServices` . Nell'esempio seguente viene impostata la dimensione massima del messaggio di ricezione su 64 KB (64 * 1024):

```csharp
services.AddServerSideBlazor()
   .AddHubOptions(options => options.MaximumReceiveMessageSize = 64 * 1024);
```

L'aumento del SignalR limite delle dimensioni dei messaggi in ingresso comporta il costo della richiesta di più risorse del server ed espone il server a maggiori rischi da parte di utenti malintenzionati. Inoltre, la lettura di una grande quantità di contenuto in memoria come stringhe o matrici di byte può comportare anche l'utilizzo di allocazioni che funzionano in modo non corretto con la Garbage Collector, con conseguente penalizzazione delle prestazioni.

Un'opzione per la lettura di payload di grandi dimensioni consiste nell'inviare il contenuto in blocchi più piccoli ed elaborare il payload come <xref:System.IO.Stream> . Può essere usato durante la lettura di payload JSON di grandi dimensioni o se i dati sono disponibili in JavaScript come byte non elaborati. Per un esempio che illustra l'invio di payload binari di grandi dimensioni in Blazor Server che usa tecniche simili al `InputFile` componente, vedere l' [app di esempio binario Submit](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/BinarySubmit).

Quando si sviluppa codice che trasferisce una grande quantità di dati tra JavaScript e, tenere presenti le linee guida seguenti Blazor :

* Sezionare i dati in parti più piccole e inviare i segmenti di dati in sequenza fino a quando non vengono ricevuti tutti i dati dal server.
* Non allocare oggetti di grandi dimensioni in codice JavaScript e C#.
* Non bloccare il thread principale dell'interfaccia utente per periodi prolungati durante l'invio o la ricezione di dati.
* Liberare la memoria utilizzata quando il processo viene completato o annullato.
* Per motivi di sicurezza, applicare i seguenti requisiti aggiuntivi:
  * Dichiarare le dimensioni massime di file o dati che è possibile passare.
  * Dichiarare la velocità di caricamento minima dal client al server.
* Dopo la ricezione dei dati da parte del server, i dati possono essere:
  * Archiviati temporaneamente in un buffer di memoria fino a quando non vengono raccolti tutti i segmenti.
  * Utilizzato immediatamente. Ad esempio, i dati possono essere archiviati immediatamente in un database o scritti su disco durante la ricezione di ogni segmento.

## <a name="js-modules"></a>Moduli JS

Per l'isolamento JS, l'interoperabilità JS funziona con il supporto predefinito del browser per i [moduli ECMAScript (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([specifica ECMAScript](https://tc39.es/ecma262/#sec-modules)).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/call-javascript-from-dotnet>
* [ `InteropComponent.razor` esempio ( `main` Branch del repository GitHub DotNet/AspNetCore)](https://github.com/dotnet/AspNetCore/blob/main/src/Components/test/testassets/BasicTestApp/InteropComponent.razor): il `main` ramo rappresenta lo sviluppo corrente dell'unità di prodotto per la prossima versione di ASP.NET Core. Per selezionare il ramo per una versione diversa (ad esempio, `release/5.0` ), usare l'elenco a discesa **Cambia rami o tag** per selezionare il ramo.
