---
title: Usare gli hub in ASP.NET Core SignalR
author: bradygaster
description: Informazioni su come usare gli hub in ASP.NET Core SignalR .
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 01/16/2020
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
uid: signalr/hubs
ms.openlocfilehash: 872b88cc3c87137365de8c50a37bf5dd5fd9fe10
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587866"
---
# <a name="use-hubs-in-signalr-for-aspnet-core"></a><span data-ttu-id="97fc2-103">Usare gli hub in SignalR per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="97fc2-103">Use hubs in SignalR for ASP.NET Core</span></span>

<span data-ttu-id="97fc2-104">Di [Rachel Appel](https://twitter.com/rachelappel) e [Kevin Griffin](https://twitter.com/1kevgriff)</span><span class="sxs-lookup"><span data-stu-id="97fc2-104">By [Rachel Appel](https://twitter.com/rachelappel) and [Kevin Griffin](https://twitter.com/1kevgriff)</span></span>

<span data-ttu-id="97fc2-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/hubs/sample/ ) [(procedura per il download)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="97fc2-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/hubs/sample/ ) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

## <a name="what-is-a-signalr-hub"></a><span data-ttu-id="97fc2-106">Che cos'è un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="97fc2-106">What is a SignalR hub</span></span>

<span data-ttu-id="97fc2-107">L' SignalR API Hub consente di chiamare i metodi sui client connessi dal server.</span><span class="sxs-lookup"><span data-stu-id="97fc2-107">The SignalR Hubs API enables you to call methods on connected clients from the server.</span></span> <span data-ttu-id="97fc2-108">Nel codice del server si definiscono i metodi che vengono chiamati dal client.</span><span class="sxs-lookup"><span data-stu-id="97fc2-108">In the server code, you define methods that are called by client.</span></span> <span data-ttu-id="97fc2-109">Nel codice client è possibile definire metodi che vengono chiamati dal server.</span><span class="sxs-lookup"><span data-stu-id="97fc2-109">In the client code, you define methods that are called from the server.</span></span> <span data-ttu-id="97fc2-110">SignalR gestisce tutti gli elementi dietro le quinte che rendono possibili le comunicazioni da client a server e da server a client in tempo reale.</span><span class="sxs-lookup"><span data-stu-id="97fc2-110">SignalR takes care of everything behind the scenes that makes real-time client-to-server and server-to-client communications possible.</span></span>

## <a name="configure-signalr-hubs"></a><span data-ttu-id="97fc2-111">Configurare gli SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="97fc2-111">Configure SignalR hubs</span></span>

<span data-ttu-id="97fc2-112">Il SignalR middleware richiede alcuni servizi, che vengono configurati chiamando `services.AddSignalR` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-112">The SignalR middleware requires some services, which are configured by calling `services.AddSignalR`.</span></span>

[!code-csharp[Configure service](hubs/sample/startup.cs?range=38)]

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="97fc2-113">Quando si aggiungono SignalR funzionalità a un'app ASP.NET Core, SignalR le route vengono impostate chiamando `endpoint.MapHub` nel callback del `Startup.Configure` metodo `app.UseEndpoints` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-113">When adding SignalR functionality to an ASP.NET Core app, setup SignalR routes by calling `endpoint.MapHub` in the `Startup.Configure` method's `app.UseEndpoints` callback.</span></span>

