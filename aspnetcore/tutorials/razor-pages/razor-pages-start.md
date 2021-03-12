---
title: 'Esercitazione: Introduzione alle Razor pagine in ASP.NET Core'
author: rick-anderson
description: Questa è la prima esercitazione di una serie che illustra le nozioni di base per la creazione di un' Razor app Web di ASP.NET Core Pages.
ms.author: riande
ms.date: 09/15/2020
ms.custom: contperf-fy21q2
no-loc:
- Index
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
uid: tutorials/razor-pages/razor-pages-start
ms.openlocfilehash: 61ff4edfaedb22ea88d6bb0be2390ff734879942
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588438"
---
# <a name="tutorial-get-started-with-razor-pages-in-aspnet-core"></a>Esercitazione: Introduzione alle Razor pagine in ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"
Questa è la prima esercitazione di una serie che illustra le nozioni di base per la creazione di un' Razor app Web di ASP.NET Core Pages.

Per un'introduzione più avanzata destinata agli sviluppatori che hanno familiarità con i controller e le visualizzazioni, vedere [Introduzione alle Razor pagine](xref:razor-pages/index).

Al termine della serie si otterrà un'app che gestisce un database di film.  

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).

In questa esercitazione:

> [!div class="checklist"]
> * Creare un' Razor app Web di pagine.
> * Eseguire l'app.
> * Esaminare i file di progetto.

Al termine di questa esercitazione, si disporrà di un' Razor app Web di pagine di lavoro che verrà migliorata nelle esercitazioni successive.

![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/5/home5.png)

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-5.0.md)]

---

## <a name="create-a-razor-pages-web-app"></a>Creare un' Razor app Web di pagine

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Avviare Visual Studio e selezionare **Crea un nuovo progetto**. Per altre informazioni, vedere [creare un nuovo progetto in Visual Studio](/visualstudio/ide/create-new-project).

   ![Creare un nuovo progetto dalla finestra di avvio](razor-pages-start/_static/5/start-window-create-new-project.png)

1. Nella finestra di dialogo **Crea un nuovo progetto** selezionare **Applicazione Web ASP.NET Core**, quindi selezionare **Avanti**.

    ![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/5/np.png)
    
1. Nella finestra di dialogo **Configura nuovo progetto** immettere `RazorPagesMovie` per **nome progetto**. È importante denominare il progetto *Razor PagesMovie*, inclusa la corrispondenza con le maiuscole, in modo che gli spazi dei nomi corrispondano quando si copia e incolla il codice di esempio.

1. Selezionare **Crea**.

    ![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/config.png)

1. Nella finestra di dialogo **Crea una nuova applicazione web ASP.NET Core** selezionare:
    1. **.NET Core** e **ASP.NET Core 5,0** negli elenchi a discesa.
    1. **Applicazione Web**.
    1. **Create**.

     ![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/5/npx.png)

    Viene creato il progetto iniziale seguente:

    ![Esplora soluzioni](razor-pages-start/_static/se2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).

1. Passare alla directory (`cd`) che conterrà il progetto.

1. Eseguire i comandi seguenti:

   ```dotnetcli
   dotnet new webapp -o RazorPagesMovie
   code -r RazorPagesMovie
   ```

   * Il `dotnet new` comando crea un nuovo Razor progetto di pagine nella cartella *Razor PagesMovie* .
   * Il `code` comando apre la cartella *Razor PagesMovie* nell'istanza corrente di Visual Studio Code.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

1. Selezionare **File** > **Nuova soluzione**.

    ![Nuova soluzione macOS](../first-mvc-app/start-mvc/_static/new_project_vsmac.png)

1. In Visual Studio per Mac precedente alla versione 8,6 selezionare   >    >  **applicazione Web** app .NET Core  >  **Avanti**. Nella versione 8,6 o successive selezionare applicazione Web **e**  >  **app**  >  **Web dell'** app console  >  **successiva**.

    ![selezione del modello di app Web macOS](razor-pages-start/_static/web_app_template_vsmac.png)

1. Nella finestra di dialogo **Configura nuova applicazione Web** :

    1. Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.
    1. Se viene visualizzata un'opzione per la selezione di un **Framework di destinazione**, selezionare la versione più recente di .NET 5. x.
    1. Selezionare **Avanti**.

