---
title: File statici in ASP.NET Core
author: rick-anderson
description: Informazioni sull'uso, sulla protezione di file statici e sulla configurazione dei comportamenti del middleware che ospita i file statici nell'app Web ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 6/23/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/static-files
ms.openlocfilehash: fb5ef59a74a768a1c202b49872b8a10327c96ff8
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413457"
---
# <a name="static-files-in-aspnet-core"></a>File statici in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Kirk Larkin](https://twitter.com/serpent5)

I file statici, ad esempio HTML, CSS, immagini e JavaScript, sono asset che un'app ASP.NET Core serve direttamente ai client per impostazione predefinita.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/static-files/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="serve-static-files"></a>Usare i file statici

I file statici vengono archiviati nella directory [radice Web](xref:fundamentals/index#web-root) del progetto. La directory predefinita è `{content root}/wwwroot` , ma può essere modificata con il <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot%2A> metodo. Per altre informazioni, vedere [radice del contenuto](xref:fundamentals/index#content-root) e [radice Web](xref:fundamentals/index#web-root).

Il metodo <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> imposta la radice del contenuto nella directory corrente:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/Program2.cs?name=snippet_Program)]

Il codice precedente è stato creato con il modello di app Web.

I file statici sono accessibili tramite un percorso relativo alla [radice Web](xref:fundamentals/index#web-root). Ad esempio, i modelli di progetto di **applicazione Web** contengono diverse cartelle all'interno della `wwwroot` cartella:

* `wwwroot`
  * `css`
  * `js`
  * `lib`

Si consiglia di creare la cartella *wwwroot/images* e di aggiungere il file *wwwroot/images/MyImage.jpg* . Il formato dell'URI per accedere a un file nella `images` cartella è `https://<hostname>/images/<image_file_name>` . Ad esempio: `https://localhost:5001/images/MyImage.jpg`

### <a name="serve-files-in-web-root"></a>Gestire i file nella radice Web

I modelli di app Web predefiniti chiamano il <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> metodo in `Startup.Configure` , che consente di servire i file statici:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/Startup.cs?name=snippet_Configure&highlight=15)]

L'overload del `UseStaticFiles` metodo senza parametri contrassegna i file nella [radice Web](xref:fundamentals/index#web-root) come servable. Il markup seguente fa riferimento a *wwwroot/images/MyImage.jpg*:

```html
<img src="~/images/MyImage.jpg" class="img" alt="My image" />
```

Nel codice precedente, il carattere tilde `~/` punta alla [radice Web](xref:fundamentals/index#web-root).

### <a name="serve-files-outside-of-web-root"></a>Usare i file all'esterno della radice Web

Si consideri una gerarchia di directory in cui i file statici da servire si trovano all'esterno della [radice Web](xref:fundamentals/index#web-root):

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `red-rose.jpg`

Una richiesta può accedere al `red-rose.jpg` file configurando il middleware dei file statici come indicato di seguito:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupRose.cs?name=snippet_Configure&highlight=15-22)]

Nel codice precedente la gerarchia di directory *MyStaticFiles* viene esposta pubblicamente tramite il segmento dell'URI *StaticFiles*. Richiesta di `https://<hostname>/StaticFiles/images/red-rose.jpg` servizio per il file di *red-rose.jpg* .

Il markup seguente fa riferimento a *MyStaticFiles/images/red-rose.jpg*:

```html
<img src="~/StaticFiles/images/red-rose.jpg" class="img" alt="A red rose" />
```

### <a name="set-http-response-headers"></a>Impostare le intestazioni della risposta HTTP

