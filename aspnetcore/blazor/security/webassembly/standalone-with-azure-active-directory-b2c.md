---
title: Proteggere un' Blazor WebAssembly app autonoma ASP.NET Core con Azure Active Directory B2C
author: guardrex
description: Informazioni su come proteggere un' Blazor WebAssembly app autonoma ASP.NET Core con Azure Active Directory B2C.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/27/2020
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
uid: blazor/security/webassembly/standalone-with-azure-active-directory-b2c
ms.openlocfilehash: dd32db8eaac70cfb3dfb3872c43c28aed6a87657
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280895"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-standalone-app-with-azure-active-directory-b2c"></a><span data-ttu-id="21d23-103">Proteggere un' Blazor WebAssembly app autonoma ASP.NET Core con Azure Active Directory B2C</span><span class="sxs-lookup"><span data-stu-id="21d23-103">Secure an ASP.NET Core Blazor WebAssembly standalone app with Azure Active Directory B2C</span></span>

<span data-ttu-id="21d23-104">Questo articolo illustra come creare un' [ Blazor WebAssembly app autonoma](xref:blazor/hosting-models#blazor-webassembly) che usa [Azure Active Directory (AAD) B2C](/azure/active-directory-b2c/overview) per l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="21d23-104">This article covers how to create a [standalone Blazor WebAssembly app](xref:blazor/hosting-models#blazor-webassembly) that uses [Azure Active Directory (AAD) B2C](/azure/active-directory-b2c/overview) for authentication.</span></span>

<span data-ttu-id="21d23-105">Creare un tenant o identificare un tenant B2C esistente per l'app da usare nel portale di Azure seguendo le istruzioni riportate nell'articolo [creare un tenant di AAD B2C (documentazione di Azure)](/azure/active-directory-b2c/tutorial-create-tenant) .</span><span class="sxs-lookup"><span data-stu-id="21d23-105">Create a tenant or identify an existing B2C tenant for the app to use in the Azure portal by following the guidance in the [Create an AAD B2C tenant (Azure documentation)](/azure/active-directory-b2c/tutorial-create-tenant) article.</span></span> <span data-ttu-id="21d23-106">Tornare a questo articolo immediatamente dopo la creazione o l'identificazione di un tenant da usare.</span><span class="sxs-lookup"><span data-stu-id="21d23-106">Return to this article immediately after creating or identifying a tenant to use.</span></span>

<span data-ttu-id="21d23-107">Registrare le seguenti informazioni:</span><span class="sxs-lookup"><span data-stu-id="21d23-107">Record the following information:</span></span>

* <span data-ttu-id="21d23-108">AAD B2C istanza (ad esempio, `https://contoso.b2clogin.com/` che include la barra finale): l'istanza è lo schema e l'host di una registrazione di app di Azure B2C, che è possibile trovare aprendo la finestra **endpoint** dalla pagina **registrazioni app** nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="21d23-108">AAD B2C instance (for example, `https://contoso.b2clogin.com/`, which includes the trailing slash): The instance is the scheme and host of an Azure B2C app registration, which can be found by opening the **Endpoints** window from the **App registrations** page in the Azure portal.</span></span>
* <span data-ttu-id="21d23-109">AAD B2C dominio primario/server di pubblicazione/tenant (ad esempio, `contoso.onmicrosoft.com` ): il dominio è disponibile come **dominio del server di pubblicazione** nel pannello **personalizzazione** del portale di Azure per l'app registrata.</span><span class="sxs-lookup"><span data-stu-id="21d23-109">AAD B2C Primary/Publisher/Tenant domain (for example, `contoso.onmicrosoft.com`): The domain is available as the **Publisher domain** in the **Branding** blade of the Azure portal for the registered app.</span></span>