1. Assegnare al progetto il nome *Razor PagesMovie* e selezionare **Crea**.

    ![macOS nome del progetto](razor-pages-start/_static/RazorPagesMovie.png)

<!-- End of VS tabs -->

---

## <a name="run-the-app"></a>Eseguire l'app

  [!INCLUDE[](~/includes/run-the-app.md)]

## <a name="examine-the-project-files"></a>Esaminare i file di progetto

Di seguito viene visualizzata una panoramica delle cartelle e dei file principali del progetto, che verranno usati in esercitazioni successive.

### <a name="pages-folder"></a>Cartella Pages

Contiene Razor pagine e file di supporto. Ogni Razor pagina è una coppia di file:

* Un file con *estensione cshtml* con markup HTML con codice C# che usa la Razor sintassi.
* Un file con *estensione cshtml.cs* con codice C# che gestisce gli eventi di pagina.

I nomi dei file di supporto iniziano con un carattere di sottolineatura. Ad esempio, il file *_Layout. cshtml* configura gli elementi dell'interfaccia utente comuni a tutte le pagine. Questo file imposta il menu di navigazione nella parte superiore della pagina e le informazioni sul copyright in fondo alla pagina. Per altre informazioni, vedere <xref:mvc/views/layout>.

### <a name="wwwroot-folder"></a>Cartella wwwroot

Contiene risorse statiche, ad esempio file HTML, file JavaScript e file CSS. Per altre informazioni, vedere <xref:fundamentals/static-files>.

### appsettings.json

Contiene i dati di configurazione, come le stringhe di connessione. Per altre informazioni, vedere <xref:fundamentals/configuration/index>.

### <a name="programcs"></a>Program.cs

Contiene il punto di ingresso per l'app. Per altre informazioni, vedere <xref:fundamentals/host/generic-host>.

### <a name="startupcs"></a>Startup.cs

Contiene codice che consente di configurare il comportamento dell'app. Per altre informazioni, vedere <xref:fundamentals/startup>.

## <a name="next-steps"></a>Passaggi successivi

> [!div class="step-by-step"]
> [Passaggio successivo: aggiungere un modello](xref:tutorials/razor-pages/model)

::: moniker-end

<!--::: moniker range=">= aspnetcore-5.0" -->

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

Questa è la prima esercitazione di una serie che illustra le nozioni di base per la creazione di un' Razor app Web di ASP.NET Core Pages.

Per un'introduzione più avanzata destinata agli sviluppatori che hanno familiarità con i controller e le visualizzazioni, vedere [Introduzione alle Razor pagine](xref:razor-pages/index).

Al termine della serie si otterrà un'app che gestisce un database di film.  

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).

In questa esercitazione:

> [!div class="checklist"]
> * Creare un' Razor app Web di pagine.
> * Eseguire l'app.
> * Esaminare i file di progetto.

Al termine di questa esercitazione, si disporrà di un' Razor app Web di pagine di lavoro che verrà compilata nelle esercitazioni successive.

![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/home2.2.png)

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-razor-pages-web-app"></a>Creare un' Razor app Web di pagine

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.
* Creare una nuova applicazione Web ASP.NET Core e selezionare **Avanti**.
  ![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/np_2.1.png)
* Denominare il progetto **Razor PagesMovie**. È importante denominare il progetto *Razor PagesMovie* in modo che gli spazi dei nomi corrispondano quando si copia e incolla il codice.
  ![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/config.png)

* Selezionare **ASP.NET Core 3,1** nell'elenco a discesa, **applicazione Web**, quindi selezionare **Crea**.

![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/3/npx.png)

  Viene creato il progetto iniziale seguente:

  ![Esplora soluzioni](razor-pages-start/_static/se2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).

* Passare alla directory (`cd`) che conterrà il progetto.

* Eseguire i comandi seguenti:

  ```dotnetcli
  dotnet new webapp -o RazorPagesMovie
  code -r RazorPagesMovie
  ```

  * Il `dotnet new` comando crea un nuovo Razor progetto di pagine nella cartella *Razor PagesMovie* .
  * Il `code` comando apre la cartella *Razor PagesMovie* nell'istanza corrente di Visual Studio Code.

