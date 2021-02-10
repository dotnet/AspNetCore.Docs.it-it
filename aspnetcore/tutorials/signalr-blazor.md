---
title: Usare ASP.NET Core SignalR con Blazor
author: guardrex
description: Creare un'app di chat che usa ASP.NET Core SignalR con Blazor .
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/25/2021
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
uid: tutorials/signalr-blazor
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: 226813f3b975e207a51eba24d6cbb57f00464073
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107207"
---
# <a name="use-aspnet-core-signalr-with-blazor"></a>Usare ASP.NET Core SignalR con Blazor

Di [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)

Questa esercitazione illustra le nozioni di base per la creazione di un'app in tempo reale usando SignalR con Blazor . Si apprenderà come:

> [!div class="checklist"]
> * Creazione di un Blazor progetto
> * Aggiungere la SignalR libreria client
> * Aggiungere un SignalR Hub
> * Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub
> * Aggiungi Razor codice componente per la chat

Al termine di questa esercitazione, si disporrà di un'app di chat funzionante.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/signalr-blazor/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

::: moniker range=">= aspnetcore-5.0"

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* [Visual Studio 2019 16,8 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* [Visual Studio per Mac versione 8,8 o successiva](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

[!INCLUDE[](~/includes/5.0-SDK.md)]

---

::: moniker-end

::: moniker range="< aspnetcore-5.0"

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* [Visual Studio 2019 16,6 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* [Visual Studio per Mac versione 8,6 o successiva](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

[!INCLUDE[](~/includes/3.1-SDK.md)]

---

::: moniker-end

::: zone pivot="webassembly"

## <a name="create-a-hosted-blazor-webassembly-app"></a>Creare un'app ospitata Blazor WebAssembly

Seguire le istruzioni per la scelta degli strumenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.

::: moniker-end

1. Creare un nuovo progetto.

1. Selezionare **Blazor app** e fare clic su **Next (avanti**).

1. Digitare `BlazorWebAssemblySignalRApp` nel campo **nome progetto** . Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto. Selezionare **Crea**.

1. Scegliere il modello di **Blazor WebAssembly app** .

1. In **Avanzate** selezionare la casella di controllo **ASP.NET Core Hosted** .

1. Selezionare **Crea**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. In una shell dei comandi eseguire il comando seguente:

   ```dotnetcli
   dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
   ```

   L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .

   L' `-o|--output` opzione Crea una cartella per la soluzione. Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.

1. In Visual Studio Code aprire la cartella del progetto dell'app.

1. Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**. Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:

1. Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.

1. Nella barra laterale selezionare app **Web e console**  >  .

1. Scegliere il modello di **Blazor WebAssembly app** . Selezionare **Avanti**.

1. Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**. Selezionare la casella di controllo **ASP.NET Core Hosted** . Selezionare **Avanti**.

1. Nel campo **nome progetto** assegnare un nome all'app `BlazorWebAssemblySignalRApp` . Selezionare **Crea**.

   Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare. Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.

1. Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

In una shell dei comandi eseguire il comando seguente:

```dotnetcli
dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
```

L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .

L' `-o|--output` opzione Crea una cartella per la soluzione. Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.

---

## <a name="add-the-signalr-client-library"></a>Aggiungere la SignalR libreria client

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio/)

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.

1. Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto. Impostare la versione in modo che corrisponda al Framework condiviso dell'app. Selezionare **Installa**.

1. Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code/)

Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.

1. Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto. Impostare la versione in modo che corrisponda al Framework condiviso dell'app. Selezionare **Aggiungi pacchetto**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a>Aggiungere il pacchetto System. Text. Encodings. Web

*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*

A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il `BlazorWebAssemblySignalRApp.Server` progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) . Il problema sottostante verrà risolto in una versione futura della patch di .NET 5. Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).

Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al `BlazorWebAssemblySignalRApp.Server` progetto della soluzione ospitata ASP.NET Core 3,1 Blazor , seguire le istruzioni per la scelta degli strumenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio/)

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.

1. Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto. Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso. Selezionare **Installa**.

1. Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code/)

Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.

1. Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

---

::: moniker-end

## <a name="add-a-signalr-hub"></a>Aggiungere un SignalR Hub

Nel `BlazorWebAssemblySignalRApp.Server` progetto creare una `Hubs` cartella (plurale) e aggiungere la classe seguente `ChatHub` ( `Hubs/ChatHub.cs` ):

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a>Aggiungere servizi e un endpoint per l' SignalR Hub

1. Nel progetto `BlazorWebAssemblySignalRApp.Server` aprire il file `Startup.cs`.