Un <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> oggetto può essere utilizzato per impostare le intestazioni di risposta http. Oltre alla configurazione del file statico che funge dalla [radice Web](xref:fundamentals/index#web-root), il codice seguente imposta l' `Cache-Control` intestazione:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupAddHeader.cs?name=snippet_Configure&highlight=15-24)]

<!-- Q: The preceding code sets max-age to 604800 seconds (7 days), so what does the following mean? -->

I file statici possono essere memorizzati nella cache pubblicamente per 600 secondi:

![Sono state aggiunte le intestazioni della risposta che visualizzano l'intestazione Cache-Control](static-files/_static/add-header.png)

## <a name="static-file-authorization"></a>Autorizzazione dei file statici

I modelli di ASP.NET Core chiamano <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> prima di chiamare <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> . La maggior parte delle app segue questo modello. Quando il middleware dei file statici viene chiamato prima del middleware di autorizzazione:

  * Non vengono eseguiti controlli di autorizzazione sui file statici.
  * I file statici serviti dal middleware dei file statici, ad esempio quelli in `wwwroot` , sono accessibili pubblicamente.
  
Per gestire i file statici in base all'autorizzazione:

  * Archiviarli all'esterno di `wwwroot` .
  * Chiamare `UseStaticFiles` , specificando un percorso, dopo la chiamata a `UseAuthorization` .
  * Impostare i [criteri di autorizzazione di fallback](xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy).

  [!code-csharp[](static-files/samples/3.x/StaticFileAuth/Startup.cs?name=snippet2&highlight=24-29)]
  
  [!code-csharp[](static-files/samples/3.x/StaticFileAuth/Startup.cs?name=snippet1&highlight=20-25)]

  Nel codice precedente, i criteri di autorizzazione di fallback richiedono che ***tutti*** gli utenti siano autenticati. Gli endpoint, ad esempio i controller, le Razor pagine e così via, che specificano i propri requisiti di autorizzazione, non usano i criteri di autorizzazione di fallback. Ad esempio, Razor le pagine, i controller o i metodi di azione con `[AllowAnonymous]` o `[Authorize(PolicyName="MyPolicy")]` utilizzano l'attributo di autorizzazione applicato anziché i criteri di autorizzazione di fallback.

  <xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> aggiunge <xref:Microsoft.AspNetCore.Authorization.Infrastructure.DenyAnonymousAuthorizationRequirement> all'istanza corrente che impone che l'utente corrente sia autenticato.

  Gli asset statici in `wwwroot` sono accessibili pubblicamente, perché il middleware del file statico predefinito ( `app.UseStaticFiles();` ) viene chiamato prima `UseAuthentication` . Per gli asset statici nella cartella *MyStaticFiles* è richiesta l'autenticazione. Il [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/static-files/samples) illustra questa operazione.

Un approccio alternativo per gestire i file in base all'autorizzazione è:

  * Archiviarli all'esterno di `wwwroot` e di qualsiasi directory accessibile al middleware di file statici.
  * Servirle tramite un metodo di azione a cui viene applicata l'autorizzazione e restituire un <xref:Microsoft.AspNetCore.Mvc.FileResult> oggetto:

  [!code-csharp[](static-files/samples/3.x/StaticFilesSample/Controllers/HomeController.cs?name=snippet_BannerImage)]

## <a name="directory-browsing"></a>Esplorazione directory

L'esplorazione directory consente l'elenco di directory nelle directory specificate.

Per motivi di sicurezza, l'esplorazione directory è disabilitata per impostazione predefinita. Per ulteriori informazioni, vedere [considerazioni](#considerations).

Abilitare l'esplorazione directory con:

* <xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A> in `Startup.ConfigureServices` .
* <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser%2A> in `Startup.Configure` .

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ClassMembers&highlight=4,21-35)]

Il codice precedente consente l'esplorazione directory della cartella *wwwroot/images* utilizzando l'URL `https://<hostname>/MyImages` , con collegamenti a ogni file e cartella:

![esplorazione directory](static-files/_static/dir-browse.png)

## <a name="serve-default-documents"></a>Gestire i documenti predefiniti

L'impostazione di una pagina predefinita fornisce ai visitatori un punto di partenza per un sito. Per gestire una pagina predefinita da `wwwroot` senza un URI completo, chiamare il <xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A> Metodo:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty.cs?name=snippet_Configure&highlight=15)]

`UseDefaultFiles` deve essere chiamato prima di `UseStaticFiles` per usare il file predefinito. `UseDefaultFiles` è un rewriter URL che non serve il file.

Con `UseDefaultFiles` , richiede a una cartella in `wwwroot` Cerca:

* *default.htm*
* *default.html*
* *index.htm*
* *index.html*

Il primo file trovato nell'elenco viene usato come se la richiesta fosse l'URI completo. L'URL del browser continua a riflettere l'URI richiesto.

