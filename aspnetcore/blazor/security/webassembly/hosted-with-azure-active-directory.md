---
title: Proteggere un' Blazor WebAssembly app ospitata ASP.NET Core con Azure Active Directory
author: guardrex
description: Informazioni su come proteggere un' Blazor WebAssembly app ospitata ASP.NET Core con Azure Active Directory.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 11/02/2020
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
uid: blazor/security/webassembly/hosted-with-azure-active-directory
ms.openlocfilehash: e38838930dba70abfcfe4db9c204132e67866041
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280945"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-azure-active-directory"></a><span data-ttu-id="d6f51-103">Proteggere un' Blazor WebAssembly app ospitata ASP.NET Core con Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="d6f51-103">Secure an ASP.NET Core Blazor WebAssembly hosted app with Azure Active Directory</span></span>

<span data-ttu-id="d6f51-104">Questo articolo descrive come creare un' [ Blazor WebAssembly app ospitata](xref:blazor/hosting-models#blazor-webassembly) che usa [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) per l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d6f51-104">This article describes how to create a [hosted Blazor WebAssembly app](xref:blazor/hosting-models#blazor-webassembly) that uses [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) for authentication.</span></span>

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="d6f51-105">Per Blazor WebAssembly le app create in Visual Studio configurate per il supporto di account in una directory organizzativa di AAD, Visual Studio non configura attualmente i progetti della soluzione o le registrazioni dell'app portale di Azure correttamente durante la generazione del progetto.</span><span class="sxs-lookup"><span data-stu-id="d6f51-105">For Blazor WebAssembly apps created in Visual Studio that are configured to support accounts in an AAD organizational directory, Visual Studio doesn't currently configure the solution's projects or the Azure portal app registrations correctly on project generation.</span></span> <span data-ttu-id="d6f51-106">Questa operazione verrà risolta in una versione futura di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="d6f51-106">This will be addressed in a future release of Visual Studio.</span></span>
>
> <span data-ttu-id="d6f51-107">Questo articolo illustra come creare la soluzione e le registrazioni del portale app di Azure con il comando dell'interfaccia della `dotnet new` riga di comando di .NET e creare manualmente le registrazioni dell'app nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="d6f51-107">This article shows how to create the solution and Azure app portal registrations with the .NET CLI `dotnet new` command and by manually creating the app registrations in the Azure portal.</span></span>
>
> <span data-ttu-id="d6f51-108">Se si preferisce creare la soluzione e le registrazioni di app di Azure con Visual Studio prima che l'IDE venga aggiornato, fare riferimento a **_ogni sezione di questo articolo_** e confermare o aggiornare le configurazioni delle app e le registrazioni delle app dopo la creazione della soluzione da parte di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="d6f51-108">If you prefer to create the solution and Azure app registrations with Visual Studio before the IDE is updated, refer to **_each section of this article_** and confirm or update the apps' configurations and the apps' registrations after Visual Studio creates the solution.</span></span>

::: moniker-end

## <a name="register-apps-in-aad-and-create-solution"></a><span data-ttu-id="d6f51-109">Registrare le app in AAD e creare la soluzione</span><span class="sxs-lookup"><span data-stu-id="d6f51-109">Register apps in AAD and create solution</span></span>

### <a name="create-a-tenant"></a><span data-ttu-id="d6f51-110">Creare un tenant</span><span class="sxs-lookup"><span data-stu-id="d6f51-110">Create a tenant</span></span>

<span data-ttu-id="d6f51-111">Seguire le istruzioni riportate nella [Guida introduttiva: configurare un tenant](/azure/active-directory/develop/quickstart-create-new-tenant) per creare un tenant in AAD.</span><span class="sxs-lookup"><span data-stu-id="d6f51-111">Follow the guidance in [Quickstart: Set up a tenant](/azure/active-directory/develop/quickstart-create-new-tenant) to create a tenant in AAD.</span></span>

### <a name="register-a-server-api-app"></a><span data-ttu-id="d6f51-112">Registrare un'app per le API server</span><span class="sxs-lookup"><span data-stu-id="d6f51-112">Register a server API app</span></span>

<span data-ttu-id="d6f51-113">Registrare un'app AAD per l' *app* per le API del server:</span><span class="sxs-lookup"><span data-stu-id="d6f51-113">Register an AAD app for the *Server API app*:</span></span>

1. <span data-ttu-id="d6f51-114">In **Azure Active Directory**  >  **registrazioni app** selezionare **nuova registrazione**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-114">In **Azure Active Directory** > **App registrations**, select **New registration**.</span></span>
1. <span data-ttu-id="d6f51-115">Specificare un **nome** per l'app (ad esempio, **Blazor Server AAD**).</span><span class="sxs-lookup"><span data-stu-id="d6f51-115">Provide a **Name** for the app (for example, **Blazor Server AAD**).</span></span>
1. <span data-ttu-id="d6f51-116">Scegliere i **tipi di conto supportati**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-116">Choose a **Supported account types**.</span></span> <span data-ttu-id="d6f51-117">Per questa esperienza è possibile selezionare **solo gli account in questa directory aziendale** (tenant singolo).</span><span class="sxs-lookup"><span data-stu-id="d6f51-117">You may select **Accounts in this organizational directory only** (single tenant) for this experience.</span></span>
1. <span data-ttu-id="d6f51-118">L' *app* per le API del server non richiede un **URI di reindirizzamento** in questo scenario, quindi lasciare l'elenco a discesa impostato su **Web** e non immettere un URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="d6f51-118">The *Server API app* doesn't require a **Redirect URI** in this scenario, so leave the drop down set to **Web** and don't enter a redirect URI.</span></span>
1. <span data-ttu-id="d6f51-119">Deselezionare la   >  casella di controllo autorizzazioni **Concedi amministratore per OpenID e autorizzazioni offline_access** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-119">Clear the **Permissions** > **Grant admin consent to openid and offline_access permissions** check box.</span></span>
1. <span data-ttu-id="d6f51-120">Selezionare **Registra**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-120">Select **Register**.</span></span>

<span data-ttu-id="d6f51-121">Registrare le seguenti informazioni:</span><span class="sxs-lookup"><span data-stu-id="d6f51-121">Record the following information:</span></span>

* <span data-ttu-id="d6f51-122">*App per le API server* ID applicazione (client) (ad esempio, `41451fa7-82d9-4673-8fa5-69eff5a761fd` )</span><span class="sxs-lookup"><span data-stu-id="d6f51-122">*Server API app* Application (client) ID (for example, `41451fa7-82d9-4673-8fa5-69eff5a761fd`)</span></span>
* <span data-ttu-id="d6f51-123">ID directory (tenant) (ad esempio, `e86c78e2-8bb4-4c41-aefd-918e0565a45e` )</span><span class="sxs-lookup"><span data-stu-id="d6f51-123">Directory (tenant) ID (for example, `e86c78e2-8bb4-4c41-aefd-918e0565a45e`)</span></span>
* <span data-ttu-id="d6f51-124">Dominio primario/editore/tenant di AAD (ad esempio, `contoso.onmicrosoft.com` ): il dominio è disponibile come **dominio del server di pubblicazione** nel pannello **personalizzazione** del portale di Azure per l'app registrata.</span><span class="sxs-lookup"><span data-stu-id="d6f51-124">AAD Primary/Publisher/Tenant domain (for example, `contoso.onmicrosoft.com`): The domain is available as the **Publisher domain** in the **Branding** blade of the Azure portal for the registered app.</span></span>

