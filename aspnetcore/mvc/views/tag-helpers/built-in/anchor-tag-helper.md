---
title: Helper tag di ancoraggio in ASP.NET Core
author: pkellner
description: Informazioni sugli attributi dell'helper tag di ancoraggio ASP.NET Core e sul ruolo di ogni attributo per l'estensione del comportamento del tag di ancoraggio HTML.
ms.author: scaddie
ms.custom: mvc
ms.date: 10/13/2019
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
uid: mvc/views/tag-helpers/builtin-th/anchor-tag-helper
ms.openlocfilehash: 2e49c545b0d343475ce44a636a6ae66324f9d3bf
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587203"
---
# <a name="anchor-tag-helper-in-aspnet-core"></a>Helper tag di ancoraggio in ASP.NET Core

Di [Peter Kellner](https://peterkellner.net) e [Scott Addie](https://github.com/scottaddie)

L'[helper tag](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper) di ancoraggio migliora il tag di ancoraggio HTML standard (`<a ... ></a>`) con l'aggiunta di nuovi attributi. Per convenzione, i nomi di attributo hanno il prefisso `asp-`. Il valore dell'attributo `href` dell'elemento di ancoraggio visualizzato dipende dai valori degli attributi `asp-`.

Per una panoramica degli helper per tag, vedere <xref:mvc/views/tag-helpers/intro>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/built-in/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

*SpeakerController* viene usato negli esempi in tutto il documento:

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?name=snippet_SpeakerController)]

## <a name="anchor-tag-helper-attributes"></a>Attributi dell'helper tag di ancoraggio

### <a name="asp-controller"></a>asp-controller

L'attributo [asp-controller](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Controller*) assegna il controller usato per generare l'URL. Il markup seguente elenca tutti gli altoparlanti:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspController)]

Codice HTML generato:

```html
<a href="/Speaker">All Speakers</a>
```

Se si specifica l'attributo `asp-controller` e l'attributo `asp-action` viene omesso, il valore `asp-action` predefinito è l'azione del controller associata alla visualizzazione di esecuzione corrente. Se `asp-action` viene omesso dal markup precedente e viene usato l'helper tag di ancoraggio nella visualizzazione *Index* di *HomeController* (*/Home*), il codice HTML generato è:

```html
<a href="/Home">All Speakers</a>
```

### <a name="asp-action"></a>asp-action

Il valore dell'attributo [asp-action](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Action*) rappresenta il nome dell'azione del controller incluso nell'attributo `href` generato. Il markup seguente imposta il valore dell'attributo `href` generato sulla pagina delle valutazioni degli altoparlanti:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAction)]

Codice HTML generato:

```html
<a href="/Speaker/Evaluations">Speaker Evaluations</a>
```

Se non viene specificato alcun attributo `asp-controller` viene usato il controller predefinito per chiamare la visualizzazione che esegue la visualizzazione corrente.

Se il valore dell'attributo `asp-action` è `Index` non viene accodata alcuna azione all'URL, con conseguente chiamata dell'azione `Index` predefinita. L'azione specificata (o usata per impostazione predefinita) deve esistere nel controller a cui si fa riferimento in `asp-controller`.

### <a name="asp-route-value"></a>asp-route-{value}

L'attributo [asp-route-{value}](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.RouteValues*) abilita un prefisso di route con caratteri jolly. Qualsiasi valore che sostituisce il segnaposto `{value}` viene interpretato come potenziale parametro di route. Se non viene trovata una route predefinita, il prefisso di route viene accodato come parametro di richiesta e relativo valore all'attributo `href` generato. In caso contrario viene sostituito nel modello di route.

Esaminare l'azione del controller seguente:

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/BuiltInTagController.cs?name=snippet_AnchorTagHelperAction)]

Con un modello di route predefinito definito in *Startup.Configure*:

[!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_UseMvc&highlight=8-10)]

La visualizzazione MVC usa il modello, fornito dall'azione, come segue:

```cshtml
@model Speaker
<!DOCTYPE html>
<html>
<body>
    <a asp-controller="Speaker"
       asp-action="Detail" 
       asp-route-id="@Model.SpeakerId">SpeakerId: @Model.SpeakerId</a>
</body>
</html>
```

