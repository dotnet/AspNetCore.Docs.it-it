---
title: BlazorGlobalizzazione e localizzazione ASP.NET Core
author: guardrex
description: Informazioni su come rendere i Razor componenti accessibili agli utenti in più impostazioni cultura e lingue.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/04/2020
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
uid: blazor/globalization-localization
ms.openlocfilehash: f8f261f25d854a9bf36ad3299f4af392d5c4fafd
ms.sourcegitcommit: c1839f2992b003c92cd958244a2e0771ae928786
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/05/2021
ms.locfileid: "93055880"
---
# <a name="aspnet-core-blazor-globalization-and-localization"></a>BlazorGlobalizzazione e localizzazione ASP.NET Core

Di [Luke Latham](https://github.com/guardrex) e [Daniel Roth](https://github.com/danroth27)

Razor i componenti possono essere resi accessibili agli utenti in più impostazioni cultura e lingue. Sono disponibili i seguenti scenari di globalizzazione e localizzazione di .NET:

* . Sistema di risorse di NET
* Formattazione di numeri e date specifiche delle impostazioni cultura

Sono attualmente supportati un set limitato di scenari di localizzazione di ASP.NET Core:

* <xref:Microsoft.Extensions.Localization.IStringLocalizer> e <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> *sono supportati* nelle Blazor app.
* <xref:Microsoft.AspNetCore.Mvc.Localization.IHtmlLocalizer>la <xref:Microsoft.AspNetCore.Mvc.Localization.IViewLocalizer> localizzazione delle annotazioni dei dati, e è ASP.NET Core scenari MVC e **non è supportata** nelle Blazor app.

Per altre informazioni, vedere <xref:fundamentals/localization>.

## <a name="globalization"></a>Globalizzazione

Blazorla [`@bind`](xref:mvc/views/razor#bind) funzionalità di esegue formati e analizza i valori per la visualizzazione in base alle impostazioni cultura correnti dell'utente.

È possibile accedere alle impostazioni cultura correnti dalla <xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName> Proprietà.

<xref:System.Globalization.CultureInfo.InvariantCulture?displayProperty=nameWithType> viene usato per i tipi di campo seguenti ( `<input type="{TYPE}" />` ):

* `date`
* `number`

I tipi di campo precedenti:

* Vengono visualizzati utilizzando le regole di formattazione appropriate basate su browser.
* Non può contenere testo in formato libero.
* Fornire le caratteristiche di interazione utente in base all'implementazione del browser.

I tipi di campo seguenti hanno requisiti di formattazione specifici e non sono attualmente supportati da perché non sono Blazor supportati da tutti i browser principali:

* `datetime-local`
* `month`
* `week`

[`@bind`](xref:mvc/views/razor#bind) supporta il `@bind:culture` parametro per fornire un oggetto <xref:System.Globalization.CultureInfo?displayProperty=fullName> per l'analisi e la formattazione di un valore. Non è consigliabile specificare impostazioni cultura quando si usano i `date` `number` tipi di campo e. `date` e `number` dispongono di supporto incorporato Blazor che fornisce le impostazioni cultura richieste.

## <a name="localization"></a>Localizzazione

### Blazor WebAssembly

Blazor WebAssembly le app impostano le impostazioni cultura usando le [Preferenze della lingua](https://developer.mozilla.org/docs/Web/API/NavigatorLanguage/languages)dell'utente.

Per configurare in modo esplicito le impostazioni cultura, impostare <xref:System.Globalization.CultureInfo.DefaultThreadCurrentCulture?displayProperty=nameWithType> e <xref:System.Globalization.CultureInfo.DefaultThreadCurrentUICulture?displayProperty=nameWithType> in `Program.Main` .

::: moniker range=">= aspnetcore-5.0"

Per impostazione predefinita, Blazor WebAssembly contiene risorse di globalizzazione minime necessarie per visualizzare i valori, ad esempio date e valute, nelle impostazioni cultura dell'utente. Le applicazioni che devono supportare la modifica dinamica delle impostazioni cultura devono essere configurate `BlazorWebAssemblyLoadAllGlobalizationData` nel file di progetto:

```xml
<PropertyGroup>
  <BlazorWebAssemblyLoadAllGlobalizationData>true</BlazorWebAssemblyLoadAllGlobalizationData>
</PropertyGroup>
```

Blazor WebAssembly può anche essere configurato per avviare usando impostazioni cultura specifiche dell'applicazione usando le opzioni passate a `Blazor.start` . Ad esempio, l'esempio seguente mostra un'app configurata per l'avvio usando le `en-GB` impostazioni cultura:

```html
<script src="_framework/blazor.webassembly.js" autostart="false"></script>
<script>
  Blazor.start({
    applicationCulture: 'en-GB'
  });
</script>
```

Il valore per `applicationCulture` deve essere conforme al [formato di tag del linguaggio BCP-47](https://tools.ietf.org/html/bcp47).

Se l'app non richiede la localizzazione, è possibile configurare l'app in modo che supporti la lingua inglese, che è basata sulle `en-US` impostazioni cultura:

```xml
<PropertyGroup>
  <InvariantGlobalization>true</InvariantGlobalization>
</PropertyGroup>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Per impostazione predefinita, la configurazione del linker linguaggio intermedio (IL) per le Blazor WebAssembly app rimuove le informazioni di internazionalizzazione ad eccezione delle impostazioni locali richieste in modo esplicito. Per altre informazioni, vedere <xref:blazor/host-and-deploy/configure-linker#configure-the-linker-for-internationalization>.

::: moniker-end

Mentre le impostazioni cultura che Blazor selezionano per impostazione predefinita potrebbero essere sufficienti per la maggior parte degli utenti, è consigliabile offrire agli utenti un modo per specificare le impostazioni locali preferite. Per un' Blazor WebAssembly app di esempio con selezione impostazioni cultura, vedere l' [`LocSample`](https://github.com/pranavkm/LocSample) app di esempio localizzazione.

### Blazor Server

Blazor Server le app vengono localizzate usando il [middleware di localizzazione](xref:fundamentals/localization#localization-middleware). Il middleware seleziona le impostazioni cultura appropriate per gli utenti che richiedono risorse dall'app.

Le impostazioni cultura possono essere impostate utilizzando uno degli approcci seguenti:

* [Cookies](#cookies)
* [Fornire l'interfaccia utente per scegliere le impostazioni cultura](#provide-ui-to-choose-the-culture)

Per altre informazioni ed esempi, vedere <xref:fundamentals/localization>.

#### <a name="cookies"></a>Cookies

Le impostazioni cultura cookie della localizzazione possono salvare in maniera permanente le impostazioni cultura dell'utente. Il middleware di localizzazione legge l'oggetto cookie sulle richieste successive per impostare le impostazioni cultura dell'utente. 

L'uso di un oggetto cookie garantisce che la connessione WebSocket possa propagare correttamente le impostazioni cultura. Se gli schemi di localizzazione sono basati sul percorso dell'URL o sulla stringa di query, lo schema potrebbe non essere in grado di funzionare con WebSocket, quindi non è possibile salvare in modo permanente le impostazioni cultura. Pertanto, l'utilizzo delle impostazioni cultura di localizzazione cookie è l'approccio consigliato.

Qualsiasi tecnica può essere utilizzata per assegnare impostazioni cultura se le impostazioni cultura vengono rese permanente in una localizzazione cookie . Se l'app ha già uno schema di localizzazione stabilito per ASP.NET Core lato server, continuare a usare l'infrastruttura di localizzazione esistente dell'app e impostare le impostazioni cultura di localizzazione nello cookie schema dell'app.

Nell'esempio seguente viene illustrato come impostare le impostazioni cultura correnti in un oggetto cookie che può essere letto dal middleware di localizzazione. Creare un' Razor espressione nel `Pages/_Host.cshtml` file immediatamente all'interno del tag di apertura `<body>` :

```cshtml
@using System.Globalization
@using Microsoft.AspNetCore.Localization

...

<body>
    @{
        this.HttpContext.Response.Cookies.Append(
            CookieRequestCultureProvider.DefaultCookieName,
            CookieRequestCultureProvider.MakeCookieValue(
                new RequestCulture(
                    CultureInfo.CurrentCulture,
                    CultureInfo.CurrentUICulture)));
    }

    ...
</body>
```

La localizzazione viene gestita dall'app nella sequenza di eventi seguente:

1. Il browser invia una richiesta HTTP iniziale all'app.
1. Le impostazioni cultura vengono assegnate dal middleware di localizzazione.
1. L' Razor espressione nella `_Host` pagina ( `_Host.cshtml` ) rende permanente le impostazioni cultura in un cookie come parte della risposta.
1. Il browser apre una connessione WebSocket per creare una Blazor Server sessione interattiva.
1. Il middleware di localizzazione legge cookie e assegna le impostazioni cultura.
1. La Blazor Server sessione inizia con le impostazioni cultura corrette.

Quando si utilizza un <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage> , utilizzare la <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.Context> proprietà:

```razor
@{
    this.Context.Response.Cookies.Append(
        CookieRequestCultureProvider.DefaultCookieName,
        CookieRequestCultureProvider.MakeCookieValue(
            new RequestCulture(
                CultureInfo.CurrentCulture,
                CultureInfo.CurrentUICulture)));
}
```

#### <a name="provide-ui-to-choose-the-culture"></a>Fornire l'interfaccia utente per scegliere le impostazioni cultura

Per consentire a un utente di selezionare le impostazioni cultura, è consigliabile un *approccio basato su Reindirizzamento* . Il processo è simile a quello che accade in un'app Web quando un utente tenta di accedere a una risorsa protetta. L'utente viene reindirizzato a una pagina di accesso, quindi viene reindirizzato di nuovo alla risorsa originale. 

L'app rende permanente le impostazioni cultura selezionate dall'utente tramite un reindirizzamento a un controller. Il controller imposta le impostazioni cultura selezionate dall'utente in un cookie e reindirizza l'utente all'URI originale.

Stabilire un endpoint HTTP nel server per impostare le impostazioni cultura selezionate dall'utente in un oggetto cookie ed eseguire di nuovo il reindirizzamento all'URI originale:

```csharp
[Route("[controller]/[action]")]
public class CultureController : Controller
{
    public IActionResult SetCulture(string culture, string redirectUri)
    {
        if (culture != null)
        {
            HttpContext.Response.Cookies.Append(
                CookieRequestCultureProvider.DefaultCookieName,
                CookieRequestCultureProvider.MakeCookieValue(
                    new RequestCulture(culture)));
        }

        return LocalRedirect(redirectUri);
    }
}
```

> [!WARNING]
> Usare il <xref:Microsoft.AspNetCore.Mvc.ControllerBase.LocalRedirect%2A> risultato dell'azione per impedire gli attacchi di reindirizzamento aperti. Per altre informazioni, vedere <xref:security/preventing-open-redirects>.

Se l'app non è configurata per l'elaborazione delle azioni del controller:

* Aggiungere i servizi MVC alla raccolta di servizi in `Startup.ConfigureServices` :

  ```csharp
  services.AddControllers();
  ```

* Aggiungi routing endpoint controller in `Startup.Configure` :

  ```csharp
  app.UseEndpoints(endpoints =>
  {
      endpoints.MapControllers();
      endpoints.MapBlazorHub();
      endpoints.MapFallbackToPage("/_Host");
  });
  ```

Il componente seguente mostra un esempio di come eseguire il reindirizzamento iniziale quando l'utente seleziona le impostazioni cultura:

```razor
@inject NavigationManager NavigationManager

<h3>Select your language</h3>

<select @onchange="OnSelected">
    <option>Select...</option>
    <option value="en-US">English</option>
    <option value="fr-FR">Français</option>
</select>

@code {
    private void OnSelected(ChangeEventArgs e)
    {
        var culture = (string)e.Value;
        var uri = new Uri(NavigationManager.Uri)
            .GetComponents(UriComponents.PathAndQuery, UriFormat.Unescaped);
        var query = $"?culture={Uri.EscapeDataString(culture)}&" +
            $"redirectUri={Uri.EscapeDataString(uri)}";

        NavigationManager.NavigateTo("/Culture/SetCulture" + query, forceLoad: true);
    }
}
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/localization>
