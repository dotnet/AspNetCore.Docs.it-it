---
title: configurazione di gRPC per .NET
author: jamesnk
description: Informazioni su come configurare gRPC per le app .NET.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 11/23/2020
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
uid: grpc/configuration
ms.openlocfilehash: 3f03d774a2223dc1fc08adcbc85cdc9a2b63dea0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588893"
---
# <a name="grpc-for-net-configuration"></a>configurazione di gRPC per .NET

## <a name="configure-services-options"></a>Configurare le opzioni dei servizi

i servizi gRPC sono configurati con `AddGrpc` in *Startup.cs*. Nella tabella seguente vengono descritte le opzioni per la configurazione dei servizi gRPC:

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| MaxSendMessageSize | `null` | Dimensione massima in byte del messaggio che può essere inviata dal server. Il tentativo di inviare un messaggio che supera le dimensioni massime configurate per i messaggi comporta un'eccezione. Se impostato su `null` , le dimensioni del messaggio sono illimitate. |
| MaxReceiveMessageSize | 4 MB | Dimensione massima in byte del messaggio che può essere ricevuto dal server. Se il server riceve un messaggio che supera questo limite, viene generata un'eccezione. L'aumento di questo valore consente al server di ricevere messaggi di dimensioni maggiori, ma può influire negativamente sull'utilizzo della memoria. Se impostato su `null` , le dimensioni del messaggio sono illimitate. |
| EnableDetailedErrors | `false` | Se `true` , i messaggi di eccezione dettagliati vengono restituiti ai client quando viene generata un'eccezione in un metodo del servizio. Il valore predefinito è `false`. L'impostazione `EnableDetailedErrors` di su `true` può infiltrare le informazioni riservate. |
| CompressionProviders | gzip | Raccolta di provider di compressione utilizzati per comprimere e decomprimere i messaggi. I provider di compressione personalizzati possono essere creati e aggiunti alla raccolta. I provider configurati predefiniti supportano la compressione **gzip** . |
| <span style="word-break:normal;word-wrap:normal">ResponseCompressionAlgorithm</span> | `null` | Algoritmo di compressione usato per comprimere i messaggi inviati dal server. L'algoritmo deve corrispondere a un provider di compressione in `CompressionProviders` . Per comprimere una risposta, il client deve indicare che supporta l'algoritmo inviando l'algoritmo nell'intestazione **grpc-Accept-Encoding** . |
| ResponseCompressionLevel | `null` | Livello di compressione utilizzato per comprimere i messaggi inviati dal server. |
| Intercettori | nessuno | Raccolta di intercettori eseguiti con ogni chiamata gRPC. Gli intercettori vengono eseguiti nell'ordine in cui sono registrati. Gli intercettori configurati a livello globale vengono eseguiti prima degli intercettori configurati per un singolo servizio. Per ulteriori informazioni sugli intercettori gRPC, vedere [GRPC Interceptors vs. middleware](xref:grpc/migration#grpc-interceptors-vs-middleware). |
| IgnoreUnknownServices | `false` | Se `true` , le chiamate a servizi e metodi sconosciuti non restituiscono uno stato non **implementato** e la richiesta passa al middleware registrato successivo in ASP.NET Core. |

È possibile configurare le opzioni per tutti i servizi fornendo le opzioni delegate alla `AddGrpc` chiamata in `Startup.ConfigureServices` :

[!code-csharp[](~/grpc/configuration/sample/GrcpService/Startup.cs?name=snippet)]

Le opzioni per un singolo servizio eseguono l'override delle opzioni globali fornite in `AddGrpc` e possono essere configurate utilizzando `AddServiceOptions<TService>` :

[!code-csharp[](~/grpc/configuration/sample/GrcpService/Startup2.cs?name=snippet)]

## <a name="configure-client-options"></a>Configurare le opzioni client