È stata trovata una corrispondenza per il segnaposto `{id?}` della route predefinita. Codice HTML generato:

```html
<a href="/Speaker/Detail/12">SpeakerId: 12</a>
```

Si supponga che il prefisso di route non faccia parte del modello di routing corrispondente, come nel caso della visualizzazione MVC seguente:

```cshtml
@model Speaker
<!DOCTYPE html>
<html>
<body>
    <a asp-controller="Speaker"
       asp-action="Detail"
       asp-route-speakerid="@Model.SpeakerId">SpeakerId: @Model.SpeakerId</a>
<body>
</html>
```

Il codice HTML seguente viene generato perché `speakerid` non è stato trovato nella route corrispondente:

```html
<a href="/Speaker/Detail?speakerid=12">SpeakerId: 12</a>
```

Se `asp-controller` o `asp-action` non vengono specificati, viene eseguita la stessa elaborazione predefinita usata nell'attributo `asp-route`.

### <a name="asp-route"></a>asp-route

L'attributo [asp-route](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Route*) viene usato per la creazione di un URL collegato direttamente a una route denominata. Mediante gli [attributi di routing](xref:mvc/controllers/routing#attribute-routing) è possibile assegnare un nome a una route come mostrato in `SpeakerController` e usarla nell'azione `Evaluations` corrispondente:

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?range=22-24)]

Nel markup seguente l'attributo `asp-route` fa riferimento alla route denominata:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspRoute)]

L'helper tag di ancoraggio genera una route direttamente per tale azione del controller usando l'URL */Speaker/Evaluations*. Codice HTML generato:

```html
<a href="/Speaker/Evaluations">Speaker Evaluations</a>
```

Se è specificato `asp-controller` o `asp-action` in aggiunta a `asp-route`, la route generata potrebbe non essere quella prevista. Per evitare un conflitto di route, è consigliabile non usare `asp-route` con gli attributi `asp-controller` o `asp-action`.

### <a name="asp-all-route-data"></a>asp-all-route-data

L'attributo [asp-all-route-data](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.RouteValues*) supporta la creazione di un dizionario di coppie chiave-valore. La chiave è il nome del parametro e il valore è il valore del parametro.

Nell'esempio seguente un dizionario viene inizializzato e passato a una Razor visualizzazione. In alternativa è possibile passare i dati con il modello.

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAllRouteData)]

Il codice precedente genera il codice HTML seguente:

```html
<a href="/Speaker/EvaluationsCurrent?speakerId=11&currentYear=true">Speaker Evaluations</a>
```

Il dizionario `asp-all-route-data` viene reso flat per produrre una stringa di query corrispondente ai requisiti dell'azione `Evaluations` in overload:

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?range=26-30)]

In presenza di chiavi nel dizionario corrispondenti ai parametri di route, questi valori vengono sostituiti nella route come appropriato. Gli altri valori non corrispondenti vengono generati come parametri di richiesta.

### <a name="asp-fragment"></a>asp-fragment

L'attributo [asp-fragment](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Fragment*) definisce un frammento di URL da accodare all'URL. L'helper tag di ancoraggio aggiunge il carattere hash (#). Considerare il markup seguente:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspFragment)]

Codice HTML generato:

```html
<a href="/Speaker/Evaluations#SpeakerEvaluations">Speaker Evaluations</a>
```

I tag hash sono utili per la compilazione di app sul lato client. Ad esempio possono essere usati per semplificare l'uso di contrassegni e la ricerca in JavaScript.

### <a name="asp-area"></a>asp-area

L'attributo [asp-area](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Area*) imposta il nome dell'area usato per impostare la route appropriata. Gli esempi seguenti illustrano come l'attributo `asp-area` determina la modifica del mapping delle route.

#### <a name="usage-in-razor-pages"></a>Utilizzo nelle Razor pagine

Razor Le aree delle pagine sono supportate in ASP.NET Core 2,1 o versioni successive.

Considerare la gerarchia di directory seguente:

* **{Nome progetto}**
  * **wwwroot**
  * **Aree**
    * **Sessioni**
      * **Pagine**
        * *\_ViewStart. cshtml*
        * *Index.cshtml*
        * *Index.cshtml.cs*
  * **Pagine**