<span data-ttu-id="21d23-110">Registrare un'app AAD B2C:</span><span class="sxs-lookup"><span data-stu-id="21d23-110">Register an AAD B2C app:</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="21d23-111">In **Azure Active Directory** > **registrazioni app** selezionare **nuova registrazione**.</span><span class="sxs-lookup"><span data-stu-id="21d23-111">In **Azure Active Directory** > **App registrations**, select **New registration**.</span></span>
1. <span data-ttu-id="21d23-112">Specificare un **nome** per l'app, ad esempio **Blazor AAD B2C autonomo**.</span><span class="sxs-lookup"><span data-stu-id="21d23-112">Provide a **Name** for the app (for example, **Blazor Standalone AAD B2C**).</span></span>
1. <span data-ttu-id="21d23-113">Per i **tipi di account supportati**, selezionare l'opzione multi-tenant: **account in qualsiasi directory organizzativa o provider di identità. Per l'autenticazione degli utenti con Azure AD B2C.**</span><span class="sxs-lookup"><span data-stu-id="21d23-113">For **Supported account types**, select the multi-tenant option: **Accounts in any organizational directory or any identity provider. For authenticating users with Azure AD B2C.**</span></span>
1. <span data-ttu-id="21d23-114">Impostare l'elenco a discesa **URI di reindirizzamento** sull' **applicazione a pagina singola (Spa)** e specificare l'URI di reindirizzamento seguente: `https://localhost:{PORT}/authentication/login-callback` .</span><span class="sxs-lookup"><span data-stu-id="21d23-114">Set the **Redirect URI** drop down to **Single-page application (SPA)** and provide the following redirect URI: `https://localhost:{PORT}/authentication/login-callback`.</span></span> <span data-ttu-id="21d23-115">La porta predefinita per un'app in esecuzione in Kestrel è la 5001.</span><span class="sxs-lookup"><span data-stu-id="21d23-115">The default port for an app running on Kestrel is 5001.</span></span> <span data-ttu-id="21d23-116">Se l'app viene eseguita su una porta di Gheppio diversa, usare la porta dell'app.</span><span class="sxs-lookup"><span data-stu-id="21d23-116">If the app is run on a different Kestrel port, use the app's port.</span></span> <span data-ttu-id="21d23-117">Per IIS Express, la porta generata in modo casuale per l'app si trova nelle proprietà dell'app nel pannello **debug** .</span><span class="sxs-lookup"><span data-stu-id="21d23-117">For IIS Express, the randomly generated port for the app can be found in the app's properties in the **Debug** panel.</span></span> <span data-ttu-id="21d23-118">Poiché l'app non esiste in questo momento e la porta IIS Express non è nota, tornare a questo passaggio dopo la creazione dell'app e aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="21d23-118">Since the app doesn't exist at this point and the IIS Express port isn't known, return to this step after the app is created and update the redirect URI.</span></span> <span data-ttu-id="21d23-119">Un commento viene visualizzato più avanti in questo argomento per ricordare IIS Express agli utenti di aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="21d23-119">A remark appears later in this topic to remind IIS Express users to update the redirect URI.</span></span>
1. <span data-ttu-id="21d23-120">Verificare che  > **le autorizzazioni concedano il consenso dell'amministratore per OpenID e offline_access autorizzazioni** siano selezionate.</span><span class="sxs-lookup"><span data-stu-id="21d23-120">Confirm that **Permissions** > **Grant admin consent to openid and offline_access permissions** is selected.</span></span>
1. <span data-ttu-id="21d23-121">Selezionare **Registra**.</span><span class="sxs-lookup"><span data-stu-id="21d23-121">Select **Register**.</span></span>

<span data-ttu-id="21d23-122">Registrare l'ID dell'applicazione (client) (ad esempio, `41451fa7-82d9-4673-8fa5-69eff5a761fd` ).</span><span class="sxs-lookup"><span data-stu-id="21d23-122">Record the Application (client) ID (for example, `41451fa7-82d9-4673-8fa5-69eff5a761fd`).</span></span>

