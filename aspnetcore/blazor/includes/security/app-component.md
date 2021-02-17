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
ms.openlocfilehash: d632ab0604f81f7b6067d4535b0f5da0afe2e0ad
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551919"
---
Il `App` componente ( `App.razor` ) è simile al `App` componente presente nelle Blazor Server app:

* Il <xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> componente gestisce l'esposizione al <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState> resto dell'app.
* Il <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> componente verifica che l'utente corrente sia autorizzato ad accedere a una pagina specificata o altrimenti esegua il rendering del `RedirectToLogin` componente.
* Il `RedirectToLogin` componente gestisce il reindirizzamento degli utenti non autorizzati alla pagina di accesso.

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="@routeData" 
                DefaultLayout="@typeof(MainLayout)">
                <NotAuthorized>
                    @if (!context.User.Identity.IsAuthenticated)
                    {
                        <RedirectToLogin />
                    }
                    else
                    {
                        <p>
                            You are not authorized to access 
                            this resource.
                        </p>
                    }
                </NotAuthorized>
            </AuthorizeRouteView>
        </Found>
        <NotFound>
            <LayoutView Layout="@typeof(MainLayout)">
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

[!include[](../prefer-exact-matches.md)]
