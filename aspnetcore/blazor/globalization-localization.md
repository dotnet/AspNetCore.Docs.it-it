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
ms.openlocfilehash: 8baee9bc0a8e569174f33dac6a406b2162d09552
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279908"
---
# <a name="aspnet-core-blazor-globalization-and-localization"></a><span data-ttu-id="f53fa-103">BlazorGlobalizzazione e localizzazione ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="f53fa-103">ASP.NET Core Blazor globalization and localization</span></span>

<span data-ttu-id="f53fa-104">Razor i componenti possono essere resi accessibili agli utenti in più impostazioni cultura e lingue.</span><span class="sxs-lookup"><span data-stu-id="f53fa-104">Razor components can be made accessible to users in multiple cultures and languages.</span></span> <span data-ttu-id="f53fa-105">Sono disponibili i seguenti scenari di globalizzazione e localizzazione di .NET:</span><span class="sxs-lookup"><span data-stu-id="f53fa-105">The following .NET globalization and localization scenarios are available:</span></span>

* <span data-ttu-id="f53fa-106">. Sistema di risorse di NET</span><span class="sxs-lookup"><span data-stu-id="f53fa-106">.NET's resources system</span></span>
* <span data-ttu-id="f53fa-107">Formattazione di numeri e date specifiche delle impostazioni cultura</span><span class="sxs-lookup"><span data-stu-id="f53fa-107">Culture-specific number and date formatting</span></span>

<span data-ttu-id="f53fa-108">Sono attualmente supportati un set limitato di scenari di localizzazione di ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="f53fa-108">A limited set of ASP.NET Core's localization scenarios are currently supported:</span></span>

* <span data-ttu-id="f53fa-109"><xref:Microsoft.Extensions.Localization.IStringLocalizer> e <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> *sono supportati* nelle Blazor app.</span><span class="sxs-lookup"><span data-stu-id="f53fa-109"><xref:Microsoft.Extensions.Localization.IStringLocalizer> and <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> *are supported* in Blazor apps.</span></span>
* <span data-ttu-id="f53fa-110"><xref:Microsoft.AspNetCore.Mvc.Localization.IHtmlLocalizer>la <xref:Microsoft.AspNetCore.Mvc.Localization.IViewLocalizer> localizzazione delle annotazioni dei dati, e è ASP.NET Core scenari MVC e **non è supportata** nelle Blazor app.</span><span class="sxs-lookup"><span data-stu-id="f53fa-110"><xref:Microsoft.AspNetCore.Mvc.Localization.IHtmlLocalizer>, <xref:Microsoft.AspNetCore.Mvc.Localization.IViewLocalizer>, and Data Annotations localization are ASP.NET Core MVC scenarios and **not supported** in Blazor apps.</span></span>

<span data-ttu-id="f53fa-111">Per altre informazioni, vedere <xref:fundamentals/localization>.</span><span class="sxs-lookup"><span data-stu-id="f53fa-111">For more information, see <xref:fundamentals/localization>.</span></span>

## <a name="globalization"></a><span data-ttu-id="f53fa-112">Globalizzazione</span><span class="sxs-lookup"><span data-stu-id="f53fa-112">Globalization</span></span>