<span data-ttu-id="d6f51-125">In **autorizzazioni API** rimuovere l'autorizzazione **Microsoft Graph**  >  **utente. Read** perché l'app non richiede l'accesso o il profilo utente.</span><span class="sxs-lookup"><span data-stu-id="d6f51-125">In **API permissions**, remove the **Microsoft Graph** > **User.Read** permission, as the app doesn't require sign in or user profile access.</span></span>

<span data-ttu-id="d6f51-126">In **esporre un'API**:</span><span class="sxs-lookup"><span data-stu-id="d6f51-126">In **Expose an API**:</span></span>

1. <span data-ttu-id="d6f51-127">Selezionare **Aggiungi un ambito**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-127">Select **Add a scope**.</span></span>
1. <span data-ttu-id="d6f51-128">Fai clic su **Salva e continua**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-128">Select **Save and continue**.</span></span>
1. <span data-ttu-id="d6f51-129">Specificare un **nome** per l'ambito (ad esempio, `API.Access` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-129">Provide a **Scope name** (for example, `API.Access`).</span></span>
1. <span data-ttu-id="d6f51-130">Fornire un **nome visualizzato** per il consenso dell'amministratore, ad esempio `Access API` .</span><span class="sxs-lookup"><span data-stu-id="d6f51-130">Provide an **Admin consent display name** (for example, `Access API`).</span></span>
1. <span data-ttu-id="d6f51-131">Fornire una **Descrizione del consenso dell'amministratore** , ad esempio `Allows the app to access server app API endpoints.` .</span><span class="sxs-lookup"><span data-stu-id="d6f51-131">Provide an **Admin consent description** (for example, `Allows the app to access server app API endpoints.`).</span></span>
1. <span data-ttu-id="d6f51-132">Verificare che lo **stato** sia impostato su **abilitato**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-132">Confirm that the **State** is set to **Enabled**.</span></span>
1. <span data-ttu-id="d6f51-133">Selezionare **Aggiungi ambito**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-133">Select **Add scope**.</span></span>

<span data-ttu-id="d6f51-134">Registrare le seguenti informazioni:</span><span class="sxs-lookup"><span data-stu-id="d6f51-134">Record the following information:</span></span>

* <span data-ttu-id="d6f51-135">URI ID app (ad esempio, `api://41451fa7-82d9-4673-8fa5-69eff5a761fd` , `https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd` o il valore personalizzato fornito)</span><span class="sxs-lookup"><span data-stu-id="d6f51-135">App ID URI (for example, `api://41451fa7-82d9-4673-8fa5-69eff5a761fd`, `https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd`, or the custom value that you provide)</span></span>
* <span data-ttu-id="d6f51-136">Nome ambito (ad esempio, `API.Access` )</span><span class="sxs-lookup"><span data-stu-id="d6f51-136">Scope name (for example, `API.Access`)</span></span>

### <a name="register-a-client-app"></a><span data-ttu-id="d6f51-137">Registrare un'app client</span><span class="sxs-lookup"><span data-stu-id="d6f51-137">Register a client app</span></span>

