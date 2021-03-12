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
# <a name="transient-fault-handling-with-grpc-retries"></a><span data-ttu-id="6acf1-103">Gestione degli errori temporanei con i tentativi gRPC</span><span class="sxs-lookup"><span data-stu-id="6acf1-103">Transient fault handling with gRPC retries</span></span>

<span data-ttu-id="6acf1-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="6acf1-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="6acf1-105">gRPC tentativi è una funzionalità che consente ai client gRPC di ritentare automaticamente le chiamate non riuscite.</span><span class="sxs-lookup"><span data-stu-id="6acf1-105">gRPC retries is a feature that allows gRPC clients to automatically retry failed calls.</span></span> <span data-ttu-id="6acf1-106">Questo articolo illustra come configurare un criterio di ripetizione per creare app gRPC resilienti e a tolleranza di errore in .NET.</span><span class="sxs-lookup"><span data-stu-id="6acf1-106">This article discusses how to configure a retry policy to make resilient, fault tolerant gRPC apps in .NET.</span></span>

<span data-ttu-id="6acf1-107">per i tentativi di gRPC è necessario [gRPC .NET. client](https://www.nuget.org/packages/Grpc.Net.Client) versione 2.36.0-pre1 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="6acf1-107">gRPC retries requires [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) version 2.36.0-pre1 or later.</span></span>

## <a name="transient-fault-handling"></a><span data-ttu-id="6acf1-108">Gestione degli errori temporanei</span><span class="sxs-lookup"><span data-stu-id="6acf1-108">Transient fault handling</span></span>

<span data-ttu-id="6acf1-109">le chiamate gRPC possono essere interrotte da errori temporanei.</span><span class="sxs-lookup"><span data-stu-id="6acf1-109">gRPC calls can be interrupted by transient faults.</span></span> <span data-ttu-id="6acf1-110">Gli errori temporanei includono:</span><span class="sxs-lookup"><span data-stu-id="6acf1-110">Transient faults include:</span></span>

* <span data-ttu-id="6acf1-111">Perdita temporanea della connettività di rete.</span><span class="sxs-lookup"><span data-stu-id="6acf1-111">Momentary loss of network connectivity.</span></span>
* <span data-ttu-id="6acf1-112">Indisponibilità temporanea di un servizio.</span><span class="sxs-lookup"><span data-stu-id="6acf1-112">Temporary unavailability of a service.</span></span>
* <span data-ttu-id="6acf1-113">Timeout a causa del carico del server.</span><span class="sxs-lookup"><span data-stu-id="6acf1-113">Timeouts due to server load.</span></span>

<span data-ttu-id="6acf1-114">Quando una chiamata a gRPC viene interrotta, il client genera un'eccezione `RpcException` con informazioni dettagliate sull'errore.</span><span class="sxs-lookup"><span data-stu-id="6acf1-114">When a gRPC call is interrupted, the client throws an `RpcException` with details about the error.</span></span> <span data-ttu-id="6acf1-115">L'app client deve rilevare l'eccezione e scegliere come gestire l'errore.</span><span class="sxs-lookup"><span data-stu-id="6acf1-115">The client app must catch the exception and choose how to handle the error.</span></span>

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

<span data-ttu-id="6acf1-116">La duplicazione della logica di ripetizione dei tentativi in un'app è dettagliata e soggetta a errori.</span><span class="sxs-lookup"><span data-stu-id="6acf1-116">Duplicating retry logic throughout an app is verbose and error prone.</span></span> <span data-ttu-id="6acf1-117">Fortunatamente il client .NET gRPC dispone di un supporto incorporato per i tentativi automatici.</span><span class="sxs-lookup"><span data-stu-id="6acf1-117">Fortunately the .NET gRPC client has a built-in support for automatic retries.</span></span>

## <a name="configure-a-grpc-retry-policy"></a><span data-ttu-id="6acf1-118">Configurare un criterio di ripetizione dei tentativi gRPC</span><span class="sxs-lookup"><span data-stu-id="6acf1-118">Configure a gRPC retry policy</span></span>

<span data-ttu-id="6acf1-119">Un criterio di ripetizione dei tentativi viene configurato una volta quando viene creato un canale gRPC:</span><span class="sxs-lookup"><span data-stu-id="6acf1-119">A retry policy is configured once when a gRPC channel is created:</span></span>

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

<span data-ttu-id="6acf1-120">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="6acf1-120">The preceding code:</span></span>

* <span data-ttu-id="6acf1-121">Crea un oggetto `MethodConfig`.</span><span class="sxs-lookup"><span data-stu-id="6acf1-121">Creates a `MethodConfig`.</span></span> <span data-ttu-id="6acf1-122">I criteri di ripetizione dei tentativi possono essere configurati in base al metodo e i metodi vengono associati tramite la `Names` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="6acf1-122">Retry policies can be configured per-method and methods are matched using the `Names` property.</span></span> <span data-ttu-id="6acf1-123">Questo metodo è configurato con `MethodName.Default` , quindi viene applicato a tutti i metodi gRPC chiamati da questo canale.</span><span class="sxs-lookup"><span data-stu-id="6acf1-123">This method is configured with `MethodName.Default`, so it's applied to all gRPC methods called by this channel.</span></span>
* <span data-ttu-id="6acf1-124">Configura un criterio di ripetizione dei tentativi.</span><span class="sxs-lookup"><span data-stu-id="6acf1-124">Configures a retry policy.</span></span> <span data-ttu-id="6acf1-125">Questo criterio indica ai client di ritentare automaticamente le chiamate gRPC che hanno esito negativo con il codice di stato `Unavailable` .</span><span class="sxs-lookup"><span data-stu-id="6acf1-125">This policy instructs clients to automatically retry gRPC calls that fail with the status code `Unavailable`.</span></span>
* <span data-ttu-id="6acf1-126">Configura il canale creato per usare i criteri di ripetizione dei tentativi impostando `GrpcChannelOptions.ServiceConfig` .</span><span class="sxs-lookup"><span data-stu-id="6acf1-126">Configures the created channel to use the retry policy by setting `GrpcChannelOptions.ServiceConfig`.</span></span>

<span data-ttu-id="6acf1-127">i client gRPC creati con il canale ritenteranno automaticamente le chiamate non riuscite:</span><span class="sxs-lookup"><span data-stu-id="6acf1-127">gRPC clients created with the channel will automatically retry failed calls:</span></span>

```csharp
var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(
    new HelloRequest { Name = ".NET" });

Console.WriteLine("From server: " + response.Message);
```

### <a name="grpc-retry-options"></a><span data-ttu-id="6acf1-128">opzioni di ripetizione dei tentativi gRPC</span><span class="sxs-lookup"><span data-stu-id="6acf1-128">gRPC retry options</span></span>

<span data-ttu-id="6acf1-129">Nella tabella seguente vengono descritte le opzioni per la configurazione dei criteri di ripetizione dei tentativi gRPC:</span><span class="sxs-lookup"><span data-stu-id="6acf1-129">The following table describes options for configuring gRPC retry policies:</span></span>

| <span data-ttu-id="6acf1-130">Opzione</span><span class="sxs-lookup"><span data-stu-id="6acf1-130">Option</span></span> | <span data-ttu-id="6acf1-131">Descrizione</span><span class="sxs-lookup"><span data-stu-id="6acf1-131">Description</span></span> |
| ------ | ----------- |
| `MaxAttempts` | <span data-ttu-id="6acf1-132">Numero massimo di tentativi di chiamata, incluso il tentativo originale.</span><span class="sxs-lookup"><span data-stu-id="6acf1-132">The maximum number of call attempts, including the original attempt.</span></span> <span data-ttu-id="6acf1-133">Questo valore è limitato da `GrpcChannelOptions.MaxRetryAttempts` e il valore predefinito è 5.</span><span class="sxs-lookup"><span data-stu-id="6acf1-133">This value is limited by `GrpcChannelOptions.MaxRetryAttempts` which defaults to 5.</span></span> <span data-ttu-id="6acf1-134">Un valore è obbligatorio e deve essere maggiore di 1.</span><span class="sxs-lookup"><span data-stu-id="6acf1-134">A value is required and must be greater than 1.</span></span> |
| `InitialBackoff` | <span data-ttu-id="6acf1-135">Ritardo backoff iniziale tra tentativi.</span><span class="sxs-lookup"><span data-stu-id="6acf1-135">The initial backoff delay between retry attempts.</span></span> <span data-ttu-id="6acf1-136">Un ritardo casuale compreso tra 0 e il backoff corrente determina quando viene eseguito il tentativo successivo.</span><span class="sxs-lookup"><span data-stu-id="6acf1-136">A randomized delay between 0 and the current backoff determines when the next retry attempt is made.</span></span> <span data-ttu-id="6acf1-137">Dopo ogni tentativo, il backoff corrente viene moltiplicato per `BackoffMultiplier` .</span><span class="sxs-lookup"><span data-stu-id="6acf1-137">After each attempt, the current backoff is multiplied by `BackoffMultiplier`.</span></span> <span data-ttu-id="6acf1-138">Un valore è obbligatorio e deve essere maggiore di zero.</span><span class="sxs-lookup"><span data-stu-id="6acf1-138">A value is required and must be greater than zero.</span></span> |
| `MaxBackoff` | <span data-ttu-id="6acf1-139">Il numero massimo di backoff pone un limite superiore alla crescita esponenziale del backoff.</span><span class="sxs-lookup"><span data-stu-id="6acf1-139">The maximum backoff places an upper limit on exponential backoff growth.</span></span> <span data-ttu-id="6acf1-140">Un valore è obbligatorio e deve essere maggiore di zero.</span><span class="sxs-lookup"><span data-stu-id="6acf1-140">A value is required and must be greater than zero.</span></span> |
| `BackoffMultiplier` | <span data-ttu-id="6acf1-141">Il valore di backoff verrà moltiplicato per questo valore dopo ogni nuovo tentativo e aumenterà in modo esponenziale quando il moltiplicatore è maggiore di 1.</span><span class="sxs-lookup"><span data-stu-id="6acf1-141">The backoff will be multiplied by this value after each retry attempt and will increase exponentially when the multiplier is greater than 1.</span></span> <span data-ttu-id="6acf1-142">Un valore è obbligatorio e deve essere maggiore di zero.</span><span class="sxs-lookup"><span data-stu-id="6acf1-142">A value is required and must be greater than zero.</span></span> |
| `RetryableStatusCodes` | <span data-ttu-id="6acf1-143">Raccolta di codici di stato.</span><span class="sxs-lookup"><span data-stu-id="6acf1-143">A collection of status codes.</span></span> <span data-ttu-id="6acf1-144">Una chiamata gRPC che ha esito negativo con stato corrispondente verrà ritentata automaticamente.</span><span class="sxs-lookup"><span data-stu-id="6acf1-144">A gRPC call that fails with a matching status will be automatically retried.</span></span> <span data-ttu-id="6acf1-145">Per altre informazioni sui codici di stato, vedere [codici di stato e loro uso in gRPC](https://grpc.github.io/grpc/core/md_doc_statuscodes.html).</span><span class="sxs-lookup"><span data-stu-id="6acf1-145">For more information about status codes, see [Status codes and their use in gRPC](https://grpc.github.io/grpc/core/md_doc_statuscodes.html).</span></span> <span data-ttu-id="6acf1-146">È necessario almeno un codice di stato irreversibile.</span><span class="sxs-lookup"><span data-stu-id="6acf1-146">At least one retryable status code is required.</span></span> |

## <a name="hedging"></a><span data-ttu-id="6acf1-147">Copertura</span><span class="sxs-lookup"><span data-stu-id="6acf1-147">Hedging</span></span>

<span data-ttu-id="6acf1-148">La copertura è una strategia di ripetizione dei tentativi alternativa.</span><span class="sxs-lookup"><span data-stu-id="6acf1-148">Hedging is an alternative retry strategy.</span></span> <span data-ttu-id="6acf1-149">La copertura consente l'invio aggressivo di più copie di una singola chiamata gRPC senza attendere una risposta.</span><span class="sxs-lookup"><span data-stu-id="6acf1-149">Hedging enables aggressively sending multiple copies of a single gRPC call without waiting for a response.</span></span> <span data-ttu-id="6acf1-150">Le chiamate gRPC hedged possono essere eseguite più volte nel server e viene usato il primo risultato riuscito.</span><span class="sxs-lookup"><span data-stu-id="6acf1-150">Hedged gRPC calls may be executed multiple times on the server and the first successful result is used.</span></span> <span data-ttu-id="6acf1-151">È importante che la copertura sia abilitata solo per i metodi che possono essere eseguiti più volte senza effetti negativi.</span><span class="sxs-lookup"><span data-stu-id="6acf1-151">It's important that hedging is only enabled for methods that are safe to execute multiple times without adverse effect.</span></span>

<span data-ttu-id="6acf1-152">La copertura presenta vantaggi e svantaggi rispetto ai tentativi:</span><span class="sxs-lookup"><span data-stu-id="6acf1-152">Hedging has pros and cons when compared to retries:</span></span> 

* <span data-ttu-id="6acf1-153">Un vantaggio della copertura è che potrebbe restituire un risultato più rapido.</span><span class="sxs-lookup"><span data-stu-id="6acf1-153">An advantage to hedging is it might return a successful result faster.</span></span> <span data-ttu-id="6acf1-154">Consente più chiamate gRPC simultanee e viene completata quando è disponibile il primo risultato riuscito.</span><span class="sxs-lookup"><span data-stu-id="6acf1-154">It allows for multiple simultaneously gRPC calls and will complete when the first successful result is available.</span></span> 
* <span data-ttu-id="6acf1-155">Uno svantaggio nella copertura è che può risultare inutile.</span><span class="sxs-lookup"><span data-stu-id="6acf1-155">A disadvantage to hedging is it can be wasteful.</span></span> <span data-ttu-id="6acf1-156">È possibile che vengano effettuate più chiamate e che abbiano esito positivo.</span><span class="sxs-lookup"><span data-stu-id="6acf1-156">Multiple calls could be made and all succeed.</span></span> <span data-ttu-id="6acf1-157">Viene usato solo il primo risultato e il resto viene scartato.</span><span class="sxs-lookup"><span data-stu-id="6acf1-157">Only the first result is used and the rest are discarded.</span></span>

## <a name="configure-a-grpc-hedging-policy"></a><span data-ttu-id="6acf1-158">Configurare un criterio di copertura gRPC</span><span class="sxs-lookup"><span data-stu-id="6acf1-158">Configure a gRPC hedging policy</span></span>

<span data-ttu-id="6acf1-159">Un criterio di copertura è configurato come criterio di ripetizione dei tentativi.</span><span class="sxs-lookup"><span data-stu-id="6acf1-159">A hedging policy is configured like a retry policy.</span></span> <span data-ttu-id="6acf1-160">Si noti che non è possibile combinare criteri di copertura con criteri di ripetizione dei tentativi.</span><span class="sxs-lookup"><span data-stu-id="6acf1-160">Note that a hedging policy can't be combined with a retry policy.</span></span>

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

### <a name="grpc-hedging-options"></a><span data-ttu-id="6acf1-161">opzioni di copertura gRPC</span><span class="sxs-lookup"><span data-stu-id="6acf1-161">gRPC hedging options</span></span>

<span data-ttu-id="6acf1-162">La tabella seguente descrive le opzioni per la configurazione dei criteri di copertura di gRPC:</span><span class="sxs-lookup"><span data-stu-id="6acf1-162">The following table describes options for configuring gRPC hedging policies:</span></span>

| <span data-ttu-id="6acf1-163">Opzione</span><span class="sxs-lookup"><span data-stu-id="6acf1-163">Option</span></span> | <span data-ttu-id="6acf1-164">Descrizione</span><span class="sxs-lookup"><span data-stu-id="6acf1-164">Description</span></span> |
| ------ | ----------- |
| `MaxAttempts` | <span data-ttu-id="6acf1-165">I criteri di copertura invieranno fino a questo numero di chiamate.</span><span class="sxs-lookup"><span data-stu-id="6acf1-165">The hedging policy will send up to this number of calls.</span></span> <span data-ttu-id="6acf1-166">`MaxAttempts` rappresenta il numero totale di tutti i tentativi, incluso il tentativo originale.</span><span class="sxs-lookup"><span data-stu-id="6acf1-166">`MaxAttempts` represents the total number of all attempts, including the original attempt.</span></span> <span data-ttu-id="6acf1-167">Questo valore è limitato da `GrpcChannelOptions.MaxRetryAttempts` e il valore predefinito è 5.</span><span class="sxs-lookup"><span data-stu-id="6acf1-167">This value is limited by `GrpcChannelOptions.MaxRetryAttempts` which defaults to 5.</span></span> <span data-ttu-id="6acf1-168">Un valore è obbligatorio e deve essere maggiore o uguale a 2.</span><span class="sxs-lookup"><span data-stu-id="6acf1-168">A value is required and must be 2 or greater.</span></span> |
| `HedgingDelay` | <span data-ttu-id="6acf1-169">La prima chiamata verrà inviata immediatamente, ma le successive chiamate di copertura verranno posticipate in base a questo valore.</span><span class="sxs-lookup"><span data-stu-id="6acf1-169">The first call will be sent immediately, but the subsequent hedging calls will be delayed by this value.</span></span> <span data-ttu-id="6acf1-170">Quando il ritardo è impostato su zero o `null` , tutte le chiamate coperte vengono inviate immediatamente.</span><span class="sxs-lookup"><span data-stu-id="6acf1-170">When the delay is set to zero or `null`, all hedged calls are sent immediately.</span></span> <span data-ttu-id="6acf1-171">Il valore predefinito è zero.</span><span class="sxs-lookup"><span data-stu-id="6acf1-171">Default value is zero.</span></span> |
| `NonFatalStatusCodes` | <span data-ttu-id="6acf1-172">Una raccolta di codici di stato che indicano altre chiamate di hedge possono comunque avere esito positivo.</span><span class="sxs-lookup"><span data-stu-id="6acf1-172">A collection of status codes which indicate other hedge calls may still succeed.</span></span> <span data-ttu-id="6acf1-173">Se il server restituisce un codice di stato non irreversibile, le chiamate hedge continueranno.</span><span class="sxs-lookup"><span data-stu-id="6acf1-173">If a non-fatal status code is returned by the server, hedged calls will continue.</span></span> <span data-ttu-id="6acf1-174">In caso contrario, le richieste in sospeso verranno annullate e l'errore restituito all'app.</span><span class="sxs-lookup"><span data-stu-id="6acf1-174">Otherwise, outstanding requests will be canceled and the error returned to the app.</span></span> <span data-ttu-id="6acf1-175">Per altre informazioni sui codici di stato, vedere [codici di stato e loro uso in gRPC](https://grpc.github.io/grpc/core/md_doc_statuscodes.html).</span><span class="sxs-lookup"><span data-stu-id="6acf1-175">For more information about status codes, see [Status codes and their use in gRPC](https://grpc.github.io/grpc/core/md_doc_statuscodes.html).</span></span> |

## <a name="additional-resources"></a><span data-ttu-id="6acf1-176">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="6acf1-176">Additional resources</span></span>

* <xref:grpc/client>
* [<span data-ttu-id="6acf1-177">Suggerimenti generali per i tentativi-procedure consigliate per le applicazioni cloud</span><span class="sxs-lookup"><span data-stu-id="6acf1-177">Retry general guidance - Best practices for cloud applications</span></span>](/azure/architecture/best-practices/transient-faults)
