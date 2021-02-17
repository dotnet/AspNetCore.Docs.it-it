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
ms.openlocfilehash: bdb60608ba4bd99b24e458f0543b1f4226cadf44
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552632"
---
<span data-ttu-id="14252-101">Il `LoginDisplay` componente ( `Shared/LoginDisplay.razor` ) viene sottoposto a rendering nel `MainLayout` componente ( `Shared/MainLayout.razor` ) e gestisce i comportamenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="14252-101">The `LoginDisplay` component (`Shared/LoginDisplay.razor`) is rendered in the `MainLayout` component (`Shared/MainLayout.razor`) and manages the following behaviors:</span></span>

* <span data-ttu-id="14252-102">Per utenti autenticati:</span><span class="sxs-lookup"><span data-stu-id="14252-102">For authenticated users:</span></span>
  * <span data-ttu-id="14252-103">Visualizza il nome utente corrente.</span><span class="sxs-lookup"><span data-stu-id="14252-103">Displays the current username.</span></span>
  * <span data-ttu-id="14252-104">Offre un pulsante per disconnettersi dall'app.</span><span class="sxs-lookup"><span data-stu-id="14252-104">Offers a button to log out of the app.</span></span>
* <span data-ttu-id="14252-105">Per gli utenti anonimi, offre la possibilità di eseguire l'accesso.</span><span class="sxs-lookup"><span data-stu-id="14252-105">For anonymous users, offers the option to log in.</span></span>

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        Hello, @context.User.Identity.Name!
        <button class="nav-link btn btn-link" @onclick="BeginLogout">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginLogout(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```