<span data-ttu-id="d6f51-138">Registrare un'app AAD per l' *app client*:</span><span class="sxs-lookup"><span data-stu-id="d6f51-138">Register an AAD app for the *Client app*:</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="d6f51-139">In **Azure Active Directory** > **registrazioni app** selezionare **nuova registrazione**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-139">In **Azure Active Directory** > **App registrations**, select **New registration**.</span></span>
1. <span data-ttu-id="d6f51-140">Specificare un **nome** per l'app (ad esempio, **Blazor client AAD**).</span><span class="sxs-lookup"><span data-stu-id="d6f51-140">Provide a **Name** for the app (for example, **Blazor Client AAD**).</span></span>
1. <span data-ttu-id="d6f51-141">Scegliere i **tipi di conto supportati**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-141">Choose a **Supported account types**.</span></span> <span data-ttu-id="d6f51-142">Per questa esperienza è possibile selezionare **solo gli account in questa directory aziendale** (tenant singolo).</span><span class="sxs-lookup"><span data-stu-id="d6f51-142">You may select **Accounts in this organizational directory only** (single tenant) for this experience.</span></span>
1. <span data-ttu-id="d6f51-143">Impostare l'elenco a discesa **URI di reindirizzamento** sull' **applicazione a pagina singola (Spa)** e specificare l'URI di reindirizzamento seguente: `https://localhost:{PORT}/authentication/login-callback` .</span><span class="sxs-lookup"><span data-stu-id="d6f51-143">Set the **Redirect URI** drop down to **Single-page application (SPA)** and provide the following redirect URI: `https://localhost:{PORT}/authentication/login-callback`.</span></span> <span data-ttu-id="d6f51-144">La porta predefinita per un'app in esecuzione in Kestrel è la 5001.</span><span class="sxs-lookup"><span data-stu-id="d6f51-144">The default port for an app running on Kestrel is 5001.</span></span> <span data-ttu-id="d6f51-145">Se l'app viene eseguita su una porta di Gheppio diversa, usare la porta dell'app.</span><span class="sxs-lookup"><span data-stu-id="d6f51-145">If the app is run on a different Kestrel port, use the app's port.</span></span> <span data-ttu-id="d6f51-146">Per IIS Express, la porta generata in modo casuale per l'app si trova nelle *`Server`* proprietà dell'app nel pannello **debug** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-146">For IIS Express, the randomly generated port for the app can be found in the *`Server`* app's properties in the **Debug** panel.</span></span> <span data-ttu-id="d6f51-147">Poiché l'app non esiste in questo momento e la porta IIS Express non è nota, tornare a questo passaggio dopo la creazione dell'app e aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="d6f51-147">Since the app doesn't exist at this point and the IIS Express port isn't known, return to this step after the app is created and update the redirect URI.</span></span> <span data-ttu-id="d6f51-148">Un contrassegno viene visualizzato nella sezione [creare l'app](#create-the-app) per ricordare IIS Express agli utenti di aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="d6f51-148">A remark appears in the [Create the app](#create-the-app) section to remind IIS Express users to update the redirect URI.</span></span>
1. <span data-ttu-id="d6f51-149">Deselezionare la  > casella di controllo autorizzazioni **Concedi amministratore per OpenID e autorizzazioni offline_access** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-149">Clear the **Permissions** > **Grant admin consent to openid and offline_access permissions** check box.</span></span>
1. <span data-ttu-id="d6f51-150">Selezionare **Registra**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-150">Select **Register**.</span></span>

<span data-ttu-id="d6f51-151">Registrare l' *`Client`* ID dell'applicazione App (client) (ad esempio, `4369008b-21fa-427c-abaa-9b53bf58e538` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-151">Record the *`Client`* app Application (client) ID (for example, `4369008b-21fa-427c-abaa-9b53bf58e538`).</span></span>

<span data-ttu-id="d6f51-152">Nelle configurazioni della piattaforma di **autenticazione** >  > **applicazione a pagina singola (Spa)**:</span><span class="sxs-lookup"><span data-stu-id="d6f51-152">In **Authentication** > **Platform configurations** > **Single-page application (SPA)**:</span></span>

1. <span data-ttu-id="d6f51-153">Verificare che l' **URI di reindirizzamento** di `https://localhost:{PORT}/authentication/login-callback` sia presente.</span><span class="sxs-lookup"><span data-stu-id="d6f51-153">Confirm the **Redirect URI** of `https://localhost:{PORT}/authentication/login-callback` is present.</span></span>
1. <span data-ttu-id="d6f51-154">Per **concessione implicita**, assicurarsi che le caselle di controllo per i **token di accesso** e i **token ID** **non** siano selezionate.</span><span class="sxs-lookup"><span data-stu-id="d6f51-154">For **Implicit grant**, ensure that the check boxes for **Access tokens** and **ID tokens** are **not** selected.</span></span>
1. <span data-ttu-id="d6f51-155">Per questa esperienza sono accettabili le impostazioni predefinite rimanenti per l'app.</span><span class="sxs-lookup"><span data-stu-id="d6f51-155">The remaining defaults for the app are acceptable for this experience.</span></span>
1. <span data-ttu-id="d6f51-156">Fare clic sul pulsante **Salva**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-156">Select the **Save** button.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="d6f51-157">In **Azure Active Directory** > **registrazioni app** selezionare **nuova registrazione**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-157">In **Azure Active Directory** > **App registrations**, select **New registration**.</span></span>
1. <span data-ttu-id="d6f51-158">Specificare un **nome** per l'app (ad esempio, **Blazor client AAD**).</span><span class="sxs-lookup"><span data-stu-id="d6f51-158">Provide a **Name** for the app (for example, **Blazor Client AAD**).</span></span>
1. <span data-ttu-id="d6f51-159">Scegliere i **tipi di conto supportati**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-159">Choose a **Supported account types**.</span></span> <span data-ttu-id="d6f51-160">Per questa esperienza è possibile selezionare **solo gli account in questa directory aziendale** (tenant singolo).</span><span class="sxs-lookup"><span data-stu-id="d6f51-160">You may select **Accounts in this organizational directory only** (single tenant) for this experience.</span></span>
1. <span data-ttu-id="d6f51-161">Lasciare l'elenco a discesa **URI di reindirizzamento** impostato su **Web** e specificare l'URI di reindirizzamento seguente: `https://localhost:{PORT}/authentication/login-callback` .</span><span class="sxs-lookup"><span data-stu-id="d6f51-161">Leave the **Redirect URI** drop down set to **Web** and provide the following redirect URI: `https://localhost:{PORT}/authentication/login-callback`.</span></span> <span data-ttu-id="d6f51-162">La porta predefinita per un'app in esecuzione in Kestrel è la 5001.</span><span class="sxs-lookup"><span data-stu-id="d6f51-162">The default port for an app running on Kestrel is 5001.</span></span> <span data-ttu-id="d6f51-163">Se l'app viene eseguita su una porta di Gheppio diversa, usare la porta dell'app.</span><span class="sxs-lookup"><span data-stu-id="d6f51-163">If the app is run on a different Kestrel port, use the app's port.</span></span> <span data-ttu-id="d6f51-164">Per IIS Express, la porta generata in modo casuale per l'app si trova nelle *`Server`* proprietà dell'app nel pannello **debug** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-164">For IIS Express, the randomly generated port for the app can be found in the *`Server`* app's properties in the **Debug** panel.</span></span> <span data-ttu-id="d6f51-165">Poiché l'app non esiste in questo momento e la porta IIS Express non è nota, tornare a questo passaggio dopo la creazione dell'app e aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="d6f51-165">Since the app doesn't exist at this point and the IIS Express port isn't known, return to this step after the app is created and update the redirect URI.</span></span> <span data-ttu-id="d6f51-166">Un contrassegno viene visualizzato nella sezione [creare l'app](#create-the-app) per ricordare IIS Express agli utenti di aggiornare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="d6f51-166">A remark appears in the [Create the app](#create-the-app) section to remind IIS Express users to update the redirect URI.</span></span>
1. <span data-ttu-id="d6f51-167">Deselezionare la  > casella di controllo autorizzazioni **Concedi amministratore per OpenID e autorizzazioni offline_access** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-167">Clear the **Permissions** > **Grant admin consent to openid and offline_access permissions** check box.</span></span>
1. <span data-ttu-id="d6f51-168">Selezionare **Registra**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-168">Select **Register**.</span></span>

<span data-ttu-id="d6f51-169">Registrare l' *`Client`* ID dell'applicazione App (client) (ad esempio, `4369008b-21fa-427c-abaa-9b53bf58e538` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-169">Record the *`Client`* app Application (client) ID (for example, `4369008b-21fa-427c-abaa-9b53bf58e538`).</span></span>

<span data-ttu-id="d6f51-170">In  > **configurazioni piattaforma** di autenticazione > **Web**:</span><span class="sxs-lookup"><span data-stu-id="d6f51-170">In **Authentication** > **Platform configurations** > **Web**:</span></span>

1. <span data-ttu-id="d6f51-171">Verificare che l' **URI di reindirizzamento** di `https://localhost:{PORT}/authentication/login-callback` sia presente.</span><span class="sxs-lookup"><span data-stu-id="d6f51-171">Confirm the **Redirect URI** of `https://localhost:{PORT}/authentication/login-callback` is present.</span></span>
1. <span data-ttu-id="d6f51-172">Per **concessione implicita**, selezionare le caselle di controllo per i token di **accesso** e i **token ID**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-172">For **Implicit grant**, select the check boxes for **Access tokens** and **ID tokens**.</span></span>
1. <span data-ttu-id="d6f51-173">Per questa esperienza sono accettabili le impostazioni predefinite rimanenti per l'app.</span><span class="sxs-lookup"><span data-stu-id="d6f51-173">The remaining defaults for the app are acceptable for this experience.</span></span>
1. <span data-ttu-id="d6f51-174">Fare clic sul pulsante **Salva**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-174">Select the **Save** button.</span></span>

::: moniker-end

<span data-ttu-id="d6f51-175">In **autorizzazioni API**:</span><span class="sxs-lookup"><span data-stu-id="d6f51-175">In **API permissions**:</span></span>

1. <span data-ttu-id="d6f51-176">Verificare che l'app disponga **Microsoft Graph**  >  autorizzazione **User. Read** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-176">Confirm that the app has **Microsoft Graph** > **User.Read** permission.</span></span>
1. <span data-ttu-id="d6f51-177">Selezionare **Aggiungi un'autorizzazione** seguita da **API personali**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-177">Select **Add a permission** followed by **My APIs**.</span></span>
1. <span data-ttu-id="d6f51-178">Selezionare l' *app per le API server* dalla colonna **nome** (ad esempio, **Blazor Server AAD**).</span><span class="sxs-lookup"><span data-stu-id="d6f51-178">Select the *Server API app* from the **Name** column (for example, **Blazor Server AAD**).</span></span>
1. <span data-ttu-id="d6f51-179">Aprire l'elenco di **API** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-179">Open the **API** list.</span></span>
1. <span data-ttu-id="d6f51-180">Abilitare l'accesso all'API (ad esempio, `API.Access` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-180">Enable access to the API (for example, `API.Access`).</span></span>
1. <span data-ttu-id="d6f51-181">Selezionare **Aggiungi autorizzazioni**.</span><span class="sxs-lookup"><span data-stu-id="d6f51-181">Select **Add permissions**.</span></span>
1. <span data-ttu-id="d6f51-182">Selezionare il pulsante **concedere il consenso dell'amministratore per {tenant Name}** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-182">Select the **Grant admin consent for {TENANT NAME}** button.</span></span> <span data-ttu-id="d6f51-183">Selezionare **Sì** per confermare.</span><span class="sxs-lookup"><span data-stu-id="d6f51-183">Select **Yes** to confirm.</span></span>

### <a name="create-the-app"></a><span data-ttu-id="d6f51-184">Creare l'app</span><span class="sxs-lookup"><span data-stu-id="d6f51-184">Create the app</span></span>

<span data-ttu-id="d6f51-185">In una cartella vuota, sostituire i segnaposto nel comando seguente con le informazioni registrate in precedenza ed eseguire il comando in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="d6f51-185">In an empty folder, replace the placeholders in the following command with the information recorded earlier and execute the command in a command shell:</span></span>

```dotnetcli
dotnet new blazorwasm -au SingleOrg --api-client-id "{SERVER API APP CLIENT ID}" --app-id-uri "{SERVER API APP ID URI}" --client-id "{CLIENT APP CLIENT ID}" --default-scope "{DEFAULT SCOPE}" --domain "{TENANT DOMAIN}" -ho -o {APP NAME} --tenant-id "{TENANT ID}"
```

| <span data-ttu-id="d6f51-186">Segnaposto</span><span class="sxs-lookup"><span data-stu-id="d6f51-186">Placeholder</span></span>                  | <span data-ttu-id="d6f51-187">Nome portale di Azure</span><span class="sxs-lookup"><span data-stu-id="d6f51-187">Azure portal name</span></span>                                     | <span data-ttu-id="d6f51-188">Esempio</span><span class="sxs-lookup"><span data-stu-id="d6f51-188">Example</span></span>                                        |
| ---------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| `{APP NAME}`                 | &mdash;                                               | `BlazorSample`                                 |
| `{CLIENT APP CLIENT ID}`     | <span data-ttu-id="d6f51-189">ID applicazione (client) per l' *`Client`* app</span><span class="sxs-lookup"><span data-stu-id="d6f51-189">Application (client) ID for the *`Client`* app</span></span>        | `4369008b-21fa-427c-abaa-9b53bf58e538`         |
| `{DEFAULT SCOPE}`            | <span data-ttu-id="d6f51-190">Nome ambito</span><span class="sxs-lookup"><span data-stu-id="d6f51-190">Scope name</span></span>                                            | `API.Access`                                   |
| `{SERVER API APP CLIENT ID}` | <span data-ttu-id="d6f51-191">ID applicazione (client) per l' *app per le API server*</span><span class="sxs-lookup"><span data-stu-id="d6f51-191">Application (client) ID for the *Server API app*</span></span>      | `41451fa7-82d9-4673-8fa5-69eff5a761fd`         |
| `{SERVER API APP ID URI}`    | <span data-ttu-id="d6f51-192">URI dell'ID applicazione&dagger;</span><span class="sxs-lookup"><span data-stu-id="d6f51-192">Application ID URI&dagger;</span></span>                            | `41451fa7-82d9-4673-8fa5-69eff5a761fd`&dagger; |
| `{TENANT DOMAIN}`            | <span data-ttu-id="d6f51-193">Dominio primario/server di pubblicazione/tenant</span><span class="sxs-lookup"><span data-stu-id="d6f51-193">Primary/Publisher/Tenant domain</span></span>                       | `contoso.onmicrosoft.com`                      |
| `{TENANT ID}`                | <span data-ttu-id="d6f51-194">ID directory (tenant)</span><span class="sxs-lookup"><span data-stu-id="d6f51-194">Directory (tenant) ID</span></span>                                 | `e86c78e2-8bb4-4c41-aefd-918e0565a45e`         |

<span data-ttu-id="d6f51-195">&dagger;Il Blazor WebAssembly modello aggiunge automaticamente uno schema `api://` all'argomento Uri ID app passato nel `dotnet new` comando.</span><span class="sxs-lookup"><span data-stu-id="d6f51-195">&dagger;The Blazor WebAssembly template automatically adds a scheme of `api://` to the App ID URI argument passed in the `dotnet new` command.</span></span> <span data-ttu-id="d6f51-196">Quando si specifica l'URI ID app per il `{SERVER API APP ID URI}` segnaposto e se lo schema è `api://` , rimuovere lo schema ( `api://` ) dall'argomento, come illustrato nel valore di esempio della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="d6f51-196">When providing the App ID URI for the `{SERVER API APP ID URI}` placeholder and if the scheme is `api://`, remove the scheme (`api://`) from the argument, as the example value in the preceding table shows.</span></span> <span data-ttu-id="d6f51-197">Se l'URI ID app è un valore personalizzato o ha un altro schema (ad esempio, `https://` per un dominio di pubblicazione non trusted simile a `https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd` ), è necessario aggiornare manualmente l'URI dell'ambito predefinito e rimuovere lo `api://` schema dopo che l' *`Client`* app è stata creata dal modello.</span><span class="sxs-lookup"><span data-stu-id="d6f51-197">If the App ID URI is a custom value or has some other scheme (for example, `https://` for an untrusted publisher domain similar to `https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd`), you must manually update the default scope URI and remove the `api://` scheme after the *`Client`* app is created by the template.</span></span> <span data-ttu-id="d6f51-198">Per ulteriori informazioni, vedere la nota nella sezione [ambiti dei token di accesso](#access-token-scopes) .</span><span class="sxs-lookup"><span data-stu-id="d6f51-198">For more information, see the note in the [Access token scopes](#access-token-scopes) section.</span></span> <span data-ttu-id="d6f51-199">Blazor WebAssemblyÈ possibile che il modello venga modificato in una versione futura di ASP.NET Core per risolvere questi scenari.</span><span class="sxs-lookup"><span data-stu-id="d6f51-199">The Blazor WebAssembly template might be changed in a future release of ASP.NET Core to address these scenarios.</span></span> <span data-ttu-id="d6f51-200">Per altre informazioni, vedere [schema doppio per URI ID app con il Blazor modello WASM (hosted, Single org) (DotNet/aspnetcore #27417)](https://github.com/dotnet/aspnetcore/issues/27417).</span><span class="sxs-lookup"><span data-stu-id="d6f51-200">For more information, see [Double scheme for App ID URI with Blazor WASM template (hosted, single org) (dotnet/aspnetcore #27417)](https://github.com/dotnet/aspnetcore/issues/27417).</span></span>

<span data-ttu-id="d6f51-201">Il percorso di output specificato con l'opzione `-o|--output` consente di creare una cartella di progetto se non esiste e viene incluso nel nome dell'app.</span><span class="sxs-lookup"><span data-stu-id="d6f51-201">The output location specified with the `-o|--output` option creates a project folder if it doesn't exist and becomes part of the app's name.</span></span>

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="d6f51-202">Potrebbe essere necessaria una modifica alla configurazione quando si usa un tenant di Azure con un dominio Publisher non verificato, descritto nella sezione [impostazioni app](#app-settings) .</span><span class="sxs-lookup"><span data-stu-id="d6f51-202">A configuration change might be required when using an Azure tenant with an unverified publisher domain, which is described in the [App settings](#app-settings) section.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="d6f51-203">Potrebbe essere necessaria una modifica alla configurazione quando si usa un tenant di Azure con un dominio del server di pubblicazione non verificato, descritto nella sezione [ambiti dei token di accesso](#access-token-scopes) .</span><span class="sxs-lookup"><span data-stu-id="d6f51-203">A configuration change might be required when using an Azure tenant with an unverified publisher domain, which is described in the [Access token scopes](#access-token-scopes) section.</span></span>

::: moniker-end

> [!NOTE]
> <span data-ttu-id="d6f51-204">Nell'portale di Azure l' *`Client`* **URI di reindirizzamento** della configurazione della piattaforma dell'app è configurato per la porta 5001 per le app eseguite nel server gheppio con le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="d6f51-204">In the Azure portal, the *`Client`* app's platform configuration **Redirect URI** is configured for port 5001 for apps that run on the Kestrel server with default settings.</span></span>
>
> <span data-ttu-id="d6f51-205">Se l' *`Client`* app viene eseguita su una porta IIS Express casuale, la porta per l'app si trova nelle proprietà dell' *app* per le API del server nel pannello **debug** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-205">If the *`Client`* app is run on a random IIS Express port, the port for the app can be found in the *Server API app's* properties in the **Debug** panel.</span></span>
>
> <span data-ttu-id="d6f51-206">Se la porta non è stata configurata in precedenza con la *`Client`* porta nota dell'app, tornare alla *`Client`* registrazione dell'app nel portale di Azure e aggiornare l'URI di reindirizzamento con la porta corretta.</span><span class="sxs-lookup"><span data-stu-id="d6f51-206">If the port wasn't configured earlier with the *`Client`* app's known port, return to the *`Client`* app's registration in the Azure portal and update the redirect URI with the correct port.</span></span>

## <a name="server-app-configuration"></a><span data-ttu-id="d6f51-207">*`Server`* configurazione dell'app</span><span class="sxs-lookup"><span data-stu-id="d6f51-207">*`Server`* app configuration</span></span>

<span data-ttu-id="d6f51-208">*Questa sezione è relativa all'app della soluzione **`Server`** .*</span><span class="sxs-lookup"><span data-stu-id="d6f51-208">*This section pertains to the solution's **`Server`** app.*</span></span>

### <a name="authentication-package"></a><span data-ttu-id="d6f51-209">Pacchetto di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d6f51-209">Authentication package</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="d6f51-210">Il supporto per l'autenticazione e l'autorizzazione delle chiamate a ASP.NET Core API Web con la Identity piattaforma Microsoft è fornito dai pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="d6f51-210">The support for authenticating and authorizing calls to ASP.NET Core Web APIs with the Microsoft Identity Platform is provided by the following packages:</span></span>

* [`Microsoft.Identity.Web`](https://www.nuget.org/packages/Microsoft.Identity.Web)
* [`Microsoft.Identity.Web.UI`](https://www.nuget.org/packages/Microsoft.Identity.Web.UI)

```xml
<PackageReference Include="Microsoft.Identity.Web" Version="{VERSION}" />
<PackageReference Include="Microsoft.Identity.Web.UI" Version="{VERSION}" />
```

<span data-ttu-id="d6f51-211">Per il segnaposto `{VERSION}` , la versione stabile più recente del pacchetto che corrisponde alla versione del Framework condiviso dell'app è disponibile nella cronologia delle **versioni** del pacchetto in NuGet.org.</span><span class="sxs-lookup"><span data-stu-id="d6f51-211">For the placeholder `{VERSION}`, the latest stable version of the package that matches the app's shared framework version can be found in the package's **Version History** at NuGet.org.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="d6f51-212">Il pacchetto fornisce il supporto per l'autenticazione e l'autorizzazione delle chiamate a ASP.NET Core API Web [`Microsoft.AspNetCore.Authentication.AzureAD.UI`](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureAD.UI) :</span><span class="sxs-lookup"><span data-stu-id="d6f51-212">The support for authenticating and authorizing calls to ASP.NET Core Web APIs is provided by the [`Microsoft.AspNetCore.Authentication.AzureAD.UI`](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureAD.UI) package:</span></span>

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.AzureAD.UI" 
  Version="{VERSION}" />
```

<span data-ttu-id="d6f51-213">Per il segnaposto `{VERSION}` , la versione stabile più recente del pacchetto che corrisponde alla versione del Framework condiviso dell'app è disponibile nella cronologia delle **versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureAD.UI).</span><span class="sxs-lookup"><span data-stu-id="d6f51-213">For the placeholder `{VERSION}`, the latest stable version of the package that matches the app's shared framework version can be found in the package's **Version History** at [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureAD.UI).</span></span>

::: moniker-end

### <a name="authentication-service-support"></a><span data-ttu-id="d6f51-214">Supporto del servizio di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d6f51-214">Authentication service support</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="d6f51-215">Il `AddAuthentication` metodo imposta i servizi di autenticazione all'interno dell'app e configura il gestore di JWT Bearer come metodo di autenticazione predefinito.</span><span class="sxs-lookup"><span data-stu-id="d6f51-215">The `AddAuthentication` method sets up authentication services within the app and configures the JWT Bearer handler as the default authentication method.</span></span> <span data-ttu-id="d6f51-216">Il <xref:Microsoft.Identity.Web.MicrosoftIdentityWebApiAuthenticationBuilderExtensions.AddMicrosoftIdentityWebApi%2A> metodo configura i servizi per proteggere l'API Web con Microsoft Identity Platform v 2.0.</span><span class="sxs-lookup"><span data-stu-id="d6f51-216">The <xref:Microsoft.Identity.Web.MicrosoftIdentityWebApiAuthenticationBuilderExtensions.AddMicrosoftIdentityWebApi%2A> method configures services to protect the web API with Microsoft Identity Platform v2.0.</span></span> <span data-ttu-id="d6f51-217">Questo metodo prevede una `AzureAd` sezione nella configurazione dell'app con le impostazioni necessarie per inizializzare le opzioni di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d6f51-217">This method expects an `AzureAd` section in the app's configuration with the necessary settings to initialize authentication options.</span></span>

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(Configuration.GetSection("AzureAd"));
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="d6f51-218">Il `AddAuthentication` metodo imposta i servizi di autenticazione all'interno dell'app e configura il gestore di JWT Bearer come metodo di autenticazione predefinito.</span><span class="sxs-lookup"><span data-stu-id="d6f51-218">The `AddAuthentication` method sets up authentication services within the app and configures the JWT Bearer handler as the default authentication method.</span></span> <span data-ttu-id="d6f51-219">Il <xref:Microsoft.AspNetCore.Authentication.AzureADAuthenticationBuilderExtensions.AddAzureADBearer%2A> metodo imposta i parametri specifici nel gestore di JWT Bearer necessario per convalidare i token emessi dal Azure Active Directory:</span><span class="sxs-lookup"><span data-stu-id="d6f51-219">The <xref:Microsoft.AspNetCore.Authentication.AzureADAuthenticationBuilderExtensions.AddAzureADBearer%2A> method sets up the specific parameters in the JWT Bearer handler required to validate tokens emitted by the Azure Active Directory:</span></span>

```csharp
services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
```

::: moniker-end

<span data-ttu-id="d6f51-220"><xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A> e <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> verificare che:</span><span class="sxs-lookup"><span data-stu-id="d6f51-220"><xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A> and <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> ensure that:</span></span>

* <span data-ttu-id="d6f51-221">L'app tenta di analizzare e convalidare i token nelle richieste in ingresso.</span><span class="sxs-lookup"><span data-stu-id="d6f51-221">The app attempts to parse and validate tokens on incoming requests.</span></span>
* <span data-ttu-id="d6f51-222">Eventuali richieste che tentano di accedere a una risorsa protetta senza credenziali appropriate hanno esito negativo.</span><span class="sxs-lookup"><span data-stu-id="d6f51-222">Any request attempting to access a protected resource without proper credentials fails.</span></span>

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### <a name="useridentityname"></a><span data-ttu-id="d6f51-223">Utente. Identity . Nome</span><span class="sxs-lookup"><span data-stu-id="d6f51-223">User.Identity.Name</span></span>

<span data-ttu-id="d6f51-224">Per impostazione predefinita, l' *`Server`* API dell'app viene popolata `User.Identity.Name` con il valore del `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` tipo di attestazione (ad esempio, `2d64b3da-d9d5-42c6-9352-53d8df33d770@contoso.onmicrosoft.com` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-224">By default, the *`Server`* app API populates `User.Identity.Name` with the value from the `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` claim type (for example, `2d64b3da-d9d5-42c6-9352-53d8df33d770@contoso.onmicrosoft.com`).</span></span>

<span data-ttu-id="d6f51-225">Per configurare l'app per la ricezione del valore dal `name` tipo di attestazione:</span><span class="sxs-lookup"><span data-stu-id="d6f51-225">To configure the app to receive the value from the `name` claim type:</span></span>

* <span data-ttu-id="d6f51-226">Aggiungere uno spazio dei nomi per <xref:Microsoft.AspNetCore.Authentication.JwtBearer?displayProperty=fullName> a `Startup.cs` :</span><span class="sxs-lookup"><span data-stu-id="d6f51-226">Add a namespace for <xref:Microsoft.AspNetCore.Authentication.JwtBearer?displayProperty=fullName> to `Startup.cs`:</span></span>

  ```csharp
  using Microsoft.AspNetCore.Authentication.JwtBearer;
  ```

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="d6f51-227">Configurare <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType?displayProperty=nameWithType> di <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6f51-227">Configure the <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType?displayProperty=nameWithType> of the <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> in `Startup.ConfigureServices`:</span></span>

  ```csharp
  services.Configure<JwtBearerOptions>(
      JwtBearerDefaults.AuthenticationScheme, options =>
      {
          options.TokenValidationParameters.NameClaimType = "name";
      });
  ```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <span data-ttu-id="d6f51-228">Configurare <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType?displayProperty=nameWithType> di <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6f51-228">Configure the <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType?displayProperty=nameWithType> of the <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> in `Startup.ConfigureServices`:</span></span>

  ```csharp
  services.Configure<JwtBearerOptions>(
      AzureADDefaults.JwtBearerAuthenticationScheme, options =>
      {
          options.TokenValidationParameters.NameClaimType = "name";
      });
  ```

::: moniker-end

### <a name="app-settings"></a><span data-ttu-id="d6f51-229">Impostazioni app</span><span class="sxs-lookup"><span data-stu-id="d6f51-229">App settings</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="d6f51-230">Il `appsettings.json` file contiene le opzioni per configurare il gestore di connessione JWT usato per convalidare i token di accesso:</span><span class="sxs-lookup"><span data-stu-id="d6f51-230">The `appsettings.json` file contains the options to configure the JWT bearer handler used to validate access tokens:</span></span>

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "{DOMAIN}",
    "TenantId": "{TENANT ID}",
    "ClientId": "{SERVER API APP CLIENT ID}",
    "CallbackPath": "/signin-oidc"
  }
}
```

<span data-ttu-id="d6f51-231">Esempio:</span><span class="sxs-lookup"><span data-stu-id="d6f51-231">Example:</span></span>

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "contoso.onmicrosoft.com",
    "TenantId": "e86c78e2-8bb4-4c41-aefd-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
    "CallbackPath": "/signin-oidc"
  }
}
```

[!INCLUDE[](~/blazor/includes/security/azure-scope-5x.md)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="d6f51-232">Il `appsettings.json` file contiene le opzioni per configurare il gestore di connessione JWT usato per convalidare i token di accesso:</span><span class="sxs-lookup"><span data-stu-id="d6f51-232">The `appsettings.json` file contains the options to configure the JWT bearer handler used to validate access tokens:</span></span>

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "{DOMAIN}",
    "TenantId": "{TENANT ID}",
    "ClientId": "{SERVER API APP CLIENT ID}",
  }
}
```

<span data-ttu-id="d6f51-233">Esempio:</span><span class="sxs-lookup"><span data-stu-id="d6f51-233">Example:</span></span>

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "contoso.onmicrosoft.com",
    "TenantId": "e86c78e2-8bb4-4c41-aefd-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
  }
}
```

::: moniker-end

### <a name="weatherforecast-controller"></a><span data-ttu-id="d6f51-234">Controller WeatherForecast</span><span class="sxs-lookup"><span data-stu-id="d6f51-234">WeatherForecast controller</span></span>

<span data-ttu-id="d6f51-235">Il controller WeatherForecast (*Controllers/WeatherForecastController. cs*) espone un'API protetta con l' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) applicato al controller.</span><span class="sxs-lookup"><span data-stu-id="d6f51-235">The WeatherForecast controller (*Controllers/WeatherForecastController.cs*) exposes a protected API with the [`[Authorize]` attribute](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) applied to the controller.</span></span> <span data-ttu-id="d6f51-236">È **importante** comprendere che:</span><span class="sxs-lookup"><span data-stu-id="d6f51-236">It's **important** to understand that:</span></span>

* <span data-ttu-id="d6f51-237">L' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) in questo controller API è l'unico elemento che protegge questa API da accessi non autorizzati.</span><span class="sxs-lookup"><span data-stu-id="d6f51-237">The [`[Authorize]` attribute](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) in this API controller is the only thing that protect this API from unauthorized access.</span></span>
* <span data-ttu-id="d6f51-238">L' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) usato nell' Blazor WebAssembly app funge solo da hint per l'app che l'utente deve essere autorizzato affinché l'app funzioni correttamente.</span><span class="sxs-lookup"><span data-stu-id="d6f51-238">The [`[Authorize]` attribute](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) used in the Blazor WebAssembly app only serves as a hint to the app that the user should be authorized for the app to work correctly.</span></span>

```csharp
[Authorize]
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        ...
    }
}
```

## <a name="client-app-configuration"></a><span data-ttu-id="d6f51-239">*`Client`* configurazione dell'app</span><span class="sxs-lookup"><span data-stu-id="d6f51-239">*`Client`* app configuration</span></span>

<span data-ttu-id="d6f51-240">*Questa sezione è relativa all'app della soluzione **`Client`** .*</span><span class="sxs-lookup"><span data-stu-id="d6f51-240">*This section pertains to the solution's **`Client`** app.*</span></span>

### <a name="authentication-package"></a><span data-ttu-id="d6f51-241">Pacchetto di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d6f51-241">Authentication package</span></span>

<span data-ttu-id="d6f51-242">Quando viene creata un'app per usare gli account aziendali o dell'Istituto di istruzione ( `SingleOrg` ), l'app riceve automaticamente un riferimento al pacchetto per [Microsoft Authentication Library](/azure/active-directory/develop/msal-overview) ( [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-242">When an app is created to use Work or School Accounts (`SingleOrg`), the app automatically receives a package reference for the [Microsoft Authentication Library](/azure/active-directory/develop/msal-overview) ([`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal)).</span></span> <span data-ttu-id="d6f51-243">Il pacchetto fornisce un set di primitive che consentono all'app di autenticare gli utenti e ottenere i token per chiamare le API protette.</span><span class="sxs-lookup"><span data-stu-id="d6f51-243">The package provides a set of primitives that help the app authenticate users and obtain tokens to call protected APIs.</span></span>

