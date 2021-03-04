---
title: Creare messaggi protobuf per le app .NET
author: jamesnk
description: Informazioni su come creare messaggi protobuf per le app .NET.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 02/12/2021
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
uid: grpc/protobuf
ms.openlocfilehash: 65a84c34e182b7a330d14e1f7ace17790ee4ed47
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110170"
---
# <a name="create-protobuf-messages-for-net-apps"></a><span data-ttu-id="4291d-103">Creare messaggi protobuf per le app .NET</span><span class="sxs-lookup"><span data-stu-id="4291d-103">Create Protobuf messages for .NET apps</span></span>

<span data-ttu-id="4291d-104">Di [James Newton-King](https://twitter.com/jamesnk) e [Mark Rendle](https://twitter.com/markrendle)</span><span class="sxs-lookup"><span data-stu-id="4291d-104">By [James Newton-King](https://twitter.com/jamesnk) and [Mark Rendle](https://twitter.com/markrendle)</span></span>

<span data-ttu-id="4291d-105">gRPC USA [protobuf](https://developers.google.com/protocol-buffers) come linguaggio di definizione dell'interfaccia (IDL).</span><span class="sxs-lookup"><span data-stu-id="4291d-105">gRPC uses [Protobuf](https://developers.google.com/protocol-buffers) as its Interface Definition Language (IDL).</span></span> <span data-ttu-id="4291d-106">Protobuf IDL è un formato indipendente dal linguaggio per la specifica dei messaggi inviati e ricevuti dai servizi di gRPC.</span><span class="sxs-lookup"><span data-stu-id="4291d-106">Protobuf IDL is a language neutral format for specifying the messages sent and received by gRPC services.</span></span> <span data-ttu-id="4291d-107">I messaggi protobuf sono definiti nei `.proto` file.</span><span class="sxs-lookup"><span data-stu-id="4291d-107">Protobuf messages are defined in `.proto` files.</span></span> <span data-ttu-id="4291d-108">Questo documento illustra come i concetti di protobuf vengono mappati a .NET.</span><span class="sxs-lookup"><span data-stu-id="4291d-108">This document explains how Protobuf concepts map to .NET.</span></span>

## <a name="protobuf-messages"></a><span data-ttu-id="4291d-109">Messaggi protobuf</span><span class="sxs-lookup"><span data-stu-id="4291d-109">Protobuf messages</span></span>

<span data-ttu-id="4291d-110">I messaggi sono l'oggetto principale per il trasferimento dei dati in protobuf.</span><span class="sxs-lookup"><span data-stu-id="4291d-110">Messages are the main data transfer object in Protobuf.</span></span> <span data-ttu-id="4291d-111">Sono concettualmente simili alle classi .NET.</span><span class="sxs-lookup"><span data-stu-id="4291d-111">They are conceptually similar to .NET classes.</span></span>

```protobuf
syntax = "proto3";

option csharp_namespace = "Contoso.Messages";

message Person {
    int32 id = 1;
    string first_name = 2;
    string last_name = 3;
}  
```

<span data-ttu-id="4291d-112">La definizione di messaggio precedente specifica tre campi come coppie nome-valore.</span><span class="sxs-lookup"><span data-stu-id="4291d-112">The preceding message definition specifies three fields as name-value pairs.</span></span> <span data-ttu-id="4291d-113">Analogamente alle proprietà sui tipi .NET, ogni campo dispone di un nome e di un tipo.</span><span class="sxs-lookup"><span data-stu-id="4291d-113">Like properties on .NET types, each field has a name and a type.</span></span> <span data-ttu-id="4291d-114">Il tipo di campo può essere un [tipo di valore scalare protobuf](#scalar-value-types), ad esempio `int32` o un altro messaggio.</span><span class="sxs-lookup"><span data-stu-id="4291d-114">The field type can be a [Protobuf scalar value type](#scalar-value-types), e.g. `int32`, or another message.</span></span>

<span data-ttu-id="4291d-115">La [Guida di stile protobuf](https://developers.google.com/protocol-buffers/docs/style) consiglia `underscore_separated_names` di usare per i nomi di campo.</span><span class="sxs-lookup"><span data-stu-id="4291d-115">The [Protobuf style guide](https://developers.google.com/protocol-buffers/docs/style) recommends using `underscore_separated_names` for field names.</span></span> <span data-ttu-id="4291d-116">I nuovi messaggi protobuf creati per le app .NET devono seguire le linee guida di stile protobuf.</span><span class="sxs-lookup"><span data-stu-id="4291d-116">New Protobuf messages created for .NET apps should follow the Protobuf style guidelines.</span></span> <span data-ttu-id="4291d-117">Gli strumenti .NET generano automaticamente tipi .NET che utilizzano standard di denominazione .NET.</span><span class="sxs-lookup"><span data-stu-id="4291d-117">.NET tooling automatically generates .NET types that use .NET naming standards.</span></span> <span data-ttu-id="4291d-118">Un campo protobuf, ad esempio, `first_name` genera una `FirstName` Proprietà .NET.</span><span class="sxs-lookup"><span data-stu-id="4291d-118">For example, a `first_name` Protobuf field generates a `FirstName` .NET property.</span></span>

<span data-ttu-id="4291d-119">Oltre a un nome, ogni campo nella definizione del messaggio ha un numero univoco.</span><span class="sxs-lookup"><span data-stu-id="4291d-119">In addition to a name, each field in the message definition has a unique number.</span></span> <span data-ttu-id="4291d-120">I numeri di campo vengono utilizzati per identificare i campi quando il messaggio viene serializzato in protobuf.</span><span class="sxs-lookup"><span data-stu-id="4291d-120">Field numbers are used to identify fields when the message is serialized to Protobuf.</span></span> <span data-ttu-id="4291d-121">La serializzazione di un numero ridotto è più veloce rispetto alla serializzazione dell'intero nome di campo.</span><span class="sxs-lookup"><span data-stu-id="4291d-121">Serializing a small number is faster than serializing the entire field name.</span></span> <span data-ttu-id="4291d-122">Poiché i numeri di campo identificano un campo, è importante prestare attenzione durante la modifica.</span><span class="sxs-lookup"><span data-stu-id="4291d-122">Because field numbers identify a field it is important to take care when changing them.</span></span> <span data-ttu-id="4291d-123">Per ulteriori informazioni sulla modifica dei messaggi protobuf <xref:grpc/versioning> , vedere.</span><span class="sxs-lookup"><span data-stu-id="4291d-123">For more information about changing Protobuf messages see <xref:grpc/versioning>.</span></span>

<span data-ttu-id="4291d-124">Quando viene compilata un'app, gli strumenti protobuf generano tipi .NET dai `.proto` file.</span><span class="sxs-lookup"><span data-stu-id="4291d-124">When an app is built the Protobuf tooling generates .NET types from `.proto` files.</span></span> <span data-ttu-id="4291d-125">Il `Person` messaggio genera una classe .NET:</span><span class="sxs-lookup"><span data-stu-id="4291d-125">The `Person` message generates a .NET class:</span></span>

```csharp
public class Person
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

<span data-ttu-id="4291d-126">Per ulteriori informazioni sui messaggi protobuf, vedere la [Guida alla lingua di protobuf](https://developers.google.com/protocol-buffers/docs/proto3#simple).</span><span class="sxs-lookup"><span data-stu-id="4291d-126">For more information about Protobuf messages see the [Protobuf language guide](https://developers.google.com/protocol-buffers/docs/proto3#simple).</span></span>

## <a name="scalar-value-types"></a><span data-ttu-id="4291d-127">Tipi di valore scalari</span><span class="sxs-lookup"><span data-stu-id="4291d-127">Scalar Value Types</span></span>

<span data-ttu-id="4291d-128">Protobuf supporta un intervallo di tipi valore scalari nativi.</span><span class="sxs-lookup"><span data-stu-id="4291d-128">Protobuf supports a range of native scalar value types.</span></span> <span data-ttu-id="4291d-129">La tabella seguente elenca tutti i tipi con il tipo C# equivalente:</span><span class="sxs-lookup"><span data-stu-id="4291d-129">The following table lists them all with their equivalent C# type:</span></span>

| <span data-ttu-id="4291d-130">Tipo protobuf</span><span class="sxs-lookup"><span data-stu-id="4291d-130">Protobuf type</span></span> | <span data-ttu-id="4291d-131">Tipo C#</span><span class="sxs-lookup"><span data-stu-id="4291d-131">C# type</span></span>      |
| ------------- | ------------ |
| `double`      | `double`     |
| `float`       | `float`      |
| `int32`       | `int`        |
| `int64`       | `long`       |
| `uint32`      | `uint`       |
| `uint64`      | `ulong`      |
| `sint32`      | `int`        |
| `sint64`      | `long`       |
| `fixed32`     | `uint`       |
| `fixed64`     | `ulong`      |
| `sfixed32`    | `int`        |
| `sfixed64`    | `long`       |
| `bool`        | `bool`       |
| `string`      | `string`     |
| `bytes`       | `ByteString` |

<span data-ttu-id="4291d-132">I valori scalari hanno sempre un valore predefinito e non possono essere impostati su `null` .</span><span class="sxs-lookup"><span data-stu-id="4291d-132">Scalar values always have a default value and can't be set to `null`.</span></span> <span data-ttu-id="4291d-133">Questo vincolo include `string` e `ByteString` che sono classi C#.</span><span class="sxs-lookup"><span data-stu-id="4291d-133">This constraint includes `string` and `ByteString` which are C# classes.</span></span> <span data-ttu-id="4291d-134">`string` il valore predefinito è un valore stringa vuoto e il valore `ByteString` predefinito è un valore byte vuoti.</span><span class="sxs-lookup"><span data-stu-id="4291d-134">`string` defaults to an empty string value and `ByteString` defaults to an empty bytes value.</span></span> <span data-ttu-id="4291d-135">Il tentativo di impostarli su `null` genera un errore.</span><span class="sxs-lookup"><span data-stu-id="4291d-135">Attempting to set them to `null` throws an error.</span></span>

<span data-ttu-id="4291d-136">I [tipi di wrapper Nullable](#nullable-types) possono essere utilizzati per supportare i valori null.</span><span class="sxs-lookup"><span data-stu-id="4291d-136">[Nullable wrapper types](#nullable-types) can be used to support null values.</span></span>

### <a name="dates-and-times"></a><span data-ttu-id="4291d-137">Date e ore</span><span class="sxs-lookup"><span data-stu-id="4291d-137">Dates and times</span></span>

<span data-ttu-id="4291d-138">I tipi scalari nativi non forniscono i valori di data e ora, equivalenti a. NET <xref:System.DateTimeOffset> , <xref:System.DateTime> e <xref:System.TimeSpan> .</span><span class="sxs-lookup"><span data-stu-id="4291d-138">The native scalar types don't provide for date and time values, equivalent to .NET's <xref:System.DateTimeOffset>, <xref:System.DateTime>, and <xref:System.TimeSpan>.</span></span> <span data-ttu-id="4291d-139">Questi tipi possono essere specificati usando alcune estensioni dei *tipi noti* di protobuf.</span><span class="sxs-lookup"><span data-stu-id="4291d-139">These types can be specified by using some of Protobuf's *Well-Known Types* extensions.</span></span> <span data-ttu-id="4291d-140">Queste estensioni forniscono il supporto per la generazione di codice e il runtime per i tipi di campo complessi nelle piattaforme supportate.</span><span class="sxs-lookup"><span data-stu-id="4291d-140">These extensions provide code generation and runtime support for complex field types across the supported platforms.</span></span>

<span data-ttu-id="4291d-141">La tabella seguente illustra i tipi di data e ora:</span><span class="sxs-lookup"><span data-stu-id="4291d-141">The following table shows the date and time types:</span></span>

| <span data-ttu-id="4291d-142">Tipo .NET</span><span class="sxs-lookup"><span data-stu-id="4291d-142">.NET type</span></span>        | <span data-ttu-id="4291d-143">Tipo Well-Known protobuf</span><span class="sxs-lookup"><span data-stu-id="4291d-143">Protobuf Well-Known Type</span></span>    |
| ---------------- | --------------------------- |
| `DateTimeOffset` | `google.protobuf.Timestamp` |
| `DateTime`       | `google.protobuf.Timestamp` |
| `TimeSpan`       | `google.protobuf.Duration`  |

```protobuf  
syntax = "proto3"

import "google/protobuf/duration.proto";  
import "google/protobuf/timestamp.proto";

message Meeting {
    string subject = 1;
    google.protobuf.Timestamp start = 2;
    google.protobuf.Duration duration = 3;
}  
```

<span data-ttu-id="4291d-144">Le proprietà generate nella classe C# non sono i tipi di data e ora .NET.</span><span class="sxs-lookup"><span data-stu-id="4291d-144">The generated properties in the C# class aren't the .NET date and time types.</span></span> <span data-ttu-id="4291d-145">Le proprietà utilizzano le `Timestamp` `Duration` classi e nello `Google.Protobuf.WellKnownTypes` spazio dei nomi.</span><span class="sxs-lookup"><span data-stu-id="4291d-145">The properties use the `Timestamp` and `Duration` classes in the `Google.Protobuf.WellKnownTypes` namespace.</span></span> <span data-ttu-id="4291d-146">Queste classi forniscono metodi per la conversione da e verso `DateTimeOffset` , `DateTime` e `TimeSpan` .</span><span class="sxs-lookup"><span data-stu-id="4291d-146">These classes provide methods for converting to and from `DateTimeOffset`, `DateTime`, and `TimeSpan`.</span></span>

```csharp
// Create Timestamp and Duration from .NET DateTimeOffset and TimeSpan.
var meeting = new Meeting
{
    Time = Timestamp.FromDateTimeOffset(meetingTime), // also FromDateTime()
    Duration = Duration.FromTimeSpan(meetingLength)
};

// Convert Timestamp and Duration to .NET DateTimeOffset and TimeSpan.
var time = meeting.Time.ToDateTimeOffset();
var duration = meeting.Duration?.ToTimeSpan();
```

> [!NOTE]
> <span data-ttu-id="4291d-147">Il `Timestamp` tipo funziona con l'ora UTC.</span><span class="sxs-lookup"><span data-stu-id="4291d-147">The `Timestamp` type works with UTC times.</span></span> <span data-ttu-id="4291d-148">`DateTimeOffset` i valori hanno sempre un offset pari a zero e la `DateTime.Kind` proprietà è sempre `DateTimeKind.Utc` .</span><span class="sxs-lookup"><span data-stu-id="4291d-148">`DateTimeOffset` values always have an offset of zero, and the `DateTime.Kind` property is always `DateTimeKind.Utc`.</span></span>

### <a name="nullable-types"></a><span data-ttu-id="4291d-149">Tipi nullable</span><span class="sxs-lookup"><span data-stu-id="4291d-149">Nullable types</span></span>

<span data-ttu-id="4291d-150">La generazione di codice protobuf per C# usa i tipi nativi, ad esempio `int` per `int32` .</span><span class="sxs-lookup"><span data-stu-id="4291d-150">The Protobuf code generation for C# uses the native types, such as `int` for `int32`.</span></span> <span data-ttu-id="4291d-151">Pertanto, i valori vengono sempre inclusi e non possono essere `null` .</span><span class="sxs-lookup"><span data-stu-id="4291d-151">So the values are always included and can't be `null`.</span></span>

<span data-ttu-id="4291d-152">Per i valori che richiedono esplicito `null` , ad esempio l'uso `int?` di nel codice c#, i tipi di Well-Known di protobuf includono wrapper compilati in tipi c# Nullable.</span><span class="sxs-lookup"><span data-stu-id="4291d-152">For values that require explicit `null`, such as using `int?` in C# code, Protobuf's Well-Known Types include wrappers that are compiled to nullable C# types.</span></span> <span data-ttu-id="4291d-153">Per usarli, importare `wrappers.proto` nel `.proto` file, come nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="4291d-153">To use them, import `wrappers.proto` into your `.proto` file, like the following code:</span></span>

```protobuf  
syntax = "proto3"

import "google/protobuf/wrappers.proto"

message Person {
    // ...
    google.protobuf.Int32Value age = 5;
}
```

<span data-ttu-id="4291d-154">`wrappers.proto` i tipi non sono esposti nelle proprietà generate.</span><span class="sxs-lookup"><span data-stu-id="4291d-154">`wrappers.proto` types aren't exposed in generated properties.</span></span> <span data-ttu-id="4291d-155">Protobuf esegue automaticamente il mapping ai tipi nullable .NET appropriati nei messaggi C#.</span><span class="sxs-lookup"><span data-stu-id="4291d-155">Protobuf automatically maps them to appropriate .NET nullable types in C# messages.</span></span> <span data-ttu-id="4291d-156">Un campo, ad esempio, `google.protobuf.Int32Value` genera una `int?` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="4291d-156">For example, a `google.protobuf.Int32Value` field generates an `int?` property.</span></span> <span data-ttu-id="4291d-157">Le proprietà del tipo `string` di riferimento, ad esempio e, `ByteString` sono invariate tranne `null` che possono essere assegnate senza errori.</span><span class="sxs-lookup"><span data-stu-id="4291d-157">Reference type properties like `string` and `ByteString` are unchanged except `null` can be assigned to them without error.</span></span>

<span data-ttu-id="4291d-158">La tabella seguente mostra l'elenco completo dei tipi di wrapper con il tipo C# equivalente:</span><span class="sxs-lookup"><span data-stu-id="4291d-158">The following table shows the complete list of wrapper types with their equivalent C# type:</span></span>

| <span data-ttu-id="4291d-159">Tipo C#</span><span class="sxs-lookup"><span data-stu-id="4291d-159">C# type</span></span>      | <span data-ttu-id="4291d-160">Wrapper di tipo Well-Known</span><span class="sxs-lookup"><span data-stu-id="4291d-160">Well-Known Type wrapper</span></span>       |
| ------------ | ----------------------------- |
| `bool?`      | `google.protobuf.BoolValue`   |
| `double?`    | `google.protobuf.DoubleValue` |
| `float?`     | `google.protobuf.FloatValue`  |
| `int?`       | `google.protobuf.Int32Value`  |
| `long?`      | `google.protobuf.Int64Value`  |
| `uint?`      | `google.protobuf.UInt32Value` |
| `ulong?`     | `google.protobuf.UInt64Value` |
| `string`     | `google.protobuf.StringValue` |
| `ByteString` | `google.protobuf.BytesValue`  |

### <a name="bytes"></a><span data-ttu-id="4291d-161">Byte</span><span class="sxs-lookup"><span data-stu-id="4291d-161">Bytes</span></span>

<span data-ttu-id="4291d-162">I payload binari sono supportati in protobuf con il `bytes` tipo di valore scalare.</span><span class="sxs-lookup"><span data-stu-id="4291d-162">Binary payloads are supported in Protobuf with the `bytes` scalar value type.</span></span> <span data-ttu-id="4291d-163">Una proprietà generata in C# usa `ByteString` come tipo di proprietà.</span><span class="sxs-lookup"><span data-stu-id="4291d-163">A generated property in C# uses `ByteString` as the property type.</span></span>

<span data-ttu-id="4291d-164">Utilizzare `ByteString.CopyFrom(byte[] data)` per creare una nuova istanza da una matrice di byte:</span><span class="sxs-lookup"><span data-stu-id="4291d-164">Use `ByteString.CopyFrom(byte[] data)` to create a new instance from a byte array:</span></span>

```csharp
var data = await File.ReadAllBytesAsync(path);

var payload = new PayloadResponse();
payload.Data = ByteString.CopyFrom(data);
```

<span data-ttu-id="4291d-165">`ByteString` è possibile accedere ai dati direttamente usando `ByteString.Span` o `ByteString.Memory` .</span><span class="sxs-lookup"><span data-stu-id="4291d-165">`ByteString` data is accessed directly using `ByteString.Span` or `ByteString.Memory`.</span></span> <span data-ttu-id="4291d-166">In alternativa `ByteString.ToByteArray()` , chiamare per convertire nuovamente un'istanza in una matrice di byte:</span><span class="sxs-lookup"><span data-stu-id="4291d-166">Or call `ByteString.ToByteArray()` to convert an instance back into a byte array:</span></span>

```csharp
var payload = await client.GetPayload(new PayloadRequest());

await File.WriteAllBytesAsync(path, payload.Data.ToByteArray());
```

### <a name="decimals"></a><span data-ttu-id="4291d-167">Decimali</span><span class="sxs-lookup"><span data-stu-id="4291d-167">Decimals</span></span>

<span data-ttu-id="4291d-168">Protobuf non supporta in modo nativo il `decimal` tipo .NET, solo `double` e `float` .</span><span class="sxs-lookup"><span data-stu-id="4291d-168">Protobuf doesn't natively support the .NET `decimal` type, just `double` and `float`.</span></span> <span data-ttu-id="4291d-169">Nel progetto protobuf è presente una discussione continuativa sulla possibilità di aggiungere un tipo decimale standard ai tipi di Well-Known, con supporto della piattaforma per linguaggi e Framework che lo supportano.</span><span class="sxs-lookup"><span data-stu-id="4291d-169">There's an ongoing discussion in the Protobuf project about the possibility of adding a standard decimal type to the Well-Known Types, with platform support for languages and frameworks that support it.</span></span> <span data-ttu-id="4291d-170">Non è stato ancora implementato alcun elemento.</span><span class="sxs-lookup"><span data-stu-id="4291d-170">Nothing has been implemented yet.</span></span>

<span data-ttu-id="4291d-171">È possibile creare una definizione di messaggio per rappresentare il `decimal` tipo che funziona per la serializzazione sicura tra client e server .NET.</span><span class="sxs-lookup"><span data-stu-id="4291d-171">It's possible to create a message definition to represent the `decimal` type that works for safe serialization between .NET clients and servers.</span></span> <span data-ttu-id="4291d-172">Gli sviluppatori su altre piattaforme dovrebbero tuttavia comprendere il formato usato e implementare la propria gestione per l'it.</span><span class="sxs-lookup"><span data-stu-id="4291d-172">But developers on other platforms would have to understand the format being used and implement their own handling for it.</span></span>

#### <a name="creating-a-custom-decimal-type-for-protobuf"></a><span data-ttu-id="4291d-173">Creazione di un tipo decimale personalizzato per protobuf</span><span class="sxs-lookup"><span data-stu-id="4291d-173">Creating a custom decimal type for Protobuf</span></span>

```protobuf
package CustomTypes;

// Example: 12345.6789 -> { units = 12345, nanos = 678900000 }
message DecimalValue {

    // Whole units part of the amount
    int64 units = 1;

    // Nano units of the amount (10^-9)
    // Must be same sign as units
    sfixed32 nanos = 2;
}
```

<span data-ttu-id="4291d-174">Il `nanos` campo rappresenta i valori da `0.999_999_999` a `-0.999_999_999` .</span><span class="sxs-lookup"><span data-stu-id="4291d-174">The `nanos` field represents values from `0.999_999_999` to `-0.999_999_999`.</span></span> <span data-ttu-id="4291d-175">Il valore, ad esempio, viene `decimal` `1.5m` rappresentato come `{ units = 1, nanos = 500_000_000 }` .</span><span class="sxs-lookup"><span data-stu-id="4291d-175">For example, the `decimal` value `1.5m` would be represented as `{ units = 1, nanos = 500_000_000 }`.</span></span> <span data-ttu-id="4291d-176">Questo è il motivo `nanos` per cui il campo in questo esempio usa il `sfixed32` tipo, che codifica in modo più efficiente rispetto `int32` ai valori più grandi.</span><span class="sxs-lookup"><span data-stu-id="4291d-176">This is why the `nanos` field in this example uses the `sfixed32` type, which encodes more efficiently than `int32` for larger values.</span></span> <span data-ttu-id="4291d-177">Se il `units` campo è negativo, `nanos` anche il campo dovrebbe essere negativo.</span><span class="sxs-lookup"><span data-stu-id="4291d-177">If the `units` field is negative, the `nanos` field should also be negative.</span></span>

> [!NOTE]
> <span data-ttu-id="4291d-178">Sono disponibili più algoritmi per la codifica `decimal` dei valori come stringhe di byte, ma questo messaggio è più facile da comprendere.</span><span class="sxs-lookup"><span data-stu-id="4291d-178">There are multiple other algorithms for encoding `decimal` values as byte strings, but this message is easier to understand than any of them.</span></span> <span data-ttu-id="4291d-179">I valori non sono interessati da Big-endian o little-endian su piattaforme diverse.</span><span class="sxs-lookup"><span data-stu-id="4291d-179">The values are not affected by big-endian or little-endian on different platforms.</span></span>

<span data-ttu-id="4291d-180">La conversione tra questo tipo e il `decimal` tipo BCL potrebbe essere implementata in C# come segue:</span><span class="sxs-lookup"><span data-stu-id="4291d-180">Conversion between this type and the BCL `decimal` type might be implemented in C# like this:</span></span>

```csharp
namespace CustomTypes
{
    public partial class DecimalValue
    {
        private const decimal NanoFactor = 1_000_000_000;
        public DecimalValue(long units, int nanos)
        {
            Units = units;
            Nanos = nanos;
        }

        public long Units { get; }
        public int Nanos { get; }

        public static implicit operator decimal(CustomTypes.DecimalValue grpcDecimal)
        {
            return grpcDecimal.Units + grpcDecimal.Nanos / NanoFactor;
        }

        public static implicit operator CustomTypes.DecimalValue(decimal value)
        {
            var units = decimal.ToInt64(value);
            var nanos = decimal.ToInt32((value - units) * NanoFactor);
            return new CustomTypes.DecimalValue(units, nanos);
        }
    }
}
```

## <a name="collections"></a><span data-ttu-id="4291d-181">Raccolte</span><span class="sxs-lookup"><span data-stu-id="4291d-181">Collections</span></span>

### <a name="lists"></a><span data-ttu-id="4291d-182">Elenchi</span><span class="sxs-lookup"><span data-stu-id="4291d-182">Lists</span></span>

<span data-ttu-id="4291d-183">Gli elenchi in protobuf vengono specificati utilizzando la `repeated` parola chiave prefix in un campo.</span><span class="sxs-lookup"><span data-stu-id="4291d-183">Lists in Protobuf are specified by using the `repeated` prefix keyword on a field.</span></span> <span data-ttu-id="4291d-184">Nell'esempio seguente viene illustrato come creare un elenco:</span><span class="sxs-lookup"><span data-stu-id="4291d-184">The following example shows how to create a list:</span></span>

```protobuf
message Person {
    // ...
    repeated string roles = 8;
}
```

<span data-ttu-id="4291d-185">Nel codice generato, `repeated` i campi sono rappresentati dal `Google.Protobuf.Collections.RepeatedField<T>` tipo generico.</span><span class="sxs-lookup"><span data-stu-id="4291d-185">In the generated code, `repeated` fields are represented by the `Google.Protobuf.Collections.RepeatedField<T>` generic type.</span></span>

```csharp
public class Person
{
    // ...
    public RepeatedField<string> Roles { get; }
}
```

<span data-ttu-id="4291d-186">`RepeatedField<T>` implementa <xref:System.Collections.Generic.IList%601>.</span><span class="sxs-lookup"><span data-stu-id="4291d-186">`RepeatedField<T>` implements <xref:System.Collections.Generic.IList%601>.</span></span> <span data-ttu-id="4291d-187">Quindi, è possibile usare le query LINQ o convertirle in una matrice o in un elenco.</span><span class="sxs-lookup"><span data-stu-id="4291d-187">So you can use LINQ queries or convert it to an array or a list.</span></span> <span data-ttu-id="4291d-188">`RepeatedField<T>` le proprietà non hanno un setter pubblico.</span><span class="sxs-lookup"><span data-stu-id="4291d-188">`RepeatedField<T>` properties don't have a public setter.</span></span> <span data-ttu-id="4291d-189">Gli elementi devono essere aggiunti alla raccolta esistente.</span><span class="sxs-lookup"><span data-stu-id="4291d-189">Items should be added to the existing collection.</span></span>

```csharp
var person = new Person();

// Add one item.
person.Roles.Add("user");

// Add all items from another collection.
var roles = new [] { "admin", "manager" };
person.Roles.Add(roles);
```

### <a name="dictionaries"></a><span data-ttu-id="4291d-190">Dizionari</span><span class="sxs-lookup"><span data-stu-id="4291d-190">Dictionaries</span></span>

<span data-ttu-id="4291d-191">Il <xref:System.Collections.Generic.IDictionary%602> tipo .NET è rappresentato in protobuf usando `map<key_type, value_type>` .</span><span class="sxs-lookup"><span data-stu-id="4291d-191">The .NET <xref:System.Collections.Generic.IDictionary%602> type is represented in Protobuf using `map<key_type, value_type>`.</span></span>

```protobuf
message Person {
    // ...
    map<string, string> attributes = 9;
}
```

<span data-ttu-id="4291d-192">Nel codice .NET generato, `map` i campi sono rappresentati dal `Google.Protobuf.Collections.MapField<TKey, TValue>` tipo generico.</span><span class="sxs-lookup"><span data-stu-id="4291d-192">In generated .NET code, `map` fields are represented by the `Google.Protobuf.Collections.MapField<TKey, TValue>` generic type.</span></span> <span data-ttu-id="4291d-193">`MapField<TKey, TValue>` implementa <xref:System.Collections.Generic.IDictionary%602>.</span><span class="sxs-lookup"><span data-stu-id="4291d-193">`MapField<TKey, TValue>` implements <xref:System.Collections.Generic.IDictionary%602>.</span></span> <span data-ttu-id="4291d-194">Analogamente alle `repeated` proprietà, le `map` proprietà non hanno un setter pubblico.</span><span class="sxs-lookup"><span data-stu-id="4291d-194">Like `repeated` properties, `map` properties don't have a public setter.</span></span> <span data-ttu-id="4291d-195">Gli elementi devono essere aggiunti alla raccolta esistente.</span><span class="sxs-lookup"><span data-stu-id="4291d-195">Items should be added to the existing collection.</span></span>

```csharp
var person = new Person();

// Add one item.
person.Attributes["created_by"] = "James";

// Add all items from another collection.
var attributes = new Dictionary<string, string>
{
    ["last_modified"] = DateTime.UtcNow.ToString()
};
person.Attributes.Add(attributes);
```

## <a name="unstructured-and-conditional-messages"></a><span data-ttu-id="4291d-196">Messaggi non strutturati e condizionali</span><span class="sxs-lookup"><span data-stu-id="4291d-196">Unstructured and conditional messages</span></span>

<span data-ttu-id="4291d-197">Protobuf è un formato di messaggistica prima del contratto.</span><span class="sxs-lookup"><span data-stu-id="4291d-197">Protobuf is a contract-first messaging format.</span></span> <span data-ttu-id="4291d-198">I messaggi di un'app, inclusi i campi e i tipi, devono essere specificati nei `.proto` file quando viene compilata l'app.</span><span class="sxs-lookup"><span data-stu-id="4291d-198">An app's messages, including its fields and types, must be specified in `.proto` files when the app is built.</span></span> <span data-ttu-id="4291d-199">La progettazione dei contratti di protobuf è ideale per applicare il contenuto dei messaggi, ma può limitare gli scenari in cui non è necessario un contratto rigido:</span><span class="sxs-lookup"><span data-stu-id="4291d-199">Protobuf's contract-first design is great at enforcing message content but can limit scenarios where a strict contract isn't required:</span></span>

* <span data-ttu-id="4291d-200">Messaggi con payload sconosciuti.</span><span class="sxs-lookup"><span data-stu-id="4291d-200">Messages with unknown payloads.</span></span> <span data-ttu-id="4291d-201">Ad esempio, un messaggio con un campo che può contenere qualsiasi messaggio.</span><span class="sxs-lookup"><span data-stu-id="4291d-201">For example, a message with a field that could contain any message.</span></span>
* <span data-ttu-id="4291d-202">Messaggi condizionali.</span><span class="sxs-lookup"><span data-stu-id="4291d-202">Conditional messages.</span></span> <span data-ttu-id="4291d-203">Un messaggio restituito da un servizio gRPC, ad esempio, potrebbe essere un risultato positivo o un errore.</span><span class="sxs-lookup"><span data-stu-id="4291d-203">For example, a message returned from a gRPC service might be a success result or an error result.</span></span>
* <span data-ttu-id="4291d-204">Valori dinamici.</span><span class="sxs-lookup"><span data-stu-id="4291d-204">Dynamic values.</span></span> <span data-ttu-id="4291d-205">Ad esempio, un messaggio con un campo che contiene una raccolta non strutturata di valori, simile a JSON.</span><span class="sxs-lookup"><span data-stu-id="4291d-205">For example, a message with a field that contains an unstructured collection of values, similar to JSON.</span></span>

<span data-ttu-id="4291d-206">Protobuf offre funzionalità del linguaggio e tipi per supportare questi scenari.</span><span class="sxs-lookup"><span data-stu-id="4291d-206">Protobuf offers language features and types to support these scenarios.</span></span>

### <a name="any"></a><span data-ttu-id="4291d-207">Qualsiasi</span><span class="sxs-lookup"><span data-stu-id="4291d-207">Any</span></span>

<span data-ttu-id="4291d-208">Il `Any` tipo consente di usare i messaggi come tipi incorporati senza avere la relativa `.proto` definizione.</span><span class="sxs-lookup"><span data-stu-id="4291d-208">The `Any` type lets you use messages as embedded types without having their `.proto` definition.</span></span> <span data-ttu-id="4291d-209">Per utilizzare il `Any` tipo, Import `any.proto` .</span><span class="sxs-lookup"><span data-stu-id="4291d-209">To use the `Any` type, import `any.proto`.</span></span>

```protobuf
import "google/protobuf/any.proto";

message Status {
    string message = 1;
    google.protobuf.Any detail = 2;
}
```

```csharp
// Create a status with a Person message set to detail.
var status = new ErrorStatus();
status.Detail = Any.Pack(new Person { FirstName = "James" });

// Read Person message from detail.
if (status.Detail.Is(Person.Descriptor))
{
    var person = status.Detail.Unpack<Person>();
    // ...
}
```

### <a name="oneof"></a><span data-ttu-id="4291d-210">Di</span><span class="sxs-lookup"><span data-stu-id="4291d-210">Oneof</span></span>

<span data-ttu-id="4291d-211">`oneof` i campi sono una funzionalità del linguaggio.</span><span class="sxs-lookup"><span data-stu-id="4291d-211">`oneof` fields are a language feature.</span></span> <span data-ttu-id="4291d-212">Il compilatore gestisce la `oneof` parola chiave quando genera la classe Message.</span><span class="sxs-lookup"><span data-stu-id="4291d-212">The compiler handles the `oneof` keyword when it generates the message class.</span></span> <span data-ttu-id="4291d-213">`oneof`L'utilizzo di per specificare un messaggio di risposta che può restituire `Person` o potrebbe essere simile al `Error` seguente:</span><span class="sxs-lookup"><span data-stu-id="4291d-213">Using `oneof` to specify a response message that could either return a `Person` or `Error` might look like this:</span></span>

```protobuf
message Person {
    // ...
}

message Error {
    // ...
}

message ResponseMessage {
  oneof result {
    Error error = 1;
    Person person = 2;
  }
}
```

<span data-ttu-id="4291d-214">I campi all'interno del `oneof` set devono avere numeri di campo univoci nella dichiarazione generale del messaggio.</span><span class="sxs-lookup"><span data-stu-id="4291d-214">Fields within the `oneof` set must have unique field numbers in the overall message declaration.</span></span>

<span data-ttu-id="4291d-215">Quando `oneof` si usa, il codice C# generato include un'enumerazione che specifica quale dei campi è stata impostata.</span><span class="sxs-lookup"><span data-stu-id="4291d-215">When using `oneof`, the generated C# code includes an enum that specifies which of the fields has been set.</span></span> <span data-ttu-id="4291d-216">È possibile testare l'enumerazione per individuare il campo da impostare.</span><span class="sxs-lookup"><span data-stu-id="4291d-216">You can test the enum to find which field is set.</span></span> <span data-ttu-id="4291d-217">I campi che non sono impostati restituiscono `null` o il valore predefinito, anziché generare un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="4291d-217">Fields that aren't set return `null` or the default value, rather than throwing an exception.</span></span>

```csharp
var response = await client.GetPersonAsync(new RequestMessage());

switch (response.ResultCase)
{
    case ResponseMessage.ResultOneofCase.Person:
        HandlePerson(response.Person);
        break;
    case ResponseMessage.ResultOneofCase.Error:
        HandleError(response.Error);
        break;
    default:
        throw new ArgumentException("Unexpected result.");
}
```

### <a name="value"></a><span data-ttu-id="4291d-218">Valore</span><span class="sxs-lookup"><span data-stu-id="4291d-218">Value</span></span>

<span data-ttu-id="4291d-219">Il `Value` tipo rappresenta un valore tipizzato dinamicamente.</span><span class="sxs-lookup"><span data-stu-id="4291d-219">The `Value` type represents a dynamically typed value.</span></span> <span data-ttu-id="4291d-220">Può essere `null` , un numero, una stringa, un valore booleano, un dizionario di valori ( `Struct` ) o un elenco di valori ( `ValueList` ).</span><span class="sxs-lookup"><span data-stu-id="4291d-220">It can be either `null`, a number, a string, a boolean, a dictionary of values (`Struct`), or a list of values (`ValueList`).</span></span> <span data-ttu-id="4291d-221">`Value` è un tipo di Well-Known protobuf che utilizza la funzionalità descritta in precedenza `oneof` .</span><span class="sxs-lookup"><span data-stu-id="4291d-221">`Value` is a Protobuf Well-Known Type that uses the previously discussed `oneof` feature.</span></span> <span data-ttu-id="4291d-222">Per utilizzare il `Value` tipo, Import `struct.proto` .</span><span class="sxs-lookup"><span data-stu-id="4291d-222">To use the `Value` type, import `struct.proto`.</span></span>

```protobuf
import "google/protobuf/struct.proto";

message Status {
    // ...
    google.protobuf.Value data = 3;
}
```

```csharp
// Create dynamic values.
var status = new Status();
status.Data = Value.FromStruct(new Struct
{
    Fields =
    {
        ["enabled"] = Value.ForBoolean(true),
        ["metadata"] = Value.ForList(
            Value.FromString("value1"),
            Value.FromString("value2"))
    }
});

// Read dynamic values.
switch (status.Data.KindCase)
{
    case Value.KindOneofCase.StructValue:
        foreach (var field in status.Data.StructValue.Fields)
        {
            // Read struct fields...
        }
        break;
    // ...
}
```

<span data-ttu-id="4291d-223">L'uso `Value` diretto può essere dettagliato.</span><span class="sxs-lookup"><span data-stu-id="4291d-223">Using `Value` directly can be verbose.</span></span> <span data-ttu-id="4291d-224">Un modo alternativo per usare `Value` è il supporto incorporato di protobuf per il mapping dei messaggi a JSON.</span><span class="sxs-lookup"><span data-stu-id="4291d-224">An alternative way to use `Value` is with Protobuf's built-in support for mapping messages to JSON.</span></span> <span data-ttu-id="4291d-225">`JsonFormatter`I tipi e di protobuf `JsonWriter` possono essere usati con qualsiasi messaggio protobuf.</span><span class="sxs-lookup"><span data-stu-id="4291d-225">Protobuf's `JsonFormatter` and `JsonWriter` types can be used with any Protobuf message.</span></span> <span data-ttu-id="4291d-226">`Value` è particolarmente adatto per essere convertito in e da JSON.</span><span class="sxs-lookup"><span data-stu-id="4291d-226">`Value` is particularly well suited to being converted to and from JSON.</span></span>

<span data-ttu-id="4291d-227">Si tratta dell'equivalente JSON del codice precedente:</span><span class="sxs-lookup"><span data-stu-id="4291d-227">This is the JSON equivalent of the previous code:</span></span>

```csharp
// Create dynamic values from JSON.
var status = new Status();
status.Data = Value.Parser.ParseJson(@"{
    ""enabled"": true,
    ""metadata"": [ ""value1"", ""value2"" ]
}");

// Convert dynamic values to JSON.
// JSON can be read with a library like System.Text.Json or Newtonsoft.Json
var json = JsonFormatter.Default.Format(status.Data);
var document = JsonDocument.Parse(json);
```

## <a name="additional-resources"></a><span data-ttu-id="4291d-228">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4291d-228">Additional resources</span></span>

* [<span data-ttu-id="4291d-229">Guida alla lingua di protobuf</span><span class="sxs-lookup"><span data-stu-id="4291d-229">Protobuf language guide</span></span>](https://developers.google.com/protocol-buffers/docs/proto3#simple)
* <xref:grpc/versioning>
