---
title: Gestione degli errori temporanei con i tentativi gRPC
author: jamesnk
description: Informazioni su come creare chiamate gRPC resilienti e a tolleranza di errore con nuovi tentativi in .NET.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 02/25/2021
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
uid: grpc/retries
ms.openlocfilehash: 613386d1fedd8b1b04081e3240b8a3aaf7b37012
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589804"
---
# <a name="transient-fault-handling-with-grpc-retries"></a>Gestione degli errori temporanei con i tentativi gRPC

Di [James Newton-King](https://twitter.com/jamesnk)

gRPC tentativi è una funzionalità che consente ai client gRPC di ritentare automaticamente le chiamate non riuscite. Questo articolo illustra come configurare un criterio di ripetizione per creare app gRPC resilienti e a tolleranza di errore in .NET.

per i tentativi di gRPC è necessario [gRPC .NET. client](https://www.nuget.org/packages/Grpc.Net.Client) versione 2.36.0-pre1 o versioni successive.

## <a name="transient-fault-handling"></a>Gestione degli errori temporanei

le chiamate gRPC possono essere interrotte da errori temporanei. Gli errori temporanei includono:

* Perdita temporanea della connettività di rete.
* Indisponibilità temporanea di un servizio.
* Timeout a causa del carico del server.

Quando una chiamata a gRPC viene interrotta, il client genera un'eccezione `RpcException` con informazioni dettagliate sull'errore. L'app client deve rilevare l'eccezione e scegliere come gestire l'errore.

```csharp
var client = new Greeter.GreeterClient(channel);
try
{
    var response = await client.SayHelloAsync(
        new HelloRequest { Name = ".NET" });

    Console.WriteLine("From server: " + response.Message);
}
catch (RpcException ex)
{
    // Write logic to inspect the error and retry
    // if the error is from a transient fault.
}
```

La duplicazione della logica di ripetizione dei tentativi in un'app è dettagliata e soggetta a errori. Fortunatamente il client .NET gRPC dispone di un supporto incorporato per i tentativi automatici.

## <a name="configure-a-grpc-retry-policy"></a>Configurare un criterio di ripetizione dei tentativi gRPC

Un criterio di ripetizione dei tentativi viene configurato una volta quando viene creato un canale gRPC:

```csharp
var defaultMethodConfig = new MethodConfig
{
    Names = { MethodName.Default },
    RetryPolicy = new RetryPolicy
    {
        MaxAttempts = 5,
        InitialBackoff = TimeSpan.FromSeconds(1),
        MaxBackoff = TimeSpan.FromSeconds(5),
        BackoffMultiplier = 1.5,
        RetryableStatusCodes = { StatusCode.Unavailable }
    }
};

var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    ServiceConfig = new ServiceConfig { MethodConfigs = { defaultMethodConfig } }
});
```

Il codice precedente:

* Crea un oggetto `MethodConfig`. I criteri di ripetizione dei tentativi possono essere configurati in base al metodo e i metodi vengono associati tramite la `Names` Proprietà. Questo metodo è configurato con `MethodName.Default` , quindi viene applicato a tutti i metodi gRPC chiamati da questo canale.
* Configura un criterio di ripetizione dei tentativi. Questo criterio indica ai client di ritentare automaticamente le chiamate gRPC che hanno esito negativo con il codice di stato `Unavailable` .
* Configura il canale creato per usare i criteri di ripetizione dei tentativi impostando `GrpcChannelOptions.ServiceConfig` .

i client gRPC creati con il canale ritenteranno automaticamente le chiamate non riuscite:

```csharp
var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(
    new HelloRequest { Name = ".NET" });

Console.WriteLine("From server: " + response.Message);
```

### <a name="grpc-retry-options"></a>opzioni di ripetizione dei tentativi gRPC

Nella tabella seguente vengono descritte le opzioni per la configurazione dei criteri di ripetizione dei tentativi gRPC:

| Opzione | Descrizione |
| ------ | ----------- |
| `MaxAttempts` | Numero massimo di tentativi di chiamata, incluso il tentativo originale. Questo valore è limitato da `GrpcChannelOptions.MaxRetryAttempts` e il valore predefinito è 5. Un valore è obbligatorio e deve essere maggiore di 1. |
| `InitialBackoff` | Ritardo backoff iniziale tra tentativi. Un ritardo casuale compreso tra 0 e il backoff corrente determina quando viene eseguito il tentativo successivo. Dopo ogni tentativo, il backoff corrente viene moltiplicato per `BackoffMultiplier` . Un valore è obbligatorio e deve essere maggiore di zero. |
| `MaxBackoff` | Il numero massimo di backoff pone un limite superiore alla crescita esponenziale del backoff. Un valore è obbligatorio e deve essere maggiore di zero. |
| `BackoffMultiplier` | Il valore di backoff verrà moltiplicato per questo valore dopo ogni nuovo tentativo e aumenterà in modo esponenziale quando il moltiplicatore è maggiore di 1. Un valore è obbligatorio e deve essere maggiore di zero. |
| `RetryableStatusCodes` | Raccolta di codici di stato. Una chiamata gRPC che ha esito negativo con stato corrispondente verrà ritentata automaticamente. Per altre informazioni sui codici di stato, vedere [codici di stato e loro uso in gRPC](https://grpc.github.io/grpc/core/md_doc_statuscodes.html). È necessario almeno un codice di stato irreversibile. |

## <a name="hedging"></a>Copertura

La copertura è una strategia di ripetizione dei tentativi alternativa. La copertura consente l'invio aggressivo di più copie di una singola chiamata gRPC senza attendere una risposta. Le chiamate gRPC hedged possono essere eseguite più volte nel server e viene usato il primo risultato riuscito. È importante che la copertura sia abilitata solo per i metodi che possono essere eseguiti più volte senza effetti negativi.

La copertura presenta vantaggi e svantaggi rispetto ai tentativi: 

* Un vantaggio della copertura è che potrebbe restituire un risultato più rapido. Consente più chiamate gRPC simultanee e viene completata quando è disponibile il primo risultato riuscito. 
* Uno svantaggio nella copertura è che può risultare inutile. È possibile che vengano effettuate più chiamate e che abbiano esito positivo. Viene usato solo il primo risultato e il resto viene scartato.

## <a name="configure-a-grpc-hedging-policy"></a>Configurare un criterio di copertura gRPC

Un criterio di copertura è configurato come criterio di ripetizione dei tentativi. Si noti che non è possibile combinare criteri di copertura con criteri di ripetizione dei tentativi.

```csharp
var defaultMethodConfig = new MethodConfig
{
    Names = { MethodName.Default },
    HedgingPolicy = new HedgingPolicy
    {
        MaxAttempts = 5,
        NonFatalStatusCodes = { StatusCode.Unavailable }
    }
};

var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    ServiceConfig = new ServiceConfig { MethodConfigs = { defaultMethodConfig } }
});
```

### <a name="grpc-hedging-options"></a>opzioni di copertura gRPC

La tabella seguente descrive le opzioni per la configurazione dei criteri di copertura di gRPC:

| Opzione | Descrizione |
| ------ | ----------- |
| `MaxAttempts` | I criteri di copertura invieranno fino a questo numero di chiamate. `MaxAttempts` rappresenta il numero totale di tutti i tentativi, incluso il tentativo originale. Questo valore è limitato da `GrpcChannelOptions.MaxRetryAttempts` e il valore predefinito è 5. Un valore è obbligatorio e deve essere maggiore o uguale a 2. |
| `HedgingDelay` | La prima chiamata verrà inviata immediatamente, ma le successive chiamate di copertura verranno posticipate in base a questo valore. Quando il ritardo è impostato su zero o `null` , tutte le chiamate coperte vengono inviate immediatamente. Il valore predefinito è zero. |
| `NonFatalStatusCodes` | Una raccolta di codici di stato che indicano altre chiamate di hedge possono comunque avere esito positivo. Se il server restituisce un codice di stato non irreversibile, le chiamate hedge continueranno. In caso contrario, le richieste in sospeso verranno annullate e l'errore restituito all'app. Per altre informazioni sui codici di stato, vedere [codici di stato e loro uso in gRPC](https://grpc.github.io/grpc/core/md_doc_statuscodes.html). |

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:grpc/client>
* [Suggerimenti generali per i tentativi-procedure consigliate per le applicazioni cloud](/azure/architecture/best-practices/transient-faults)