<span data-ttu-id="d6f51-244">Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il pacchetto al file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="d6f51-244">If adding authentication to an app, manually add the package to the app's project file:</span></span>

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
  Version="{VERSION}" />
```

<span data-ttu-id="d6f51-245">Per il segnaposto `{VERSION}` , la versione stabile più recente del pacchetto che corrisponde alla versione del Framework condiviso dell'app è disponibile nella cronologia delle **versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal).</span><span class="sxs-lookup"><span data-stu-id="d6f51-245">For the placeholder `{VERSION}`, the latest stable version of the package that matches the app's shared framework version can be found in the package's **Version History** at [NuGet.org](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal).</span></span>

<span data-ttu-id="d6f51-246">Il [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) pacchetto aggiunge il [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) pacchetto all'app in modo transitivo.</span><span class="sxs-lookup"><span data-stu-id="d6f51-246">The [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) package transitively adds the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) package to the app.</span></span>

### <a name="authentication-service-support"></a><span data-ttu-id="d6f51-247">Supporto del servizio di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d6f51-247">Authentication service support</span></span>

<span data-ttu-id="d6f51-248"><xref:System.Net.Http.HttpClient>Viene aggiunto il supporto per le istanze di che includono token di accesso quando si effettuano richieste al progetto server.</span><span class="sxs-lookup"><span data-stu-id="d6f51-248">Support for <xref:System.Net.Http.HttpClient> instances is added that include access tokens when making requests to the server project.</span></span>

<span data-ttu-id="d6f51-249">`Program.cs`:</span><span class="sxs-lookup"><span data-stu-id="d6f51-249">`Program.cs`:</span></span>

```csharp
builder.Services.AddHttpClient("{APP ASSEMBLY}.ServerAPI", client => 
        client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("{APP ASSEMBLY}.ServerAPI"));