* Dopo che l'icona di OmniSharp Flame della barra di stato è verde, una finestra di dialogo richiede che le **risorse necessarie per la compilazione e il debug siano mancanti da' Razor PagesMovie '. Aggiungerli?** Selezionare **Sì**.

  Una directory *.vscode* che contiene i file *launch.json* e *tasks.json* viene aggiunta alla directory radice del progetto.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Selezionare **File** > **Nuova soluzione**.

  ![Nuova soluzione macOS](../first-mvc-app/start-mvc/_static/new_project_vsmac.png)

* In Visual Studio per Mac precedente alla versione 8,6 selezionare   >    >  **applicazione Web** app .NET Core  >  **Avanti**. Nella versione 8,6 o successive selezionare applicazione Web **e**  >  **app**  >  **Web dell'** app console  >  **successiva**.

  ![selezione del modello di app Web macOS](razor-pages-start/_static/web_app_template_vsmac.png)

* Nella finestra di dialogo **Configura nuova applicazione Web** :

  * Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.
  * Se viene visualizzata un'opzione per la selezione di un **Framework di destinazione**, selezionare la versione 3. x più recente.

  Selezionare **Avanti**.

* Assegnare al progetto il nome **Razor PagesMovie** e quindi selezionare **Crea**.

  ![macOS nome del progetto](razor-pages-start/_static/RazorPagesMovie.png)

<!-- End of VS tabs -->

---

## <a name="run-the-app"></a>Eseguire l'app

  [!INCLUDE[](~/includes/run-the-app.md)]

## <a name="examine-the-project-files"></a>Esaminare i file di progetto

Di seguito viene visualizzata una panoramica delle cartelle e dei file principali del progetto, che verranno usati in esercitazioni successive.

### <a name="pages-folder"></a>Cartella Pages

Contiene Razor pagine e file di supporto. Ogni Razor pagina è una coppia di file:

* Un file con *estensione cshtml* con markup HTML con codice C# che usa la Razor sintassi.
* Un file con *estensione cshtml.cs* con codice C# che gestisce gli eventi di pagina.

I nomi dei file di supporto iniziano con un carattere di sottolineatura. Ad esempio, il file *_Layout. cshtml* configura gli elementi dell'interfaccia utente comuni a tutte le pagine. Questo file imposta il menu di navigazione nella parte superiore della pagina e le informazioni sul copyright in fondo alla pagina. Per altre informazioni, vedere <xref:mvc/views/layout>.

### <a name="wwwroot-folder"></a>Cartella wwwroot

Contiene i file statici, ad esempio i file HTML, i file JavaScript e i file CSS. Per altre informazioni, vedere <xref:fundamentals/static-files>.

### <a name="appsettingsjson"></a>appSettings.json

Contiene i dati di configurazione, come le stringhe di connessione. Per altre informazioni, vedere <xref:fundamentals/configuration/index>.

### <a name="programcs"></a>Program.cs

Contiene il punto di ingresso per il programma. Per altre informazioni, vedere <xref:fundamentals/host/generic-host>.

### <a name="startupcs"></a>Startup.cs

Contiene codice che consente di configurare il comportamento dell'app. Per altre informazioni, vedere <xref:fundamentals/startup>.

## <a name="next-steps"></a>Passaggi successivi

> [!div class="step-by-step"]
> [Passaggio successivo: aggiungere un modello](xref:tutorials/razor-pages/model)

::: moniker-end

<!--::: moniker range=">= aspnetcore-3.0" -->

::: moniker range="< aspnetcore-3.0"

Questa è la prima esercitazione di una serie. [La serie](xref:tutorials/razor-pages/index) illustra le nozioni di base per la creazione di un' Razor app Web di ASP.NET Core Pages.

Per un'introduzione più avanzata destinata agli sviluppatori che hanno familiarità con i controller e le visualizzazioni, vedere [Introduzione alle Razor pagine](xref:razor-pages/index).

Al termine della serie si otterrà un'app che gestisce un database di film.  

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).

In questa esercitazione:

> [!div class="checklist"]
> * Creare un' Razor app Web di pagine.
> * Eseguire l'app.
> * Esaminare i file di progetto.

Al termine di questa esercitazione, si disporrà di un' Razor app Web di pagine di lavoro che verrà compilata nelle esercitazioni successive.

![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/home2.2.png)

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

## <a name="create-a-razor-pages-web-app"></a>Creare un' Razor app Web di pagine

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.