<span data-ttu-id="f53fa-113">Blazorla [`@bind`](xref:mvc/views/razor#bind) funzionalità di esegue formati e analizza i valori per la visualizzazione in base alle impostazioni cultura correnti dell'utente.</span><span class="sxs-lookup"><span data-stu-id="f53fa-113">Blazor's [`@bind`](xref:mvc/views/razor#bind) functionality performs formats and parses values for display based on the user's current culture.</span></span>

<span data-ttu-id="f53fa-114">È possibile accedere alle impostazioni cultura correnti dalla <xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName> Proprietà.</span><span class="sxs-lookup"><span data-stu-id="f53fa-114">The current culture can be accessed from the <xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName> property.</span></span>

<span data-ttu-id="f53fa-115"><xref:System.Globalization.CultureInfo.InvariantCulture?displayProperty=nameWithType> viene usato per i tipi di campo seguenti ( `<input type="{TYPE}" />` ):</span><span class="sxs-lookup"><span data-stu-id="f53fa-115"><xref:System.Globalization.CultureInfo.InvariantCulture?displayProperty=nameWithType> is used for the following field types (`<input type="{TYPE}" />`):</span></span>

* `date`
* `number`

<span data-ttu-id="f53fa-116">I tipi di campo precedenti:</span><span class="sxs-lookup"><span data-stu-id="f53fa-116">The preceding field types:</span></span>

* <span data-ttu-id="f53fa-117">Vengono visualizzati utilizzando le regole di formattazione appropriate basate su browser.</span><span class="sxs-lookup"><span data-stu-id="f53fa-117">Are displayed using their appropriate browser-based formatting rules.</span></span>
* <span data-ttu-id="f53fa-118">Non può contenere testo in formato libero.</span><span class="sxs-lookup"><span data-stu-id="f53fa-118">Can't contain free-form text.</span></span>
* <span data-ttu-id="f53fa-119">Fornire le caratteristiche di interazione utente in base all'implementazione del browser.</span><span class="sxs-lookup"><span data-stu-id="f53fa-119">Provide user interaction characteristics based on the browser's implementation.</span></span>

<span data-ttu-id="f53fa-120">I tipi di campo seguenti hanno requisiti di formattazione specifici e non sono attualmente supportati da perché non sono Blazor supportati da tutti i browser principali:</span><span class="sxs-lookup"><span data-stu-id="f53fa-120">The following field types have specific formatting requirements and aren't currently supported by Blazor because they aren't supported by all major browsers:</span></span>

* `datetime-local`
* `month`
* `week`

<span data-ttu-id="f53fa-121">[`@bind`](xref:mvc/views/razor#bind) supporta il `@bind:culture` parametro per fornire un oggetto <xref:System.Globalization.CultureInfo?displayProperty=fullName> per l'analisi e la formattazione di un valore.</span><span class="sxs-lookup"><span data-stu-id="f53fa-121">[`@bind`](xref:mvc/views/razor#bind) supports the `@bind:culture` parameter to provide a <xref:System.Globalization.CultureInfo?displayProperty=fullName> for parsing and formatting a value.</span></span> <span data-ttu-id="f53fa-122">Non è consigliabile specificare impostazioni cultura quando si usano i `date` `number` tipi di campo e.</span><span class="sxs-lookup"><span data-stu-id="f53fa-122">Specifying a culture isn't recommended when using the `date` and `number` field types.</span></span> <span data-ttu-id="f53fa-123">`date` e `number` dispongono di supporto incorporato Blazor che fornisce le impostazioni cultura richieste.</span><span class="sxs-lookup"><span data-stu-id="f53fa-123">`date` and `number` have built-in Blazor support that provides the required culture.</span></span>

## <a name="localization"></a><span data-ttu-id="f53fa-124">Localizzazione</span><span class="sxs-lookup"><span data-stu-id="f53fa-124">Localization</span></span>

### Blazor WebAssembly

<span data-ttu-id="f53fa-125">Blazor WebAssembly le app impostano le impostazioni cultura usando le [Preferenze della lingua](https://developer.mozilla.org/docs/Web/API/NavigatorLanguage/languages)dell'utente.</span><span class="sxs-lookup"><span data-stu-id="f53fa-125">Blazor WebAssembly apps set the culture using the user's [language preference](https://developer.mozilla.org/docs/Web/API/NavigatorLanguage/languages).</span></span>

<span data-ttu-id="f53fa-126">Per configurare in modo esplicito le impostazioni cultura, impostare <xref:System.Globalization.CultureInfo.DefaultThreadCurrentCulture?displayProperty=nameWithType> e <xref:System.Globalization.CultureInfo.DefaultThreadCurrentUICulture?displayProperty=nameWithType> in `Program.Main` .</span><span class="sxs-lookup"><span data-stu-id="f53fa-126">To explicitly configure the culture, set <xref:System.Globalization.CultureInfo.DefaultThreadCurrentCulture?displayProperty=nameWithType> and <xref:System.Globalization.CultureInfo.DefaultThreadCurrentUICulture?displayProperty=nameWithType> in `Program.Main`.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="f53fa-127">Per impostazione predefinita, Blazor WebAssembly contiene risorse di globalizzazione minime necessarie per visualizzare i valori, ad esempio date e valute, nelle impostazioni cultura dell'utente.</span><span class="sxs-lookup"><span data-stu-id="f53fa-127">By default, Blazor WebAssembly carries minimal globalization resources required to display values, such as dates and currency, in the user's culture.</span></span> <span data-ttu-id="f53fa-128">Le applicazioni che devono supportare la modifica dinamica delle impostazioni cultura devono essere configurate `BlazorWebAssemblyLoadAllGlobalizationData` nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="f53fa-128">Applications that must support dynamically changing the culture should configure `BlazorWebAssemblyLoadAllGlobalizationData` in the project file:</span></span>

```xml
<PropertyGroup>
  <BlazorWebAssemblyLoadAllGlobalizationData>true</BlazorWebAssemblyLoadAllGlobalizationData>
</PropertyGroup>
```

<span data-ttu-id="f53fa-129">Blazor WebAssembly può anche essere configurato per avviare usando impostazioni cultura specifiche dell'applicazione usando le opzioni passate a `Blazor.start` .</span><span class="sxs-lookup"><span data-stu-id="f53fa-129">Blazor WebAssembly can also be configured to launch using a specific application culture using options passed to `Blazor.start`.</span></span> <span data-ttu-id="f53fa-130">Ad esempio, l'esempio seguente mostra un'app configurata per l'avvio usando le `en-GB` impostazioni cultura:</span><span class="sxs-lookup"><span data-stu-id="f53fa-130">For instance, the sample below shows an app configured to launch using the `en-GB` culture:</span></span>

```html
<script src="_framework/blazor.webassembly.js" autostart="false"></script>
<script>
  Blazor.start({
    applicationCulture: 'en-GB'
  });
</script>
```

<span data-ttu-id="f53fa-131">Il valore per `applicationCulture` deve essere conforme al [formato di tag del linguaggio BCP-47](https://tools.ietf.org/html/bcp47).</span><span class="sxs-lookup"><span data-stu-id="f53fa-131">The value for `applicationCulture` should conform to the [BCP-47 language tag format](https://tools.ietf.org/html/bcp47).</span></span>

<span data-ttu-id="f53fa-132">Se l'app non richiede la localizzazione, è possibile configurare l'app in modo che supporti la lingua inglese, che è basata sulle `en-US` impostazioni cultura:</span><span class="sxs-lookup"><span data-stu-id="f53fa-132">If the app doesn't require localization, you may configure the app to support the invariant culture, which is based on the `en-US` culture:</span></span>

```xml
<PropertyGroup>
  <InvariantGlobalization>true</InvariantGlobalization>
</PropertyGroup>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="f53fa-133">Per impostazione predefinita, la configurazione del linker linguaggio intermedio (IL) per le Blazor WebAssembly app rimuove le informazioni di internazionalizzazione ad eccezione delle impostazioni locali richieste in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="f53fa-133">By default, the Intermediate Language (IL) Linker configuration for Blazor WebAssembly apps strips out internationalization information except for locales explicitly requested.</span></span> <span data-ttu-id="f53fa-134">Per altre informazioni, vedere <xref:blazor/host-and-deploy/configure-linker#configure-the-linker-for-internationalization>.</span><span class="sxs-lookup"><span data-stu-id="f53fa-134">For more information, see <xref:blazor/host-and-deploy/configure-linker#configure-the-linker-for-internationalization>.</span></span>

::: moniker-end

<span data-ttu-id="f53fa-135">Mentre le impostazioni cultura che Blazor selezionano per impostazione predefinita potrebbero essere sufficienti per la maggior parte degli utenti, è consigliabile offrire agli utenti un modo per specificare le impostazioni locali preferite.</span><span class="sxs-lookup"><span data-stu-id="f53fa-135">While the culture that Blazor selects by default might be sufficient for most users, consider offering a way for users to specify their preferred locale.</span></span> <span data-ttu-id="f53fa-136">Per un' Blazor WebAssembly app di esempio con selezione impostazioni cultura, vedere l' [`LocSample`](https://github.com/pranavkm/LocSample) app di esempio localizzazione.</span><span class="sxs-lookup"><span data-stu-id="f53fa-136">For a Blazor WebAssembly sample app with a culture picker, see the [`LocSample`](https://github.com/pranavkm/LocSample) localization sample app.</span></span>

### Blazor Server

<span data-ttu-id="f53fa-137">Blazor Server le app vengono localizzate usando il [middleware di localizzazione](xref:fundamentals/localization#localization-middleware).</span><span class="sxs-lookup"><span data-stu-id="f53fa-137">Blazor Server apps are localized using [Localization Middleware](xref:fundamentals/localization#localization-middleware).</span></span> <span data-ttu-id="f53fa-138">Il middleware seleziona le impostazioni cultura appropriate per gli utenti che richiedono risorse dall'app.</span><span class="sxs-lookup"><span data-stu-id="f53fa-138">The middleware selects the appropriate culture for users requesting resources from the app.</span></span>

<span data-ttu-id="f53fa-139">Le impostazioni cultura possono essere impostate utilizzando uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="f53fa-139">The culture can be set using one of the following approaches:</span></span>

* <span data-ttu-id="f53fa-140">[Cookies](#cookies)</span><span class="sxs-lookup"><span data-stu-id="f53fa-140">[Cookies](#cookies)</span></span>
* [<span data-ttu-id="f53fa-141">Fornire l'interfaccia utente per scegliere le impostazioni cultura</span><span class="sxs-lookup"><span data-stu-id="f53fa-141">Provide UI to choose the culture</span></span>](#provide-ui-to-choose-the-culture)

<span data-ttu-id="f53fa-142">Per altre informazioni ed esempi, vedere <xref:fundamentals/localization>.</span><span class="sxs-lookup"><span data-stu-id="f53fa-142">For more information and examples, see <xref:fundamentals/localization>.</span></span>

#### <a name="cookies"></a><span data-ttu-id="f53fa-143">Cookies</span><span class="sxs-lookup"><span data-stu-id="f53fa-143">Cookies</span></span>

<span data-ttu-id="f53fa-144">Le impostazioni cultura cookie della localizzazione possono salvare in maniera permanente le impostazioni cultura dell'utente.</span><span class="sxs-lookup"><span data-stu-id="f53fa-144">A localization culture cookie can persist the user's culture.</span></span> <span data-ttu-id="f53fa-145">Il middleware di localizzazione legge l'oggetto cookie sulle richieste successive per impostare le impostazioni cultura dell'utente.</span><span class="sxs-lookup"><span data-stu-id="f53fa-145">The Localization Middleware reads the cookie on subsequent requests to set the user's culture.</span></span> 

<span data-ttu-id="f53fa-146">L'uso di un oggetto cookie garantisce che la connessione WebSocket possa propagare correttamente le impostazioni cultura.</span><span class="sxs-lookup"><span data-stu-id="f53fa-146">Use of a cookie ensures that the WebSocket connection can correctly propagate the culture.</span></span> <span data-ttu-id="f53fa-147">Se gli schemi di localizzazione sono basati sul percorso dell'URL o sulla stringa di query, lo schema potrebbe non essere in grado di funzionare con WebSocket, quindi non è possibile salvare in modo permanente le impostazioni cultura.</span><span class="sxs-lookup"><span data-stu-id="f53fa-147">If localization schemes are based on the URL path or query string, the scheme might not be able to work with WebSockets, thus fail to persist the culture.</span></span> <span data-ttu-id="f53fa-148">Pertanto, l'utilizzo delle impostazioni cultura di localizzazione cookie è l'approccio consigliato.</span><span class="sxs-lookup"><span data-stu-id="f53fa-148">Therefore, use of a localization culture cookie is the recommended approach.</span></span>

<span data-ttu-id="f53fa-149">Qualsiasi tecnica può essere utilizzata per assegnare impostazioni cultura se le impostazioni cultura vengono rese permanente in una localizzazione cookie .</span><span class="sxs-lookup"><span data-stu-id="f53fa-149">Any technique can be used to assign a culture if the culture is persisted in a localization cookie.</span></span> <span data-ttu-id="f53fa-150">Se l'app ha già uno schema di localizzazione stabilito per ASP.NET Core lato server, continuare a usare l'infrastruttura di localizzazione esistente dell'app e impostare le impostazioni cultura di localizzazione nello cookie schema dell'app.</span><span class="sxs-lookup"><span data-stu-id="f53fa-150">If the app already has an established localization scheme for server-side ASP.NET Core, continue to use the app's existing localization infrastructure and set the localization culture cookie within the app's scheme.</span></span>

<span data-ttu-id="f53fa-151">Nell'esempio seguente viene illustrato come impostare le impostazioni cultura correnti in un oggetto cookie che può essere letto dal middleware di localizzazione.</span><span class="sxs-lookup"><span data-stu-id="f53fa-151">The following example shows how to set the current culture in a cookie that can be read by the Localization Middleware.</span></span> <span data-ttu-id="f53fa-152">Creare un' Razor espressione nel `Pages/_Host.cshtml` file immediatamente all'interno del tag di apertura `<body>` :</span><span class="sxs-lookup"><span data-stu-id="f53fa-152">Create a Razor expression in the `Pages/_Host.cshtml` file immediately inside the opening `<body>` tag:</span></span>

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

<span data-ttu-id="f53fa-153">La localizzazione viene gestita dall'app nella sequenza di eventi seguente:</span><span class="sxs-lookup"><span data-stu-id="f53fa-153">Localization is handled by the app in the following sequence of events:</span></span>

1. <span data-ttu-id="f53fa-154">Il browser invia una richiesta HTTP iniziale all'app.</span><span class="sxs-lookup"><span data-stu-id="f53fa-154">The browser sends an initial HTTP request to the app.</span></span>
1. <span data-ttu-id="f53fa-155">Le impostazioni cultura vengono assegnate dal middleware di localizzazione.</span><span class="sxs-lookup"><span data-stu-id="f53fa-155">The culture is assigned by the Localization Middleware.</span></span>
1. <span data-ttu-id="f53fa-156">L' Razor espressione nella `_Host` pagina ( `_Host.cshtml` ) rende permanente le impostazioni cultura in un cookie come parte della risposta.</span><span class="sxs-lookup"><span data-stu-id="f53fa-156">The Razor expression in the `_Host` page (`_Host.cshtml`) persists the culture in a cookie as part of the response.</span></span>
1. <span data-ttu-id="f53fa-157">Il browser apre una connessione WebSocket per creare una Blazor Server sessione interattiva.</span><span class="sxs-lookup"><span data-stu-id="f53fa-157">The browser opens a WebSocket connection to create an interactive Blazor Server session.</span></span>
1. <span data-ttu-id="f53fa-158">Il middleware di localizzazione legge cookie e assegna le impostazioni cultura.</span><span class="sxs-lookup"><span data-stu-id="f53fa-158">The Localization Middleware reads the cookie and assigns the culture.</span></span>
1. <span data-ttu-id="f53fa-159">La Blazor Server sessione inizia con le impostazioni cultura corrette.</span><span class="sxs-lookup"><span data-stu-id="f53fa-159">The Blazor Server session begins with the correct culture.</span></span>

<span data-ttu-id="f53fa-160">Quando si utilizza un <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage> , utilizzare la <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.Context> proprietà:</span><span class="sxs-lookup"><span data-stu-id="f53fa-160">When working with a <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage>, use the <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.Context> property:</span></span>

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

#### <a name="provide-ui-to-choose-the-culture"></a><span data-ttu-id="f53fa-161">Fornire l'interfaccia utente per scegliere le impostazioni cultura</span><span class="sxs-lookup"><span data-stu-id="f53fa-161">Provide UI to choose the culture</span></span>

<span data-ttu-id="f53fa-162">Per consentire a un utente di selezionare le impostazioni cultura, è consigliabile un *approccio basato su Reindirizzamento* .</span><span class="sxs-lookup"><span data-stu-id="f53fa-162">To provide UI to allow a user to select a culture, a *redirect-based approach* is recommended.</span></span> <span data-ttu-id="f53fa-163">Il processo è simile a quello che accade in un'app Web quando un utente tenta di accedere a una risorsa protetta.</span><span class="sxs-lookup"><span data-stu-id="f53fa-163">The process is similar to what happens in a web app when a user attempts to access a secure resource.</span></span> <span data-ttu-id="f53fa-164">L'utente viene reindirizzato a una pagina di accesso, quindi viene reindirizzato di nuovo alla risorsa originale.</span><span class="sxs-lookup"><span data-stu-id="f53fa-164">The user is redirected to a sign-in page and then redirected back to the original resource.</span></span> 

<span data-ttu-id="f53fa-165">L'app rende permanente le impostazioni cultura selezionate dall'utente tramite un reindirizzamento a un controller.</span><span class="sxs-lookup"><span data-stu-id="f53fa-165">The app persists the user's selected culture via a redirect to a controller.</span></span> <span data-ttu-id="f53fa-166">Il controller imposta le impostazioni cultura selezionate dall'utente in un cookie e reindirizza l'utente all'URI originale.</span><span class="sxs-lookup"><span data-stu-id="f53fa-166">The controller sets the user's selected culture into a cookie and redirects the user back to the original URI.</span></span>

<span data-ttu-id="f53fa-167">Stabilire un endpoint HTTP nel server per impostare le impostazioni cultura selezionate dall'utente in un oggetto cookie ed eseguire di nuovo il reindirizzamento all'URI originale:</span><span class="sxs-lookup"><span data-stu-id="f53fa-167">Establish an HTTP endpoint on the server to set the user's selected culture in a cookie and perform the redirect back to the original URI:</span></span>

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
> <span data-ttu-id="f53fa-168">Usare il <xref:Microsoft.AspNetCore.Mvc.ControllerBase.LocalRedirect%2A> risultato dell'azione per impedire gli attacchi di reindirizzamento aperti.</span><span class="sxs-lookup"><span data-stu-id="f53fa-168">Use the <xref:Microsoft.AspNetCore.Mvc.ControllerBase.LocalRedirect%2A> action result to prevent open redirect attacks.</span></span> <span data-ttu-id="f53fa-169">Per altre informazioni, vedere <xref:security/preventing-open-redirects>.</span><span class="sxs-lookup"><span data-stu-id="f53fa-169">For more information, see <xref:security/preventing-open-redirects>.</span></span>

<span data-ttu-id="f53fa-170">Se l'app non è configurata per l'elaborazione delle azioni del controller:</span><span class="sxs-lookup"><span data-stu-id="f53fa-170">If the app isn't configured to process controller actions:</span></span>

* <span data-ttu-id="f53fa-171">Aggiungere i servizi MVC alla raccolta di servizi in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="f53fa-171">Add MVC services to the service collection in `Startup.ConfigureServices`:</span></span>

  ```csharp
  services.AddControllers();
  ```

* <span data-ttu-id="f53fa-172">Aggiungi routing endpoint controller in `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="f53fa-172">Add controller endpoint routing in `Startup.Configure`:</span></span>

  ```csharp
  app.UseEndpoints(endpoints =>
  {
      endpoints.MapControllers();
      endpoints.MapBlazorHub();
      endpoints.MapFallbackToPage("/_Host");
  });
  ```

<span data-ttu-id="f53fa-173">Il componente seguente mostra un esempio di come eseguire il reindirizzamento iniziale quando l'utente seleziona le impostazioni cultura:</span><span class="sxs-lookup"><span data-stu-id="f53fa-173">The following component shows an example of how to perform the initial redirection when the user selects a culture:</span></span>

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

## <a name="additional-resources"></a><span data-ttu-id="f53fa-174">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="f53fa-174">Additional resources</span></span>

* <xref:fundamentals/localization>