```

<span data-ttu-id="d6f51-250">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample.ServerAPI` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-250">The placeholder `{APP ASSEMBLY}` is the app's assembly name (for example, `BlazorSample.ServerAPI`).</span></span>

<span data-ttu-id="d6f51-251">Il supporto per l'autenticazione degli utenti viene registrato nel contenitore del servizio con il <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> metodo di estensione fornito dal [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="d6f51-251">Support for authenticating users is registered in the service container with the <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> extension method provided by the [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal) package.</span></span> <span data-ttu-id="d6f51-252">Questo metodo configura i servizi necessari per l'interazione dell'app con il Identity provider (IP).</span><span class="sxs-lookup"><span data-stu-id="d6f51-252">This method sets up the services required for the app to interact with the Identity Provider (IP).</span></span>

<span data-ttu-id="d6f51-253">`Program.cs`:</span><span class="sxs-lookup"><span data-stu-id="d6f51-253">`Program.cs`:</span></span>

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

<span data-ttu-id="d6f51-254">Il <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> metodo accetta un callback per configurare i parametri necessari per autenticare un'app.</span><span class="sxs-lookup"><span data-stu-id="d6f51-254">The <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> method accepts a callback to configure the parameters required to authenticate an app.</span></span> <span data-ttu-id="d6f51-255">Quando si registra l'app, è possibile ottenere i valori necessari per la configurazione dell'app dalla configurazione di AAD del portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="d6f51-255">The values required for configuring the app can be obtained from the Azure Portal AAD configuration when you register the app.</span></span>

