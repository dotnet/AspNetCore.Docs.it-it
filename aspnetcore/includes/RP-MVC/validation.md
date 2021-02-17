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

## <a name="add-validation-rules-to-the-movie-model"></a><span data-ttu-id="877ed-101">Aggiungere regole di convalida al modello movie</span><span class="sxs-lookup"><span data-stu-id="877ed-101">Add validation rules to the movie model</span></span>

<span data-ttu-id="877ed-102">Lo spazio dei nomi DataAnnotations offre un set di attributi di convalida predefiniti che vengono applicati in modo dichiarativo a una classe o una proprietà.</span><span class="sxs-lookup"><span data-stu-id="877ed-102">The DataAnnotations namespace provides a set of built-in validation attributes that are applied declaratively to a class or property.</span></span> <span data-ttu-id="877ed-103">DataAnnotations contiene anche gli attributi di formattazione come `DataType` che guidano nella formattazione e non offrono alcuna convalida.</span><span class="sxs-lookup"><span data-stu-id="877ed-103">DataAnnotations also contains formatting attributes like `DataType` that help with formatting and don't provide any validation.</span></span>

<span data-ttu-id="877ed-104">Aggiornare la classe `Movie` per poter sfruttare gli attributi di convalida `Required`, `StringLength`, `RegularExpression` e `Range` predefiniti.</span><span class="sxs-lookup"><span data-stu-id="877ed-104">Update the `Movie` class to take advantage of the built-in `Required`, `StringLength`, `RegularExpression`, and `Range` validation attributes.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Models/MovieDateRatingDA.cs?name=snippet1)]

<span data-ttu-id="877ed-105">Gli attributi di convalida specificano il comportamento da implementare nelle proprietà del modello a cui vengono applicati:</span><span class="sxs-lookup"><span data-stu-id="877ed-105">The validation attributes specify behavior that you want to enforce on the model properties they're applied to:</span></span>

* <span data-ttu-id="877ed-106">Gli attributi `Required` e `MinimumLength` indicano che una proprietà deve avere un valore, ma nulla impedisce all'utente di inserire spazi vuoti per soddisfare questa convalida.</span><span class="sxs-lookup"><span data-stu-id="877ed-106">The `Required` and `MinimumLength` attributes indicate that a property must have a value; but nothing prevents a user from entering white space to satisfy this validation.</span></span>
* <span data-ttu-id="877ed-107">L'attributo `RegularExpression` viene usato per limitare i caratteri che possono essere inseriti.</span><span class="sxs-lookup"><span data-stu-id="877ed-107">The `RegularExpression` attribute is used to limit what characters can be input.</span></span> <span data-ttu-id="877ed-108">Nel codice precedente "Genre":</span><span class="sxs-lookup"><span data-stu-id="877ed-108">In the preceding code, "Genre":</span></span>

  * <span data-ttu-id="877ed-109">Deve includere solo lettere.</span><span class="sxs-lookup"><span data-stu-id="877ed-109">Must only use letters.</span></span>
  * <span data-ttu-id="877ed-110">La prima lettera deve essere maiuscola.</span><span class="sxs-lookup"><span data-stu-id="877ed-110">The first letter is required to be uppercase.</span></span> <span data-ttu-id="877ed-111">Gli spazi, i numeri e i caratteri speciali non sono consentiti.</span><span class="sxs-lookup"><span data-stu-id="877ed-111">White space, numbers, and special characters are not allowed.</span></span>

* <span data-ttu-id="877ed-112">`RegularExpression` "Rating":</span><span class="sxs-lookup"><span data-stu-id="877ed-112">The `RegularExpression` "Rating":</span></span>

  * <span data-ttu-id="877ed-113">Richiede che il primo carattere sia una lettera maiuscola.</span><span class="sxs-lookup"><span data-stu-id="877ed-113">Requires that the first character be an uppercase letter.</span></span>
  * <span data-ttu-id="877ed-114">Consente i caratteri speciali e i numeri negli spazi successivi.</span><span class="sxs-lookup"><span data-stu-id="877ed-114">Allows special characters and numbers in  subsequent spaces.</span></span> <span data-ttu-id="877ed-115">"PG-13" è valido per una classificazione, ma non per "Genre".</span><span class="sxs-lookup"><span data-stu-id="877ed-115">"PG-13" is valid for a rating, but fails for a "Genre".</span></span>

* <span data-ttu-id="877ed-116">L'attributo `Range` vincola un valore all'interno di un intervallo specificato.</span><span class="sxs-lookup"><span data-stu-id="877ed-116">The `Range` attribute constrains a value to within a specified range.</span></span>
* <span data-ttu-id="877ed-117">L'attributo `StringLength` consente di impostare la lunghezza massima di una proprietà stringa e, facoltativamente, la lunghezza minima.</span><span class="sxs-lookup"><span data-stu-id="877ed-117">The `StringLength` attribute lets you set the maximum length of a string property, and optionally its minimum length.</span></span>
* <span data-ttu-id="877ed-118">I tipi di valore, ad esempio `decimal`, `int`, `float` e `DateTime`, sono intrinsecamente necessari e non richiedono l'attributo `[Required]`.</span><span class="sxs-lookup"><span data-stu-id="877ed-118">Value types (such as `decimal`, `int`, `float`, `DateTime`) are inherently required and don't need the `[Required]` attribute.</span></span>

<span data-ttu-id="877ed-119">L'applicazione automatica di regole di convalida da ASP.NET Core è utile per rendere un'app più solida.</span><span class="sxs-lookup"><span data-stu-id="877ed-119">Having validation rules automatically enforced by ASP.NET Core helps make your app more robust.</span></span> <span data-ttu-id="877ed-120">In questo modo inoltre non è possibile omettere la convalida di un elemento e quindi inserire involontariamente dati errati nel database.</span><span class="sxs-lookup"><span data-stu-id="877ed-120">It also ensures that you can't forget to validate something and inadvertently let bad data into the database.</span></span>