1. Aggiungere lo spazio dei nomi per la `ChatHub` classe all'inizio del file:

   ```csharp
   using BlazorWebAssemblySignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,6-10)]

1. In `Startup.Configure`:

   * Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.
   * Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,26)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,5-9)]

1. In `Startup.Configure`:

   * Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.
   * Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,25)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a>Aggiungi Razor codice componente per la chat

1. Nel progetto `BlazorWebAssemblySignalRApp.Client` aprire il file `Pages/Index.razor`.

::: moniker range=">= aspnetcore-5.0"

1. Sostituire il markup con il codice seguente:

   [!code-razor[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. Sostituire il markup con il codice seguente:

   [!code-razor[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a>Eseguire l'app

Seguire le istruzioni per gli strumenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. In **Esplora soluzioni** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto. Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Nella barra laterale della **soluzione** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto. Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

1. In una shell dei comandi dalla cartella della soluzione eseguire i comandi seguenti:

   ```dotnetcli
   cd Server
   dotnet run
   ```

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

---

::: zone-end

::: zone pivot="server"

## <a name="create-a-blazor-server-app"></a>Creare un' Blazor Server app

Seguire le istruzioni per la scelta degli strumenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.

::: moniker-end

1. Creare un nuovo progetto.

1. Selezionare **Blazor app** e fare clic su **Next (avanti**).

1. Digitare `BlazorServerSignalRApp` nel campo **nome progetto** . Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto. Selezionare **Crea**.

1. Scegliere il modello di **Blazor Server app** .

1. Selezionare **Crea**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. In una shell dei comandi eseguire il comando seguente:

   ```dotnetcli
   dotnet new blazorserver -o BlazorServerSignalRApp
   ```

   L' `-o|--output` opzione Crea una cartella per il progetto. Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.

1. In Visual Studio Code aprire la cartella del progetto dell'app.

1. Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**. Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:

1. Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.

1. Nella barra laterale selezionare app **Web e console**  >  .

1. Scegliere il modello di **Blazor Server app** . Selezionare **Avanti**.

1. Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**. Selezionare **Avanti**.

1. Nel campo **nome progetto** assegnare un nome all'app `BlazorServerSignalRApp` . Selezionare **Crea**.

   Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare. Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.

1. Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

In una shell dei comandi eseguire il comando seguente:

```dotnetcli
dotnet new blazorserver -o BlazorServerSignalRApp
```

L' `-o|--output` opzione Crea una cartella per il progetto. Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.

---

## <a name="add-the-signalr-client-library"></a>Aggiungere la SignalR libreria client

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio/)

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.

1. Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto. Impostare la versione in modo che corrisponda al Framework condiviso dell'app. Selezionare **Installa**.

1. Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code/)

Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.

1. Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto. Impostare la versione in modo che corrisponda al Framework condiviso dell'app. Selezionare **Aggiungi pacchetto**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a>Aggiungere il pacchetto System. Text. Encodings. Web

*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*

A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) . Il problema sottostante verrà risolto in una versione futura della patch di .NET 5. Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).

Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al progetto, seguire le istruzioni per la scelta degli strumenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio/)

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.

1. Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto. Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso. Selezionare **Installa**.

1. Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code/)

Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.

1. Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .

1. Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.

1. Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.

1. Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.

---

::: moniker-end

## <a name="add-a-signalr-hub"></a>Aggiungere un SignalR Hub

Creare una `Hubs` cartella (plurale) e aggiungere la `ChatHub` classe seguente ( `Hubs/ChatHub.cs` ):

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a>Aggiungere servizi e un endpoint per l' SignalR Hub

1. Aprire il file `Startup.cs`.

1. Aggiungere gli spazi dei nomi per <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> e la `ChatHub` classe all'inizio del file:

   ```csharp
   using Microsoft.AspNetCore.ResponseCompression;
   using BlazorServerSignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. In `Startup.Configure`:

   * Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.
   * Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. In `Startup.Configure`:

   * Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.
   * Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a>Aggiungi Razor codice componente per la chat

1. Aprire il file `Pages/Index.razor`.

::: moniker range=">= aspnetcore-5.0"

1. Sostituire il markup con il codice seguente:

   [!code-razor[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. Sostituire il markup con il codice seguente:

   [!code-razor[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a>Eseguire l'app

Seguire le istruzioni per gli strumenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli/)

1. In una shell dei comandi dalla cartella del progetto, eseguire i comandi seguenti:

   ```dotnetcli
   dotnet run
   ```

1. Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.

1. Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio. Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)

---

::: zone-end

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione sono state illustrate le procedure per:

> [!div class="checklist"]
> * Creazione di un Blazor progetto
> * Aggiungere la SignalR libreria client
> * Aggiungere un SignalR Hub
> * Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub
> * Aggiungi Razor codice componente per la chat

Per altre informazioni sulla creazione di Blazor app, vedere la Blazor documentazione:

> [!div class="nextstepaction"]
> <xref:blazor/index>
> [Autenticazione del token di porta con Identity Server, WebSocket ed eventi di Server-Sent](xref:signalr/authn-and-authz#bearer-token-authentication)

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:signalr/introduction>
* [SignalR negoziazione tra le origini per l'autenticazione](xref:blazor/fundamentals/signalr#signalr-cross-origin-negotiation-for-authentication)
* <xref:blazor/debug>
* <xref:blazor/security/server/threat-mitigation>
