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
# <a name="use-hubs-in-signalr-for-aspnet-core"></a>Usare gli hub in SignalR per ASP.NET Core

Di [Rachel Appel](https://twitter.com/rachelappel) e [Kevin Griffin](https://twitter.com/1kevgriff)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/hubs/sample/ ) [(procedura per il download)](xref:index#how-to-download-a-sample)

## <a name="what-is-a-signalr-hub"></a>Che cos'è un SignalR Hub

L' SignalR API Hub consente di chiamare i metodi sui client connessi dal server. Nel codice del server si definiscono i metodi che vengono chiamati dal client. Nel codice client è possibile definire metodi che vengono chiamati dal server. SignalR gestisce tutti gli elementi dietro le quinte che rendono possibili le comunicazioni da client a server e da server a client in tempo reale.

## <a name="configure-signalr-hubs"></a>Configurare gli SignalR Hub

Il SignalR middleware richiede alcuni servizi, che vengono configurati chiamando `services.AddSignalR` .

[!code-csharp[Configure service](hubs/sample/startup.cs?range=38)]

::: moniker range=">= aspnetcore-3.0"

Quando si aggiungono SignalR funzionalità a un'app ASP.NET Core, SignalR le route vengono impostate chiamando `endpoint.MapHub` nel callback del `Startup.Configure` metodo `app.UseEndpoints` .

```csharp
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapHub<ChatHub>("/chathub");
});
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

Quando si aggiungono SignalR funzionalità a un'app ASP.NET Core, SignalR le route vengono impostate chiamando `app.UseSignalR` nel `Startup.Configure` metodo.

[!code-csharp[Configure routes to hubs](hubs/sample/startup.cs?range=57-60)]

::: moniker-end

## <a name="create-and-use-hubs"></a>Creare e usare gli hub

Creare un hub dichiarando una classe che eredita da `Hub` e aggiungervi metodi pubblici. I client possono chiamare metodi definiti come `public` .

```csharp
public class ChatHub : Hub
{
    public Task SendMessage(string user, string message)
    {
        return Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

È possibile specificare un tipo restituito e i parametri, inclusi i tipi e le matrici complessi, come si farebbe con qualsiasi metodo C#. SignalR gestisce la serializzazione e la deserializzazione di oggetti e matrici complessi nei parametri e valori restituiti.

> [!NOTE]
> Gli hub sono temporanei:
>
> * Non archiviare lo stato in una proprietà nella classe Hub. Ogni chiamata al metodo dell'hub viene eseguita su una nuova istanza dell'hub.
> * Usare `await` quando si chiamano metodi asincroni che dipendono dall'hub rimanendo attivo. Ad esempio, un metodo come `Clients.All.SendAsync(...)` può avere esito negativo se viene chiamato senza `await` e il metodo Hub viene completato prima del completamento `SendAsync` .

## <a name="the-context-object"></a>Oggetto context

La `Hub` classe dispone di una `Context` proprietà che contiene le proprietà seguenti con le informazioni sulla connessione:

| Proprietà | Descrizione |
| ------ | ----------- |
| `ConnectionId` | Ottiene l'ID univoco per la connessione, assegnato da SignalR . È presente un ID di connessione per ogni connessione.|
| `UserIdentifier` | Ottiene l' [identificatore utente](xref:signalr/groups). Per impostazione predefinita, SignalR utilizza l'oggetto `ClaimTypes.NameIdentifier` dall'oggetto associato alla `ClaimsPrincipal` connessione come identificatore utente. |
| `User` | Ottiene l'oggetto `ClaimsPrincipal` associato all'utente corrente. |
| `Items` | Ottiene una raccolta chiave/valore che può essere utilizzata per condividere i dati nell'ambito di questa connessione. I dati possono essere archiviati in questa raccolta e verranno mantenuti per la connessione tra diverse chiamate al metodo dell'hub. |
| `Features` | Ottiene la raccolta di funzionalità disponibili nella connessione. Per il momento, questa raccolta non è necessaria nella maggior parte degli scenari, quindi non è ancora documentata in dettaglio. |
| `ConnectionAborted` | Ottiene un oggetto che invia una `CancellationToken` notifica quando la connessione viene interrotta. |

`Hub.Context` contiene inoltre i metodi seguenti:

| Metodo | Descrizione |
| ------ | ----------- |
| `GetHttpContext` | Restituisce l'oggetto `HttpContext` per la connessione o `null` se la connessione non è associata a una richiesta HTTP. Per le connessioni HTTP, è possibile usare questo metodo per ottenere informazioni come le intestazioni HTTP e le stringhe di query. |
| `Abort` | Interrompe la connessione. |

## <a name="the-clients-object"></a>Oggetto clients

La `Hub` classe dispone di una `Clients` proprietà che contiene le proprietà seguenti per la comunicazione tra server e client:

| Proprietà | Descrizione |
| ------ | ----------- |
| `All` | Chiama un metodo su tutti i client connessi |
| `Caller` | Chiama un metodo sul client che ha richiamato il metodo Hub |
| `Others` | Chiama un metodo su tutti i client connessi eccetto il client che ha richiamato il metodo. |

`Hub.Clients` contiene inoltre i metodi seguenti:

| Metodo | Descrizione |
| ------ | ----------- |
| `AllExcept` | Chiama un metodo su tutti i client connessi ad eccezione delle connessioni specificate |
| `Client` | Chiama un metodo su uno specifico client connesso |
| `Clients` | Chiama un metodo su client connessi specifici |
| `Group` | Chiama un metodo su tutte le connessioni nel gruppo specificato  |
| `GroupExcept` | Chiama un metodo su tutte le connessioni nel gruppo specificato, ad eccezione delle connessioni specificate. |
| `Groups` | Chiama un metodo su più gruppi di connessioni  |
| `OthersInGroup` | Chiama un metodo su un gruppo di connessioni, escluso il client che ha richiamato il metodo Hub  |
| `User` | Chiama un metodo su tutte le connessioni associate a un utente specifico |
| `Users` | Chiama un metodo su tutte le connessioni associate agli utenti specificati |

Ogni proprietà o metodo nelle tabelle precedenti restituisce un oggetto con un `SendAsync` metodo. Il `SendAsync` metodo consente di specificare il nome e i parametri del metodo client da chiamare.

## <a name="send-messages-to-clients"></a>Inviare messaggi ai client

Per effettuare chiamate a client specifici, utilizzare le proprietà dell' `Clients` oggetto. Nell'esempio seguente sono disponibili tre metodi dell'hub:

* `SendMessage` Invia un messaggio a tutti i client connessi tramite `Clients.All` .
* `SendMessageToCaller` Invia un messaggio di nuovo al chiamante utilizzando `Clients.Caller` .
* `SendMessageToGroups` Invia un messaggio a tutti i client del `SignalR Users` gruppo.

[!code-csharp[Send messages](hubs/sample/hubs/chathub.cs?name=HubMethods)]

## <a name="strongly-typed-hubs"></a>Hub fortemente tipizzati

Uno svantaggio dell'uso di `SendAsync` è che si basa su una stringa magica per specificare il metodo client da chiamare. In questo modo si lascia il codice aperto agli errori di runtime se il nome del metodo non è stato digitato correttamente o non è presente nel client.

Un'alternativa all'utilizzo di `SendAsync` consiste nel digitare fortemente `Hub` con <xref:Microsoft.AspNetCore.SignalR.Hub%601> . Nell'esempio seguente, i `ChatHub` metodi client sono stati estratti in un'interfaccia denominata `IChatClient` .

[!code-csharp[Interface for IChatClient](hubs/sample/hubs/ichatclient.cs?name=snippet_IChatClient)]

Questa interfaccia può essere utilizzata per effettuare il refactoring dell' `ChatHub` esempio precedente.

[!code-csharp[Strongly typed ChatHub](hubs/sample/hubs/StronglyTypedChatHub.cs?range=8-18,36)]

Tramite viene `Hub<IChatClient>` abilitato il controllo in fase di compilazione dei metodi client. In questo modo si evitano i problemi causati dall'uso di stringhe magiche, poiché `Hub<T>` può fornire solo l'accesso ai metodi definiti nell'interfaccia.

L'uso di un oggetto fortemente tipizzato `Hub<T>` Disabilita la possibilità di usare `SendAsync` . Tutti i metodi definiti nell'interfaccia possono comunque essere definiti come asincroni. Ognuno di questi metodi deve infatti restituire `Task` . Poiché si tratta di un'interfaccia, non usare la `async` parola chiave. Ad esempio:

```csharp
public interface IClient
{
    Task ClientMethod();
}
```

> [!NOTE]
> Il `Async` suffisso non viene rimosso dal nome del metodo. A meno che il metodo client non sia definito con, non è consigliabile `.on('MyMethodAsync')` utilizzare `MyMethodAsync` come nome.

## <a name="change-the-name-of-a-hub-method"></a>Modificare il nome di un metodo Hub

Per impostazione predefinita, un nome di metodo dell'Hub server è il nome del metodo .NET. Tuttavia, è possibile usare l'attributo [HubMethodName](xref:Microsoft.AspNetCore.SignalR.HubMethodNameAttribute) per modificare questa impostazione predefinita e specificare manualmente un nome per il metodo. Il client deve usare questo nome, anziché il nome del metodo .NET, quando viene richiamato il metodo.

[!code-csharp[HubMethodName attribute](hubs/sample/hubs/chathub.cs?name=HubMethodName&highlight=1)]

## <a name="handle-events-for-a-connection"></a>Gestire gli eventi per una connessione

L' SignalR API Hub fornisce i `OnConnectedAsync` `OnDisconnectedAsync` metodi virtuali e per gestire e tenere traccia delle connessioni. Eseguire l'override del `OnConnectedAsync` metodo virtuale per eseguire azioni quando un client si connette all'hub, ad esempio aggiungendolo a un gruppo.

[!code-csharp[Handle connection](hubs/sample/hubs/chathub.cs?name=OnConnectedAsync)]

Eseguire l'override del `OnDisconnectedAsync` metodo virtuale per eseguire azioni quando un client si disconnette. Se il client si disconnette intenzionalmente (chiamando `connection.stop()` , ad esempio), il `exception` parametro sarà `null` . Tuttavia, se il client è disconnesso a causa di un errore, ad esempio un errore di rete, il parametro conterrà `exception` un'eccezione che descrive l'errore.

[!code-csharp[Handle disconnection](hubs/sample/hubs/chathub.cs?name=OnDisconnectedAsync)]

[!INCLUDE[](~/includes/connectionid-signalr.md)]

## <a name="handle-errors"></a>Gestire gli errori

Le eccezioni generate nei metodi dell'hub vengono inviate al client che ha richiamato il metodo. Nel client JavaScript il `invoke` metodo restituisce una [promessa JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises). Quando il client riceve un errore con un gestore associato alla promessa usando `catch` , viene richiamato e passato come `Error` oggetto JavaScript.

[!code-javascript[Error](hubs/sample/wwwroot/js/chat.js?range=23)]

Se l'Hub genera un'eccezione, le connessioni non vengono chiuse. Per impostazione predefinita, nel SignalR client viene restituito un messaggio di errore generico. Ad esempio:

```
Microsoft.AspNetCore.SignalR.HubException: An unexpected error occurred invoking 'MethodName' on the server.
```

Le eccezioni impreviste spesso contengono informazioni riservate, ad esempio il nome di un server di database in un'eccezione attivata quando la connessione al database non riesce. SignalR non espone questi messaggi di errore dettagliati per impostazione predefinita come misura di sicurezza. Vedere l' [articolo Considerazioni sulla sicurezza](xref:signalr/security#exceptions) per ulteriori informazioni sul motivo per cui i dettagli dell'eccezione vengono eliminati.

Se si ha una condizione *eccezionale da* propagare al client, è possibile usare la `HubException` classe. Se si genera un'operazione `HubException` dal metodo Hub, SignalR  invierà l'intero messaggio al client, senza modifiche.

[!code-csharp[ThrowHubException](hubs/sample/hubs/chathub.cs?name=ThrowHubException&highlight=3)]

> [!NOTE]
> SignalR Invia solo la `Message` proprietà dell'eccezione al client. L'analisi dello stack e altre proprietà dell'eccezione non sono disponibili per il client.

## <a name="related-resources"></a>Risorse correlate

* [Introduzione a ASP.NET Core SignalR](xref:signalr/introduction)
* [Client JavaScript](xref:signalr/javascript-client)
* [Pubblicare in Azure](xref:signalr/publish-to-azure-web-app)