Il codice seguente modifica il nome file predefinito in *mydefault.html*:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupDefault.cs?name=snippet_DefaultFiles)]

Il codice seguente illustra `Startup.Configure` con il codice precedente:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupDefault.cs?name=snippet_Configure&highlight=15-19)]

### <a name="usefileserver-for-default-documents"></a>UseFileServer per i documenti predefiniti

<xref:Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer*> combina la funzionalità di `UseStaticFiles` , `UseDefaultFiles` e facoltativamente `UseDirectoryBrowser` .

Chiamare `app.UseFileServer` per abilitare la funzione di file statici e il file predefinito. L'esplorazione directory non è abilitata. Il codice seguente illustra `Startup.Configure` con `UseFileServer` :

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty2.cs?name=snippet_Configure&highlight=15)]

Il codice seguente consente di mantenere i file statici, il file predefinito e l'esplorazione delle directory:

```csharp
app.UseFileServer(enableDirectoryBrowsing: true);
```

Il codice seguente illustra `Startup.Configure` con il codice precedente:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty3.cs?name=snippet_Configure&highlight=15)]

Considerare la gerarchia di directory seguente:

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `MyImage.jpg`
  * `default.html`

Il codice seguente consente di mantenere i file statici, il file predefinito e l'esplorazione della directory di `MyStaticFiles` :

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileServer.cs?name=snippet_ClassMembers&highlight=4,21-31)]

<xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A> deve essere chiamato quando il `EnableDirectoryBrowsing` valore della proprietà è `true` .

Dall'uso della gerarchia di file e del codice precedente ne deriva quanto segue:

| URI            |      Risposta  |
| ------- | ------|
| `https://<hostname>/StaticFiles/images/MyImage.jpg` | *MyStaticFiles/immagini/MyImage.jpg* |
| `https://<hostname>/StaticFiles` | *MyStaticFiles/default.html* |

Se nella directory *MyStaticFiles* non è presente alcun file con nome predefinito, `https://<hostname>/StaticFiles` restituisce l'elenco di directory con i collegamenti selezionabili:

![Elenco di file statici](static-files/_static/db2.png)

<xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles*> ed <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser*> eseguono un reindirizzamento sul lato client dall'URI di destinazione senza una finalizzazione `/`  all'URI di destinazione con una finalizzazione `/` . Ad esempio, da `https://<hostname>/StaticFiles` a `https://<hostname>/StaticFiles/` . Gli URL relativi nella directory *staticfiles* non sono validi senza una barra finale ( `/` ).

## <a name="fileextensioncontenttypeprovider"></a>Classe FileExtensionContentTypeProvider

La <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> classe contiene una `Mappings` proprietà che funge da mapping tra le estensioni di file e i tipi di contenuto MIME. Nell'esempio seguente viene eseguito il mapping di diverse estensioni di file a tipi MIME noti. L'estensione *RTF* viene sostituita e viene rimosso *. mp4* :

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_Provider)]

Il codice seguente illustra `Startup.Configure` con il codice precedente:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_Configure&highlight=15-43)]

Vedere [Tipi di contenuto MIME](https://www.iana.org/assignments/media-types/media-types.xhtml).

## <a name="non-standard-content-types"></a>Tipi di contenuto non standard

Il middleware dei file statici riconosce quasi 400 tipi di contenuto di file noti. Se l'utente richiede un file con un tipo di file sconosciuto, il middleware dei file statici passa la richiesta al middleware successivo nella pipeline. Se nessun middleware gestisce la richiesta, viene restituita la risposta *404 Non trovato*. Se è abilitata l'esplorazione directory, viene visualizzato un collegamento al file nella visualizzazione directory.

Il codice seguente consente di usare tipi sconosciuti ed esegue il rendering del file sconosciuto come immagine:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_UseStaticFiles)]

Il codice seguente illustra `Startup.Configure` con il codice precedente:

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_Configure&highlight=15-19)]

Con il codice precedente, una richiesta per un file con un tipo di contenuto sconosciuto viene restituita come immagine.