Il markup per fare riferimento alla pagina di *Indice* dell'area *sessioni* Razor è:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAreaRazorPages)]

Codice HTML generato:

```html
<a href="/Sessions">View Sessions</a>
```

> [!TIP]
> Per supportare le aree in un' Razor app pagine, effettuare una delle operazioni seguenti in `Startup.ConfigureServices` :
>
> * Impostare la [versione di compatibilità](xref:mvc/compatibility-version) su 2.1 o successiva.
> * Impostare la proprietà [ Razor PagesOptions. AllowAreas](xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.AllowAreas*) su `true` :
>
>   [!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_AllowAreas)]

#### <a name="usage-in-mvc"></a>Utilizzo in MVC

Considerare la gerarchia di directory seguente:

* **{Nome progetto}**
  * **wwwroot**
  * **Aree**
    * **Blog**
      * **Controllers**
        * *HomeController.cs*
      * **Visualizzazioni**
        * **Home**
          * *AboutBlog.cshtml*
          * *Index.cshtml*
        * *\_ViewStart. cshtml*
  * **Controllers**

Se si imposta `asp-area` su "Blogs", la directory *Areas/Blogs* viene aggiunta come prefisso alle route dei controller e delle visualizzazioni associati per questo tag di ancoraggio. Il markup per fare riferimento alla visualizzazione *AboutBlog* è:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspArea)]

Codice HTML generato:

```html
<a href="/Blogs/Home/AboutBlog">About Blog</a>
```

> [!TIP]
> Per supportare le aree in un'app MVC, il modello di route deve includere un riferimento all'area, se esistente. Tale modello è rappresentato dal secondo parametro della `routes.MapRoute` chiamata al metodo in *Startup.ConfigUre*:
>
> [!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_UseMvc&highlight=5)]

### <a name="asp-protocol"></a>asp-protocol

L'attributo [asp-protocol](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Protocol*) consente di specificare un protocollo (ad esempio `https`) nell'URL. Ad esempio:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspProtocol)]

Codice HTML generato:

```html
<a href="https://localhost/Home/About">About</a>
```

Il nome host nell'esempio è localhost. L'helper tag di ancoraggio usa il dominio pubblico del sito Web per generare l'URL.

### <a name="asp-host"></a>asp-host

L'attributo [asp-host](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Host*) consente di specificare un nome host nell'URL. Ad esempio:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspHost)]

Codice HTML generato:

```html
<a href="https://microsoft.com/Home/About">About</a>
```

### <a name="asp-page"></a>asp-page

L'attributo [asp-page](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Page*) viene utilizzato con le Razor pagine. Usarlo per impostare il valore dell'attributo `href` per un tag di ancoraggio su una pagina specifica. L'URL viene creato anteponendo una barra ("/") al nome della pagina.

L'esempio seguente fa riferimento alla pagina partecipante Razor :

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPage)]

Codice HTML generato:

```html
<a href="/Attendee">All Attendees</a>
```

L'attributo `asp-page` e gli attributi `asp-route`, `asp-controller` e `asp-action` si escludono a vicenda. `asp-page` può tuttavia essere usato con `asp-route-{value}` per il controllo del routing, come dimostra il markup seguente:

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPageAspRouteId)]

Codice HTML generato:

```html
<a href="/Attendee?attendeeid=10">View Attendee</a>
```

### <a name="asp-page-handler"></a>asp-page-handler

L'attributo [ASP-Page-Handler](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.PageHandler*) viene utilizzato con le Razor pagine. È progettato per il collegamento di gestori di pagine specifici.

Si consideri il gestore di pagine seguente:

[!code-csharp[](samples/TagHelpersBuiltIn/Pages/Attendee.cshtml.cs?name=snippet_OnGetProfileHandler)]

Il markup associato del modello di pagina si collega al gestore di pagina `OnGetProfile`. Si noti che il prefisso `On<Verb>` del nome del metodo del gestore di pagina viene omesso nel valore dell'attributo `asp-page-handler`. Quando il metodo è asincrono, viene omesso anche il suffisso `Async`.

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPageHandler)]

Codice HTML generato:

```html
<a href="/Attendee?attendeeid=12&handler=Profile">Attendee Profile</a>
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
