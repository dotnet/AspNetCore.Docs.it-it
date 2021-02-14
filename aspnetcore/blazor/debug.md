---
title: ASP.NET Core di debug Blazor WebAssembly
author: guardrex
description: Informazioni su come eseguire il debug delle Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2020
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
uid: blazor/debug
ms.openlocfilehash: aed9e800074dc72a2b55b816a2710722470cd7d1
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280052"
---
# <a name="debug-aspnet-core-blazor-webassembly"></a>ASP.NET Core di debug Blazor WebAssembly

Blazor WebAssembly è possibile eseguire il debug delle app usando gli strumenti di sviluppo del browser nei browser basati su cromo (Edge/Chrome). È anche possibile eseguire il debug dell'app usando gli ambienti di sviluppo integrato (IDE) seguenti:

* Visual Studio
* Visual Studio per Mac
* Visual Studio Code

Gli scenari disponibili includono:

* Impostare e rimuovere punti di interruzione.
* Eseguire l'app con supporto per il debug in IDE.
* Singolo passaggio del codice.
* Riprendere l'esecuzione del codice con un tasto di scelta rapida negli IDE.
* Nella finestra variabili *locali* osservare i valori delle variabili locali.
* Vedere lo stack di chiamate, incluse le catene di chiamate tra JavaScript e .NET.

Per il momento *non è possibile*:

* Interrompi in corrispondenza di eccezioni non gestite.
* Premere i punti di interruzione durante l'avvio dell'app prima che il proxy di debug sia in esecuzione. Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.
* Eseguire il debug in scenari non locali (ad esempio, il [sottosistema Windows per Linux (WSL)](/windows/wsl/) o gli spazi dei nomi di [Visual Studio](/visualstudio/codespaces/overview/what-is-vsonline)).
* Ricompilare automaticamente l'app back-end `*Server*` di una soluzione ospitata Blazor durante il debug, ad esempio eseguendo l'app con [`dotnet watch run`](xref:tutorials/dotnet-watch) .

## <a name="prerequisites"></a>Prerequisiti

Il debug richiede uno dei seguenti browser:

* Google Chrome (versione 70 o successiva) (impostazione predefinita)
* Microsoft Edge (versione 80 o successiva)