> [!WARNING]
> L'abilitazione <xref:Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes> è un rischio per la sicurezza. È disabilitato per impostazione predefinita e ne è sconsigliato l'uso. La classe [FileExtensionContentTypeProvider](#fileextensioncontenttypeprovider) offre un'alternativa più sicura per usare i file con estensioni non standard.

## <a name="serve-files-from-multiple-locations"></a>Gestire i file da più posizioni

`UseStaticFiles` e `UseFileServer` per impostazione predefinita il provider di file che punta a `wwwroot` . Istanze aggiuntive di `UseStaticFiles` e `UseFileServer` possono essere fornite con altri provider di file per gestire i file da altri percorsi. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/15578).

<a name="sc"></a>

### <a name="security-considerations-for-static-files"></a>Considerazioni sulla sicurezza per i file statici

> [!WARNING]
> L'uso di `UseDirectoryBrowser` e `UseStaticFiles` può comportare la perdita di informazioni riservate. È consigliabile disabilitare l'esplorazione directory nell'ambiente di produzione. Controllare attentamente quali sono le directory abilitate tramite `UseStaticFiles` o `UseDirectoryBrowser`. L'intera directory e le relative sottodirectory diventano pubblicamente accessibili. Archiviare i file adatti al servizio pubblico in una directory dedicata, ad esempio `<content_root>/wwwroot` . Separare questi file da visualizzazioni MVC, Razor pagine, file di configurazione e così via.

* Gli URL per il contenuto esposto con `UseDirectoryBrowser` e `UseStaticFiles` sono soggetti a distinzione tra maiuscole e minuscole e a limitazione di caratteri del file system sottostante. Windows, ad esempio, non fa distinzione tra maiuscole e minuscole, ma macOS e Linux non lo sono.

* Le app ASP.NET Core ospitate in IIS usano il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) per inoltrare tutte le richieste all'app, incluse le richieste di file statici. Il gestore di file statici di IIS non viene usato e non ha la possibilità di gestire le richieste.

* Completare la procedura seguente in Gestione IIS per rimuovere il gestore di file statici di IIS a livello di server o di sito Web:
    1. Passare alla funzionalità **Moduli**.
    1. Selezionare **StaticFileModule** nell'elenco.
    1. Fare clic su **Rimuovi** nell'intestazione laterale **Azioni**.

> [!WARNING]
> Se il gestore di file statici di IIS è abilitato **e** il modulo ASP.NET Core non è configurato correttamente, vengono usati i file statici. Tale scenario si verifica ad esempio se il file *web.config* non è stato distribuito.

* Inserire i file di codice, tra cui *. cs* e *. cshtml*, all'esterno della [radice Web](xref:fundamentals/index#web-root)del progetto dell'app. Si crea quindi un separazione logica tra il contenuto sul lato client dell'app e il codice basato su server. In questo modo si impedisce la perdita del codice sul lato server.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Middleware](xref:fundamentals/middleware/index)
* [Introduzione ad ASP.NET Core](xref:index)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://twitter.com/Scott_Addie)

I file statici, ad esempio HTML, CSS, immagini e JavaScript, sono asset che un'app ASP.NET Core rende direttamente disponibili nei client. Per abilitare l'uso di questi file, sono necessarie alcune configurazioni.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/static-files/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="serve-static-files"></a>Usare i file statici

I file statici vengono archiviati nella directory [radice Web](xref:fundamentals/index#web-root) del progetto. La directory predefinita è *{Content root}/wwwroot*, ma può essere modificata tramite il <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot%2A> metodo. Vedere [Radice del contenuto](xref:fundamentals/index#content-root) e [Radice Web](xref:fundamentals/index#web-root) per altre informazioni.

L'host Web dell'app deve conoscere la directory radice del contenuto.

Il metodo `WebHost.CreateDefaultBuilder` imposta la radice del contenuto nella directory corrente:

[!code-csharp[](../common/samples/WebApplication1DotNetCore2.0App/Program.cs?name=snippet_Main&highlight=9)]

I file statici sono accessibili tramite un percorso relativo alla [radice Web](xref:fundamentals/index#web-root). Ad esempio, il modello di progetto **applicazione Web** contiene diverse cartelle all'interno della `wwwroot` cartella:

* `wwwroot`
  * `css`
  * `images`
  * `js`

Il formato URI per accedere a un file nella sottocartella *images* è *http:// \<server_address> \<image_file_name> /images/*. ad esempio *http://localhost:9189/images/banner3.svg*.

Se la destinazione è .NET Framework, aggiungere il pacchetto [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles/) al progetto. Se la destinazione è .NET Core, questo pacchetto è incluso nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app).

Configurare il [middleware](xref:fundamentals/middleware/index), che consente la conservazione dei file statici.

### <a name="serve-files-inside-of-web-root"></a>Usare i file all'interno della radice Web

Richiamare il <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> metodo all'interno di `Startup.Configure` :

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupStaticFiles.cs?name=snippet_ConfigureMethod&highlight=3)]