<span data-ttu-id="21d23-123">Nelle configurazioni della piattaforma di **autenticazione** >  > **applicazione a pagina singola (Spa)**:</span><span class="sxs-lookup"><span data-stu-id="21d23-123">In **Authentication** > **Platform configurations** > **Single-page application (SPA)**:</span></span>

1. <span data-ttu-id="21d23-124">Verificare che l' **URI di reindirizzamento** di `https://localhost:{PORT}/authentication/login-callback` sia presente.</span><span class="sxs-lookup"><span data-stu-id="21d23-124">Confirm the **Redirect URI** of `https://localhost:{PORT}/authentication/login-callback` is present.</span></span>
1. <span data-ttu-id="21d23-125">Per **concessione implicita**, assicurarsi che le caselle di controllo per i **token di accesso** e i **token ID** **non** siano selezionate.</span><span class="sxs-lookup"><span data-stu-id="21d23-125">For **Implicit grant**, ensure that the check boxes for **Access tokens** and **ID tokens** are **not** selected.</span></span>
1. <span data-ttu-id="21d23-126">Per questa esperienza sono accettabili le impostazioni predefinite rimanenti per l'app.</span><span class="sxs-lookup"><span data-stu-id="21d23-126">The remaining defaults for the app are acceptable for this experience.</span></span>
1. <span data-ttu-id="21d23-127">Fare clic sul pulsante **Salva**.</span><span class="sxs-lookup"><span data-stu-id="21d23-127">Select the **Save** button.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="21d23-128">In **Azure Active Directory** > **registrazioni app** selezionare **nuova registrazione**.</span><span class="sxs-lookup"><span data-stu-id="21d23-128">In **Azure Active Directory** > **App registrations**, select **New registration**.</span></span>
1. <span data-ttu-id="21d23-129">Specificare un **nome** per l'app, ad esempio **Blazor AAD B2C autonomo**.</span><span class="sxs-lookup"><span data-stu-id="21d23-129">Provide a **Name** for the app (for example, **Blazor Standalone AAD B2C**).</span></span>
1. <span data-ttu-id="21d23-130">Per i **tipi di account supportati**, selezionare l'opzione multi-tenant: **account in qualsiasi directory organizzativa o provider di identità. Per l'autenticazione degli utenti con Azure AD B2C.**</span><span class="sxs-lookup"><span data-stu-id="21d23-130">For **Supported account types**, select the multi-tenant option: **Accounts in any organizational directory or any identity provider. For authenticating users with Azure AD B2C.**</span></span>
1. <span data-ttu-id="21d23-131">Lasciare l'elenco a discesa **URI di reindirizzamento** impostato su **Web** e specificare l'URI di reindirizzamento seguente: `https://localhost:{PORT}/authentication/login-callback` .</span><span class="sxs-lookup"><span data-stu-id="21d23-131">Leave the **Redirect URI** drop down set to **Web** and provide the following redirect URI: `https://localhost:{PORT}/authentication/login-callback`.</span></span> <span data-ttu-id="21d23-132">La porta predefinita per un'app in esecuzione in Kestrel è la 5001.</span><span class="sxs-lookup"><span data-stu-id="21d23-132">The default port for an app running on Kestrel is 5001.</span></span> <span data-ttu-id="21d23-133">Se l'app viene eseguita su una porta di Gheppio diversa, usare la porta dell'app.</span><span class="sxs-lookup"><span data-stu-id="21d23-133">If the app is run on a different Kestrel port, use the app's port.</span></span> <span data-ttu-id="21d23-134">Per IIS Express, la porta generata in modo casuale per l'app si trova nelle proprietà dell'app nel pannello **debug** .</span><span class="sxs-lookup"><span data-stu-id="21d23-134">For IIS Express, the randomly generated port for the app can be found in the app's properties in the **Debug** panel.</span></span> <span data-ttu-id="21d23-135">Poiché l'app non esiste in questo momento e la porta IIS Express non è nota, tornare a questo passaggio dopo la creazione dell'app e aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="21d23-135">Since the app doesn't exist at this point and the IIS Express port isn't known, return to this step after the app is created and update the redirect URI.</span></span> <span data-ttu-id="21d23-136">Un commento viene visualizzato più avanti in questo argomento per ricordare IIS Express agli utenti di aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="21d23-136">A remark appears later in this topic to remind IIS Express users to update the redirect URI.</span></span>
1. <span data-ttu-id="21d23-137">Verificare che  > **le autorizzazioni concedano il consenso dell'amministratore per OpenID e offline_access autorizzazioni** siano selezionate.</span><span class="sxs-lookup"><span data-stu-id="21d23-137">Confirm that **Permissions** > **Grant admin consent to openid and offline_access permissions** is selected.</span></span>
1. <span data-ttu-id="21d23-138">Selezionare **Registra**.</span><span class="sxs-lookup"><span data-stu-id="21d23-138">Select **Register**.</span></span>

