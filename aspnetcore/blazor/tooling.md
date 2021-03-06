---
title: Strumenti per ASP.NET Core Blazor
author: guardrex
description: Informazioni sugli strumenti disponibili per la compilazione di Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/11/2021
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
uid: blazor/tooling
zone_pivot_groups: operating-systems
ms.openlocfilehash: 19270bb74326dccfee9466b7c1fa61daeab805a2
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394460"
---
# <a name="tooling-for-aspnet-core-blazor"></a>Strumenti per ASP.NET Core Blazor

::: zone pivot="windows"

1. Installare la versione più recente di [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) con il carico di lavoro **sviluppo di ASP.NET e Web** .

1. Creare un nuovo progetto.

1. Selezionare **Blazor app**. Selezionare **Avanti**.

1. Specificare il nome di un progetto nel campo **Nome progetto** oppure accettare il nome predefinito. Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto. Selezionare **Crea**.

1. Per un' Blazor WebAssembly esperienza, scegliere il modello **Blazor WebAssembly app** . Per un' Blazor Server esperienza, scegliere il modello **Blazor Server app** . Selezionare **Crea**.

   Per un'esperienza ospitata Blazor WebAssembly , selezionare la casella di controllo **ASP.NET Core Hosted** .

   Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* (standalone e Hosted) e *Blazor Server* , vedere <xref:blazor/hosting-models> .

1. Premere <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app.

Per ulteriori informazioni su come considerare attendibile il certificato di sviluppo HTTPS ASP.NET Core, vedere <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos> .

Quando si esegue un'app ospitata Blazor WebAssembly , eseguire l'app dal progetto della soluzione **`Server`** .

::: zone-end

::: zone pivot="linux"

