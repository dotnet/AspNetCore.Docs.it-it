---
title: Introduzione ad ASP.NET Core MVC
author: rick-anderson
description: Informazioni introduttive su ASP.NET Core MVC.
ms.author: riande
ms.date: 01/20/2021
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
uid: tutorials/first-mvc-app/start-mvc
ms.custom: contperf-fy21q3
ms.openlocfilehash: 8e5f216a354b1ed7559b433d3a4867627bf60df3
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589777"
---
# <a name="get-started-with-aspnet-core-mvc"></a>Introduzione ad ASP.NET Core MVC

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

Questa è la prima esercitazione di una serie che insegna ASP.NET Core lo sviluppo Web MVC con i controller e le visualizzazioni.

Alla fine della serie, si disporrà di un'app che gestisce e Visualizza i dati dei film. Si apprenderà come:

> [!div class="checklist"]
> * Creare un'app Web.
> * Aggiungere un modello ed eseguirne lo scaffolding.
> * Usare un database.
> * Aggiungere ricerca e convalida.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/first-mvc-app/start-mvc/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-5.0.md)]

---

## <a name="create-a-web-app"></a>Creare un'app Web

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Avviare Visual Studio e selezionare **Crea un nuovo progetto**.
* Nella finestra di dialogo **Crea un nuovo progetto** selezionare **ASP.NET Core applicazione Web** > **Avanti**.
* Nella finestra di dialogo **Configura nuovo progetto** immettere `MvcMovie` per **nome progetto**. È importante denominare il progetto *MvcMovie*. Quando viene copiato il codice, l'utilizzo delle maiuscole deve corrispondere a ogni corrispondenza `namespace` .
* Selezionare **Crea**.
* Nella finestra di dialogo **Crea una nuova applicazione web ASP.NET Core** selezionare:
  * **.NET Core** e **ASP.NET Core 5,0** negli elenchi a discesa.
  * **ASP.NET Core app Web (Model-View-Controller)**.
  * **Create**.

![Creare una nuova applicazione Web ASP.NET Core ](start-mvc/_static/mvcVS19v16.9.png)

Per approcci alternativi alla creazione del progetto, vedere [creare un nuovo progetto in Visual Studio](/visualstudio/ide/create-new-project).

Visual Studio ha usato il modello di progetto predefinito per il progetto MVC creato. Il progetto creato:

* È un'app funzionante.
* È un progetto iniziale di base.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