<span data-ttu-id="21d23-139">Registrare l'ID dell'applicazione (client) (ad esempio, `41451fa7-82d9-4673-8fa5-69eff5a761fd` ).</span><span class="sxs-lookup"><span data-stu-id="21d23-139">Record the Application (client) ID (for example, `41451fa7-82d9-4673-8fa5-69eff5a761fd`).</span></span>

<span data-ttu-id="21d23-140">In  > **configurazioni piattaforma** di autenticazione > **Web**:</span><span class="sxs-lookup"><span data-stu-id="21d23-140">In **Authentication** > **Platform configurations** > **Web**:</span></span>

1. <span data-ttu-id="21d23-141">Verificare che l' **URI di reindirizzamento** di `https://localhost:{PORT}/authentication/login-callback` sia presente.</span><span class="sxs-lookup"><span data-stu-id="21d23-141">Confirm the **Redirect URI** of `https://localhost:{PORT}/authentication/login-callback` is present.</span></span>
1. <span data-ttu-id="21d23-142">Per **concessione implicita**, selezionare le caselle di controllo per i token di **accesso** e i **token ID**.</span><span class="sxs-lookup"><span data-stu-id="21d23-142">For **Implicit grant**, select the check boxes for **Access tokens** and **ID tokens**.</span></span>
1. <span data-ttu-id="21d23-143">Per questa esperienza sono accettabili le impostazioni predefinite rimanenti per l'app.</span><span class="sxs-lookup"><span data-stu-id="21d23-143">The remaining defaults for the app are acceptable for this experience.</span></span>
1. <span data-ttu-id="21d23-144">Fare clic sul pulsante **Salva**.</span><span class="sxs-lookup"><span data-stu-id="21d23-144">Select the **Save** button.</span></span>

::: moniker-end

<span data-ttu-id="21d23-145">In **Home**  >  **Azure ad B2C**  >  **flussi utente**:</span><span class="sxs-lookup"><span data-stu-id="21d23-145">In **Home** > **Azure AD B2C** > **User flows**:</span></span>

[<span data-ttu-id="21d23-146">Creare un flusso utente di iscrizione e accesso</span><span class="sxs-lookup"><span data-stu-id="21d23-146">Create a sign-up and sign-in user flow</span></span>](/azure/active-directory-b2c/tutorial-create-user-flows)

<span data-ttu-id="21d23-147">Selezionare almeno l'attributo utente nome visualizzato dell' **applicazione Claims**  >   per popolare `context.User.Identity.Name` nel `LoginDisplay` componente ( `Shared/LoginDisplay.razor` ).</span><span class="sxs-lookup"><span data-stu-id="21d23-147">At a minimum, select the **Application claims** > **Display Name** user attribute to populate the `context.User.Identity.Name` in the `LoginDisplay` component (`Shared/LoginDisplay.razor`).</span></span>

<span data-ttu-id="21d23-148">Registrare il nome del flusso utente di iscrizione e accesso creato per l'app (ad esempio, `B2C_1_signupsignin` ).</span><span class="sxs-lookup"><span data-stu-id="21d23-148">Record the sign-up and sign-in user flow name created for the app (for example, `B2C_1_signupsignin`).</span></span>

