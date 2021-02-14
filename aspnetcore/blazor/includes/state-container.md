---
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
ms.openlocfilehash: 76dbf3cae1c264fa474101bc4398da28f45a1c10
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100254378"
---
I componenti annidati in genere associano i dati mediante *Binding concatenato* come descritto in <xref:blazor/components/data-binding> . I componenti annidati e non annidati possono condividere l'accesso ai dati usando un contenitore di stato in memoria registrato. Una classe di contenitori di stato personalizzata può usare un'assegnabile <xref:System.Action> per notificare ai componenti in parti diverse dell'app le modifiche di stato. Nell'esempio seguente:

* Una coppia di componenti usa un contenitore di stato per tenere traccia di una proprietà.
* I componenti dell'esempio sono annidati, ma l'annidamento non è necessario per il funzionamento di questo approccio.

`StateContainer.cs`:

```csharp
public class StateContainer
{
    public string Property { get; set; } = "Initial value from StateContainer";

    public event Action OnChange;

    public void SetProperty(string value)
    {
        Property = value;
        NotifyStateChanged();
    }

    private void NotifyStateChanged() => OnChange?.Invoke();
}
```

In `Program.Main` ( Blazor WebAssembly ):

```csharp
builder.Services.AddSingleton<StateContainer>();
```

In `Startup.ConfigureServices` ( Blazor Server ):

```csharp
services.AddSingleton<StateContainer>();
```

`Pages/Component1.razor`:

```razor
@page "/Component1"
@inject StateContainer StateContainer
@implements IDisposable

<h1>Component 1</h1>

<p>Component 1 Property: <b>@StateContainer.Property</b></p>

<p>
    <button @onclick="ChangePropertyValue">Change Property from Component 1</button>
</p>

<Component2 />

@code {
    protected override void OnInitialized()
    {
        StateContainer.OnChange += StateHasChanged;
    }

    private void ChangePropertyValue()
    {
        StateContainer.SetProperty($"New value set in Component 1: {DateTime.Now}");
    }

    public void Dispose()
    {
        StateContainer.OnChange -= StateHasChanged;
    }
}
```

`Shared/Component2.razor`:

```razor
@inject StateContainer StateContainer
@implements IDisposable

<h2>Component 2</h2>

<p>Component 2 Property: <b>@StateContainer.Property</b></p>

<p>
    <button @onclick="ChangePropertyValue">Change Property from Component 2</button>
</p>

@code {
    protected override void OnInitialized()
    {
        StateContainer.OnChange += StateHasChanged;
    }

    private void ChangePropertyValue()
    {
        StateContainer.SetProperty($"New value set in Component 2: {DateTime.Now}");
    }

    public void Dispose()
    {
        StateContainer.OnChange -= StateHasChanged;
    }
}
```

I componenti precedenti implementano <xref:System.IDisposable> e i `OnChange` delegati hanno annullato la sottoscrizione nei `Dispose` metodi, che vengono chiamati dal framework quando i componenti vengono eliminati. Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.
