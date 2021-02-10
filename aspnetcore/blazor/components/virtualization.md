---
title: BlazorVirtualizzazione componenti ASP.NET Core
author: guardrex
description: Informazioni su come usare la virtualizzazione di componenti nelle Blazor app ASP.NET Core.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/02/2020
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
ms.openlocfilehash: 5e058c33dcf52faf722b27e361491dd8dbc47dc1
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107025"
---
# <a name="aspnet-core-blazor-component-virtualization"></a>BlazorVirtualizzazione componenti ASP.NET Core

Di [Daniel Roth](https://github.com/danroth27)

Migliorare le prestazioni percepite del rendering dei componenti utilizzando il Blazor supporto della virtualizzazione incorporato del Framework. La virtualizzazione è una tecnica che consente di limitare il rendering dell'interfaccia utente solo alle parti attualmente visibili. La virtualizzazione, ad esempio, è utile quando l'applicazione deve eseguire il rendering di un lungo elenco di elementi ed è necessario che sia visibile solo un subset di elementi in un determinato momento. Blazorfornisce il [ `Virtualize` componente](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601) che può essere usato per aggiungere la virtualizzazione ai componenti di un'app.

Il `Virtualize` componente può essere usato nei casi seguenti:

* Rendering di un set di elementi di dati in un ciclo.
* La maggior parte degli elementi non è visibile a causa dello scorrimento.
* Gli elementi di cui è stato eseguito il rendering hanno esattamente le stesse dimensioni. Quando l'utente scorre fino a un punto arbitrario, il componente può calcolare gli elementi visibili da visualizzare.

Senza virtualizzazione, un elenco tipico potrebbe usare un [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo C# per eseguire il rendering di ogni elemento nell'elenco:

```razor
<div style="height:500px;overflow-y:scroll">
    @foreach (var flight in allFlights)
    {
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    }
</div>
```

Se l'elenco contiene migliaia di elementi, il rendering dell'elenco potrebbe richiedere molto tempo. L'utente può riscontrare un ritardo dell'interfaccia utente evidente.

Anziché eseguire il rendering di ogni elemento nell'elenco in una sola volta, sostituire il [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo con il `Virtualize` componente e specificare un'origine di elemento fisso con <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType> . Vengono visualizzati solo gli elementi attualmente visibili:

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights" Context="flight">
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    </Virtualize>
</div>
```

Se non si specifica un contesto per il componente con `Context` , usare il `context` valore nel modello di contenuto dell'elemento:

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights">
        <FlightSummary @key="context.FlightId" Details="@context.Summary" />
    </Virtualize>
</div>
```

> [!NOTE]
> Il processo di mapping degli oggetti modello a elementi e componenti può essere controllato con l' [`@key`](xref:mvc/views/razor#key) attributo della direttiva. `@key` fa in modo che l'algoritmo diffe garantisca la conservazione di elementi o componenti in base al valore della chiave.
>
> Per altre informazioni, vedere gli articoli seguenti:
>
> * <xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components>
> * [Razor informazioni di riferimento sulla sintassi per ASP.NET Core](xref:mvc/views/razor#key)

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

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> indica al componente di ririchiedere i dati dal relativo oggetto <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A> . Questa operazione è utile quando i dati esterni cambiano. Non è necessario chiamare questo quando si usa <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A> .

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

L'altezza di ogni elemento in pixel può essere impostata con <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> (impostazione predefinita: 50):

```razor
<Virtualize Context="employee" Items="@employees" ItemSize="25">
    ...
</Virtualize>
```

Per impostazione predefinita, il `Virtualize` componente misura le dimensioni effettive del rendering *dopo che* è stato eseguito il rendering iniziale. Usare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> per fornire in anticipo le dimensioni esatte degli elementi per supportare le prestazioni di rendering iniziali accurate e per garantire la corretta posizione di scorrimento per i ricaricamenti della pagina. Se per impostazione predefinita <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> viene eseguito il rendering di alcuni elementi al di fuori della visualizzazione attualmente visibile, viene attivato un secondo nuovo rendering. Per mantenere correttamente la posizione di scorrimento del browser in un elenco virtualizzato, il rendering iniziale deve essere corretto. In caso contrario, gli utenti potrebbero visualizzare gli elementi non corretti. 

## <a name="overscan-count"></a>Conteggio overscan

<xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> determina il numero di elementi aggiuntivi che vengono sottoposti a rendering prima e dopo l'area visibile. Questa impostazione consente di ridurre la frequenza di rendering durante lo scorrimento. Tuttavia, i valori più elevati comportano il rendering di più elementi nella pagina (impostazione predefinita: 3):

```razor
<Virtualize Context="employee" Items="@employees" OverscanCount="4">
    ...
</Virtualize>
```

## <a name="state-changes"></a>Modifiche stato

Quando si apportano modifiche agli elementi di cui è stato eseguito il rendering dal `Virtualize` componente, chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> per forzare nuovamente la valutazione e il rendering del componente. Per altre informazioni, vedere <xref:blazor/components/rendering>.
