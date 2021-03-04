---
title: BlazorVirtualizzazione componenti ASP.NET Core
author: guardrex
description: Informazioni su come usare la virtualizzazione di componenti nelle Blazor app ASP.NET Core.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 02/26/2021
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
uid: blazor/components/virtualization
ms.openlocfilehash: c81732c29b262e9134a4ff7dab077a4f31db96af
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109819"
---
# <a name="aspnet-core-blazor-component-virtualization"></a>BlazorVirtualizzazione componenti ASP.NET Core

Migliorare le prestazioni percepite del rendering dei componenti utilizzando il Blazor supporto di virtualizzazione incorporato del Framework con il [ `Virtualize` componente](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601). La virtualizzazione è una tecnica che consente di limitare il rendering dell'interfaccia utente solo alle parti attualmente visibili. La virtualizzazione, ad esempio, è utile quando l'applicazione deve eseguire il rendering di un lungo elenco di elementi ed è necessario che sia visibile solo un subset di elementi in un determinato momento.

Usare il componente nei casi seguenti `Virtualize` :

* Rendering di un set di elementi di dati in un ciclo.
* La maggior parte degli elementi non è visibile a causa dello scorrimento.
* Gli elementi di cui è stato eseguito il rendering hanno le stesse dimensioni.

Quando l'utente scorre fino a un punto arbitrario nell' `Virtualize` elenco di elementi del componente, il componente calcola gli elementi visibili da visualizzare. Non viene eseguito il rendering degli elementi non visibili.

