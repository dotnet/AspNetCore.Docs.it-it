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
# <a name="secure-an-aspnet-core-blazor-webassembly-standalone-app-with-the-authentication-library"></a><span data-ttu-id="a8711-103">Proteggere un' Blazor WebAssembly app ASP.NET Core autonoma con la libreria di autenticazione</span><span class="sxs-lookup"><span data-stu-id="a8711-103">Secure an ASP.NET Core Blazor WebAssembly standalone app with the Authentication library</span></span>

<span data-ttu-id="a8711-104">*Per Azure Active Directory (AAD) e Azure Active Directory B2C (AAD B2C), non seguire le istruzioni riportate in questo argomento. Vedere gli argomenti di AAD e AAD B2C in questo nodo Sommario.*</span><span class="sxs-lookup"><span data-stu-id="a8711-104">*For Azure Active Directory (AAD) and Azure Active Directory B2C (AAD B2C), don't follow the guidance in this topic. See the AAD and AAD B2C topics in this table of contents node.*</span></span>

<span data-ttu-id="a8711-105">Per creare un' [ Blazor WebAssembly app autonoma](xref:blazor/hosting-models#blazor-webassembly) che usa [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) la libreria, seguire le istruzioni per la scelta degli strumenti.</span><span class="sxs-lookup"><span data-stu-id="a8711-105">To create a [standalone Blazor WebAssembly app](xref:blazor/hosting-models#blazor-webassembly) that uses [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) library, follow the guidance for your choice of tooling.</span></span> <span data-ttu-id="a8711-106">Se si aggiunge il supporto per l'autenticazione, vedere le sezioni seguenti di questo articolo per informazioni sulla configurazione e la configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="a8711-106">If adding support for authentication, see the following sections of this article for guidance on setting up and configuring the app.</span></span>