L'esercitazione presuppone una certa familiarità con VS Code. Per ulteriori informazioni, vedere la pagina relativa all'introduzione [a vs code](https://code.visualstudio.com/docs) e [Visual Studio Code Guida](#visual-studio-code-help).

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Passare alla directory ( `cd` ) che conterrà il progetto.
* Eseguire il comando seguente:

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * Se viene visualizzata una finestra di dialogo con le **risorse necessarie per la compilazione e il debug non è presente in ' MvcMovie '. Aggiungi?**, seleziona **Sì**

  * `dotnet new mvc -o MvcMovie`: Crea un nuovo progetto MVC ASP.NET Core nella cartella *MvcMovie* .
  * `code -r MvcMovie`: Carica il file di progetto *MvcMovie. csproj* in Visual Studio Code.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Selezionare **File** > **Nuova soluzione**.

  ![Nuova soluzione macOS](start-mvc/_static/new_project_vsmac.png)

* In Visual Studio per Mac precedente alla versione 8,6, selezionare applicazione Web **.NET Core**  >  **app**  >  **(Model-View-Controller)**  >  **Avanti**. Nella versione 8,6 o successive selezionare **Web e applicazione console**  >  **app**  >  **Web (Model-View-Controller)**  >  **Avanti**.

  ![selezione del modello di app Web macOS](start-mvc/_static/web_app_template_vsmac.png)

* Nella finestra di dialogo **Configura nuova applicazione Web** :

  * Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.
  * Se viene visualizzata un'opzione per la selezione di un **Framework di destinazione** , selezionare la versione 5. x più recente.
  * Selezionare **Avanti**.

* Denominare il progetto **MvcMovie**, quindi selezionare **Crea**.

  ![macOS nome del progetto](start-mvc/_static/MvcMovie.png)

---

### <a name="run-the-app"></a>Eseguire l'app

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Premere CTRL + F5 per eseguire l'app senza il debugger.

  [!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio:

  * Avvia [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview).
  * Esegue l'app.

  La barra degli indirizzi visualizza `localhost:port#` e non `example.com` o simili. Il nome host standard per il computer locale è `localhost` . Quando Visual Studio crea un progetto Web, viene usata una porta casuale per il server Web.

L'avvio dell'app senza debug selezionando CTRL + F5 consente di:

* Apportare modifiche al codice.
* Salvare il file.
* Aggiornare rapidamente il browser e visualizzare le modifiche al codice.

È possibile scegliere se avviare l'app in modalità di debug o non di debug nella voce di menu **Debug**:

![Menu Debug](start-mvc/_static/debug_menu50.png)

È possibile eseguire il debug dell'app toccando il pulsante **IIS Express**.

![IIS Express](start-mvc/_static/iis_express50.png)

La figura seguente mostra l'app:

![Pagina Home o di indice](start-mvc/_static/home50-vs.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Premere CTRL + F5 per eseguire senza il debugger.

  [!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code:

  * Avvia [gheppio](xref:fundamentals/servers/kestrel)
  * Avvia un browser.
  * Passa a `https://localhost:5001` .

  La barra degli indirizzi visualizza `localhost:port:5001` e non `example.com` o simili. Il nome host standard per il computer locale è `localhost` . Localhost viene usato solo per le richieste web del computer locale.

L'avvio dell'app senza debug selezionando CTRL + F5 consente di:

* Apportare modifiche al codice.
* Salvare il file.
* Aggiornare rapidamente il browser e visualizzare le modifiche al codice.

  ![Pagina Home o di indice](start-mvc/_static/home50-port5001.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Per avviare l'app, selezionare **Esegui** > **Avvia senza eseguire debug**.

  Visual Studio per Mac:

  * Avvia il server [gheppio](xref:fundamentals/servers/index#kestrel) .
  * Avvia un browser.
  * Passa a `http://localhost:port` , dove *porta* è un numero di porta scelto in modo casuale.

  [!INCLUDE[](~/includes/trustCertMac.md)]

  La barra degli indirizzi visualizza `localhost:port#` e non `example.com` o simili. Il nome host standard per il computer locale è `localhost` . Quando Visual Studio crea un progetto Web, viene usata una porta casuale per il server Web.

È possibile scegliere se avviare l'app in modalità di debug o non di dal menu **Esegui**.

La figura seguente mostra l'app:

![Pagina Home o di indice](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

Nella parte seguente di questa esercitazione vengono fornite informazioni su MVC e istruzioni per iniziare a creare codice.

> [!div class="step-by-step"]
> [Passaggio successivo: aggiungere un controller](adding-controller.md)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

Questa è la prima esercitazione di una serie che insegna ASP.NET Core lo sviluppo Web MVC con i controller e le visualizzazioni.

Alla fine della serie, si disporrà di un'app che gestisce e Visualizza i dati dei film. Si apprenderà come:

> [!div class="checklist"]
> * Creare un'app Web.
> * Aggiungere un modello ed eseguirne lo scaffolding.
> * Usare un database.
> * Aggiungere ricerca e convalida.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/first-mvc-app/start-mvc/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-web-app"></a>Creare un'app Web

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* In Visual Studio selezionare **Crea un nuovo progetto**.

* Selezionare **ASP.NET Core applicazione Web** > **Avanti**.

  ![Creare un nuovo progetto di applicazione Web di ASP.NET Core](start-mvc/_static/np_2.1.png)

* Assegnare al progetto il nome **MvcMovie** e selezionare **Crea**. È importante assegnare al progetto il nome **MvcMovie**, in modo che quando si copia il codice lo spazio dei nomi corrisponda.

  ![Configurare il nuovo progetto](start-mvc/_static/config.png)

* Selezionare **applicazione Web (Model-View-Controller)**. Nelle caselle a discesa selezionare **.NET Core** e **ASP.NET Core 3,1**, quindi selezionare **Crea**.

  ![Finestra di dialogo Nuovo progetto, .NET Core nel riquadro sinistro, Web ASP.NET Core ](start-mvc/_static/new_project30.png)

Visual Studio ha usato il modello di progetto predefinito per il progetto MVC creato. Il progetto creato:

* È un'app funzionante.
* È un progetto iniziale di base.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

L'esercitazione presuppone una certa familiarità con VS Code. Per ulteriori informazioni, vedere la pagina relativa all'introduzione [a vs code](https://code.visualstudio.com/docs) e [Visual Studio Code Guida](#visual-studio-code-help).

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Modificare le directory ( `cd` ) in una cartella che conterrà il progetto.
* Eseguire il comando seguente:

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * Viene visualizzata una finestra di dialogo con le **risorse necessarie per la compilazione e il debug non è presente in ' MvcMovie '. Aggiungerli?** selezionare **Sì**.

  * `dotnet new mvc -o MvcMovie`: Crea un nuovo progetto MVC ASP.NET Core nella cartella *MvcMovie* .
  * `code -r MvcMovie`: Carica il file di progetto *MvcMovie. csproj* in Visual Studio Code.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Selezionare **File** > **Nuova soluzione**.

  ![Nuova soluzione macOS](start-mvc/_static/new_project_vsmac.png)

* In Visual Studio per Mac precedente alla versione 8,6, selezionare applicazione Web **.NET Core**  >  **app**  >  **(Model-View-Controller)**  >  **Avanti**. Nella versione 8,6 o successive selezionare **Web e applicazione console**  >  **app**  >  **Web (Model-View-Controller)**  >  **Avanti**.

  ![selezione del modello di app Web macOS](start-mvc/_static/web_app_template_vsmac.png)

* Nella finestra di dialogo **Configura nuova applicazione Web** :

  * Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.
  * Se viene visualizzata un'opzione per la selezione di un **Framework di destinazione** , selezionare la versione 3. x più recente.
  * Selezionare **Avanti**.

* Denominare il progetto **MvcMovie**, quindi selezionare **Crea**.

  ![macOS nome del progetto](start-mvc/_static/MvcMovie.png)

---

### <a name="run-the-app"></a>Eseguire l'app

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Premere CTRL + F5 per eseguire l'app senza debug.

  [!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio:

  * Avvia [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview).
  * Esegue l'app.

  La barra degli indirizzi visualizza `localhost:port#` e non `example.com` o simili. Il nome host standard per il computer locale è `localhost` . Quando Visual Studio crea un progetto Web, viene usata una porta casuale per il server Web.

L'avvio dell'app senza debug selezionando CTRL + F5 consente di:

* Apportare modifiche al codice.
* Salvare il file.
* Aggiornare rapidamente il browser e visualizzare le modifiche al codice.

È possibile scegliere se avviare l'app in modalità di debug o non di debug nella voce di menu **Debug**:

![Menu Debug](start-mvc/_static/debug_menu.png)

È possibile eseguire il debug dell'app toccando il pulsante **IIS Express**.

![IIS Express](start-mvc/_static/iis_express.png)

La figura seguente mostra l'app:

![Pagina Home o di indice](start-mvc/_static/home2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Premere CTRL + F5 per eseguire l'app senza debug.

  [!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code:

  * Avvia [gheppio](xref:fundamentals/servers/kestrel)
  * Avvia un browser.
  * Passa a `https://localhost:5001` .

  La barra degli indirizzi visualizza `localhost:port:5001` e non `example.com` o simili. Il nome host standard per il computer locale è `localhost` . Localhost viene usato solo per le richieste web del computer locale.

L'avvio dell'app senza debug selezionando CTRL + F5 consente di:

* Apportare modifiche al codice.
* Salvare il file.
* Aggiornare rapidamente il browser e visualizzare le modifiche al codice.

  ![Pagina Home o di indice](start-mvc/_static/home2.2.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Per avviare l'app, selezionare **Esegui** > **Avvia senza eseguire debug**.

  Visual Studio per Mac: avvia il server [gheppio](xref:fundamentals/servers/index#kestrel) , avvia un browser e passa a `http://localhost:port` , dove *porta* è un numero di porta scelto in modo casuale.

[!INCLUDE[](~/includes/trustCertMac.md)]

La barra degli indirizzi visualizza `localhost:port#` e non `example.com` o simili. Il nome host standard per il computer locale è `localhost` . Quando Visual Studio crea un progetto Web, viene usata una porta casuale per il server Web. Quando si esegue l'app verrà visualizzato un numero di porta diverso.

È possibile scegliere se avviare l'app in modalità di debug o non di dal menu **Esegui**.

La figura seguente mostra l'app:

![Pagina Home o di indice](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

Nella parte seguente di questa esercitazione vengono fornite informazioni su MVC e istruzioni per iniziare a creare codice.

> [!div class="step-by-step"]
> [Avanti](adding-controller.md)

::: moniker-end