1. Installare la versione più recente del [.NET Core SDK](https://dotnet.microsoft.com/download). Se in precedenza è stato installato l'SDK, è possibile determinare la versione installata eseguendo il comando seguente in una shell dei comandi:

   ```dotnetcli
   dotnet --version
   ```

1. Installare la versione più recente di [Visual Studio Code](https://code.visualstudio.com).

1. Installare la versione più recente [di C# per Visual Studio Code estensione](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).

1. Per un' Blazor WebAssembly esperienza, eseguire il comando seguente in una shell dei comandi:

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   ```

   Per un'esperienza ospitata Blazor WebAssembly , aggiungere l'opzione Hosted ( `-ho` o `--hosted` ) al comando:

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1 -ho
   ```

   Per un' Blazor Server esperienza, eseguire il comando seguente in una shell dei comandi:

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   ```

   Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* (standalone e Hosted) e *Blazor Server* , vedere <xref:blazor/hosting-models> .

1. Aprire la cartella `WebApplication1` in Visual Studio Code.

1. Le richieste dell'IDE aggiungono risorse per compilare ed eseguire il debug del progetto. Selezionare **Sì**.

   **Blazor WebAssemblyAvvio e configurazione attività ospitate**

   Per le Blazor WebAssembly soluzioni ospitate, aggiungere (o spostare) la `.vscode` cartella con i `launch.json` file e nella `tasks.json` cartella padre della soluzione, ovvero la cartella che contiene i nomi tipici delle cartelle di progetto `Client` , `Server` e `Shared` . Aggiornare o verificare che la configurazione nei `launch.json` file e `tasks.json` esegua un'app ospitata Blazor WebAssembly dal **`Server`** progetto.

   **`.vscode/launch.json`** ( `launch` configurazione):

   ```json
   ...
   "cwd": "${workspaceFolder}/{SERVER APP FOLDER}",
   ...
   ```

   Nella configurazione precedente per la directory di lavoro corrente ( `cwd` ), il `{SERVER APP FOLDER}` segnaposto è la **`Server`** cartella del progetto, in genere " `Server` ".

   Se viene usato Microsoft Edge e Google Chrome non è installato nel sistema, aggiungere una proprietà aggiuntiva di `"browser": "edge"` alla configurazione.

   Esempio per una cartella di progetto di `Server` e che genera Microsoft Edge come browser per le esecuzioni di debug anziché il browser predefinito Google Chrome:

   ```json
   ...
   "cwd": "${workspaceFolder}/Server",
   "browser": "edge"
   ...
   ```

   **`.vscode/tasks.json`**(argomenti del [ `dotnet` comando](/dotnet/core/tools/dotnet) ):

   ```json
   ...
   "${workspaceFolder}/{SERVER APP FOLDER}/{PROJECT NAME}.csproj",
   ...
   ```

   Nell'argomento precedente:

   * Il `{SERVER APP FOLDER}` segnaposto è la **`Server`** cartella del progetto, in genere " `Server` ".
   * Il `{PROJECT NAME}` segnaposto è il nome dell'app, in genere in base al nome della soluzione seguito da " `.Server` " in un'app generata dal [ Blazor modello di progetto](xref:blazor/project-structure).

   Nell'esempio seguente dell' [esercitazione per l'uso di SignalR con un' Blazor WebAssembly app](xref:tutorials/signalr-blazor) viene usato il nome della cartella del progetto `Server` e il nome di un progetto `BlazorWebAssemblySignalRApp.Server` :

   ```json
   ...
   "args": [
     "build",
       "${workspaceFolder}/Server/BlazorWebAssemblySignalRApp.Server.csproj",
       "/property:GenerateFullPaths=true",
       "/consoleloggerparameters:NoSummary"
   ],
   ...
   ```

1. Premere <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app.

## <a name="trust-a-development-certificate"></a>Attendibilità di un certificato di sviluppo

Non esiste un modo centralizzato per considerare attendibile un certificato in Linux. Viene in genere adottato uno degli approcci seguenti:

* Escludere l'URL dell'app nell'elenco di esclusione del browser.
* Considerare attendibili tutti i certificati autofirmati per `localhost` .
* Aggiungere il certificato all'elenco dei certificati attendibili nel browser.

Per ulteriori informazioni, vedere le indicazioni fornite dal produttore del browser e dalla distribuzione di Linux.

::: zone-end

::: zone pivot="macos"

1. Installare [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/).

1. Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.

1. Nella barra laterale selezionare app **Web e console**  >  .

   Per un' Blazor WebAssembly esperienza, scegliere il modello **Blazor WebAssembly app** . Per un' Blazor Server esperienza, scegliere il modello **Blazor Server app** . Selezionare **Avanti**.

   Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* (standalone e Hosted) e *Blazor Server* , vedere <xref:blazor/hosting-models> .

1. Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**. Selezionare **Avanti**.

1. Per un'esperienza ospitata Blazor WebAssembly , selezionare la casella di controllo **ASP.NET Core Hosted** .

1. Nel campo **nome progetto** assegnare un nome all'app `WebApplication1` . Selezionare **Crea**.

1. Selezionare **Esegui**  >  **Avvia senza eseguire debug** per eseguire l'app *senza il debugger*. Eseguire l'app con **Esegui**  >  **debug Avvia debug** o il pulsante Esegui (&#9654;) per eseguire l'app *con il debugger*.

Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare. Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain. Per ulteriori informazioni su come considerare attendibile il certificato di sviluppo HTTPS ASP.NET Core, vedere <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos> .

Quando si esegue un'app ospitata Blazor WebAssembly , eseguire l'app dal progetto della soluzione **`Server`** .

::: zone-end

## <a name="use-visual-studio-code-for-cross-platform-blazor-development"></a>Usare Visual Studio Code per Blazor lo sviluppo multipiattaforma

[Visual Studio Code](https://code.visualstudio.com/) è un ambiente di sviluppo integrato (IDE) open source e multipiattaforma che può essere usato per sviluppare Blazor app. Usare l'interfaccia della riga di comando di .NET per creare una nuova Blazor app per lo sviluppo con Visual Studio Code. Per altre informazioni, vedere la [versione di questo articolo](?pivots=linux)relativa a Linux.

## <a name="blazor-template-options"></a>Blazor opzioni del modello

Il Blazor Framework fornisce modelli per la creazione di nuove app per ognuno dei due Blazor modelli di hosting. I modelli vengono usati per creare nuovi Blazor progetti e soluzioni indipendentemente dagli strumenti selezionati per Blazor lo sviluppo (Visual Studio, Visual Studio per Mac, Visual Studio Code o l'interfaccia della riga di comando di .NET):

* Blazor WebAssembly modello di progetto: `blazorwasm`
* Blazor Server modello di progetto: `blazorserver`

Per ulteriori informazioni sui Blazor modelli di hosting di, vedere <xref:blazor/hosting-models> . Per ulteriori informazioni sui Blazor modelli di progetto di, vedere <xref:blazor/project-structure> .

Le opzioni del modello sono disponibili passando l'opzione Help ( `-h` o `--help` ) al [`dotnet new`](/dotnet/core/tools/dotnet-new) comando CLI in una shell dei comandi:

```dotnetcli
dotnet new blazorwasm -h
dotnet new blazorserver -h
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/hosting-models>
* <xref:blazor/project-structure>
