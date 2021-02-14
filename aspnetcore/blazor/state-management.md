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
# <a name="aspnet-core-blazor-state-management"></a>BlazorGestione dello stato ASP.NET Core

::: zone pivot="webassembly"

Lo stato utente creato in un' Blazor WebAssembly app viene mantenuto nella memoria del browser.

Esempi di stato utente contenuti nella memoria del browser includono:

* La gerarchia delle istanze dei componenti e l'output di rendering più recente nell'interfaccia utente sottoposta a rendering.
* Valori dei campi e delle proprietà nelle istanze del componente.
* Dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) .
* Valori impostati tramite chiamate di [interoperabilità JavaScript](xref:blazor/call-javascript-from-dotnet) .

Quando un utente chiude e riapre il browser o ricarica la pagina, lo stato utente contenuto nella memoria del browser viene perso.

> [!NOTE]
> L' [archiviazione del browser protetto](xref:blazor/state-management?pivots=server#aspnet-core-protected-browser-storage) ( <xref:Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage?displayProperty=fullName> spazio dei nomi) si basa sulla protezione dei dati ASP.NET Core ed è supportata solo per le Blazor Server app.

## <a name="persist-state-across-browser-sessions"></a>Mantieni stato tra le sessioni del browser

In genere, mantenere lo stato tra le sessioni del browser in cui gli utenti stanno creando attivamente dati, non semplicemente leggendo i dati già esistenti.

Per mantenere lo stato tra le sessioni del browser, l'app deve salvare in modo permanente i dati in un'altra posizione di archiviazione rispetto alla memoria del browser. La persistenza dello stato non è automatica. È necessario eseguire i passaggi quando si sviluppa l'app per implementare la persistenza dei dati con stato.

La persistenza dei dati è in genere necessaria solo per uno stato di valore elevato che gli utenti hanno impiegato per creare. Negli esempi seguenti lo stato di salvataggio permanente Risparmia tempo o facilita le attività commerciali:

* Web Form in più passaggi: è necessario che un utente immetta nuovamente i dati per diversi passaggi completati di un modulo Web in più passaggi se il relativo stato viene perso. Un utente perde lo stato in questo scenario se si allontana dal modulo e viene restituito in un secondo momento.
* Carrelli acquisti: è possibile mantenere un componente commercialmente importante di un'app che rappresenti potenziali ricavi. Un utente che perde il proprio stato e quindi il carrello della spesa può acquistare un minor numero di prodotti o servizi quando ritornano al sito in un secondo momento.

Un'app può solo salvare *lo stato dell'app*. Non è possibile rendere permanente le interfacce utente, ad esempio le istanze dei componenti e le relative strutture di rendering. I componenti e gli alberi di rendering non sono in genere serializzabili. Per salvare in modo permanente lo stato dell'interfaccia utente, ad esempio i nodi espansi di un controllo di visualizzazione albero, l'app deve usare codice personalizzato per modellare il comportamento dello stato dell'interfaccia utente come stato dell'app serializzabile.

## <a name="where-to-persist-state"></a>Posizione in cui salvare lo stato

Esistono località comuni per lo stato permanente:

* [Archiviazione sul lato server](#server-side-storage-wasm)
* [URL](#url-wasm)
* [Archiviazione del browser](#browser-storage-wasm)
* [Servizio contenitore stato in memoria](#in-memory-state-container-service-wasm)

<h2 id="server-side-storage-wasm">Archiviazione sul lato server</h2>

Per la persistenza permanente dei dati che si estende su più utenti e dispositivi, l'app può usare l'archiviazione indipendente del lato server a cui si accede tramite un'API Web. Le opzioni includono:

* Archiviazione BLOB
* Archiviazione chiave-valore
* Database relazionale
* Archiviazione tabelle

Dopo il salvataggio dei dati, lo stato dell'utente viene mantenuto e disponibile in qualsiasi nuova sessione del browser.

Poiché Blazor WebAssembly le app vengono eseguite interamente nel browser dell'utente, richiedono misure aggiuntive per accedere a sistemi esterni protetti, ad esempio i servizi di archiviazione e i database. Blazor WebAssembly le app sono protette in modo analogo alle applicazioni a pagina singola (Spa). In genere, un'app autentica un utente tramite [OAuth](https://oauth.net) / [OpenID Connect (OIDC)](https://openid.net/connect/) e quindi interagisce con i servizi di archiviazione e i database tramite chiamate API Web a un'app sul lato server. L'app sul lato server media il trasferimento dei dati tra l' Blazor WebAssembly app e il servizio di archiviazione o il database. L'app Blazor WebAssembly mantiene una connessione temporanea all'app sul lato server, mentre l'app sul lato server dispone di una connessione permanente alla risorsa di archiviazione.

Per altre informazioni, vedere le risorse seguenti:

* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>
* Blazor*Sicurezza e Identity* articoli

Per ulteriori informazioni sulle opzioni di archiviazione dei dati di Azure, vedere gli argomenti seguenti:

* [Database di Azure](https://azure.microsoft.com/product-categories/databases/)
* [Documentazione di archiviazione di Azure](/azure/storage/)

<h2 id="url-wasm">URL</h2>

Per i dati temporanei che rappresentano lo stato di navigazione, modellare i dati come parte dell'URL. Esempi di stato utente modellati nell'URL includono:

* ID di un'entità visualizzata.
* Numero di pagina corrente in una griglia di paging.

Il contenuto della barra degli indirizzi del browser viene mantenuto se l'utente ricarica manualmente la pagina.

Per informazioni sulla definizione di modelli URL con la [`@page`](xref:mvc/views/razor#page) direttiva, vedere <xref:blazor/fundamentals/routing> .

<h2 id="browser-storage-wasm">Archiviazione del browser</h2>

Per i dati temporanei che l'utente sta creando attivamente, un percorso di archiviazione di uso comune è costituito dalle [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) raccolte e del browser [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) :

* `localStorage` ha come ambito la finestra del browser. Se l'utente ricarica la pagina o chiude e riapre il browser, lo stato è permanente. Se l'utente apre più schede del browser, lo stato viene condiviso tra le schede. I dati rimangono in `localStorage` fino a quando non vengono cancellati in modo esplicito.
* `sessionStorage` ha come ambito la scheda Esplorazione. Se l'utente ricarica la scheda, lo stato è permanente. Se l'utente chiude la scheda o il browser, lo stato viene perso. Se l'utente apre più schede del browser, ogni scheda ha una propria versione indipendente dei dati.

> [!NOTE]
> `localStorage` e `sessionStorage` possono essere usati nelle Blazor WebAssembly app, ma solo scrivendo codice personalizzato o usando un pacchetto di terze parti.

In genere, `sessionStorage` è più sicuro da usare. `sessionStorage` evita il rischio che un utente apra più schede e riscontri quanto segue:

* Bug nell'archiviazione dello stato tra le schede.
* Comportamento confuso quando una scheda sovrascrive lo stato di altre schede.

`localStorage` è la scelta migliore se l'app deve conservare lo stato tra la chiusura e la riapertura del browser.

> [!WARNING]
> Gli utenti possono visualizzare o manomettere i dati archiviati in `localStorage` e `sessionStorage` .

<h2 id="in-memory-state-container-service-wasm">Servizio contenitore stato in memoria</h2>

[!INCLUDE[](~/blazor/includes/state-container.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* [Salva lo stato dell'app prima di un'operazione di autenticazione](xref:blazor/security/webassembly/additional-scenarios#save-app-state-before-an-authentication-operation)
* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>

::: zone-end

::: zone pivot="server"

Blazor Server è un Framework di app con stato. Nella maggior parte dei casi, l'app mantiene una connessione al server. Lo stato dell'utente viene mantenuto nella memoria del server in un *circuito*. 

Esempi di stato utente contenuti in un circuito includono:

* La gerarchia delle istanze dei componenti e l'output di rendering più recente nell'interfaccia utente sottoposta a rendering.
* Valori dei campi e delle proprietà nelle istanze del componente.
* Dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) che hanno come ambito il circuito.

Lo stato utente potrebbe trovarsi anche in variabili JavaScript nel set di memoria del browser tramite chiamate di [interoperabilità JavaScript](xref:blazor/call-javascript-from-dotnet) .

Se un utente riscontra una perdita di connessione di rete temporanea, Blazor tenta di riconnettere l'utente al circuito originale con lo stato originale. Tuttavia, la riconnessione di un utente al circuito originale nella memoria del server non è sempre possibile:

* Il server non è in grado di mantenere sempre un circuito disconnesso. Il server deve rilasciare un circuito disconnesso dopo un timeout o quando il server è sottoposto a un numero eccessivo di richieste di memoria.
* Negli ambienti di distribuzione con bilanciamento del carico multiserver, i singoli server possono avere esito negativo o essere rimossi automaticamente quando non sono più necessari per gestire il volume complessivo delle richieste. Le richieste di elaborazione del server originale per un utente potrebbero non essere più disponibili quando l'utente tenta di riconnettersi.
* L'utente potrebbe chiudere e riaprire il browser o ricaricare la pagina, che rimuove qualsiasi stato contenuto nella memoria del browser. Ad esempio, i valori delle variabili JavaScript impostati tramite le chiamate di interoperabilità JavaScript vengono persi.

Quando un utente non può essere riconnesso al circuito originale, l'utente riceve un nuovo circuito con uno stato vuoto. Questa operazione equivale a chiudere e riaprire un'app desktop.

## <a name="persist-state-across-circuits"></a>Mantieni lo stato tra circuiti

In genere, mantenere lo stato nei circuiti in cui gli utenti stanno creando attivamente dati, non semplicemente leggendo i dati già esistenti.

Per mantenere lo stato tra circuiti, l'app deve salvare in modo permanente i dati in un'altra posizione di archiviazione rispetto alla memoria del server. La persistenza dello stato non è automatica. È necessario eseguire i passaggi quando si sviluppa l'app per implementare la persistenza dei dati con stato.

La persistenza dei dati è in genere necessaria solo per uno stato di valore elevato che gli utenti hanno impiegato per creare. Negli esempi seguenti lo stato di salvataggio permanente Risparmia tempo o facilita le attività commerciali:

* Web Form in più passaggi: è necessario che un utente immetta nuovamente i dati per diversi passaggi completati di un modulo Web in più passaggi se il relativo stato viene perso. Un utente perde lo stato in questo scenario se si allontana dal modulo e viene restituito in un secondo momento.
* Carrelli acquisti: è possibile mantenere un componente commercialmente importante di un'app che rappresenti potenziali ricavi. Un utente che perde il proprio stato e quindi il carrello della spesa può acquistare un minor numero di prodotti o servizi quando ritornano al sito in un secondo momento.

Un'app può solo salvare *lo stato dell'app*. Non è possibile rendere permanente le interfacce utente, ad esempio le istanze dei componenti e le relative strutture di rendering. I componenti e gli alberi di rendering non sono in genere serializzabili. Per salvare in modo permanente lo stato dell'interfaccia utente, ad esempio i nodi espansi di un controllo di visualizzazione albero, l'app deve usare codice personalizzato per modellare il comportamento dello stato dell'interfaccia utente come stato dell'app serializzabile.

## <a name="where-to-persist-state"></a>Posizione in cui salvare lo stato

Esistono località comuni per lo stato permanente:

* [Archiviazione sul lato server](#server-side-storage-server)
* [URL](#url-server)
* [Archiviazione del browser](#browser-storage-server)
* [Servizio contenitore stato in memoria](#in-memory-state-container-service-server)

<h2 id="server-side-storage-server">Archiviazione sul lato server</h2>

Per la persistenza permanente dei dati che si estende su più utenti e dispositivi, l'app può usare l'archiviazione sul lato server. Le opzioni includono:

* Archiviazione BLOB
* Archiviazione chiave-valore
* Database relazionale
* Archiviazione tabelle

Dopo il salvataggio dei dati, lo stato dell'utente viene mantenuto e disponibile in qualsiasi nuovo circuito.

Per ulteriori informazioni sulle opzioni di archiviazione dei dati di Azure, vedere gli argomenti seguenti:

* [Database di Azure](https://azure.microsoft.com/product-categories/databases/)
* [Documentazione di archiviazione di Azure](/azure/storage/)

<h2 id="url-server">URL</h2>

Per i dati temporanei che rappresentano lo stato di navigazione, modellare i dati come parte dell'URL. Esempi di stato utente modellati nell'URL includono:

* ID di un'entità visualizzata.
* Numero di pagina corrente in una griglia di paging.

Il contenuto della barra degli indirizzi del browser viene mantenuto:

* Se l'utente ricarica manualmente la pagina.
* Se il server Web non è più disponibile e l'utente è costretto a ricaricare la pagina per connettersi a un server diverso.

Per informazioni sulla definizione di modelli URL con la [`@page`](xref:mvc/views/razor#page) direttiva, vedere <xref:blazor/fundamentals/routing> .

<h2 id="browser-storage-server">Archiviazione del browser</h2>

Per i dati temporanei che l'utente sta creando attivamente, un percorso di archiviazione di uso comune è costituito dalle [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) raccolte e del browser [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) :

* `localStorage` ha come ambito la finestra del browser. Se l'utente ricarica la pagina o chiude e riapre il browser, lo stato è permanente. Se l'utente apre più schede del browser, lo stato viene condiviso tra le schede. I dati rimangono in `localStorage` fino a quando non vengono cancellati in modo esplicito.
* `sessionStorage` ha come ambito la scheda Esplorazione. Se l'utente ricarica la scheda, lo stato è permanente. Se l'utente chiude la scheda o il browser, lo stato viene perso. Se l'utente apre più schede del browser, ogni scheda ha una propria versione indipendente dei dati.

In genere, `sessionStorage` è più sicuro da usare. `sessionStorage` evita il rischio che un utente apra più schede e riscontri quanto segue:

* Bug nell'archiviazione dello stato tra le schede.
* Comportamento confuso quando una scheda sovrascrive lo stato di altre schede.

`localStorage` è la scelta migliore se l'app deve conservare lo stato tra la chiusura e la riapertura del browser.

Avvertenze per l'uso dell'archiviazione del browser:

* Analogamente all'utilizzo di un database lato server, il caricamento e il salvataggio dei dati sono asincroni.
* A differenza di un database lato server, l'archiviazione non è disponibile durante il prerendering perché la pagina richiesta non esiste nel browser durante la fase di pre-rendering.
* L'archiviazione di alcuni kilobyte di dati è ragionevole per la permanenza per le Blazor Server app. Oltre alcuni kilobyte, è necessario considerare le implicazioni relative alle prestazioni perché i dati vengono caricati e salvati in rete.
* Gli utenti possono visualizzare o manomettere i dati. [ASP.NET Core protezione dei dati](xref:security/data-protection/introduction) può ridurre il rischio. Ad esempio, [ASP.NET Core archiviazione del browser protetta](#aspnet-core-protected-browser-storage) utilizza ASP.NET Core la protezione dei dati.

I pacchetti NuGet di terze parti forniscono le API per l'uso di `localStorage` e `sessionStorage` . Vale la pena considerare la scelta di un pacchetto che usa in modo trasparente [ASP.NET Core la protezione dei dati](xref:security/data-protection/introduction). La protezione dei dati consente di crittografare i dati archiviati e riduce il rischio potenziale di manomissione dei dati archiviati. Se i dati serializzati in JSON vengono archiviati in testo normale, gli utenti possono visualizzare i dati usando gli strumenti di sviluppo del browser e modificare anche i dati archiviati. La protezione dei dati non è sempre un problema perché i dati potrebbero essere di natura banale. Ad esempio, la lettura o la modifica del colore archiviato di un elemento dell'interfaccia utente non costituisce un rischio di sicurezza significativo per l'utente o l'organizzazione. Evitare di consentire agli utenti di ispezionare o manomettere *i dati sensibili*.

::: moniker range=">= aspnetcore-5.0"

## <a name="aspnet-core-protected-browser-storage"></a>ASP.NET Core archiviazione del browser protetta

ASP.NET Core archiviazione del browser protetta sfrutta la [protezione dei dati ASP.NET Core](xref:security/data-protection/introduction) per [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) e [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) .

> [!NOTE]
> L'archiviazione del browser protetto si basa sulla protezione dei dati ASP.NET Core ed è supportata solo per le Blazor Server app.

### <a name="save-and-load-data-within-a-component"></a>Salvare e caricare i dati all'interno di un componente

In tutti i componenti che richiedono il caricamento o il salvataggio dei dati nell'archiviazione del browser, utilizzare la [`@inject`](xref:mvc/views/razor#inject) direttiva per inserire un'istanza di uno dei seguenti elementi:

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

La scelta dipende dal percorso di archiviazione del browser che si desidera utilizzare. Nell'esempio seguente `sessionStorage` viene usato:

```razor
@using Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore
```

La `@using` direttiva può essere inserita nel file dell'app `_Imports.razor` anziché nel componente. L'uso del `_Imports.razor` file rende lo spazio dei nomi disponibile a segmenti più grandi dell'app o dell'intera app.

Per salvare in modo permanente il `currentCount` valore nel `Counter` componente di un'app in base al Blazor Server modello di progetto, modificare il `IncrementCount` metodo in modo da usare `ProtectedSessionStore.SetAsync` :

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

Nelle app più grandi e più realistiche, l'archiviazione dei singoli campi è uno scenario improbabile. È più probabile che le app memorizzino interi oggetti modello che includono lo stato complesso. `ProtectedSessionStore` serializza e deserializza automaticamente i dati JSON per archiviare oggetti di stato complessi.

Nell'esempio di codice precedente i `currentCount` dati vengono archiviati come `sessionStorage['count']` nel browser dell'utente. I dati non vengono archiviati in testo normale ma sono invece protetti tramite la protezione dei dati ASP.NET Core. I dati crittografati possono essere controllati se `sessionStorage['count']` viene valutato nella console per sviluppatori del browser.

Per ripristinare i `currentCount` dati se l'utente torna al `Counter` componente in un secondo momento, anche se l'utente si trova in un nuovo circuito, usare `ProtectedSessionStore.GetAsync` :

```csharp
protected override async Task OnInitializedAsync()
{
    var result = await ProtectedSessionStore.GetAsync<int>("count");
    currentCount = result.Success ? result.Value : 0;
}
```

Se i parametri del componente includono lo stato di navigazione, chiamare `ProtectedSessionStore.GetAsync` e assegnare un non `null` risultato in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> , non <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> . <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene chiamato una sola volta al momento della creazione di un'istanza del componente. <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> non viene chiamato di nuovo in un secondo momento se l'utente passa a un URL diverso rimanendo nella stessa pagina. Per altre informazioni, vedere <xref:blazor/components/lifecycle>.

> [!WARNING]
> Gli esempi in questa sezione funzionano solo se per il server non è abilitato il prerendering. Con il prerendering abilitato, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.
>
> Disabilitare il prerendering o aggiungere altro codice per lavorare con il prerendering. Per ulteriori informazioni sulla scrittura di codice che funziona con il prerendering, vedere la sezione [handle prerendering](#handle-prerendering) .

### <a name="handle-the-loading-state"></a>Gestire lo stato di caricamento

Poiché l'accesso all'archiviazione del browser avviene in modo asincrono tramite una connessione di rete, è sempre necessario un certo periodo di tempo prima che i dati vengano caricati e disponibili per un componente. Per ottenere risultati ottimali, è possibile eseguire il rendering di un messaggio di stato di caricamento mentre è in corso il caricamento anziché visualizzare dati vuoti o predefiniti.

Un approccio consiste nel rilevare se i dati sono `null` , il che significa che i dati sono ancora in corso di caricamento. Nel componente predefinito `Counter` , il conteggio viene mantenuto in un oggetto `int` . [Rendere `currentCount` Nullable](/dotnet/csharp/language-reference/builtin-types/nullable-value-types) aggiungendo un punto interrogativo ( `?` ) al tipo ( `int` ):

```csharp
private int? currentCount;
```

Anziché visualizzare in modo incondizionato il conteggio e il **`Increment`** pulsante, visualizzare questi elementi solo se i dati vengono caricati controllando <xref:System.Nullable%601.HasValue%2A> :

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

### <a name="handle-prerendering"></a>Gestione preliminare

Durante il prerendering:

* Una connessione interattiva al browser dell'utente non esiste.
* Il browser non dispone ancora di una pagina in cui è possibile eseguire il codice JavaScript.

`localStorage` o `sessionStorage` non sono disponibili durante il prerendering. Se il componente tenta di interagire con l'archiviazione, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.

Un modo per risolvere l'errore consiste nel disabilitare il prerendering. Questa è in genere la scelta migliore se l'app usa in modo intensivo l'archiviazione basata su browser. Il prerendering aggiunge complessità e non è vantaggioso per l'app perché l'app non può prerenderizzare contenuti utili fino a quando non `localStorage` `sessionStorage` sono disponibili o.

Per disabilitare il prerendering, aprire il `Pages/_Host.cshtml` file e modificare l' `render-mode` attributo dell' [Helper tag dei componenti](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) in <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> :

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

Il prerendering può essere utile per altre pagine che non utilizzano `localStorage` o `sessionStorage` . Per mantenere il prerendering, rinviare l'operazione di caricamento finché il browser non è connesso al circuito. Di seguito è riportato un esempio per l'archiviazione di un valore del contatore:

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

### <a name="factor-out-the-state-preservation-to-a-common-location"></a>Fattorizzare la conservazione dello stato in una posizione comune

Se molti componenti si basano sull'archiviazione basata su browser, la reimplementazione del codice del provider di stato più volte crea una duplicazione del codice. Un'opzione per evitare la duplicazione del codice consiste nel creare un *componente padre del provider di stato* che incapsula la logica del provider di stato. I componenti figlio possono funzionare con i dati persistenti senza considerare il meccanismo di persistenza dello stato.

Nell'esempio seguente di un `CounterStateProvider` componente, i dati del contatore vengono salvati in modo permanente in `sessionStorage` :

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

Il `CounterStateProvider` componente gestisce la fase di caricamento non eseguendo il rendering del relativo contenuto figlio fino al completamento del caricamento.

Per utilizzare il `CounterStateProvider` componente, eseguire il wrapping di un'istanza del componente intorno a qualsiasi altro componente che richiede l'accesso allo stato del contatore. Per rendere lo stato accessibile a tutti i componenti di un'app, eseguire il wrapping del `CounterStateProvider` componente intorno all'oggetto <xref:Microsoft.AspNetCore.Components.Routing.Router> nel `App` componente ( `App.razor` ):

```razor
<CounterStateProvider>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

I componenti di cui è stato eseguito il wrapper ricevono e possono modificare lo stato del contatore permanente. Il `Counter` componente seguente implementa il modello:

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

Il componente precedente non è necessario per interagire con `ProtectedBrowserStorage` , né gestire una fase di "caricamento".

Per gestire il prerendering come descritto in precedenza, `CounterStateProvider` può essere modificato in modo che tutti i componenti che utilizzano i dati del contatore funzionino automaticamente con il prerendering. Per ulteriori informazioni, vedere la sezione [handle prerendering](#handle-prerendering) .

In generale, è consigliabile usare il modello di *componente padre del provider di stato* :

* Per utilizzare lo stato tra molti componenti.
* Se è presente un solo oggetto di stato di primo livello da salvare in modo permanente.

Per salvare in modo permanente molti oggetti di stato diversi e utilizzare subset diversi di oggetti in posizioni diverse, è preferibile evitare di salvare in modo permanente lo stato a livello globale.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="protected-browser-storage-experimental-nuget-package"></a>Pacchetto NuGet sperimentale di archiviazione del browser protetto

ASP.NET Core archiviazione del browser protetta sfrutta la [protezione dei dati ASP.NET Core](xref:security/data-protection/introduction) per [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) e [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) .

> [!WARNING]
> `Microsoft.AspNetCore.ProtectedBrowserStorage` è un pacchetto sperimentale non supportato non adatto per l'uso in ambiente di produzione.
>
> Il pacchetto è disponibile solo per l'uso in app ASP.NET Core 3,1 Blazor Server .

### <a name="configuration"></a>Configurazione

1. Aggiungere un riferimento al pacchetto a [`Microsoft.AspNetCore.ProtectedBrowserStorage`](https://www.nuget.org/packages/Microsoft.AspNetCore.ProtectedBrowserStorage) .
1. Nel `Pages/_Host.cshtml` file aggiungere il seguente script all'interno del tag di chiusura `</body>` :

   ```cshtml
   <script src="_content/Microsoft.AspNetCore.ProtectedBrowserStorage/protectedBrowserStorage.js"></script>
   ```

1. In `Startup.ConfigureServices` chiamare `AddProtectedBrowserStorage` per aggiungere `localStorage` e `sessionStorage` servizi alla raccolta di servizi:

   ```csharp
   services.AddProtectedBrowserStorage();
   ```

### <a name="save-and-load-data-within-a-component"></a>Salvare e caricare i dati all'interno di un componente

In tutti i componenti che richiedono il caricamento o il salvataggio dei dati nell'archiviazione del browser, utilizzare la [`@inject`](xref:mvc/views/razor#inject) direttiva per inserire un'istanza di uno dei seguenti elementi:

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

La scelta dipende dal percorso di archiviazione del browser che si desidera utilizzare. Nell'esempio seguente `sessionStorage` viene usato:

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore
```

L' `@using` istruzione può essere inserita in un `_Imports.razor` file anziché nel componente. L'uso del `_Imports.razor` file rende lo spazio dei nomi disponibile a segmenti più grandi dell'app o dell'intera app.

Per salvare in modo permanente il `currentCount` valore nel `Counter` componente di un'app in base al Blazor Server modello di progetto, modificare il `IncrementCount` metodo in modo da usare `ProtectedSessionStore.SetAsync` :

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

Nelle app più grandi e più realistiche, l'archiviazione dei singoli campi è uno scenario improbabile. È più probabile che le app memorizzino interi oggetti modello che includono lo stato complesso. `ProtectedSessionStore` serializza e deserializza automaticamente i dati JSON.

Nell'esempio di codice precedente i `currentCount` dati vengono archiviati come `sessionStorage['count']` nel browser dell'utente. I dati non vengono archiviati in testo normale ma sono invece protetti tramite la protezione dei dati ASP.NET Core. I dati crittografati possono essere controllati se `sessionStorage['count']` viene valutato nella console per sviluppatori del browser.

Per ripristinare i `currentCount` dati se l'utente torna al `Counter` componente in un secondo momento, anche se si trovano in un circuito completamente nuovo, usare `ProtectedSessionStore.GetAsync` :

```csharp
protected override async Task OnInitializedAsync()
{
    currentCount = await ProtectedSessionStore.GetAsync<int>("count");
}
```

Se i parametri del componente includono lo stato di navigazione, chiamare `ProtectedSessionStore.GetAsync` e assegnare il risultato in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> , non <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> . <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene chiamato una sola volta al momento della creazione di un'istanza del componente. <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> non viene chiamato di nuovo in un secondo momento se l'utente passa a un URL diverso rimanendo nella stessa pagina. Per altre informazioni, vedere <xref:blazor/components/lifecycle>.

> [!WARNING]
> Gli esempi in questa sezione funzionano solo se per il server non è abilitato il prerendering. Con il prerendering abilitato, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.
>
> Disabilitare il prerendering o aggiungere altro codice per lavorare con il prerendering. Per ulteriori informazioni sulla scrittura di codice che funziona con il prerendering, vedere la sezione [handle prerendering](#handle-prerendering) .

### <a name="handle-the-loading-state"></a>Gestire lo stato di caricamento

Poiché l'accesso all'archiviazione del browser avviene in modo asincrono tramite una connessione di rete, è sempre necessario un certo periodo di tempo prima che i dati vengano caricati e disponibili per un componente. Per ottenere risultati ottimali, è possibile eseguire il rendering di un messaggio di stato di caricamento mentre è in corso il caricamento anziché visualizzare dati vuoti o predefiniti.

Un approccio consiste nel rilevare se i dati sono `null` , il che significa che i dati sono ancora in corso di caricamento. Nel componente predefinito `Counter` , il conteggio viene mantenuto in un oggetto `int` . [Rendere `currentCount` Nullable](/dotnet/csharp/language-reference/builtin-types/nullable-value-types) aggiungendo un punto interrogativo ( `?` ) al tipo ( `int` ):

```csharp
private int? currentCount;
```

Anziché visualizzare in modo non condizionale il numero e il **`Increment`** pulsante, scegliere di visualizzare questi elementi solo se i dati vengono caricati:

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

### <a name="handle-prerendering"></a>Gestione preliminare

Durante il prerendering:

* Una connessione interattiva al browser dell'utente non esiste.
* Il browser non dispone ancora di una pagina in cui è possibile eseguire il codice JavaScript.

`localStorage` o `sessionStorage` non sono disponibili durante il prerendering. Se il componente tenta di interagire con l'archiviazione, viene generato un errore che indica che non è possibile eseguire chiamate di interoperabilità JavaScript perché è in corso il rendering del componente.

Un modo per risolvere l'errore consiste nel disabilitare il prerendering. Questa è in genere la scelta migliore se l'app usa in modo intensivo l'archiviazione basata su browser. Il prerendering aggiunge complessità e non è vantaggioso per l'app perché l'app non può prerenderizzare contenuti utili fino a quando non `localStorage` `sessionStorage` sono disponibili o.

Per disabilitare il prerendering, aprire il `Pages/_Host.cshtml` file e modificare l' `render-mode` attributo dell' [Helper tag dei componenti](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) in <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> :

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

Il prerendering può essere utile per altre pagine che non utilizzano `localStorage` o `sessionStorage` . Per mantenere il prerendering, rinviare l'operazione di caricamento finché il browser non è connesso al circuito. Di seguito è riportato un esempio per l'archiviazione di un valore del contatore:

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

### <a name="factor-out-the-state-preservation-to-a-common-location"></a>Fattorizzare la conservazione dello stato in una posizione comune

Se molti componenti si basano sull'archiviazione basata su browser, la reimplementazione del codice del provider di stato più volte crea una duplicazione del codice. Un'opzione per evitare la duplicazione del codice consiste nel creare un *componente padre del provider di stato* che incapsula la logica del provider di stato. I componenti figlio possono funzionare con i dati persistenti senza considerare il meccanismo di persistenza dello stato.

Nell'esempio seguente di un `CounterStateProvider` componente, i dati del contatore vengono salvati in modo permanente in `sessionStorage` :

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

Il `CounterStateProvider` componente gestisce la fase di caricamento non eseguendo il rendering del relativo contenuto figlio fino al completamento del caricamento.

Per utilizzare il `CounterStateProvider` componente, eseguire il wrapping di un'istanza del componente intorno a qualsiasi altro componente che richiede l'accesso allo stato del contatore. Per rendere lo stato accessibile a tutti i componenti di un'app, eseguire il wrapping del `CounterStateProvider` componente intorno all'oggetto <xref:Microsoft.AspNetCore.Components.Routing.Router> nel `App` componente ( `App.razor` ):

```razor
<CounterStateProvider>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

I componenti di cui è stato eseguito il wrapper ricevono e possono modificare lo stato del contatore permanente. Il `Counter` componente seguente implementa il modello:

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

Il componente precedente non è necessario per interagire con `ProtectedBrowserStorage` , né gestire una fase di "caricamento".

Per gestire il prerendering come descritto in precedenza, `CounterStateProvider` può essere modificato in modo che tutti i componenti che utilizzano i dati del contatore funzionino automaticamente con il prerendering. Per ulteriori informazioni, vedere la sezione [handle prerendering](#handle-prerendering) .

In generale, è consigliabile usare il modello di *componente padre del provider di stato* :

* Per utilizzare lo stato tra molti componenti.
* Se è presente un solo oggetto di stato di primo livello da salvare in modo permanente.

Per salvare in modo permanente molti oggetti di stato diversi e utilizzare subset diversi di oggetti in posizioni diverse, è preferibile evitare di salvare in modo permanente lo stato a livello globale.

::: moniker-end

<h2 id="in-memory-state-container-service-server">Servizio contenitore stato in memoria</h2>

[!INCLUDE[](~/blazor/includes/state-container.md)]

::: zone-end
