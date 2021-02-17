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
ms.openlocfilehash: 8772f383a830936881564ca95a7f034ba08a5798
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552630"
---
<span data-ttu-id="e6ee6-101">Il `FetchData` componente Mostra come:</span><span class="sxs-lookup"><span data-stu-id="e6ee6-101">The `FetchData` component shows how to:</span></span>

* <span data-ttu-id="e6ee6-102">Effettuare il provisioning di un token di accesso.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-102">Provision an access token.</span></span>
* <span data-ttu-id="e6ee6-103">Usare il token di accesso per chiamare un'API di risorse protette nell'app *Server* .</span><span class="sxs-lookup"><span data-stu-id="e6ee6-103">Use the access token to call a protected resource API in the *Server* app.</span></span>

<span data-ttu-id="e6ee6-104">La [`@attribute [Authorize]`](xref:mvc/views/razor#attribute) direttiva indica al Blazor WebAssembly sistema di autorizzazione che l'utente deve essere autorizzato per visitare il componente.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-104">The [`@attribute [Authorize]`](xref:mvc/views/razor#attribute) directive indicates to the Blazor WebAssembly authorization system that the user must be authorized in order to visit this component.</span></span> <span data-ttu-id="e6ee6-105">La presenza dell'attributo nell' `Client` app non impedisce che l'API sul server venga chiamata senza credenziali appropriate.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-105">The presence of the attribute in the `Client` app doesn't prevent the API on the server from being called without proper credentials.</span></span> <span data-ttu-id="e6ee6-106">`Server`Per una corretta protezione, l'app deve usare anche negli `[Authorize]` endpoint appropriati.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-106">The `Server` app also must use `[Authorize]` on the appropriate endpoints to correctly protect them.</span></span>

<span data-ttu-id="e6ee6-107"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.IAccessTokenProvider.RequestAccessToken%2A?displayProperty=nameWithType> si occupa della richiesta di un token di accesso che può essere aggiunto alla richiesta per chiamare l'API.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-107"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.IAccessTokenProvider.RequestAccessToken%2A?displayProperty=nameWithType> takes care of requesting an access token that can be added to the request to call the API.</span></span> <span data-ttu-id="e6ee6-108">Se il token è memorizzato nella cache oppure il servizio è in grado di effettuare il provisioning di un nuovo token di accesso senza interazione dell'utente, la richiesta del token viene completata.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-108">If the token is cached or the service is able to provision a new access token without user interaction, the token request succeeds.</span></span> <span data-ttu-id="e6ee6-109">In caso contrario, la richiesta di token ha esito negativo con un oggetto <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> , che viene rilevato in un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-109">Otherwise, the token request fails with an <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException>, which is caught in a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement.</span></span>

<span data-ttu-id="e6ee6-110">Per ottenere il token effettivo da includere nella richiesta, l'app deve verificare che la richiesta abbia avuto esito positivo chiamando [`tokenResult.TryGetToken(out var token)`](xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenResult.TryGetToken%2A) .</span><span class="sxs-lookup"><span data-stu-id="e6ee6-110">In order to obtain the actual token to include in the request, the app must check that the request succeeded by calling [`tokenResult.TryGetToken(out var token)`](xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenResult.TryGetToken%2A).</span></span>

<span data-ttu-id="e6ee6-111">Se la richiesta ha avuto esito positivo, la variabile del token viene popolata con il token di accesso.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-111">If the request was successful, the token variable is populated with the access token.</span></span> <span data-ttu-id="e6ee6-112">La <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessToken.Value?displayProperty=nameWithType> proprietà del token espone la stringa letterale da includere nell' `Authorization` intestazione della richiesta.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-112">The <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessToken.Value?displayProperty=nameWithType> property of the token exposes the literal string to include in the `Authorization` request header.</span></span>

<span data-ttu-id="e6ee6-113">Se la richiesta non è riuscita perché non è stato possibile eseguire il provisioning del token senza l'interazione dell'utente, il risultato del token contiene un URL di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-113">If the request failed because the token couldn't be provisioned without user interaction, the token result contains a redirect URL.</span></span> <span data-ttu-id="e6ee6-114">Se si passa a questo URL, l'utente viene indirizzato alla pagina di accesso e torna alla pagina corrente dopo un'autenticazione corretta.</span><span class="sxs-lookup"><span data-stu-id="e6ee6-114">Navigating to this URL takes the user to the login page and back to the current page after a successful authentication.</span></span>

```razor
@page "/fetchdata"
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using {APP NAMESPACE}.Shared
@attribute [Authorize]
@inject HttpClient Http

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        try
        {
            forecasts = await Http.GetFromJsonAsync<WeatherForecast[]>("WeatherForecast");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
    }
}
```