> [!NOTE]
> <span data-ttu-id="a8711-107">Il Identity provider (IP) deve usare [OpenID Connect (OIDC)](https://openid.net/connect/).</span><span class="sxs-lookup"><span data-stu-id="a8711-107">The Identity Provider (IP) must use [OpenID Connect (OIDC)](https://openid.net/connect/).</span></span> <span data-ttu-id="a8711-108">Ad esempio, l'indirizzo IP di Facebook non è un provider conforme a OIDC, pertanto le linee guida in questo argomento non funzionano con l'indirizzo IP di Facebook.</span><span class="sxs-lookup"><span data-stu-id="a8711-108">For example, Facebook's IP isn't an OIDC-compliant provider, so the guidance in this topic doesn't work with the Facebook IP.</span></span> <span data-ttu-id="a8711-109">Per altre informazioni, vedere <xref:blazor/security/webassembly/index#authentication-library>.</span><span class="sxs-lookup"><span data-stu-id="a8711-109">For more information, see <xref:blazor/security/webassembly/index#authentication-library>.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a8711-110">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a8711-110">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="a8711-111">Per creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione:</span><span class="sxs-lookup"><span data-stu-id="a8711-111">To create a new Blazor WebAssembly project with an authentication mechanism:</span></span>

1. <span data-ttu-id="a8711-112">Dopo aver scelto il modello **Blazor WebAssembly app** nella finestra di dialogo **Crea un nuovo ASP.NET Core applicazione Web** , selezionare **Cambia** in **autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="a8711-112">After choosing the **Blazor WebAssembly App** template in the **Create a new ASP.NET Core Web Application** dialog, select **Change** under **Authentication**.</span></span>

1. <span data-ttu-id="a8711-113">Selezionare **singoli account utente** con l'opzione **Archivia account utente in-app** per usare il sistema di ASP.NET Core [Identity](xref:security/authentication/identity) .</span><span class="sxs-lookup"><span data-stu-id="a8711-113">Select **Individual User Accounts** with the **Store user accounts in-app** option to use ASP.NET Core's [Identity](xref:security/authentication/identity) system.</span></span> <span data-ttu-id="a8711-114">Questa selezione aggiunge il supporto per l'autenticazione e non comporta l'archiviazione degli utenti in un database.</span><span class="sxs-lookup"><span data-stu-id="a8711-114">This selection adds authentication support and doesn't result in storing users in a database.</span></span> <span data-ttu-id="a8711-115">Le sezioni seguenti di questo articolo forniscono ulteriori dettagli.</span><span class="sxs-lookup"><span data-stu-id="a8711-115">The following sections of this article provide further details.</span></span>

# <a name="visual-studio-code--net-core-cli"></a>[<span data-ttu-id="a8711-116">Visual Studio Code/Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a8711-116">Visual Studio Code / .NET Core CLI</span></span>](#tab/visual-studio-code+netcore-cli)

<span data-ttu-id="a8711-117">Creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione in una cartella vuota.</span><span class="sxs-lookup"><span data-stu-id="a8711-117">Create a new Blazor WebAssembly project with an authentication mechanism in an empty folder.</span></span> <span data-ttu-id="a8711-118">Specificare il `Individual` meccanismo di autenticazione con l' `-au|--auth` opzione per usare il [Identity](xref:security/authentication/identity) sistema di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a8711-118">Specify the `Individual` authentication mechanism with the `-au|--auth` option to use ASP.NET Core's [Identity](xref:security/authentication/identity) system.</span></span> <span data-ttu-id="a8711-119">Questa selezione aggiunge il supporto per l'autenticazione e non comporta l'archiviazione degli utenti in un database.</span><span class="sxs-lookup"><span data-stu-id="a8711-119">This selection adds authentication support and doesn't result in storing users in a database.</span></span> <span data-ttu-id="a8711-120">Le sezioni seguenti di questo articolo forniscono ulteriori dettagli.</span><span class="sxs-lookup"><span data-stu-id="a8711-120">The following sections of this article provide further details.</span></span>

```dotnetcli
dotnet new blazorwasm -au Individual -o {APP NAME}
```

| <span data-ttu-id="a8711-121">Segnaposto</span><span class="sxs-lookup"><span data-stu-id="a8711-121">Placeholder</span></span>  | <span data-ttu-id="a8711-122">Esempio</span><span class="sxs-lookup"><span data-stu-id="a8711-122">Example</span></span>        |
| ------------ | -------------- |
| `{APP NAME}` | `BlazorSample` |

<span data-ttu-id="a8711-123">Il percorso di output specificato con l'opzione `-o|--output` consente di creare una cartella di progetto se non esiste e viene incluso nel nome dell'app.</span><span class="sxs-lookup"><span data-stu-id="a8711-123">The output location specified with the `-o|--output` option creates a project folder if it doesn't exist and becomes part of the app's name.</span></span>

<span data-ttu-id="a8711-124">Per ulteriori informazioni, vedere il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando nella Guida di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="a8711-124">For more information, see the [`dotnet new`](/dotnet/core/tools/dotnet-new) command in the .NET Core Guide.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a8711-125">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a8711-125">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="a8711-126">Per creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione:</span><span class="sxs-lookup"><span data-stu-id="a8711-126">To create a new Blazor WebAssembly project with an authentication mechanism:</span></span>

1. <span data-ttu-id="a8711-127">Nel passaggio **configurare la nuova Blazor WebAssembly app** selezionare **autenticazione singola (in-app) nell'elenco a** discesa **autenticazione** .</span><span class="sxs-lookup"><span data-stu-id="a8711-127">On the **Configure your new Blazor WebAssembly App** step, select **Individual Authentication (in-app)** from the **Authentication** drop down.</span></span>

1. <span data-ttu-id="a8711-128">L'app viene creata per usare ASP.NET Core [Identity](xref:security/authentication/identity) e non comporta l'archiviazione degli utenti in un database.</span><span class="sxs-lookup"><span data-stu-id="a8711-128">The app is created to use ASP.NET Core [Identity](xref:security/authentication/identity) and doesn't result in storing users in a database.</span></span> <span data-ttu-id="a8711-129">Le sezioni seguenti di questo articolo forniscono ulteriori dettagli.</span><span class="sxs-lookup"><span data-stu-id="a8711-129">The following sections of this article provide further details.</span></span>

---

## <a name="authentication-package"></a><span data-ttu-id="a8711-130">Pacchetto di autenticazione</span><span class="sxs-lookup"><span data-stu-id="a8711-130">Authentication package</span></span>

<span data-ttu-id="a8711-131">Quando si crea un'app per usare account utente singoli, l'app riceve automaticamente un riferimento al pacchetto per il [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) pacchetto nel file di progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="a8711-131">When an app is created to use Individual User Accounts, the app automatically receives a package reference for the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) package in the app's project file.</span></span> <span data-ttu-id="a8711-132">Il pacchetto fornisce un set di primitive che consentono all'app di autenticare gli utenti e ottenere i token per chiamare le API protette.</span><span class="sxs-lookup"><span data-stu-id="a8711-132">The package provides a set of primitives that help the app authenticate users and obtain tokens to call protected APIs.</span></span>