Senza virtualizzazione, un elenco tipico potrebbe usare un [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo C# per eseguire il rendering di ogni elemento in un elenco. Nell'esempio seguente:

* `allFlights` è una raccolta di voli aereo.
* Il `FlightSummary` componente Visualizza i dettagli relativi a ogni volo.
* L' [ `@key` attributo della direttiva](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components) conserva la relazione di ogni `FlightSummary` componente con il relativo volo di cui è stato eseguito il rendering da parte del volo `FlightId` .

```razor
<div style="height:500px;overflow-y:scroll">
    @foreach (var flight in allFlights)
    {
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    }
</div>
```

Se la raccolta contiene migliaia di voli, il rendering dei voli richiede molto tempo e gli utenti riscontrano un ritardo dell'interfaccia utente evidente. La maggior parte dei voli non viene sottoposta a rendering perché non rientrano nell'altezza dell' `<div>` elemento.

Anziché eseguire il rendering dell'intero elenco dei voli contemporaneamente, sostituire il [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo nell'esempio precedente con il `Virtualize` componente:

* Specificare `allFlights` come origine dell'elemento fisso su <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType> . Il componente Visualizza solo i voli attualmente visibili `Virtualize` .
* Specificare un contesto per ogni volo con il `Context` parametro. Nell'esempio seguente `flight` viene usato come contesto, che fornisce l'accesso ai membri di ogni volo.

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights" Context="flight">
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    </Virtualize>
</div>
```

Se il contesto non è specificato con il `Context` parametro, utilizzare il valore di `context` nel modello di contenuto dell'elemento per accedere ai membri di ogni volo:

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights">
        <FlightSummary @key="context.FlightId" Details="@context.Summary" />
    </Virtualize>
</div>
```

Il `Virtualize` componente:

* Calcola il numero di elementi di cui eseguire il rendering in base all'altezza del contenitore e alle dimensioni degli elementi di cui è stato eseguito il rendering.
* Ricalcola ed esegue nuovamente il rendering degli elementi quando l'utente scorre.
* Recupera solo la sezione di record da un'API esterna che corrisponde all'area visibile corrente, anziché scaricare tutti i dati dalla raccolta.

Il contenuto dell'elemento per il `Virtualize` componente può includere:

* HTML semplice e Razor codice, come illustrato nell'esempio precedente.
* Uno o più Razor componenti.
* Combinazione di componenti HTML/ Razor e Razor .

## <a name="item-provider-delegate"></a>Delegato del provider di elementi

Se non si desidera caricare tutti gli elementi in memoria, è possibile specificare un metodo delegato del provider di elementi per il parametro del componente <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A?displayProperty=nameWithType> che recupera in modo asincrono gli elementi richiesti su richiesta. Nell'esempio seguente il `LoadEmployees` metodo fornisce gli elementi al `Virtualize` componente:

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <p>
        @employee.FirstName @employee.LastName has the 
        job title of @employee.JobTitle.
    </p>
</Virtualize>
```

Il provider Items riceve un oggetto <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderRequest> che specifica il numero di elementi richiesto a partire da un indice iniziale specifico. Il provider Items recupera quindi gli elementi richiesti da un database o da un altro servizio e li restituisce come <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderResult%601> insieme al numero totale di elementi. Il provider di elementi può scegliere di recuperare gli elementi con ogni richiesta o di memorizzarli nella cache in modo che siano immediatamente disponibili.

Un `Virtualize` componente può accettare solo **un'origine elemento** dai relativi parametri, quindi non tentare di usare contemporaneamente un provider di elementi e assegnare una raccolta a `Items` . Se entrambi sono assegnati, <xref:System.InvalidOperationException> viene generata un'eccezione quando i parametri del componente vengono impostati in fase di esecuzione.

Nell' `LoadEmployees` esempio di metodo seguente vengono caricati i dipendenti da un `EmployeeService` (non visualizzato):

```csharp
private async ValueTask<ItemsProviderResult<Employee>> LoadEmployees(
    ItemsProviderRequest request)
{
    var numEmployees = Math.Min(request.Count, totalEmployees - request.StartIndex);
    var employees = await EmployeesService.GetEmployeesAsync(request.StartIndex, 
        numEmployees, request.CancellationToken);

    return new ItemsProviderResult<Employee>(employees, totalEmployees);
}
```

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> indica al componente di ririchiedere i dati dal relativo oggetto <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A> . Questa operazione è utile quando i dati esterni cambiano. Non è necessario chiamare quando si <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A> Usa <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A> .

## <a name="placeholder"></a>Segnaposto

Poiché la richiesta di elementi da un'origine dati remota potrebbe richiedere del tempo, è possibile eseguire il rendering di un segnaposto con contenuto dell'elemento:

* Usare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Placeholder%2A> ( `<Placeholder>...</Placeholder>` ) per visualizzare il contenuto fino a quando i dati dell'elemento non sono disponibili.
* Utilizzare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemContent%2A?displayProperty=nameWithType> per impostare il modello di elemento per l'elenco.

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <ItemContent>
        <p>
            @employee.FirstName @employee.LastName has the 
            job title of @employee.JobTitle.
        </p>
    </ItemContent>
    <Placeholder>
        <p>
            Loading&hellip;
        </p>
    </Placeholder>
</Virtualize>
```

## <a name="item-size"></a>Dimensioni dell'elemento

L'altezza di ogni elemento in pixel può essere impostata con <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> (impostazione predefinita: 50). Nell'esempio seguente viene modificata l'altezza di ogni elemento dal valore predefinito di 50 pixel a 25 pixel:

```razor
<Virtualize Context="employee" Items="@employees" ItemSize="25">
    ...
</Virtualize>
```

Per impostazione predefinita, il `Virtualize` componente misura le dimensioni di rendering (altezza) dei singoli elementi *dopo che* è stato eseguito il rendering iniziale. Usare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> per fornire in anticipo le dimensioni esatte degli elementi per supportare le prestazioni di rendering iniziali accurate e per garantire la corretta posizione di scorrimento per i ricaricamenti della pagina. Se per impostazione predefinita <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> viene eseguito il rendering di alcuni elementi al di fuori della visualizzazione attualmente visibile, viene attivato un secondo nuovo rendering. Per mantenere correttamente la posizione di scorrimento del browser in un elenco virtualizzato, il rendering iniziale deve essere corretto. In caso contrario, gli utenti potrebbero visualizzare gli elementi non corretti.

## <a name="overscan-count"></a>Conteggio overscan

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> determina il numero di elementi aggiuntivi che vengono sottoposti a rendering prima e dopo l'area visibile. Questa impostazione consente di ridurre la frequenza di rendering durante lo scorrimento. Tuttavia, i valori più elevati generano più elementi sottoposti a rendering nella pagina (impostazione predefinita: 3). Nell'esempio seguente viene modificato il conteggio overscan dal valore predefinito di tre elementi a quattro elementi:

```razor
<Virtualize Context="employee" Items="@employees" OverscanCount="4">
    ...
</Virtualize>
```

## <a name="state-changes"></a>Modifiche stato

Quando si apportano modifiche agli elementi di cui è stato eseguito il rendering dal `Virtualize` componente, chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> per forzare nuovamente la valutazione e il rendering del componente. Per altre informazioni, vedere <xref:blazor/components/rendering>.
