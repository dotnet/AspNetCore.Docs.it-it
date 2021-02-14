---
title: Creazione di applicazioni Web progressive con ASP.NET Core Blazor WebAssembly
author: guardrex
description: Informazioni su come creare un' Blazor applicazione Web progressiva basata su (PWA) che usa le funzionalità moderne del browser per comportarsi come un'app desktop.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/11/2021
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
uid: blazor/progressive-web-app
ms.openlocfilehash: 515da543fc6b6cca0b90968b154d91b611ea3345
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280549"
---
# <a name="build-progressive-web-applications-with-aspnet-core-blazor-webassembly"></a>Creazione di applicazioni Web progressive con ASP.NET Core Blazor WebAssembly

Un'applicazione Web progressiva (PWA) è in genere un'applicazione a pagina singola (SPA) che usa le API e le funzionalità moderne del browser per comportarsi come un'app desktop. Blazor WebAssembly è una piattaforma di app Web lato client basata su standard, pertanto può usare qualsiasi API del browser, incluse le API di PWA richieste per le funzionalità seguenti:

* Lavorare offline e caricare immediatamente, indipendentemente dalla velocità di rete.
* In esecuzione nella propria finestra dell'app, non solo in una finestra del browser.
* Viene avviato dal menu Start del sistema operativo dell'host, ancorato o dalla schermata iniziale.
* Ricezione di notifiche push da un server back-end, anche quando l'utente non usa l'app.
* Aggiornamento automatico in background.

La parola *progressive* viene usata per descrivere tali app perché:

* Un utente potrebbe prima di tutto individuare e usare l'app nel Web browser come qualsiasi altra SPA.
* Successivamente, l'utente procede all'installazione nel sistema operativo e Abilita le notifiche push.

## <a name="create-a-project-from-the-pwa-template"></a>Creare un progetto dal modello PWA

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Quando si crea una nuova **Blazor WebAssembly app** nella finestra di dialogo **Crea un nuovo progetto** , selezionare la casella di controllo **applicazione Web progressiva** :

![La casella di controllo ' applicazione Web progressiva ' è selezionata nella finestra di dialogo nuovo progetto di Visual Studio.](progressive-web-app/_static/image1.png)

<!--

# [Visual Studio for Mac](#tab/visual-studio-mac)

-->

# <a name="visual-studio-code--net-core-cli"></a>[Visual Studio Code/Interfaccia della riga di comando di .NET Core](#tab/visual-studio-code+netcore-cli)

Usare il comando seguente per creare un progetto PWA in una shell dei comandi con l' `--pwa` opzione:

```dotnetcli
dotnet new blazorwasm -o MyBlazorPwa --pwa
```

Nel comando precedente, l' `-o|--output` opzione Crea una nuova cartella per l'app denominata `MyBlazorPwa` .

---

Facoltativamente, è possibile configurare PWA per un'app creata dal modello di ASP.NET Core Hosted. Lo scenario PWA è indipendente dal modello di hosting.

## <a name="convert-an-existing-blazor-webassembly-app-into-a-pwa"></a>Convertire un' Blazor WebAssembly app esistente in un PWA

Convertire un' Blazor WebAssembly app esistente in un PWA seguendo le istruzioni riportate in questa sezione.

Nel file di progetto dell'app:

* Aggiungere la `ServiceWorkerAssetsManifest` proprietà seguente a `PropertyGroup` :

  ```xml
    ...
    <ServiceWorkerAssetsManifest>service-worker-assets.js</ServiceWorkerAssetsManifest>
  </PropertyGroup>
   ```

* Aggiungere l' `ServiceWorker` elemento seguente a un `ItemGroup` :

  ```xml
  <ItemGroup>
    <ServiceWorker Include="wwwroot\service-worker.js" 
      PublishedContent="wwwroot\service-worker.published.js" />
  </ItemGroup>
  ```

Per ottenere risorse statiche, usare **uno** degli approcci seguenti:

::: moniker range=">= aspnetcore-5.0"

* Creare un nuovo progetto PWA separato con il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando in una shell dei comandi:

  ```dotnetcli
  dotnet new blazorwasm -o MyBlazorPwa --pwa
  ```
  
  Nel comando precedente, l' `-o|--output` opzione Crea una nuova cartella per l'app denominata `MyBlazorPwa` .
  
  Se non si converte un'app per la versione più recente, passare l' `-f|--framework` opzione. Nell'esempio seguente viene creata l'app per ASP.NET Core versione 3,1:
  
  ```dotnetcli
  dotnet new blazorwasm -o MyBlazorPwa --pwa -f netcoreapp3.1
  ```