la configurazione del client gRPC è impostata su on `GrpcChannelOptions` . Nella tabella seguente vengono descritte le opzioni per la configurazione dei canali gRPC:

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| HttpHandler | Nuova istanza | Oggetto `HttpMessageHandler` utilizzato per eseguire chiamate gRPC. Un client può essere impostato per configurare un oggetto personalizzato `HttpClientHandler` o aggiungere ulteriori gestori alla pipeline HTTP per le chiamate gRPC. Se non `HttpMessageHandler` si specifica alcun valore, `HttpClientHandler` viene creata una nuova istanza per il canale con l'eliminazione automatica. |
| HttpClient | `null` | Oggetto `HttpClient` utilizzato per eseguire chiamate gRPC. Questa impostazione è un'alternativa a `HttpHandler` . |
| DisposeHttpClient | `false` | Se è impostato su `true` e `HttpMessageHandler` viene specificato un oggetto o `HttpClient` , l'oggetto `HttpHandler` o `HttpClient` , rispettivamente, viene eliminato quando viene `GrpcChannel` eliminato. |
| LoggerFactory | `null` | `LoggerFactory`Utilizzato dal client per registrare le informazioni sulle chiamate gRPC. Un' `LoggerFactory` istanza può essere risolta dall'inserimento delle dipendenze o creata utilizzando `LoggerFactory.Create` . Per esempi di configurazione della registrazione, vedere <xref:grpc/diagnostics#grpc-client-logging> . |
| MaxSendMessageSize | `null` | Dimensione massima in byte del messaggio che può essere inviato dal client. Il tentativo di inviare un messaggio che supera le dimensioni massime configurate per i messaggi comporta un'eccezione. Se impostato su `null` , le dimensioni del messaggio sono illimitate. |
| MaxReceiveMessageSize | 4 MB | Dimensione massima del messaggio, in byte, che può essere ricevuta dal client. Se il client riceve un messaggio che supera questo limite, viene generata un'eccezione. L'aumento di questo valore consente al client di ricevere messaggi di dimensioni maggiori, ma può influire negativamente sull'utilizzo della memoria. Se impostato su `null` , le dimensioni del messaggio sono illimitate. |
| Credenziali | `null` | Istanza di `ChannelCredentials`. Le credenziali vengono usate per aggiungere i metadati di autenticazione alle chiamate gRPC. |
| CompressionProviders | gzip | Raccolta di provider di compressione utilizzati per comprimere e decomprimere i messaggi. I provider di compressione personalizzati possono essere creati e aggiunti alla raccolta. I provider configurati predefiniti supportano la compressione **gzip** . |
| ThrowOperationCanceledOnCancellation | `false` | Se impostato su `true` , i client generano <xref:System.OperationCanceledException> quando una chiamata viene annullata o viene superata la scadenza. |
| MaxRetryAttempts | 5 | Numero massimo di tentativi. Questo valore limita i valori dei tentativi e di copertura specificati nella configurazione del servizio. L'impostazione di questo valore da solo non Abilita i tentativi. I tentativi sono abilitati nella configurazione del servizio, che può essere eseguita usando `ServiceConfig` . Un `null` valore rimuove il limite massimo di tentativi di ripetizione. Per ulteriori informazioni sui tentativi, vedere <xref:grpc/retries> . |
| MaxRetryBufferSize | 16 MB | Dimensioni massime del buffer, in byte, che possono essere utilizzate per archiviare i messaggi inviati durante il nuovo tentativo o la copertura delle chiamate. Se viene superato il limite del buffer, non vengono eseguiti altri tentativi e tutte le chiamate di copertura, ma ne viene annullata una. Questo limite viene applicato a tutte le chiamate effettuate tramite il canale. Un `null` valore rimuove il limite massimo delle dimensioni del buffer di ripetizione. |
| <span style="word-break:normal;word-wrap:normal">MaxRetryBufferPerCallSize</span> | 1 MB | Dimensioni massime del buffer, in byte, che possono essere utilizzate per archiviare i messaggi inviati durante il nuovo tentativo o la copertura delle chiamate. Se viene superato il limite del buffer, non vengono eseguiti altri tentativi e tutte le chiamate di copertura, ma ne viene annullata una. Questo limite viene applicato a una sola chiamata. Un `null` valore rimuove il limite massimo delle dimensioni del buffer per ogni chiamata. |
| ServiceConfig | `null` | Configurazione del servizio per un canale gRPC. Per configurare i [tentativi gRPC](xref:grpc/retries), è possibile usare una configurazione del servizio. |

Il codice seguente:

* Imposta la dimensione massima del messaggio di invio e ricezione sul canale.
* Crea un client.

[!code-csharp[](~/grpc/configuration/sample/Program.cs?name=snippet&highlight=3-8)]

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:grpc/aspnetcore>
* <xref:grpc/client>
* <xref:grpc/diagnostics>
* <xref:tutorials/grpc/grpc-start>