<span data-ttu-id="d6f51-256">La configurazione viene fornita dal `wwwroot/appsettings.json` file:</span><span class="sxs-lookup"><span data-stu-id="d6f51-256">Configuration is supplied by the `wwwroot/appsettings.json` file:</span></span>

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/{TENANT ID}",
    "ClientId": "{CLIENT APP CLIENT ID}",
    "ValidateAuthority": true
  }
}
```

<span data-ttu-id="d6f51-257">Esempio:</span><span class="sxs-lookup"><span data-stu-id="d6f51-257">Example:</span></span>

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/e86c78e2-...-918e0565a45e",
    "ClientId": "4369008b-21fa-427c-abaa-9b53bf58e538",
    "ValidateAuthority": true
  }
}
```

### <a name="access-token-scopes"></a><span data-ttu-id="d6f51-258">Ambiti del token di accesso</span><span class="sxs-lookup"><span data-stu-id="d6f51-258">Access token scopes</span></span>

<span data-ttu-id="d6f51-259">Gli ambiti dei token di accesso predefiniti rappresentano l'elenco degli ambiti dei token di accesso:</span><span class="sxs-lookup"><span data-stu-id="d6f51-259">The default access token scopes represent the list of access token scopes that are:</span></span>

* <span data-ttu-id="d6f51-260">Incluso per impostazione predefinita nella richiesta di accesso.</span><span class="sxs-lookup"><span data-stu-id="d6f51-260">Included by default in the sign in request.</span></span>
* <span data-ttu-id="d6f51-261">Utilizzato per eseguire il provisioning di un token di accesso immediatamente dopo l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d6f51-261">Used to provision an access token immediately after authentication.</span></span>

