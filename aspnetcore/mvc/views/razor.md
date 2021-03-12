---
title: informazioni di riferimento sulla sintassi Razor per ASP.NET Core
author: rick-anderson
description: Informazioni sulla Razor sintassi di markup per l'incorporamento di codice basato su server in pagine Web.
ms.author: riande
ms.date: 02/12/2020
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
uid: mvc/views/razor
ms.openlocfilehash: 60471232b3373039404b27c4afd1a1725d4d21eb
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586865"
---
# <a name="razor-syntax-reference-for-aspnet-core"></a>Razor informazioni di riferimento sulla sintassi per ASP.NET Core

Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Taylor Mullen](https://twitter.com/ntaylormullen)e [Dan Vicarel](https://github.com/Rabadash8820)

Razor è una sintassi di markup per l'incorporamento di codice basato su server in pagine Web. La Razor sintassi è costituita da Razor markup, C# e HTML. I file che contengono Razor in genere hanno un'estensione di file *. cshtml* . Razorsi trova anche nei file dei [ Razor componenti](xref:blazor/components/index) (*Razor*).

## <a name="rendering-html"></a>Rendering di HTML

La Razor lingua predefinita è HTML. Il rendering del codice HTML dal Razor markup non è diverso dal rendering HTML da un file HTML. Il rendering del markup HTML nei file con *estensione cshtml* Razor viene eseguito senza modifiche al server.

## <a name="razor-syntax"></a>Sintassi Razor

Razor supporta C# e usa il `@` simbolo per eseguire la transizione da HTML a c#. Razor valuta le espressioni C# e ne esegue il rendering nell'output HTML.

Quando un `@` simbolo è seguito da una [ Razor parola chiave riservata](#razor-reserved-keywords), viene eseguita la transizione a un Razor markup specifico. in caso contrario in C# semplice.

Per eseguire l'escape di un `@` simbolo nel Razor markup, usare un secondo `@` simbolo:

```cshtml
<p>@@Username</p>
```

Il rendering del codice viene eseguito in HTML con un solo simbolo `@`:

```html
<p>@Username</p>
```

Gli attributi e il contenuto HTML contenenti indirizzi di posta elettronica non considerano il simbolo `@` come un carattere di transizione. Gli indirizzi di posta elettronica nell'esempio seguente non vengono modificati mediante l' Razor analisi:

```cshtml
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

## <a name="implicit-razor-expressions"></a>Espressioni implicite Razor

RazorLe espressioni implicite iniziano con `@` seguito dal codice C#:

```cshtml
<p>@DateTime.Now</p>
<p>@DateTime.IsLeapYear(2016)</p>
```

Fatta eccezione per la parola chiave `await` di C#, le espressioni implicite non devono contenere spazi. Se l'istruzione C# ha una fine chiaramente definita, possono coesistere spazi:

```cshtml
<p>@await DoSomething("hello", "world")</p>
```

Le espressioni implicite **non possono** contenere generics C#, poiché i caratteri all'interno delle parentesi (`<>`) vengono interpretati come un tag HTML. Il codice seguente **non** è valido:

```cshtml
<p>@GenericMethod<int>()</p>
```

Il codice precedente genera un errore del compilatore simile a uno dei seguenti:

* L'elemento "int" non è stato chiuso. Tutti gli elementi devono essere a chiusura automatica o avere un tag di fine corrispondente.
* Non è possibile convertire il gruppo di metodi 'GenericMethod' nel tipo non delegato 'object'. Si intendeva chiamare il metodo?'

Le chiamate al metodo generico devono essere incapsulate in un' [ Razor espressione esplicita](#explicit-razor-expressions) o in un [ Razor blocco di codice](#razor-code-blocks).

## <a name="explicit-razor-expressions"></a>Espressioni esplicite Razor

Le Razor espressioni esplicite sono costituite da un `@` simbolo con parentesi bilanciate. Per eseguire il rendering dell'ora dell'ultima settimana, Razor viene usato il markup seguente:

```cshtml
<p>Last week this time: @(DateTime.Now - TimeSpan.FromDays(7))</p>
```

Qualsiasi contenuto all'interno delle parentesi `@()` viene valutato e sottoposto a rendering nell'output.

Le espressioni implicite, descritte nella sezione precedente, in genere non possono contenere spazi. Nel codice seguente, una settimana non viene sottratta dall'ora corrente:

[!code-cshtml[](razor/sample/Views/Home/Contact.cshtml?range=17)]

Il codice esegue il rendering dell'HTML seguente:

```html
<p>Last week: 7/7/2016 4:39:52 PM - TimeSpan.FromDays(7)</p>
```

È possibile usare le espressioni esplicite per concatenare testo con un risultato dell'espressione:

```cshtml
@{
    var joe = new Person("Joe", 33);
}

<p>Age@(joe.Age)</p>
```

Senza l'espressione esplicita, `<p>Age@joe.Age</p>` viene considerato come un indirizzo di posta elettronica e `<p>Age@joe.Age</p>` viene sottoposto a rendering. Se viene scritto come espressione esplicita, `<p>Age33</p>` viene sottoposto a rendering.

È possibile usare le espressioni esplicite per eseguire il rendering dell'output da metodi generici in file con estensione *cshtml*. Il markup seguente illustra come correggere l'errore riportato in precedenza, causato dalle parentesi quadre di un oggetto generico C#. Il codice viene scritto come un'espressione esplicita:

```cshtml
<p>@(GenericMethod<int>())</p>
```

## <a name="expression-encoding"></a>Codifica di espressioni

Le espressioni C# che restituiscono una stringa sono codificate in HTML. Le espressioni C# che restituiscono `IHtmlContent` vengono sottoposte a rendering direttamente tramite `IHtmlContent.WriteTo`. Le espressioni C# che non restituiscono `IHtmlContent` vengono convertite in una stringa da `ToString` e codificate prima di essere sottoposte a rendering.

```cshtml
@("<span>Hello World</span>")
```

Il codice precedente esegue il rendering del codice HTML seguente:

```html
&lt;span&gt;Hello World&lt;/span&gt;
```

Il codice HTML viene visualizzato nel browser come testo normale:

&lt;span &gt; Hello World &lt; /span&gt;

L'output `HtmlHelper.Raw` non è codificato ma viene sottoposto a rendering come markup HTML.

> [!WARNING]
> L'uso di `HtmlHelper.Raw` su input utente non purificato costituisce un rischio per la sicurezza. L'input utente potrebbe contenere JavaScript dannoso o altri attacchi. La purificazione degli input utente è difficile. Evitare l'uso di `HtmlHelper.Raw` con l'input utente.

```cshtml
@Html.Raw("<span>Hello World</span>")
```

Il codice esegue il rendering dell'HTML seguente:

```html
<span>Hello World</span>
```

## <a name="razor-code-blocks"></a>Razor blocchi di codice

Razor i blocchi di codice iniziano con `@` e sono racchiusi tra `{}` . A differenza delle espressioni, il codice C# all'interno di blocchi di codice non viene sottoposto a rendering. I blocchi di codice e le espressioni in una visualizzazione condividono lo stesso ambito e vengono definiti in ordine:

```cshtml
@{
    var quote = "The future depends on what you do today. - Mahatma Gandhi";
}

<p>@quote</p>

@{
    quote = "Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.";
}

<p>@quote</p>
```

Il codice esegue il rendering dell'HTML seguente:

```html
<p>The future depends on what you do today. - Mahatma Gandhi</p>
<p>Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.</p>
```

::: moniker range=">= aspnetcore-3.0"

Nei blocchi di codice dichiarare [funzioni locali](/dotnet/csharp/programming-guide/classes-and-structs/local-functions) con markup da usare come metodi per la creazione di modelli:

```cshtml
@{
    void RenderName(string name)
    {
        <p>Name: <strong>@name</strong></p>
    }

    RenderName("Mahatma Gandhi");
    RenderName("Martin Luther King, Jr.");
}
```

Il codice esegue il rendering dell'HTML seguente:

```html
<p>Name: <strong>Mahatma Gandhi</strong></p>
<p>Name: <strong>Martin Luther King, Jr.</strong></p>
```

::: moniker-end

### <a name="implicit-transitions"></a>Transizioni implicite

La lingua predefinita in un blocco di codice è C#, ma la pagina è in Razor grado di eseguire la transizione a HTML:

```cshtml
@{
    var inCSharp = true;
    <p>Now in HTML, was in C# @inCSharp</p>
}
```

### <a name="explicit-delimited-transition"></a>Transizione esplicita delimitata

Per definire una sottosezione di un blocco di codice che deve eseguire il rendering di HTML, racchiudere i caratteri per il rendering con il Razor `<text>` Tag:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <text>Name: @person.Name</text>
}
```

Usare questo approccio per eseguire il rendering di HTML che non è racchiuso tra tag HTML. Senza un tag HTML o Razor , Razor si verifica un errore di Runtime.

Il tag `<text>` è utile per controllare gli spazi vuoti durante il rendering del contenuto:

* Solo il contenuto all'interno del tag `<text>` viene sottoposto a rendering.
* Non vengono visualizzati spazi vuoti prima o dopo il tag `<text>` nell'output HTML.

### <a name="explicit-line-transition"></a>Transizione di riga esplicita

Per eseguire il rendering del resto di un'intera riga come HTML all'interno di un blocco di codice, usare la `@:` sintassi:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    @:Name: @person.Name
}
```

Senza l'oggetto `@:` nel codice, Razor viene generato un errore di Runtime.

`@`I caratteri aggiuntivi in un Razor file possono causare errori del compilatore nelle istruzioni successive nel blocco. Questi errori del compilatore possono essere difficili da comprendere perché l'errore effettivo si verifica prima dell'errore segnalato. Questo errore è comune dopo la combinazione di più espressioni implicite/esplicite in un singolo blocco di codice.

## <a name="control-structures"></a>Strutture di controllo

Le strutture di controllo sono un'estensione dei blocchi di codice. Tutti gli aspetti dei blocchi di codice (transizione al markup, C# inline) sono validi anche per le strutture seguenti:

### <a name="conditionals-if-else-if-else-and-switch"></a>Istruzioni condizionali `@if, else if, else, and @switch`

`@if` controlla quando viene eseguito il codice:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
```

`else` e `else if` non richiedono il simbolo `@`:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
else if (value >= 1337)
{
    <p>The value is large.</p>
}
else
{
    <p>The value is odd and small.</p>
}
```

Nel markup seguente viene illustrato come usare un'istruzione switch:

```cshtml
@switch (value)
{
    case 1:
        <p>The value is 1!</p>
        break;
    case 1337:
        <p>Your number is 1337!</p>
        break;
    default:
        <p>Your number wasn't 1 or 1337.</p>
        break;
}
```

### <a name="looping-for-foreach-while-and-do-while"></a>Ciclo `@for, @foreach, @while, and @do while`

È possibile eseguire il rendering di HTML basato su modelli con le istruzioni di controllo ciclo. Per eseguire il rendering di un elenco di persone:

```cshtml
@{
    var people = new Person[]
    {
          new Person("Weston", 33),
          new Person("Johnathon", 41),
          ...
    };
}
```

Sono supportate le seguenti istruzioni di ciclo:

`@for`

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@foreach`

```cshtml
@foreach (var person in people)
{
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@while`

```cshtml
@{ var i = 0; }
@while (i < people.Length)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
}
```

`@do while`

```cshtml
@{ var i = 0; }
@do
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
} while (i < people.Length);
```

### <a name="compound-using"></a>Istruzione `@using` composita

In C# viene usata un'istruzione `using` per verificare che un oggetto sia stato eliminato. In Razor lo stesso meccanismo viene usato per creare helper HTML che contengono contenuto aggiuntivo. Nel codice seguente gli helper HTML eseguono il rendering di un tag `<form>` con l'istruzione `@using`:

```cshtml
@using (Html.BeginForm())
{
    <div>
        Email: <input type="email" id="Email" value="">
        <button>Register</button>
    </div>
}
```

### `@try, catch, finally`

La gestione delle eccezioni è simile a C#:

[!code-cshtml[](razor/sample/Views/Home/Contact7.cshtml)]

### `@lock`

Razor offre la possibilità di proteggere le sezioni critiche con le istruzioni lock:

```cshtml
@lock (SomeLock)
{
    // Do critical section work
}
```

### <a name="comments"></a>Commenti

Razor supporta i commenti in C# e HTML:

```cshtml
@{
    /* C# comment */
    // Another C# comment
}
<!-- HTML comment -->
```

Il codice esegue il rendering dell'HTML seguente:

```html
<!-- HTML comment -->
```

Razor i commenti vengono rimossi dal server prima che venga eseguito il rendering della pagina Web. Razor USA `@*  *@` per delimitare i commenti. Il codice seguente è commentato, pertanto il server non esegue il rendering di alcun markup:

```cshtml
@*
    @{
        /* C# comment */
        // Another C# comment
    }
    <!-- HTML comment -->
*@
```

## <a name="directives"></a>Direttive

Razor le direttive sono rappresentate da espressioni implicite con parole chiave riservate che seguono il `@` simbolo. Una direttiva cambia in genere il modo in cui viene analizzata una visualizzazione o abilita funzionalità diverse.

Comprendere in che modo Razor genera il codice per una vista rende più semplice comprendere il funzionamento delle direttive.

[!code-cshtml[](razor/sample/Views/Home/Contact8.cshtml)]

Il codice genera una classe simile alla seguente:

```csharp
public class _Views_Something_cshtml : RazorPage<dynamic>
{
    public override async Task ExecuteAsync()
    {
        var output = "Getting old ain't for wimps! - Anonymous";

        WriteLiteral("/r/n<div>Quote of the Day: ");
        Write(output);
        WriteLiteral("</div>");
    }
}
```

Più avanti in questo articolo, la sezione [esaminare la Razor classe C# generata per una visualizzazione](#inspect-the-razor-c-class-generated-for-a-view) spiega come visualizzare questa classe generata.

### `@attribute`

La direttiva `@attribute` aggiunge l'attributo specificato alla classe della pagina o della visualizzazione generata. L'esempio seguente aggiunge l'attributo `[Authorize]`:

```cshtml
@attribute [Authorize]
```

::: moniker range=">= aspnetcore-3.0"

### `@code`

*Questo scenario si applica solo ai Razor componenti (Razor).*

Il `@code` blocco consente a un [ Razor componente](xref:blazor/components/index) di aggiungere membri C# (campi, proprietà e metodi) a un componente:

```razor
@code {
    // C# members (fields, properties, and methods)
}
```

Per i Razor componenti di, `@code` è un alias di [`@functions`](#functions) e consigliato rispetto a `@functions` . È consentito più di un blocco `@code`.

::: moniker-end

### `@functions`

La direttiva `@functions` consente di aggiungere membri C# (campi, proprietà e metodi) alla classe generata:

```cshtml
@functions {
    // C# members (fields, properties, and methods)
}
```

::: moniker range=">= aspnetcore-3.0"

In [ Razor Components](xref:blazor/components/index)usare `@code` over `@functions` per aggiungere membri C#.

::: moniker-end

Ad esempio:

[!code-cshtml[](razor/sample/Views/Home/Contact6.cshtml)]

Il codice genera il markup HTML seguente:

```html
<div>From method: Hello</div>
```

Il codice seguente è la Razor classe C# generata:

[!code-csharp[](razor/sample/Classes/Views_Home_Test_cshtml.cs?range=1-19)]

::: moniker range=">= aspnetcore-3.0"

I metodi `@functions` fungono da metodi per la creazione di modelli quando includono markup:

```cshtml
@{
    RenderName("Mahatma Gandhi");
    RenderName("Martin Luther King, Jr.");
}

@functions {
    private void RenderName(string name)
    {
        <p>Name: <strong>@name</strong></p>
    }
}
```

Il codice esegue il rendering dell'HTML seguente:

```html
<p>Name: <strong>Mahatma Gandhi</strong></p>
<p>Name: <strong>Martin Luther King, Jr.</strong></p>
```

### `@implements`

La direttiva `@implements` implementa un'interfaccia per la classe generata.

L'esempio seguente implementa <xref:System.IDisposable?displayProperty=fullName> in modo che sia possibile chiamare il metodo <xref:System.IDisposable.Dispose*>:

```cshtml
@implements IDisposable

<h1>Example</h1>

@functions {
    private bool _isDisposed;

    ...

    public void Dispose() => _isDisposed = true;
}
```

::: moniker-end

### `@inherits`

La direttiva `@inherits` offre il controllo completo della classe che viene ereditata dalla visualizzazione:

```cshtml
@inherits TypeNameOfClassToInheritFrom
```

Il codice seguente è un Razor tipo di pagina personalizzato:

[!code-csharp[](razor/sample/Classes/CustomRazorPage.cs)]

L'elemento `CustomText` viene inserito in una visualizzazione:

[!code-cshtml[](razor/sample/Views/Home/Contact10.cshtml)]

Il codice esegue il rendering dell'HTML seguente:

```html
<div>
    Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping
    a slop bucket on the street below.
</div>
```

 È possibile usare `@model` e `@inherits` nella stessa visualizzazione. `@inherits` può trovarsi in un file *_ViewImports.cshtml* che viene importato dalla visualizzazione:

[!code-cshtml[](razor/sample/Views/_ViewImportsModel.cshtml)]

Il codice seguente è un esempio di visualizzazione fortemente tipizzata:

[!code-cshtml[](razor/sample/Views/Home/Login1.cshtml)]

Se "rick@contoso.com" viene passato nel modello, la visualizzazione genera il markup HTML seguente:

```html
<div>The Login Email: rick@contoso.com</div>
<div>
    Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping
    a slop bucket on the street below.
</div>
```

### `@inject`

La `@inject` direttiva consente alla Razor pagina di inserire un servizio dal [contenitore di servizi](xref:fundamentals/dependency-injection) in una vista. Per altre informazioni, vedere [Inserimento di dipendenze in visualizzazioni](xref:mvc/views/dependency-injection).

::: moniker range=">= aspnetcore-3.0"

### `@layout`

*Questo scenario si applica solo ai Razor componenti (Razor).*

La `@layout` direttiva specifica un layout per i componenti instradabili con Razor una [`@page`](#page) direttiva. I componenti di layout vengono usati per evitare la duplicazione e l'incoerenza del codice. Per altre informazioni, vedere <xref:blazor/layouts>.

::: moniker-end

### `@model`

*Questo scenario si applica solo a viste e Razor pagine MVC (cshtml).*

La direttiva `@model` specifica il tipo del modello passato a una vista o a una pagina:

```cshtml
@model TypeNameOfModel
```

In un'app ASP.NET Core MVC o Razor pagine creata con singoli account utente, *views/account/login. cshtml* contiene la dichiarazione del modello seguente:

```cshtml
@model LoginViewModel
```

La classe generata eredita da `RazorPage<dynamic>`:

```csharp
public class _Views_Account_Login_cshtml : RazorPage<LoginViewModel>
```

Razor espone una `Model` proprietà per l'accesso al modello passato alla visualizzazione:

```cshtml
<div>The Login Email: @Model.Email</div>
```

La direttiva `@model` specifica il tipo della proprietà `Model`. La direttiva specifica l'elemento `T` in `RazorPage<T>` che ha generato la classe da cui deriva la visualizzazione. Se la direttiva `@model` non è specificata, la proprietà `Model` è di tipo `dynamic`. Per ulteriori informazioni, vedere [modelli fortemente tipizzati e la @model parola chiave](xref:tutorials/first-mvc-app/adding-model#strongly-typed-models-and-the--keyword).

### `@namespace`

La direttiva `@namespace`:

* Imposta lo spazio dei nomi della classe della Razor Pagina generata, della visualizzazione MVC o del Razor componente.
* Imposta gli spazi dei nomi derivati radice di una classe di pagine, viste o componenti dal file di importazione più vicino nell'albero di directory *_ViewImports. cshtml* (viste o pagine) o *_Imports. Razor* ( Razor Components).

```cshtml
@namespace Your.Namespace.Here
```

Per l' Razor esempio di pagine illustrato nella tabella seguente:

* Ogni pagina importa *Pages/_ViewImports.cshtml*.
* *Pages/_ViewImports.cshtml* contiene `@namespace Hello.World`.
* Ogni pagina ha `Hello.World` come radice dello spazio dei nomi.

| Pagina                                        | Spazio dei nomi                             |
| ------------------------------------------- | ------------------------------------- |
| *Pages/index. cshtml*                        | `Hello.World`                         |
| *Pages/MorePages/Page.cshtml*               | `Hello.World.MorePages`               |
| *Pages/MorePages/EvenMorePages/Page.cshtml* | `Hello.World.MorePages.EvenMorePages` |

Le relazioni precedenti si applicano ai file di importazione utilizzati con i componenti e le visualizzazioni MVC Razor .

Quando più file di importazione hanno una direttiva `@namespace`, per impostare lo spazio dei nomi radice, viene usato il file più vicino alla pagina, alla vista o al componente nell'albero di directory.

Se la cartella *EvenMorePages* nell'esempio precedente ha un file di importazione con `@namespace Another.Planet` (oppure il file *Pages/MorePages/EvenMorePages/Page.cshtml* contiene `@namespace Another.Planet`), il risultato è illustrato nella tabella seguente.

| Pagina                                        | Spazio dei nomi               |
| ------------------------------------------- | ----------------------- |
| *Pages/index. cshtml*                        | `Hello.World`           |
| *Pages/MorePages/Page.cshtml*               | `Hello.World.MorePages` |
| *Pages/MorePages/EvenMorePages/Page.cshtml* | `Another.Planet`        |

### `@page`

::: moniker range=">= aspnetcore-3.0"

La direttiva `@page` ha effetti diversi a seconda del tipo del file in cui viene visualizzata. La direttiva:

* In un file con *estensione cshtml* indica che il file è una Razor pagina. Per ulteriori informazioni, vedere [route personalizzate](xref:razor-pages/index#custom-routes) e <xref:razor-pages/index> .
* Specifica che un Razor componente deve gestire direttamente le richieste. Per altre informazioni, vedere <xref:blazor/fundamentals/routing>.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

La `@page` direttiva nella prima riga di un file con *estensione cshtml* indica che il file è una Razor pagina. Per altre informazioni, vedere <xref:razor-pages/index>.

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

### `@preservewhitespace`

*Questo scenario si applica solo ai Razor componenti ( `.razor` ).*

Quando è impostato su `false` (impostazione predefinita), lo spazio vuoto nel markup di cui è stato eseguito il rendering da Razor Components ( `.razor` ) viene rimosso se:

* Iniziali o finali all'interno di un elemento.
* Iniziali o finali in un `RenderFragment` parametro. Ad esempio, il contenuto figlio viene passato a un altro componente.
* Precede o segue un blocco di codice C#, ad esempio `@if` o `@foreach` .

::: moniker-end

### `@section`

*Questo scenario si applica solo a viste e Razor pagine MVC (cshtml).*

La `@section` direttiva viene utilizzata insieme ai [layout MVC e Razor pagine](xref:mvc/views/layout) per consentire alle visualizzazioni o alle pagine di eseguire il rendering del contenuto in diverse parti della pagina HTML. Per altre informazioni, vedere <xref:mvc/views/layout>.

### `@using`

La direttiva `@using` aggiunge la direttiva C# `using` alla visualizzazione generata:

[!code-cshtml[](razor/sample/Views/Home/Contact9.cshtml)]

::: moniker range=">= aspnetcore-3.0"

Nei [ Razor componenti](xref:blazor/components/index)di `@using` Controlla anche quali componenti sono inclusi nell'ambito.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="directive-attributes"></a>Attributi delle direttive

Razor gli attributi di direttiva sono rappresentati da espressioni implicite con parole chiave riservate che seguono il `@` simbolo. Un attributo di direttiva cambia in genere il modo in cui un elemento viene analizzato o Abilita diverse funzionalità.

### `@attributes`

*Questo scenario si applica solo ai Razor componenti (Razor).*

`@attributes` consente a un componente di eseguire il rendering di attributi non dichiarati. Per altre informazioni, vedere <xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters>.

### `@bind`

*Questo scenario si applica solo ai Razor componenti (Razor).*

Il data binding nei componenti viene eseguito con l'attributo `@bind`. Per altre informazioni, vedere <xref:blazor/components/data-binding>.

### `@on{EVENT}`

*Questo scenario si applica solo ai Razor componenti (Razor).*

Razor fornisce funzionalità di gestione degli eventi per i componenti di. Per altre informazioni, vedere <xref:blazor/components/event-handling>.

::: moniker-end

::: moniker range=">= aspnetcore-3.1"

### `@on{EVENT}:preventDefault`

*Questo scenario si applica solo ai Razor componenti (Razor).*

Impedisce l'azione predefinita per l'evento.

### `@on{EVENT}:stopPropagation`

*Questo scenario si applica solo ai Razor componenti (Razor).*

Arresta la propagazione degli eventi per l'evento.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### `@key`

*Questo scenario si applica solo ai Razor componenti (Razor).*

L'attributo della direttiva `@key` fa in modo che l'algoritmo di controllo delle differenze tra componenti garantisca la conservazione degli elementi o dei componenti in base al valore della chiave. Per altre informazioni, vedere <xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components>.

### `@ref`

*Questo scenario si applica solo ai Razor componenti (Razor).*

I riferimenti ai componenti (`@ref`) consentono di fare riferimento a un'istanza di un componente in modo che sia possibile eseguire comandi su tale istanza. Per altre informazioni, vedere <xref:blazor/components/index#capture-references-to-components>.

### `@typeparam`

*Questo scenario si applica solo ai Razor componenti (Razor).*

La `@typeparam` direttiva dichiara un parametro di tipo generico per la classe Component generata. Per altre informazioni, vedere <xref:blazor/components/templated-components>.

::: moniker-end

## <a name="templated-razor-delegates"></a>Delegati basati su modelli Razor

Razor i modelli consentono di definire un frammento di interfaccia utente con il formato seguente:

```cshtml
@<tag>...</tag>
```

Nell'esempio seguente viene illustrato come specificare un delegato basato su modelli Razor come <xref:System.Func%602> . Per il parametro del metodo incapsulato dal delegato viene specificato il [tipo dinamico](/dotnet/csharp/programming-guide/types/using-type-dynamic). Come valore restituito del delegato viene specificato un [tipo di oggetto](/dotnet/csharp/language-reference/keywords/object). Il modello viene usato con un oggetto <xref:System.Collections.Generic.List%601> di `Pet` dotato della proprietà `Name`.

```csharp
public class Pet
{
    public string Name { get; set; }
}
```

```cshtml
@{
    Func<dynamic, object> petTemplate = @<p>You have a pet named <strong>@item.Name</strong>.</p>;

    var pets = new List<Pet>
    {
        new Pet { Name = "Rin Tin Tin" },
        new Pet { Name = "Mr. Bigglesworth" },
        new Pet { Name = "K-9" }
    };
}
```

Il rendering del modello viene eseguito con `pets` in un'istruzione `foreach`:

```cshtml
@foreach (var pet in pets)
{
    @petTemplate(pet)
}
```

Output sottoposto a rendering:

```html
<p>You have a pet named <strong>Rin Tin Tin</strong>.</p>
<p>You have a pet named <strong>Mr. Bigglesworth</strong>.</p>
<p>You have a pet named <strong>K-9</strong>.</p>
```

È anche possibile fornire un modello inline Razor come argomento a un metodo. Nell'esempio seguente il `Repeat` metodo riceve un Razor modello. Il metodo usa il modello per generare contenuto HTML tramite ripetizioni di elementi (item) ricavati da un elenco:

```cshtml
@using Microsoft.AspNetCore.Html

@functions {
    public static IHtmlContent Repeat(IEnumerable<dynamic> items, int times,
        Func<dynamic, IHtmlContent> template)
    {
        var html = new HtmlContentBuilder();

        foreach (var item in items)
        {
            for (var i = 0; i < times; i++)
            {
                html.AppendHtml(template(item));
            }
        }

        return html;
    }
}
```

Usando l'elenco di animali domestici (pets) dell'esempio precedente, il metodo `Repeat` viene chiamato con:

* <xref:System.Collections.Generic.List%601> di `Pet`.
* Numero di ripetizioni di ogni animale domestico.
* Modello inline da usare per gli elementi elenco di un elenco non ordinato.

```cshtml
<ul>
    @Repeat(pets, 3, @<li>@item.Name</li>)
</ul>
```

Output sottoposto a rendering:

```html
<ul>
    <li>Rin Tin Tin</li>
    <li>Rin Tin Tin</li>
    <li>Rin Tin Tin</li>
    <li>Mr. Bigglesworth</li>
    <li>Mr. Bigglesworth</li>
    <li>Mr. Bigglesworth</li>
    <li>K-9</li>
    <li>K-9</li>
    <li>K-9</li>
</ul>
```

## <a name="tag-helpers"></a>Helper tag

*Questo scenario si applica solo a viste e Razor pagine MVC (cshtml).*

Esistono tre direttive che riguardano gli [helper tag](xref:mvc/views/tag-helpers/intro).

| Direttiva | Funzione |
| --------- | -------- |
| [`@addTagHelper`](xref:mvc/views/tag-helpers/intro#add-helper-label) | Rende gli helper tag disponibili per una visualizzazione. |
| [`@removeTagHelper`](xref:mvc/views/tag-helpers/intro#remove-razor-directives-label) | Rimuove gli helper tag aggiunti in precedenza da una visualizzazione. |
| [`@tagHelperPrefix`](xref:mvc/views/tag-helpers/intro#prefix-razor-directives-label) | Specifica un prefisso del tag per abilitare il supporto dell'helper tag e renderne esplicito l'uso. |

## <a name="razor-reserved-keywords"></a>Razor Parole chiave riservate

### <a name="razor-keywords"></a>Razor Parole

* `page` (Richiede ASP.NET Core 2,1 o versione successiva)
* `namespace`
* `functions`
* `inherits`
* `model`
* `section`
* `helper` (Attualmente non supportata da ASP.NET Core)

Razor le parole chiave sono precedute da un carattere di escape `@(Razor Keyword)` (ad esempio, `@(functions)` ).

### <a name="c-razor-keywords"></a>RazorParole chiave di C#

* `case`
* `do`
* `default`
* `for`
* `foreach`
* `if`
* `else`
* `lock`
* `switch`
* `try`
* `catch`
* `finally`
* `using`
* `while`

RazorLe parole chiave di C# devono essere con doppio carattere di escape `@(@C# Razor Keyword)` (ad esempio, `@(@case)` ). Il primo carattere di `@` escape del Razor parser. Il secondo `@` è il carattere di escape del parser C#.

### <a name="reserved-keywords-not-used-by-razor"></a>Parole chiave riservate non usate da Razor

* `class`

## <a name="inspect-the-razor-c-class-generated-for-a-view"></a>Esaminare la Razor classe C# generata per una visualizzazione

::: moniker range=">= aspnetcore-2.1"

Con .NET Core SDK 2,1 o versioni successive, l' [ Razor SDK](xref:razor-pages/sdk) gestisce la compilazione dei Razor file. Quando si compila un progetto, l' Razor SDK genera una directory *obj/<build_configuration>/<target_framework_moniker Razor>/* nella radice del progetto. La struttura di directory all'interno della *Razor* directory rispecchia la struttura di directory del progetto.

Si consideri la struttura di directory seguente in un progetto ASP.NET Core 2,1 Razor pages destinato a .NET Core 2,1:

```
 Areas/
   Admin/
     Pages/
       Index.cshtml
       Index.cshtml.cs
 Pages/
   Shared/
     _Layout.cshtml
   _ViewImports.cshtml
   _ViewStart.cshtml
   Index.cshtml
   Index.cshtml.cs
  ```

La compilazione del progetto nella configurazione *Debug* produce la seguente directory *obj*:

```
 obj/
   Debug/
     netcoreapp2.1/
       Razor/
         Areas/
           Admin/
             Pages/
               Index.g.cshtml.cs
         Pages/
           Shared/
             _Layout.g.cshtml.cs
           _ViewImports.g.cshtml.cs
           _ViewStart.g.cshtml.cs
           Index.g.cshtml.cs
```

Per visualizzare la classe generata per *pages/index. cshtml*, aprire *obj/debug/netcoreapp 2.1/ Razor /pages/index.g.cshtml.cs*.

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

Aggiungere la classe seguente al progetto ASP.NET Core MVC:

[!code-csharp[](razor/sample/Utilities/CustomTemplateEngine.cs)]

In `Startup.ConfigureServices` eseguire l'override dell'elemento `RazorTemplateEngine` aggiunto da MVC con la classe `CustomTemplateEngine`:

[!code-csharp[](razor/sample/Startup.cs?highlight=4&range=10-14)]

Impostare un punto di interruzione per l'istruzione `return csharpDocument;` di `CustomTemplateEngine`. Quando l'esecuzione del programma si interrompe nel punto di interruzione, visualizzare il valore di `generatedCode`.

![Visualizzazione nel visualizzatore testo del codice generato](razor/_static/tvr.png)

::: moniker-end

## <a name="view-lookups-and-case-sensitivity"></a>Visualizzazione di ricerche e distinzione tra maiuscole e minuscole

Il Razor motore di visualizzazione esegue ricerche con distinzione tra maiuscole e minuscole per le visualizzazioni. La ricerca effettiva è tuttavia determinata dal file system sottostante:

* Origine basata su file:
  * Nei sistemi operativi con file system che non fanno distinzione tra maiuscole e minuscole (ad esempio, Windows), le ricerche del provider di file fisici non eseguono la distinzione tra maiuscole e minuscole. Ad esempio, `return View("Test")` comporta corrispondenze per */Views/Home/Test.cshtml*, */Views/home/test.cshtml* e qualsiasi altra variante di maiuscole e minuscole.
  * Nei file system che fanno distinzione tra maiuscole e minuscole (ad esempio Linux, OS x e con `EmbeddedFileProvider`), le ricerche eseguono la distinzione tra maiuscole e minuscole. Ad esempio, `return View("Test")` trova specificamente le corrispondenze per */Views/Home/Test.cshtml*.
* Visualizzazioni precompilate: con ASP.NET Core 2.0 e versioni successive, la ricerca di visualizzazioni precompilate non esegue la distinzione tra maiuscole e minuscole in tutti i sistemi operativi. Questo comportamento è identico al comportamento del provider di file fisici in Windows. Se due visualizzazioni precompilate differiscono solo nelle lettere maiuscole e minuscole, il risultato di ricerca è non deterministico.

Gli sviluppatori sono invitati a far corrispondere le maiuscole e minuscole dei nomi di file e directory per quanto riguarda le maiuscole e minuscole di:

* Nomi di area, controller e azione.
* Razor Pagine.

La distinzione tra maiuscole e minuscole assicura che le distribuzioni trovino le proprie visualizzazioni indipendentemente dal file system sottostante.

## <a name="additional-resources"></a>Risorse aggiuntive

[Introduzione alla programmazione Web ASP.NET con il Razor La sintassi](/aspnet/web-pages/overview/getting-started/introducing-razor-syntax-c) fornisce molti esempi di programmazione con la Razor sintassi.
