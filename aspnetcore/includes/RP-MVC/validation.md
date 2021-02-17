---
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
ms.openlocfilehash: c96c5e66d2e15db03c321d239a64b98119a7543a
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552806"
---
<!-- USED in RP and MVC tutorial -->

## <a name="add-validation-rules-to-the-movie-model"></a>Aggiungere regole di convalida al modello movie

Lo spazio dei nomi DataAnnotations offre un set di attributi di convalida predefiniti che vengono applicati in modo dichiarativo a una classe o una proprietà. DataAnnotations contiene anche gli attributi di formattazione come `DataType` che guidano nella formattazione e non offrono alcuna convalida.

Aggiornare la classe `Movie` per poter sfruttare gli attributi di convalida `Required`, `StringLength`, `RegularExpression` e `Range` predefiniti.

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Models/MovieDateRatingDA.cs?name=snippet1)]

Gli attributi di convalida specificano il comportamento da implementare nelle proprietà del modello a cui vengono applicati:

* Gli attributi `Required` e `MinimumLength` indicano che una proprietà deve avere un valore, ma nulla impedisce all'utente di inserire spazi vuoti per soddisfare questa convalida.
* L'attributo `RegularExpression` viene usato per limitare i caratteri che possono essere inseriti. Nel codice precedente "Genre":

  * Deve includere solo lettere.
  * La prima lettera deve essere maiuscola. Gli spazi, i numeri e i caratteri speciali non sono consentiti.

* `RegularExpression` "Rating":

  * Richiede che il primo carattere sia una lettera maiuscola.
  * Consente i caratteri speciali e i numeri negli spazi successivi. "PG-13" è valido per una classificazione, ma non per "Genre".

* L'attributo `Range` vincola un valore all'interno di un intervallo specificato.
* L'attributo `StringLength` consente di impostare la lunghezza massima di una proprietà stringa e, facoltativamente, la lunghezza minima.
* I tipi di valore, ad esempio `decimal`, `int`, `float` e `DateTime`, sono intrinsecamente necessari e non richiedono l'attributo `[Required]`.

L'applicazione automatica di regole di convalida da ASP.NET Core è utile per rendere un'app più solida. In questo modo inoltre non è possibile omettere la convalida di un elemento e quindi inserire involontariamente dati errati nel database.