Assicurarsi che i firewall o i proxy non blocchino le comunicazioni con il proxy di debug ( `NodeJS` processo). Per ulteriori informazioni, vedere la sezione relativa alla [configurazione del firewall](#firewall-configuration) .

Visual Studio per Mac richiede la versione 8,8 (Build 1532) o versioni successive:

1. Installare la versione più recente di Visual Studio per Mac selezionando il pulsante **scarica Visual Studio per Mac** in [Microsoft: Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/).
1. Selezionare il canale di *Anteprima* dall'interno di Visual Studio. Per ulteriori informazioni, vedere [installare una versione di anteprima di Visual Studio per Mac](/visualstudio/mac/install-preview).

> [!NOTE]
> Apple Safari in macOS attualmente non è supportato.

## <a name="enable-debugging"></a>Abilitare il debug

Per abilitare il debug per un' Blazor WebAssembly app esistente, aggiornare il `launchSettings.json` file nel progetto di avvio per includere la `inspectUri` proprietà seguente in ogni profilo di avvio:

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
```

Al termine dell'aggiornamento, il `launchSettings.json` file dovrebbe essere simile all'esempio seguente:

[!code-json[](debug/launchSettings.json?highlight=14,22)]

`inspectUri`Proprietà:

* Consente all'IDE di rilevare che l'app è un' Blazor WebAssembly app.
* Indica all'infrastruttura di debug degli script di connettersi al browser tramite il Blazor proxy di debug.

I valori segnaposto per i protocolli WebSockets ( `wsProtocol` ), host ( `url.hostname` ), Port ( `url.port` ) e Inspector URI nel browser avviato ( `browserInspectUri` ) sono forniti dal Framework.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Per eseguire il debug di un' Blazor WebAssembly app in Visual Studio:

1. Creare una nuova app ospitata ASP.NET Core Blazor WebAssembly .
1. Premere <kbd>F5</kbd> per eseguire l'app nel debugger.

   > [!NOTE]
   > **Avvia senza eseguire debug** (<kbd>CTRL</kbd> + <kbd>F5</kbd>) non è supportato. Quando l'app viene eseguita nella configurazione di debug, il sovraccarico del debug comporta sempre una riduzione delle prestazioni.

1. Nell' `*Client*` app, impostare un punto di interruzione nella `currentCount++;` riga in `Pages/Counter.razor` .
1. Nel browser passare alla `Counter` pagina e selezionare il pulsante **fare clic su me** per raggiungere il punto di interruzione.
1. In Visual Studio, controllare il valore del `currentCount` campo nella finestra **variabili locali** .
1. Premere <kbd>F5</kbd> per continuare l'esecuzione.

Durante il debug di un' Blazor WebAssembly app, è anche possibile eseguire il debug del codice del server:

1. Impostare un punto di interruzione nella `Pages/FetchData.razor` pagina in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .
1. Impostare un punto di interruzione in `WeatherForecastController` nel `Get` metodo di azione.
1. Passare alla `Fetch Data` pagina per raggiungere il primo punto di interruzione nel `FetchData` componente immediatamente prima di eseguire una richiesta HTTP al server.
1. Premere <kbd>F5</kbd> per continuare l'esecuzione e quindi raggiungere il punto di interruzione sul server nel `WeatherForecastController` .
1. Premere di nuovo <kbd>F5</kbd> per consentire l'esecuzione continua e visualizzare la tabella delle previsioni meteorologiche sottoposta a rendering nel browser.

> [!NOTE]
> I punti di interruzione **non** vengono raggiunti durante l'avvio dell'app prima che il proxy di debug sia in esecuzione. Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.

Se l'app è ospitata in un [percorso di base dell'app](xref:blazor/host-and-deploy/index#app-base-path) diverso da `/` , aggiornare le proprietà seguenti in `Properties/launchSettings.json` per riflettere il percorso di base dell'app:

* `applicationUrl`:

  ```json
  "iisSettings": {
    ...
    "iisExpress": {
      "applicationUrl": "http://localhost:{INSECURE PORT}/{APP BASE PATH}/",
      "sslPort": {SECURE PORT}
    }
  },
  ```

* `inspectUri` di ogni profilo:

  ```json
  "profiles": {
    ...
    "{PROFILE 1, 2, ... N}": {
      ...
      "inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/{APP BASE PATH}/_framework/debug/ws-proxy?browser={browserInspectUri}",
      ...
    }
  }
  ```

I segnaposto nelle impostazioni precedenti:

* `{INSECURE PORT}`: Porta non protetta. Per impostazione predefinita viene fornito un valore casuale, ma è consentita una porta personalizzata.
* `{APP BASE PATH}`: Percorso di base dell'app.
* `{SECURE PORT}`: Porta protetta. Per impostazione predefinita viene fornito un valore casuale, ma è consentita una porta personalizzata.
* `{PROFILE 1, 2, ... N}`: Avvia profili impostazioni. In genere, un'app specifica più di un profilo per impostazione predefinita (ad esempio, un profilo per IIS Express e un profilo di progetto, che viene usato dal server gheppio).

Negli esempi seguenti l'app è ospitata in `/OAT` con un percorso di base dell'app configurato in `wwwroot/index.html` come `<base href="/OAT/">` :

```json
"applicationUrl": "http://localhost:{INSECURE PORT}/OAT/",
```

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/OAT/_framework/debug/ws-proxy?browser={browserInspectUri}",
```

Per informazioni sull'uso di un percorso di base dell'app personalizzato per le Blazor WebAssembly app, vedere <xref:blazor/host-and-deploy/index#app-base-path> .

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<h2 id="vscode">Debug autonomo Blazor WebAssembly</h2>

1. Aprire l'app autonoma Blazor WebAssembly in vs code.

   È possibile che venga ricevuta una notifica che è necessaria un'installazione aggiuntiva per abilitare il debug:

   > Per eseguire il debug delle applicazioni, è necessaria un'installazione aggiuntiva Blazor WebAssembly .

   Se si riceve la notifica:

   * Verificare che sia installata la versione più recente [di C# per Visual Studio Code estensione](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) . Per esaminare le estensioni installate, aprire **View**  >  **Extensions** dalla barra dei menu o selezionare l'icona **Extensions** nella barra laterale **Activity** .
   * Verificare che l'anteprima JavaScript sia abilitata. Aprire le impostazioni dalla barra dei menu (  >    >  **Impostazioni** preferenze file). Eseguire la ricerca usando le parole chiave `debug preview` . Nei risultati della ricerca verificare che la casella di controllo per **Debug > JavaScript: USA anteprima** sia selezionata. Se l'opzione per abilitare il debug in anteprima non è presente, eseguire l'aggiornamento alla versione più recente di VS Code o installare l' [estensione del debugger JavaScript](https://marketplace.visualstudio.com/items?itemName=ms-vscode.js-debug-nightly) (VS Code versioni 1,46 o precedenti).
   * Ricaricare la finestra.

1. Avviare il debug usando il tasto di scelta rapida <kbd>F5</kbd> o la voce di menu.

   > [!NOTE]
   > **Avvia senza eseguire debug** (<kbd>CTRL</kbd> + <kbd>F5</kbd>) non è supportato. Quando l'app viene eseguita nella configurazione di debug, il sovraccarico del debug comporta sempre una riduzione delle prestazioni.

1. Quando richiesto, selezionare l'opzione **Blazor WebAssembly debug** per avviare il debug.

1. Viene avviata l'app autonoma e viene aperto un browser di debug.

1. Nell' `*Client*` app, impostare un punto di interruzione nella `currentCount++;` riga in `Pages/Counter.razor` .

1. Nel browser passare alla `Counter` pagina e selezionare il pulsante **fare clic su me** per raggiungere il punto di interruzione.

> [!NOTE]
> I punti di interruzione **non** vengono raggiunti durante l'avvio dell'app prima che il proxy di debug sia in esecuzione. Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.

## <a name="debug-hosted-blazor-webassembly"></a>Debug ospitato Blazor WebAssembly

1. Aprire la cartella della soluzione dell'app ospitata Blazor WebAssembly in vs code.

1. Se non è impostata alcuna configurazione di avvio per il progetto, viene visualizzata la notifica seguente. Selezionare **Sì**.

   > Gli asset necessari per la compilazione e il debug non sono presenti in ' {nome applicazione}'. e se si vuole aggiungerle.

1. Nel riquadro comandi nella parte superiore della finestra selezionare il progetto *Server* all'interno della soluzione ospitata.

`launch.json`Viene generato un file con la configurazione di avvio per l'avvio del debugger.

## <a name="attach-to-an-existing-debugging-session"></a>Connetti a una sessione di debug esistente

Per connettersi a un'app in esecuzione Blazor , creare un `launch.json` file con la configurazione seguente:

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach to Existing Blazor WebAssembly Application"
}
```

> [!NOTE]
> Il fissaggio a una sessione di debug è supportato solo per le app autonome. Per usare il debug dello stack completo, è necessario avviare l'app da VS Code.

## <a name="launch-configuration-options"></a>Opzioni di configurazione di avvio

Le seguenti opzioni di configurazione di avvio sono supportate per il `blazorwasm` tipo di debug ( `.vscode/launch.json` ).

| Opzione    | Descrizione |
| --------- | ----------- |
| `request` | Usare `launch` per avviare e alleghi una sessione di debug a un' Blazor WebAssembly app o `attach` per alleghi una sessione di debug a un'app già in esecuzione. |
| `url`     | URL da aprire nel browser durante il debug. Il valore predefinito è `https://localhost:5001`. |
| `browser` | Browser da avviare per la sessione di debug. Impostare su `edge` o `chrome`. Il valore predefinito è `chrome`. |
| `trace`   | Usato per generare log dal debugger JS. Impostare su `true` per generare i log. |
| `hosted`  | Deve essere impostato su `true` se si avvia e si esegue il debug di un'app ospitata Blazor WebAssembly . |
| `webRoot` | Specifica il percorso assoluto del server Web. Deve essere impostato se un'app viene gestita da una route secondaria. |
| `timeout` | Numero di millisecondi di attesa per il fissaggio della sessione di debug. Il valore predefinito è 30.000 millisecondi (30 secondi). |
| `program` | Riferimento al file eseguibile per eseguire il server dell'app ospitata. Deve essere impostato se `hosted` è `true` . |
| `cwd`     | Directory di lavoro in cui avviare l'app. Deve essere impostato se `hosted` è `true` . |
| `env`     | Variabili di ambiente da fornire al processo avviato. Applicabile solo se `hosted` è impostato su `true` . |

## <a name="example-launch-configurations"></a>Configurazioni di avvio di esempio

### <a name="launch-and-debug-a-standalone-blazor-webassembly-app"></a>Avviare ed eseguire il debug di un'app autonoma Blazor WebAssembly

```json
{
  "type": "blazorwasm",
  "request": "launch",
  "name": "Launch and Debug"
}
```

### <a name="attach-to-a-running-app-at-a-specified-url"></a>Connettersi a un'app in esecuzione in un URL specificato

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach and Debug",
  "url": "http://localhost:5000"
}
```

### <a name="launch-and-debug-a-hosted-blazor-webassembly-app-with-microsoft-edge"></a>Avviare ed eseguire il debug di un'app ospitata Blazor WebAssembly con Microsoft Edge

Per impostazione predefinita, la configurazione del browser è Google Chrome. Quando si usa Microsoft Edge per il debug, impostare `browser` su `edge` . Per usare Google Chrome, non impostare l' `browser` opzione o impostare il valore dell'opzione su `chrome` .

```json
{
  "name": "Launch and Debug Hosted Blazor WebAssembly App",
  "type": "blazorwasm",
  "request": "launch",
  "hosted": true,
  "program": "${workspaceFolder}/Server/bin/Debug/netcoreapp3.1/MyHostedApp.Server.dll",
  "cwd": "${workspaceFolder}/Server",
  "browser": "edge"
}
```

Nell'esempio precedente, `MyHostedApp.Server.dll` è l'assembly dell'app *Server* . La `.vscode` cartella si trova nella cartella della soluzione accanto alle `Client` `Server` cartelle, e `Shared` .

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Per eseguire il debug di un' Blazor WebAssembly app in Visual Studio per Mac:

1. Creare una nuova app ospitata ASP.NET Core Blazor WebAssembly .
1. Premere <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> per eseguire l'applicazione nel debugger.

   > [!NOTE]
   > **Avvia senza eseguire debug** (<kbd>&#8997;</kbd> + <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd>) non è supportato. Quando l'app viene eseguita nella configurazione di debug, il sovraccarico del debug comporta sempre una riduzione delle prestazioni.

   > [!IMPORTANT]
   > Google Chrome o Microsoft Edge deve essere il browser selezionato per la sessione di debug.

1. Nell' `*Client*` app, impostare un punto di interruzione nella `currentCount++;` riga in `Pages/Counter.razor` .
1. Nel browser passare alla `Counter` pagina e selezionare il pulsante **fare clic su me** per raggiungere il punto di interruzione:
1. In Visual Studio, controllare il valore del `currentCount` campo nella finestra **variabili locali** .
1. Premere <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> per continuare l'esecuzione.

Durante il debug di un' Blazor WebAssembly app, è anche possibile eseguire il debug del codice del server:

1. Impostare un punto di interruzione nella `Pages/FetchData.razor` pagina in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .
1. Impostare un punto di interruzione in `WeatherForecastController` nel `Get` metodo di azione.
1. Passare alla `Fetch Data` pagina per raggiungere il primo punto di interruzione nel `FetchData` componente immediatamente prima di eseguire una richiesta HTTP al server.
1. Premere <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> per continuare l'esecuzione e quindi raggiungere il punto di interruzione sul server nel `WeatherForecastController` .
1. Premere <kbd></kbd> + di nuovo&#8984;<kbd>&#8617;</kbd> per consentire l'esecuzione continua e visualizzare la tabella delle previsioni meteorologiche sottoposta a rendering nel browser.

> [!NOTE]
> I punti di interruzione **non** vengono raggiunti durante l'avvio dell'app prima che il proxy di debug sia in esecuzione. Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.

Per ulteriori informazioni, vedere [debug con Visual Studio per Mac](/visualstudio/mac/debugging).

---

## <a name="debug-in-the-browser"></a>Debug nel browser

*Le indicazioni fornite in questa sezione si applicano a Google Chrome e Microsoft Edge in esecuzione su Windows.*

1. Eseguire una build di debug dell'app nell'ambiente di sviluppo.

1. Avviare un browser e passare all'URL dell'app, ad esempio `https://localhost:5001` .

1. Nel browser provare ad avviare il debug remoto premendo <kbd>MAIUSC</kbd> + <kbd>ALT</kbd> + <kbd>d</kbd>.

   Il browser deve essere in esecuzione con il debug remoto abilitato, che non è il valore predefinito. Se il debug remoto è disabilitato, viene eseguito il rendering della pagina di errore della **scheda del browser di cui è possibile eseguire** il debug con le istruzioni per avviare il browser con la porta di debug aperta. Seguire le istruzioni per il browser, che consente di aprire una nuova finestra del browser. Chiudere la finestra del browser precedente.

<!-- HOLD 
1. In the browser, attempt to commence remote debugging by pressing:

   * <kbd>Shift</kbd>+<kbd>Alt</kbd>+<kbd>d</kbd> on Windows.
   * <kbd>Shift</kbd>+<kbd>&#8984;</kbd>+<kbd>d</kbd> on macOS.

   The browser must be running with remote debugging enabled, which isn't the default. If remote debugging is disabled, an **Unable to find debuggable browser tab** error page is rendered with instructions for launching the browser with the debugging port open. Follow the instructions for your browser, which opens a new browser window. Close the previous browser window.
-->

1. Quando il browser è in esecuzione con il debug remoto abilitato, il tasto di scelta rapida di debug nel passaggio precedente apre una nuova scheda del debugger.

1. Dopo qualche istante, la scheda **origini** Mostra un elenco degli assembly .NET dell'app all'interno del `file://` nodo.

1. Nel codice componente ( `.razor` file) e nei file di codice C# ( `.cs` ), i punti di interruzione impostati vengono raggiunti durante l'esecuzione del codice. Quando viene raggiunto un punto di interruzione, l'esecuzione<kbd></kbd>del codice in un singolo passaggio (F10<kbd>) viene</kbd>eseguito normalmente.

Blazor fornisce un proxy di debug che implementa il [protocollo devtools di Chrome](https://chromedevtools.github.io/devtools-protocol/) e potenzia il protocollo con. Informazioni specifiche del NET. Quando si preme il tasto di scelta rapida Blazor per il debug, punta il devtools di Chrome sul proxy. Il proxy si connette alla finestra del browser che si sta tentando di eseguire il debug, quindi è necessario abilitare il debug remoto.

## <a name="browser-source-maps"></a>Mappe di origine del browser

Le mappe di origine del browser consentono al browser di eseguire il mapping dei file compilati ai file di origine originali e vengono comunemente usati per il debug sul lato client. Tuttavia, Blazor attualmente non esegue il mapping di C# direttamente a JavaScript/WASM. Al contrario, Blazor l'interpretazione il nel browser, quindi le mappe di origine non sono rilevanti.

## <a name="firewall-configuration"></a>Configurazione del firewall

Se un firewall blocca la comunicazione con il proxy di debug, creare una regola di eccezione del firewall che consenta la comunicazione tra il browser e il `NodeJS` processo.

> [!WARNING]
> La modifica di una configurazione del firewall deve essere eseguita con cautela per evitare la creazione di vulnerablities di sicurezza. Applicare con attenzione le linee guida per la sicurezza, attenersi alle procedure di sicurezza consigliate e rispettare gli avvisi emessi dal produttore del firewall.
>
> Consentire la comunicazione aperta con il `NodeJS` processo:
>
> * Apre il server del nodo a qualsiasi connessione, a seconda delle funzionalità e della configurazione del firewall.
> * Potrebbe essere rischioso a seconda della rete.
> * **È consigliato solo nei computer di sviluppo.**
>
> Se possibile, consentire la comunicazione aperta solo con il `NodeJS` processo **su reti private o attendibili**.

Per informazioni aggiuntive sulla configurazione di [Windows Firewall](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security) , vedere [creare una regola di servizio o programma in ingresso](/windows/security/threat-protection/windows-firewall/create-an-inbound-program-or-service-rule). Per ulteriori informazioni, vedere [Windows Defender Firewall con protezione avanzata](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security) e articoli correlati nel set di documentazione Windows Firewall.

## <a name="troubleshoot"></a>Risolvere problemi

Se si verificano errori, è possibile che vengano visualizzati i suggerimenti seguenti:

* Nella scheda **debugger** aprire gli strumenti di sviluppo nel browser. Nella console eseguire `localStorage.clear()` per rimuovere tutti i punti di interruzione.
* Verificare di aver installato e considerato attendibile il certificato di sviluppo ASP.NET Core HTTPS. Per altre informazioni, vedere <xref:security/enforcing-ssl#troubleshoot-certificate-problems>.
* Visual Studio richiede l'opzione **Abilita debug JavaScript per ASP.NET (Chrome, Edge e IE)** in **strumenti**  >  **Opzioni**  >  **debug**  >  **generale**. Questa è l'impostazione predefinita per Visual Studio. Se il debug non funziona, verificare che sia selezionata l'opzione.
* Se l'ambiente usa un proxy HTTP, assicurarsi che `localhost` sia incluso nelle impostazioni di bypass del proxy. Questa operazione può essere eseguita impostando la `NO_PROXY` variabile di ambiente in uno dei seguenti valori:
  * `launchSettings.json`File per il progetto.
  * A livello di utente o di variabili di ambiente di sistema per applicarlo a tutte le app. Quando si usa una variabile di ambiente, riavviare Visual Studio per rendere effettive le modifiche.
* Assicurarsi che i firewall o i proxy non blocchino le comunicazioni con il proxy di debug ( `NodeJS` processo). Per ulteriori informazioni, vedere la sezione relativa alla [configurazione del firewall](#firewall-configuration) .

### <a name="breakpoints-in-oninitializedasync-not-hit"></a>Punti di interruzione in `OnInitialized{Async}` non riscontri

Il Blazor proxy di debug del Framework richiede un breve intervallo di tempo, quindi i punti di interruzione nel [ `OnInitialized{Async}` metodo del ciclo](xref:blazor/components/lifecycle#component-initialization-methods) di vita potrebbero non essere raggiunti. Si consiglia di aggiungere un ritardo all'inizio del corpo del metodo per consentire al proxy di debug di avviarsi prima che venga raggiunto il punto di interruzione. È possibile includere il ritardo in base a una [ `if` direttiva del compilatore](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if) per assicurarsi che il ritardo non sia presente per una build di rilascio dell'app.

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:

```csharp
protected override void OnInitialized()
{
#if DEBUG
    Thread.Sleep(10000)
#endif

    ...
}
```

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>:

```csharp
protected override async Task OnInitializedAsync()
{
#if DEBUG
    await Task.Delay(10000)
#endif

    ...
}
```

### <a name="visual-studio-windows-timeout"></a>Timeout di Visual Studio (Windows)

Se Visual Studio genera un'eccezione che non è stato possibile avviare l'adattatore di debug, indicando che è stato raggiunto il timeout, è possibile modificare il timeout con un'impostazione del registro di sistema:

```console
VsRegEdit.exe set "<VSInstallFolder>" HKCU JSDebugger\Options\Debugging "BlazorTimeoutInMilliseconds" dword {TIMEOUT}
```

Il `{TIMEOUT}` segnaposto nel comando precedente è in millisecondi. Ad esempio, un minuto viene assegnato come `60000` .
