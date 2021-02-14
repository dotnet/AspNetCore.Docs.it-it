---
title: Proteggere un' Blazor WebAssembly app ASP.NET Core autonoma con la libreria di autenticazione
author: guardrex
description: Informazioni su come proteggere un' Blazor WebAssembly app autonoma ASP.NET Core con la libreria di autenticazione.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/10/2021
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
uid: blazor/security/webassembly/standalone-with-authentication-library
ms.openlocfilehash: a198606caf55232c221f1d1f1224918d3f87f04c
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280893"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-standalone-app-with-the-authentication-library"></a>Proteggere un' Blazor WebAssembly app ASP.NET Core autonoma con la libreria di autenticazione

*Per Azure Active Directory (AAD) e Azure Active Directory B2C (AAD B2C), non seguire le istruzioni riportate in questo argomento. Vedere gli argomenti di AAD e AAD B2C in questo nodo Sommario.*

Per creare un' [ Blazor WebAssembly app autonoma](xref:blazor/hosting-models#blazor-webassembly) che usa [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) la libreria, seguire le istruzioni per la scelta degli strumenti. Se si aggiunge il supporto per l'autenticazione, vedere le sezioni seguenti di questo articolo per informazioni sulla configurazione e la configurazione dell'app.

> [!NOTE]
> Il Identity provider (IP) deve usare [OpenID Connect (OIDC)](https://openid.net/connect/). Ad esempio, l'indirizzo IP di Facebook non è un provider conforme a OIDC, pertanto le linee guida in questo argomento non funzionano con l'indirizzo IP di Facebook. Per altre informazioni, vedere <xref:blazor/security/webassembly/index#authentication-library>.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Per creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione:

1. Dopo aver scelto il modello **Blazor WebAssembly app** nella finestra di dialogo **Crea un nuovo ASP.NET Core applicazione Web** , selezionare **Cambia** in **autenticazione**.

1. Selezionare **singoli account utente** con l'opzione **Archivia account utente in-app** per usare il sistema di ASP.NET Core [Identity](xref:security/authentication/identity) . Questa selezione aggiunge il supporto per l'autenticazione e non comporta l'archiviazione degli utenti in un database. Le sezioni seguenti di questo articolo forniscono ulteriori dettagli.

# <a name="visual-studio-code--net-core-cli"></a>[Visual Studio Code/Interfaccia della riga di comando di .NET Core](#tab/visual-studio-code+netcore-cli)

Creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione in una cartella vuota. Specificare il `Individual` meccanismo di autenticazione con l' `-au|--auth` opzione per usare il [Identity](xref:security/authentication/identity) sistema di ASP.NET Core. Questa selezione aggiunge il supporto per l'autenticazione e non comporta l'archiviazione degli utenti in un database. Le sezioni seguenti di questo articolo forniscono ulteriori dettagli.

```dotnetcli
dotnet new blazorwasm -au Individual -o {APP NAME}
```

| Segnaposto  | Esempio        |
| ------------ | -------------- |
| `{APP NAME}` | `BlazorSample` |

Il percorso di output specificato con l'opzione `-o|--output` consente di creare una cartella di progetto se non esiste e viene incluso nel nome dell'app.

Per ulteriori informazioni, vedere il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando nella Guida di .NET Core.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Per creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione:

1. Nel passaggio **configurare la nuova Blazor WebAssembly app** selezionare **autenticazione singola (in-app) nell'elenco a** discesa **autenticazione** .

1. L'app viene creata per usare ASP.NET Core [Identity](xref:security/authentication/identity) e non comporta l'archiviazione degli utenti in un database. Le sezioni seguenti di questo articolo forniscono ulteriori dettagli.

---

## <a name="authentication-package"></a>Pacchetto di autenticazione

Quando si crea un'app per usare account utente singoli, l'app riceve automaticamente un riferimento al pacchetto per il [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) pacchetto nel file di progetto dell'app. Il pacchetto fornisce un set di primitive che consentono all'app di autenticare gli utenti e ottenere i token per chiamare le API protette.

Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il pacchetto al file di progetto dell'app:

```xml
<PackageReference 
  Include="Microsoft.AspNetCore.Components.WebAssembly.Authentication" 
  Version="{VERSION}" />
```

Per il segnaposto `{VERSION}` , la versione stabile più recente del pacchetto che corrisponde alla versione del Framework condiviso dell'app è disponibile nella cronologia delle **versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication).

## <a name="authentication-service-support"></a>Supporto del servizio di autenticazione

Il supporto per l'autenticazione degli utenti viene registrato nel contenitore del servizio con il <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A> metodo di estensione fornito dal [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) pacchetto. Questo metodo configura i servizi necessari per l'interazione dell'app con il Identity provider (IP).

Per una nuova app, fornire i valori per `{AUTHORITY}` i `{CLIENT ID}` segnaposto e nella configurazione seguente. Fornire altri valori di configurazione necessari per l'uso con l'indirizzo IP dell'app. L'esempio è per Google, che richiede `PostLogoutRedirectUri` , `RedirectUri` e `ResponseType` . Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il codice e la configurazione seguenti all'app con i valori per i segnaposto e altri valori di configurazione.

`Program.cs`:

```csharp
builder.Services.AddOidcAuthentication(options =>
{
    builder.Configuration.Bind("Local", options.ProviderOptions);
});
```

La configurazione viene fornita dal `wwwroot/appsettings.json` file:

```json
{
  "Local": {
    "Authority": "{AUTHORITY}",
    "ClientId": "{CLIENT ID}"
  }
}
```

Esempio di Google OAuth 2,0 OIDC:

```json
{
  "Local": {
    "Authority": "https://accounts.google.com/",
    "ClientId": "2.......7-e.....................q.apps.googleusercontent.com",
    "PostLogoutRedirectUri": "https://localhost:5001/authentication/logout-callback",
    "RedirectUri": "https://localhost:5001/authentication/login-callback",
    "ResponseType": "id_token"
  }
}
```

L'URI di reindirizzamento ( `https://localhost:5001/authentication/login-callback` ) viene registrato nella [console di Google Apis](https://console.developers.google.com/apis/dashboard) in **Credentials**  >  **`{NAME}`**  >  **Authorized Redirect URI**, dove `{NAME}` è il nome del client dell'app nell'elenco di app **ID client OAuth 2,0** della console di Google APIs.

Il supporto dell'autenticazione per le app autonome viene offerto tramite OpenID Connect (OIDC). Il <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A> metodo accetta un callback per configurare i parametri necessari per autenticare un'app usando OIDC. I valori necessari per la configurazione dell'app possono essere ottenuti dall'IP conforme a OIDC. Ottenere i valori quando si registra l'app, che in genere si verifica nel portale online.

## <a name="access-token-scopes"></a>Ambiti del token di accesso

Il Blazor WebAssembly modello configura automaticamente gli ambiti predefiniti per `openid` e `profile` .

Il Blazor WebAssembly modello non configura automaticamente l'app per richiedere un token di accesso per un'API protetta. Per eseguire il provisioning di un token di accesso come parte del flusso di accesso, aggiungere l'ambito agli ambiti dei token predefiniti di <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.OidcProviderOptions> . Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il codice seguente e configurare l'URI dell'ambito.

`Program.cs`:

```csharp
builder.Services.AddOidcAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultScopes.Add("{SCOPE URI}");
});
```

Per ulteriori informazioni, vedere le sezioni seguenti dell'articolo *scenari aggiuntivi* :

* [Richiedere token di accesso aggiuntivi](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [Connetti token alle richieste in uscita](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)

## <a name="imports-file"></a>Importa file

[!INCLUDE[](~/blazor/includes/security/imports-file-standalone.md)]

## <a name="index-page"></a>Pagina di indice

[!INCLUDE[](~/blazor/includes/security/index-page-authentication.md)]

## <a name="app-component"></a>Componente app

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

## <a name="redirecttologin-component"></a>Componente RedirectToLogin

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

## <a name="logindisplay-component"></a>Componente LoginDisplay

Il `LoginDisplay` componente ( `Shared/LoginDisplay.razor` ) viene sottoposto a rendering nel `MainLayout` componente ( `Shared/MainLayout.razor` ) e gestisce i comportamenti seguenti:

* Per utenti autenticati:
  * Visualizza il nome utente corrente.
  * Offre un pulsante per disconnettersi dall'app.
* Per gli utenti anonimi, offre la possibilità di eseguire l'accesso.

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        Hello, @context.User.Identity.Name!
        <button class="nav-link btn btn-link" @onclick="BeginSignOut">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginSignOut(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```

## <a name="authentication-component"></a>Componente di autenticazione

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/security/webassembly/additional-scenarios>
* [Richieste API Web non autenticate o non autorizzate in un'app con un client predefinito sicuro](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:host-and-deploy/proxy-load-balancer>: Include informazioni aggiuntive su:
  * Uso del middleware intestazioni con inoltro per mantenere le informazioni sullo schema HTTPS tra i server proxy e le reti interne.
  * Scenari e casi d'uso aggiuntivi, tra cui la configurazione manuale dello schema, la modifica del percorso della richiesta per il routing delle richieste corretto e l'inoltro dello schema di richiesta per i proxy inversi Linux e non IIS.