* Creare una nuova applicazione Web ASP.NET Core e selezionare **Avanti**.

  ![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/np_2.1.png)

* Denominare il progetto **Razor PagesMovie**. È importante denominare il progetto *Razor PagesMovie* in modo che gli spazi dei nomi corrispondano quando si copia e incolla il codice.

  ![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/config.png)

* Selezionare **ASP.NET Core 2.2** nell'elenco a discesa **Applicazione Web** e quindi selezionare **Crea**.

![nuova applicazione Web ASP.NET Core](razor-pages-start/_static/np_2_2.2.png)

  Viene creato il progetto iniziale seguente:

  ![Esplora soluzioni](razor-pages-start/_static/se2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).

* Passare alla directory (`cd`) che conterrà il progetto.

* Eseguire i comandi seguenti:

  ```dotnetcli
  dotnet new webapp -o RazorPagesMovie
  code -r RazorPagesMovie
  ```

  * Il `dotnet new` comando crea un nuovo Razor progetto di pagine nella cartella *Razor PagesMovie* .
  * Il `code` comando apre la cartella *Razor PagesMovie* nell'istanza corrente di Visual Studio Code.

* Dopo che l'icona di OmniSharp Flame della barra di stato è verde, una finestra di dialogo richiede che le **risorse necessarie per la compilazione e il debug siano mancanti da' Razor PagesMovie '. Aggiungerli?** Selezionare **Sì**.

  Una directory *.vscode* che contiene i file *launch.json* e *tasks.json* viene aggiunta alla directory radice del progetto.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Selezionare **File** > **Nuova soluzione**.

![Nuova soluzione macOS](../first-mvc-app/start-mvc/_static/new_project_vsmac.png)

* In Visual Studio per Mac precedente alla versione 8,6 selezionare   >    >  **applicazione Web** app .NET Core  >  **Avanti**. Nella versione 8,6 o successive selezionare applicazione Web **e**  >  **app**  >  **Web dell'** app console  >  **successiva**.

* Nella finestra di dialogo **Configura nuova applicazione Web** :

  * Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.
  * Se viene visualizzata un'opzione per la selezione di un **Framework di destinazione**, selezionare la versione 2. x più recente.

  Selezionare **Avanti**.

* Assegnare al progetto il nome **Razor PagesMovie** e quindi selezionare **Crea**.

  ![nameproj](razor-pages-start/_static/RazorPagesMovie.png)

<!-- End of VS tabs -->

---