<span data-ttu-id="a8711-133">Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il pacchetto al file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="a8711-133">If adding authentication to an app, manually add the package to the app's project file:</span></span>

```xml
<PackageReference 
  Include="Microsoft.AspNetCore.Components.WebAssembly.Authentication" 
  Version="{VERSION}" />
```

<span data-ttu-id="a8711-134">Per il segnaposto `{VERSION}` , la versione stabile più recente del pacchetto che corrisponde alla versione del Framework condiviso dell'app è disponibile nella cronologia delle **versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication).</span><span class="sxs-lookup"><span data-stu-id="a8711-134">For the placeholder `{VERSION}`, the latest stable version of the package that matches the app's shared framework version can be found in the package's **Version History** at [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication).</span></span>

## <a name="authentication-service-support"></a><span data-ttu-id="a8711-135">Supporto del servizio di autenticazione</span><span class="sxs-lookup"><span data-stu-id="a8711-135">Authentication service support</span></span>

<span data-ttu-id="a8711-136">Il supporto per l'autenticazione degli utenti viene registrato nel contenitore del servizio con il <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A> metodo di estensione fornito dal [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="a8711-136">Support for authenticating users is registered in the service container with the <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A> extension method provided by the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) package.</span></span> <span data-ttu-id="a8711-137">Questo metodo configura i servizi necessari per l'interazione dell'app con il Identity provider (IP).</span><span class="sxs-lookup"><span data-stu-id="a8711-137">This method sets up the services required for the app to interact with the Identity Provider (IP).</span></span>

<span data-ttu-id="a8711-138">Per una nuova app, fornire i valori per `{AUTHORITY}` i `{CLIENT ID}` segnaposto e nella configurazione seguente.</span><span class="sxs-lookup"><span data-stu-id="a8711-138">For a new app, provide values for the `{AUTHORITY}` and `{CLIENT ID}` placeholders in the following configuration.</span></span> <span data-ttu-id="a8711-139">Fornire altri valori di configurazione necessari per l'uso con l'indirizzo IP dell'app.</span><span class="sxs-lookup"><span data-stu-id="a8711-139">Provide other configuration values that are required for use with the app's IP.</span></span> <span data-ttu-id="a8711-140">L'esempio è per Google, che richiede `PostLogoutRedirectUri` , `RedirectUri` e `ResponseType` .</span><span class="sxs-lookup"><span data-stu-id="a8711-140">The example is for Google, which requires `PostLogoutRedirectUri`, `RedirectUri`, and `ResponseType`.</span></span> <span data-ttu-id="a8711-141">Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il codice e la configurazione seguenti all'app con i valori per i segnaposto e altri valori di configurazione.</span><span class="sxs-lookup"><span data-stu-id="a8711-141">If adding authentication to an app, manually add the following code and configuration to the app with values for the placeholders and other configuration values.</span></span>

<span data-ttu-id="a8711-142">`Program.cs`:</span><span class="sxs-lookup"><span data-stu-id="a8711-142">`Program.cs`:</span></span>

```csharp
builder.Services.AddOidcAuthentication(options =>
{
    builder.Configuration.Bind("Local", options.ProviderOptions);
});
```

<span data-ttu-id="a8711-143">La configurazione viene fornita dal `wwwroot/appsettings.json` file:</span><span class="sxs-lookup"><span data-stu-id="a8711-143">Configuration is supplied by the `wwwroot/appsettings.json` file:</span></span>

```json
{
  "Local": {
    "Authority": "{AUTHORITY}",
    "ClientId": "{CLIENT ID}"
  }
}
```

<span data-ttu-id="a8711-144">Esempio di Google OAuth 2,0 OIDC:</span><span class="sxs-lookup"><span data-stu-id="a8711-144">Google OAuth 2.0 OIDC example:</span></span>

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