```csharp
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapHub<ChatHub>("/chathub");
});
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

<span data-ttu-id="97fc2-114">Quando si aggiungono SignalR funzionalità a un'app ASP.NET Core, SignalR le route vengono impostate chiamando `app.UseSignalR` nel `Startup.Configure` metodo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-114">When adding SignalR functionality to an ASP.NET Core app, setup SignalR routes by calling `app.UseSignalR` in the `Startup.Configure` method.</span></span>

[!code-csharp[Configure routes to hubs](hubs/sample/startup.cs?range=57-60)]

::: moniker-end

## <a name="create-and-use-hubs"></a><span data-ttu-id="97fc2-115">Creare e usare gli hub</span><span class="sxs-lookup"><span data-stu-id="97fc2-115">Create and use hubs</span></span>

<span data-ttu-id="97fc2-116">Creare un hub dichiarando una classe che eredita da `Hub` e aggiungervi metodi pubblici.</span><span class="sxs-lookup"><span data-stu-id="97fc2-116">Create a hub by declaring a class that inherits from `Hub`, and add public methods to it.</span></span> <span data-ttu-id="97fc2-117">I client possono chiamare metodi definiti come `public` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-117">Clients can call methods that are defined as `public`.</span></span>

```csharp
public class ChatHub : Hub
{
    public Task SendMessage(string user, string message)
    {
        return Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

<span data-ttu-id="97fc2-118">È possibile specificare un tipo restituito e i parametri, inclusi i tipi e le matrici complessi, come si farebbe con qualsiasi metodo C#.</span><span class="sxs-lookup"><span data-stu-id="97fc2-118">You can specify a return type and parameters, including complex types and arrays, as you would in any C# method.</span></span> <span data-ttu-id="97fc2-119">SignalR gestisce la serializzazione e la deserializzazione di oggetti e matrici complessi nei parametri e valori restituiti.</span><span class="sxs-lookup"><span data-stu-id="97fc2-119">SignalR handles the serialization and deserialization of complex objects and arrays in your parameters and return values.</span></span>

> [!NOTE]
> <span data-ttu-id="97fc2-120">Gli hub sono temporanei:</span><span class="sxs-lookup"><span data-stu-id="97fc2-120">Hubs are transient:</span></span>
>
> * <span data-ttu-id="97fc2-121">Non archiviare lo stato in una proprietà nella classe Hub.</span><span class="sxs-lookup"><span data-stu-id="97fc2-121">Don't store state in a property on the hub class.</span></span> <span data-ttu-id="97fc2-122">Ogni chiamata al metodo dell'hub viene eseguita su una nuova istanza dell'hub.</span><span class="sxs-lookup"><span data-stu-id="97fc2-122">Every hub method call is executed on a new hub instance.</span></span>
> * <span data-ttu-id="97fc2-123">Usare `await` quando si chiamano metodi asincroni che dipendono dall'hub rimanendo attivo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-123">Use `await` when calling asynchronous methods that depend on the hub staying alive.</span></span> <span data-ttu-id="97fc2-124">Ad esempio, un metodo come `Clients.All.SendAsync(...)` può avere esito negativo se viene chiamato senza `await` e il metodo Hub viene completato prima del completamento `SendAsync` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-124">For example, a method such as `Clients.All.SendAsync(...)` can fail if it's called without `await` and the hub method completes before `SendAsync` finishes.</span></span>

## <a name="the-context-object"></a><span data-ttu-id="97fc2-125">Oggetto context</span><span class="sxs-lookup"><span data-stu-id="97fc2-125">The Context object</span></span>

<span data-ttu-id="97fc2-126">La `Hub` classe dispone di una `Context` proprietà che contiene le proprietà seguenti con le informazioni sulla connessione:</span><span class="sxs-lookup"><span data-stu-id="97fc2-126">The `Hub` class has a `Context` property that contains the following properties with information about the connection:</span></span>

| <span data-ttu-id="97fc2-127">Proprietà</span><span class="sxs-lookup"><span data-stu-id="97fc2-127">Property</span></span> | <span data-ttu-id="97fc2-128">Descrizione</span><span class="sxs-lookup"><span data-stu-id="97fc2-128">Description</span></span> |
| ------ | ----------- |
| `ConnectionId` | <span data-ttu-id="97fc2-129">Ottiene l'ID univoco per la connessione, assegnato da SignalR .</span><span class="sxs-lookup"><span data-stu-id="97fc2-129">Gets the unique ID for the connection, assigned by SignalR.</span></span> <span data-ttu-id="97fc2-130">È presente un ID di connessione per ogni connessione.</span><span class="sxs-lookup"><span data-stu-id="97fc2-130">There is one connection ID for each connection.</span></span>|
| `UserIdentifier` | <span data-ttu-id="97fc2-131">Ottiene l' [identificatore utente](xref:signalr/groups).</span><span class="sxs-lookup"><span data-stu-id="97fc2-131">Gets the [user identifier](xref:signalr/groups).</span></span> <span data-ttu-id="97fc2-132">Per impostazione predefinita, SignalR utilizza l'oggetto `ClaimTypes.NameIdentifier` dall'oggetto associato alla `ClaimsPrincipal` connessione come identificatore utente.</span><span class="sxs-lookup"><span data-stu-id="97fc2-132">By default, SignalR uses the `ClaimTypes.NameIdentifier` from the `ClaimsPrincipal` associated with the connection as the user identifier.</span></span> |
| `User` | <span data-ttu-id="97fc2-133">Ottiene l'oggetto `ClaimsPrincipal` associato all'utente corrente.</span><span class="sxs-lookup"><span data-stu-id="97fc2-133">Gets the `ClaimsPrincipal` associated with the current user.</span></span> |
| `Items` | <span data-ttu-id="97fc2-134">Ottiene una raccolta chiave/valore che può essere utilizzata per condividere i dati nell'ambito di questa connessione.</span><span class="sxs-lookup"><span data-stu-id="97fc2-134">Gets a key/value collection that can be used to share data within the scope of this connection.</span></span> <span data-ttu-id="97fc2-135">I dati possono essere archiviati in questa raccolta e verranno mantenuti per la connessione tra diverse chiamate al metodo dell'hub.</span><span class="sxs-lookup"><span data-stu-id="97fc2-135">Data can be stored in this collection and it will persist for the connection across different hub method invocations.</span></span> |
| `Features` | <span data-ttu-id="97fc2-136">Ottiene la raccolta di funzionalità disponibili nella connessione.</span><span class="sxs-lookup"><span data-stu-id="97fc2-136">Gets the collection of features available on the connection.</span></span> <span data-ttu-id="97fc2-137">Per il momento, questa raccolta non è necessaria nella maggior parte degli scenari, quindi non è ancora documentata in dettaglio.</span><span class="sxs-lookup"><span data-stu-id="97fc2-137">For now, this collection isn't needed in most scenarios, so it isn't documented in detail yet.</span></span> |
| `ConnectionAborted` | <span data-ttu-id="97fc2-138">Ottiene un oggetto che invia una `CancellationToken` notifica quando la connessione viene interrotta.</span><span class="sxs-lookup"><span data-stu-id="97fc2-138">Gets a `CancellationToken` that notifies when the connection is aborted.</span></span> |

<span data-ttu-id="97fc2-139">`Hub.Context` contiene inoltre i metodi seguenti:</span><span class="sxs-lookup"><span data-stu-id="97fc2-139">`Hub.Context` also contains the following methods:</span></span>

| <span data-ttu-id="97fc2-140">Metodo</span><span class="sxs-lookup"><span data-stu-id="97fc2-140">Method</span></span> | <span data-ttu-id="97fc2-141">Descrizione</span><span class="sxs-lookup"><span data-stu-id="97fc2-141">Description</span></span> |
| ------ | ----------- |
| `GetHttpContext` | <span data-ttu-id="97fc2-142">Restituisce l'oggetto `HttpContext` per la connessione o `null` se la connessione non è associata a una richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="97fc2-142">Returns the `HttpContext` for the connection, or `null` if the connection is not associated with an HTTP request.</span></span> <span data-ttu-id="97fc2-143">Per le connessioni HTTP, è possibile usare questo metodo per ottenere informazioni come le intestazioni HTTP e le stringhe di query.</span><span class="sxs-lookup"><span data-stu-id="97fc2-143">For HTTP connections, you can use this method to get information such as HTTP headers and query strings.</span></span> |
| `Abort` | <span data-ttu-id="97fc2-144">Interrompe la connessione.</span><span class="sxs-lookup"><span data-stu-id="97fc2-144">Aborts the connection.</span></span> |

## <a name="the-clients-object"></a><span data-ttu-id="97fc2-145">Oggetto clients</span><span class="sxs-lookup"><span data-stu-id="97fc2-145">The Clients object</span></span>

<span data-ttu-id="97fc2-146">La `Hub` classe dispone di una `Clients` proprietà che contiene le proprietà seguenti per la comunicazione tra server e client:</span><span class="sxs-lookup"><span data-stu-id="97fc2-146">The `Hub` class has a `Clients` property that contains the following properties for communication between server and client:</span></span>

| <span data-ttu-id="97fc2-147">Proprietà</span><span class="sxs-lookup"><span data-stu-id="97fc2-147">Property</span></span> | <span data-ttu-id="97fc2-148">Descrizione</span><span class="sxs-lookup"><span data-stu-id="97fc2-148">Description</span></span> |
| ------ | ----------- |
| `All` | <span data-ttu-id="97fc2-149">Chiama un metodo su tutti i client connessi</span><span class="sxs-lookup"><span data-stu-id="97fc2-149">Calls a method on all connected clients</span></span> |
| `Caller` | <span data-ttu-id="97fc2-150">Chiama un metodo sul client che ha richiamato il metodo Hub</span><span class="sxs-lookup"><span data-stu-id="97fc2-150">Calls a method on the client that invoked the hub method</span></span> |
| `Others` | <span data-ttu-id="97fc2-151">Chiama un metodo su tutti i client connessi eccetto il client che ha richiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-151">Calls a method on all connected clients except the client that invoked the method</span></span> |

<span data-ttu-id="97fc2-152">`Hub.Clients` contiene inoltre i metodi seguenti:</span><span class="sxs-lookup"><span data-stu-id="97fc2-152">`Hub.Clients` also contains the following methods:</span></span>

| <span data-ttu-id="97fc2-153">Metodo</span><span class="sxs-lookup"><span data-stu-id="97fc2-153">Method</span></span> | <span data-ttu-id="97fc2-154">Descrizione</span><span class="sxs-lookup"><span data-stu-id="97fc2-154">Description</span></span> |
| ------ | ----------- |
| `AllExcept` | <span data-ttu-id="97fc2-155">Chiama un metodo su tutti i client connessi ad eccezione delle connessioni specificate</span><span class="sxs-lookup"><span data-stu-id="97fc2-155">Calls a method on all connected clients except for the specified connections</span></span> |
| `Client` | <span data-ttu-id="97fc2-156">Chiama un metodo su uno specifico client connesso</span><span class="sxs-lookup"><span data-stu-id="97fc2-156">Calls a method on a specific connected client</span></span> |
| `Clients` | <span data-ttu-id="97fc2-157">Chiama un metodo su client connessi specifici</span><span class="sxs-lookup"><span data-stu-id="97fc2-157">Calls a method on specific connected clients</span></span> |
| `Group` | <span data-ttu-id="97fc2-158">Chiama un metodo su tutte le connessioni nel gruppo specificato</span><span class="sxs-lookup"><span data-stu-id="97fc2-158">Calls a method on all connections in the specified group</span></span>  |
| `GroupExcept` | <span data-ttu-id="97fc2-159">Chiama un metodo su tutte le connessioni nel gruppo specificato, ad eccezione delle connessioni specificate.</span><span class="sxs-lookup"><span data-stu-id="97fc2-159">Calls a method on all connections in the specified group, except the specified connections</span></span> |
| `Groups` | <span data-ttu-id="97fc2-160">Chiama un metodo su più gruppi di connessioni</span><span class="sxs-lookup"><span data-stu-id="97fc2-160">Calls a method on multiple groups of connections</span></span>  |
| `OthersInGroup` | <span data-ttu-id="97fc2-161">Chiama un metodo su un gruppo di connessioni, escluso il client che ha richiamato il metodo Hub</span><span class="sxs-lookup"><span data-stu-id="97fc2-161">Calls a method on a group of connections, excluding the client that invoked the hub method</span></span>  |
| `User` | <span data-ttu-id="97fc2-162">Chiama un metodo su tutte le connessioni associate a un utente specifico</span><span class="sxs-lookup"><span data-stu-id="97fc2-162">Calls a method on all connections associated with a specific user</span></span> |
| `Users` | <span data-ttu-id="97fc2-163">Chiama un metodo su tutte le connessioni associate agli utenti specificati</span><span class="sxs-lookup"><span data-stu-id="97fc2-163">Calls a method on all connections associated with the specified users</span></span> |

<span data-ttu-id="97fc2-164">Ogni proprietà o metodo nelle tabelle precedenti restituisce un oggetto con un `SendAsync` metodo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-164">Each property or method in the preceding tables returns an object with a `SendAsync` method.</span></span> <span data-ttu-id="97fc2-165">Il `SendAsync` metodo consente di specificare il nome e i parametri del metodo client da chiamare.</span><span class="sxs-lookup"><span data-stu-id="97fc2-165">The `SendAsync` method allows you to supply the name and parameters of the client method to call.</span></span>

## <a name="send-messages-to-clients"></a><span data-ttu-id="97fc2-166">Inviare messaggi ai client</span><span class="sxs-lookup"><span data-stu-id="97fc2-166">Send messages to clients</span></span>

<span data-ttu-id="97fc2-167">Per effettuare chiamate a client specifici, utilizzare le proprietà dell' `Clients` oggetto.</span><span class="sxs-lookup"><span data-stu-id="97fc2-167">To make calls to specific clients, use the properties of the `Clients` object.</span></span> <span data-ttu-id="97fc2-168">Nell'esempio seguente sono disponibili tre metodi dell'hub:</span><span class="sxs-lookup"><span data-stu-id="97fc2-168">In the following example, there are three Hub methods:</span></span>

* <span data-ttu-id="97fc2-169">`SendMessage` Invia un messaggio a tutti i client connessi tramite `Clients.All` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-169">`SendMessage` sends a message to all connected clients, using `Clients.All`.</span></span>
* <span data-ttu-id="97fc2-170">`SendMessageToCaller` Invia un messaggio di nuovo al chiamante utilizzando `Clients.Caller` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-170">`SendMessageToCaller` sends a message back to the caller, using `Clients.Caller`.</span></span>
* <span data-ttu-id="97fc2-171">`SendMessageToGroups` Invia un messaggio a tutti i client del `SignalR Users` gruppo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-171">`SendMessageToGroups` sends a message to all clients in the `SignalR Users` group.</span></span>

[!code-csharp[Send messages](hubs/sample/hubs/chathub.cs?name=HubMethods)]

## <a name="strongly-typed-hubs"></a><span data-ttu-id="97fc2-172">Hub fortemente tipizzati</span><span class="sxs-lookup"><span data-stu-id="97fc2-172">Strongly typed hubs</span></span>

<span data-ttu-id="97fc2-173">Uno svantaggio dell'uso di `SendAsync` è che si basa su una stringa magica per specificare il metodo client da chiamare.</span><span class="sxs-lookup"><span data-stu-id="97fc2-173">A drawback of using `SendAsync` is that it relies on a magic string to specify the client method to be called.</span></span> <span data-ttu-id="97fc2-174">In questo modo si lascia il codice aperto agli errori di runtime se il nome del metodo non è stato digitato correttamente o non è presente nel client.</span><span class="sxs-lookup"><span data-stu-id="97fc2-174">This leaves code open to runtime errors if the method name is misspelled or missing from the client.</span></span>

<span data-ttu-id="97fc2-175">Un'alternativa all'utilizzo di `SendAsync` consiste nel digitare fortemente `Hub` con <xref:Microsoft.AspNetCore.SignalR.Hub%601> .</span><span class="sxs-lookup"><span data-stu-id="97fc2-175">An alternative to using `SendAsync` is to strongly type the `Hub` with <xref:Microsoft.AspNetCore.SignalR.Hub%601>.</span></span> <span data-ttu-id="97fc2-176">Nell'esempio seguente, i `ChatHub` metodi client sono stati estratti in un'interfaccia denominata `IChatClient` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-176">In the following example, the `ChatHub` client methods have been extracted out into an interface called `IChatClient`.</span></span>

[!code-csharp[Interface for IChatClient](hubs/sample/hubs/ichatclient.cs?name=snippet_IChatClient)]

<span data-ttu-id="97fc2-177">Questa interfaccia può essere utilizzata per effettuare il refactoring dell' `ChatHub` esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="97fc2-177">This interface can be used to refactor the preceding `ChatHub` example.</span></span>

[!code-csharp[Strongly typed ChatHub](hubs/sample/hubs/StronglyTypedChatHub.cs?range=8-18,36)]

<span data-ttu-id="97fc2-178">Tramite viene `Hub<IChatClient>` abilitato il controllo in fase di compilazione dei metodi client.</span><span class="sxs-lookup"><span data-stu-id="97fc2-178">Using `Hub<IChatClient>` enables compile-time checking of the client methods.</span></span> <span data-ttu-id="97fc2-179">In questo modo si evitano i problemi causati dall'uso di stringhe magiche, poiché `Hub<T>` può fornire solo l'accesso ai metodi definiti nell'interfaccia.</span><span class="sxs-lookup"><span data-stu-id="97fc2-179">This prevents issues caused by using magic strings, since `Hub<T>` can only provide access to the methods defined in the interface.</span></span>

<span data-ttu-id="97fc2-180">L'uso di un oggetto fortemente tipizzato `Hub<T>` Disabilita la possibilità di usare `SendAsync` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-180">Using a strongly typed `Hub<T>` disables the ability to use `SendAsync`.</span></span> <span data-ttu-id="97fc2-181">Tutti i metodi definiti nell'interfaccia possono comunque essere definiti come asincroni.</span><span class="sxs-lookup"><span data-stu-id="97fc2-181">Any methods defined on the interface can still be defined as asynchronous.</span></span> <span data-ttu-id="97fc2-182">Ognuno di questi metodi deve infatti restituire `Task` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-182">In fact, each of these methods should return a `Task`.</span></span> <span data-ttu-id="97fc2-183">Poiché si tratta di un'interfaccia, non usare la `async` parola chiave.</span><span class="sxs-lookup"><span data-stu-id="97fc2-183">Since it's an interface, don't use the `async` keyword.</span></span> <span data-ttu-id="97fc2-184">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="97fc2-184">For example:</span></span>

```csharp
public interface IClient
{
    Task ClientMethod();
}
```

> [!NOTE]
> <span data-ttu-id="97fc2-185">Il `Async` suffisso non viene rimosso dal nome del metodo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-185">The `Async` suffix isn't stripped from the method name.</span></span> <span data-ttu-id="97fc2-186">A meno che il metodo client non sia definito con, non è consigliabile `.on('MyMethodAsync')` utilizzare `MyMethodAsync` come nome.</span><span class="sxs-lookup"><span data-stu-id="97fc2-186">Unless your client method is defined with `.on('MyMethodAsync')`, you shouldn't use `MyMethodAsync` as a name.</span></span>

## <a name="change-the-name-of-a-hub-method"></a><span data-ttu-id="97fc2-187">Modificare il nome di un metodo Hub</span><span class="sxs-lookup"><span data-stu-id="97fc2-187">Change the name of a hub method</span></span>

<span data-ttu-id="97fc2-188">Per impostazione predefinita, un nome di metodo dell'Hub server è il nome del metodo .NET.</span><span class="sxs-lookup"><span data-stu-id="97fc2-188">By default, a server hub method name is the name of the .NET method.</span></span> <span data-ttu-id="97fc2-189">Tuttavia, è possibile usare l'attributo [HubMethodName](xref:Microsoft.AspNetCore.SignalR.HubMethodNameAttribute) per modificare questa impostazione predefinita e specificare manualmente un nome per il metodo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-189">However, you can use the [HubMethodName](xref:Microsoft.AspNetCore.SignalR.HubMethodNameAttribute) attribute to change this default and manually specify a name for the method.</span></span> <span data-ttu-id="97fc2-190">Il client deve usare questo nome, anziché il nome del metodo .NET, quando viene richiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-190">The client should use this name, instead of the .NET method name, when invoking the method.</span></span>

[!code-csharp[HubMethodName attribute](hubs/sample/hubs/chathub.cs?name=HubMethodName&highlight=1)]

## <a name="handle-events-for-a-connection"></a><span data-ttu-id="97fc2-191">Gestire gli eventi per una connessione</span><span class="sxs-lookup"><span data-stu-id="97fc2-191">Handle events for a connection</span></span>

<span data-ttu-id="97fc2-192">L' SignalR API Hub fornisce i `OnConnectedAsync` `OnDisconnectedAsync` metodi virtuali e per gestire e tenere traccia delle connessioni.</span><span class="sxs-lookup"><span data-stu-id="97fc2-192">The SignalR Hubs API provides the `OnConnectedAsync` and `OnDisconnectedAsync` virtual methods to manage and track connections.</span></span> <span data-ttu-id="97fc2-193">Eseguire l'override del `OnConnectedAsync` metodo virtuale per eseguire azioni quando un client si connette all'hub, ad esempio aggiungendolo a un gruppo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-193">Override the `OnConnectedAsync` virtual method to perform actions when a client connects to the Hub, such as adding it to a group.</span></span>

[!code-csharp[Handle connection](hubs/sample/hubs/chathub.cs?name=OnConnectedAsync)]

<span data-ttu-id="97fc2-194">Eseguire l'override del `OnDisconnectedAsync` metodo virtuale per eseguire azioni quando un client si disconnette.</span><span class="sxs-lookup"><span data-stu-id="97fc2-194">Override the `OnDisconnectedAsync` virtual method to perform actions when a client disconnects.</span></span> <span data-ttu-id="97fc2-195">Se il client si disconnette intenzionalmente (chiamando `connection.stop()` , ad esempio), il `exception` parametro sarà `null` .</span><span class="sxs-lookup"><span data-stu-id="97fc2-195">If the client disconnects intentionally (by calling `connection.stop()`, for example), the `exception` parameter will be `null`.</span></span> <span data-ttu-id="97fc2-196">Tuttavia, se il client è disconnesso a causa di un errore, ad esempio un errore di rete, il parametro conterrà `exception` un'eccezione che descrive l'errore.</span><span class="sxs-lookup"><span data-stu-id="97fc2-196">However, if the client is disconnected due to an error (such as a network failure), the `exception` parameter will contain an exception describing the failure.</span></span>

[!code-csharp[Handle disconnection](hubs/sample/hubs/chathub.cs?name=OnDisconnectedAsync)]

[!INCLUDE[](~/includes/connectionid-signalr.md)]

## <a name="handle-errors"></a><span data-ttu-id="97fc2-197">Gestire gli errori</span><span class="sxs-lookup"><span data-stu-id="97fc2-197">Handle errors</span></span>

<span data-ttu-id="97fc2-198">Le eccezioni generate nei metodi dell'hub vengono inviate al client che ha richiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="97fc2-198">Exceptions thrown in your hub methods are sent to the client that invoked the method.</span></span> <span data-ttu-id="97fc2-199">Nel client JavaScript il `invoke` metodo restituisce una [promessa JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises).</span><span class="sxs-lookup"><span data-stu-id="97fc2-199">On the JavaScript client, the `invoke` method returns a [JavaScript Promise](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises).</span></span> <span data-ttu-id="97fc2-200">Quando il client riceve un errore con un gestore associato alla promessa usando `catch` , viene richiamato e passato come `Error` oggetto JavaScript.</span><span class="sxs-lookup"><span data-stu-id="97fc2-200">When the client receives an error with a handler attached to the promise using `catch`, it's invoked and passed as a JavaScript `Error` object.</span></span>

[!code-javascript[Error](hubs/sample/wwwroot/js/chat.js?range=23)]

<span data-ttu-id="97fc2-201">Se l'Hub genera un'eccezione, le connessioni non vengono chiuse.</span><span class="sxs-lookup"><span data-stu-id="97fc2-201">If your Hub throws an exception, connections aren't closed.</span></span> <span data-ttu-id="97fc2-202">Per impostazione predefinita, nel SignalR client viene restituito un messaggio di errore generico.</span><span class="sxs-lookup"><span data-stu-id="97fc2-202">By default, SignalR returns a generic error message to the client.</span></span> <span data-ttu-id="97fc2-203">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="97fc2-203">For example:</span></span>

```
Microsoft.AspNetCore.SignalR.HubException: An unexpected error occurred invoking 'MethodName' on the server.
```

<span data-ttu-id="97fc2-204">Le eccezioni impreviste spesso contengono informazioni riservate, ad esempio il nome di un server di database in un'eccezione attivata quando la connessione al database non riesce.</span><span class="sxs-lookup"><span data-stu-id="97fc2-204">Unexpected exceptions often contain sensitive information, such as the name of a database server in an exception triggered when the database connection fails.</span></span> <span data-ttu-id="97fc2-205">SignalR non espone questi messaggi di errore dettagliati per impostazione predefinita come misura di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="97fc2-205">SignalR doesn't expose these detailed error messages by default as a security measure.</span></span> <span data-ttu-id="97fc2-206">Vedere l' [articolo Considerazioni sulla sicurezza](xref:signalr/security#exceptions) per ulteriori informazioni sul motivo per cui i dettagli dell'eccezione vengono eliminati.</span><span class="sxs-lookup"><span data-stu-id="97fc2-206">See the [Security considerations article](xref:signalr/security#exceptions) for more information on why exception details are suppressed.</span></span>

<span data-ttu-id="97fc2-207">Se si ha una condizione *eccezionale da* propagare al client, è possibile usare la `HubException` classe.</span><span class="sxs-lookup"><span data-stu-id="97fc2-207">If you have an exceptional condition you *do* want to propagate to the client, you can use the `HubException` class.</span></span> <span data-ttu-id="97fc2-208">Se si genera un'operazione `HubException` dal metodo Hub, SignalR  invierà l'intero messaggio al client, senza modifiche.</span><span class="sxs-lookup"><span data-stu-id="97fc2-208">If you throw a `HubException` from your hub method, SignalR **will** send the entire message to the client, unmodified.</span></span>

[!code-csharp[ThrowHubException](hubs/sample/hubs/chathub.cs?name=ThrowHubException&highlight=3)]

> [!NOTE]
> <span data-ttu-id="97fc2-209">SignalR Invia solo la `Message` proprietà dell'eccezione al client.</span><span class="sxs-lookup"><span data-stu-id="97fc2-209">SignalR only sends the `Message` property of the exception to the client.</span></span> <span data-ttu-id="97fc2-210">L'analisi dello stack e altre proprietà dell'eccezione non sono disponibili per il client.</span><span class="sxs-lookup"><span data-stu-id="97fc2-210">The stack trace and other properties on the exception aren't available to the client.</span></span>

## <a name="related-resources"></a><span data-ttu-id="97fc2-211">Risorse correlate</span><span class="sxs-lookup"><span data-stu-id="97fc2-211">Related resources</span></span>

* [<span data-ttu-id="97fc2-212">Introduzione a ASP.NET Core SignalR</span><span class="sxs-lookup"><span data-stu-id="97fc2-212">Intro to ASP.NET Core SignalR</span></span>](xref:signalr/introduction)
* [<span data-ttu-id="97fc2-213">Client JavaScript</span><span class="sxs-lookup"><span data-stu-id="97fc2-213">JavaScript client</span></span>](xref:signalr/javascript-client)
* [<span data-ttu-id="97fc2-214">Pubblicare in Azure</span><span class="sxs-lookup"><span data-stu-id="97fc2-214">Publish to Azure</span></span>](xref:signalr/publish-to-azure-web-app)
