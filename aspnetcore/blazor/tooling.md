---
title: Strumenti per ASP.NET CoreBlazor
author: guardrex
description: Informazioni sugli strumenti disponibili per la compilazione di Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/07/2020
no-loc:
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/tooling
zone_pivot_groups: operating-systems
ms.openlocfilehash: 33245e669b317ed577a8a1652b2eed8f9ea5b915
ms.sourcegitcommit: 6fb27ea41a92f6d0e91dfd0eba905d2ac1a707f7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/15/2020
ms.locfileid: "86407645"
---
# <a name="tooling-for-aspnet-core-blazor"></a>Strumenti per ASP.NET CoreBlazor

Di [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)

::: zone pivot="os-windows"

1. Installare la versione più recente di [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) con il carico di lavoro **sviluppo di ASP.NET e Web** .

1. Creare un nuovo progetto.

1. Selezionare ** Blazor app**. Selezionare **Avanti**.

1. Specificare il nome di un progetto nel campo **Nome progetto** oppure accettare il nome predefinito. Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto. Selezionare **Crea**.

1. Per un' Blazor WebAssembly esperienza, scegliere il modello ** Blazor WebAssembly app** . Per un' Blazor Server esperienza, scegliere il modello ** Blazor Server app** . Selezionare **Crea**.

   Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* e *Blazor Server* , vedere <xref:blazor/hosting-models> .

1. Premere <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app.

Per ulteriori informazioni su come considerare attendibile il certificato di sviluppo HTTPS ASP.NET Core, vedere <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos> .

::: zone-end

::: zone pivot="os-linux"

1. Installare la versione più recente di [.NET Core 3,1 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1). Se in precedenza è stato installato l'SDK, è possibile determinare la versione installata eseguendo il comando seguente in una shell dei comandi:

   ```dotnetcli
   dotnet --version
   ```

1. Installare la versione più recente di [Visual Studio Code](https://code.visualstudio.com/).

1. Installare la versione più recente [di C# per Visual Studio Code estensione](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).

1. Per un' Blazor WebAssembly esperienza, eseguire il comando seguente in una shell dei comandi:

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   ```

   Per un' Blazor Server esperienza, eseguire il comando seguente in una shell dei comandi:

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   ```

   Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* e *Blazor Server* , vedere <xref:blazor/hosting-models> .

1. Aprire la cartella `WebApplication1` in Visual Studio Code.

1. Le richieste dell'IDE aggiungono risorse per compilare ed eseguire il debug del progetto. Selezionare **Sì**.

1. Premere <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app.

## <a name="trust-a-development-certificate"></a>Attendibilità di un certificato di sviluppo

Non esiste un modo centralizzato per considerare attendibile un certificato in Linux. Viene in genere adottato uno degli approcci seguenti:

* Escludere l'URL dell'app nell'elenco di esclusione del browser.
* Considerare attendibili tutti i certificati autofirmati per `localhost` .
* Aggiungere il certificato all'elenco dei certificati attendibili nel browser.

Per altre informazioni, vedere le linee guida fornite dal browser e dalla distribuzione Linux.

::: zone-end

::: zone pivot="os-macos"

1. Installare [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/).

1. Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.

1. Nella barra laterale selezionare app **Web e console**  >  **App**.

   Per un' Blazor WebAssembly esperienza, scegliere il modello ** Blazor WebAssembly app** . Per un' Blazor Server esperienza, scegliere il modello ** Blazor Server app** . Selezionare **Avanti**.

   Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* e *Blazor Server* , vedere <xref:blazor/hosting-models> .

1. Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**. Selezionare **Avanti**.

1. Nel campo **nome progetto** assegnare un nome all'app `WebApplication1` . Selezionare **Crea**.

1. Selezionare **Esegui**  >  **Avvia senza eseguire debug** per eseguire l'app *senza il debugger*. Eseguire l'app con **Esegui**  >  **debug Avvia debug** o il pulsante Esegui (&#9654;) per eseguire l'app *con il debugger*.

Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare. Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain. Per ulteriori informazioni su come considerare attendibile il certificato di sviluppo HTTPS ASP.NET Core, vedere <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos> .

::: zone-end