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
ms.openlocfilehash: adb048c9b20e172ef8a7bc13f9b56928b1d4bf77
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280296"
---
# <a name="create-protobuf-messages-for-net-apps"></a>Creare messaggi protobuf per le app .NET

Di [James Newton-King](https://twitter.com/jamesnk) e [Mark Rendle](https://twitter.com/markrendle)

gRPC USA [protobuf](https://developers.google.com/protocol-buffers) come linguaggio di definizione dell'interfaccia (IDL). Protobuf IDL è un formato indipendente dal linguaggio per la specifica dei messaggi inviati e ricevuti dai servizi di gRPC. I messaggi protobuf sono definiti nei `.proto` file. Questo documento illustra come i concetti di protobuf vengono mappati a .NET.

## <a name="protobuf-messages"></a>Messaggi protobuf

I messaggi sono l'oggetto principale per il trasferimento dei dati in protobuf. Sono concettualmente simili alle classi .NET.

```protobuf
syntax = "proto3";

option csharp_namespace = "Contoso.Messages";

message Person {
    int32 id = 1;
    string first_name = 2;
    string last_name = 3;
}  
```

La definizione di messaggio precedente specifica tre campi come coppie nome-valore. Analogamente alle proprietà sui tipi .NET, ogni campo dispone di un nome e di un tipo. Il tipo di campo può essere un [tipo di valore scalare protobuf](#scalar-value-types), ad esempio `int32` o un altro messaggio.

La [Guida di stile protobuf](https://developers.google.com/protocol-buffers/docs/style) consiglia `underscore_separated_names` di usare per i nomi di campo. I nuovi messaggi protobuf creati per le app .NET devono seguire le linee guida di stile protobuf. Gli strumenti .NET generano automaticamente tipi .NET che utilizzano standard di denominazione .NET. Un campo protobuf, ad esempio, `first_name` genera una `FirstName` Proprietà .NET.

Oltre a un nome, ogni campo nella definizione del messaggio ha un numero univoco. I numeri di campo vengono utilizzati per identificare i campi quando il messaggio viene serializzato in protobuf. La serializzazione di un numero ridotto è più veloce rispetto alla serializzazione dell'intero nome di campo. Poiché i numeri di campo identificano un campo, è importante prestare attenzione durante la modifica. Per ulteriori informazioni sulla modifica dei messaggi protobuf <xref:grpc/versioning> , vedere.

Quando viene compilata un'app, gli strumenti protobuf generano tipi .NET dai `.proto` file. Il `Person` messaggio genera una classe .NET:

```csharp
public class Person
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

Per ulteriori informazioni sui messaggi protobuf, vedere la [Guida alla lingua di protobuf](https://developers.google.com/protocol-buffers/docs/proto3#simple).

## <a name="scalar-value-types"></a>Tipi di valore scalari

Protobuf supporta un intervallo di tipi valore scalari nativi. La tabella seguente elenca tutti i tipi con il tipo C# equivalente:

| Tipo protobuf | Tipo C#      |
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

I valori scalari hanno sempre un valore predefinito e non possono essere impostati su `null` . Questo vincolo include `string` e `ByteString` che sono classi C#. `string` il valore predefinito è un valore stringa vuoto e il valore `ByteString` predefinito è un valore byte vuoti. Il tentativo di impostarli su `null` genera un errore.

I [tipi di wrapper Nullable](#nullable-types) possono essere utilizzati per supportare i valori null.

### <a name="dates-and-times"></a>Date e ore

I tipi scalari nativi non forniscono i valori di data e ora, equivalenti a. NET <xref:System.DateTimeOffset> , <xref:System.DateTime> e <xref:System.TimeSpan> . Questi tipi possono essere specificati usando alcune estensioni dei *tipi noti* di protobuf. Queste estensioni forniscono il supporto per la generazione di codice e il runtime per i tipi di campo complessi nelle piattaforme supportate.

La tabella seguente illustra i tipi di data e ora:

| Tipo .NET        | Tipo Well-Known protobuf    |
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

Le proprietà generate nella classe C# non sono i tipi di data e ora .NET. Le proprietà utilizzano le `Timestamp` `Duration` classi e nello `Google.Protobuf.WellKnownTypes` spazio dei nomi. Queste classi forniscono metodi per la conversione da e verso `DateTimeOffset` , `DateTime` e `TimeSpan` .

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
> Il `Timestamp` tipo funziona con l'ora UTC. `DateTimeOffset` i valori hanno sempre un offset pari a zero e la `DateTime.Kind` proprietà è sempre `DateTimeKind.Utc` .

### <a name="nullable-types"></a>Tipi nullable

La generazione di codice protobuf per C# usa i tipi nativi, ad esempio `int` per `int32` . Pertanto, i valori vengono sempre inclusi e non possono essere `null` .

Per i valori che richiedono esplicito `null` , ad esempio l'uso `int?` di nel codice c#, i tipi di Well-Known di protobuf includono wrapper compilati in tipi c# Nullable. Per usarli, importare `wrappers.proto` nel `.proto` file, come nel codice seguente:

```protobuf  
syntax = "proto3"

import "google/protobuf/wrappers.proto"

message Person {
    // ...
    google.protobuf.Int32Value age = 5;
}
```

`wrappers.proto` i tipi non sono esposti nelle proprietà generate. Protobuf esegue automaticamente il mapping ai tipi nullable .NET appropriati nei messaggi C#. Un campo, ad esempio, `google.protobuf.Int32Value` genera una `int?` Proprietà. Le proprietà del tipo `string` di riferimento, ad esempio e, `ByteString` sono invariate tranne `null` che possono essere assegnate senza errori.

La tabella seguente mostra l'elenco completo dei tipi di wrapper con il tipo C# equivalente:

| Tipo C#      | Wrapper di tipo Well-Known       |
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

### <a name="bytes"></a>Byte

I payload binari sono supportati in protobuf con il `bytes` tipo di valore scalare. Una proprietà generata in C# usa `ByteString` come tipo di proprietà.

Utilizzare `ByteString.CopyFrom(byte[] data)` per creare una nuova istanza da una matrice di byte:

```csharp
var data = await File.ReadAllBytesAsync(path);

var payload = new PayloadResponse();
payload.Data = ByteString.CopyFrom(data);
```

`ByteString` è possibile accedere ai dati direttamente usando `ByteString.Span` o `ByteString.Memory` . In alternativa `ByteString.ToByteArray()` , chiamare per convertire nuovamente un'istanza in una matrice di byte:

```csharp
var payload = await client.GetPayload(new PayloadRequest());

await File.WriteAllBytesAsync(path, payload.Data.ToByteArray());
```

### <a name="decimals"></a>Decimali

Protobuf non supporta in modo nativo il `decimal` tipo .NET, solo `double` e `float` . Nel progetto protobuf è presente una discussione continuativa sulla possibilità di aggiungere un tipo decimale standard ai tipi di Well-Known, con supporto della piattaforma per linguaggi e Framework che lo supportano. Non è stato ancora implementato alcun elemento.

È possibile creare una definizione di messaggio per rappresentare il `decimal` tipo che funziona per la serializzazione sicura tra client e server .NET. Gli sviluppatori su altre piattaforme dovrebbero tuttavia comprendere il formato usato e implementare la propria gestione per l'it.

#### <a name="creating-a-custom-decimal-type-for-protobuf"></a>Creazione di un tipo decimale personalizzato per protobuf

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

Il `nanos` campo rappresenta i valori da `0.999_999_999` a `-0.999_999_999` . Il valore, ad esempio, viene `decimal` `1.5m` rappresentato come `{ units = 1, nanos = 500_000_000 }` . Questo è il motivo `nanos` per cui il campo in questo esempio usa il `sfixed32` tipo, che codifica in modo più efficiente rispetto `int32` ai valori più grandi. Se il `units` campo è negativo, `nanos` anche il campo dovrebbe essere negativo.

> [!NOTE]
> Sono disponibili più algoritmi per la codifica `decimal` dei valori come stringhe di byte, ma questo messaggio è più facile da comprendere. I valori non sono interessati da Big-endian o little-endian su piattaforme diverse.

La conversione tra questo tipo e il `decimal` tipo BCL potrebbe essere implementata in C# come segue:

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

## <a name="collections"></a>Raccolte

### <a name="lists"></a>Elenchi

Gli elenchi in protobuf vengono specificati utilizzando la `repeated` parola chiave prefix in un campo. Nell'esempio seguente viene illustrato come creare un elenco:

```protobuf
message Person {
    // ...
    repeated string roles = 8;
}
```

Nel codice generato, `repeated` i campi sono rappresentati dal `Google.Protobuf.Collections.RepeatedField<T>` tipo generico.

```csharp
public class Person
{
    // ...
    public RepeatedField<string> Roles { get; }
}
```

`RepeatedField<T>` implementa <xref:System.Collections.Generic.IList%601>. Quindi, è possibile usare le query LINQ o convertirle in una matrice o in un elenco. `RepeatedField<T>` le proprietà non hanno un setter pubblico. Gli elementi devono essere aggiunti alla raccolta esistente.

```csharp
var person = new Person();

// Add one item.
person.Roles.Add("user");

// Add all items from another collection.
var roles = new [] { "admin", "manager" };
person.Roles.Add(roles);
```

### <a name="dictionaries"></a>Dizionari

Il <xref:System.Collections.Generic.IDictionary%602> tipo .NET è rappresentato in protobuf usando `map<key_type, value_type>` .

```protobuf
message Person {
    // ...
    map<string, string> attributes = 9;
}
```

Nel codice .NET generato, `map` i campi sono rappresentati dal `Google.Protobuf.Collections.MapField<TKey, TValue>` tipo generico. `MapField<TKey, TValue>` implementa <xref:System.Collections.Generic.IDictionary%602>. Analogamente alle `repeated` proprietà, le `map` proprietà non hanno un setter pubblico. Gli elementi devono essere aggiunti alla raccolta esistente.

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

## <a name="unstructured-and-conditional-messages"></a>Messaggi non strutturati e condizionali

Protobuf è un formato di messaggistica prima del contratto. I messaggi di un'app, inclusi i campi e i tipi, devono essere specificati nei `.proto` file quando viene compilata l'app. La progettazione dei contratti di protobuf è ideale per applicare il contenuto dei messaggi, ma può limitare gli scenari in cui non è necessario un contratto rigido:

* Messaggi con payload sconosciuti. Ad esempio, un messaggio con un campo che può contenere qualsiasi messaggio.
* Messaggi condizionali. Un messaggio restituito da un servizio gRPC, ad esempio, potrebbe essere un risultato positivo o un errore.
* Valori dinamici. Ad esempio, un messaggio con un campo che contiene una raccolta non strutturata di valori, simile a JSON.

Protobuf offre funzionalità del linguaggio e tipi per supportare questi scenari.

### <a name="any"></a>Qualsiasi

Il `Any` tipo consente di usare i messaggi come tipi incorporati senza avere la relativa `.proto` definizione. Per utilizzare il `Any` tipo, Import `any.proto` .

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

### <a name="oneof"></a>Di

`oneof` i campi sono una funzionalità del linguaggio. Il compilatore gestisce la `oneof` parola chiave quando genera la classe Message. `oneof`L'utilizzo di per specificare un messaggio di risposta che può restituire `Person` o potrebbe essere simile al `Error` seguente:

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

I campi all'interno del `oneof` set devono avere numeri di campo univoci nella dichiarazione generale del messaggio.

Quando `oneof` si usa, il codice C# generato include un'enumerazione che specifica quale dei campi è stata impostata. È possibile testare l'enumerazione per individuare il campo da impostare. I campi che non sono impostati restituiscono `null` o il valore predefinito, anziché generare un'eccezione.

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

### <a name="value"></a>Valore

Il `Value` tipo rappresenta un valore tipizzato dinamicamente. Può essere `null` , un numero, una stringa, un valore booleano, un dizionario di valori ( `Struct` ) o un elenco di valori ( `ValueList` ). `Value` è un tipo di Well-Known protobuf che utilizza la funzionalità descritta in precedenza `oneof` . Per utilizzare il `Value` tipo, Import `struct.proto` .

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

L'uso `Value` diretto può essere dettagliato. Un modo alternativo per usare `Value` è il supporto incorporato di protobuf per il mapping dei messaggi a JSON. `JsonFormatter`I tipi e di protobuf `JsonWriter` possono essere usati con qualsiasi messaggio protobuf. `Value` è particolarmente adatto per essere convertito in e da JSON.

Si tratta dell'equivalente JSON del codice precedente:

```csharp
// Create dynamic values from JSON.
var status = new Status();
status.Data = Value.Parser.ParseJson(@"{
    ""enabled"": true,
    ""metadata"": [ ""value1"", ""value2"" ]
}");

// Convert dynamic values to JSON.
// JSON can be read with a library like System.Text.Json or Newtonsoft.Json
var json = JsonFormatter.Default.Format(status.Metadata);
var document = JsonDocument.Parse(json);
```

## <a name="additional-resources"></a>Risorse aggiuntive

* [Guida alla lingua di protobuf](https://developers.google.com/protocol-buffers/docs/proto3#simple)
* <xref:grpc/versioning>
