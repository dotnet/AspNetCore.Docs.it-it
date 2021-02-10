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
# <a name="introduction-to-aspnet-core-blazor"></a><span data-ttu-id="21c4a-103">Introduzione a ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="21c4a-103">Introduction to ASP.NET Core Blazor</span></span>

<span data-ttu-id="21c4a-104">Di [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="21c4a-104">By [Daniel Roth](https://github.com/danroth27) and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="21c4a-105">*Benvenuti in Blazor .*</span><span class="sxs-lookup"><span data-stu-id="21c4a-105">*Welcome to Blazor!*</span></span>

<span data-ttu-id="21c4a-106">Blazor è un Framework per la creazione di un'interfaccia utente Web interattiva sul lato client con [.NET](/dotnet/standard/tour):</span><span class="sxs-lookup"><span data-stu-id="21c4a-106">Blazor is a framework for building interactive client-side web UI with [.NET](/dotnet/standard/tour):</span></span>

* <span data-ttu-id="21c4a-107">Creare interfacce utente interattive avanzate usando [C#](/dotnet/csharp/) anziché [JavaScript](https://www.javascript.com).</span><span class="sxs-lookup"><span data-stu-id="21c4a-107">Create rich interactive UIs using [C#](/dotnet/csharp/) instead of [JavaScript](https://www.javascript.com).</span></span>
* <span data-ttu-id="21c4a-108">Condividere la logica dell'app scritta in .NET sul lato client e sul lato server.</span><span class="sxs-lookup"><span data-stu-id="21c4a-108">Share server-side and client-side app logic written in .NET.</span></span>
* <span data-ttu-id="21c4a-109">Eseguire il rendering dell'interfaccia utente come HTML e CSS per un ampio supporto dei browser, inclusi i browser per dispositivi mobili.</span><span class="sxs-lookup"><span data-stu-id="21c4a-109">Render the UI as HTML and CSS for wide browser support, including mobile browsers.</span></span>
* <span data-ttu-id="21c4a-110">Integrarsi con piattaforme di hosting moderne, ad esempio [Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index).</span><span class="sxs-lookup"><span data-stu-id="21c4a-110">Integrate with modern hosting platforms, such as [Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index).</span></span>

<span data-ttu-id="21c4a-111">L'uso di .NET per lo sviluppo Web lato client offre i vantaggi seguenti:</span><span class="sxs-lookup"><span data-stu-id="21c4a-111">Using .NET for client-side web development offers the following advantages:</span></span>

* <span data-ttu-id="21c4a-112">scrivere codice in C# invece che in JavaScript.</span><span class="sxs-lookup"><span data-stu-id="21c4a-112">Write code in C# instead of JavaScript.</span></span>
* <span data-ttu-id="21c4a-113">Sfruttare l'ecosistema .NET esistente delle [librerie .NET](/dotnet/standard/class-libraries).</span><span class="sxs-lookup"><span data-stu-id="21c4a-113">Leverage the existing .NET ecosystem of [.NET libraries](/dotnet/standard/class-libraries).</span></span>
* <span data-ttu-id="21c4a-114">Permette di condividere la logica dell'app tra server e client.</span><span class="sxs-lookup"><span data-stu-id="21c4a-114">Share app logic across server and client.</span></span>
* <span data-ttu-id="21c4a-115">Permette di ottenere le prestazioni, l'affidabilità e la sicurezza di .NET.</span><span class="sxs-lookup"><span data-stu-id="21c4a-115">Benefit from .NET's performance, reliability, and security.</span></span>
* <span data-ttu-id="21c4a-116">È sempre più produttivo con [Visual Studio](https://visualstudio.microsoft.com) in Windows, Linux e MacOS.</span><span class="sxs-lookup"><span data-stu-id="21c4a-116">Stay productive with [Visual Studio](https://visualstudio.microsoft.com) on Windows, Linux, and macOS.</span></span>
* <span data-ttu-id="21c4a-117">basato su un set comune di linguaggi, framework e strumenti che sono stabili, ricchi di funzionalità e facili da usare.</span><span class="sxs-lookup"><span data-stu-id="21c4a-117">Build on a common set of languages, frameworks, and tools that are stable, feature-rich, and easy to use.</span></span>

## <a name="components"></a><span data-ttu-id="21c4a-118">Componenti</span><span class="sxs-lookup"><span data-stu-id="21c4a-118">Components</span></span>

<span data-ttu-id="21c4a-119">Blazor le app sono basate su *componenti*.</span><span class="sxs-lookup"><span data-stu-id="21c4a-119">Blazor apps are based on *components*.</span></span> <span data-ttu-id="21c4a-120">Un componente Blazor di è un elemento dell'interfaccia utente, ad esempio una pagina, una finestra di dialogo o un form di immissione dati.</span><span class="sxs-lookup"><span data-stu-id="21c4a-120">A component in Blazor is an element of UI, such as a page, dialog, or data entry form.</span></span>

<span data-ttu-id="21c4a-121">I componenti sono classi .NET C# compilate in [assembly .NET](/dotnet/standard/assembly/) che:</span><span class="sxs-lookup"><span data-stu-id="21c4a-121">Components are .NET C# classes built into [.NET assemblies](/dotnet/standard/assembly/) that:</span></span>

* <span data-ttu-id="21c4a-122">Definiscono la logica di rendering dell'interfaccia utente flessibile.</span><span class="sxs-lookup"><span data-stu-id="21c4a-122">Define flexible UI rendering logic.</span></span>
* <span data-ttu-id="21c4a-123">Gestiscono gli eventi utente.</span><span class="sxs-lookup"><span data-stu-id="21c4a-123">Handle user events.</span></span>
* <span data-ttu-id="21c4a-124">Possono essere annidati e riutilizzati.</span><span class="sxs-lookup"><span data-stu-id="21c4a-124">Can be nested and reused.</span></span>
* <span data-ttu-id="21c4a-125">Può essere condiviso e distribuito come [ Razor librerie di classi](xref:razor-pages/ui-class) o [pacchetti NuGet](/nuget/what-is-nuget).</span><span class="sxs-lookup"><span data-stu-id="21c4a-125">Can be shared and distributed as [Razor class libraries](xref:razor-pages/ui-class) or [NuGet packages](/nuget/what-is-nuget).</span></span>

<span data-ttu-id="21c4a-126">La classe Component viene in genere scritta sotto forma di [Razor](xref:mvc/views/razor) pagina di markup con `.razor` estensione di file.</span><span class="sxs-lookup"><span data-stu-id="21c4a-126">The component class is usually written in the form of a [Razor](xref:mvc/views/razor) markup page with a `.razor` file extension.</span></span> <span data-ttu-id="21c4a-127">I componenti di Blazor sono definiti formalmente come *Razor componenti* di.</span><span class="sxs-lookup"><span data-stu-id="21c4a-127">Components in Blazor are formally referred to as *Razor components*.</span></span> <span data-ttu-id="21c4a-128">Razor è una sintassi per combinare il markup HTML con il codice C# progettato per la produttività degli sviluppatori.</span><span class="sxs-lookup"><span data-stu-id="21c4a-128">Razor is a syntax for combining HTML markup with C# code designed for developer productivity.</span></span> <span data-ttu-id="21c4a-129">Razor consente di passare dal markup HTML al linguaggio C# e viceversa nello stesso file con supporto per la programmazione [IntelliSense](/visualstudio/ide/using-intellisense) in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="21c4a-129">Razor allows you to switch between HTML markup and C# in the same file with [IntelliSense](/visualstudio/ide/using-intellisense) programming support in Visual Studio.</span></span> <span data-ttu-id="21c4a-130">Razor Anche le pagine e MVC usano Razor .</span><span class="sxs-lookup"><span data-stu-id="21c4a-130">Razor Pages and MVC also use Razor.</span></span> <span data-ttu-id="21c4a-131">Diversamente dalle Razor pagine e da MVC, che sono basate su un modello di richiesta/risposta, i componenti vengono usati in modo specifico per la composizione e la logica dell'interfaccia utente lato client.</span><span class="sxs-lookup"><span data-stu-id="21c4a-131">Unlike Razor Pages and MVC, which are built around a request/response model, components are used specifically for client-side UI logic and composition.</span></span>

<span data-ttu-id="21c4a-132">Blazor Usa tag HTML naturali per la composizione dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="21c4a-132">Blazor uses natural HTML tags for UI composition.</span></span> <span data-ttu-id="21c4a-133">Il Razor markup seguente illustra un componente ( `Dialog.razor` ) che visualizza una finestra di dialogo e elabora un evento quando l'utente seleziona un pulsante:</span><span class="sxs-lookup"><span data-stu-id="21c4a-133">The following Razor markup demonstrates a component (`Dialog.razor`) that displays a dialog and processes an event when the user selects a button:</span></span>

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

<span data-ttu-id="21c4a-134">Nell'esempio precedente, `OnYes` è un metodo C# attivato dall'evento del pulsante `onclick` .</span><span class="sxs-lookup"><span data-stu-id="21c4a-134">In the preceding example, `OnYes` is a C# method triggered by the button's `onclick` event.</span></span> <span data-ttu-id="21c4a-135">Il testo ( `ChildContent` ) e il titolo () della finestra di dialogo `Title` vengono forniti dal componente seguente che usa questo componente nell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="21c4a-135">The dialog's text (`ChildContent`) and title (`Title`) are provided by the following component that uses this component in its UI.</span></span>

<span data-ttu-id="21c4a-136">Il `Dialog` componente è annidato all'interno di un altro componente usando un tag HTML.</span><span class="sxs-lookup"><span data-stu-id="21c4a-136">The `Dialog` component is nested within another component using an HTML tag.</span></span> <span data-ttu-id="21c4a-137">Nell'esempio seguente, il `Index` componente ( `Pages/Index.razor` ) usa il componente precedente `Dialog` .</span><span class="sxs-lookup"><span data-stu-id="21c4a-137">In the following example, the `Index` component (`Pages/Index.razor`) uses the preceding `Dialog` component.</span></span> <span data-ttu-id="21c4a-138">L'attributo del tag `Title` passa un valore per il titolo alla `Dialog` proprietà del componente `Title` .</span><span class="sxs-lookup"><span data-stu-id="21c4a-138">The tag's `Title` attribute passes a value for the title to the `Dialog` component's `Title` property.</span></span>  <span data-ttu-id="21c4a-139">Il `Dialog` testo () del componente `ChildContent` viene impostato dal contenuto dell' `<Dialog>` elemento.</span><span class="sxs-lookup"><span data-stu-id="21c4a-139">The `Dialog` component's text (`ChildContent`) are set by the content of the `<Dialog>` element.</span></span> <span data-ttu-id="21c4a-140">Quando il `Dialog` componente viene aggiunto al `Index` componente, [IntelliSense in Visual Studio](/visualstudio/ide/using-intellisense) accelera lo sviluppo con la sintassi e il completamento dei parametri.</span><span class="sxs-lookup"><span data-stu-id="21c4a-140">When the `Dialog` component is added to the `Index` component, [IntelliSense in Visual Studio](/visualstudio/ide/using-intellisense) speeds development with syntax and parameter completion.</span></span>

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

<span data-ttu-id="21c4a-141">Il rendering della finestra di dialogo viene eseguito quando `Index` si accede al componente in un browser.</span><span class="sxs-lookup"><span data-stu-id="21c4a-141">The dialog is rendered when the `Index` component is accessed in a browser.</span></span> <span data-ttu-id="21c4a-142">Quando il pulsante è selezionato dall'utente, la console degli strumenti di sviluppo del browser Mostra il messaggio scritto dal `OnYes` Metodo:</span><span class="sxs-lookup"><span data-stu-id="21c4a-142">When the button is selected by the user, the browser's developer tools console shows the message written by the `OnYes` method:</span></span>

![Componente della finestra di dialogo sottoposto a rendering nel browser annidato all'interno del componente dell'indice.](index/_static/dialog.png)

<span data-ttu-id="21c4a-146">I componenti eseguono il rendering in una rappresentazione in memoria del Document Object Model del browser [(Dom)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) denominato *albero di rendering*, che viene usato per aggiornare l'interfaccia utente in modo flessibile ed efficiente.</span><span class="sxs-lookup"><span data-stu-id="21c4a-146">Components render into an in-memory representation of the browser's [Document Object Model (DOM)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) called a *render tree*, which is used to update the UI in a flexible and efficient way.</span></span>

## Blazor WebAssembly

<span data-ttu-id="21c4a-147">Blazor WebAssembly è un [Framework di app a singola pagina (Spa)](/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps) per la creazione di app Web interattive sul lato client con .NET.</span><span class="sxs-lookup"><span data-stu-id="21c4a-147">Blazor WebAssembly is a [single-page app (SPA) framework](/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps) for building interactive client-side web apps with .NET.</span></span> <span data-ttu-id="21c4a-148">Blazor WebAssembly USA gli standard Web aperti senza plug-in o ricompilando il codice in altri linguaggi.</span><span class="sxs-lookup"><span data-stu-id="21c4a-148">Blazor WebAssembly uses open web standards without plugins or recompiling code into other languages.</span></span> <span data-ttu-id="21c4a-149">Blazor WebAssembly funziona in tutti i Web browser moderni, inclusi i browser per dispositivi mobili.</span><span class="sxs-lookup"><span data-stu-id="21c4a-149">Blazor WebAssembly works in all modern web browsers, including mobile browsers.</span></span>

<span data-ttu-id="21c4a-150">Il codice .NET in esecuzione all'interno di Web browser è reso possibile dal [webassembly](https://webassembly.org) (abbreviato `wasm` ).</span><span class="sxs-lookup"><span data-stu-id="21c4a-150">Running .NET code inside web browsers is made possible by [WebAssembly](https://webassembly.org) (abbreviated `wasm`).</span></span> <span data-ttu-id="21c4a-151">WebAssembly è un formato bytecode compatto ottimizzato per il download veloce e la velocità massima di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="21c4a-151">WebAssembly is a compact bytecode format optimized for fast download and maximum execution speed.</span></span> <span data-ttu-id="21c4a-152">WebAssembly è un standard Web aperto ed è supportato nei Web browser senza plug-in.</span><span class="sxs-lookup"><span data-stu-id="21c4a-152">WebAssembly is an open web standard and supported in web browsers without plugins.</span></span>

<span data-ttu-id="21c4a-153">Il codice webassembly può accedere alle funzionalità complete del browser tramite JavaScript, denominato *interoperabilità JavaScript*, spesso abbreviato in interoperabilità di *JavaScript* o di interoperabilità *JS*.</span><span class="sxs-lookup"><span data-stu-id="21c4a-153">WebAssembly code can access the full functionality of the browser via JavaScript, called *JavaScript interoperability*, often shortened to *JavaScript interop* or *JS interop*.</span></span> <span data-ttu-id="21c4a-154">Il codice .NET eseguito tramite WebAssembly nel browser viene eseguito nella sandbox JavaScript del browser con le misure di sicurezza offerte dalla sandbox per la protezione da azioni dannose nel computer client.</span><span class="sxs-lookup"><span data-stu-id="21c4a-154">.NET code executed via WebAssembly in the browser runs in the browser's JavaScript sandbox with the protections that the sandbox provides against malicious actions on the client machine.</span></span>

![::: NO-LOC (webassembly Blazer)::: esegue il codice .NET nel browser con webassembly.](index/_static/blazor-webassembly.png)

<span data-ttu-id="21c4a-156">Quando un' Blazor WebAssembly app viene compilata ed eseguita in un browser:</span><span class="sxs-lookup"><span data-stu-id="21c4a-156">When a Blazor WebAssembly app is built and run in a browser:</span></span>

* <span data-ttu-id="21c4a-157">I file e i file di codice C# Razor vengono compilati in assembly .NET.</span><span class="sxs-lookup"><span data-stu-id="21c4a-157">C# code files and Razor files are compiled into .NET assemblies.</span></span>
* <span data-ttu-id="21c4a-158">Gli assembly e il [Runtime .NET](/dotnet/framework/get-started/overview) vengono scaricati nel browser.</span><span class="sxs-lookup"><span data-stu-id="21c4a-158">The assemblies and the [.NET runtime](/dotnet/framework/get-started/overview) are downloaded to the browser.</span></span>
* <span data-ttu-id="21c4a-159">Blazor WebAssembly avvia il Runtime .NET e configura il runtime per caricare gli assembly per l'app.</span><span class="sxs-lookup"><span data-stu-id="21c4a-159">Blazor WebAssembly bootstraps the .NET runtime and configures the runtime to load the assemblies for the app.</span></span> <span data-ttu-id="21c4a-160">Il Blazor WebAssembly Runtime usa l'interoperabilità JavaScript per gestire la manipolazione Dom e le chiamate API del browser.</span><span class="sxs-lookup"><span data-stu-id="21c4a-160">The Blazor WebAssembly runtime uses JavaScript interop to handle DOM manipulation and browser API calls.</span></span>

<span data-ttu-id="21c4a-161">La dimensione dell'app pubblicata, le *dimensioni del payload*, rappresenta un fattore di prestazioni cruciale per l'usabilità di un'app.</span><span class="sxs-lookup"><span data-stu-id="21c4a-161">The size of the published app, its *payload size*, is a critical performance factor for an app's usability.</span></span> <span data-ttu-id="21c4a-162">Un'app di grandi dimensioni impiega relativamente molto tempo a essere scaricata in un browser, influendo negativamente sull'esperienza utente.</span><span class="sxs-lookup"><span data-stu-id="21c4a-162">A large app takes a relatively long time to download to a browser, which diminishes the user experience.</span></span> <span data-ttu-id="21c4a-163">Blazor WebAssembly Ottimizza le dimensioni del payload per ridurre i tempi di download:</span><span class="sxs-lookup"><span data-stu-id="21c4a-163">Blazor WebAssembly optimizes payload size to reduce download times:</span></span>

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="21c4a-164">Il codice inutilizzato viene rimosso dall'app quando viene pubblicato dal [trimmer del linguaggio intermedio (il)](xref:blazor/host-and-deploy/configure-trimmer).</span><span class="sxs-lookup"><span data-stu-id="21c4a-164">Unused code is stripped out of the app when it's published by the [Intermediate Language (IL) Trimmer](xref:blazor/host-and-deploy/configure-trimmer).</span></span>
* <span data-ttu-id="21c4a-165">Le risposte HTTP vengono compresse.</span><span class="sxs-lookup"><span data-stu-id="21c4a-165">HTTP responses are compressed.</span></span>
* <span data-ttu-id="21c4a-166">Il runtime e gli assembly .NET vengono memorizzati nella cache nel browser.</span><span class="sxs-lookup"><span data-stu-id="21c4a-166">The .NET runtime and assemblies are cached in the browser.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <span data-ttu-id="21c4a-167">Il codice non usato viene rimosso dall'app quando questa viene pubblicata dal [linker del linguaggio intermedio](xref:blazor/host-and-deploy/configure-linker).</span><span class="sxs-lookup"><span data-stu-id="21c4a-167">Unused code is stripped out of the app when it's published by the [Intermediate Language (IL) Linker](xref:blazor/host-and-deploy/configure-linker).</span></span>
* <span data-ttu-id="21c4a-168">Le risposte HTTP vengono compresse.</span><span class="sxs-lookup"><span data-stu-id="21c4a-168">HTTP responses are compressed.</span></span>
* <span data-ttu-id="21c4a-169">Il runtime e gli assembly .NET vengono memorizzati nella cache nel browser.</span><span class="sxs-lookup"><span data-stu-id="21c4a-169">The .NET runtime and assemblies are cached in the browser.</span></span>

::: moniker-end

## Blazor Server

<span data-ttu-id="21c4a-170">Blazor separa la logica di rendering dei componenti dal modo in cui vengono applicati gli aggiornamenti dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="21c4a-170">Blazor decouples component rendering logic from how UI updates are applied.</span></span> <span data-ttu-id="21c4a-171">*Blazor Server* fornisce supporto per l'hosting di Razor componenti nel server in un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="21c4a-171">*Blazor Server* provides support for hosting Razor components on the server in an ASP.NET Core app.</span></span> <span data-ttu-id="21c4a-172">Gli aggiornamenti dell'interfaccia utente vengono gestiti tramite una [SignalR](xref:signalr/introduction) connessione.</span><span class="sxs-lookup"><span data-stu-id="21c4a-172">UI updates are handled over a [SignalR](xref:signalr/introduction) connection.</span></span>

<span data-ttu-id="21c4a-173">Il runtime rimane sul server e gestisce:</span><span class="sxs-lookup"><span data-stu-id="21c4a-173">The runtime stays on the server and handles:</span></span>

* <span data-ttu-id="21c4a-174">Esecuzione del codice C# dell'app.</span><span class="sxs-lookup"><span data-stu-id="21c4a-174">Executing the app's C# code.</span></span>
* <span data-ttu-id="21c4a-175">Invio di eventi dell'interfaccia utente dal browser al server.</span><span class="sxs-lookup"><span data-stu-id="21c4a-175">Sending UI events from the browser to the server.</span></span>
* <span data-ttu-id="21c4a-176">Applicazione degli aggiornamenti dell'interfaccia utente al componente di cui è stato eseguito il rendering che vengono restituiti dal server.</span><span class="sxs-lookup"><span data-stu-id="21c4a-176">Applying UI updates to the rendered component that are sent back by the server.</span></span>

<span data-ttu-id="21c4a-177">La connessione utilizzata da Blazor Server per comunicare con il browser viene utilizzata anche per gestire le chiamate di interoperabilità JavaScript.</span><span class="sxs-lookup"><span data-stu-id="21c4a-177">The connection used by Blazor Server to communicate with the browser is also used to handle JavaScript interop calls.</span></span>

![::: NO-LOC (server Blazer)::: esegue il codice .NET sul server e interagisce con il Document Object Model sul client su un oggetto::: NO-LOC (SignalR)::: Connection](index/_static/blazor-server.png)

## <a name="javascript-interop"></a><span data-ttu-id="21c4a-179">Interoperabilità JavaScript</span><span class="sxs-lookup"><span data-stu-id="21c4a-179">JavaScript interop</span></span>

<span data-ttu-id="21c4a-180">Per le app che richiedono librerie JavaScript di terze parti e l'accesso alle API del browser, i componenti supportano l'interoperabilità con JavaScript.</span><span class="sxs-lookup"><span data-stu-id="21c4a-180">For apps that require third-party JavaScript libraries and access to browser APIs, components interoperate with JavaScript.</span></span> <span data-ttu-id="21c4a-181">I componenti sono in grado di usare qualsiasi libreria o API supportata da JavaScript.</span><span class="sxs-lookup"><span data-stu-id="21c4a-181">Components are capable of using any library or API that JavaScript is able to use.</span></span> <span data-ttu-id="21c4a-182">Il codice c# può [chiamare il codice JavaScript](xref:blazor/call-javascript-from-dotnet)e il codice JavaScript può chiamare il codice [c#](xref:blazor/call-dotnet-from-javascript).</span><span class="sxs-lookup"><span data-stu-id="21c4a-182">C# code can [call into JavaScript code](xref:blazor/call-javascript-from-dotnet), and JavaScript code can [call into C# code](xref:blazor/call-dotnet-from-javascript).</span></span>

## <a name="code-sharing-and-net-standard"></a><span data-ttu-id="21c4a-183">Condivisione del codice e .NET Standard</span><span class="sxs-lookup"><span data-stu-id="21c4a-183">Code sharing and .NET Standard</span></span>

<span data-ttu-id="21c4a-184">Blazor implementa l' [.NET standard](/dotnet/standard/net-standard), che consente Blazor ai progetti di fare riferimento a librerie conformi alle specifiche di .NET standard.</span><span class="sxs-lookup"><span data-stu-id="21c4a-184">Blazor implements the [.NET Standard](/dotnet/standard/net-standard), which enables Blazor projects to reference libraries that conform to .NET Standard specifications.</span></span> <span data-ttu-id="21c4a-185">.NET Standard è una specifica formale delle API .NET comuni tra le implementazioni di .NET.</span><span class="sxs-lookup"><span data-stu-id="21c4a-185">.NET Standard is a formal specification of .NET APIs that are common across .NET implementations.</span></span> <span data-ttu-id="21c4a-186">Le librerie di classi .NET Standard possono essere condivise tra diverse piattaforme .NET, ad esempio Blazor .NET Framework, .NET Core, Novell, mono e Unity.</span><span class="sxs-lookup"><span data-stu-id="21c4a-186">.NET Standard class libraries can be shared across different .NET platforms, such as Blazor, .NET Framework, .NET Core, Xamarin, Mono, and Unity.</span></span>

<span data-ttu-id="21c4a-187">Le API non valide all'interno di un Web browser (ad esempio per l'accesso al file system, l'apertura di un socket e la gestione dei thread) generano un'eccezione <xref:System.PlatformNotSupportedException>.</span><span class="sxs-lookup"><span data-stu-id="21c4a-187">APIs that aren't applicable inside of a web browser (for example, accessing the file system, opening a socket, and threading) throw a <xref:System.PlatformNotSupportedException>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="21c4a-188">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="21c4a-188">Additional resources</span></span>

* [<span data-ttu-id="21c4a-189">WebAssembly</span><span class="sxs-lookup"><span data-stu-id="21c4a-189">WebAssembly</span></span>](https://webassembly.org)
* <xref:blazor/hosting-models>
* <xref:tutorials/signalr-blazor>
* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>
* [<span data-ttu-id="21c4a-190">Guida a C#</span><span class="sxs-lookup"><span data-stu-id="21c4a-190">C# Guide</span></span>](/dotnet/csharp/)
* [<span data-ttu-id="21c4a-191">Razor informazioni di riferimento sulla sintassi per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="21c4a-191">Razor syntax reference for ASP.NET Core</span></span>](xref:mvc/views/razor)
* [<span data-ttu-id="21c4a-192">HTML</span><span class="sxs-lookup"><span data-stu-id="21c4a-192">HTML</span></span>](https://www.w3.org/html/)
* <span data-ttu-id="21c4a-193">[Awesome Blazor ](https://github.com/AdrienTorris/awesome-blazor) collegamenti della community</span><span class="sxs-lookup"><span data-stu-id="21c4a-193">[Awesome Blazor](https://github.com/AdrienTorris/awesome-blazor) community links</span></span>
