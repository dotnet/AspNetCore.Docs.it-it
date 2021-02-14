---
title: BlazorGestione dello stato ASP.NET Core
author: guardrex
description: Informazioni su come salvare in modo permanente lo stato nelle Blazor Server app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/29/2020
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
uid: blazor/state-management
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: 811ec08eb457fcf0697a64dc4990d29082454f73
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280867"
---
# <a name="aspnet-core-blazor-state-management"></a><span data-ttu-id="3b2d9-103">BlazorGestione dello stato ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="3b2d9-103">ASP.NET Core Blazor state management</span></span>

::: zone pivot="webassembly"

<span data-ttu-id="3b2d9-104">Lo stato utente creato in un' Blazor WebAssembly app viene mantenuto nella memoria del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-104">User state created in a Blazor WebAssembly app is held in the browser's memory.</span></span>

<span data-ttu-id="3b2d9-105">Esempi di stato utente contenuti nella memoria del browser includono:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-105">Examples of user state held in browser memory include:</span></span>

* <span data-ttu-id="3b2d9-106">La gerarchia delle istanze dei componenti e l'output di rendering più recente nell'interfaccia utente sottoposta a rendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-106">The hierarchy of component instances and their most recent render output in the rendered UI.</span></span>
* <span data-ttu-id="3b2d9-107">Valori dei campi e delle proprietà nelle istanze del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-107">The values of fields and properties in component instances.</span></span>
* <span data-ttu-id="3b2d9-108">Dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-108">Data held in [dependency injection (DI)](xref:fundamentals/dependency-injection) service instances.</span></span>
* <span data-ttu-id="3b2d9-109">Valori impostati tramite chiamate di [interoperabilità JavaScript](xref:blazor/call-javascript-from-dotnet) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-109">Values set through [JavaScript interop](xref:blazor/call-javascript-from-dotnet) calls.</span></span>

<span data-ttu-id="3b2d9-110">Quando un utente chiude e riapre il browser o ricarica la pagina, lo stato utente contenuto nella memoria del browser viene perso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-110">When a user closes and re-opens their browser or reloads the page, user state held in the browser's memory is lost.</span></span>

> [!NOTE]
> <span data-ttu-id="3b2d9-111">L' [archiviazione del browser protetto](xref:blazor/state-management?pivots=server#aspnet-core-protected-browser-storage) ( <xref:Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage?displayProperty=fullName> spazio dei nomi) si basa sulla protezione dei dati ASP.NET Core ed è supportata solo per le Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-111">[Protected Browser Storage](xref:blazor/state-management?pivots=server#aspnet-core-protected-browser-storage) (<xref:Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage?displayProperty=fullName> namespace) relies on ASP.NET Core Data Protection and is only supported for Blazor Server apps.</span></span>

## <a name="persist-state-across-browser-sessions"></a><span data-ttu-id="3b2d9-112">Mantieni stato tra le sessioni del browser</span><span class="sxs-lookup"><span data-stu-id="3b2d9-112">Persist state across browser sessions</span></span>

<span data-ttu-id="3b2d9-113">In genere, mantenere lo stato tra le sessioni del browser in cui gli utenti stanno creando attivamente dati, non semplicemente leggendo i dati già esistenti.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-113">Generally, maintain state across browser sessions where users are actively creating data, not simply reading data that already exists.</span></span>

<span data-ttu-id="3b2d9-114">Per mantenere lo stato tra le sessioni del browser, l'app deve salvare in modo permanente i dati in un'altra posizione di archiviazione rispetto alla memoria del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-114">To preserve state across browser sessions, the app must persist the data to some other storage location than the browser's memory.</span></span> <span data-ttu-id="3b2d9-115">La persistenza dello stato non è automatica.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-115">State persistence isn't automatic.</span></span> <span data-ttu-id="3b2d9-116">È necessario eseguire i passaggi quando si sviluppa l'app per implementare la persistenza dei dati con stato.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-116">You must take steps when developing the app to implement stateful data persistence.</span></span>

<span data-ttu-id="3b2d9-117">La persistenza dei dati è in genere necessaria solo per uno stato di valore elevato che gli utenti hanno impiegato per creare.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-117">Data persistence is typically only required for high-value state that users expended effort to create.</span></span> <span data-ttu-id="3b2d9-118">Negli esempi seguenti lo stato di salvataggio permanente Risparmia tempo o facilita le attività commerciali:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-118">In the following examples, persisting state either saves time or aids in commercial activities:</span></span>

* <span data-ttu-id="3b2d9-119">Web Form in più passaggi: è necessario che un utente immetta nuovamente i dati per diversi passaggi completati di un modulo Web in più passaggi se il relativo stato viene perso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-119">Multi-step web forms: It's time-consuming for a user to re-enter data for several completed steps of a multi-step web form if their state is lost.</span></span> <span data-ttu-id="3b2d9-120">Un utente perde lo stato in questo scenario se si allontana dal modulo e viene restituito in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-120">A user loses state in this scenario if they navigate away from the form and return later.</span></span>
* <span data-ttu-id="3b2d9-121">Carrelli acquisti: è possibile mantenere un componente commercialmente importante di un'app che rappresenti potenziali ricavi.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-121">Shopping carts: Any commercially important component of an app that represents potential revenue can be maintained.</span></span> <span data-ttu-id="3b2d9-122">Un utente che perde il proprio stato e quindi il carrello della spesa può acquistare un minor numero di prodotti o servizi quando ritornano al sito in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-122">A user who loses their state, and thus their shopping cart, may purchase fewer products or services when they return to the site later.</span></span>

<span data-ttu-id="3b2d9-123">Un'app può solo salvare *lo stato dell'app*.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-123">An app can only persist *app state*.</span></span> <span data-ttu-id="3b2d9-124">Non è possibile rendere permanente le interfacce utente, ad esempio le istanze dei componenti e le relative strutture di rendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-124">UIs can't be persisted, such as component instances and their render trees.</span></span> <span data-ttu-id="3b2d9-125">I componenti e gli alberi di rendering non sono in genere serializzabili.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-125">Components and render trees aren't generally serializable.</span></span> <span data-ttu-id="3b2d9-126">Per salvare in modo permanente lo stato dell'interfaccia utente, ad esempio i nodi espansi di un controllo di visualizzazione albero, l'app deve usare codice personalizzato per modellare il comportamento dello stato dell'interfaccia utente come stato dell'app serializzabile.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-126">To persist UI state, such as the expanded nodes of a tree view control, the app must use custom code to model the behavior of the UI state as serializable app state.</span></span>

## <a name="where-to-persist-state"></a><span data-ttu-id="3b2d9-127">Posizione in cui salvare lo stato</span><span class="sxs-lookup"><span data-stu-id="3b2d9-127">Where to persist state</span></span>

<span data-ttu-id="3b2d9-128">Esistono località comuni per lo stato permanente:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-128">Common locations exist for persisting state:</span></span>