<span data-ttu-id="a8711-145">L'URI di reindirizzamento ( `https://localhost:5001/authentication/login-callback` ) viene registrato nella [console di Google Apis](https://console.developers.google.com/apis/dashboard) in **Credentials**  >  **`{NAME}`**  >  **Authorized Redirect URI**, dove `{NAME}` è il nome del client dell'app nell'elenco di app **ID client OAuth 2,0** della console di Google APIs.</span><span class="sxs-lookup"><span data-stu-id="a8711-145">The redirect URI (`https://localhost:5001/authentication/login-callback`) is registered in the [Google APIs console](https://console.developers.google.com/apis/dashboard) in **Credentials** > **`{NAME}`** > **Authorized redirect URIs**, where `{NAME}` is the app's client name in the **OAuth 2.0 Client IDs** app list of the Google APIs console.</span></span>

<span data-ttu-id="a8711-146">Il supporto dell'autenticazione per le app autonome viene offerto tramite OpenID Connect (OIDC).</span><span class="sxs-lookup"><span data-stu-id="a8711-146">Authentication support for standalone apps is offered using OpenID Connect (OIDC).</span></span> <span data-ttu-id="a8711-147">Il <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A> metodo accetta un callback per configurare i parametri necessari per autenticare un'app usando OIDC.</span><span class="sxs-lookup"><span data-stu-id="a8711-147">The <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A> method accepts a callback to configure the parameters required to authenticate an app using OIDC.</span></span> <span data-ttu-id="a8711-148">I valori necessari per la configurazione dell'app possono essere ottenuti dall'IP conforme a OIDC.</span><span class="sxs-lookup"><span data-stu-id="a8711-148">The values required for configuring the app can be obtained from the OIDC-compliant IP.</span></span> <span data-ttu-id="a8711-149">Ottenere i valori quando si registra l'app, che in genere si verifica nel portale online.</span><span class="sxs-lookup"><span data-stu-id="a8711-149">Obtain the values when you register the app, which typically occurs in their online portal.</span></span>

## <a name="access-token-scopes"></a><span data-ttu-id="a8711-150">Ambiti del token di accesso</span><span class="sxs-lookup"><span data-stu-id="a8711-150">Access token scopes</span></span>

<span data-ttu-id="a8711-151">Il Blazor WebAssembly modello configura automaticamente gli ambiti predefiniti per `openid` e `profile` .</span><span class="sxs-lookup"><span data-stu-id="a8711-151">The Blazor WebAssembly template automatically configures default scopes for `openid` and `profile`.</span></span>

<span data-ttu-id="a8711-152">Il Blazor WebAssembly modello non configura automaticamente l'app per richiedere un token di accesso per un'API protetta.</span><span class="sxs-lookup"><span data-stu-id="a8711-152">The Blazor WebAssembly template doesn't automatically configure the app to request an access token for a secure API.</span></span> <span data-ttu-id="a8711-153">Per eseguire il provisioning di un token di accesso come parte del flusso di accesso, aggiungere l'ambito agli ambiti dei token predefiniti di <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.OidcProviderOptions> .</span><span class="sxs-lookup"><span data-stu-id="a8711-153">To provision an access token as part of the sign-in flow, add the scope to the default token scopes of the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.OidcProviderOptions>.</span></span> <span data-ttu-id="a8711-154">Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il codice seguente e configurare l'URI dell'ambito.</span><span class="sxs-lookup"><span data-stu-id="a8711-154">If adding authentication to an app, manually add the following code and configure the scope URI.</span></span>

<span data-ttu-id="a8711-155">`Program.cs`:</span><span class="sxs-lookup"><span data-stu-id="a8711-155">`Program.cs`:</span></span>

```csharp
builder.Services.AddOidcAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultScopes.Add("{SCOPE URI}");
});
```

<span data-ttu-id="a8711-156">Per ulteriori informazioni, vedere le sezioni seguenti dell'articolo *scenari aggiuntivi* :</span><span class="sxs-lookup"><span data-stu-id="a8711-156">For more information, see the following sections of the *Additional scenarios* article:</span></span>