## <a name="run-the-app"></a>Eseguire l'app

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Premere CTRL+F5 per l'esecuzione senza il debugger.

  L'avvio dell'app con <kbd>CTRL + F5</kbd> (modalità non di debug) consente di apportare modifiche al codice, salvare il file, aggiornare il browser e visualizzare le modifiche apportate al codice. Molti sviluppatori preferiscono usare la modalità non di debug per avviare l'app rapidamente e visualizzare le modifiche.

  [!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio avvia [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) ed esegue l'app. La barra degli indirizzi visualizza `localhost:port#` e non `example.com` o simili. Ciò accade perché `localhost` è il nome host standard per il computer locale. Localhost viene usato solo per le richieste web del computer locale. Quando Visual Studio crea un progetto Web, viene usata una porta casuale per il server Web.

* Nella home page dell'app selezionare **Accetto** per autorizzare il rilevamento.

  Questa app non tiene traccia delle informazioni personali, ma il modello di progetto include la funzionalità di consenso nel caso in cui risulti necessaria la conformità al [Regolamento generale sulla protezione dei dati (GDPR) dell'Unione Europea](xref:security/gdpr).

  ![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/homeGDPR2.2.png)

  L'immagine seguente mostra l'app dopo il consenso al rilevamento:

  ![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/home2.2.png)
  
# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

  [!INCLUDE[](~/includes/trustCertVSC.md)]

* Premere <kbd>CTRL + F5</kbd> per eseguire senza il debugger.

  L'avvio dell'app con <kbd>CTRL + F5</kbd> (modalità non di debug) consente di apportare modifiche al codice, salvare il file, aggiornare il browser e visualizzare le modifiche apportate al codice. Molti sviluppatori preferiscono usare la modalità non di debug per avviare l'app rapidamente e visualizzare le modifiche.

  Visual Studio Code avvia [gheppio](xref:fundamentals/servers/kestrel), avvia un browser e passa a `http://localhost:5001` . La barra degli indirizzi visualizza `localhost:port#` e non `example.com` o simili. Ciò accade perché `localhost` è il nome host standard per il computer locale. Localhost viene usato solo per le richieste web del computer locale.

* Nella home page dell'app selezionare **Accetto** per autorizzare il rilevamento.

  Questa app non tiene traccia delle informazioni personali, ma il modello di progetto include la funzionalità di consenso nel caso in cui risulti necessaria la conformità al [Regolamento generale sulla protezione dei dati (GDPR) dell'Unione Europea](xref:security/gdpr).

  ![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/homeGDPR2.2.png)

  La figura seguente visualizza l'app dopo che è stato impostato il consenso al rilevamento:

  ![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/home2.2.png)
  
# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

  [!INCLUDE[](~/includes/trustCertMac.md)]

* Premere **Cmd-Opt-F5** per l'esecuzione senza il debugger.

  L'avvio dell'app con <kbd>cmd + opt + F5</kbd> (modalità non di debug) consente di apportare modifiche al codice, salvare il file, aggiornare il browser e visualizzare le modifiche del codice. Molti sviluppatori preferiscono usare la modalità non di debug per avviare l'app rapidamente e visualizzare le modifiche.

  Visual Studio avvia [gheppio](xref:fundamentals/servers/kestrel), avvia un browser e passa a `http://localhost:5001` .

* Nella home page dell'app selezionare **Accetto** per autorizzare il rilevamento.

  Questa app non tiene traccia delle informazioni personali, ma il modello di progetto include la funzionalità di consenso nel caso in cui risulti necessaria la conformità al [Regolamento generale sulla protezione dei dati (GDPR) dell'Unione Europea](xref:security/gdpr).

  ![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/homeGDPR2.2_safari.png)

  L'immagine seguente mostra l'app dopo il consenso al rilevamento:

  ![Home o::: NO-LOC (index)::: page](razor-pages-start/_static/home2.2_safari.png)

<!-- End of VS tabs -->

---

## <a name="examine-the-project-files"></a>Esaminare i file di progetto

Di seguito viene visualizzata una panoramica delle cartelle e dei file principali del progetto, che verranno usati in esercitazioni successive.

### <a name="pages-folder"></a>Cartella Pages

Contiene Razor pagine e file di supporto. Ogni Razor pagina è una coppia di file:

* Un file con *estensione cshtml* con markup HTML con codice C# che usa la Razor sintassi.
* Un file con *estensione cshtml.cs* con codice C# che gestisce gli eventi di pagina.

I nomi dei file di supporto iniziano con un carattere di sottolineatura. Ad esempio, il file *_Layout. cshtml* configura gli elementi dell'interfaccia utente comuni a tutte le pagine. Questo file imposta il menu di navigazione nella parte superiore della pagina e le informazioni sul copyright in fondo alla pagina. Per altre informazioni, vedere <xref:mvc/views/layout>.

Razor Le pagine sono derivate da `PageModel` . Per convenzione, la `PageModel` classe derivata da è denominata `<PageName>Model` .

### <a name="wwwroot-folder"></a>Cartella wwwroot

Contiene i file statici, ad esempio i file HTML, i file JavaScript e i file CSS. Per altre informazioni, vedere <xref:fundamentals/static-files>.

### <a name="appsettingsjson"></a>appSettings.json

Contiene i dati di configurazione, come le stringhe di connessione. Per altre informazioni, vedere <xref:fundamentals/configuration/index>.

### <a name="programcs"></a>Program.cs

Contiene il punto di ingresso per il programma. Per altre informazioni, vedere <xref:fundamentals/host/generic-host>.

### <a name="startupcs"></a>Startup.cs

Contiene il codice che configura il comportamento dell'app, ad esempio se richiede il consenso per cookie s. Per altre informazioni, vedere <xref:fundamentals/startup>.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Versione YouTube dell'esercitazione](https://www.youtube.com/watch?v=F0SP7Ry4flQ&feature=youtu.be)

## <a name="next-steps"></a>Passaggi successivi

> [!div class="step-by-step"]
> [Passaggio successivo: aggiungere un modello](xref:tutorials/razor-pages/model)

::: moniker-end