L'overload del `UseStaticFiles` metodo senza parametri contrassegna i file nella [radice Web](xref:fundamentals/index#web-root) come servable. Il markup seguente si riferisce a *wwwroot/images/banner1.svg*:

[!code-cshtml[](static-files/samples/1.x/StaticFilesSample/Views/Home/Index.cshtml?name=snippet_static_file_wwwroot)]

Nel codice precedente, il carattere tilde `~/` punta alla [radice Web](xref:fundamentals/index#web-root).

### <a name="serve-files-outside-of-web-root"></a>Usare i file all'esterno della radice Web

Si consideri una gerarchia di directory in cui i file statici da servire si trovano all'esterno della [radice Web](xref:fundamentals/index#web-root):

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `banner1.svg`

Una richiesta può accedere al file *banner1.svg* configurando il middleware dei file statici come indicato di seguito:

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupTwoStaticFiles.cs?name=snippet_ConfigureMethod&highlight=5-10)]

Nel codice precedente la gerarchia di directory *MyStaticFiles* viene esposta pubblicamente tramite il segmento dell'URI *StaticFiles*. Una richiesta a *http:// \<server_address> /staticfiles/images/banner1.svg* serve il file *banner1. svg* .

Il markup seguente si riferisce a *MyStaticFiles/images/banner1.svg*:

[!code-cshtml[](static-files/samples/1.x/StaticFilesSample/Views/Home/Index.cshtml?name=snippet_static_file_outside)]

### <a name="set-http-response-headers"></a>Impostare le intestazioni della risposta HTTP

Un <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> oggetto può essere utilizzato per impostare le intestazioni di risposta http. Oltre alla configurazione del file statico che funge dalla [radice Web](xref:fundamentals/index#web-root), il codice seguente imposta l' `Cache-Control` intestazione:

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupAddHeader.cs?name=snippet_ConfigureMethod)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

Il <xref:Microsoft.AspNetCore.Http.HeaderDictionaryExtensions.Append%2A?displayProperty=nameWithType> metodo esiste nel pacchetto [Microsoft. AspNetCore. http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) .

I file sono stati resi pubblicamente inseribili nella cache per 10 minuti (600 secondi) nell'ambiente di sviluppo:

![Sono state aggiunte le intestazioni della risposta che visualizzano l'intestazione Cache-Control](static-files/_static/add-header.png)

## <a name="static-file-authorization"></a>Autorizzazione dei file statici

Il middleware dei file statici non offre controlli di autorizzazione. I file usati dal middleware, inclusi i file in *wwwroot*, sono disponibili pubblicamente. Per usare i file in base alle autorizzazioni:

* Archiviare i file all'esterno di *wwwroot* e di qualsiasi directory alla quale può accedere il middleware dei file statici.
* Usarli tramite un metodo di azione al quale è applicata l'autorizzazione. Restituisce un <xref:Microsoft.AspNetCore.Mvc.FileResult> oggetto:

  [!code-csharp[](static-files/samples/1.x/StaticFilesSample/Controllers/HomeController.cs?name=snippet_BannerImageAction)]

## <a name="enable-directory-browsing"></a>Abilitare l'esplorazione directory

L'esplorazione directory consente agli utenti dell'app Web di visualizzare un elenco di directory e i file all'interno di una directory specifica. Per motivi di sicurezza, l'esplorazione directory è disabilitata per impostazione predefinita (vedere [Considerazioni](#considerations)). Abilitare l'esplorazione directory richiamando il <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser%2A> metodo in `Startup.Configure` :

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureMethod&highlight=12-17)]

Aggiungere i servizi necessari richiamando il <xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A> metodo da `Startup.ConfigureServices` :

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureServicesMethod&highlight=3)]