* Passare al repository GitHub ASP.NET Core all'URL seguente, che si collega all'origine e alle risorse di riferimento della versione 5,0. Se non si converte un'app per la versione 5,0, selezionare la versione che si sta usando dall'elenco a discesa **Cambia rami o tag** applicabile all'app.

  [cartella del modello di progetto DotNet/aspnetcore (versione 5,0) Blazor WebAssembly `wwwroot`](https://github.com/dotnet/aspnetcore/tree/release/5.0/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/wwwroot)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* Creare un nuovo progetto PWA separato con il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando in una shell dei comandi. Passare l' `-f|--framework` opzione per selezionare la versione. Nell'esempio seguente viene creata l'app per ASP.NET Core versione 3,1:
  
  ```dotnetcli
  dotnet new blazorwasm -o MyBlazorPwa --pwa -f netcoreapp3.1
  ```
  
  Nel comando precedente, l' `-o|--output` opzione Crea una nuova cartella per l'app denominata `MyBlazorPwa` .

* Passare al repository GitHub ASP.NET Core all'URL seguente, che si collega all'origine e agli asset di riferimento della versione 3,1:

  [cartella del modello di progetto DotNet/aspnetcore (versione 3,1) Blazor WebAssembly `wwwroot`](https://github.com/dotnet/aspnetcore/tree/release/3.1/src/ProjectTemplates/ComponentsWebAssembly.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/wwwroot)

  > [!NOTE]
  > L'URL per il modello di progetto è stato Blazor WebAssembly modificato dopo il rilascio di ASP.NET Core 3,1. Gli asset di riferimento per 5,0 o versioni successive sono disponibili all'URL seguente:
  >
  > [cartella del modello di progetto DotNet/aspnetcore (versione 5,0) Blazor WebAssembly `wwwroot`](https://github.com/dotnet/aspnetcore/tree/release/5.0/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/wwwroot)

::: moniker-end

Dalla cartella di origine `wwwroot` nell'app creata dall'utente o dagli asset di riferimento nel `dotnet/aspnetcore` repository GitHub, copiare i file seguenti nella cartella dell'app `wwwroot` :

* `icon-512.png`
* `manifest.json`
* `service-worker.js`
* `service-worker.published.js`

Nel file dell'app `wwwroot/index.html` :

* Aggiungere `<link>` elementi per il manifesto e l'icona dell'app:

  ```html
  <link href="manifest.json" rel="manifest" />
  <link rel="apple-touch-icon" sizes="512x512" href="icon-512.png" />
  ```

* Aggiungere il `<script>` tag seguente all'interno del `</body>` tag di chiusura subito dopo il `blazor.webassembly.js` tag di script:

  ```html
      ...
      <script>navigator.serviceWorker.register('service-worker.js');</script>
  </body>
  ```

## <a name="installation-and-app-manifest"></a>Installazione e manifesto dell'applicazione

Quando si visita un'app creata usando il modello di PWA, gli utenti hanno la possibilità di installare l'app nel menu Start del sistema operativo, nell'ancoraggio o nella schermata iniziale. Il modo in cui viene presentata questa opzione dipende dal browser dell'utente. Quando si usano i browser basati su Chrome Desktop, ad esempio Edge o Chrome, viene visualizzato un pulsante **Aggiungi** nella barra degli URL. Dopo che l'utente ha selezionato il pulsante **Aggiungi** , viene visualizzata una finestra di dialogo di conferma:

![La finestra di dialogo di conferma in Google Chrome visualizza un pulsante di installazione all'utente per l'app ' My::: NO-LOC (Blazer):::P WA '.](progressive-web-app/_static/image2.png)

In iOS i visitatori possono installare il PWA usando il pulsante di **condivisione** di Safari e l'opzione **Aggiungi a homescreen** . In Chrome per Android, gli utenti devono selezionare il pulsante di **menu** nell'angolo in alto a destra, seguito da **Aggiungi alla schermata iniziale**.

Una volta installato, l'app viene visualizzata nella propria finestra senza una barra degli indirizzi:

![L'app ' My::: NO-LOC (Blazer):::P WA ' viene eseguita in Google Chrome senza una barra degli indirizzi.](progressive-web-app/_static/image3.png)

Per personalizzare il titolo, la combinazione di colori, l'icona o altri dettagli della finestra, vedere il `manifest.json` file nella directory del progetto `wwwroot` . Lo schema di questo file è definito dagli standard Web. Per altre informazioni, vedere la pagina relativa ai [documenti MDN Web: manifesto dell'app Web](https://developer.mozilla.org/docs/Web/Manifest).

## <a name="offline-support"></a>Supporto offline

Per impostazione predefinita, le app create usando l'opzione del modello di PWA hanno il supporto per l'esecuzione offline. Un utente deve prima visitare l'app mentre è in linea. Il browser Scarica e memorizza automaticamente tutte le risorse necessarie per il funzionamento offline.

> [!IMPORTANT]
> Il supporto per lo sviluppo interferisce con il normale ciclo di sviluppo di apportare modifiche e testarle. Il supporto offline è pertanto abilitato solo per le app *pubblicate* . 

> [!WARNING]
> Se si intende distribuire un PWA abilitato offline, sono presenti [diversi avvisi e Avvertenze importanti](#caveats-for-offline-pwas). Questi scenari sono inerenti a PWA offline e non sono specifici di Blazor . Assicurarsi di leggere e comprendere questi avvertimenti prima di ipotizzare il funzionamento dell'app abilitata per offline.

Per verificare il funzionamento del supporto offline:

1. Pubblicare l'app. Per altre informazioni, vedere <xref:blazor/host-and-deploy/index#publish-the-app>.
1. Distribuire l'app in un server che supporta HTTPS e accedere all'app in un browser con il relativo indirizzo HTTPS protetto.
1. Aprire gli strumenti di sviluppo del browser e verificare che un ruolo di *lavoro del servizio* sia registrato per l'host nella scheda **applicazione** :

   ![La scheda ' applicazione ' degli strumenti di sviluppo di Google Chrome Mostra un servizio di lavoro attivato e in esecuzione.](progressive-web-app/_static/image4.png)

1. Ricaricare la pagina ed esaminare la scheda **rete** . il ruolo di **lavoro del servizio** o la **cache della memoria** sono elencati come origini per tutti gli asset della pagina:

   ![Scheda ' rete ' degli strumenti di sviluppo di Google Chrome che mostra le origini per tutte le risorse della pagina.](progressive-web-app/_static/image5.png)

1. Per verificare che il browser non dipenda dall'accesso di rete per caricare l'app, eseguire una delle operazioni seguenti:

   * Arrestare il server Web e verificare il funzionamento normale dell'app, che include il ricaricamento della pagina. Analogamente, l'applicazione continua a funzionare normalmente quando è presente una connessione di rete lenta.
   * Indica al browser di simulare la modalità offline nella scheda **rete** :

   ![Scheda "rete" degli strumenti di sviluppo di Google Chrome con l'elenco a discesa Modalità browser modificato da "online" a "offline".](progressive-web-app/_static/image6.png)

Il supporto offline tramite un worker del servizio è uno standard Web, non specifico di Blazor . Per altre informazioni sui ruoli di lavoro del servizio, vedere la pagina relativa all' [API del servizio Web MDN](https://developer.mozilla.org/docs/Web/API/Service_Worker_API). Per ulteriori informazioni sui modelli di utilizzo comuni per i ruoli di lavoro del servizio, vedere [Google Web: ciclo di vita del servizio](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle).

Blazoril modello di PWA produce due file del ruolo di lavoro del servizio:

* `wwwroot/service-worker.js`, che viene usato durante lo sviluppo.
* `wwwroot/service-worker.published.js`, che viene usato dopo la pubblicazione dell'app.

Per condividere la logica tra i due file del ruolo di lavoro del servizio, prendere in considerazione l'approccio seguente:

* Aggiungere un terzo file JavaScript per conservare la logica comune.
* Usare [`self.importScripts`](https://developer.mozilla.org/docs/Web/API/WorkerGlobalScope/importScripts) per caricare la logica comune in entrambi i file del ruolo di lavoro del servizio.

### <a name="cache-first-fetch-strategy"></a>Strategia di recupero cache-First

Il ruolo di `service-worker.published.js` lavoro del servizio incorporato risolve le richieste utilizzando una strategia di *cache-First* . Questo significa che il ruolo di lavoro del servizio preferisce restituire contenuto memorizzato nella cache, indipendentemente dal fatto che l'utente abbia accesso alla rete o che nel server sia disponibile contenuto più recente.

La strategia di cache-First è utile perché:

* **Garantisce l'affidabilità.** L'accesso alla rete non è uno stato booleano. Un utente non è semplicemente online o offline:

  * Il dispositivo dell'utente può presupporre che sia online, ma la rete potrebbe essere così lenta da risultare poco pratica da attendere.
  * La rete potrebbe restituire risultati non validi per determinati URL, ad esempio quando è presente un portale WIFI captive che sta attualmente bloccando o reindirizzando determinate richieste.
  
  Questo è il motivo per cui l'API del browser `navigator.onLine` non è affidabile e non deve dipendere da.

* **Garantisce la correttezza.** Quando si compila una cache di risorse offline, il ruolo di lavoro del servizio utilizza l'hashing del contenuto per garantire che sia stato recuperato uno snapshot completo e coerente delle risorse in un singolo momento. Questa cache viene quindi utilizzata come unità atomica. Non è necessario chiedere alla rete le risorse più recenti, perché le uniche versioni richieste sono quelle già memorizzate nella cache. Altri rischiano incoerenza e incompatibilità (ad esempio, il tentativo di usare le versioni degli assembly .NET che non sono stati compilati insieme).

### <a name="background-updates"></a>Aggiornamenti in background

Come modello mentale, è possibile pensare a un PWA offline-First come si comporta come un'app per dispositivi mobili che può essere installata. L'app viene avviata immediatamente indipendentemente dalla connettività di rete, ma la logica dell'app installata deriva da uno snapshot temporizzato che potrebbe non essere la versione più recente.

Il Blazor modello di PWA genera app che tentano automaticamente di aggiornarsi in background ogni volta che l'utente visita e ha una connessione di rete funzionante. Il funzionamento di questo approccio è il seguente:

* Durante la compilazione, il progetto genera un manifesto delle risorse del ruolo di *lavoro del servizio*. Per impostazione predefinita, viene chiamato il metodo `service-worker-assets.js` . Il manifesto elenca tutte le risorse statiche richieste dall'app per funzionare offline, ad esempio assembly .NET, file JavaScript e CSS, inclusi gli hash del contenuto. L'elenco di risorse viene caricato dal ruolo di lavoro del servizio in modo da sapere quali risorse memorizzare nella cache.
* Ogni volta che l'utente visita l'app, il browser richiede nuovamente `service-worker.js` e `service-worker-assets.js` in background. I file vengono confrontati byte per byte con il ruolo di lavoro del servizio installato esistente. Se il server restituisce contenuto modificato per uno di questi file, il ruolo di lavoro del servizio tenta di installare una nuova versione di se stessa.
* Quando si installa una nuova versione di se stessa, il ruolo di lavoro del servizio crea una nuova cache separata per le risorse offline e inizia a popolare la cache con le risorse elencate in `service-worker-assets.js` . Questa logica viene implementata nella `onInstall` funzione all'interno di `service-worker.published.js` .
* Il processo viene completato correttamente quando tutte le risorse vengono caricate senza errori e tutti gli hash di contenuto corrispondono. In caso di esito positivo, il nuovo ruolo di lavoro del servizio entra *in attesa dello stato di attivazione* . Non appena l'utente chiude l'app (nessuna scheda o Windows dell'app rimanente), il nuovo Worker del servizio diventa *attivo* e viene usato per le visite successive all'app. Il servizio di lavoro precedente e la relativa cache vengono eliminati.
* Se il processo non viene completato correttamente, la nuova istanza del ruolo di lavoro del servizio viene eliminata. Il processo di aggiornamento viene nuovamente ritentato alla visita successiva dell'utente, quando si spera che il client disponga di una connessione di rete migliore in grado di completare le richieste.

Personalizzare questo processo modificando la logica del ruolo di lavoro del servizio. Nessuno dei comportamenti precedenti è specifico di, Blazor ma è semplicemente l'esperienza predefinita fornita dall'opzione del modello PWA. Per ulteriori informazioni, vedere la pagina relativa all' [API Web di MDN: Service Worker](https://developer.mozilla.org/docs/Web/API/Service_Worker_API).

### <a name="how-requests-are-resolved"></a>Modalità di risoluzione delle richieste

Come descritto nella sezione relativa alla [strategia di recupero cache-First](#cache-first-fetch-strategy) , il ruolo di lavoro predefinito del servizio utilizza una strategia di *cache-First* , ovvero tenta di gestire il contenuto memorizzato nella cache, se disponibile. Se non sono presenti contenuti memorizzati nella cache per un determinato URL, ad esempio quando si richiedono dati da un'API back-end, il ruolo di lavoro del servizio esegue il fallback su una normale richiesta di rete. La richiesta di rete ha esito positivo se il server è raggiungibile. Questa logica viene implementata all'interno di una `onFetch` funzione all'interno di `service-worker.published.js` .

Se i componenti dell'app Razor si basano sulla richiesta di dati dalle API back-end e si vuole fornire un'esperienza utente intuitiva per le richieste non riuscite a causa della mancata disponibilità della rete, implementare la logica all'interno dei componenti dell'app. Usare, ad esempio `try/catch` , <xref:System.Net.Http.HttpClient> le richieste di circa.

### <a name="support-server-rendered-pages"></a>Pagine di supporto per il rendering del server

Si consideri cosa accade quando l'utente accede per la prima volta a un URL, ad esempio `/counter` o qualsiasi altro collegamento profondo nell'app. In questi casi, non si vuole restituire contenuto memorizzato nella cache come `/counter` , ma è necessario il browser per caricare il contenuto memorizzato nella cache come `/index.html` per avviare l' Blazor WebAssembly app. Queste richieste iniziali sono note come richieste di *navigazione* , anziché:

* `subresource` richieste per immagini, fogli di stile o altri file.
* `fetch/XHR` richieste di dati dell'API.

Il ruolo di lavoro del servizio predefinito contiene la logica speciale per le richieste di navigazione. Il ruolo di lavoro del servizio risolve le richieste restituendo il contenuto memorizzato nella cache per `/index.html` , indipendentemente dall'URL richiesto. Questa logica viene implementata nella `onFetch` funzione all'interno di `service-worker.published.js` .

Se l'app dispone di determinati URL che devono restituire codice HTML sottoposto a rendering del server e non `/index.html` vengono usati dalla cache, è necessario modificare la logica nel ruolo di lavoro del servizio. Se tutti gli URL che contengono `/Identity/` devono essere gestiti come richieste normali solo in linea al server, modificare la `service-worker.published.js` `onFetch` logica. Individuare il codice seguente:

```javascript
const shouldServeIndexHtml = event.request.mode === 'navigate';
```

Modificare il codice nel modo seguente:

```javascript
const shouldServeIndexHtml = event.request.mode === 'navigate'
  && !event.request.url.includes('/Identity/');
```

Se non si esegue questa operazione, indipendentemente dalla connettività di rete, il ruolo di lavoro del servizio intercetta le richieste per tali URL e le risolve utilizzando `/index.html` .

Aggiungere al controllo altri endpoint per i provider di autenticazione esterni. Nell'esempio seguente, `/signin-google` per l'autenticazione Google viene aggiunto al controllo:

```javascript
const shouldServeIndexHtml = event.request.mode === 'navigate'
  && !event.request.url.includes('/Identity/')
  && !event.request.url.includes('/signin-google');
```

Non è necessaria alcuna azione per l'ambiente di sviluppo, in cui il contenuto viene sempre recuperato dalla rete.

### <a name="control-asset-caching"></a>Controllare la memorizzazione nella cache degli asset

Se il progetto definisce la `ServiceWorkerAssetsManifest` Proprietà MSBuild, Blazor gli strumenti di compilazione di generano un manifesto delle risorse del ruolo di lavoro del servizio con il nome specificato. Il modello predefinito di PWA produce un file di progetto contenente la proprietà seguente:

```xml
<ServiceWorkerAssetsManifest>service-worker-assets.js</ServiceWorkerAssetsManifest>
```

Il file viene inserito nella `wwwroot` directory di output, quindi il browser può recuperare il file richiedendo `/service-worker-assets.js` . Per visualizzare il contenuto di questo file, aprire `/bin/Debug/{TARGET FRAMEWORK}/wwwroot/service-worker-assets.js` in un editor di testo. Tuttavia, non modificare il file perché viene rigenerato in ogni compilazione.

Per impostazione predefinita, questo manifesto elenca:

* Tutte le Blazor risorse gestite, ad esempio gli assembly .NET e i file di runtime di .NET webassembly, necessari per il funzionamento offline.
* Tutte le risorse per la pubblicazione nella directory dell'app `wwwroot` , ad esempio immagini, fogli di stile e file JavaScript, inclusi gli asset Web statici forniti da progetti esterni e pacchetti NuGet.

È possibile controllare quali di queste risorse vengono recuperate e memorizzate nella cache dal ruolo di lavoro del servizio modificando la logica in `onInstall` in `service-worker.published.js` . Per impostazione predefinita, il ruolo di lavoro del servizio recupera e memorizza nella cache i file corrispondenti a estensioni di file Web tipiche, ad esempio `.html` ,, `.css` `.js` e `.wasm` , oltre ai tipi di file specifici di Blazor WebAssembly ( `.dll` , `.pdb` ).

Per includere risorse aggiuntive che non sono presenti nella directory dell'app `wwwroot` , definire voci MSBuild aggiuntive `ItemGroup` , come illustrato nell'esempio seguente:

```xml
<ItemGroup>
  <ServiceWorkerAssetsManifestItem Include="MyDirectory\AnotherFile.json"
    RelativePath="MyDirectory\AnotherFile.json" AssetUrl="files/AnotherFile.json" />
</ItemGroup>
```

I `AssetUrl` metadati specificano l'URL relativo di base che il browser deve usare durante il recupero della risorsa nella cache. Questo può essere indipendente dal nome del file di origine originale sul disco.

> [!IMPORTANT]
> L'aggiunta di un `ServiceWorkerAssetsManifestItem` non comporta la pubblicazione del file nella directory dell'app `wwwroot` . L'output di pubblicazione deve essere controllato separatamente. Il `ServiceWorkerAssetsManifestItem` solo determina la visualizzazione di una voce aggiuntiva nel manifesto delle risorse del ruolo di lavoro del servizio.

## <a name="push-notifications"></a>Notifiche push

Analogamente a qualsiasi altro PWA, un Blazor WebAssembly PWA può ricevere notifiche push da un server back-end. Il server può inviare notifiche push in qualsiasi momento, anche quando l'utente non usa attivamente l'app. Ad esempio, è possibile inviare notifiche push quando un altro utente esegue un'azione pertinente.

Il meccanismo per l'invio di una notifica push è interamente indipendente da Blazor WebAssembly , poiché è implementato dal server back-end che può utilizzare qualsiasi tecnologia. Se si desidera inviare notifiche push da un server di ASP.NET Core, è consigliabile [utilizzare una tecnica simile all'approccio adottato nel workshop sulla pizza ardente](https://github.com/dotnet-presentations/blazor-workshop/blob/master/docs/09-progressive-web-app.md#sending-push-notifications).

Il meccanismo per la ricezione e la visualizzazione di una notifica push sul client è anche indipendente da Blazor WebAssembly , dal momento che è implementato nel file JavaScript del servizio Worker. Per un esempio, vedere [l'approccio usato in the Blazing pizza workshop](https://github.com/dotnet-presentations/blazor-workshop/blob/master/docs/09-progressive-web-app.md#displaying-notifications).

## <a name="caveats-for-offline-pwas"></a>Avvertenze per PWA offline

Non tutte le app devono provare a supportare l'uso offline. Il supporto offline aggiunge una complessità significativa, sebbene non sempre pertinente per i casi d'uso richiesti.

Il supporto offline è in genere rilevante solo:

* Se l'archivio dati primario è locale nel browser. Ad esempio, l'approccio è pertinente in un'app con un'interfaccia utente per [un dispositivo Internet](https://en.wikipedia.org/wiki/Internet_of_things) delle cose che archivia i dati in `localStorage` o [IndexedDB](https://developer.mozilla.org/docs/Web/API/IndexedDB_API).
* Se l'app esegue una quantità significativa di lavoro per recuperare e memorizzare nella cache i dati dell'API back-end rilevanti per ogni utente, in modo che possano spostarsi tra i dati offline. Se l'app deve supportare la modifica, è necessario compilare un sistema per tenere traccia delle modifiche e sincronizzare i dati con il back-end.
* Se l'obiettivo è garantire che l'app venga caricata immediatamente indipendentemente dalle condizioni della rete. Implementare un'esperienza utente adatta alle richieste dell'API back-end per mostrare lo stato di avanzamento delle richieste e comportarsi correttamente quando le richieste hanno esito negativo a causa della mancata disponibilità della rete

Inoltre, le PWA con supporto offline devono gestire una serie di complicazioni aggiuntive. Gli sviluppatori devono acquisire familiarità con attenzione con le avvertenze riportate nelle sezioni riportate di seguito.

### <a name="offline-support-only-when-published"></a>Supporto offline solo dopo la pubblicazione

Durante lo sviluppo, in genere si desidera visualizzare tutte le modifiche riflesse immediatamente nel browser senza passare attraverso un processo di aggiornamento in background. Di conseguenza, Blazor il modello di PWA consente il supporto offline solo dopo la pubblicazione.

Quando si compila un'app che supporta offline, non è sufficiente testare l'app nell'ambiente di sviluppo. È necessario testare l'app nello stato pubblicato per comprendere come risponde a condizioni di rete diverse.

### <a name="update-completion-after-user-navigation-away-from-app"></a>Completamento dell'aggiornamento dopo la navigazione dell'utente dall'app

Gli aggiornamenti non vengono completati fino a quando l'utente non si è spostato dall'app in tutte le schede. Come spiegato nella sezione [aggiornamenti in background](#background-updates) , dopo la distribuzione di un aggiornamento nell'app, il browser recupera i file di lavoro del servizio aggiornati per avviare il processo di aggiornamento.

Quali sono le sorprese che molti sviluppatori, anche al termine dell'aggiornamento, diventano effettive fino a quando l'utente **non** si è spostato in tutte le schede. **Non** è sufficiente aggiornare la scheda visualizzando l'app, anche se si tratta dell'unica scheda che Visualizza l'app. Finché l'app non viene chiusa completamente, il nuovo ruolo di lavoro del servizio rimane in *attesa di attivazione* . **Questo non è specifico di Blazor , ma è piuttosto un comportamento standard della piattaforma Web.**

Questo problema si verifica in genere dagli sviluppatori che tentano di testare gli aggiornamenti al ruolo di lavoro del servizio o alle risorse nella cache offline. Se si archiviano gli strumenti di sviluppo del browser, è possibile che venga visualizzato un risultato simile al seguente:

![La scheda "applicazione" di Google Chrome indica che il ruolo di lavoro del servizio dell'app è in attesa di attivazione.](progressive-web-app/_static/image7.png)

Per tutto il tempo in cui l'elenco di "client", che sono schede o finestre che visualizzano l'app, non è vuoto, il lavoro continua ad attendere. Il motivo per cui i dipendenti del servizio eseguono questa operazione è garantire la coerenza. La coerenza significa che tutte le risorse vengono recuperate dalla stessa cache atomica.

Quando si verificano le modifiche, può essere utile fare clic sul collegamento "skipWaiting", come illustrato nella schermata precedente, quindi ricaricare la pagina. È possibile automatizzare questa operazione per tutti gli utenti codificando il ruolo di lavoro del servizio in modo da [ignorare la fase di attesa e attivare immediatamente l'aggiornamento](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle#skip_the_waiting_phase). Se si ignora la fase di attesa, si cede la garanzia che le risorse vengano sempre recuperate in modo coerente dalla stessa istanza della cache.

### <a name="users-may-run-any-historical-version-of-the-app"></a>Gli utenti possono eseguire qualsiasi versione cronologica dell'app

Gli sviluppatori Web si aspettano abitualmente che gli utenti eseguano solo la versione più recente distribuita dell'app Web, dal momento che è normale nel modello di distribuzione Web tradizionale. Tuttavia, un PWA per la prima volta offline è più simile a un'app per dispositivi mobili nativa, in cui gli utenti non eseguono necessariamente la versione più recente.

Come spiegato nella sezione [aggiornamenti in background](#background-updates) , dopo la distribuzione di un aggiornamento nell'app, **ogni utente esistente continua a usare una versione precedente per almeno un'altra visita** perché l'aggiornamento viene eseguito in background e non viene attivato fino a quando l'utente non si sposta in seguito. Inoltre, la versione precedente utilizzata non è necessariamente quella precedente distribuita. La versione precedente può essere *qualsiasi* versione cronologica, a seconda di quando l'utente ha completato un aggiornamento.

Questo può essere un problema se le parti front-end e back-end dell'app richiedono un accordo sullo schema per le richieste API. Non è necessario distribuire le modifiche dello schema API incompatibili con le versioni precedenti fino a quando non si è certi che tutti gli utenti abbiano eseguito l'aggiornamento. In alternativa, impedire agli utenti di usare versioni precedenti dell'app incompatibili. Questo requisito dello scenario è identico a quello delle app per dispositivi mobili native. Se si distribuisce una modifica di rilievo nelle API del server, l'app client viene interrotta per gli utenti che non hanno ancora eseguito l'aggiornamento.

Se possibile, non distribuire le modifiche di rilievo alle API back-end. Se necessario, provare a usare le [API standard del ruolo di lavoro del servizio, ad esempio ServiceWorkerRegistration](https://developer.mozilla.org/docs/Web/API/ServiceWorkerRegistration) , per determinare se l'app è aggiornata e, in caso contrario, per impedire l'utilizzo.

### <a name="interference-with-server-rendered-pages"></a>Interferenze con le pagine sottoposte a rendering server

Come descritto nella sezione [pagine del server di supporto](#support-server-rendered-pages) di cui è stato eseguito il rendering, se si desidera ignorare il comportamento del ruolo di lavoro del servizio per la restituzione di `/index.html` contenuto per tutte le richieste di navigazione, modificare la logica nel ruolo di lavoro del servizio.

### <a name="all-service-worker-asset-manifest-contents-are-cached-by-default"></a>Tutti i contenuti del manifesto dell'asset del servizio di lavoro vengono memorizzati nella cache

Come descritto nella sezione [controllare la memorizzazione nella cache degli asset](#control-asset-caching) , il file `service-worker-assets.js` viene generato durante la compilazione ed elenca tutti gli asset che devono essere recuperati e memorizzati nella cache dal ruolo di lavoro del servizio.

Poiché per impostazione predefinita questo elenco include tutto ciò che `wwwroot` è stato emesso, incluso il contenuto fornito da pacchetti e progetti esterni, è necessario prestare attenzione a non inserire troppi contenuti. Se la `wwwroot` directory contiene milioni di immagini, il ruolo di lavoro del servizio tenterà di recuperarli e memorizzarli nella cache, consumando una larghezza di banda eccessiva e probabilmente non viene completata correttamente.

Implementare una logica arbitraria per controllare quale subset del contenuto del manifesto deve essere recuperato e memorizzato nella cache modificando la `onInstall` funzione in `service-worker.published.js` .

### <a name="interaction-with-authentication"></a>Interazione con l'autenticazione

Il modello PWA può essere usato in combinazione con l'autenticazione di. Un PWA con supporto offline può anche supportare l'autenticazione quando l'utente ha la connettività di rete iniziale.

Quando un utente non dispone di connettività di rete, non è in grado di autenticare o ottenere i token di accesso. Per impostazione predefinita, il tentativo di visitare la pagina di accesso senza accesso alla rete genera un messaggio di errore di rete. È necessario progettare un flusso dell'interfaccia utente che consenta all'utente di eseguire attività utili in modalità offline senza tentare di autenticare l'utente o ottenere i token di accesso. In alternativa, è possibile progettare l'applicazione in modo che non riesca correttamente quando la rete non è disponibile. Se l'app non può essere progettata per gestire questi scenari, è possibile che non si desideri abilitare il supporto offline.

Quando un'app progettata per l'uso online e offline è di nuovo online:

* L'app potrebbe dover effettuare il provisioning di un nuovo token di accesso.
* L'app deve rilevare se un altro utente è connesso al servizio, in modo da poter applicare le operazioni all'account dell'utente che sono state effettuate mentre erano offline.

Per creare un'app PWA offline che interagisce con l'autenticazione:

* Sostituire <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccountClaimsPrincipalFactory%601> con una factory che archivia l'ultimo utente connesso e usa l'utente archiviato quando l'app è offline.
* Accodare le operazioni mentre l'app è offline e applicarle quando l'app torna online.
* Durante la disconnessione, cancellare l'utente archiviato.

L' [`CarChecker`](https://github.com/SteveSandersonMS/CarChecker) app di esempio illustra gli approcci precedenti. Vedere le parti seguenti dell'app:

* `OfflineAccountClaimsPrincipalFactory` (`Client/Data/OfflineAccountClaimsPrincipalFactory.cs`)
* `LocalVehiclesStore` (`Client/Data/LocalVehiclesStore.cs`)
* `LoginStatus` componente ( `Client/Shared/LoginStatus.razor` )

## <a name="additional-resources"></a>Risorse aggiuntive

* [Risolvere i problemi dello script di PowerShell di integrità](xref:blazor/host-and-deploy/webassembly#troubleshoot-integrity-powershell-script)
* [SignalR negoziazione tra le origini per l'autenticazione](xref:blazor/fundamentals/signalr#signalr-cross-origin-negotiation-for-authentication)