* [<span data-ttu-id="3b2d9-129">Archiviazione sul lato server</span><span class="sxs-lookup"><span data-stu-id="3b2d9-129">Server-side storage</span></span>](#server-side-storage-wasm)
* [<span data-ttu-id="3b2d9-130">URL</span><span class="sxs-lookup"><span data-stu-id="3b2d9-130">URL</span></span>](#url-wasm)
* [<span data-ttu-id="3b2d9-131">Archiviazione del browser</span><span class="sxs-lookup"><span data-stu-id="3b2d9-131">Browser storage</span></span>](#browser-storage-wasm)
* [<span data-ttu-id="3b2d9-132">Servizio contenitore stato in memoria</span><span class="sxs-lookup"><span data-stu-id="3b2d9-132">In-memory state container service</span></span>](#in-memory-state-container-service-wasm)

<h2 id="server-side-storage-wasm"><span data-ttu-id="3b2d9-133">Archiviazione sul lato server</span><span class="sxs-lookup"><span data-stu-id="3b2d9-133">Server-side storage</span></span></h2>

<span data-ttu-id="3b2d9-134">Per la persistenza permanente dei dati che si estende su più utenti e dispositivi, l'app può usare l'archiviazione indipendente del lato server a cui si accede tramite un'API Web.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-134">For permanent data persistence that spans multiple users and devices, the app can use independent server-side storage accessed via a web API.</span></span> <span data-ttu-id="3b2d9-135">Le opzioni includono:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-135">Options include:</span></span>

* <span data-ttu-id="3b2d9-136">Archiviazione BLOB</span><span class="sxs-lookup"><span data-stu-id="3b2d9-136">Blob storage</span></span>
* <span data-ttu-id="3b2d9-137">Archiviazione chiave-valore</span><span class="sxs-lookup"><span data-stu-id="3b2d9-137">Key-value storage</span></span>
* <span data-ttu-id="3b2d9-138">Database relazionale</span><span class="sxs-lookup"><span data-stu-id="3b2d9-138">Relational database</span></span>
* <span data-ttu-id="3b2d9-139">Archiviazione tabelle</span><span class="sxs-lookup"><span data-stu-id="3b2d9-139">Table storage</span></span>

<span data-ttu-id="3b2d9-140">Dopo il salvataggio dei dati, lo stato dell'utente viene mantenuto e disponibile in qualsiasi nuova sessione del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-140">After data is saved, the user's state is retained and available in any new browser session.</span></span>

<span data-ttu-id="3b2d9-141">Poiché Blazor WebAssembly le app vengono eseguite interamente nel browser dell'utente, richiedono misure aggiuntive per accedere a sistemi esterni protetti, ad esempio i servizi di archiviazione e i database.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-141">Because Blazor WebAssembly apps run entirely in the user's browser, they require additional measures to access secure external systems, such as storage services and databases.</span></span> <span data-ttu-id="3b2d9-142">Blazor WebAssembly le app sono protette in modo analogo alle applicazioni a pagina singola (Spa).</span><span class="sxs-lookup"><span data-stu-id="3b2d9-142">Blazor WebAssembly apps are secured in the same manner as Single Page Applications (SPAs).</span></span> <span data-ttu-id="3b2d9-143">In genere, un'app autentica un utente tramite [OAuth](https://oauth.net) / [OpenID Connect (OIDC)](https://openid.net/connect/) e quindi interagisce con i servizi di archiviazione e i database tramite chiamate API Web a un'app sul lato server.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-143">Typically, an app authenticates a user via [OAuth](https://oauth.net)/[OpenID Connect (OIDC)](https://openid.net/connect/) and then interacts with storage services and databases through web API calls to a server-side app.</span></span> <span data-ttu-id="3b2d9-144">L'app sul lato server media il trasferimento dei dati tra l' Blazor WebAssembly app e il servizio di archiviazione o il database.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-144">The server-side app mediates the transfer of data between the Blazor WebAssembly app and the storage service or database.</span></span> <span data-ttu-id="3b2d9-145">L'app Blazor WebAssembly mantiene una connessione temporanea all'app sul lato server, mentre l'app sul lato server dispone di una connessione permanente alla risorsa di archiviazione.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-145">The Blazor WebAssembly app maintains an ephemeral connection to the server-side app, while the server-side app has a persistent connection to storage.</span></span>

<span data-ttu-id="3b2d9-146">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-146">For more information, see the following resources:</span></span>

* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>
* <span data-ttu-id="3b2d9-147">Blazor*Sicurezza e Identity* articoli</span><span class="sxs-lookup"><span data-stu-id="3b2d9-147">Blazor *Security and Identity* articles</span></span>

<span data-ttu-id="3b2d9-148">Per ulteriori informazioni sulle opzioni di archiviazione dei dati di Azure, vedere gli argomenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-148">For more information on Azure data storage options, see the following:</span></span>

* [<span data-ttu-id="3b2d9-149">Database di Azure</span><span class="sxs-lookup"><span data-stu-id="3b2d9-149">Azure Databases</span></span>](https://azure.microsoft.com/product-categories/databases/)
* [<span data-ttu-id="3b2d9-150">Documentazione di archiviazione di Azure</span><span class="sxs-lookup"><span data-stu-id="3b2d9-150">Azure Storage Documentation</span></span>](/azure/storage/)

<h2 id="url-wasm"><span data-ttu-id="3b2d9-151">URL</span><span class="sxs-lookup"><span data-stu-id="3b2d9-151">URL</span></span></h2>

<span data-ttu-id="3b2d9-152">Per i dati temporanei che rappresentano lo stato di navigazione, modellare i dati come parte dell'URL.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-152">For transient data representing navigation state, model the data as a part of the URL.</span></span> <span data-ttu-id="3b2d9-153">Esempi di stato utente modellati nell'URL includono:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-153">Examples of user state modeled in the URL include:</span></span>

* <span data-ttu-id="3b2d9-154">ID di un'entità visualizzata.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-154">The ID of a viewed entity.</span></span>
* <span data-ttu-id="3b2d9-155">Numero di pagina corrente in una griglia di paging.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-155">The current page number in a paged grid.</span></span>

<span data-ttu-id="3b2d9-156">Il contenuto della barra degli indirizzi del browser viene mantenuto se l'utente ricarica manualmente la pagina.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-156">The contents of the browser's address bar are retained if the user manually reloads the page.</span></span>

<span data-ttu-id="3b2d9-157">Per informazioni sulla definizione di modelli URL con la [`@page`](xref:mvc/views/razor#page) direttiva, vedere <xref:blazor/fundamentals/routing> .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-157">For information on defining URL patterns with the [`@page`](xref:mvc/views/razor#page) directive, see <xref:blazor/fundamentals/routing>.</span></span>

<h2 id="browser-storage-wasm"><span data-ttu-id="3b2d9-158">Archiviazione del browser</span><span class="sxs-lookup"><span data-stu-id="3b2d9-158">Browser storage</span></span></h2>

<span data-ttu-id="3b2d9-159">Per i dati temporanei che l'utente sta creando attivamente, un percorso di archiviazione di uso comune è costituito dalle [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) raccolte e del browser [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-159">For transient data that the user is actively creating, a commonly used storage location is the browser's [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) and [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) collections:</span></span>

* <span data-ttu-id="3b2d9-160">`localStorage` ha come ambito la finestra del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-160">`localStorage` is scoped to the browser's window.</span></span> <span data-ttu-id="3b2d9-161">Se l'utente ricarica la pagina o chiude e riapre il browser, lo stato è permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-161">If the user reloads the page or closes and re-opens the browser, the state persists.</span></span> <span data-ttu-id="3b2d9-162">Se l'utente apre più schede del browser, lo stato viene condiviso tra le schede.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-162">If the user opens multiple browser tabs, the state is shared across the tabs.</span></span> <span data-ttu-id="3b2d9-163">I dati rimangono in `localStorage` fino a quando non vengono cancellati in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-163">Data persists in `localStorage` until explicitly cleared.</span></span>
* <span data-ttu-id="3b2d9-164">`sessionStorage` ha come ambito la scheda Esplorazione. Se l'utente ricarica la scheda, lo stato è permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-164">`sessionStorage` is scoped to the browser tab. If the user reloads the tab, the state persists.</span></span> <span data-ttu-id="3b2d9-165">Se l'utente chiude la scheda o il browser, lo stato viene perso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-165">If the user closes the tab or the browser, the state is lost.</span></span> <span data-ttu-id="3b2d9-166">Se l'utente apre più schede del browser, ogni scheda ha una propria versione indipendente dei dati.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-166">If the user opens multiple browser tabs, each tab has its own independent version of the data.</span></span>

> [!NOTE]
> <span data-ttu-id="3b2d9-167">`localStorage` e `sessionStorage` possono essere usati nelle Blazor WebAssembly app, ma solo scrivendo codice personalizzato o usando un pacchetto di terze parti.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-167">`localStorage` and `sessionStorage` can be used in Blazor WebAssembly apps but only by writing custom code or using a third-party package.</span></span>

<span data-ttu-id="3b2d9-168">In genere, `sessionStorage` è più sicuro da usare.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-168">Generally, `sessionStorage` is safer to use.</span></span> <span data-ttu-id="3b2d9-169">`sessionStorage` evita il rischio che un utente apra più schede e riscontri quanto segue:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-169">`sessionStorage` avoids the risk that a user opens multiple tabs and encounters the following:</span></span>

* <span data-ttu-id="3b2d9-170">Bug nell'archiviazione dello stato tra le schede.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-170">Bugs in state storage across tabs.</span></span>
* <span data-ttu-id="3b2d9-171">Comportamento confuso quando una scheda sovrascrive lo stato di altre schede.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-171">Confusing behavior when a tab overwrites the state of other tabs.</span></span>

<span data-ttu-id="3b2d9-172">`localStorage` è la scelta migliore se l'app deve conservare lo stato tra la chiusura e la riapertura del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-172">`localStorage` is the better choice if the app must persist state across closing and re-opening the browser.</span></span>

> [!WARNING]
> <span data-ttu-id="3b2d9-173">Gli utenti possono visualizzare o manomettere i dati archiviati in `localStorage` e `sessionStorage` .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-173">Users may view or tamper with the data stored in `localStorage` and `sessionStorage`.</span></span>

<h2 id="in-memory-state-container-service-wasm"><span data-ttu-id="3b2d9-174">Servizio contenitore stato in memoria</span><span class="sxs-lookup"><span data-stu-id="3b2d9-174">In-memory state container service</span></span></h2>

[!INCLUDE[](~/blazor/includes/state-container.md)]

## <a name="additional-resources"></a><span data-ttu-id="3b2d9-175">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="3b2d9-175">Additional resources</span></span>

* [<span data-ttu-id="3b2d9-176">Salva lo stato dell'app prima di un'operazione di autenticazione</span><span class="sxs-lookup"><span data-stu-id="3b2d9-176">Save app state before an authentication operation</span></span>](xref:blazor/security/webassembly/additional-scenarios#save-app-state-before-an-authentication-operation)
* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>

::: zone-end

::: zone pivot="server"

<span data-ttu-id="3b2d9-177">Blazor Server è un Framework di app con stato.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-177">Blazor Server is a stateful app framework.</span></span> <span data-ttu-id="3b2d9-178">Nella maggior parte dei casi, l'app mantiene una connessione al server.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-178">Most of the time, the app maintains a connection to the server.</span></span> <span data-ttu-id="3b2d9-179">Lo stato dell'utente viene mantenuto nella memoria del server in un *circuito*.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-179">The user's state is held in the server's memory in a *circuit*.</span></span> 

<span data-ttu-id="3b2d9-180">Esempi di stato utente contenuti in un circuito includono:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-180">Examples of user state held in a circuit include:</span></span>

* <span data-ttu-id="3b2d9-181">La gerarchia delle istanze dei componenti e l'output di rendering più recente nell'interfaccia utente sottoposta a rendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-181">The hierarchy of component instances and their most recent render output in the rendered UI.</span></span>
* <span data-ttu-id="3b2d9-182">Valori dei campi e delle proprietà nelle istanze del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-182">The values of fields and properties in component instances.</span></span>
* <span data-ttu-id="3b2d9-183">Dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) che hanno come ambito il circuito.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-183">Data held in [dependency injection (DI)](xref:fundamentals/dependency-injection) service instances that are scoped to the circuit.</span></span>

<span data-ttu-id="3b2d9-184">Lo stato utente potrebbe trovarsi anche in variabili JavaScript nel set di memoria del browser tramite chiamate di [interoperabilità JavaScript](xref:blazor/call-javascript-from-dotnet) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-184">User state might also be found in JavaScript variables in the browser's memory set via [JavaScript interop](xref:blazor/call-javascript-from-dotnet) calls.</span></span>

<span data-ttu-id="3b2d9-185">Se un utente riscontra una perdita di connessione di rete temporanea, Blazor tenta di riconnettere l'utente al circuito originale con lo stato originale.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-185">If a user experiences a temporary network connection loss, Blazor attempts to reconnect the user to their original circuit with their original state.</span></span> <span data-ttu-id="3b2d9-186">Tuttavia, la riconnessione di un utente al circuito originale nella memoria del server non è sempre possibile:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-186">However, reconnecting a user to their original circuit in the server's memory isn't always possible:</span></span>

* <span data-ttu-id="3b2d9-187">Il server non è in grado di mantenere sempre un circuito disconnesso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-187">The server can't retain a disconnected circuit forever.</span></span> <span data-ttu-id="3b2d9-188">Il server deve rilasciare un circuito disconnesso dopo un timeout o quando il server è sottoposto a un numero eccessivo di richieste di memoria.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-188">The server must release a disconnected circuit after a timeout or when the server is under memory pressure.</span></span>
* <span data-ttu-id="3b2d9-189">Negli ambienti di distribuzione con bilanciamento del carico multiserver, i singoli server possono avere esito negativo o essere rimossi automaticamente quando non sono più necessari per gestire il volume complessivo delle richieste.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-189">In multi-server, load-balanced deployment environments, individual servers may fail or be automatically removed when no longer required to handle the overall volume of requests.</span></span> <span data-ttu-id="3b2d9-190">Le richieste di elaborazione del server originale per un utente potrebbero non essere più disponibili quando l'utente tenta di riconnettersi.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-190">The original server processing requests for a user may become unavailable when the user attempts to reconnect.</span></span>
* <span data-ttu-id="3b2d9-191">L'utente potrebbe chiudere e riaprire il browser o ricaricare la pagina, che rimuove qualsiasi stato contenuto nella memoria del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-191">The user might close and re-open their browser or reload the page, which removes any state held in the browser's memory.</span></span> <span data-ttu-id="3b2d9-192">Ad esempio, i valori delle variabili JavaScript impostati tramite le chiamate di interoperabilità JavaScript vengono persi.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-192">For example, JavaScript variable values set through JavaScript interop calls are lost.</span></span>

<span data-ttu-id="3b2d9-193">Quando un utente non può essere riconnesso al circuito originale, l'utente riceve un nuovo circuito con uno stato vuoto.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-193">When a user can't be reconnected to their original circuit, the user receives a new circuit with an empty state.</span></span> <span data-ttu-id="3b2d9-194">Questa operazione equivale a chiudere e riaprire un'app desktop.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-194">This is equivalent to closing and re-opening a desktop app.</span></span>

## <a name="persist-state-across-circuits"></a><span data-ttu-id="3b2d9-195">Mantieni lo stato tra circuiti</span><span class="sxs-lookup"><span data-stu-id="3b2d9-195">Persist state across circuits</span></span>

<span data-ttu-id="3b2d9-196">In genere, mantenere lo stato nei circuiti in cui gli utenti stanno creando attivamente dati, non semplicemente leggendo i dati già esistenti.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-196">Generally, maintain state across circuits where users are actively creating data, not simply reading data that already exists.</span></span>

<span data-ttu-id="3b2d9-197">Per mantenere lo stato tra circuiti, l'app deve salvare in modo permanente i dati in un'altra posizione di archiviazione rispetto alla memoria del server.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-197">To preserve state across circuits, the app must persist the data to some other storage location than the server's memory.</span></span> <span data-ttu-id="3b2d9-198">La persistenza dello stato non è automatica.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-198">State persistence isn't automatic.</span></span> <span data-ttu-id="3b2d9-199">È necessario eseguire i passaggi quando si sviluppa l'app per implementare la persistenza dei dati con stato.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-199">You must take steps when developing the app to implement stateful data persistence.</span></span>

<span data-ttu-id="3b2d9-200">La persistenza dei dati è in genere necessaria solo per uno stato di valore elevato che gli utenti hanno impiegato per creare.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-200">Data persistence is typically only required for high-value state that users expended effort to create.</span></span> <span data-ttu-id="3b2d9-201">Negli esempi seguenti lo stato di salvataggio permanente Risparmia tempo o facilita le attività commerciali:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-201">In the following examples, persisting state either saves time or aids in commercial activities:</span></span>

* <span data-ttu-id="3b2d9-202">Web Form in più passaggi: è necessario che un utente immetta nuovamente i dati per diversi passaggi completati di un modulo Web in più passaggi se il relativo stato viene perso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-202">Multi-step web forms: It's time-consuming for a user to re-enter data for several completed steps of a multi-step web form if their state is lost.</span></span> <span data-ttu-id="3b2d9-203">Un utente perde lo stato in questo scenario se si allontana dal modulo e viene restituito in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-203">A user loses state in this scenario if they navigate away from the form and return later.</span></span>
* <span data-ttu-id="3b2d9-204">Carrelli acquisti: è possibile mantenere un componente commercialmente importante di un'app che rappresenti potenziali ricavi.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-204">Shopping carts: Any commercially important component of an app that represents potential revenue can be maintained.</span></span> <span data-ttu-id="3b2d9-205">Un utente che perde il proprio stato e quindi il carrello della spesa può acquistare un minor numero di prodotti o servizi quando ritornano al sito in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-205">A user who loses their state, and thus their shopping cart, may purchase fewer products or services when they return to the site later.</span></span>

<span data-ttu-id="3b2d9-206">Un'app può solo salvare *lo stato dell'app*.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-206">An app can only persist *app state*.</span></span> <span data-ttu-id="3b2d9-207">Non è possibile rendere permanente le interfacce utente, ad esempio le istanze dei componenti e le relative strutture di rendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-207">UIs can't be persisted, such as component instances and their render trees.</span></span> <span data-ttu-id="3b2d9-208">I componenti e gli alberi di rendering non sono in genere serializzabili.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-208">Components and render trees aren't generally serializable.</span></span> <span data-ttu-id="3b2d9-209">Per salvare in modo permanente lo stato dell'interfaccia utente, ad esempio i nodi espansi di un controllo di visualizzazione albero, l'app deve usare codice personalizzato per modellare il comportamento dello stato dell'interfaccia utente come stato dell'app serializzabile.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-209">To persist UI state, such as the expanded nodes of a tree view control, the app must use custom code to model the behavior of the UI state as serializable app state.</span></span>

## <a name="where-to-persist-state"></a><span data-ttu-id="3b2d9-210">Posizione in cui salvare lo stato</span><span class="sxs-lookup"><span data-stu-id="3b2d9-210">Where to persist state</span></span>

<span data-ttu-id="3b2d9-211">Esistono località comuni per lo stato permanente:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-211">Common locations exist for persisting state:</span></span>

* [<span data-ttu-id="3b2d9-212">Archiviazione sul lato server</span><span class="sxs-lookup"><span data-stu-id="3b2d9-212">Server-side storage</span></span>](#server-side-storage-server)
* [<span data-ttu-id="3b2d9-213">URL</span><span class="sxs-lookup"><span data-stu-id="3b2d9-213">URL</span></span>](#url-server)
* [<span data-ttu-id="3b2d9-214">Archiviazione del browser</span><span class="sxs-lookup"><span data-stu-id="3b2d9-214">Browser storage</span></span>](#browser-storage-server)
* [<span data-ttu-id="3b2d9-215">Servizio contenitore stato in memoria</span><span class="sxs-lookup"><span data-stu-id="3b2d9-215">In-memory state container service</span></span>](#in-memory-state-container-service-server)

<h2 id="server-side-storage-server"><span data-ttu-id="3b2d9-216">Archiviazione sul lato server</span><span class="sxs-lookup"><span data-stu-id="3b2d9-216">Server-side storage</span></span></h2>

<span data-ttu-id="3b2d9-217">Per la persistenza permanente dei dati che si estende su più utenti e dispositivi, l'app può usare l'archiviazione sul lato server.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-217">For permanent data persistence that spans multiple users and devices, the app can use server-side storage.</span></span> <span data-ttu-id="3b2d9-218">Le opzioni includono:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-218">Options include:</span></span>

* <span data-ttu-id="3b2d9-219">Archiviazione BLOB</span><span class="sxs-lookup"><span data-stu-id="3b2d9-219">Blob storage</span></span>
* <span data-ttu-id="3b2d9-220">Archiviazione chiave-valore</span><span class="sxs-lookup"><span data-stu-id="3b2d9-220">Key-value storage</span></span>
* <span data-ttu-id="3b2d9-221">Database relazionale</span><span class="sxs-lookup"><span data-stu-id="3b2d9-221">Relational database</span></span>
* <span data-ttu-id="3b2d9-222">Archiviazione tabelle</span><span class="sxs-lookup"><span data-stu-id="3b2d9-222">Table storage</span></span>

<span data-ttu-id="3b2d9-223">Dopo il salvataggio dei dati, lo stato dell'utente viene mantenuto e disponibile in qualsiasi nuovo circuito.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-223">After data is saved, the user's state is retained and available in any new circuit.</span></span>

<span data-ttu-id="3b2d9-224">Per ulteriori informazioni sulle opzioni di archiviazione dei dati di Azure, vedere gli argomenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-224">For more information on Azure data storage options, see the following:</span></span>

* [<span data-ttu-id="3b2d9-225">Database di Azure</span><span class="sxs-lookup"><span data-stu-id="3b2d9-225">Azure Databases</span></span>](https://azure.microsoft.com/product-categories/databases/)
* [<span data-ttu-id="3b2d9-226">Documentazione di archiviazione di Azure</span><span class="sxs-lookup"><span data-stu-id="3b2d9-226">Azure Storage Documentation</span></span>](/azure/storage/)

<h2 id="url-server"><span data-ttu-id="3b2d9-227">URL</span><span class="sxs-lookup"><span data-stu-id="3b2d9-227">URL</span></span></h2>

<span data-ttu-id="3b2d9-228">Per i dati temporanei che rappresentano lo stato di navigazione, modellare i dati come parte dell'URL.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-228">For transient data representing navigation state, model the data as a part of the URL.</span></span> <span data-ttu-id="3b2d9-229">Esempi di stato utente modellati nell'URL includono:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-229">Examples of user state modeled in the URL include:</span></span>

* <span data-ttu-id="3b2d9-230">ID di un'entità visualizzata.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-230">The ID of a viewed entity.</span></span>
* <span data-ttu-id="3b2d9-231">Numero di pagina corrente in una griglia di paging.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-231">The current page number in a paged grid.</span></span>

<span data-ttu-id="3b2d9-232">Il contenuto della barra degli indirizzi del browser viene mantenuto:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-232">The contents of the browser's address bar are retained:</span></span>

* <span data-ttu-id="3b2d9-233">Se l'utente ricarica manualmente la pagina.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-233">If the user manually reloads the page.</span></span>
* <span data-ttu-id="3b2d9-234">Se il server Web non è più disponibile e l'utente è costretto a ricaricare la pagina per connettersi a un server diverso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-234">If the web server becomes unavailable, and the user is forced to reload the page in order to connect to a different server.</span></span>

<span data-ttu-id="3b2d9-235">Per informazioni sulla definizione di modelli URL con la [`@page`](xref:mvc/views/razor#page) direttiva, vedere <xref:blazor/fundamentals/routing> .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-235">For information on defining URL patterns with the [`@page`](xref:mvc/views/razor#page) directive, see <xref:blazor/fundamentals/routing>.</span></span>

<h2 id="browser-storage-server"><span data-ttu-id="3b2d9-236">Archiviazione del browser</span><span class="sxs-lookup"><span data-stu-id="3b2d9-236">Browser storage</span></span></h2>

<span data-ttu-id="3b2d9-237">Per i dati temporanei che l'utente sta creando attivamente, un percorso di archiviazione di uso comune è costituito dalle [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) raccolte e del browser [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-237">For transient data that the user is actively creating, a commonly used storage location is the browser's [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) and [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) collections:</span></span>

* <span data-ttu-id="3b2d9-238">`localStorage` ha come ambito la finestra del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-238">`localStorage` is scoped to the browser's window.</span></span> <span data-ttu-id="3b2d9-239">Se l'utente ricarica la pagina o chiude e riapre il browser, lo stato è permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-239">If the user reloads the page or closes and re-opens the browser, the state persists.</span></span> <span data-ttu-id="3b2d9-240">Se l'utente apre più schede del browser, lo stato viene condiviso tra le schede.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-240">If the user opens multiple browser tabs, the state is shared across the tabs.</span></span> <span data-ttu-id="3b2d9-241">I dati rimangono in `localStorage` fino a quando non vengono cancellati in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-241">Data persists in `localStorage` until explicitly cleared.</span></span>
* <span data-ttu-id="3b2d9-242">`sessionStorage` ha come ambito la scheda Esplorazione. Se l'utente ricarica la scheda, lo stato è permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-242">`sessionStorage` is scoped to the browser tab. If the user reloads the tab, the state persists.</span></span> <span data-ttu-id="3b2d9-243">Se l'utente chiude la scheda o il browser, lo stato viene perso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-243">If the user closes the tab or the browser, the state is lost.</span></span> <span data-ttu-id="3b2d9-244">Se l'utente apre più schede del browser, ogni scheda ha una propria versione indipendente dei dati.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-244">If the user opens multiple browser tabs, each tab has its own independent version of the data.</span></span>

<span data-ttu-id="3b2d9-245">In genere, `sessionStorage` è più sicuro da usare.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-245">Generally, `sessionStorage` is safer to use.</span></span> <span data-ttu-id="3b2d9-246">`sessionStorage` evita il rischio che un utente apra più schede e riscontri quanto segue:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-246">`sessionStorage` avoids the risk that a user opens multiple tabs and encounters the following:</span></span>

* <span data-ttu-id="3b2d9-247">Bug nell'archiviazione dello stato tra le schede.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-247">Bugs in state storage across tabs.</span></span>
* <span data-ttu-id="3b2d9-248">Comportamento confuso quando una scheda sovrascrive lo stato di altre schede.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-248">Confusing behavior when a tab overwrites the state of other tabs.</span></span>

<span data-ttu-id="3b2d9-249">`localStorage` è la scelta migliore se l'app deve conservare lo stato tra la chiusura e la riapertura del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-249">`localStorage` is the better choice if the app must persist state across closing and re-opening the browser.</span></span>

<span data-ttu-id="3b2d9-250">Avvertenze per l'uso dell'archiviazione del browser:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-250">Caveats for using browser storage:</span></span>

* <span data-ttu-id="3b2d9-251">Analogamente all'utilizzo di un database lato server, il caricamento e il salvataggio dei dati sono asincroni.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-251">Similar to the use of a server-side database, loading and saving data are asynchronous.</span></span>
* <span data-ttu-id="3b2d9-252">A differenza di un database lato server, l'archiviazione non è disponibile durante il prerendering perché la pagina richiesta non esiste nel browser durante la fase di pre-rendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-252">Unlike a server-side database, storage isn't available during prerendering because the requested page doesn't exist in the browser during the prerendering stage.</span></span>
* <span data-ttu-id="3b2d9-253">L'archiviazione di alcuni kilobyte di dati è ragionevole per la permanenza per le Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-253">Storage of a few kilobytes of data is reasonable to persist for Blazor Server apps.</span></span> <span data-ttu-id="3b2d9-254">Oltre alcuni kilobyte, è necessario considerare le implicazioni relative alle prestazioni perché i dati vengono caricati e salvati in rete.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-254">Beyond a few kilobytes, you must consider the performance implications because the data is loaded and saved across the network.</span></span>
* <span data-ttu-id="3b2d9-255">Gli utenti possono visualizzare o manomettere i dati.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-255">Users may view or tamper with the data.</span></span> <span data-ttu-id="3b2d9-256">[ASP.NET Core protezione dei dati](xref:security/data-protection/introduction) può ridurre il rischio.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-256">[ASP.NET Core Data Protection](xref:security/data-protection/introduction) can mitigate the risk.</span></span> <span data-ttu-id="3b2d9-257">Ad esempio, [ASP.NET Core archiviazione del browser protetta](#aspnet-core-protected-browser-storage) utilizza ASP.NET Core la protezione dei dati.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-257">For example, [ASP.NET Core Protected Browser Storage](#aspnet-core-protected-browser-storage) uses ASP.NET Core Data Protection.</span></span>

<span data-ttu-id="3b2d9-258">I pacchetti NuGet di terze parti forniscono le API per l'uso di `localStorage` e `sessionStorage` .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-258">Third-party NuGet packages provide APIs for working with `localStorage` and `sessionStorage`.</span></span> <span data-ttu-id="3b2d9-259">Vale la pena considerare la scelta di un pacchetto che usa in modo trasparente [ASP.NET Core la protezione dei dati](xref:security/data-protection/introduction).</span><span class="sxs-lookup"><span data-stu-id="3b2d9-259">It's worth considering choosing a package that transparently uses [ASP.NET Core Data Protection](xref:security/data-protection/introduction).</span></span> <span data-ttu-id="3b2d9-260">La protezione dei dati consente di crittografare i dati archiviati e riduce il rischio potenziale di manomissione dei dati archiviati.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-260">Data Protection encrypts stored data and reduces the potential risk of tampering with stored data.</span></span> <span data-ttu-id="3b2d9-261">Se i dati serializzati in JSON vengono archiviati in testo normale, gli utenti possono visualizzare i dati usando gli strumenti di sviluppo del browser e modificare anche i dati archiviati.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-261">If JSON-serialized data is stored in plain text, users can see the data using browser developer tools and also modify the stored data.</span></span> <span data-ttu-id="3b2d9-262">La protezione dei dati non è sempre un problema perché i dati potrebbero essere di natura banale.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-262">Securing data isn't always a problem because the data might be trivial in nature.</span></span> <span data-ttu-id="3b2d9-263">Ad esempio, la lettura o la modifica del colore archiviato di un elemento dell'interfaccia utente non costituisce un rischio di sicurezza significativo per l'utente o l'organizzazione.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-263">For example, reading or modifying the stored color of a UI element isn't a significant security risk to the user or the organization.</span></span> <span data-ttu-id="3b2d9-264">Evitare di consentire agli utenti di ispezionare o manomettere *i dati sensibili*.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-264">Avoid allowing users to inspect or tamper with *sensitive data*.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="aspnet-core-protected-browser-storage"></a><span data-ttu-id="3b2d9-265">ASP.NET Core archiviazione del browser protetta</span><span class="sxs-lookup"><span data-stu-id="3b2d9-265">ASP.NET Core Protected Browser Storage</span></span>

<span data-ttu-id="3b2d9-266">ASP.NET Core archiviazione del browser protetta sfrutta la [protezione dei dati ASP.NET Core](xref:security/data-protection/introduction) per [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) e [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-266">ASP.NET Core Protected Browser Storage leverages [ASP.NET Core Data Protection](xref:security/data-protection/introduction) for [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) and [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage).</span></span>

> [!NOTE]
> <span data-ttu-id="3b2d9-267">L'archiviazione del browser protetto si basa sulla protezione dei dati ASP.NET Core ed è supportata solo per le Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-267">Protected Browser Storage relies on ASP.NET Core Data Protection and is only supported for Blazor Server apps.</span></span>

### <a name="save-and-load-data-within-a-component"></a><span data-ttu-id="3b2d9-268">Salvare e caricare i dati all'interno di un componente</span><span class="sxs-lookup"><span data-stu-id="3b2d9-268">Save and load data within a component</span></span>

<span data-ttu-id="3b2d9-269">In tutti i componenti che richiedono il caricamento o il salvataggio dei dati nell'archiviazione del browser, utilizzare la [`@inject`](xref:mvc/views/razor#inject) direttiva per inserire un'istanza di uno dei seguenti elementi:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-269">In any component that requires loading or saving data to browser storage, use the [`@inject`](xref:mvc/views/razor#inject) directive to inject an instance of either of the following:</span></span>

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

<span data-ttu-id="3b2d9-270">La scelta dipende dal percorso di archiviazione del browser che si desidera utilizzare.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-270">The choice depends on which browser storage location you wish to use.</span></span> <span data-ttu-id="3b2d9-271">Nell'esempio seguente `sessionStorage` viene usato:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-271">In the following example, `sessionStorage` is used:</span></span>

```razor
@using Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore
```

<span data-ttu-id="3b2d9-272">La `@using` direttiva può essere inserita nel file dell'app `_Imports.razor` anziché nel componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-272">The `@using` directive can be placed in the app's `_Imports.razor` file instead of in the component.</span></span> <span data-ttu-id="3b2d9-273">L'uso del `_Imports.razor` file rende lo spazio dei nomi disponibile a segmenti più grandi dell'app o dell'intera app.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-273">Use of the `_Imports.razor` file makes the namespace available to larger segments of the app or the whole app.</span></span>

<span data-ttu-id="3b2d9-274">Per salvare in modo permanente il `currentCount` valore nel `Counter` componente di un'app in base al Blazor Server modello di progetto, modificare il `IncrementCount` metodo in modo da usare `ProtectedSessionStore.SetAsync` :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-274">To persist the `currentCount` value in the `Counter` component of an app based on the Blazor Server project template, modify the `IncrementCount` method to use `ProtectedSessionStore.SetAsync`:</span></span>

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

<span data-ttu-id="3b2d9-275">Nelle app più grandi e più realistiche, l'archiviazione dei singoli campi è uno scenario improbabile.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-275">In larger, more realistic apps, storage of individual fields is an unlikely scenario.</span></span> <span data-ttu-id="3b2d9-276">È più probabile che le app memorizzino interi oggetti modello che includono lo stato complesso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-276">Apps are more likely to store entire model objects that include complex state.</span></span> <span data-ttu-id="3b2d9-277">`ProtectedSessionStore` serializza e deserializza automaticamente i dati JSON per archiviare oggetti di stato complessi.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-277">`ProtectedSessionStore` automatically serializes and deserializes JSON data to store complex state objects.</span></span>

<span data-ttu-id="3b2d9-278">Nell'esempio di codice precedente i `currentCount` dati vengono archiviati come `sessionStorage['count']` nel browser dell'utente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-278">In the preceding code example, the `currentCount` data is stored as `sessionStorage['count']` in the user's browser.</span></span> <span data-ttu-id="3b2d9-279">I dati non vengono archiviati in testo normale ma sono invece protetti tramite la protezione dei dati ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-279">The data isn't stored in plain text but rather is protected using ASP.NET Core Data Protection.</span></span> <span data-ttu-id="3b2d9-280">I dati crittografati possono essere controllati se `sessionStorage['count']` viene valutato nella console per sviluppatori del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-280">The encrypted data can be inspected if `sessionStorage['count']` is evaluated in the browser's developer console.</span></span>

<span data-ttu-id="3b2d9-281">Per ripristinare i `currentCount` dati se l'utente torna al `Counter` componente in un secondo momento, anche se l'utente si trova in un nuovo circuito, usare `ProtectedSessionStore.GetAsync` :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-281">To recover the `currentCount` data if the user returns to the `Counter` component later, including if the user is on a new circuit, use `ProtectedSessionStore.GetAsync`:</span></span>

```csharp
protected override async Task OnInitializedAsync()
{
    var result = await ProtectedSessionStore.GetAsync<int>("count");
    currentCount = result.Success ? result.Value : 0;
}
```

<span data-ttu-id="3b2d9-282">Se i parametri del componente includono lo stato di navigazione, chiamare `ProtectedSessionStore.GetAsync` e assegnare un non `null` risultato in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> , non <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-282">If the component's parameters include navigation state, call `ProtectedSessionStore.GetAsync` and assign a non-`null` result in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A>, not <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>.</span></span> <span data-ttu-id="3b2d9-283"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene chiamato una sola volta al momento della creazione di un'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-283"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> is only called once when the component is first instantiated.</span></span> <span data-ttu-id="3b2d9-284"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> non viene chiamato di nuovo in un secondo momento se l'utente passa a un URL diverso rimanendo nella stessa pagina.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-284"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> isn't called again later if the user navigates to a different URL while remaining on the same page.</span></span> <span data-ttu-id="3b2d9-285">Per altre informazioni, vedere <xref:blazor/components/lifecycle>.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-285">For more information, see <xref:blazor/components/lifecycle>.</span></span>

> [!WARNING]
> <span data-ttu-id="3b2d9-286">Gli esempi in questa sezione funzionano solo se per il server non è abilitato il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-286">The examples in this section only work if the server doesn't have prerendering enabled.</span></span> <span data-ttu-id="3b2d9-287">Con il prerendering abilitato, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-287">With prerendering enabled, an error is generated explaining that JavaScript interop calls cannot be issued because the component is being prerendered.</span></span>
>
> <span data-ttu-id="3b2d9-288">Disabilitare il prerendering o aggiungere altro codice per lavorare con il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-288">Either disable prerendering or add additional code to work with prerendering.</span></span> <span data-ttu-id="3b2d9-289">Per ulteriori informazioni sulla scrittura di codice che funziona con il prerendering, vedere la sezione [handle prerendering](#handle-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-289">To learn more about writing code that works with prerendering, see the [Handle prerendering](#handle-prerendering) section.</span></span>

### <a name="handle-the-loading-state"></a><span data-ttu-id="3b2d9-290">Gestire lo stato di caricamento</span><span class="sxs-lookup"><span data-stu-id="3b2d9-290">Handle the loading state</span></span>

<span data-ttu-id="3b2d9-291">Poiché l'accesso all'archiviazione del browser avviene in modo asincrono tramite una connessione di rete, è sempre necessario un certo periodo di tempo prima che i dati vengano caricati e disponibili per un componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-291">Since browser storage is accessed asynchronously over a network connection, there's always a period of time before the data is loaded and available to a component.</span></span> <span data-ttu-id="3b2d9-292">Per ottenere risultati ottimali, è possibile eseguire il rendering di un messaggio di stato di caricamento mentre è in corso il caricamento anziché visualizzare dati vuoti o predefiniti.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-292">For the best results, render a loading-state message while loading is in progress instead of displaying blank or default data.</span></span>

<span data-ttu-id="3b2d9-293">Un approccio consiste nel rilevare se i dati sono `null` , il che significa che i dati sono ancora in corso di caricamento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-293">One approach is to track whether the data is `null`, which means that the data is still loading.</span></span> <span data-ttu-id="3b2d9-294">Nel componente predefinito `Counter` , il conteggio viene mantenuto in un oggetto `int` .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-294">In the default `Counter` component, the count is held in an `int`.</span></span> <span data-ttu-id="3b2d9-295">[Rendere `currentCount` Nullable](/dotnet/csharp/language-reference/builtin-types/nullable-value-types) aggiungendo un punto interrogativo ( `?` ) al tipo ( `int` ):</span><span class="sxs-lookup"><span data-stu-id="3b2d9-295">[Make `currentCount` nullable](/dotnet/csharp/language-reference/builtin-types/nullable-value-types) by adding a question mark (`?`) to the type (`int`):</span></span>

```csharp
private int? currentCount;
```

<span data-ttu-id="3b2d9-296">Anziché visualizzare in modo incondizionato il conteggio e il **`Increment`** pulsante, visualizzare questi elementi solo se i dati vengono caricati controllando <xref:System.Nullable%601.HasValue%2A> :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-296">Instead of unconditionally displaying the count and **`Increment`** button, display these elements only if the data is loaded by checking <xref:System.Nullable%601.HasValue%2A>:</span></span>

```razor
@if (currentCount.HasValue)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}
```

### <a name="handle-prerendering"></a><span data-ttu-id="3b2d9-297">Gestione preliminare</span><span class="sxs-lookup"><span data-stu-id="3b2d9-297">Handle prerendering</span></span>

<span data-ttu-id="3b2d9-298">Durante il prerendering:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-298">During prerendering:</span></span>

* <span data-ttu-id="3b2d9-299">Una connessione interattiva al browser dell'utente non esiste.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-299">An interactive connection to the user's browser doesn't exist.</span></span>
* <span data-ttu-id="3b2d9-300">Il browser non dispone ancora di una pagina in cui è possibile eseguire il codice JavaScript.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-300">The browser doesn't yet have a page in which it can run JavaScript code.</span></span>

<span data-ttu-id="3b2d9-301">`localStorage` o `sessionStorage` non sono disponibili durante il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-301">`localStorage` or `sessionStorage` aren't available during prerendering.</span></span> <span data-ttu-id="3b2d9-302">Se il componente tenta di interagire con l'archiviazione, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-302">If the component attempts to interact with storage, an error is generated explaining that JavaScript interop calls cannot be issued because the component is being prerendered.</span></span>

<span data-ttu-id="3b2d9-303">Un modo per risolvere l'errore consiste nel disabilitare il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-303">One way to resolve the error is to disable prerendering.</span></span> <span data-ttu-id="3b2d9-304">Questa è in genere la scelta migliore se l'app usa in modo intensivo l'archiviazione basata su browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-304">This is usually the best choice if the app makes heavy use of browser-based storage.</span></span> <span data-ttu-id="3b2d9-305">Il prerendering aggiunge complessità e non è vantaggioso per l'app perché l'app non può prerenderizzare contenuti utili fino a quando non `localStorage` `sessionStorage` sono disponibili o.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-305">Prerendering adds complexity and doesn't benefit the app because the app can't prerender any useful content until `localStorage` or `sessionStorage` are available.</span></span>

<span data-ttu-id="3b2d9-306">Per disabilitare il prerendering, aprire il `Pages/_Host.cshtml` file e modificare l' `render-mode` attributo dell' [Helper tag dei componenti](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) in <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-306">To disable prerendering, open the `Pages/_Host.cshtml` file and change the `render-mode` attribute of the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) to <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server>:</span></span>

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

<span data-ttu-id="3b2d9-307">Il prerendering può essere utile per altre pagine che non utilizzano `localStorage` o `sessionStorage` .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-307">Prerendering might be useful for other pages that don't use `localStorage` or `sessionStorage`.</span></span> <span data-ttu-id="3b2d9-308">Per mantenere il prerendering, rinviare l'operazione di caricamento finché il browser non è connesso al circuito.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-308">To retain prerendering, defer the loading operation until the browser is connected to the circuit.</span></span> <span data-ttu-id="3b2d9-309">Di seguito è riportato un esempio per l'archiviazione di un valore del contatore:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-309">The following is an example for storing a counter value:</span></span>

```razor
@using Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage
@inject ProtectedLocalStorage ProtectedLocalStore

@if (isConnected)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}

@code {
    private int currentCount;
    private bool isConnected;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            isConnected = true;
            await LoadStateAsync();
            StateHasChanged();
        }
    }

    private async Task LoadStateAsync()
    {
        var result = await ProtectedLocalStore.GetAsync<int>("count");
        currentCount = result.Success ? result.Value : 0;
    }

    private async Task IncrementCount()
    {
        currentCount++;
        await ProtectedLocalStore.SetAsync("count", currentCount);
    }
}
```

### <a name="factor-out-the-state-preservation-to-a-common-location"></a><span data-ttu-id="3b2d9-310">Fattorizzare la conservazione dello stato in una posizione comune</span><span class="sxs-lookup"><span data-stu-id="3b2d9-310">Factor out the state preservation to a common location</span></span>

<span data-ttu-id="3b2d9-311">Se molti componenti si basano sull'archiviazione basata su browser, la reimplementazione del codice del provider di stato più volte crea una duplicazione del codice.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-311">If many components rely on browser-based storage, re-implementing state provider code many times creates code duplication.</span></span> <span data-ttu-id="3b2d9-312">Un'opzione per evitare la duplicazione del codice consiste nel creare un *componente padre del provider di stato* che incapsula la logica del provider di stato.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-312">One option for avoiding code duplication is to create a *state provider parent component* that encapsulates the state provider logic.</span></span> <span data-ttu-id="3b2d9-313">I componenti figlio possono funzionare con i dati persistenti senza considerare il meccanismo di persistenza dello stato.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-313">Child components can work with persisted data without regard to the state persistence mechanism.</span></span>

<span data-ttu-id="3b2d9-314">Nell'esempio seguente di un `CounterStateProvider` componente, i dati del contatore vengono salvati in modo permanente in `sessionStorage` :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-314">In the following example of a `CounterStateProvider` component, counter data is persisted to `sessionStorage`:</span></span>

```razor
@using Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore

@if (isLoaded)
{
    <CascadingValue Value="@this">
        @ChildContent
    </CascadingValue>
}
else
{
    <p>Loading...</p>
}

@code {
    private bool isLoaded;

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public int CurrentCount { get; set; }

    protected override async Task OnInitializedAsync()
    {
        var result = await ProtectedSessionStore.GetAsync<int>("count");
        currentCount = result.Success ? result.Value : 0;
        isLoaded = true;
    }

    public async Task SaveChangesAsync()
    {
        await ProtectedSessionStore.SetAsync("count", CurrentCount);
    }
}
```

<span data-ttu-id="3b2d9-315">Il `CounterStateProvider` componente gestisce la fase di caricamento non eseguendo il rendering del relativo contenuto figlio fino al completamento del caricamento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-315">The `CounterStateProvider` component handles the loading phase by not rendering its child content until loading is complete.</span></span>

<span data-ttu-id="3b2d9-316">Per utilizzare il `CounterStateProvider` componente, eseguire il wrapping di un'istanza del componente intorno a qualsiasi altro componente che richiede l'accesso allo stato del contatore.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-316">To use the `CounterStateProvider` component, wrap an instance of the component around any other component that requires access to the counter state.</span></span> <span data-ttu-id="3b2d9-317">Per rendere lo stato accessibile a tutti i componenti di un'app, eseguire il wrapping del `CounterStateProvider` componente intorno all'oggetto <xref:Microsoft.AspNetCore.Components.Routing.Router> nel `App` componente ( `App.razor` ):</span><span class="sxs-lookup"><span data-stu-id="3b2d9-317">To make the state accessible to all components in an app, wrap the `CounterStateProvider` component around the <xref:Microsoft.AspNetCore.Components.Routing.Router> in the `App` component (`App.razor`):</span></span>

```razor
<CounterStateProvider>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<span data-ttu-id="3b2d9-318">I componenti di cui è stato eseguito il wrapper ricevono e possono modificare lo stato del contatore permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-318">Wrapped components receive and can modify the persisted counter state.</span></span> <span data-ttu-id="3b2d9-319">Il `Counter` componente seguente implementa il modello:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-319">The following `Counter` component implements the pattern:</span></span>

```razor
@page "/counter"

<p>Current count: <strong>@CounterStateProvider.CurrentCount</strong></p>
<button @onclick="IncrementCount">Increment</button>

@code {
    [CascadingParameter]
    private CounterStateProvider CounterStateProvider { get; set; }

    private async Task IncrementCount()
    {
        CounterStateProvider.CurrentCount++;
        await CounterStateProvider.SaveChangesAsync();
    }
}
```

<span data-ttu-id="3b2d9-320">Il componente precedente non è necessario per interagire con `ProtectedBrowserStorage` , né gestire una fase di "caricamento".</span><span class="sxs-lookup"><span data-stu-id="3b2d9-320">The preceding component isn't required to interact with `ProtectedBrowserStorage`, nor does it deal with a "loading" phase.</span></span>

<span data-ttu-id="3b2d9-321">Per gestire il prerendering come descritto in precedenza, `CounterStateProvider` può essere modificato in modo che tutti i componenti che utilizzano i dati del contatore funzionino automaticamente con il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-321">To deal with prerendering as described earlier, `CounterStateProvider` can be amended so that all of the components that consume the counter data automatically work with prerendering.</span></span> <span data-ttu-id="3b2d9-322">Per ulteriori informazioni, vedere la sezione [handle prerendering](#handle-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-322">For more information, see the [Handle prerendering](#handle-prerendering) section.</span></span>

<span data-ttu-id="3b2d9-323">In generale, è consigliabile usare il modello di *componente padre del provider di stato* :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-323">In general, the *state provider parent component* pattern is recommended:</span></span>

* <span data-ttu-id="3b2d9-324">Per utilizzare lo stato tra molti componenti.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-324">To consume state across many components.</span></span>
* <span data-ttu-id="3b2d9-325">Se è presente un solo oggetto di stato di primo livello da salvare in modo permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-325">If there's just one top-level state object to persist.</span></span>

<span data-ttu-id="3b2d9-326">Per salvare in modo permanente molti oggetti di stato diversi e utilizzare subset diversi di oggetti in posizioni diverse, è preferibile evitare di salvare in modo permanente lo stato a livello globale.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-326">To persist many different state objects and consume different subsets of objects in different places, it's better to avoid persisting state globally.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="protected-browser-storage-experimental-nuget-package"></a><span data-ttu-id="3b2d9-327">Pacchetto NuGet sperimentale di archiviazione del browser protetto</span><span class="sxs-lookup"><span data-stu-id="3b2d9-327">Protected Browser Storage experimental NuGet package</span></span>

<span data-ttu-id="3b2d9-328">ASP.NET Core archiviazione del browser protetta sfrutta la [protezione dei dati ASP.NET Core](xref:security/data-protection/introduction) per [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) e [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-328">ASP.NET Core Protected Browser Storage leverages [ASP.NET Core Data Protection](xref:security/data-protection/introduction) for [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) and [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage).</span></span>

> [!WARNING]
> <span data-ttu-id="3b2d9-329">`Microsoft.AspNetCore.ProtectedBrowserStorage` è un pacchetto sperimentale non supportato non adatto per l'uso in ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-329">`Microsoft.AspNetCore.ProtectedBrowserStorage` is an unsupported, experimental package unsuitable for production use.</span></span>
>
> <span data-ttu-id="3b2d9-330">Il pacchetto è disponibile solo per l'uso in app ASP.NET Core 3,1 Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-330">The package is only available for use in ASP.NET Core 3.1 Blazor Server apps.</span></span>

### <a name="configuration"></a><span data-ttu-id="3b2d9-331">Configurazione</span><span class="sxs-lookup"><span data-stu-id="3b2d9-331">Configuration</span></span>

1. <span data-ttu-id="3b2d9-332">Aggiungere un riferimento al pacchetto a [`Microsoft.AspNetCore.ProtectedBrowserStorage`](https://www.nuget.org/packages/Microsoft.AspNetCore.ProtectedBrowserStorage) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-332">Add a package reference to [`Microsoft.AspNetCore.ProtectedBrowserStorage`](https://www.nuget.org/packages/Microsoft.AspNetCore.ProtectedBrowserStorage).</span></span>
1. <span data-ttu-id="3b2d9-333">Nel `Pages/_Host.cshtml` file aggiungere il seguente script all'interno del tag di chiusura `</body>` :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-333">In the `Pages/_Host.cshtml` file, add the following script inside the closing `</body>` tag:</span></span>

   ```cshtml
   <script src="_content/Microsoft.AspNetCore.ProtectedBrowserStorage/protectedBrowserStorage.js"></script>
   ```

1. <span data-ttu-id="3b2d9-334">In `Startup.ConfigureServices` chiamare `AddProtectedBrowserStorage` per aggiungere `localStorage` e `sessionStorage` servizi alla raccolta di servizi:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-334">In `Startup.ConfigureServices`, call `AddProtectedBrowserStorage` to add `localStorage` and `sessionStorage` services to the service collection:</span></span>

   ```csharp
   services.AddProtectedBrowserStorage();
   ```

### <a name="save-and-load-data-within-a-component"></a><span data-ttu-id="3b2d9-335">Salvare e caricare i dati all'interno di un componente</span><span class="sxs-lookup"><span data-stu-id="3b2d9-335">Save and load data within a component</span></span>

<span data-ttu-id="3b2d9-336">In tutti i componenti che richiedono il caricamento o il salvataggio dei dati nell'archiviazione del browser, utilizzare la [`@inject`](xref:mvc/views/razor#inject) direttiva per inserire un'istanza di uno dei seguenti elementi:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-336">In any component that requires loading or saving data to browser storage, use the [`@inject`](xref:mvc/views/razor#inject) directive to inject an instance of either of the following:</span></span>

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

<span data-ttu-id="3b2d9-337">La scelta dipende dal percorso di archiviazione del browser che si desidera utilizzare.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-337">The choice depends on which browser storage location you wish to use.</span></span> <span data-ttu-id="3b2d9-338">Nell'esempio seguente `sessionStorage` viene usato:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-338">In the following example, `sessionStorage` is used:</span></span>

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore
```

<span data-ttu-id="3b2d9-339">L' `@using` istruzione può essere inserita in un `_Imports.razor` file anziché nel componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-339">The `@using` statement can be placed into an `_Imports.razor` file instead of in the component.</span></span> <span data-ttu-id="3b2d9-340">L'uso del `_Imports.razor` file rende lo spazio dei nomi disponibile a segmenti più grandi dell'app o dell'intera app.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-340">Use of the `_Imports.razor` file makes the namespace available to larger segments of the app or the whole app.</span></span>

<span data-ttu-id="3b2d9-341">Per salvare in modo permanente il `currentCount` valore nel `Counter` componente di un'app in base al Blazor Server modello di progetto, modificare il `IncrementCount` metodo in modo da usare `ProtectedSessionStore.SetAsync` :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-341">To persist the `currentCount` value in the `Counter` component of an app based on the Blazor Server project template, modify the `IncrementCount` method to use `ProtectedSessionStore.SetAsync`:</span></span>

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

<span data-ttu-id="3b2d9-342">Nelle app più grandi e più realistiche, l'archiviazione dei singoli campi è uno scenario improbabile.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-342">In larger, more realistic apps, storage of individual fields is an unlikely scenario.</span></span> <span data-ttu-id="3b2d9-343">È più probabile che le app memorizzino interi oggetti modello che includono lo stato complesso.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-343">Apps are more likely to store entire model objects that include complex state.</span></span> <span data-ttu-id="3b2d9-344">`ProtectedSessionStore` serializza e deserializza automaticamente i dati JSON.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-344">`ProtectedSessionStore` automatically serializes and deserializes JSON data.</span></span>

<span data-ttu-id="3b2d9-345">Nell'esempio di codice precedente i `currentCount` dati vengono archiviati come `sessionStorage['count']` nel browser dell'utente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-345">In the preceding code example, the `currentCount` data is stored as `sessionStorage['count']` in the user's browser.</span></span> <span data-ttu-id="3b2d9-346">I dati non vengono archiviati in testo normale ma sono invece protetti tramite la protezione dei dati ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-346">The data isn't stored in plain text but rather is protected using ASP.NET Core Data Protection.</span></span> <span data-ttu-id="3b2d9-347">I dati crittografati possono essere controllati se `sessionStorage['count']` viene valutato nella console per sviluppatori del browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-347">The encrypted data can be inspected if `sessionStorage['count']` is evaluated in the browser's developer console.</span></span>

<span data-ttu-id="3b2d9-348">Per ripristinare i `currentCount` dati se l'utente torna al `Counter` componente in un secondo momento, anche se si trovano in un circuito completamente nuovo, usare `ProtectedSessionStore.GetAsync` :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-348">To recover the `currentCount` data if the user returns to the `Counter` component later, including if they're on an entirely new circuit, use `ProtectedSessionStore.GetAsync`:</span></span>

```csharp
protected override async Task OnInitializedAsync()
{
    currentCount = await ProtectedSessionStore.GetAsync<int>("count");
}
```

<span data-ttu-id="3b2d9-349">Se i parametri del componente includono lo stato di navigazione, chiamare `ProtectedSessionStore.GetAsync` e assegnare il risultato in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> , non <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-349">If the component's parameters include navigation state, call `ProtectedSessionStore.GetAsync` and assign the result in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A>, not <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>.</span></span> <span data-ttu-id="3b2d9-350"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene chiamato una sola volta al momento della creazione di un'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-350"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> is only called once when the component is first instantiated.</span></span> <span data-ttu-id="3b2d9-351"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> non viene chiamato di nuovo in un secondo momento se l'utente passa a un URL diverso rimanendo nella stessa pagina.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-351"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> isn't called again later if the user navigates to a different URL while remaining on the same page.</span></span> <span data-ttu-id="3b2d9-352">Per altre informazioni, vedere <xref:blazor/components/lifecycle>.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-352">For more information, see <xref:blazor/components/lifecycle>.</span></span>

> [!WARNING]
> <span data-ttu-id="3b2d9-353">Gli esempi in questa sezione funzionano solo se per il server non è abilitato il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-353">The examples in this section only work if the server doesn't have prerendering enabled.</span></span> <span data-ttu-id="3b2d9-354">Con il prerendering abilitato, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-354">With prerendering enabled, an error is generated explaining that JavaScript interop calls cannot be issued because the component is being prerendered.</span></span>
>
> <span data-ttu-id="3b2d9-355">Disabilitare il prerendering o aggiungere altro codice per lavorare con il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-355">Either disable prerendering or add additional code to work with prerendering.</span></span> <span data-ttu-id="3b2d9-356">Per ulteriori informazioni sulla scrittura di codice che funziona con il prerendering, vedere la sezione [handle prerendering](#handle-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-356">To learn more about writing code that works with prerendering, see the [Handle prerendering](#handle-prerendering) section.</span></span>

### <a name="handle-the-loading-state"></a><span data-ttu-id="3b2d9-357">Gestire lo stato di caricamento</span><span class="sxs-lookup"><span data-stu-id="3b2d9-357">Handle the loading state</span></span>

<span data-ttu-id="3b2d9-358">Poiché l'accesso all'archiviazione del browser avviene in modo asincrono tramite una connessione di rete, è sempre necessario un certo periodo di tempo prima che i dati vengano caricati e disponibili per un componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-358">Since browser storage is accessed asynchronously over a network connection, there's always a period of time before the data is loaded and available to a component.</span></span> <span data-ttu-id="3b2d9-359">Per ottenere risultati ottimali, è possibile eseguire il rendering di un messaggio di stato di caricamento mentre è in corso il caricamento anziché visualizzare dati vuoti o predefiniti.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-359">For the best results, render a loading-state message while loading is in progress instead of displaying blank or default data.</span></span>

<span data-ttu-id="3b2d9-360">Un approccio consiste nel rilevare se i dati sono `null` , il che significa che i dati sono ancora in corso di caricamento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-360">One approach is to track whether the data is `null`, which means that the data is still loading.</span></span> <span data-ttu-id="3b2d9-361">Nel componente predefinito `Counter` , il conteggio viene mantenuto in un oggetto `int` .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-361">In the default `Counter` component, the count is held in an `int`.</span></span> <span data-ttu-id="3b2d9-362">[Rendere `currentCount` Nullable](/dotnet/csharp/language-reference/builtin-types/nullable-value-types) aggiungendo un punto interrogativo ( `?` ) al tipo ( `int` ):</span><span class="sxs-lookup"><span data-stu-id="3b2d9-362">[Make `currentCount` nullable](/dotnet/csharp/language-reference/builtin-types/nullable-value-types) by adding a question mark (`?`) to the type (`int`):</span></span>

```csharp
private int? currentCount;
```

<span data-ttu-id="3b2d9-363">Anziché visualizzare in modo non condizionale il numero e il **`Increment`** pulsante, scegliere di visualizzare questi elementi solo se i dati vengono caricati:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-363">Instead of unconditionally displaying the count and **`Increment`** button, choose to display these elements only if the data is loaded:</span></span>

```razor
@if (currentCount.HasValue)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}
```

### <a name="handle-prerendering"></a><span data-ttu-id="3b2d9-364">Gestione preliminare</span><span class="sxs-lookup"><span data-stu-id="3b2d9-364">Handle prerendering</span></span>

<span data-ttu-id="3b2d9-365">Durante il prerendering:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-365">During prerendering:</span></span>

* <span data-ttu-id="3b2d9-366">Una connessione interattiva al browser dell'utente non esiste.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-366">An interactive connection to the user's browser doesn't exist.</span></span>
* <span data-ttu-id="3b2d9-367">Il browser non dispone ancora di una pagina in cui è possibile eseguire il codice JavaScript.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-367">The browser doesn't yet have a page in which it can run JavaScript code.</span></span>

<span data-ttu-id="3b2d9-368">`localStorage` o `sessionStorage` non sono disponibili durante il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-368">`localStorage` or `sessionStorage` aren't available during prerendering.</span></span> <span data-ttu-id="3b2d9-369">Se il componente tenta di interagire con l'archiviazione, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-369">If the component attempts to interact with storage, an error is generated explaining that JavaScript interop calls cannot be issued because the component is being prerendered.</span></span>

<span data-ttu-id="3b2d9-370">Un modo per risolvere l'errore consiste nel disabilitare il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-370">One way to resolve the error is to disable prerendering.</span></span> <span data-ttu-id="3b2d9-371">Questa è in genere la scelta migliore se l'app usa in modo intensivo l'archiviazione basata su browser.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-371">This is usually the best choice if the app makes heavy use of browser-based storage.</span></span> <span data-ttu-id="3b2d9-372">Il prerendering aggiunge complessità e non è vantaggioso per l'app perché l'app non può prerenderizzare contenuti utili fino a quando non `localStorage` `sessionStorage` sono disponibili o.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-372">Prerendering adds complexity and doesn't benefit the app because the app can't prerender any useful content until `localStorage` or `sessionStorage` are available.</span></span>

<span data-ttu-id="3b2d9-373">Per disabilitare il prerendering, aprire il `Pages/_Host.cshtml` file e modificare l' `render-mode` attributo dell' [Helper tag dei componenti](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) in <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-373">To disable prerendering, open the `Pages/_Host.cshtml` file and change the `render-mode` attribute of the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) to <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server>:</span></span>

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

<span data-ttu-id="3b2d9-374">Il prerendering può essere utile per altre pagine che non utilizzano `localStorage` o `sessionStorage` .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-374">Prerendering might be useful for other pages that don't use `localStorage` or `sessionStorage`.</span></span> <span data-ttu-id="3b2d9-375">Per mantenere il prerendering, rinviare l'operazione di caricamento finché il browser non è connesso al circuito.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-375">To retain prerendering, defer the loading operation until the browser is connected to the circuit.</span></span> <span data-ttu-id="3b2d9-376">Di seguito è riportato un esempio per l'archiviazione di un valore del contatore:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-376">The following is an example for storing a counter value:</span></span>

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedLocalStorage ProtectedLocalStore

@if (isConnected)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}

@code {
    private int? currentCount;
    private bool isConnected = false;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            isConnected = true;
            await LoadStateAsync();
            StateHasChanged();
        }
    }

    private async Task LoadStateAsync()
    {
        currentCount = await ProtectedLocalStore.GetAsync<int>("count");
    }

    private async Task IncrementCount()
    {
        currentCount++;
        await ProtectedLocalStore.SetAsync("count", currentCount);
    }
}
```

### <a name="factor-out-the-state-preservation-to-a-common-location"></a><span data-ttu-id="3b2d9-377">Fattorizzare la conservazione dello stato in una posizione comune</span><span class="sxs-lookup"><span data-stu-id="3b2d9-377">Factor out the state preservation to a common location</span></span>

<span data-ttu-id="3b2d9-378">Se molti componenti si basano sull'archiviazione basata su browser, la reimplementazione del codice del provider di stato più volte crea una duplicazione del codice.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-378">If many components rely on browser-based storage, re-implementing state provider code many times creates code duplication.</span></span> <span data-ttu-id="3b2d9-379">Un'opzione per evitare la duplicazione del codice consiste nel creare un *componente padre del provider di stato* che incapsula la logica del provider di stato.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-379">One option for avoiding code duplication is to create a *state provider parent component* that encapsulates the state provider logic.</span></span> <span data-ttu-id="3b2d9-380">I componenti figlio possono funzionare con i dati persistenti senza considerare il meccanismo di persistenza dello stato.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-380">Child components can work with persisted data without regard to the state persistence mechanism.</span></span>

<span data-ttu-id="3b2d9-381">Nell'esempio seguente di un `CounterStateProvider` componente, i dati del contatore vengono salvati in modo permanente in `sessionStorage` :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-381">In the following example of a `CounterStateProvider` component, counter data is persisted to `sessionStorage`:</span></span>

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore

@if (isLoaded)
{
    <CascadingValue Value="@this">
        @ChildContent
    </CascadingValue>
}
else
{
    <p>Loading...</p>
}

@code {
    private bool isLoaded;

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public int CurrentCount { get; set; }

    protected override async Task OnInitializedAsync()
    {
        CurrentCount = await ProtectedSessionStore.GetAsync<int>("count");
        isLoaded = true;
    }

    public async Task SaveChangesAsync()
    {
        await ProtectedSessionStore.SetAsync("count", CurrentCount);
    }
}
```

<span data-ttu-id="3b2d9-382">Il `CounterStateProvider` componente gestisce la fase di caricamento non eseguendo il rendering del relativo contenuto figlio fino al completamento del caricamento.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-382">The `CounterStateProvider` component handles the loading phase by not rendering its child content until loading is complete.</span></span>

<span data-ttu-id="3b2d9-383">Per utilizzare il `CounterStateProvider` componente, eseguire il wrapping di un'istanza del componente intorno a qualsiasi altro componente che richiede l'accesso allo stato del contatore.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-383">To use the `CounterStateProvider` component, wrap an instance of the component around any other component that requires access to the counter state.</span></span> <span data-ttu-id="3b2d9-384">Per rendere lo stato accessibile a tutti i componenti di un'app, eseguire il wrapping del `CounterStateProvider` componente intorno all'oggetto <xref:Microsoft.AspNetCore.Components.Routing.Router> nel `App` componente ( `App.razor` ):</span><span class="sxs-lookup"><span data-stu-id="3b2d9-384">To make the state accessible to all components in an app, wrap the `CounterStateProvider` component around the <xref:Microsoft.AspNetCore.Components.Routing.Router> in the `App` component (`App.razor`):</span></span>

```razor
<CounterStateProvider>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<span data-ttu-id="3b2d9-385">I componenti di cui è stato eseguito il wrapper ricevono e possono modificare lo stato del contatore permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-385">Wrapped components receive and can modify the persisted counter state.</span></span> <span data-ttu-id="3b2d9-386">Il `Counter` componente seguente implementa il modello:</span><span class="sxs-lookup"><span data-stu-id="3b2d9-386">The following `Counter` component implements the pattern:</span></span>

```razor
@page "/counter"

<p>Current count: <strong>@CounterStateProvider.CurrentCount</strong></p>
<button @onclick="IncrementCount">Increment</button>

@code {
    [CascadingParameter]
    private CounterStateProvider CounterStateProvider { get; set; }

    private async Task IncrementCount()
    {
        CounterStateProvider.CurrentCount++;
        await CounterStateProvider.SaveChangesAsync();
    }
}
```

<span data-ttu-id="3b2d9-387">Il componente precedente non è necessario per interagire con `ProtectedBrowserStorage` , né gestire una fase di "caricamento".</span><span class="sxs-lookup"><span data-stu-id="3b2d9-387">The preceding component isn't required to interact with `ProtectedBrowserStorage`, nor does it deal with a "loading" phase.</span></span>

<span data-ttu-id="3b2d9-388">Per gestire il prerendering come descritto in precedenza, `CounterStateProvider` può essere modificato in modo che tutti i componenti che utilizzano i dati del contatore funzionino automaticamente con il prerendering.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-388">To deal with prerendering as described earlier, `CounterStateProvider` can be amended so that all of the components that consume the counter data automatically work with prerendering.</span></span> <span data-ttu-id="3b2d9-389">Per ulteriori informazioni, vedere la sezione [handle prerendering](#handle-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="3b2d9-389">For more information, see the [Handle prerendering](#handle-prerendering) section.</span></span>

<span data-ttu-id="3b2d9-390">In generale, è consigliabile usare il modello di *componente padre del provider di stato* :</span><span class="sxs-lookup"><span data-stu-id="3b2d9-390">In general, *state provider parent component* pattern is recommended:</span></span>

* <span data-ttu-id="3b2d9-391">Per utilizzare lo stato tra molti componenti.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-391">To consume state across many components.</span></span>
* <span data-ttu-id="3b2d9-392">Se è presente un solo oggetto di stato di primo livello da salvare in modo permanente.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-392">If there's just one top-level state object to persist.</span></span>

<span data-ttu-id="3b2d9-393">Per salvare in modo permanente molti oggetti di stato diversi e utilizzare subset diversi di oggetti in posizioni diverse, è preferibile evitare di salvare in modo permanente lo stato a livello globale.</span><span class="sxs-lookup"><span data-stu-id="3b2d9-393">To persist many different state objects and consume different subsets of objects in different places, it's better to avoid persisting state globally.</span></span>

::: moniker-end

<h2 id="in-memory-state-container-service-server"><span data-ttu-id="3b2d9-394">Servizio contenitore stato in memoria</span><span class="sxs-lookup"><span data-stu-id="3b2d9-394">In-memory state container service</span></span></h2>

[!INCLUDE[](~/blazor/includes/state-container.md)]

::: zone-end
