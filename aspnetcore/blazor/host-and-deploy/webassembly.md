---
title: Ospitare e distribuire ASP.NET Core Blazor WebAssembly
author: guardrex
description: Informazioni su come ospitare e distribuire un' Blazor app usando ASP.NET Core, le reti per la distribuzione di contenuti (CDN), i file server e le pagine github.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/12/2021
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
uid: blazor/host-and-deploy/webassembly
ms.openlocfilehash: 67d7ed9656b2236ffe4f6b65899b807c0ba46ebb
ms.sourcegitcommit: 19a004ff2be73876a9ef0f1ac44d0331849ad159
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/07/2021
ms.locfileid: "99804526"
---
# <a name="host-and-deploy-aspnet-core-blazor-webassembly"></a>Ospitare e distribuire ASP.NET Core Blazor WebAssembly

Di [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com), [Daniel Roth](https://github.com/danroth27), [ben Adams](https://twitter.com/ben_a_adams)e [Safia Amodio](https://safia.rocks)

Con il [ Blazor WebAssembly modello di hosting](xref:blazor/hosting-models#blazor-webassembly):

* L' Blazor app, le relative dipendenze e il Runtime .NET vengono scaricati nel browser in parallelo.
* L'app viene eseguita direttamente nel thread dell'interfaccia utente del browser.

Sono supportate le strategie di distribuzione seguenti:

* L' Blazor app viene gestita da un'app ASP.NET Core. Questa strategia viene trattata nella sezione [Distribuzione ospitata con ASP.NET Core](#hosted-deployment-with-aspnet-core).
* L' Blazor app si trova in un server Web o in un servizio di hosting statico, in cui .NET non viene usato per gestire l' Blazor app. Questa strategia è illustrata nella sezione relativa alla [distribuzione autonoma](#standalone-deployment) , che include informazioni sull'hosting di un' Blazor WebAssembly app come app secondaria IIS.

## <a name="compression"></a>Compressione

Quando Blazor WebAssembly viene pubblicata un'app, l'output viene compresso in modo statico durante la pubblicazione per ridurre le dimensioni dell'app e rimuovere l'overhead per la compressione in fase di esecuzione. Vengono utilizzati gli algoritmi di compressione seguenti:

* [Brotli](https://tools.ietf.org/html/rfc7932) (livello massimo)
* [Gzip](https://tools.ietf.org/html/rfc1952)

Blazor si basa sull'host per gestire i file compressi appropriati. Quando si usa un ASP.NET Core progetto ospitato, il progetto host è in grado di eseguire la negoziazione del contenuto e di servire i file compressi in modo statico. Quando si ospita un' Blazor WebAssembly app autonoma, potrebbe essere necessario lavoro aggiuntivo per garantire che vengano serviti file compressi in modo statico:

* Per `web.config` la configurazione della compressione IIS, vedere la sezione [IIS: Brotli e la compressione gzip](#brotli-and-gzip-compression) . 
* Quando si ospitano soluzioni di hosting statiche che non supportano la negoziazione del contenuto di file compressi in modo statico, ad esempio pagine di GitHub, è consigliabile configurare l'app per recuperare e decodificare i file compressi Brotli:

  * Ottenere il decodificatore JavaScript Brotli dal [repository GitHub Google/Brotli](https://github.com/google/brotli). Il file del decodificatore è denominato `decode.js` e viene trovato nella [ `js` cartella](https://github.com/google/brotli/tree/master/js)del repository.
  
    > [!NOTE]
    > Una regressione è presente nella versione minimizzati dello `decode.js` script ( `decode.min.js` ) nel [repository GitHub Google/brotli](https://github.com/google/brotli). Minimizzare lo script autonomamente o usare il [pacchetto NPM](https://www.npmjs.com/package/brotli) fino a quando non viene risolto il problema [TypeError in decode.min.js (google/brotli #881)](https://github.com/google/brotli/issues/881) . Il codice di esempio in questa sezione usa la versione **unminified** dello script.

  * Aggiornare l'app per l'uso del decodificatore. Modificare il markup all'interno del `<body>` tag di chiusura in `wwwroot/index.html` come segue:
  
    ```html
    <script src="decode.js"></script>
    <script src="_framework/blazor.webassembly.js" autostart="false"></script>
    <script>
      Blazor.start({
        loadBootResource: function (type, name, defaultUri, integrity) {
          if (type !== 'dotnetjs' && location.hostname !== 'localhost') {
            return (async function () {
              const response = await fetch(defaultUri + '.br', { cache: 'no-cache' });
              if (!response.ok) {
                throw new Error(response.statusText);
              }
              const originalResponseBuffer = await response.arrayBuffer();
              const originalResponseArray = new Int8Array(originalResponseBuffer);
              const decompressedResponseArray = BrotliDecode(originalResponseArray);
              const contentType = type === 
                'dotnetwasm' ? 'application/wasm' : 'application/octet-stream';
              return new Response(decompressedResponseArray, 
                { headers: { 'content-type': contentType } });
            })();
          }
        }
      });
    </script>
    ```
 
Per disabilitare la compressione, aggiungere la `BlazorEnableCompression` Proprietà MSBuild al file di progetto dell'app e impostare il valore su `false` :

```xml
<PropertyGroup>
  <BlazorEnableCompression>false</BlazorEnableCompression>
</PropertyGroup>
```

La `BlazorEnableCompression` proprietà può essere passata al [`dotnet publish`](/dotnet/core/tools/dotnet-publish) comando con la sintassi seguente in una shell dei comandi:

```dotnetcli
dotnet publish -p:BlazorEnableCompression=false
```

## <a name="rewrite-urls-for-correct-routing"></a>Riscrivere gli URL per il routing corretto

Il routing delle richieste per i componenti della pagina in un' Blazor WebAssembly app non è semplice come il routing delle richieste in un' Blazor Server app ospitata. Si consideri un' Blazor WebAssembly app con due componenti:

* `Main.razor`: Carica alla radice dell'app e contiene un collegamento al `About` componente ( `href="About"` ).
* `About.razor`: `About` componente.

Quando viene richiesto il documento predefinito dell'app usando la barra degli indirizzi del browser (ad esempio, `https://www.contoso.com/`):

1. Il browser invia una richiesta.
1. Viene restituita la pagina predefinita, in genere `index.html` .
1. `index.html` avvia l'app.
1. Blazoril router viene caricato e il componente viene sottoposto a Razor `Main` rendering.

Nella pagina principale, selezionando il collegamento al `About` componente funziona sul client perché il Blazor router interrompe il browser per effettuare una richiesta su Internet per `www.contoso.com` e serve il componente di cui è stato `About` eseguito il rendering `About` . Tutte le richieste per gli endpoint interni *all'interno dell' Blazor WebAssembly app* funzionano allo stesso modo: le richieste non attivano richieste basate su browser a risorse ospitate su server su Internet. Le richieste vengono gestite internamente dal router.

Se una richiesta viene effettuata usando la barra degli indirizzi del browser per `www.contoso.com/About`, la richiesta ha esito negativo. La risorsa non esiste nell'host Internet dell'app, quindi viene restituita la risposta *404 non trovato*.

Poiché i browser effettuano richieste a host basati su Internet per le pagine lato client, i server Web e i servizi di hosting devono riscrivere tutte le richieste di risorse non fisicamente presenti sul server nella `index.html` pagina. Quando `index.html` viene restituito, il router dell'app Blazor accetta e risponde con la risorsa corretta.

Quando si esegue la distribuzione in un server IIS, è possibile usare il modulo URL Rewrite con il file pubblicato dell'app `web.config` . Per ulteriori informazioni, vedere la sezione [IIS](#iis) .

## <a name="hosted-deployment-with-aspnet-core"></a>Distribuzione ospitata con ASP.NET Core

Una *distribuzione ospitata* serve all' Blazor WebAssembly app per i browser da un' [app ASP.NET Core](xref:index) eseguita in un server Web.

L' Blazor WebAssembly app client viene pubblicata nella `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` cartella dell'app Server, insieme a qualsiasi altra risorsa Web statica dell'app Server. Le due app vengono distribuite insieme. È necessario un server Web in grado di ospitare un'app ASP.NET Core. Per una distribuzione ospitata, Visual Studio include il modello di progetto **Blazor WebAssembly app** ( `blazorwasm` modello quando si usa il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando) con l' **`Hosted`** opzione selezionata ( `-ho|--hosted` quando si usa il `dotnet new` comando).

Per altre informazioni su hosting e distribuzione di app ASP.NET Core, vedere <xref:host-and-deploy/index>.

Per informazioni sulla distribuzione in Servizio app di Azure, vedere <xref:tutorials/publish-to-azure-webapp-using-vs>.

## <a name="hosted-deployment-with-multiple-blazor-webassembly-apps"></a>Distribuzione ospitata con più Blazor WebAssembly app

### <a name="app-configuration"></a>Configurazione delle app

Le Blazor soluzioni ospitate possono gestire più Blazor WebAssembly app.

> [!NOTE]
> L'esempio in questa sezione fa riferimento all'uso di una *soluzione* di Visual Studio, ma l'uso di Visual Studio e di una soluzione di Visual Studio non è necessario per il funzionamento di più app client in uno scenario di app ospitate Blazor WebAssembly . Se non si usa Visual Studio, ignorare il `{SOLUTION NAME}.sln` file e tutti gli altri file creati per Visual Studio.

Nell'esempio seguente:

* L'app client iniziale (prima) è il progetto client predefinito di una soluzione creata dal Blazor WebAssembly modello di progetto. La prima app client è accessibile in un browser dall'URL `/FirstApp` sulla porta 5001 o con un host di `firstapp.com` .
* Alla soluzione viene aggiunta una seconda app client, `SecondBlazorApp.Client` . La seconda app client è accessibile in un browser dall'URL `/SecondApp` sulla porta 5002 o con un host di `secondapp.com` .

Usa una soluzione ospitata esistente Blazor o crea una nuova soluzione dal Blazor modello di progetto ospitato:

* Nel file di progetto dell'app client aggiungere una `<StaticWebAssetBasePath>` proprietà a `<PropertyGroup>` con un valore di `FirstApp` per impostare il percorso di base per gli asset statici del progetto:

  ```xml
  <PropertyGroup>
    ...
    <StaticWebAssetBasePath>FirstApp</StaticWebAssetBasePath>
  </PropertyGroup>
  ```

* Aggiungere una seconda app client alla soluzione:

  * Aggiungere una cartella denominata `SecondClient` alla cartella della soluzione. La cartella della soluzione creata dal modello di progetto contiene le cartelle e i file di soluzione seguenti dopo l' `SecondClient` aggiunta della cartella:
  
    * `Client` cartella
    * `SecondClient` cartella
    * `Server` cartella
    * `Shared` cartella
    * `{SOLUTION NAME}.sln` file
    
    Il segnaposto `{SOLUTION NAME}` è il nome della soluzione.

  * Creare un' Blazor WebAssembly App denominata `SecondBlazorApp.Client` nella `SecondClient` cartella dal modello di Blazor WebAssembly progetto.

  * Nel `SecondBlazorApp.Client` file di progetto dell'app:

    * Aggiungere una `<StaticWebAssetBasePath>` proprietà a `<PropertyGroup>` con un valore di `SecondApp` :

      ```xml
      <PropertyGroup>
        ...
        <StaticWebAssetBasePath>SecondApp</StaticWebAssetBasePath>
      </PropertyGroup>
      ```

    * Aggiungere un riferimento al progetto `Shared` :

      ```xml
      <ItemGroup>
        <ProjectReference Include="..\Shared\{SOLUTION NAME}.Shared.csproj" />
      </ItemGroup>
      ```

      Il segnaposto `{SOLUTION NAME}` è il nome della soluzione.

* Nel file di progetto dell'app Server creare un riferimento al progetto per l' `SecondBlazorApp.Client` app client aggiunta:

  ```xml
  <ItemGroup>
    <ProjectReference Include="..\Client\{SOLUTION NAME}.Client.csproj" />
    <ProjectReference Include="..\SecondClient\SecondBlazorApp.Client.csproj" />
    <ProjectReference Include="..\Shared\{SOLUTION NAME}.Shared.csproj" />
  </ItemGroup>
  ```
  
  Il segnaposto `{SOLUTION NAME}` è il nome della soluzione.

* Nel file dell'app Server `Properties/launchSettings.json` configurare il `applicationUrl` del profilo gheppio ( `{SOLUTION NAME}.Server` ) per accedere alle app client sulle porte 5001 e 5002:

  ```json
  "applicationUrl": "https://localhost:5001;https://localhost:5002",
  ```

* Nel metodo dell'app Server `Startup.Configure` ( `Startup.cs` ) rimuovere le righe seguenti, che vengono visualizzate dopo la chiamata a <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> :

  ```csharp
  app.UseBlazorFrameworkFiles();
  app.UseStaticFiles();

  app.UseRouting();

  app.UseEndpoints(endpoints =>
  {
      endpoints.MapRazorPages();
      endpoints.MapControllers();
      endpoints.MapFallbackToFile("index.html");
  });
  ```

  Aggiungere un middleware che esegue il mapping delle richieste alle app client. Nell'esempio seguente viene configurato il middleware per l'esecuzione nei casi seguenti:

  * La porta di richiesta è 5001 per l'app client originale o 5002 per l'app client aggiunta.
  * L'host della richiesta è `firstapp.com` per l'app client originale o `secondapp.com` per l'app client aggiunta.

    > [!NOTE]
    > L'esempio illustrato in questa sezione richiede una configurazione aggiuntiva per:
    >
    > * Accesso alle app nei domini host di esempio, `firstapp.com` e `secondapp.com` .
    > * Certificati per le app client per abilitare la sicurezza TLS (HTTPS).
    >
    > La configurazione richiesta esula dall'ambito di questo articolo e dipende dalla modalità di hosting della soluzione. Per ulteriori informazioni, vedere gli [articoli host e deploy](xref:host-and-deploy/index).

  Inserire il codice seguente in cui le righe sono state rimosse in precedenza:

  ```csharp
  app.MapWhen(ctx => ctx.Request.Host.Port == 5001 || 
      ctx.Request.Host.Equals("firstapp.com"), first =>
  {
      first.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/FirstApp" + ctx.Request.Path;
          return nxt();
      });

      first.UseBlazorFrameworkFiles("/FirstApp");
      first.UseStaticFiles();
      first.UseStaticFiles("/FirstApp");
      first.UseRouting();

      first.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/FirstApp/{*path:nonfile}", 
              "FirstApp/index.html");
      });
  });
  
  app.MapWhen(ctx => ctx.Request.Host.Port == 5002 || 
      ctx.Request.Host.Equals("secondapp.com"), second =>
  {
      second.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/SecondApp" + ctx.Request.Path;
          return nxt();
      });

      second.UseBlazorFrameworkFiles("/SecondApp");
      second.UseStaticFiles();
      second.UseStaticFiles("/SecondApp");
      second.UseRouting();

      second.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/SecondApp/{*path:nonfile}", 
              "SecondApp/index.html");
      });
  });
  ```

* Nel controller delle previsioni meteorologiche dell'app Server ( `Controllers/WeatherForecastController.cs` ) sostituire la route esistente ( `[Route("[controller]")]` ) `WeatherForecastController` con le route seguenti:

  ```csharp
  [Route("FirstApp/[controller]")]
  [Route("SecondApp/[controller]")]
  ```

  Il middleware aggiunto al metodo dell'app Server `Startup.Configure` precedente modifica le richieste in ingresso in `/WeatherForecast` in `/FirstApp/WeatherForecast` o a `/SecondApp/WeatherForecast` seconda della porta (5001/5002) o del dominio ( `firstapp.com` / `secondapp.com` ). Le route del controller precedenti sono necessarie per restituire i dati meteo dall'app Server alle app client.

### <a name="static-assets-and-class-libraries"></a>Asset statici e librerie di classi

Usare gli approcci seguenti per gli asset statici:

* Quando l'asset si trova nella cartella dell'app client `wwwroot` , fornire i percorsi in genere:

  ```razor
  <img alt="..." src="/{ASSET FILE NAME}" />
  ```

* Quando l'asset si trova nella `wwwroot` cartella di una [ Razor libreria di classi (RCL)](xref:blazor/components/class-libraries), fare riferimento all'asset statico nell'app client secondo le indicazioni contenute nell' [articolo RCL](xref:razor-pages/ui-class#consume-content-from-a-referenced-rcl):

  ```razor
  <img alt="..." src="_content/{LIBRARY NAME}/{ASSET FILE NAME}" />
  ```

<!-- HOLD for reactivation at 5.x

::: moniker range=">= aspnetcore-5.0"

Components provided to a client app by a class library are referenced normally. If any components require stylesheets or JavaScript files, use either of the following approaches to obtain the static assets:

* The client app's `wwwroot/index.html` file can link (`<link>`) to the static assets.
* The component can use the framework's [`Link` component](xref:blazor/fundamentals/additional-scenarios#influence-html-head-tag-elements) to obtain the static assets.

The preceding approaches are demonstrated in the following examples.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

-->

Per i componenti forniti a un'app client da una libreria di classi viene fatto riferimento normalmente. Se i componenti richiedono fogli di stile o file JavaScript, il file dell'app client `wwwroot/index.html` deve includere i collegamenti statici corretti. Questi approcci sono illustrati negli esempi seguenti.

<!-- HOLD for reactivation at 5.x

::: moniker-end

-->

Aggiungere il `Jeep` componente seguente a una delle app client. Il `Jeep` componente USA:

* Un'immagine dalla cartella dell'app client `wwwroot` ( `jeep-cj.png` ).
* Un'immagine da una cartella della [ Razor libreria dei componenti](xref:blazor/components/class-libraries) () aggiunta () `JeepImage` `wwwroot` `jeep-yj.png` .
* Il componente di esempio ( `Component1` ) viene creato automaticamente dal modello di progetto RCL quando la `JeepImage` libreria viene aggiunta alla soluzione.

```razor
@page "/Jeep"

<h1>1979 Jeep CJ-5&trade;</h1>

<p>
    <img alt="1979 Jeep CJ-5&trade;" src="/jeep-cj.png" />
</p>

<h1>1991 Jeep YJ&trade;</h1>

<p>
    <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
</p>

<p>
    <em>Jeep CJ-5</em> and <em>Jeep YJ</em> are a trademarks of 
    <a href="https://www.fcagroup.com">Fiat Chrysler Automobiles</a>.
</p>

<JeepImage.Component1 />
```

> [!WARNING]
> **Non** pubblicare le immagini dei veicoli pubblicamente, a meno che non si possiedano le immagini. In caso contrario, si rischia di violare il copyright.

<!-- HOLD for reactivation at 5.x

::: moniker range=">= aspnetcore-5.0"

The library's `jeep-yj.png` image can also be added to the library's `Component1` component (`Component1.razor`). To provide the `my-component` CSS class to the client app's page, link to the library's stylesheet using the framework's [`Link` component](xref:blazor/fundamentals/additional-scenarios#influence-html-head-tag-elements):

```razor
<div class="my-component">
    <Link href="_content/JeepImage/styles.css" rel="stylesheet" />

    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

An alternative to using the [`Link` component](xref:blazor/fundamentals/additional-scenarios#influence-html-head-tag-elements) is to load the stylesheet from the client app's `wwwroot/index.html` file. This approach makes the stylesheet available to all of the components in the client app:

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

-->

È `jeep-yj.png` possibile aggiungere l'immagine della libreria anche al componente della libreria `Component1` ( `Component1.razor` ):

```razor
<div class="my-component">
    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

Il file dell'app client `wwwroot/index.html` richiede il foglio di stile della libreria con il seguente `<link>` tag aggiunto:

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

<!-- HOLD for reactivation at 5.x

::: moniker-end

-->

Aggiungere la navigazione al `Jeep` componente nel componente dell'app client `NavMenu` ( `Shared/NavMenu.razor` ):

```razor
<li class="nav-item px-3">
    <NavLink class="nav-link" href="Jeep">
        <span class="oi oi-list-rich" aria-hidden="true"></span> Jeep
    </NavLink>
</li>
```

Per ulteriori informazioni su RCL, vedere:

* <xref:blazor/components/class-libraries>
* <xref:razor-pages/ui-class>

## <a name="standalone-deployment"></a>Distribuzione autonoma

Una *distribuzione autonoma* serve l' Blazor WebAssembly app come un set di file statici richiesti direttamente dai client. Qualsiasi file server statico è in grado di gestire l' Blazor app.

Le risorse di distribuzione autonome vengono pubblicate nella `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` cartella.

### <a name="azure-app-service"></a>Servizio app di Azure

Blazor WebAssembly le app possono essere distribuite in app Azure Services in Windows, che ospita l'app in [IIS](#iis).

La distribuzione di un'app autonoma Blazor WebAssembly nel servizio app Azure per Linux non è attualmente supportata. Un'immagine server Linux per ospitare l'app non è disponibile in questo momento. Il lavoro è in corso per abilitare questo scenario.

### <a name="iis"></a>IIS

IIS è un file server statico idoneo per le Blazor app. Per configurare IIS per l'hosting Blazor , vedere [la pagina relativa alla creazione di un sito Web statico in IIS](/iis/manage/creating-websites/scenario-build-a-static-website-on-iis).

Gli asset pubblicati vengono creati nella `/bin/Release/{TARGET FRAMEWORK}/publish` cartella. Ospitare il contenuto della `publish` cartella nel server Web o nel servizio di hosting.

#### <a name="webconfig"></a>web.config

Quando Blazor viene pubblicato un progetto, `web.config` viene creato un file con la seguente configurazione di IIS:

* Vengono impostati tipi MIME per le estensioni di file seguenti:
  * `.dll`: `application/octet-stream`
  * `.json`: `application/json`
  * `.wasm`: `application/wasm`
  * `.woff`: `application/font-woff`
  * `.woff2`: `application/font-woff`
* Viene abilitata la compressione HTTP per i tipi MIME seguenti:
  * `application/octet-stream`
  * `application/wasm`
* Vengono stabilite le regole URL Rewrite Module:
  * Gestire la sottodirectory in cui risiedono gli asset statici dell'app ( `wwwroot/{PATH REQUESTED}` ).
  * Creare il routing di fallback SPA in modo che le richieste di risorse non file vengano reindirizzate al documento predefinito dell'app nella cartella Asset statici ( `wwwroot/index.html` ).
  
#### <a name="use-a-custom-webconfig"></a>Usare un web.config personalizzato

Per usare un `web.config` file personalizzato, inserire il `web.config` file personalizzato nella radice della cartella del progetto. Configurare il progetto per pubblicare asset specifici di IIS usando `PublishIISAssets` nel file di progetto dell'app e pubblicare il progetto:

```xml
<PropertyGroup>
  <PublishIISAssets>true</PublishIISAssets>
</PropertyGroup>
```

#### <a name="install-the-url-rewrite-module"></a>Installare URL Rewrite Module

[URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) è necessario per riscrivere gli URL. Il modulo non viene installato per impostazione predefinita e non è disponibile per l'installazione come funzionalità del servizio ruolo Server Web (IIS). Il modulo deve essere scaricato dal sito Web IIS. Usare Installazione guidata piattaforma Web per installare il modulo:

1. In locale, passare alla [pagina di download di URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads). Per la versione in lingua inglese selezionare **WebPI** per scaricare il programma di installazione di Installazione guidata piattaforma Web. Per altre lingue, selezionare l'architettura appropriata per il server (x86 o x64) per scaricare il programma di installazione.
1. Copiare il programma di installazione nel server. Eseguire il programma di installazione. Selezionare il pulsante **Installa** e accettare le condizioni di licenza. Al termine dell'installazione non è necessario un riavvio del server.

#### <a name="configure-the-website"></a>Configurare il sito Web

Impostare il **percorso fisico** del sito Web sulla cartella dell'app. La cartella contiene:

* Il `web.config` file utilizzato da IIS per configurare il sito Web, incluse le regole di reindirizzamento richieste e i tipi di contenuto del file.
* Cartella degli asset statici dell'app.

#### <a name="host-as-an-iis-sub-app"></a>Ospitare come applicazione secondaria IIS

Se un'app autonoma è ospitata come una sub-app IIS, eseguire una delle operazioni seguenti:

* Disabilitare il gestore del modulo ASP.NET Core ereditato.

  Rimuovere il gestore nel Blazor file pubblicato dell'app `web.config` aggiungendo una `<handlers>` sezione al file:

  ```xml
  <handlers>
    <remove name="aspNetCore" />
  </handlers>
  ```

* Disabilitare l'ereditarietà della sezione radice (padre) dell'app `<system.webServer>` usando un `<location>` elemento con `inheritInChildApplications` impostato su `false` :

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <handlers>
          <add name="aspNetCore" ... />
        </handlers>
        <aspNetCore ... />
      </system.webServer>
    </location>
  </configuration>
  ```

La rimozione del gestore o la disabilitazione dell'ereditarietà viene eseguita oltre alla [configurazione del percorso di base dell'applicazione](xref:blazor/host-and-deploy/index#app-base-path). Impostare il percorso di base dell'app nel file dell'app `index.html` sull'alias IIS usato durante la configurazione della Sub-app in IIS.

#### <a name="brotli-and-gzip-compression"></a>Compressione Brotli e gzip

*Questa sezione si applica solo alle app autonome Blazor WebAssembly . Blazor le app ospitate usano un file di ASP.NET Core app predefinito `web.config` , non il file collegato in questa sezione.*

IIS può essere configurato tramite `web.config` per gestire asset Brotli o gzip compressi Blazor per le Blazor WebAssembly app autonome. Per un esempio di file di configurazione, vedere [`web.config`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/web.config?raw=true) .

È possibile che sia necessaria una configurazione aggiuntiva del file di esempio `web.config` negli scenari seguenti:

* La specifica dell'app chiama per uno dei seguenti elementi:
  * Vengono serviti file compressi che non sono configurati dal file di esempio `web.config` .
  * Conservazione dei file compressi configurati dal file di esempio `web.config` in un formato non compresso.
* La configurazione IIS del server (ad esempio, `applicationHost.config` ) fornisce le impostazioni predefinite di IIS a livello di server. A seconda della configurazione a livello di server, è possibile che l'app richieda una configurazione IIS diversa rispetto a quella `web.config` contenuta nel file di esempio.

#### <a name="troubleshooting"></a>Risoluzione dei problemi

Se si riceve un *errore interno del server 500* e Gestione IIS genera errori durante il tentativo di accedere alla configurazione del sito Web, verificare che sia installato URL Rewrite Module. Quando il modulo non è installato, il `web.config` file non può essere analizzato da IIS. In questo modo si impedisce al gestore IIS di caricare la configurazione del sito Web e il sito Web dai Blazor file statici del servizio.

Per altre informazioni sulla risoluzione dei problemi relativi alle distribuzioni in IIS, vedere <xref:test/troubleshoot-azure-iis>.

### <a name="azure-storage"></a>Archiviazione di Azure

L'hosting di file statici di [archiviazione di Azure](/azure/storage/) consente l'hosting di app senza server Blazor . Sono supportati nomi di dominio personalizzati, la rete per la distribuzione di contenuti (rete CDN) di Azure e HTTPS.

Quando il servizio BLOB è abilitato per l'hosting di siti Web statici in un account di archiviazione:

* Impostare **Nome del documento di indice** su `index.html`.
* Impostare **Percorso del documento di errore** su `index.html`. Razor i componenti e altri endpoint non di file non si trovano in percorsi fisici nel contenuto statico archiviato dal servizio BLOB. Quando viene ricevuta una richiesta per una di queste risorse che il Blazor router deve gestire, l'errore *404 non trovato* generato dal servizio BLOB instrada la richiesta al **percorso del documento di errore**. `index.html`Viene restituito il BLOB e il Blazor router carica ed elabora il percorso.

Se i file non vengono caricati in fase di esecuzione a causa di tipi MIME non appropriati nelle intestazioni dei file `Content-Type` , eseguire una delle azioni seguenti:

* Configurare gli strumenti per impostare i tipi MIME corretti ( `Content-Type` intestazioni) quando vengono distribuiti i file.
* Modificare i tipi MIME ( `Content-Type` intestazioni) per i file dopo la distribuzione dell'app.

  In Storage Explorer (portale di Azure) per ogni file:
  
  1. Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà**.
  1. Impostare **ContentType** e selezionare il pulsante **Salva** .

Per altre informazioni, vedere [Hosting di siti Web statici in Archiviazione di Azure](/azure/storage/blobs/storage-blob-static-website).

### <a name="nginx"></a>Nginx

Il `nginx.conf` file seguente è stato semplificato per illustrare come configurare Nginx per inviare il `index.html` file ogni volta che non riesce a trovare un file corrispondente sul disco.

```
events { }
http {
    server {
        listen 80;

        location / {
            root      /usr/share/nginx/html;
            try_files $uri $uri/ /index.html =404;
        }
    }
}
```

Quando si imposta il [limite di velocità](https://www.nginx.com/blog/rate-limiting-nginx/#bursts) di espansione nginx con, le [`limit_req`](https://nginx.org/docs/http/ngx_http_limit_req_module.html#limit_req) Blazor WebAssembly app possono richiedere un `burst` valore di parametro di grandi dimensioni per contenere il numero relativamente elevato di richieste effettuate da un'app. Inizialmente, impostare il valore su almeno 60:

```
http {
    server {
        ...

        location / {
            ...

            limit_req zone=one burst=60 nodelay;
        }
    }
}
```

Aumentare il valore se browser Developer Tools o uno strumento di traffico di rete indica che le richieste ricevono un codice di stato *503-servizio non disponibile* .

Per altre informazioni sulla configurazione del server Web Nginx in ambiente di produzione, vedere [Creating NGINX Plus and NGINX Configuration Files](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/) (Creazione di file di configurazione NGINX Plus e NGINX).

### <a name="apache"></a>Apache

Per distribuire un' Blazor WebAssembly app in CentOS 7 o versione successiva:

1. Creare il file di configurazione Apache. L'esempio seguente è un file di configurazione semplificato ( `blazorapp.config` ):

   ```
   <VirtualHost *:80>
       ServerName www.example.com
       ServerAlias *.example.com

       DocumentRoot "/var/www/blazorapp"
       ErrorDocument 404 /index.html

       AddType application/wasm .wasm
       AddType application/octet-stream .dll
   
       <Directory "/var/www/blazorapp">
           Options -Indexes
           AllowOverride None
       </Directory>

       <IfModule mod_deflate.c>
           AddOutputFilterByType DEFLATE text/css
           AddOutputFilterByType DEFLATE application/javascript
           AddOutputFilterByType DEFLATE text/html
           AddOutputFilterByType DEFLATE application/octet-stream
           AddOutputFilterByType DEFLATE application/wasm
           <IfModule mod_setenvif.c>
           BrowserMatch ^Mozilla/4 gzip-only-text/html
           BrowserMatch ^Mozilla/4.0[678] no-gzip
           BrowserMatch bMSIE !no-gzip !gzip-only-text/html
       </IfModule>
       </IfModule>

       ErrorLog /var/log/httpd/blazorapp-error.log
       CustomLog /var/log/httpd/blazorapp-access.log common
   </VirtualHost>
   ```

1. Inserire il file di configurazione Apache nella `/etc/httpd/conf.d/` Directory, ovvero la directory di configurazione predefinita di Apache in CentOS 7.

1. Inserire i file dell'app nella `/var/www/blazorapp` Directory (il percorso specificato `DocumentRoot` nel file di configurazione).

1. Riavviare il servizio Apache.

Per altre informazioni, vedere [`mod_mime`](https://httpd.apache.org/docs/2.4/mod/mod_mime.html) e [`mod_deflate`](https://httpd.apache.org/docs/current/mod/mod_deflate.html).

### <a name="github-pages"></a>GitHub Pages

Per gestire le riscritture URL, aggiungere un `wwwroot/404.html` file con uno script che gestisce il reindirizzamento della richiesta alla `index.html` pagina. Per un esempio, vedere il [ Blazor repository GitHub SteveSandersonMS/OnGitHubPages](https://github.com/SteveSandersonMS/BlazorOnGitHubPages):

* [`wwwroot/404.html`](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/wwwroot/404.html)
* [Sito Live](https://stevesandersonms.github.io/BlazorOnGitHubPages/))

Quando si usa un sito del progetto anziché un sito dell'organizzazione, aggiornare il `<base>` tag in `wwwroot/index.html` . Impostare il `href` valore dell'attributo sul nome del repository GitHub con una barra finale (ad esempio, `/my-repository/` ). Nel [ Blazor repository GitHub SteveSandersonMS/OnGitHubPages](https://github.com/SteveSandersonMS/BlazorOnGitHubPages)la base `href` viene aggiornata in fase di pubblicazione dal [ `.github/workflows/main.yml` file di configurazione](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/.github/workflows/main.yml).

> [!NOTE]
> Il [ Blazor repository GitHub SteveSandersonMS/OnGitHubPages](https://github.com/SteveSandersonMS/BlazorOnGitHubPages) non è di proprietà, è stato mantenuto o supportato da .NET Foundation o Microsoft.

## <a name="host-configuration-values"></a>Valori di configurazione dell'host

le [ Blazor WebAssembly app](xref:blazor/hosting-models#blazor-webassembly) possono accettare i valori di configurazione host seguenti come argomenti della riga di comando in fase di esecuzione nell'ambiente di sviluppo.

### <a name="content-root"></a>Radice del contenuto

L' `--contentroot` argomento imposta il percorso assoluto della directory che contiene i file di contenuto dell'app ([radice del contenuto](xref:fundamentals/index#content-root)). Negli esempi seguenti `/content-root-path` è il percorso radice del contenuto dell'app.

* Passare l'argomento quando si esegue localmente l'app a un prompt dei comandi. Dalla directory dell'app, eseguire:

  ```dotnetcli
  dotnet run --contentroot=/content-root-path
  ```

* Aggiungere una voce al file dell'app `launchSettings.json` nel profilo di **IIS Express** . Questa impostazione viene usata quando l'app viene eseguita con il debugger di Visual Studio e da un prompt dei comandi con `dotnet run`.

  ```json
  "commandLineArgs": "--contentroot=/content-root-path"
  ```

* In Visual Studio specificare l'argomento in **Proprietà**  >  **debug**  >  **argomenti dell'applicazione**. L'impostazione dell'argomento nella pagina delle proprietà di Visual Studio consente di aggiungere l'argomento al `launchSettings.json` file.

  ```console
  --contentroot=/content-root-path
  ```

### <a name="path-base"></a>Base del percorso

L' `--pathbase` argomento imposta il percorso di base dell'app per un'app eseguita localmente con un percorso URL relativo non radice (il `<base>` tag `href` è impostato su un percorso diverso da `/` per la gestione temporanea e la produzione). Negli esempi seguenti `/relative-URL-path` è la base del percorso dell'app. Per altre informazioni, vedere [percorso di base dell'app](xref:blazor/host-and-deploy/index#app-base-path).

> [!IMPORTANT]
> A differenza del percorso specificato per `href` del tag `<base>`, non includere una barra finale (`/`) quando si passa il valore dell'argomento `--pathbase`. Se il percorso di base dell'app non viene specificato nel tag `<base>` come `<base href="/CoolApp/">` (include una barra finale), passare il valore dell'argomento della riga di comando come `--pathbase=/CoolApp` (senza barra finale).

* Passare l'argomento quando si esegue localmente l'app a un prompt dei comandi. Dalla directory dell'app, eseguire:

  ```dotnetcli
  dotnet run --pathbase=/relative-URL-path
  ```

* Aggiungere una voce al file dell'app `launchSettings.json` nel profilo di **IIS Express** . Questa impostazione viene usata quando l'app viene eseguita con il debugger di Visual Studio e da un prompt dei comandi con `dotnet run`.

  ```json
  "commandLineArgs": "--pathbase=/relative-URL-path"
  ```

* In Visual Studio specificare l'argomento in **Proprietà**  >  **debug**  >  **argomenti dell'applicazione**. L'impostazione dell'argomento nella pagina delle proprietà di Visual Studio consente di aggiungere l'argomento al `launchSettings.json` file.

  ```console
  --pathbase=/relative-URL-path
  ```

### <a name="urls"></a>URL

L'argomento `--urls` imposta gli indirizzi IP o gli indirizzi host con le porte e i protocolli su cui eseguire l'ascolto per le richieste.

* Passare l'argomento quando si esegue localmente l'app a un prompt dei comandi. Dalla directory dell'app, eseguire:

  ```dotnetcli
  dotnet run --urls=http://127.0.0.1:0
  ```

* Aggiungere una voce al file dell'app `launchSettings.json` nel profilo di **IIS Express** . Questa impostazione viene usata quando l'app viene eseguita con il debugger di Visual Studio e da un prompt dei comandi con `dotnet run`.

  ```json
  "commandLineArgs": "--urls=http://127.0.0.1:0"
  ```

* In Visual Studio specificare l'argomento in **Proprietà**  >  **debug**  >  **argomenti dell'applicazione**. L'impostazione dell'argomento nella pagina delle proprietà di Visual Studio consente di aggiungere l'argomento al `launchSettings.json` file.

  ```console
  --urls=http://127.0.0.1:0
  ```

::: moniker range=">= aspnetcore-5.0"

## <a name="configure-the-trimmer"></a>Configurare il trimmer

Blazor esegue il trimming del linguaggio intermedio (IL) in ogni build di rilascio per rimuovere IL linguaggio intermedio non necessario dagli assembly di output. Per altre informazioni, vedere <xref:blazor/host-and-deploy/configure-trimmer>.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="configure-the-linker"></a>Configurare il linker

Blazor esegue il collegamento Intermediate Language (IL) a ogni build di rilascio per rimuovere IL linguaggio intermedio non necessario dagli assembly di output. Per altre informazioni, vedere <xref:blazor/host-and-deploy/configure-linker>.

::: moniker-end

## <a name="custom-boot-resource-loading"></a>Caricamento di risorse di avvio personalizzate

Un' Blazor WebAssembly app può essere inizializzata con la `loadBootResource` funzione per eseguire l'override del meccanismo di caricamento delle risorse di avvio predefinito. Utilizzare `loadBootResource` per gli scenari seguenti:

* Consente agli utenti di caricare risorse statiche, ad esempio dati del fuso orario o `dotnet.wasm` da una rete CDN.
* Caricare gli assembly compressi usando una richiesta HTTP e decomprimerli sul client per gli host che non supportano il recupero di contenuti compressi dal server.
* Eseguire l'alias delle risorse con un nome diverso reindirizzando ogni `fetch` richiesta a un nuovo nome.

`loadBootResource` i parametri vengono visualizzati nella tabella seguente.

| Parametro    | Descrizione |
| ------------ | ----------- |
| `type`       | Tipo di risorsa. Tipi di consentiti: `assembly` , `pdb` , `dotnetjs` , `dotnetwasm` , `timezonedata` |
| `name`       | Nome della risorsa. |
| `defaultUri` | URI relativo o assoluto della risorsa. |
| `integrity`  | Stringa di integrità che rappresenta il contenuto previsto nella risposta. |

`loadBootResource` restituisce uno dei seguenti elementi per eseguire l'override del processo di caricamento:

* Stringa URI. Nell'esempio seguente ( `wwwroot/index.html` ), i file seguenti vengono serviti da una rete CDN all'indirizzo `https://my-awesome-cdn.com/` :

  * `dotnet.*.js`
  * `dotnet.wasm`
  * Dati TimeZone

  ```html
  ...

  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        console.log(`Loading: '${type}', '${name}', '${defaultUri}', '${integrity}'`);
        switch (type) {
          case 'dotnetjs':
          case 'dotnetwasm':
          case 'timezonedata':
            return `https://my-awesome-cdn.com/blazorwebassembly/3.2.0/${name}`;
        }
      }
    });
  </script>
  ```

* `Promise<Response>`. Passare il `integrity` parametro in un'intestazione per mantenere il comportamento predefinito di controllo dell'integrità.

  L'esempio seguente ( `wwwroot/index.html` ) aggiunge un'intestazione HTTP personalizzata alle richieste in uscita e passa il `integrity` parametro tramite alla `fetch` chiamata:
  
  ```html
  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        return fetch(defaultUri, { 
          cache: 'no-cache',
          integrity: integrity,
          headers: { 'MyCustomHeader': 'My custom value' }
        });
      }
    });
  </script>
  ```

* `null`/`undefined`, che comporta il comportamento di caricamento predefinito.

Per consentire il caricamento delle risorse tra le origini, le origini esterne devono restituire le intestazioni CORS necessarie per i browser. Per impostazione predefinita, CDNs fornisce in genere le intestazioni obbligatorie.

È sufficiente specificare i tipi per i comportamenti personalizzati. I tipi non specificati in `loadBootResource` vengono caricati dal Framework in base ai relativi comportamenti di caricamento predefiniti.

## <a name="change-the-filename-extension-of-dll-files"></a>Modificare l'estensione di file DLL

Se è necessario modificare le estensioni dei file pubblicati dell'app `.dll` , seguire le istruzioni riportate in questa sezione.

Dopo la pubblicazione dell'app, usare uno script della shell o una pipeline di compilazione DevOps per rinominare `.dll` i file in modo da usare un'estensione di file diversa. Specificare come destinazione i `.dll` file nella `wwwroot` directory dell'output pubblicato dell'app (ad esempio, `{CONTENT ROOT}/bin/Release/netstandard2.1/publish/wwwroot` ).

Negli esempi seguenti, `.dll` i file vengono rinominati per l'utilizzo dell' `.bin` estensione di file.

In Windows:

```powershell
dir .\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content .\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content .\_framework\blazor.boot.json
```

Se gli asset di lavoro del servizio sono anche in uso, aggiungere il comando seguente:

```powershell
((Get-Content .\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content .\service-worker-assets.js
```

In Linux o macOS:

```console
for f in _framework/_bin/*; do mv "$f" "`echo $f | sed -e 's/\.dll/.bin/g'`"; done
sed -i 's/\.dll"/.bin"/g' _framework/blazor.boot.json
```

Se gli asset di lavoro del servizio sono anche in uso, aggiungere il comando seguente:

```console
sed -i 's/\.dll"/.bin"/g' service-worker-assets.js
```
   
Per usare un'estensione di file diversa da `.bin` , sostituire `.bin` nei comandi precedenti.

Per risolvere i `blazor.boot.json.gz` file compressi e `blazor.boot.json.br` , adottare uno degli approcci seguenti:

* Rimuovere i `blazor.boot.json.gz` file compressi e `blazor.boot.json.br` . La compressione è disabilitata con questo approccio.
* Ricomprimere il `blazor.boot.json` file aggiornato.

Le linee guida precedenti si applicano anche quando le risorse di lavoro del servizio sono in uso. Rimuovere o ricomprimere `wwwroot/service-worker-assets.js.br` e `wwwroot/service-worker-assets.js.gz` . In caso contrario, i controlli di integrità dei file avranno esito negativo nel browser.

Nell'esempio di Windows seguente viene usato uno script di PowerShell inserito nella radice del progetto.

`ChangeDLLExtensions.ps1:`:

```powershell
param([string]$filepath,[string]$tfm)
dir $filepath\bin\Release\$tfm\wwwroot\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json
Remove-Item $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json.gz
```

Se gli asset di lavoro del servizio sono anche in uso, aggiungere il comando seguente:

```powershell
((Get-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js
```

Nel file di progetto, lo script viene eseguito dopo la pubblicazione dell'app:

```xml
<Target Name="ChangeDLLFileExtensions" AfterTargets="Publish" Condition="'$(Configuration)'=='Release'">
  <Exec Command="powershell.exe -command &quot;&amp; { .\ChangeDLLExtensions.ps1 '$(SolutionDir)' '$(TargetFramework)'}&quot;" />
</Target>
```

> [!NOTE]
> Quando si rinominano e si caricano in modalità lazy gli stessi assembly, vedere le istruzioni in <xref:blazor/webassembly-lazy-load-assemblies#onnavigateasync-events-and-renamed-assembly-files> .

## <a name="resolve-integrity-check-failures"></a>Risolvere gli errori di controllo dell'integrità

Quando Blazor WebAssembly Scarica i file di avvio di un'app, indica al browser di eseguire controlli di integrità sulle risposte. Usa le informazioni nel `blazor.boot.json` file per specificare i valori hash SHA-256 previsti per `.dll` , `.wasm` e altri file. Questa operazione è utile per i motivi seguenti:

* Garantisce che non si rischi il caricamento di un set di file incoerente, ad esempio se viene applicata una nuova distribuzione al server Web mentre è in corso il download dei file dell'applicazione. I file incoerenti possono causare un comportamento indefinito.
* Garantisce che il browser dell'utente non memorizza mai nella cache le risposte incoerenti o non valide, che potrebbero impedire l'avvio dell'app anche se aggiornano manualmente la pagina.
* Consente di memorizzare nella cache le risposte in modo sicuro e non di controllare le modifiche sul lato server fino a quando non si modificano gli hash SHA-256 previsti, quindi i carichi di pagina successivi coinvolgono un minor numero di richieste e vengono completate molto più velocemente.

Se il server Web restituisce risposte non corrispondenti agli hash SHA-256 previsti, nella console per sviluppatori del browser verrà visualizzato un errore simile al seguente:

> Non è stato possibile trovare un digest valido nell'attributo ' Integrity ' per la risorsa ' https://myapp.example.com/\_framework/My BlazorApp.dll' con integrità di SHA-256 calcolata ' IIa70iwvmEg5WiDV17OpQ5eCztNYqL186J56852RpJY ='. La risorsa è stata bloccata.

Nella maggior parte dei casi, *non* si tratta di un problema con il controllo dell'integrità. Al contrario, significa che è presente un altro problema e che il controllo di integrità è in grado di avvisare dell'altro problema.

### <a name="diagnosing-integrity-problems"></a>Diagnosi dei problemi di integrità

Quando viene compilata un'app, il `blazor.boot.json` manifesto generato descrive gli hash SHA-256 delle risorse di avvio (ad esempio,, `.dll` `.wasm` e altri file) nel momento in cui viene generato l'output di compilazione. Il controllo di integrità passa a condizione che gli hash SHA-256 `blazor.boot.json` corrispondano ai file recapitati al browser.

I motivi più comuni per cui questo errore sono:

 * La risposta del server Web è un errore (ad esempio, *404-non trovato* o un *errore del server interno 500*) invece del file richiesto dal browser. Viene segnalato dal browser come un errore di controllo dell'integrità e non come un errore di risposta.
 * Un elemento ha modificato il contenuto dei file tra la compilazione e il recapito dei file nel browser. Questo problema può verificarsi:
   * Se gli strumenti di compilazione o di compilazione modificano manualmente l'output di compilazione.
   * Se alcuni aspetti del processo di distribuzione hanno modificato i file. Se, ad esempio, si usa un meccanismo di distribuzione basato su git, tenere presente che git converte in modo trasparente le terminazioni riga di tipo Windows in terminazioni riga di tipo Unix se si esegue il commit di file in Windows e li si estrae in Linux. La modifica delle terminazioni di riga del file modifica gli hash SHA-256. Per evitare questo problema, considerare l' [utilizzo `.gitattributes` di per considerare gli artefatti di compilazione come `binary` file](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes).
   * Il server Web modifica il contenuto del file come parte del servizio. Ad esempio, alcune reti di distribuzione del contenuto (CDNs) tentano automaticamente di [minimizzare](xref:client-side/bundling-and-minification#minification) HTML, quindi la modifica. Potrebbe essere necessario disabilitare queste funzionalità.

Per diagnosticare quali di queste condizioni si applicano nel caso:

 1. Si noti il file che sta attivando l'errore leggendo il messaggio di errore.
 1. Aprire gli strumenti di sviluppo del browser ed esaminare la scheda *rete* . Se necessario, ricaricare la pagina per visualizzare l'elenco di richieste e risposte. Individuare il file che sta attivando l'errore in tale elenco.
 1. Verificare il codice di stato HTTP nella risposta. Se il server restituisce un valore diverso da *200-OK* (o un altro codice di stato 2xx), è necessario un problema sul lato server da diagnosticare. Ad esempio, il codice di stato 403 indica che si è verificato un problema di autorizzazione, mentre il codice di stato 500 indica che il server ha esito negativo in modo non specificato. Consultare log lato server per diagnosticare e correggere l'app.
 1. Se il codice di stato è *200-OK* per la risorsa, esaminare il contenuto della risposta negli strumenti di sviluppo del browser e verificare che il contenuto corrisponda ai dati previsti. Un problema comune, ad esempio, è quello di configurare correttamente il routing, in modo che le richieste restituiscano i `index.html` dati anche per altri file. Assicurarsi che le risposte alle `.wasm` richieste siano file binari webassembly e che le risposte alle `.dll` richieste siano file binari dell'assembly .NET. In caso contrario, si disporrà di un problema di routing lato server da diagnosticare.
 1. Cercare per convalidare l'output pubblicato e distribuito dell'app con lo script di PowerShell per la [risoluzione dei problemi di integrità](#troubleshoot-integrity-powershell-script).

Se si conferma che il server restituisce dati corretti in modo plausibile, è necessario modificare il contenuto tra la compilazione e il recapito del file. Per esaminare questa operazione:

 * Esaminare il meccanismo di distribuzione e la combinazione di elementi di compilazione nel caso in cui modifichino i file dopo la compilazione dei file. Un esempio è quando git trasforma le terminazioni riga file, come descritto in precedenza.
 * Esaminare la configurazione del server Web o della rete CDN nel caso in cui siano configurati per modificare dinamicamente le risposte, ad esempio provando a minimizzare HTML. Il server Web implementa la compressione HTTP (ad esempio, restituendo `content-encoding: br` o `content-encoding: gzip` ), perché questo non influisce sul risultato dopo la decompressione. Tuttavia, non è corretto per il server Web modificare i dati *non* compressi.

### <a name="troubleshoot-integrity-powershell-script"></a>Risolvere i problemi dello script di PowerShell di integrità

Usare lo [`integrity.ps1`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/integrity.ps1?raw=true) script di PowerShell per convalidare un'app pubblicata e distribuita Blazor . Lo script viene fornito come punto di partenza quando l'app presenta problemi di integrità che il Blazor Framework non è in grado di identificare. Potrebbe essere necessario personalizzare lo script per le app.

Lo script controlla i file nella `publish` cartella e scaricati dall'app distribuita per rilevare i problemi nei diversi manifesti che contengono hash di integrità. Questi controlli dovrebbero rilevare i problemi più comuni:

* È stato modificato un file nell'output pubblicato senza rendersene conto.
* L'app non è stata distribuita correttamente nella destinazione di distribuzione o è stata modificata nell'ambiente della destinazione di distribuzione.
* Esistono differenze tra l'app distribuita e l'output della pubblicazione dell'app.

Richiamare lo script con il comando seguente in una shell dei comandi di PowerShell:

```powershell
.\integrity.ps1 {BASE URL} {PUBLISH OUTPUT FOLDER}
```

Segnaposto

* `{BASE URL}`: URL dell'app distribuita.
* `{PUBLISH OUTPUT FOLDER}`: Percorso della cartella o della posizione dell'app in `publish` cui è pubblicata l'app per la distribuzione.

> [!NOTE]
> Per clonare il `dotnet/AspNetCore.Docs` repository GitHub in un sistema che usa [BitDefender](https://www.bitdefender.com) virus scanner, aggiungere un'eccezione a BitDefender per lo `integrity.ps1` script. Aggiungere l'eccezione a BitDefender prima di clonare il repository per evitare che lo script venga messo in quarantena dal programma antivirus. L'esempio seguente è un percorso tipico dello script per il repository clonato in un sistema Windows. Modificare il percorso in base alle esigenze. Il segnaposto `{USER}` è il segmento del percorso dell'utente.
>
> ```
> C:\Users\{USER}\Documents\GitHub\AspNetCore.Docs\aspnetcore\blazor\host-and-deploy\webassembly\_samples\integrity.ps1
> ```

### <a name="disable-integrity-checking-for-non-pwa-apps"></a>Disabilitare il controllo dell'integrità per le app non PWA

Nella maggior parte dei casi, non disabilitare il controllo dell'integrità. La disabilitazione del controllo di integrità non risolve il problema sottostante che ha causato le risposte impreviste e comporta la perdita dei vantaggi elencati in precedenza.

In alcuni casi non è possibile fare affidamento sul server Web per restituire risposte coerenti e non è possibile scegliere di disabilitare i controlli di integrità. Per disabilitare i controlli di integrità, aggiungere il codice seguente a un gruppo di proprietà nel Blazor WebAssembly file del progetto `.csproj` :

```xml
<BlazorCacheBootResources>false</BlazorCacheBootResources>
```

`BlazorCacheBootResources` Disabilita inoltre il Blazor comportamento predefinito di memorizzazione nella cache di `.dll` , `.wasm` e altri file basati sui rispettivi hash SHA-256, perché la proprietà indica che non è possibile fare affidamento sugli hash SHA-256 per correttezza. Anche con questa impostazione, la cache HTTP normale del browser può comunque memorizzare nella cache tali file, ma indipendentemente dalla configurazione del server Web e dalle `cache-control` intestazioni che svolge.

> [!NOTE]
> La `BlazorCacheBootResources` proprietà non Disabilita i controlli di integrità per [le applicazioni Web progressive (PWA)](xref:blazor/progressive-web-app). Per informazioni aggiuntive relative a PWA, vedere la sezione [disabilitare il controllo dell'integrità per PWA](#disable-integrity-checking-for-pwas) .

### <a name="disable-integrity-checking-for-pwas"></a>Disabilitare il controllo dell'integrità per PWA

Blazoril modello di applicazione Web progressiva (PWA) contiene un `service-worker.published.js` file suggerito che è responsabile del recupero e dell'archiviazione dei file dell'applicazione per l'uso offline. Si tratta di un processo separato dal meccanismo di avvio normale dell'app e ha una propria logica di controllo dell'integrità separata.

All'interno del `service-worker.published.js` file è presente la riga seguente:

```javascript
.map(asset => new Request(asset.url, { integrity: asset.hash }));
```

Per disabilitare il controllo dell'integrità, rimuovere il `integrity` parametro cambiando la riga come indicato di seguito:

```javascript
.map(asset => new Request(asset.url));
```

Anche in questo caso, la disabilitazione del controllo dell'integrità comporta la perdita delle garanzie di sicurezza offerte dal controllo dell'integrità. Ad esempio, esiste il rischio che se il browser dell'utente memorizza nella cache l'app nel momento esatto in cui si distribuisce una nuova versione, potrebbe memorizzare nella cache alcuni file della distribuzione precedente e alcuni dalla nuova distribuzione. In tal caso, l'app si blocca in uno stato di interruzione finché non si distribuisce un altro aggiornamento.