Il codice precedente consente l'esplorazione directory della cartella *wwwroot/images* utilizzando l'URL *http:// \<server_address> /myImages*, con collegamenti a ogni file e cartella:

![esplorazione directory](static-files/_static/dir-browse.png)

Vedere la sezione [Considerazioni](#considerations) per conoscere i rischi per la sicurezza quando l'esplorazione viene abilitata.

Si notino le due chiamate `UseStaticFiles` nell'esempio seguente. La prima chiamata consente l'uso di file statici nella cartella *wwwroot*. La seconda chiamata Abilita l'esplorazione directory della cartella *wwwroot/images* utilizzando l'URL *http:// \<server_address> /myImages*:

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureMethod&highlight=3,5)]

## <a name="serve-a-default-document"></a>Usare un documento predefinito

L'impostazione di una home page predefinita offre ai visitatori un punto di partenza logico per la visita del sito. Per gestire una pagina predefinita senza che l'utente completi l'URI, chiamare il <xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A> metodo da `Startup.Configure` :

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupEmpty.cs?name=snippet_ConfigureMethod&highlight=3)]

> [!IMPORTANT]
> `UseDefaultFiles` deve essere chiamato prima di `UseStaticFiles` per usare il file predefinito. `UseDefaultFiles` è un URL rewriter che effettivamente non usa il file. Abilitare il middleware dei file statici tramite `UseStaticFiles` per usare il file.

Con `UseDefaultFiles`, si richiede la ricerca nella cartella degli elementi seguenti:

* *default.htm*
* *default.html*
* *index.htm*
* *index.html*

Il primo file trovato nell'elenco viene usato come se la richiesta fosse l'URI completo. L'URL del browser continua a riflettere l'URI richiesto.

Il codice seguente modifica il nome file predefinito in *mydefault.html*:

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupDefault.cs?name=snippet_ConfigureMethod)]

## <a name="usefileserver"></a>UseFileServer

<xref:Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer*> combina la funzionalità di `UseStaticFiles` , `UseDefaultFiles` e facoltativamente `UseDirectoryBrowser` .

Il codice seguente consente di usare i file statici e il file predefinito. L'esplorazione directory non è abilitata.

```csharp
app.UseFileServer();
```

Il codice seguente si basa sull'overload senza parametri, consentendo l'esplorazione directory:

```csharp
app.UseFileServer(enableDirectoryBrowsing: true);
```

Considerare la gerarchia di directory seguente:

* **wwwroot**
  * **CSS**
  * **images**
  * **JS**
* **MyStaticFiles**
  * **images**
    * *banner1.svg*
  * *default.html*

Il codice seguente abilita i file statici, i file predefiniti e l'esplorazione directory di `MyStaticFiles`:

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupUseFileServer.cs?name=snippet_ConfigureMethod&highlight=5-11)]

Chiamare il metodo `AddDirectoryBrowser` quando il valore della proprietà `EnableDirectoryBrowsing` è `true`:

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupUseFileServer.cs?name=snippet_ConfigureServicesMethod)]

Dall'uso della gerarchia di file e del codice precedente ne deriva quanto segue:

| URI            |                             Risposta  |
| ------- | ------|
| *\<server_address>/StaticFiles/images/banner1.svg http://*    |      MyStaticFiles/images/banner1.svg |
| *\<server_address>/StaticFiles http://*             |     MyStaticFiles/default.html |

Se nella directory *MyStaticFiles* non è presente alcun file con nome predefinito, *http:// \<server_address> /staticfiles* restituisce l'elenco di directory con i collegamenti che è possibile selezionare:

![Elenco di file statici](static-files/_static/db2.png)

> [!NOTE]
> <xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles*> e <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser*> eseguono un reindirizzamento sul lato client da `http://{SERVER ADDRESS}/StaticFiles` (senza barra rovesciata) a `http://{SERVER ADDRESS}/StaticFiles/` (con barra rovesciata). Gli URL relativi all'interno della directory *StaticFiles* non sono validi senza barra finale.

## <a name="fileextensioncontenttypeprovider"></a>Classe FileExtensionContentTypeProvider

