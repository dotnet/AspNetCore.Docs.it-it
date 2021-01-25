---
title: BlazorRouting ASP.NET Core
author: guardrex
description: Informazioni su come gestire il routing delle richieste nelle app e su come usare il componente NavLink nelle Blazor app per la navigazione.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/09/2020
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
uid: blazor/fundamentals/routing
ms.openlocfilehash: 74886eae7431045e56de535b5221040bd56cdc2d
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751568"
---
# <a name="aspnet-core-no-locblazor-routing"></a>BlazorRouting ASP.NET Core

Di [Luke Latham](https://github.com/guardrex)

Questo articolo illustra come gestire il routing delle richieste e come usare il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente per creare collegamenti di navigazione nelle Blazor app.

## <a name="route-templates"></a>Modelli di route

Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente consente il routing ai Razor componenti in un' Blazor app. Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente viene usato nel `App` componente delle Blazor app.

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/App1.razor)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/App1.razor)]

::: moniker-end

Quando Razor viene compilato un componente ( `.razor` ) con una [ `@page` direttiva](xref:mvc/views/razor#page) , alla classe del componente generato viene fornito un oggetto che <xref:Microsoft.AspNetCore.Components.RouteAttribute> specifica il modello di route del componente.

All'avvio dell'app, l'assembly specificato come router `AppAssembly` viene analizzato per raccogliere informazioni sulla route per i componenti dell'app che hanno un <xref:Microsoft.AspNetCore.Components.RouteAttribute> .

In fase di esecuzione, il <xref:Microsoft.AspNetCore.Components.RouteView> componente:

* Riceve <xref:Microsoft.AspNetCore.Components.RouteData> da <xref:Microsoft.AspNetCore.Components.Routing.Router> insieme ai parametri di route.
* Esegue il rendering del componente specificato con il [layout](xref:blazor/layouts), inclusi eventuali altri layout annidati.

Facoltativamente, specificare un <xref:Microsoft.AspNetCore.Components.RouteView.DefaultLayout> parametro con una classe layout per i componenti che non specificano un layout con la [ `@layout` direttiva](xref:blazor/layouts#specify-a-layout-in-a-component). I modelli di Blazor progetto del Framework specificano il `MainLayout` componente ( `Shared/MainLayout.razor` ) come layout predefinito dell'app. Per ulteriori informazioni sui layout, vedere <xref:blazor/layouts> .

I componenti supportano più modelli di route usando più [ `@page` direttive](xref:mvc/views/razor#page). Il componente di esempio seguente carica le richieste per `/BlazorRoute` e `/DifferentBlazorRoute` .

`Pages/BlazorRoute.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/BlazorRoute.razor?highlight=1-2)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/BlazorRoute.razor?highlight=1-2)]

::: moniker-end

> [!IMPORTANT]
> Per la corretta risoluzione degli URL, l'app deve includere un `<base>` tag nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ) con il percorso di base dell'applicazione specificato nell' `href` attributo. Per altre informazioni, vedere <xref:blazor/host-and-deploy/index#app-base-path>.

## <a name="provide-custom-content-when-content-isnt-found"></a>Fornire contenuto personalizzato quando il contenuto non è stato trovato

Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente consente all'app di specificare contenuto personalizzato se non viene trovato contenuto per la route richiesta.

Nel `App` componente impostare il contenuto personalizzato nel modello del <xref:Microsoft.AspNetCore.Components.Routing.Router> componente <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> .

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/App2.razor?highlight=5-8)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/App2.razor?highlight=5-8)]

::: moniker-end

Gli elementi arbitrari sono supportati come contenuto dei `<NotFound>` tag, ad esempio altri componenti interattivi. Per applicare un layout predefinito al <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> contenuto, vedere <xref:blazor/layouts#default-layout> .

## <a name="route-to-components-from-multiple-assemblies"></a>Routing a componenti da più assembly

Usare il <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> parametro per specificare gli assembly aggiuntivi da <xref:Microsoft.AspNetCore.Components.Routing.Router> considerare per il componente durante la ricerca di componenti instradabili. Verranno analizzati altri assembly oltre all'assembly specificato in `AppAssembly` . Nell'esempio seguente `Component1` è un componente instradabile definito in una [libreria di classi di componenti](xref:blazor/components/class-libraries)a cui si fa riferimento. Nell' <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> esempio seguente viene restituito il supporto del routing per `Component1` .

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/App3.razor)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/App3.razor)]

::: moniker-end

## <a name="route-parameters"></a>Parametri di route

Il router usa parametri di route per popolare i [parametri del componente](xref:blazor/components/index#component-parameters) corrispondenti con lo stesso nome. Per i nomi dei parametri di route non viene fatta distinzione tra maiuscole Nell'esempio seguente il `text` parametro assegna il valore del segmento di route alla proprietà del componente `Text` . Quando viene effettuata una richiesta per `/RouteParameter/amazing` , `<h1>` viene eseguito il rendering del contenuto del tag come `Blazor is amazing!` .

`Pages/RouteParameter.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/RouteParameter1.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/RouteParameter1.razor?highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

Sono supportati i parametri facoltativi. Nell'esempio seguente, il `text` parametro facoltativo assegna il valore del segmento di route alla proprietà del componente `Text` . Se il segmento non è presente, il valore di `Text` è impostato su `fantastic` .

`Pages/RouteParameter.razor`:

[!code-razor[](routing/samples_snapshot/5.x/RouteParameter2.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

I parametri facoltativi non sono supportati. Nell'esempio seguente vengono applicate due [ `@page` direttive](xref:mvc/views/razor#page) . La prima direttiva consente la navigazione al componente senza un parametro. La seconda direttiva assegna il `{text}` valore del parametro di route alla proprietà del componente `Text` .

`Pages/RouteParameter.razor`:

[!code-razor[](routing/samples_snapshot/3.x/RouteParameter2.razor?highlight=2)]

::: moniker-end

Usare on [`OnParametersSet`](xref:blazor/components/lifecycle#after-parameters-are-set) anziché [`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) per consentire l'esplorazione delle app allo stesso componente con un valore di parametro facoltativo diverso. In base all'esempio precedente, usare `OnParametersSet` quando l'utente deve essere in grado di spostarsi da `/RouteParameter` a `/RouteParameter/amazing` o da `/RouteParameter/amazing` a `/RouteParameter` :

```csharp
protected override void OnParametersSet()
{
    Text = Text ?? "fantastic";
}
```

## <a name="route-constraints"></a>Vincoli di route

Un vincolo di route impone la corrispondenza tra i tipi in un segmento di route e un componente.

Nell'esempio seguente, la route per il `User` componente corrisponde solo a se:

* Un `Id` segmento di route è presente nell'URL della richiesta.
* Il `Id` segmento è un tipo Integer ( `int` ).

`Pages/User.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/User.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/User.razor?highlight=1)]

::: moniker-end

Sono disponibili i vincoli di route indicati nella tabella seguente. Per ulteriori informazioni sui vincoli di route che corrispondono alle impostazioni cultura invarianti, vedere l'avviso sotto la tabella.

| Vincolo | Esempio           | Esempi di corrispondenza                                                                  | Invariante<br>culture<br>corrispondenti |
| ---------- | ----------------- | -------------------------------------------------------------------------------- | :------------------------------: |
| `bool`     | `{active:bool}`   | `true`, `FALSE`                                                                  | No                               |
| `datetime` | `{dob:datetime}`  | `2016-12-31`, `2016-12-31 7:32pm`                                                | Sì                              |
| `decimal`  | `{price:decimal}` | `49.99`, `-1,000.01`                                                             | Sì                              |
| `double`   | `{weight:double}` | `1.234`, `-1,001.01e8`                                                           | Sì                              |
| `float`    | `{weight:float}`  | `1.234`, `-1,001.01e8`                                                           | Sì                              |
| `guid`     | `{id:guid}`       | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | No                               |
| `int`      | `{id:int}`        | `123456789`, `-123456789`                                                        | Sì                              |
| `long`     | `{ticks:long}`    | `123456789`, `-123456789`                                                        | Sì                              |

> [!WARNING]
> I vincoli di route che verificano l'URL e vengono convertiti in un tipo CLR, ad esempio `int` o <xref:System.DateTime>, usano sempre le impostazioni cultura inglese non dipendenti da paese/area geografica, presupponendo che l'URL sia non localizzabile.

## <a name="routing-with-urls-that-contain-dots"></a>Routing con URL contenenti punti

Per Blazor WebAssembly le app e ospitate Blazor Server , il modello di route predefinito sul lato server presuppone che se l'ultimo segmento di un URL della richiesta contiene un punto ( `.` ) richiesto da un file. Ad esempio, l'URL `https://localhost.com:5001/example/some.thing` viene interpretato dal router come una richiesta di un file denominato `some.thing` . Senza alcuna configurazione aggiuntiva, un'app restituisce una risposta *404 non trovata* se `some.thing` è stata progettata per il routing a un componente con una [ `@page` direttiva](xref:mvc/views/razor#page) e `some.thing` è un valore di parametro di route. Per usare una route con uno o più parametri che contengono un punto, l'app deve configurare la route con un modello personalizzato.

Si consideri il `Example` componente seguente che può ricevere un parametro di route dall'ultimo segmento dell'URL.

`Pages/Example.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/Example.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/Example.razor?highlight=2)]

::: moniker-end

Per consentire all' *`Server`* app di una soluzione ospitata Blazor WebAssembly di instradare la richiesta con un punto nel `param` parametro di route, aggiungere un modello di route per il file di fallback con il parametro facoltativo in `Startup.Configure` .

`Startup.cs`:

```csharp
endpoints.MapFallbackToFile("/example/{param?}", "index.html");
```

Per configurare un' Blazor Server app per instradare la richiesta con un punto nel `param` parametro di route, aggiungere un modello di route della pagina di fallback con il parametro facoltativo in `Startup.Configure` .

`Startup.cs`:

```csharp
endpoints.MapFallbackToPage("/example/{param?}", "/_Host");
```

Per altre informazioni, vedere <xref:fundamentals/routing>.

## <a name="catch-all-route-parameters"></a>Parametri di route catch-all

::: moniker range=">= aspnetcore-5.0"

I parametri di route catch-all, che acquisiscono percorsi tra più limiti di cartella, sono supportati nei componenti di.

I parametri di route catch-all sono:

* Denominato in modo che corrisponda al nome del segmento di route. La denominazione non distingue tra maiuscole e minuscole.
* Tipo `string`. Il Framework non fornisce il cast automatico.
* Alla fine dell'URL.

`Pages/CatchAll.razor`:

[!code-razor[](routing/samples_snapshot/5.x/CatchAll.razor)]

Per l'URL `/catch-all/this/is/a/test` con un modello di route di `/catch-all/{*pageRoute}` , il valore di `PageRoute` è impostato su `this/is/a/test` .

Le barre e i segmenti del percorso acquisito vengono decodificati. Per un modello di route di `/catch-all/{*pageRoute}` , l'URL `/catch-all/this/is/a%2Ftest%2A` restituisce `this/is/a/test*` .

::: moniker-end

::: moniker range="< aspnetcore-5.0"

I parametri di route catch-all sono supportati in ASP.NET Core 5,0 o versioni successive. Per ulteriori informazioni, selezionare la versione 5,0 di questo articolo.

::: moniker-end

## <a name="uri-and-navigation-state-helpers"></a>Helper per lo stato di navigazione e URI

Usare <xref:Microsoft.AspNetCore.Components.NavigationManager> per gestire gli URI e la navigazione nel codice C#. <xref:Microsoft.AspNetCore.Components.NavigationManager> fornisce l'evento e i metodi illustrati nella tabella seguente.

| Membro | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri> | Ottiene l'URI assoluto corrente. |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> | Ottiene l'URI di base (con una barra finale) che può essere anteposto ai percorsi URI relativi per produrre un URI assoluto. In genere, <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> corrisponde all' `href` attributo sull'elemento del documento `<base>` in `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ). |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A> | Passa all'URI specificato. Se `forceLoad` è `true` :<ul><li>Il routing lato client viene ignorato.</li><li>Il browser è forzato a caricare la nuova pagina dal server, indipendentemente dal fatto che l'URI venga normalmente gestito dal router lato client.</li></ul> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> | Un evento che viene attivato quando il percorso di navigazione viene modificato. |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.ToAbsoluteUri%2A> | Converte un URI relativo in un URI assoluto. |
| <span style="word-break:normal;word-wrap:normal"><xref:Microsoft.AspNetCore.Components.NavigationManager.ToBaseRelativePath%2A></span> | Dato un URI di base (ad esempio, un URI restituito in precedenza da <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> ), converte un URI assoluto in un URI relativo al prefisso URI di base. |

Per l' <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> evento, <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs> fornisce le informazioni seguenti sugli eventi di navigazione:

* <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.Location>: URL della nuova posizione.
* <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.IsNavigationIntercepted>: Se `true` , Blazor intercetta la navigazione dal browser. Se `false` , <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> ha causato la navigazione.

Il componente seguente:

* Passa al componente dell'app `Counter` ( `Pages/Counter.razor` ) quando si seleziona il pulsante usando <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A> .
* Gestisce l'evento di modifica della posizione sottoscrivendo <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged?displayProperty=nameWithType> .
  * Il `HandleLocationChanged` metodo è unhooked quando `Dispose` viene chiamato dal Framework. L'unhook del metodo consente Garbage Collection del componente.
  * Quando si seleziona il pulsante, l'implementazione del logger registra le informazioni seguenti:

    > `BlazorSample.Pages.Navigate: Information: URL of new location: https://localhost:5001/counter`

`Pages/Navigate.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/Navigate.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/Navigate.razor)]

::: moniker-end

Per ulteriori informazioni sull'eliminazione dei componenti, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable> .

## <a name="query-string-and-parse-parameters"></a>Stringa di query e parametri di analisi

La stringa di query di una richiesta viene ottenuta dalla <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri?displayProperty=nameWithType> proprietà:

```razor
@inject NavigationManager NavigationManager

...

var query = new Uri(NavigationManager.Uri).Query;
```

Per analizzare i parametri della stringa di query:

* Un'app può usare l' <xref:Microsoft.AspNetCore.WebUtilities> API. Se l'API non è disponibile per l'app, aggiungere un riferimento al pacchetto nel file di progetto dell'app per [Microsoft. AspNetCore. webutilities](https://www.nuget.org/packages/Microsoft.AspNetCore.WebUtilities).
* Ottenere il valore dopo aver analizzato la stringa di query con <xref:Microsoft.AspNetCore.WebUtilities.QueryHelpers.ParseQuery%2A?displayProperty=nameWithType> .

Nell' `ParseQueryString` esempio di componente seguente viene analizzata una chiave del parametro della stringa di query denominata `ship` . Ad esempio, la coppia chiave-valore della stringa di query dell'URL `?ship=Tardis` acquisisce il valore `Tardis` in `queryValue` . Per l'esempio seguente, passare all'app con l'URL `https://localhost:5001/parse-query-string?ship=Tardis` .

`Pages/ParseQueryString.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/ParseQueryString.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/ParseQueryString.razor)]

::: moniker-end

## <a name="navlink-and-navmenu-components"></a>`NavLink``NavMenu`componenti e

Usare un <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente al posto di elementi collegamento ipertestuale HTML ( `<a>` ) durante la creazione di collegamenti di navigazione. Un <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente si comporta come un `<a>` elemento, con la differenza che viene attivata o disattivata una `active` classe CSS a seconda che `href` corrisponda all'URL corrente. La `active` classe consente a un utente di comprendere quale pagina è la pagina attiva tra i collegamenti di navigazione visualizzati. Facoltativamente, assegnare un nome di classe CSS a <xref:Microsoft.AspNetCore.Components.Routing.NavLink.ActiveClass?displayProperty=nameWithType> per applicare una classe CSS personalizzata al collegamento di cui è stato eseguito il rendering quando la route corrente corrisponde a `href` .

Il `NavMenu` componente seguente crea una [`Bootstrap`](https://getbootstrap.com/docs/) barra di navigazione che illustra come utilizzare i <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componenti di:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/5.x/NavMenu.razor?highlight=4,9)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](routing/samples_snapshot/3.x/NavMenu.razor?highlight=4,9)]

::: moniker-end

> [!NOTE]
> Il `NavMenu` componente ( `NavMenu.razor` ) viene fornito nella `Shared` cartella di un'app generata dai modelli di Blazor progetto.

Sono disponibili due <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch> opzioni che è possibile assegnare all' `Match` attributo dell' `<NavLink>` elemento:

* <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.All?displayProperty=nameWithType>: <xref:Microsoft.AspNetCore.Components.Routing.NavLink> È attivo quando corrisponde all'intero URL corrente.
* <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.Prefix?displayProperty=nameWithType> (*impostazione predefinita*): l'oggetto <xref:Microsoft.AspNetCore.Components.Routing.NavLink> è attivo quando corrisponde a qualsiasi prefisso dell'URL corrente.

Nell'esempio precedente la Home <xref:Microsoft.AspNetCore.Components.Routing.NavLink> `href=""` corrisponde all'URL Home e riceve solo la `active` classe CSS nell'URL del percorso di base predefinito dell'app (ad esempio, `https://localhost:5001/` ). Il secondo <xref:Microsoft.AspNetCore.Components.Routing.NavLink> riceve la `active` classe quando l'utente visita un URL con un `component` prefisso, ad esempio `https://localhost:5001/component` e `https://localhost:5001/component/another-segment` .

<xref:Microsoft.AspNetCore.Components.Routing.NavLink>Gli attributi dei componenti aggiuntivi vengono passati al tag di ancoraggio sottoposto a rendering. Nell'esempio seguente il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente include l' `target` attributo:

```razor
<NavLink href="example-page" target="_blank">Example page</NavLink>
```

Viene eseguito il rendering del markup HTML seguente:

```html
<a href="example-page" target="_blank">Example page</a>
```

> [!WARNING]
> A causa del modo in cui Blazor viene eseguito il rendering del contenuto figlio, `NavLink` i componenti di rendering all'interno di un `for` ciclo richiedono una variabile di indice locale se la variabile del ciclo di incremento viene utilizzata nel `NavLink` contenuto del componente (figlio):
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @current
>         </NavLink>
>     </li>
> }
> ```
>
> L'uso di una variabile di indice in questo scenario è un requisito per **qualsiasi** componente figlio che usa una variabile di ciclo nel [contenuto figlio](xref:blazor/components/index#child-content), non solo nel `NavLink` componente.
>
> In alternativa, usare un `foreach` ciclo con <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType> :
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @c
>         </NavLink>
>     </li>
> }
> ```

## <a name="aspnet-core-endpoint-routing-integration"></a>Integrazione di routing endpoint ASP.NET Core

*Questa sezione si applica solo alle Blazor Server app.*

Blazor Server è integrato in [ASP.net core routing dell'endpoint](xref:fundamentals/routing). Un'app ASP.NET Core è configurata in modo da accettare le connessioni in ingresso per i componenti interattivi con <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> in `Startup.Configure` .

`Startup.cs`:

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](routing/samples_snapshot/5.x/Startup.cs?highlight=5)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](routing/samples_snapshot/3.x/Startup.cs?highlight=5)]

::: moniker-end

La configurazione tipica consiste nel routing di tutte le richieste a una Razor pagina, che funge da host per la parte lato server dell' Blazor Server app. Per convenzione, la pagina *host* viene in genere denominata `_Host.cshtml` nella `Pages` cartella dell'app.

La route specificata nel file host viene chiamata route di *fallback* perché funziona con una priorità bassa nella corrispondenza della route. La route di fallback viene utilizzata quando altre route non corrispondono. Ciò consente all'app di usare altri controller e pagine senza interferire con il routing dei componenti nell' Blazor Server app.

Per informazioni sulla configurazione <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A> di per l'hosting di server URL non radice, vedere <xref:blazor/host-and-deploy/index#app-base-path> .
