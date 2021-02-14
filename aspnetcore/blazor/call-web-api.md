---
title: Chiamare un'API Web da ASP.NET Core Blazor WebAssembly
author: guardrex
description: Informazioni su come chiamare un'API Web da un' Blazor WebAssembly app usando Helper JSON, inclusa la creazione di richieste di condivisione di risorse tra le origini (CORS).
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/24/2020
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
uid: blazor/call-web-api
ms.openlocfilehash: b3c783623252512621a0cee7a3607c69cb6d09bb
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280278"
---
# <a name="call-a-web-api-from-aspnet-core-blazor"></a><span data-ttu-id="276be-103">Chiamare un'API Web da ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="276be-103">Call a web API from ASP.NET Core Blazor</span></span>

> [!NOTE]
> <span data-ttu-id="276be-104">Questo argomento si applica a Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="276be-104">This topic applies to Blazor WebAssembly.</span></span> <span data-ttu-id="276be-105">[Blazor Server](xref:blazor/hosting-models#blazor-server) le app chiamano API Web usando le <xref:System.Net.Http.HttpClient> istanze, in genere create usando <xref:System.Net.Http.IHttpClientFactory> .</span><span class="sxs-lookup"><span data-stu-id="276be-105">[Blazor Server](xref:blazor/hosting-models#blazor-server) apps call web APIs using <xref:System.Net.Http.HttpClient> instances, typically created using <xref:System.Net.Http.IHttpClientFactory>.</span></span> <span data-ttu-id="276be-106">Per istruzioni applicabili a Blazor Server , vedere <xref:fundamentals/http-requests> .</span><span class="sxs-lookup"><span data-stu-id="276be-106">For guidance that applies to Blazor Server, see <xref:fundamentals/http-requests>.</span></span>

<span data-ttu-id="276be-107">[Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly) le app chiamano API Web usando un servizio preconfigurato <xref:System.Net.Http.HttpClient> .</span><span class="sxs-lookup"><span data-stu-id="276be-107">[Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly) apps call web APIs using a preconfigured <xref:System.Net.Http.HttpClient> service.</span></span> <span data-ttu-id="276be-108">Comporre richieste, che possono includere opzioni [API di recupero](https://developer.mozilla.org/docs/Web/API/Fetch_API) JavaScript, usando Blazor Helper JSON o con <xref:System.Net.Http.HttpRequestMessage> .</span><span class="sxs-lookup"><span data-stu-id="276be-108">Compose requests, which can include JavaScript [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) options, using Blazor JSON helpers or with <xref:System.Net.Http.HttpRequestMessage>.</span></span> <span data-ttu-id="276be-109">Il <xref:System.Net.Http.HttpClient> servizio nelle Blazor WebAssembly app è incentrato sulla restituzione delle richieste al server di origine.</span><span class="sxs-lookup"><span data-stu-id="276be-109">The <xref:System.Net.Http.HttpClient> service in Blazor WebAssembly apps is focused on making requests back to the server of origin.</span></span> <span data-ttu-id="276be-110">Le indicazioni fornite in questo argomento riguardano solo le Blazor WebAssembly app.</span><span class="sxs-lookup"><span data-stu-id="276be-110">The guidance in this topic only pertains to Blazor WebAssembly apps.</span></span>

<span data-ttu-id="276be-111">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procedura per il download](xref:index#how-to-download-a-sample)): selezionare l' `BlazorWebAssemblySample` app.</span><span class="sxs-lookup"><span data-stu-id="276be-111">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([how to download](xref:index#how-to-download-a-sample)): Select the `BlazorWebAssemblySample` app.</span></span>

<span data-ttu-id="276be-112">Vedere i componenti seguenti nell' `BlazorWebAssemblySample` app di esempio:</span><span class="sxs-lookup"><span data-stu-id="276be-112">See the following components in the `BlazorWebAssemblySample` sample app:</span></span>

* <span data-ttu-id="276be-113">Chiama API Web ( `Pages/CallWebAPI.razor` )</span><span class="sxs-lookup"><span data-stu-id="276be-113">Call Web API (`Pages/CallWebAPI.razor`)</span></span>
* <span data-ttu-id="276be-114">Tester richieste HTTP ( `Components/HTTPRequestTester.razor` )</span><span class="sxs-lookup"><span data-stu-id="276be-114">HTTP Request Tester (`Components/HTTPRequestTester.razor`)</span></span>

## <a name="packages"></a><span data-ttu-id="276be-115">Pacchetti</span><span class="sxs-lookup"><span data-stu-id="276be-115">Packages</span></span>

<span data-ttu-id="276be-116">Fare riferimento al [`System.Net.Http.Json`](https://www.nuget.org/packages/System.Net.Http.Json) pacchetto NuGet nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="276be-116">Reference the [`System.Net.Http.Json`](https://www.nuget.org/packages/System.Net.Http.Json) NuGet package in the project file.</span></span>

## <a name="add-the-httpclient-service"></a><span data-ttu-id="276be-117">Aggiungere il servizio HttpClient</span><span class="sxs-lookup"><span data-stu-id="276be-117">Add the HttpClient service</span></span>

<span data-ttu-id="276be-118">In `Program.Main` aggiungere un <xref:System.Net.Http.HttpClient> servizio se non esiste già:</span><span class="sxs-lookup"><span data-stu-id="276be-118">In `Program.Main`, add an <xref:System.Net.Http.HttpClient> service if it doesn't already exist:</span></span>

```csharp
builder.Services.AddScoped(sp => 
    new HttpClient
    {
        BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
    });
```

## <a name="httpclient-and-json-helpers"></a><span data-ttu-id="276be-119">Helper HttpClient e JSON</span><span class="sxs-lookup"><span data-stu-id="276be-119">HttpClient and JSON helpers</span></span>

<span data-ttu-id="276be-120">In un' Blazor WebAssembly app [`HttpClient`](xref:fundamentals/http-requests) è disponibile come servizio preconfigurato per l'esecuzione di richieste al server di origine.</span><span class="sxs-lookup"><span data-stu-id="276be-120">In a Blazor WebAssembly app, [`HttpClient`](xref:fundamentals/http-requests) is available as a preconfigured service for making requests back to the origin server.</span></span>

<span data-ttu-id="276be-121">Blazor ServerPer impostazione predefinita, un'app non include un <xref:System.Net.Http.HttpClient> servizio.</span><span class="sxs-lookup"><span data-stu-id="276be-121">A Blazor Server app doesn't include an <xref:System.Net.Http.HttpClient> service by default.</span></span> <span data-ttu-id="276be-122">Fornire un <xref:System.Net.Http.HttpClient> all'app usando l' [ `HttpClient` infrastruttura Factory](xref:fundamentals/http-requests).</span><span class="sxs-lookup"><span data-stu-id="276be-122">Provide an <xref:System.Net.Http.HttpClient> to the app using the [`HttpClient` factory infrastructure](xref:fundamentals/http-requests).</span></span>

<span data-ttu-id="276be-123"><xref:System.Net.Http.HttpClient> gli helper JSON e vengono usati anche per chiamare endpoint dell'API Web di terze parti.</span><span class="sxs-lookup"><span data-stu-id="276be-123"><xref:System.Net.Http.HttpClient> and JSON helpers are also used to call third-party web API endpoints.</span></span> <span data-ttu-id="276be-124"><xref:System.Net.Http.HttpClient> viene implementato usando l' [API fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API) del browser ed è soggetto alle limitazioni, inclusa l'applicazione degli stessi criteri di origine.</span><span class="sxs-lookup"><span data-stu-id="276be-124"><xref:System.Net.Http.HttpClient> is implemented using the browser [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) and is subject to its limitations, including enforcement of the same origin policy.</span></span>

<span data-ttu-id="276be-125">L'indirizzo di base del client viene impostato sull'indirizzo del server di origine.</span><span class="sxs-lookup"><span data-stu-id="276be-125">The client's base address is set to the originating server's address.</span></span> <span data-ttu-id="276be-126">Inserire un' <xref:System.Net.Http.HttpClient> istanza usando la [`@inject`](xref:mvc/views/razor#inject) direttiva:</span><span class="sxs-lookup"><span data-stu-id="276be-126">Inject an <xref:System.Net.Http.HttpClient> instance using the [`@inject`](xref:mvc/views/razor#inject) directive:</span></span>

```razor
@using System.Net.Http
@inject HttpClient Http
```

<span data-ttu-id="276be-127">Negli esempi seguenti, un'API Web todo elabora le operazioni di creazione, lettura, aggiornamento ed eliminazione (CRUD).</span><span class="sxs-lookup"><span data-stu-id="276be-127">In the following examples, a Todo web API processes create, read, update, and delete (CRUD) operations.</span></span> <span data-ttu-id="276be-128">Gli esempi sono basati su una `TodoItem` classe che archivia:</span><span class="sxs-lookup"><span data-stu-id="276be-128">The examples are based on a `TodoItem` class that stores the:</span></span>

* <span data-ttu-id="276be-129">ID ( `Id` , `long` ): ID univoco dell'elemento.</span><span class="sxs-lookup"><span data-stu-id="276be-129">ID (`Id`, `long`): Unique ID of the item.</span></span>
* <span data-ttu-id="276be-130">Nome ( `Name` , `string` ): nome dell'elemento.</span><span class="sxs-lookup"><span data-stu-id="276be-130">Name (`Name`, `string`): Name of the item.</span></span>
* <span data-ttu-id="276be-131">Status ( `IsComplete` , `bool` ): indica se l'elemento todo è terminato.</span><span class="sxs-lookup"><span data-stu-id="276be-131">Status (`IsComplete`, `bool`): Indication if the Todo item is finished.</span></span>

```csharp
private class TodoItem
{
    public long Id { get; set; }
    public string Name { get; set; }
    public bool IsComplete { get; set; }
}
```

<span data-ttu-id="276be-132">I metodi helper JSON inviano richieste a un URI (un'API Web negli esempi seguenti) ed elaborano la risposta:</span><span class="sxs-lookup"><span data-stu-id="276be-132">JSON helper methods send requests to a URI (a web API in the following examples) and process the response:</span></span>

* <span data-ttu-id="276be-133"><xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A>: Invia una richiesta HTTP GET e analizza il corpo della risposta JSON per creare un oggetto.</span><span class="sxs-lookup"><span data-stu-id="276be-133"><xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A>: Sends an HTTP GET request and parses the JSON response body to create an object.</span></span>

  <span data-ttu-id="276be-134">Nel codice seguente, l'oggetto `todoItems` viene visualizzato dal componente.</span><span class="sxs-lookup"><span data-stu-id="276be-134">In the following code, the `todoItems` are displayed by the component.</span></span> <span data-ttu-id="276be-135">Il `GetTodoItems` metodo viene attivato quando il componente termina il rendering ( [`OnInitializedAsync`](xref:blazor/components/lifecycle#component-initialization-methods) ).</span><span class="sxs-lookup"><span data-stu-id="276be-135">The `GetTodoItems` method is triggered when the component is finished rendering ([`OnInitializedAsync`](xref:blazor/components/lifecycle#component-initialization-methods)).</span></span> <span data-ttu-id="276be-136">Per un esempio completo, vedere l'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="276be-136">See the sample app for a complete example.</span></span>

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  @code {
      private TodoItem[] todoItems;

      protected override async Task OnInitializedAsync() => 
          todoItems = await Http.GetFromJsonAsync<TodoItem[]>("api/TodoItems");
  }
  ```

* <span data-ttu-id="276be-137"><xref:System.Net.Http.Json.HttpClientJsonExtensions.PostAsJsonAsync%2A>: Invia una richiesta HTTP POST, incluso il contenuto con codifica JSON, e analizza il corpo della risposta JSON per creare un oggetto.</span><span class="sxs-lookup"><span data-stu-id="276be-137"><xref:System.Net.Http.Json.HttpClientJsonExtensions.PostAsJsonAsync%2A>: Sends an HTTP POST request, including JSON-encoded content, and parses the JSON response body to create an object.</span></span>

  <span data-ttu-id="276be-138">Nel codice seguente, `newItemName` viene fornito da un elemento associato del componente.</span><span class="sxs-lookup"><span data-stu-id="276be-138">In the following code, `newItemName` is provided by a bound element of the component.</span></span> <span data-ttu-id="276be-139">Il `AddItem` metodo viene attivato selezionando un `<button>` elemento.</span><span class="sxs-lookup"><span data-stu-id="276be-139">The `AddItem` method is triggered by selecting a `<button>` element.</span></span> <span data-ttu-id="276be-140">Per un esempio completo, vedere l'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="276be-140">See the sample app for a complete example.</span></span>

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  <input @bind="newItemName" placeholder="New Todo Item" />
  <button @onclick="@AddItem">Add</button>

  @code {
      private string newItemName;

      private async Task AddItem()
      {
          var addItem = new TodoItem { Name = newItemName, IsComplete = false };
          await Http.PostAsJsonAsync("api/TodoItems", addItem);
      }
  }
  ```
  
  <span data-ttu-id="276be-141">Chiama per <xref:System.Net.Http.Json.HttpClientJsonExtensions.PostAsJsonAsync%2A> restituire un oggetto <xref:System.Net.Http.HttpResponseMessage> .</span><span class="sxs-lookup"><span data-stu-id="276be-141">Calls to <xref:System.Net.Http.Json.HttpClientJsonExtensions.PostAsJsonAsync%2A> return an <xref:System.Net.Http.HttpResponseMessage>.</span></span> <span data-ttu-id="276be-142">Per deserializzare il contenuto JSON dal messaggio di risposta, usare il `ReadFromJsonAsync<T>` metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="276be-142">To deserialize the JSON content from the response message, use the `ReadFromJsonAsync<T>` extension method:</span></span>
  
  ```csharp
  var content = response.Content.ReadFromJsonAsync<WeatherForecast>();
  ```

* <span data-ttu-id="276be-143"><xref:System.Net.Http.Json.HttpClientJsonExtensions.PutAsJsonAsync%2A>: Invia una richiesta HTTP PUT, incluso il contenuto con codifica JSON.</span><span class="sxs-lookup"><span data-stu-id="276be-143"><xref:System.Net.Http.Json.HttpClientJsonExtensions.PutAsJsonAsync%2A>: Sends an HTTP PUT request, including JSON-encoded content.</span></span>

  <span data-ttu-id="276be-144">Nel codice seguente, `editItem` i valori per `Name` e `IsCompleted` vengono forniti dagli elementi associati del componente.</span><span class="sxs-lookup"><span data-stu-id="276be-144">In the following code, `editItem` values for `Name` and `IsCompleted` are provided by bound elements of the component.</span></span> <span data-ttu-id="276be-145">L'elemento `Id` viene impostato quando l'elemento viene selezionato in un'altra parte dell'interfaccia utente e `EditItem` viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="276be-145">The item's `Id` is set when the item is selected in another part of the UI and `EditItem` is called.</span></span> <span data-ttu-id="276be-146">Il `SaveItem` metodo viene attivato selezionando l'elemento Save `<button>` .</span><span class="sxs-lookup"><span data-stu-id="276be-146">The `SaveItem` method is triggered by selecting the Save `<button>` element.</span></span> <span data-ttu-id="276be-147">Per un esempio completo, vedere l'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="276be-147">See the sample app for a complete example.</span></span>

  ```razor
  @using System.Net.Http
  @inject HttpClient Http

  <input type="checkbox" @bind="editItem.IsComplete" />
  <input @bind="editItem.Name" />
  <button @onclick="@SaveItem">Save</button>

  @code {
      private TodoItem editItem = new TodoItem();

      private void EditItem(long id)
      {
          editItem = todoItems.Single(i => i.Id == id);
      }

      private async Task SaveItem() =>
          await Http.PutAsJsonAsync($"api/TodoItems/{editItem.Id}, editItem);
  }
  ```
  
  <span data-ttu-id="276be-148">Chiama per <xref:System.Net.Http.Json.HttpClientJsonExtensions.PutAsJsonAsync%2A> restituire un oggetto <xref:System.Net.Http.HttpResponseMessage> .</span><span class="sxs-lookup"><span data-stu-id="276be-148">Calls to <xref:System.Net.Http.Json.HttpClientJsonExtensions.PutAsJsonAsync%2A> return an <xref:System.Net.Http.HttpResponseMessage>.</span></span> <span data-ttu-id="276be-149">Per deserializzare il contenuto JSON dal messaggio di risposta, usare il <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="276be-149">To deserialize the JSON content from the response message, use the <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> extension method:</span></span>
  
  ```csharp
  var content = response.Content.ReadFromJsonAsync<WeatherForecast>();
  ```

<span data-ttu-id="276be-150"><xref:System.Net.Http> include metodi di estensione aggiuntivi per l'invio di richieste HTTP e la ricezione di risposte HTTP.</span><span class="sxs-lookup"><span data-stu-id="276be-150"><xref:System.Net.Http> includes additional extension methods for sending HTTP requests and receiving HTTP responses.</span></span> <span data-ttu-id="276be-151"><xref:System.Net.Http.HttpClient.DeleteAsync%2A?displayProperty=nameWithType> viene usato per inviare una richiesta HTTP DELETE a un'API Web.</span><span class="sxs-lookup"><span data-stu-id="276be-151"><xref:System.Net.Http.HttpClient.DeleteAsync%2A?displayProperty=nameWithType> is used to send an HTTP DELETE request to a web API.</span></span>

<span data-ttu-id="276be-152">Nel codice seguente, l'elemento Delete `<button>` chiama il `DeleteItem` metodo.</span><span class="sxs-lookup"><span data-stu-id="276be-152">In the following code, the Delete `<button>` element calls the `DeleteItem` method.</span></span> <span data-ttu-id="276be-153">L' `<input>` elemento associato fornisce l' `id` oggetto dell'elemento da eliminare.</span><span class="sxs-lookup"><span data-stu-id="276be-153">The bound `<input>` element supplies the `id` of the item to delete.</span></span> <span data-ttu-id="276be-154">Per un esempio completo, vedere l'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="276be-154">See the sample app for a complete example.</span></span>

```razor
@using System.Net.Http
@inject HttpClient Http

<input @bind="id" />
<button @onclick="@DeleteItem">Delete</button>

@code {
    private long id;

    private async Task DeleteItem() =>
        await Http.DeleteAsync($"api/TodoItems/{id}");
}
```

## <a name="named-httpclient-with-ihttpclientfactory"></a><span data-ttu-id="276be-155">Denominato HttpClient con IHttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="276be-155">Named HttpClient with IHttpClientFactory</span></span>

<span data-ttu-id="276be-156"><xref:System.Net.Http.IHttpClientFactory> i servizi e la configurazione di un oggetto denominato <xref:System.Net.Http.HttpClient> sono supportati.</span><span class="sxs-lookup"><span data-stu-id="276be-156"><xref:System.Net.Http.IHttpClientFactory> services and the configuration of a named <xref:System.Net.Http.HttpClient> are supported.</span></span>

<span data-ttu-id="276be-157">Fare riferimento al [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) pacchetto NuGet nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="276be-157">Reference the [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) NuGet package in the project file.</span></span>

<span data-ttu-id="276be-158">`Program.Main` (`Program.cs`):</span><span class="sxs-lookup"><span data-stu-id="276be-158">`Program.Main` (`Program.cs`):</span></span>

```csharp
builder.Services.AddHttpClient("ServerAPI", client => 
    client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress));
```

<span data-ttu-id="276be-159">`FetchData` componente ( `Pages/FetchData.razor` ):</span><span class="sxs-lookup"><span data-stu-id="276be-159">`FetchData` component (`Pages/FetchData.razor`):</span></span>

```razor
@inject IHttpClientFactory ClientFactory

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        var client = ClientFactory.CreateClient("ServerAPI");

        forecasts = await client.GetFromJsonAsync<WeatherForecast[]>(
            "WeatherForecast");
    }
}
```

## <a name="typed-httpclient"></a><span data-ttu-id="276be-160">HttpClient tipizzato</span><span class="sxs-lookup"><span data-stu-id="276be-160">Typed HttpClient</span></span>

<span data-ttu-id="276be-161">Tipizzato <xref:System.Net.Http.HttpClient> Usa una o più istanze dell'app <xref:System.Net.Http.HttpClient> , predefinite o denominate, per restituire i dati da uno o più endpoint dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="276be-161">Typed <xref:System.Net.Http.HttpClient> uses one or more of the app's <xref:System.Net.Http.HttpClient> instances, default or named, to return data from one or more web API endpoints.</span></span>

<span data-ttu-id="276be-162">`WeatherForecastClient.cs`:</span><span class="sxs-lookup"><span data-stu-id="276be-162">`WeatherForecastClient.cs`:</span></span>

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;

public class WeatherForecastHttpClient
{
    private readonly HttpClient http;

    public WeatherForecastHttpClient(HttpClient http)
    {
        this.http = http;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var forecasts = new WeatherForecast[0];

        try
        {
            forecasts = await http.GetFromJsonAsync<WeatherForecast[]>(
                "WeatherForecast");
        }
        catch
        {
            ...
        }

        return forecasts;
    }
}
```

<span data-ttu-id="276be-163">`Program.Main` (`Program.cs`):</span><span class="sxs-lookup"><span data-stu-id="276be-163">`Program.Main` (`Program.cs`):</span></span>

```csharp
builder.Services.AddHttpClient<WeatherForecastHttpClient>(client => 
    client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress));
```

<span data-ttu-id="276be-164">I componenti inseriscono il tipizzato <xref:System.Net.Http.HttpClient> per chiamare l'API Web.</span><span class="sxs-lookup"><span data-stu-id="276be-164">Components inject the typed <xref:System.Net.Http.HttpClient> to call the web API.</span></span>

<span data-ttu-id="276be-165">`FetchData` componente ( `Pages/FetchData.razor` ):</span><span class="sxs-lookup"><span data-stu-id="276be-165">`FetchData` component (`Pages/FetchData.razor`):</span></span>

```razor
@inject WeatherForecastHttpClient Http

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        forecasts = await Http.GetForecastAsync();
    }
}
```

## <a name="httpclient-and-httprequestmessage-with-fetch-api-request-options"></a><span data-ttu-id="276be-166">`HttpClient` e `HttpRequestMessage` con le opzioni di richiesta dell'API fetch</span><span class="sxs-lookup"><span data-stu-id="276be-166">`HttpClient` and `HttpRequestMessage` with Fetch API request options</span></span>

<span data-ttu-id="276be-167">Quando è in esecuzione in un webassembly in un' Blazor WebAssembly app, [`HttpClient`](xref:fundamentals/http-requests) ([documentazione API](xref:System.Net.Http.HttpClient)) e <xref:System.Net.Http.HttpRequestMessage> può essere usato per personalizzare le richieste.</span><span class="sxs-lookup"><span data-stu-id="276be-167">When running on WebAssembly in a Blazor WebAssembly app, [`HttpClient`](xref:fundamentals/http-requests) ([API documentation](xref:System.Net.Http.HttpClient)) and <xref:System.Net.Http.HttpRequestMessage> can be used to customize requests.</span></span> <span data-ttu-id="276be-168">Ad esempio, è possibile specificare il metodo HTTP e le intestazioni di richiesta.</span><span class="sxs-lookup"><span data-stu-id="276be-168">For example, you can specify the HTTP method and request headers.</span></span> <span data-ttu-id="276be-169">Il componente seguente effettua una `POST` richiesta a un endpoint API to do list nel server e Mostra il corpo della risposta:</span><span class="sxs-lookup"><span data-stu-id="276be-169">The following component makes a `POST` request to a To Do List API endpoint on the server and shows the response body:</span></span>

```razor
@page "/todorequest"
@using System.Net.Http
@using System.Net.Http.Headers
@using System.Net.Http.Json
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject HttpClient Http
@inject IAccessTokenProvider TokenProvider

<h1>ToDo Request</h1>

<button @onclick="PostRequest">Submit POST request</button>

<p>Response body returned by the server:</p>

<p>@responseBody</p>

@code {
    private string responseBody;

    private async Task PostRequest()
    {
        var requestMessage = new HttpRequestMessage()
        {
            Method = new HttpMethod("POST"),
            RequestUri = new Uri("https://localhost:10000/api/TodoItems"),
            Content =
                JsonContent.Create(new TodoItem
                {
                    Name = "My New Todo Item",
                    IsComplete = false
                })
        };

        var tokenResult = await TokenProvider.RequestAccessToken();

        if (tokenResult.TryGetToken(out var token))
        {
            requestMessage.Headers.Authorization =
                new AuthenticationHeaderValue("Bearer", token.Value);

            requestMessage.Content.Headers.TryAddWithoutValidation(
                "x-custom-header", "value");

            var response = await Http.SendAsync(requestMessage);
            var responseStatusCode = response.StatusCode;

            responseBody = await response.Content.ReadAsStringAsync();
        }
    }

    public class TodoItem
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
```

<span data-ttu-id="276be-170">L'implementazione di .NET webassembly <xref:System.Net.Http.HttpClient> Usa [WindowOrWorkerGlobalScope. fetch ()](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch).</span><span class="sxs-lookup"><span data-stu-id="276be-170">.NET WebAssembly's implementation of <xref:System.Net.Http.HttpClient> uses [WindowOrWorkerGlobalScope.fetch()](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch).</span></span> <span data-ttu-id="276be-171">Il recupero consente la configurazione [di diverse opzioni specifiche della richiesta](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters).</span><span class="sxs-lookup"><span data-stu-id="276be-171">Fetch allows configuring several [request-specific options](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters).</span></span> 

<span data-ttu-id="276be-172">Le opzioni di richiesta di recupero HTTP possono essere configurate con <xref:System.Net.Http.HttpRequestMessage> i metodi di estensione illustrati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="276be-172">HTTP fetch request options can be configured with <xref:System.Net.Http.HttpRequestMessage> extension methods shown in the following table.</span></span>

| <span data-ttu-id="276be-173">Metodo di estensione</span><span class="sxs-lookup"><span data-stu-id="276be-173">Extension method</span></span> | <span data-ttu-id="276be-174">Recupera la proprietà della richiesta</span><span class="sxs-lookup"><span data-stu-id="276be-174">Fetch request property</span></span> |
| --- | --- |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> | [`credentials`](https://developer.mozilla.org/docs/Web/API/Request/credentials) |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCache%2A> | [`cache`](https://developer.mozilla.org/docs/Web/API/Request/cache) |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestMode%2A> | [`mode`](https://developer.mozilla.org/docs/Web/API/Request/mode) |
| <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestIntegrity%2A> | [`integrity`](https://developer.mozilla.org/docs/Web/API/Request/integrity) |

<span data-ttu-id="276be-175">È possibile impostare opzioni aggiuntive usando il metodo di <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestOption%2A> estensione più generico.</span><span class="sxs-lookup"><span data-stu-id="276be-175">You can set additional options using the more generic <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestOption%2A> extension method.</span></span>
 
<span data-ttu-id="276be-176">La risposta HTTP viene in genere memorizzata nel buffer in un' Blazor WebAssembly applicazione per abilitare il supporto per le letture della sincronizzazione sul contenuto della risposta.</span><span class="sxs-lookup"><span data-stu-id="276be-176">The HTTP response is typically buffered in a Blazor WebAssembly app to enable support for sync reads on the response content.</span></span> <span data-ttu-id="276be-177">Per abilitare il supporto per il flusso di risposta, usare il <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserResponseStreamingEnabled%2A> metodo di estensione nella richiesta.</span><span class="sxs-lookup"><span data-stu-id="276be-177">To enable support for response streaming, use the <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserResponseStreamingEnabled%2A> extension method on the request.</span></span>

<span data-ttu-id="276be-178">Per includere le credenziali in una richiesta tra più origini, usare il <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="276be-178">To include credentials in a cross-origin request, use the <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> extension method:</span></span>

```csharp
requestMessage.SetBrowserRequestCredentials(BrowserRequestCredentials.Include);
```

<span data-ttu-id="276be-179">Per altre informazioni sulle opzioni di recupero delle API, vedere la pagina relativa alla [documentazione Web MDN: WindowOrWorkerGlobalScope. fetch ():P arameters](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters).</span><span class="sxs-lookup"><span data-stu-id="276be-179">For more information on Fetch API options, see [MDN web docs: WindowOrWorkerGlobalScope.fetch():Parameters](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters).</span></span>

## <a name="handle-errors"></a><span data-ttu-id="276be-180">Gestire gli errori</span><span class="sxs-lookup"><span data-stu-id="276be-180">Handle errors</span></span>

<span data-ttu-id="276be-181">Quando si verificano errori durante l'interazione con un'API Web, possono essere gestiti dal codice dello sviluppatore.</span><span class="sxs-lookup"><span data-stu-id="276be-181">When errors occur while interacting with a web API, they can be handled by developer code.</span></span> <span data-ttu-id="276be-182"><xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A>Si prevede ad esempio una risposta JSON dall'API del server con un valore `Content-Type` di `application/json` .</span><span class="sxs-lookup"><span data-stu-id="276be-182">For example, <xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A> expects a JSON response from the server API with a `Content-Type` of `application/json`.</span></span> <span data-ttu-id="276be-183">Se la risposta non è in formato JSON, la convalida del contenuto genera un'eccezione <xref:System.NotSupportedException> .</span><span class="sxs-lookup"><span data-stu-id="276be-183">If the response isn't in JSON format, content validation throws a <xref:System.NotSupportedException>.</span></span>

<span data-ttu-id="276be-184">Nell'esempio seguente l'endpoint URI per la richiesta di dati meteorologici non è stato digitato correttamente.</span><span class="sxs-lookup"><span data-stu-id="276be-184">In the following example, the URI endpoint for the weather forecast data request is misspelled.</span></span> <span data-ttu-id="276be-185">L'URI deve essere, `WeatherForecast` ma viene visualizzato nella chiamata a `WeatherForcast` (mancante "e").</span><span class="sxs-lookup"><span data-stu-id="276be-185">The URI should be to `WeatherForecast` but appears in the call as `WeatherForcast` (missing "e").</span></span>

<span data-ttu-id="276be-186">La <xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A> chiamata prevede che venga restituito JSON, ma il server restituisce il codice HTML per un'eccezione non gestita nel server con un valore `Content-Type` di `text/html` .</span><span class="sxs-lookup"><span data-stu-id="276be-186">The <xref:System.Net.Http.Json.HttpClientJsonExtensions.GetFromJsonAsync%2A> call expects JSON to be returned, but the server returns HTML for an unhandled exception on the server with a `Content-Type` of `text/html`.</span></span> <span data-ttu-id="276be-187">L'eccezione non gestita si verifica nel server perché il percorso non è stato trovato e il middleware non può gestire una pagina o una vista per la richiesta.</span><span class="sxs-lookup"><span data-stu-id="276be-187">The unhandled exception occurs on the server because the path isn't found and middleware can't serve a page or view for the request.</span></span>

<span data-ttu-id="276be-188">In <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> sul client <xref:System.NotSupportedException> viene generata un'eccezione quando il contenuto della risposta viene convalidato come non JSON.</span><span class="sxs-lookup"><span data-stu-id="276be-188">In <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> on the client, <xref:System.NotSupportedException> is thrown when the response content is validated as non-JSON.</span></span> <span data-ttu-id="276be-189">L'eccezione viene rilevata nel `catch` blocco, in cui la logica personalizzata può registrare l'errore o presentare un messaggio di errore descrittivo all'utente:</span><span class="sxs-lookup"><span data-stu-id="276be-189">The exception is caught in the `catch` block, where custom logic could log the error or present a friendly error message to the user:</span></span>

```csharp
protected override async Task OnInitializedAsync()
{
    try
    {
        forecasts = await Http.GetFromJsonAsync<WeatherForecast[]>(
            "WeatherForcast");
    }
    catch (NotSupportedException exception)
    {
        ...
    }
}
```

> [!NOTE]
> <span data-ttu-id="276be-190">L'esempio precedente è a scopo dimostrativo.</span><span class="sxs-lookup"><span data-stu-id="276be-190">The preceding example is for demonstration purposes.</span></span> <span data-ttu-id="276be-191">Un'app Server API Web può essere configurata in modo da restituire JSON anche quando non esiste un endpoint o si verifica un'eccezione non gestita nel server.</span><span class="sxs-lookup"><span data-stu-id="276be-191">A web API server app can be configured to return JSON even when an endpoint doesn't exist or an unhandled exception on the server occurs.</span></span>

<span data-ttu-id="276be-192">Per altre informazioni, vedere <xref:blazor/fundamentals/handle-errors>.</span><span class="sxs-lookup"><span data-stu-id="276be-192">For more information, see <xref:blazor/fundamentals/handle-errors>.</span></span>

## <a name="cross-origin-resource-sharing-cors"></a><span data-ttu-id="276be-193">Condivisione di risorse tra le origini (CORS)</span><span class="sxs-lookup"><span data-stu-id="276be-193">Cross-origin resource sharing (CORS)</span></span>

<span data-ttu-id="276be-194">La sicurezza del browser impedisce a una pagina Web di effettuare richieste a un dominio diverso da quello che ha servito la pagina Web.</span><span class="sxs-lookup"><span data-stu-id="276be-194">Browser security prevents a webpage from making requests to a different domain than the one that served the webpage.</span></span> <span data-ttu-id="276be-195">Questa restrizione è detta *criterio della stessa origine*.</span><span class="sxs-lookup"><span data-stu-id="276be-195">This restriction is called the *same-origin policy*.</span></span> <span data-ttu-id="276be-196">Il criterio della stessa origine impedisce a un sito dannoso di leggere dati sensibili da un altro sito.</span><span class="sxs-lookup"><span data-stu-id="276be-196">The same-origin policy prevents a malicious site from reading sensitive data from another site.</span></span> <span data-ttu-id="276be-197">Per eseguire richieste dal browser a un endpoint con un'origine diversa, l' *endpoint* deve abilitare la [condivisione di risorse tra le origini (CORS)](https://www.w3.org/TR/cors/).</span><span class="sxs-lookup"><span data-stu-id="276be-197">To make requests from the browser to an endpoint with a different origin, the *endpoint* must enable [cross-origin resource sharing (CORS)](https://www.w3.org/TR/cors/).</span></span>

<span data-ttu-id="276be-198">L' [ Blazor WebAssembly app di esempio ( Blazor WebAssemblySample)](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) illustra l'uso di CORS nel componente chiamata API Web ( `Pages/CallWebAPI.razor` ).</span><span class="sxs-lookup"><span data-stu-id="276be-198">The [Blazor WebAssembly sample app (BlazorWebAssemblySample)](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) demonstrates the use of CORS in the Call Web API component (`Pages/CallWebAPI.razor`).</span></span>

<span data-ttu-id="276be-199">Per ulteriori informazioni su CORS con richieste sicure nelle Blazor app, vedere <xref:blazor/security/webassembly/additional-scenarios#cross-origin-resource-sharing-cors> .</span><span class="sxs-lookup"><span data-stu-id="276be-199">For more information on CORS with secure requests in Blazor apps, see <xref:blazor/security/webassembly/additional-scenarios#cross-origin-resource-sharing-cors>.</span></span>

<span data-ttu-id="276be-200">Per informazioni generali su CORS con app ASP.NET Core, vedere <xref:security/cors> .</span><span class="sxs-lookup"><span data-stu-id="276be-200">For general information on CORS with ASP.NET Core apps, see <xref:security/cors>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="276be-201">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="276be-201">Additional resources</span></span>

* <span data-ttu-id="276be-202"><xref:blazor/security/webassembly/additional-scenarios>: Include la copertura sull'uso <xref:System.Net.Http.HttpClient> di per creare richieste API Web sicure.</span><span class="sxs-lookup"><span data-stu-id="276be-202"><xref:blazor/security/webassembly/additional-scenarios>: Includes coverage on using <xref:System.Net.Http.HttpClient> to make secure web API requests.</span></span>
* <xref:fundamentals/http-requests>
* <xref:security/enforcing-ssl>
::: moniker range=">= aspnetcore-5.0"
* [<span data-ttu-id="276be-203">Configurazione dell'endpoint HTTPS di gheppio</span><span class="sxs-lookup"><span data-stu-id="276be-203">Kestrel HTTPS endpoint configuration</span></span>](xref:fundamentals/servers/kestrel/endpoints)
::: moniker-end
::: moniker range="< aspnetcore-5.0"
* [<span data-ttu-id="276be-204">Configurazione dell'endpoint HTTPS di gheppio</span><span class="sxs-lookup"><span data-stu-id="276be-204">Kestrel HTTPS endpoint configuration</span></span>](xref:fundamentals/servers/kestrel#endpoint-configuration)
::: moniker-end
* [<span data-ttu-id="276be-205">Condivisione di risorse tra le origini (CORS) in W3C</span><span class="sxs-lookup"><span data-stu-id="276be-205">Cross Origin Resource Sharing (CORS) at W3C</span></span>](https://www.w3.org/TR/cors/)