* [<span data-ttu-id="a8711-157">Richiedere token di accesso aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="a8711-157">Request additional access tokens</span></span>](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [<span data-ttu-id="a8711-158">Connetti token alle richieste in uscita</span><span class="sxs-lookup"><span data-stu-id="a8711-158">Attach tokens to outgoing requests</span></span>](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)

## <a name="imports-file"></a><span data-ttu-id="a8711-159">Importa file</span><span class="sxs-lookup"><span data-stu-id="a8711-159">Imports file</span></span>

[!INCLUDE[](~/blazor/includes/security/imports-file-standalone.md)]

## <a name="index-page"></a><span data-ttu-id="a8711-160">Pagina di indice</span><span class="sxs-lookup"><span data-stu-id="a8711-160">Index page</span></span>

[!INCLUDE[](~/blazor/includes/security/index-page-authentication.md)]

## <a name="app-component"></a><span data-ttu-id="a8711-161">Componente app</span><span class="sxs-lookup"><span data-stu-id="a8711-161">App component</span></span>

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

## <a name="redirecttologin-component"></a><span data-ttu-id="a8711-162">Componente RedirectToLogin</span><span class="sxs-lookup"><span data-stu-id="a8711-162">RedirectToLogin component</span></span>

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

## <a name="logindisplay-component"></a><span data-ttu-id="a8711-163">Componente LoginDisplay</span><span class="sxs-lookup"><span data-stu-id="a8711-163">LoginDisplay component</span></span>

<span data-ttu-id="a8711-164">Il `LoginDisplay` componente ( `Shared/LoginDisplay.razor` ) viene sottoposto a rendering nel `MainLayout` componente ( `Shared/MainLayout.razor` ) e gestisce i comportamenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="a8711-164">The `LoginDisplay` component (`Shared/LoginDisplay.razor`) is rendered in the `MainLayout` component (`Shared/MainLayout.razor`) and manages the following behaviors:</span></span>

* <span data-ttu-id="a8711-165">Per utenti autenticati:</span><span class="sxs-lookup"><span data-stu-id="a8711-165">For authenticated users:</span></span>
  * <span data-ttu-id="a8711-166">Visualizza il nome utente corrente.</span><span class="sxs-lookup"><span data-stu-id="a8711-166">Displays the current username.</span></span>
  * <span data-ttu-id="a8711-167">Offre un pulsante per disconnettersi dall'app.</span><span class="sxs-lookup"><span data-stu-id="a8711-167">Offers a button to log out of the app.</span></span>
* <span data-ttu-id="a8711-168">Per gli utenti anonimi, offre la possibilità di eseguire l'accesso.</span><span class="sxs-lookup"><span data-stu-id="a8711-168">For anonymous users, offers the option to log in.</span></span>

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

## <a name="authentication-component"></a><span data-ttu-id="a8711-169">Componente di autenticazione</span><span class="sxs-lookup"><span data-stu-id="a8711-169">Authentication component</span></span>

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a><span data-ttu-id="a8711-170">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a8711-170">Additional resources</span></span>

* <xref:blazor/security/webassembly/additional-scenarios>
* [<span data-ttu-id="a8711-171">Richieste API Web non autenticate o non autorizzate in un'app con un client predefinito sicuro</span><span class="sxs-lookup"><span data-stu-id="a8711-171">Unauthenticated or unauthorized web API requests in an app with a secure default client</span></span>](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <span data-ttu-id="a8711-172"><xref:host-and-deploy/proxy-load-balancer>: Include informazioni aggiuntive su:</span><span class="sxs-lookup"><span data-stu-id="a8711-172"><xref:host-and-deploy/proxy-load-balancer>: Includes guidance on:</span></span>
  * <span data-ttu-id="a8711-173">Uso del middleware intestazioni con inoltro per mantenere le informazioni sullo schema HTTPS tra i server proxy e le reti interne.</span><span class="sxs-lookup"><span data-stu-id="a8711-173">Using Forwarded Headers Middleware to preserve HTTPS scheme information across proxy servers and internal networks.</span></span>
  * <span data-ttu-id="a8711-174">Scenari e casi d'uso aggiuntivi, tra cui la configurazione manuale dello schema, la modifica del percorso della richiesta per il routing delle richieste corretto e l'inoltro dello schema di richiesta per i proxy inversi Linux e non IIS.</span><span class="sxs-lookup"><span data-stu-id="a8711-174">Additional scenarios and use cases, including manual scheme configuration, request path changes for correct request routing, and forwarding the request scheme for Linux and non-IIS reverse proxies.</span></span>