<span data-ttu-id="d6f51-262">Tutti gli ambiti devono appartenere alla stessa app per ogni regola di Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="d6f51-262">All scopes must belong to the same app per Azure Active Directory rules.</span></span> <span data-ttu-id="d6f51-263">Per altre app per le API è possibile aggiungere altri ambiti, se necessario:</span><span class="sxs-lookup"><span data-stu-id="d6f51-263">Additional scopes can be added for additional API apps as needed:</span></span>

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

> [!NOTE]
> <span data-ttu-id="d6f51-264">Il Blazor WebAssembly modello aggiunge automaticamente uno schema `api://` all'argomento Uri ID app passato nel `dotnet new` comando.</span><span class="sxs-lookup"><span data-stu-id="d6f51-264">The Blazor WebAssembly template automatically adds a scheme of `api://` to the App ID URI argument passed in the `dotnet new` command.</span></span> <span data-ttu-id="d6f51-265">Quando si genera un'app dal Blazor modello di progetto, verificare che il valore dell'ambito del token di accesso predefinito usi il valore URI dell'ID app personalizzato corretto specificato nell'portale di Azure o un valore con **uno** dei formati seguenti:</span><span class="sxs-lookup"><span data-stu-id="d6f51-265">When generating an app from the Blazor project template, confirm that the value of the default access token scope uses either the correct custom App ID URI value that you provided in the Azure portal or a value with **one** of the following formats:</span></span>
>
> * <span data-ttu-id="d6f51-266">Quando il dominio del server di pubblicazione della directory è **attendibile**, l'ambito del token di accesso predefinito è in genere un valore simile all'esempio seguente, dove `API.Access` è il nome dell'ambito predefinito:</span><span class="sxs-lookup"><span data-stu-id="d6f51-266">When the publisher domain of the directory is **trusted**, the default access token scope is typically a value similar to the following example, where `API.Access` is the default scope name:</span></span>
>
>   ```csharp
>   options.ProviderOptions.DefaultAccessTokenScopes.Add(
>       "api://41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
>   ```
>
>   <span data-ttu-id="d6f51-267">Esaminare il valore per uno schema doppio ( `api://api://...` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-267">Inspect the value for a double scheme (`api://api://...`).</span></span> <span data-ttu-id="d6f51-268">Se è presente uno schema doppio, rimuovere il primo `api://` schema dal valore.</span><span class="sxs-lookup"><span data-stu-id="d6f51-268">If a double scheme is present, remove the first `api://` scheme from the value.</span></span>
>
> * <span data-ttu-id="d6f51-269">Quando il dominio del server di pubblicazione della directory non è **attendibile**, l'ambito del token di accesso predefinito è in genere un valore simile all'esempio seguente, dove `API.Access` è il nome dell'ambito predefinito:</span><span class="sxs-lookup"><span data-stu-id="d6f51-269">When the publisher domain of the directory is **untrusted**, the default access token scope is typically a value similar to the following example, where `API.Access` is the default scope name:</span></span>
>
>   ```csharp
>   options.ProviderOptions.DefaultAccessTokenScopes.Add(
>       "https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
>   ```
>
>   <span data-ttu-id="d6f51-270">Esaminare il valore per uno `api://` schema aggiuntivo ( `api://https://contoso.onmicrosoft.com/...` ).</span><span class="sxs-lookup"><span data-stu-id="d6f51-270">Inspect the value for an extra `api://` scheme (`api://https://contoso.onmicrosoft.com/...`).</span></span> <span data-ttu-id="d6f51-271">Se `api://` è presente uno schema aggiuntivo, rimuovere lo `api://` schema dal valore.</span><span class="sxs-lookup"><span data-stu-id="d6f51-271">If an extra `api://` scheme is present, remove the `api://` scheme from the value.</span></span>
>
> <span data-ttu-id="d6f51-272">Blazor WebAssemblyÈ possibile che il modello venga modificato in una versione futura di ASP.NET Core per risolvere questi scenari.</span><span class="sxs-lookup"><span data-stu-id="d6f51-272">The Blazor WebAssembly template might be changed in a future release of ASP.NET Core to address these scenarios.</span></span> <span data-ttu-id="d6f51-273">Per altre informazioni, vedere [schema doppio per URI ID app con il Blazor modello WASM (hosted, Single org) (DotNet/aspnetcore #27417)](https://github.com/dotnet/aspnetcore/issues/27417).</span><span class="sxs-lookup"><span data-stu-id="d6f51-273">For more information, see [Double scheme for App ID URI with Blazor WASM template (hosted, single org) (dotnet/aspnetcore #27417)](https://github.com/dotnet/aspnetcore/issues/27417).</span></span>

<span data-ttu-id="d6f51-274">Specificare gli ambiti aggiuntivi con `AdditionalScopesToConsent` :</span><span class="sxs-lookup"><span data-stu-id="d6f51-274">Specify additional scopes with `AdditionalScopesToConsent`:</span></span>

```csharp
options.ProviderOptions.AdditionalScopesToConsent.Add("{ADDITIONAL SCOPE URI}");
```

::: moniker range="< aspnetcore-5.0"

[!INCLUDE[](~/blazor/includes/security/azure-scope-3x.md)]

::: moniker-end

<span data-ttu-id="d6f51-275">Per ulteriori informazioni, vedere le sezioni seguenti dell'articolo *scenari aggiuntivi* :</span><span class="sxs-lookup"><span data-stu-id="d6f51-275">For more information, see the following sections of the *Additional scenarios* article:</span></span>

* [<span data-ttu-id="d6f51-276">Richiedere token di accesso aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="d6f51-276">Request additional access tokens</span></span>](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [<span data-ttu-id="d6f51-277">Connetti token alle richieste in uscita</span><span class="sxs-lookup"><span data-stu-id="d6f51-277">Attach tokens to outgoing requests</span></span>](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)

::: moniker range=">= aspnetcore-5.0"

### <a name="login-mode"></a><span data-ttu-id="d6f51-278">Modalità di accesso</span><span class="sxs-lookup"><span data-stu-id="d6f51-278">Login mode</span></span>

[!INCLUDE[](~/blazor/includes/security/msal-login-mode.md)]

::: moniker-end

### <a name="imports-file"></a><span data-ttu-id="d6f51-279">Importa file</span><span class="sxs-lookup"><span data-stu-id="d6f51-279">Imports file</span></span>

[!INCLUDE[](~/blazor/includes/security/imports-file-hosted.md)]

### <a name="index-page"></a><span data-ttu-id="d6f51-280">Pagina di indice</span><span class="sxs-lookup"><span data-stu-id="d6f51-280">Index page</span></span>

[!INCLUDE[](~/blazor/includes/security/index-page-msal.md)]

### <a name="app-component"></a><span data-ttu-id="d6f51-281">Componente app</span><span class="sxs-lookup"><span data-stu-id="d6f51-281">App component</span></span>

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

### <a name="redirecttologin-component"></a><span data-ttu-id="d6f51-282">Componente RedirectToLogin</span><span class="sxs-lookup"><span data-stu-id="d6f51-282">RedirectToLogin component</span></span>

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a><span data-ttu-id="d6f51-283">Componente LoginDisplay</span><span class="sxs-lookup"><span data-stu-id="d6f51-283">LoginDisplay component</span></span>

[!INCLUDE[](~/blazor/includes/security/logindisplay-component.md)]

### <a name="authentication-component"></a><span data-ttu-id="d6f51-284">Componente di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d6f51-284">Authentication component</span></span>

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

### <a name="fetchdata-component"></a><span data-ttu-id="d6f51-285">Componente FetchData</span><span class="sxs-lookup"><span data-stu-id="d6f51-285">FetchData component</span></span>

[!INCLUDE[](~/blazor/includes/security/fetchdata-component.md)]

## <a name="run-the-app"></a><span data-ttu-id="d6f51-286">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="d6f51-286">Run the app</span></span>

<span data-ttu-id="d6f51-287">Eseguire l'app dal progetto server.</span><span class="sxs-lookup"><span data-stu-id="d6f51-287">Run the app from the Server project.</span></span> <span data-ttu-id="d6f51-288">Quando si usa Visual Studio, eseguire una delle operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="d6f51-288">When using Visual Studio, either:</span></span>

* <span data-ttu-id="d6f51-289">Impostare l'elenco a discesa **progetti di avvio** nella barra degli strumenti sull'app per le *API server* e selezionare il pulsante **Esegui** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-289">Set the **Startup Projects** drop down list in the toolbar to the *Server API app* and select the **Run** button.</span></span>
* <span data-ttu-id="d6f51-290">Selezionare il progetto server in **Esplora soluzioni** , quindi fare clic sul pulsante **Esegui** nella barra degli strumenti o avviare l'app dal menu **debug** .</span><span class="sxs-lookup"><span data-stu-id="d6f51-290">Select the Server project in **Solution Explorer** and select the **Run** button in the toolbar or start the app from the **Debug** menu.</span></span>

<!-- HOLD
[!INCLUDE[](~/blazor/includes/security/usermanager-signinmanager.md)]
-->

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a><span data-ttu-id="d6f51-291">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="d6f51-291">Additional resources</span></span>

* <xref:blazor/security/webassembly/additional-scenarios>
* [<span data-ttu-id="d6f51-292">Compilare una versione personalizzata della libreria JavaScript Authentication. MSAL</span><span class="sxs-lookup"><span data-stu-id="d6f51-292">Build a custom version of the Authentication.MSAL JavaScript library</span></span>](xref:blazor/security/webassembly/additional-scenarios#build-a-custom-version-of-the-authenticationmsal-javascript-library)
* [<span data-ttu-id="d6f51-293">Richieste API Web non autenticate o non autorizzate in un'app con un client predefinito sicuro</span><span class="sxs-lookup"><span data-stu-id="d6f51-293">Unauthenticated or unauthorized web API requests in an app with a secure default client</span></span>](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:blazor/security/webassembly/aad-groups-roles>
* <xref:security/authentication/azure-active-directory/index>
* [<span data-ttu-id="d6f51-294">Documentazione di Microsoft Identity Platform</span><span class="sxs-lookup"><span data-stu-id="d6f51-294">Microsoft identity platform documentation</span></span>](/azure/active-directory/develop/)
* [<span data-ttu-id="d6f51-295">Avvio rapido: Registrare un'applicazione con Microsoft Identity Platform</span><span class="sxs-lookup"><span data-stu-id="d6f51-295">Quickstart: Register an application with the Microsoft identity platform</span></span>](/azure/active-directory/develop/quickstart-register-app)