<span data-ttu-id="21d23-149">In una cartella vuota, sostituire i segnaposto nel comando seguente con le informazioni registrate in precedenza ed eseguire il comando in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="21d23-149">In an empty folder, replace the placeholders in the following command with the information recorded earlier and execute the command in a command shell:</span></span>

```dotnetcli
dotnet new blazorwasm -au IndividualB2C --aad-b2c-instance "{AAD B2C INSTANCE}" --client-id "{CLIENT ID}" --domain "{TENANT DOMAIN}" -o {APP NAME} -ssp "{SIGN UP OR SIGN IN POLICY}"
```

| <span data-ttu-id="21d23-150">Segnaposto</span><span class="sxs-lookup"><span data-stu-id="21d23-150">Placeholder</span></span>                   | <span data-ttu-id="21d23-151">Nome portale di Azure</span><span class="sxs-lookup"><span data-stu-id="21d23-151">Azure portal name</span></span>               | <span data-ttu-id="21d23-152">Esempio</span><span class="sxs-lookup"><span data-stu-id="21d23-152">Example</span></span>                                |
| ----------------------------- | ------------------------------- | -------------------------------------- |
| `{AAD B2C INSTANCE}`          | <span data-ttu-id="21d23-153">Istanza</span><span class="sxs-lookup"><span data-stu-id="21d23-153">Instance</span></span>                        | `https://contoso.b2clogin.com/`        |
| `{APP NAME}`                  | &mdash;                         | `BlazorSample`                         |
| `{CLIENT ID}`                 | <span data-ttu-id="21d23-154">ID applicazione (client)</span><span class="sxs-lookup"><span data-stu-id="21d23-154">Application (client) ID</span></span>         | `41451fa7-82d9-4673-8fa5-69eff5a761fd` |
| `{SIGN UP OR SIGN IN POLICY}` | <span data-ttu-id="21d23-155">Flusso utente di iscrizione/accesso</span><span class="sxs-lookup"><span data-stu-id="21d23-155">Sign-up/sign-in user flow</span></span>       | `B2C_1_signupsignin1`                  |
| `{TENANT DOMAIN}`             | <span data-ttu-id="21d23-156">Dominio primario/server di pubblicazione/tenant</span><span class="sxs-lookup"><span data-stu-id="21d23-156">Primary/Publisher/Tenant domain</span></span> | `contoso.onmicrosoft.com`              |

<span data-ttu-id="21d23-157">Il percorso di output specificato con l'opzione `-o|--output` consente di creare una cartella di progetto se non esiste e viene incluso nel nome dell'app.</span><span class="sxs-lookup"><span data-stu-id="21d23-157">The output location specified with the `-o|--output` option creates a project folder if it doesn't exist and becomes part of the app's name.</span></span>

> [!NOTE]
> <span data-ttu-id="21d23-158">Nell'portale di Azure l' **URI di reindirizzamento** della configurazione della piattaforma dell'app è configurato per la porta 5001 per le app eseguite nel server gheppio con le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="21d23-158">In the Azure portal, the app's platform configuration **Redirect URI** is configured for port 5001 for apps that run on the Kestrel server with default settings.</span></span>
>
> <span data-ttu-id="21d23-159">Se l'app viene eseguita su una porta IIS Express casuale, la porta per l'app si trova nelle proprietà dell'app nel pannello **debug** .</span><span class="sxs-lookup"><span data-stu-id="21d23-159">If the app is run on a random IIS Express port, the port for the app can be found in the app's properties in the **Debug** panel.</span></span>
>
> <span data-ttu-id="21d23-160">Se la porta non è stata configurata in precedenza con la porta nota dell'app, tornare alla registrazione dell'app nel portale di Azure e aggiornare l'URI di reindirizzamento con la porta corretta.</span><span class="sxs-lookup"><span data-stu-id="21d23-160">If the port wasn't configured earlier with the app's known port, return to the app's registration in the Azure portal and update the redirect URI with the correct port.</span></span>

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE[](~/blazor/includes/security/additional-scopes-standalone-nonAAD.md)]