La <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> classe contiene una `Mappings` proprietà che funge da mapping tra le estensioni di file e i tipi di contenuto MIME. Nell'esempio seguente varie estensioni di file vengono registrate in tipi MIME noti. L'estensione *rtf* viene sostituita e l'estensione *mp4* viene rimossa.

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_ConfigureMethod&highlight=3-12,19)]

Vedere [Tipi di contenuto MIME](https://www.iana.org/assignments/media-types/media-types.xhtml).

Per informazioni sull'utilizzo di un oggetto personalizzato <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> o per la configurazione di altri <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> in Blazor app Server, vedere <xref:blazor/fundamentals/static-files> .

## <a name="non-standard-content-types"></a>Tipi di contenuto non standard

Il middleware dei file statici riconosce almeno 400 tipi di contenuto file noti. Se viene richiesto un file con un tipo di file non noto, il middleware dei file statici passa la richiesta al middleware successivo nella pipeline. Se nessun middleware gestisce la richiesta, viene restituita la risposta *404 Non trovato*. Se è abilitata l'esplorazione directory, viene visualizzato un collegamento al file nella visualizzazione directory.

Il codice seguente consente di usare tipi sconosciuti ed esegue il rendering del file sconosciuto come immagine:

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_ConfigureMethod)]

Con il codice precedente, una richiesta per un file con un tipo di contenuto sconosciuto viene restituita come immagine.

> [!WARNING]
> L'abilitazione <xref:Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes> è un rischio per la sicurezza. È disabilitato per impostazione predefinita e ne è sconsigliato l'uso. La classe [FileExtensionContentTypeProvider](#fileextensioncontenttypeprovider) offre un'alternativa più sicura per usare i file con estensioni non standard.

## <a name="serve-files-from-multiple-locations"></a>Gestire i file da più posizioni

`UseStaticFiles``UseFileServer`per impostazione predefinita, il provider di file punta a *wwwroot*. È possibile fornire istanze aggiuntive di `UseStaticFiles` e `UseFileServer` con altri provider di file per gestire i file da altri percorsi. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/15578).

### <a name="considerations"></a>Considerazioni

> [!WARNING]
> L'uso di `UseDirectoryBrowser` e `UseStaticFiles` può comportare la perdita di informazioni riservate. È consigliabile disabilitare l'esplorazione directory nell'ambiente di produzione. Controllare attentamente quali sono le directory abilitate tramite `UseStaticFiles` o `UseDirectoryBrowser`. L'intera directory e le relative sottodirectory diventano pubblicamente accessibili. Archiviare i file adatti al servizio pubblico in una directory dedicata, ad esempio *\<content_root> /wwwroot*. Separare i file da visualizzazioni MVC, Razor pagine (solo 2. x), file di configurazione e così via.

* Gli URL per il contenuto esposto con `UseDirectoryBrowser` e `UseStaticFiles` sono soggetti a distinzione tra maiuscole e minuscole e a limitazione di caratteri del file system sottostante. Ad esempio, in Windows viene fatta distinzione tra maiuscole e minuscole, al contrario di quanto avviene in macOS e Linux.

* Le app ASP.NET Core ospitate in IIS usano il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) per inoltrare tutte le richieste all'app, incluse le richieste di file statici. Non viene usato il gestore di file statici di IIS. Non è possibile gestire le richieste se queste non sono state prima gestite dal modulo.

* Completare la procedura seguente in Gestione IIS per rimuovere il gestore di file statici di IIS a livello di server o di sito Web:
    1. Passare alla funzionalità **Moduli**.
    1. Selezionare **StaticFileModule** nell'elenco.
    1. Fare clic su **Rimuovi** nell'intestazione laterale **Azioni**.

> [!WARNING]
> Se il gestore di file statici di IIS è abilitato **e** il modulo ASP.NET Core non è configurato correttamente, vengono usati i file statici. Tale scenario si verifica ad esempio se il file *web.config* non è stato distribuito.

* Inserire i file di codice (incluso *. cs* e *. cshtml*) all'esterno della [radice Web](xref:fundamentals/index#web-root)del progetto dell'app. Si crea quindi un separazione logica tra il contenuto sul lato client dell'app e il codice basato su server. In questo modo si impedisce la perdita del codice sul lato server.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Middleware](xref:fundamentals/middleware/index)
* [Introduzione ad ASP.NET Core](xref:index)

::: moniker-end
