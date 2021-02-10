---
title: Introduzione a ASP.NET Core Blazor
author: guardrex
description: Esplora ASP.NET Core Blazor , un modo per creare un'interfaccia utente Web interattiva sul lato client con .NET in un'app ASP.NET Core.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, seoapril2019, devx-track-js
ms.date: 09/25/2020
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
uid: blazor/index
ms.openlocfilehash: d42136de00ccc9b7565c5ae088e1ecf4560a63c5
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100106882"
---
# <a name="introduction-to-aspnet-core-blazor"></a>Introduzione a ASP.NET Core Blazor

Di [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)

*Benvenuti in Blazor .*

Blazor è un Framework per la creazione di un'interfaccia utente Web interattiva sul lato client con [.NET](/dotnet/standard/tour):

* Creare interfacce utente interattive avanzate usando [C#](/dotnet/csharp/) anziché [JavaScript](https://www.javascript.com).
* Condividere la logica dell'app scritta in .NET sul lato client e sul lato server.
* Eseguire il rendering dell'interfaccia utente come HTML e CSS per un ampio supporto dei browser, inclusi i browser per dispositivi mobili.
* Integrarsi con piattaforme di hosting moderne, ad esempio [Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index).

L'uso di .NET per lo sviluppo Web lato client offre i vantaggi seguenti:

* scrivere codice in C# invece che in JavaScript.
* Sfruttare l'ecosistema .NET esistente delle [librerie .NET](/dotnet/standard/class-libraries).
* Permette di condividere la logica dell'app tra server e client.
* Permette di ottenere le prestazioni, l'affidabilità e la sicurezza di .NET.
* È sempre più produttivo con [Visual Studio](https://visualstudio.microsoft.com) in Windows, Linux e MacOS.
* basato su un set comune di linguaggi, framework e strumenti che sono stabili, ricchi di funzionalità e facili da usare.

## <a name="components"></a>Componenti

Blazor le app sono basate su *componenti*. Un componente Blazor di è un elemento dell'interfaccia utente, ad esempio una pagina, una finestra di dialogo o un form di immissione dati.

I componenti sono classi .NET C# compilate in [assembly .NET](/dotnet/standard/assembly/) che:

* Definiscono la logica di rendering dell'interfaccia utente flessibile.
* Gestiscono gli eventi utente.
* Possono essere annidati e riutilizzati.
* Può essere condiviso e distribuito come [ Razor librerie di classi](xref:razor-pages/ui-class) o [pacchetti NuGet](/nuget/what-is-nuget).

La classe Component viene in genere scritta sotto forma di [Razor](xref:mvc/views/razor) pagina di markup con `.razor` estensione di file. I componenti di Blazor sono definiti formalmente come *Razor componenti* di. Razor è una sintassi per combinare il markup HTML con il codice C# progettato per la produttività degli sviluppatori. Razor consente di passare dal markup HTML al linguaggio C# e viceversa nello stesso file con supporto per la programmazione [IntelliSense](/visualstudio/ide/using-intellisense) in Visual Studio. Razor Anche le pagine e MVC usano Razor . Diversamente dalle Razor pagine e da MVC, che sono basate su un modello di richiesta/risposta, i componenti vengono usati in modo specifico per la composizione e la logica dell'interfaccia utente lato client.

Blazor Usa tag HTML naturali per la composizione dell'interfaccia utente. Il Razor markup seguente illustra un componente ( `Dialog.razor` ) che visualizza una finestra di dialogo e elabora un evento quando l'utente seleziona un pulsante:

```razor
<div class="card" style="width:22rem">
    <div class="card-body">
        <h3 class="card-title">@Title</h3>
        <p class="card-text">@ChildContent</p>
        <button @onclick="OnYes">Yes!</button>
    </div>
</div>

@code {
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    [Parameter]
    public string Title { get; set; }

    private void OnYes()
    {
        Console.WriteLine("Write to the console in C#! 'Yes' button selected.");
    }
}
```

Nell'esempio precedente, `OnYes` è un metodo C# attivato dall'evento del pulsante `onclick` . Il testo ( `ChildContent` ) e il titolo () della finestra di dialogo `Title` vengono forniti dal componente seguente che usa questo componente nell'interfaccia utente.

Il `Dialog` componente è annidato all'interno di un altro componente usando un tag HTML. Nell'esempio seguente, il `Index` componente ( `Pages/Index.razor` ) usa il componente precedente `Dialog` . L'attributo del tag `Title` passa un valore per il titolo alla `Dialog` proprietà del componente `Title` .  Il `Dialog` testo () del componente `ChildContent` viene impostato dal contenuto dell' `<Dialog>` elemento. Quando il `Dialog` componente viene aggiunto al `Index` componente, [IntelliSense in Visual Studio](/visualstudio/ide/using-intellisense) accelera lo sviluppo con la sintassi e il completamento dei parametri.

```razor
@page "/"

<h1>Hello, world!</h1>

<p>
    Welcome to your new app.
</p>

<Dialog Title="Learn More">
    Do you want to <i>learn more</i> about Blazor?
</Dialog>
```

Il rendering della finestra di dialogo viene eseguito quando `Index` si accede al componente in un browser. Quando il pulsante è selezionato dall'utente, la console degli strumenti di sviluppo del browser Mostra il messaggio scritto dal `OnYes` Metodo:

![Componente della finestra di dialogo sottoposto a rendering nel browser annidato all'interno del componente dell'indice. La console degli strumenti di sviluppo del browser Mostra il messaggio scritto dal codice C# quando l'utente seleziona l'impostazione Sì! nell'interfaccia utente.](index/_static/dialog.png)

I componenti eseguono il rendering in una rappresentazione in memoria del Document Object Model del browser [(Dom)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) denominato *albero di rendering*, che viene usato per aggiornare l'interfaccia utente in modo flessibile ed efficiente.

## Blazor WebAssembly

Blazor WebAssembly è un [Framework di app a singola pagina (Spa)](/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps) per la creazione di app Web interattive sul lato client con .NET. Blazor WebAssembly USA gli standard Web aperti senza plug-in o ricompilando il codice in altri linguaggi. Blazor WebAssembly funziona in tutti i Web browser moderni, inclusi i browser per dispositivi mobili.

Il codice .NET in esecuzione all'interno di Web browser è reso possibile dal [webassembly](https://webassembly.org) (abbreviato `wasm` ). WebAssembly è un formato bytecode compatto ottimizzato per il download veloce e la velocità massima di esecuzione. WebAssembly è un standard Web aperto ed è supportato nei Web browser senza plug-in.

Il codice webassembly può accedere alle funzionalità complete del browser tramite JavaScript, denominato *interoperabilità JavaScript*, spesso abbreviato in interoperabilità di *JavaScript* o di interoperabilità *JS*. Il codice .NET eseguito tramite WebAssembly nel browser viene eseguito nella sandbox JavaScript del browser con le misure di sicurezza offerte dalla sandbox per la protezione da azioni dannose nel computer client.

![::: NO-LOC (webassembly Blazer)::: esegue il codice .NET nel browser con webassembly.](index/_static/blazor-webassembly.png)

Quando un' Blazor WebAssembly app viene compilata ed eseguita in un browser:

* I file e i file di codice C# Razor vengono compilati in assembly .NET.
* Gli assembly e il [Runtime .NET](/dotnet/framework/get-started/overview) vengono scaricati nel browser.
* Blazor WebAssembly avvia il Runtime .NET e configura il runtime per caricare gli assembly per l'app. Il Blazor WebAssembly Runtime usa l'interoperabilità JavaScript per gestire la manipolazione Dom e le chiamate API del browser.

La dimensione dell'app pubblicata, le *dimensioni del payload*, rappresenta un fattore di prestazioni cruciale per l'usabilità di un'app. Un'app di grandi dimensioni impiega relativamente molto tempo a essere scaricata in un browser, influendo negativamente sull'esperienza utente. Blazor WebAssembly Ottimizza le dimensioni del payload per ridurre i tempi di download:

::: moniker range=">= aspnetcore-5.0"

* Il codice inutilizzato viene rimosso dall'app quando viene pubblicato dal [trimmer del linguaggio intermedio (il)](xref:blazor/host-and-deploy/configure-trimmer).
* Le risposte HTTP vengono compresse.
* Il runtime e gli assembly .NET vengono memorizzati nella cache nel browser.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* Il codice non usato viene rimosso dall'app quando questa viene pubblicata dal [linker del linguaggio intermedio](xref:blazor/host-and-deploy/configure-linker).
* Le risposte HTTP vengono compresse.
* Il runtime e gli assembly .NET vengono memorizzati nella cache nel browser.

::: moniker-end

## Blazor Server

Blazor separa la logica di rendering dei componenti dal modo in cui vengono applicati gli aggiornamenti dell'interfaccia utente. *Blazor Server* fornisce supporto per l'hosting di Razor componenti nel server in un'app ASP.NET Core. Gli aggiornamenti dell'interfaccia utente vengono gestiti tramite una [SignalR](xref:signalr/introduction) connessione.

Il runtime rimane sul server e gestisce:

* Esecuzione del codice C# dell'app.
* Invio di eventi dell'interfaccia utente dal browser al server.
* Applicazione degli aggiornamenti dell'interfaccia utente al componente di cui è stato eseguito il rendering che vengono restituiti dal server.

La connessione utilizzata da Blazor Server per comunicare con il browser viene utilizzata anche per gestire le chiamate di interoperabilità JavaScript.

![::: NO-LOC (server Blazer)::: esegue il codice .NET sul server e interagisce con il Document Object Model sul client su un oggetto::: NO-LOC (SignalR)::: Connection](index/_static/blazor-server.png)

## <a name="javascript-interop"></a>Interoperabilità JavaScript

Per le app che richiedono librerie JavaScript di terze parti e l'accesso alle API del browser, i componenti supportano l'interoperabilità con JavaScript. I componenti sono in grado di usare qualsiasi libreria o API supportata da JavaScript. Il codice c# può [chiamare il codice JavaScript](xref:blazor/call-javascript-from-dotnet)e il codice JavaScript può chiamare il codice [c#](xref:blazor/call-dotnet-from-javascript).

## <a name="code-sharing-and-net-standard"></a>Condivisione del codice e .NET Standard

Blazor implementa l' [.NET standard](/dotnet/standard/net-standard), che consente Blazor ai progetti di fare riferimento a librerie conformi alle specifiche di .NET standard. .NET Standard è una specifica formale delle API .NET comuni tra le implementazioni di .NET. Le librerie di classi .NET Standard possono essere condivise tra diverse piattaforme .NET, ad esempio Blazor .NET Framework, .NET Core, Novell, mono e Unity.

Le API non valide all'interno di un Web browser (ad esempio per l'accesso al file system, l'apertura di un socket e la gestione dei thread) generano un'eccezione <xref:System.PlatformNotSupportedException>.

## <a name="additional-resources"></a>Risorse aggiuntive

* [WebAssembly](https://webassembly.org)
* <xref:blazor/hosting-models>
* <xref:tutorials/signalr-blazor>
* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>
* [Guida a C#](/dotnet/csharp/)
* [Razor informazioni di riferimento sulla sintassi per ASP.NET Core](xref:mvc/views/razor)
* [HTML](https://www.w3.org/html/)
* [Awesome Blazor ](https://github.com/AdrienTorris/awesome-blazor) collegamenti della community