::: moniker-end

<span data-ttu-id="21d23-161">Dopo aver creato l'app, dovrebbe essere possibile:</span><span class="sxs-lookup"><span data-stu-id="21d23-161">After creating the app, you should be able to:</span></span>

* <span data-ttu-id="21d23-162">Accedere all'app usando un account utente di AAD.</span><span class="sxs-lookup"><span data-stu-id="21d23-162">Log into the app using an AAD user account.</span></span>
* <span data-ttu-id="21d23-163">Richiedere i token di accesso per le API Microsoft.</span><span class="sxs-lookup"><span data-stu-id="21d23-163">Request access tokens for Microsoft APIs.</span></span> <span data-ttu-id="21d23-164">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="21d23-164">For more information, see:</span></span>
  * [<span data-ttu-id="21d23-165">Ambiti del token di accesso</span><span class="sxs-lookup"><span data-stu-id="21d23-165">Access token scopes</span></span>](#access-token-scopes)
  * <span data-ttu-id="21d23-166">[Guida introduttiva: configurare un'applicazione per esporre le API Web](/azure/active-directory/develop/quickstart-configure-app-expose-web-apis).</span><span class="sxs-lookup"><span data-stu-id="21d23-166">[Quickstart: Configure an application to expose web APIs](/azure/active-directory/develop/quickstart-configure-app-expose-web-apis).</span></span>

## <a name="authentication-package"></a><span data-ttu-id="21d23-167">Pacchetto di autenticazione</span><span class="sxs-lookup"><span data-stu-id="21d23-167">Authentication package</span></span>

<span data-ttu-id="21d23-168">Quando viene creata un'app per usare un singolo account B2C ( `IndividualB2C` ), l'app riceve automaticamente un riferimento al pacchetto per [Microsoft Authentication Library](/azure/active-directory/develop/msal-overview) ( [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) ).</span><span class="sxs-lookup"><span data-stu-id="21d23-168">When an app is created to use an Individual B2C Account (`IndividualB2C`), the app automatically receives a package reference for the [Microsoft Authentication Library](/azure/active-directory/develop/msal-overview) ([`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal)).</span></span> <span data-ttu-id="21d23-169">Il pacchetto fornisce un set di primitive che consentono all'app di autenticare gli utenti e ottenere i token per chiamare le API protette.</span><span class="sxs-lookup"><span data-stu-id="21d23-169">The package provides a set of primitives that help the app authenticate users and obtain tokens to call protected APIs.</span></span>

<span data-ttu-id="21d23-170">Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il pacchetto al file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="21d23-170">If adding authentication to an app, manually add the package to the app's project file:</span></span>

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
  Version="{VERSION}" />
```

<span data-ttu-id="21d23-171">Per il segnaposto `{VERSION}` , la versione stabile più recente del pacchetto che corrisponde alla versione del Framework condiviso dell'app è disponibile nella cronologia delle **versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal).</span><span class="sxs-lookup"><span data-stu-id="21d23-171">For the placeholder `{VERSION}`, the latest stable version of the package that matches the app's shared framework version can be found in the package's **Version History** at [NuGet.org](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal).</span></span>

<span data-ttu-id="21d23-172">Il [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) pacchetto aggiunge il [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) pacchetto all'app in modo transitivo.</span><span class="sxs-lookup"><span data-stu-id="21d23-172">The [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) package transitively adds the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) package to the app.</span></span>

## <a name="authentication-service-support"></a><span data-ttu-id="21d23-173">Supporto del servizio di autenticazione</span><span class="sxs-lookup"><span data-stu-id="21d23-173">Authentication service support</span></span>

<span data-ttu-id="21d23-174">Il supporto per l'autenticazione degli utenti viene registrato nel contenitore del servizio con il <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> metodo di estensione fornito dal [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="21d23-174">Support for authenticating users is registered in the service container with the <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> extension method provided by the [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) package.</span></span> <span data-ttu-id="21d23-175">Questo metodo configura tutti i servizi necessari per l'interazione dell'app con il Identity provider (IP).</span><span class="sxs-lookup"><span data-stu-id="21d23-175">This method sets up all of the services required for the app to interact with the Identity Provider (IP).</span></span>

<span data-ttu-id="21d23-176">`Program.cs`:</span><span class="sxs-lookup"><span data-stu-id="21d23-176">`Program.cs`:</span></span>

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAdB2C", options.ProviderOptions.Authentication);
});
```

<span data-ttu-id="21d23-177">Il <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> metodo accetta un callback per configurare i parametri necessari per autenticare un'app.</span><span class="sxs-lookup"><span data-stu-id="21d23-177">The <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> method accepts a callback to configure the parameters required to authenticate an app.</span></span> <span data-ttu-id="21d23-178">Quando si registra l'app, è possibile ottenere i valori necessari per la configurazione dell'app dalla configurazione di AAD.</span><span class="sxs-lookup"><span data-stu-id="21d23-178">The values required for configuring the app can be obtained from the AAD configuration when you register the app.</span></span>

<span data-ttu-id="21d23-179">La configurazione viene fornita dal `wwwroot/appsettings.json` file:</span><span class="sxs-lookup"><span data-stu-id="21d23-179">Configuration is supplied by the `wwwroot/appsettings.json` file:</span></span>

```json
{
  "AzureAdB2C": {
    "Authority": "{AAD B2C INSTANCE}{DOMAIN}/{SIGN UP OR SIGN IN POLICY}",
    "ClientId": "{CLIENT ID}",
    "ValidateAuthority": false
  }
}
```

<span data-ttu-id="21d23-180">Esempio:</span><span class="sxs-lookup"><span data-stu-id="21d23-180">Example:</span></span>

```json
{
  "AzureAdB2C": {
    "Authority": "https://contoso.b2clogin.com/contoso.onmicrosoft.com/B2C_1_signupsignin1",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
    "ValidateAuthority": false
  }
}
```

## <a name="access-token-scopes"></a><span data-ttu-id="21d23-181">Ambiti del token di accesso</span><span class="sxs-lookup"><span data-stu-id="21d23-181">Access token scopes</span></span>

<span data-ttu-id="21d23-182">Il Blazor WebAssembly modello non configura automaticamente l'app per richiedere un token di accesso per un'API protetta.</span><span class="sxs-lookup"><span data-stu-id="21d23-182">The Blazor WebAssembly template doesn't automatically configure the app to request an access token for a secure API.</span></span> <span data-ttu-id="21d23-183">Per eseguire il provisioning di un token di accesso come parte del flusso di accesso, aggiungere l'ambito agli ambiti dei token di accesso predefiniti di <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> :</span><span class="sxs-lookup"><span data-stu-id="21d23-183">To provision an access token as part of the sign-in flow, add the scope to the default access token scopes of the <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions>:</span></span>

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

<span data-ttu-id="21d23-184">Specificare gli ambiti aggiuntivi con `AdditionalScopesToConsent` :</span><span class="sxs-lookup"><span data-stu-id="21d23-184">Specify additional scopes with `AdditionalScopesToConsent`:</span></span>

```csharp
options.ProviderOptions.AdditionalScopesToConsent.Add("{ADDITIONAL SCOPE URI}");
```

<span data-ttu-id="21d23-185">Per ulteriori informazioni, vedere le sezioni seguenti dell'articolo *scenari aggiuntivi* :</span><span class="sxs-lookup"><span data-stu-id="21d23-185">For more information, see the following sections of the *Additional scenarios* article:</span></span>

* [<span data-ttu-id="21d23-186">Richiedere token di accesso aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="21d23-186">Request additional access tokens</span></span>](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [<span data-ttu-id="21d23-187">Connetti token alle richieste in uscita</span><span class="sxs-lookup"><span data-stu-id="21d23-187">Attach tokens to outgoing requests</span></span>](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)

::: moniker range=">= aspnetcore-5.0"

## <a name="login-mode"></a><span data-ttu-id="21d23-188">Modalità di accesso</span><span class="sxs-lookup"><span data-stu-id="21d23-188">Login mode</span></span>

[!INCLUDE[](~/blazor/includes/security/msal-login-mode.md)]

::: moniker-end

## <a name="imports-file"></a><span data-ttu-id="21d23-189">Importa file</span><span class="sxs-lookup"><span data-stu-id="21d23-189">Imports file</span></span>

[!INCLUDE[](~/blazor/includes/security/imports-file-standalone.md)]

## <a name="index-page"></a><span data-ttu-id="21d23-190">Pagina di indice</span><span class="sxs-lookup"><span data-stu-id="21d23-190">Index page</span></span>

[!INCLUDE[](~/blazor/includes/security/index-page-msal.md)]

## <a name="app-component"></a><span data-ttu-id="21d23-191">Componente app</span><span class="sxs-lookup"><span data-stu-id="21d23-191">App component</span></span>

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

## <a name="redirecttologin-component"></a><span data-ttu-id="21d23-192">Componente RedirectToLogin</span><span class="sxs-lookup"><span data-stu-id="21d23-192">RedirectToLogin component</span></span>

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

## <a name="logindisplay-component"></a><span data-ttu-id="21d23-193">Componente LoginDisplay</span><span class="sxs-lookup"><span data-stu-id="21d23-193">LoginDisplay component</span></span>

[!INCLUDE[](~/blazor/includes/security/logindisplay-component.md)]

## <a name="authentication-component"></a><span data-ttu-id="21d23-194">Componente di autenticazione</span><span class="sxs-lookup"><span data-stu-id="21d23-194">Authentication component</span></span>

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

[!INCLUDE[](~/blazor/includes/security/wasm-aad-b2c-userflows.md)]

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a><span data-ttu-id="21d23-195">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="21d23-195">Additional resources</span></span>

* <xref:blazor/security/webassembly/additional-scenarios>
* [<span data-ttu-id="21d23-196">Compilare una versione personalizzata della libreria JavaScript Authentication. MSAL</span><span class="sxs-lookup"><span data-stu-id="21d23-196">Build a custom version of the Authentication.MSAL JavaScript library</span></span>](xref:blazor/security/webassembly/additional-scenarios#build-a-custom-version-of-the-authenticationmsal-javascript-library)
* [<span data-ttu-id="21d23-197">Richieste API Web non autenticate o non autorizzate in un'app con un client predefinito sicuro</span><span class="sxs-lookup"><span data-stu-id="21d23-197">Unauthenticated or unauthorized web API requests in an app with a secure default client</span></span>](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:security/authentication/azure-ad-b2c>
* [<span data-ttu-id="21d23-198">Esercitazione: Creare un tenant di Azure Active Directory B2C</span><span class="sxs-lookup"><span data-stu-id="21d23-198">Tutorial: Create an Azure Active Directory B2C tenant</span></span>](/azure/active-directory-b2c/tutorial-create-tenant)
* [<span data-ttu-id="21d23-199">Esercitazione: Registrare un'applicazione in Azure Active Directory B2C</span><span class="sxs-lookup"><span data-stu-id="21d23-199">Tutorial: Register an application in Azure Active Directory B2C</span></span>](/azure/active-directory-b2c/tutorial-register-applications)
* [<span data-ttu-id="21d23-200">Documentazione di Microsoft Identity Platform</span><span class="sxs-lookup"><span data-stu-id="21d23-200">Microsoft identity platform documentation</span></span>](/azure/active-directory/develop/)
