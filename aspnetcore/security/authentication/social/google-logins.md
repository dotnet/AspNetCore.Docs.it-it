---
title: Installazione di Google External login in ASP.NET Core
author: rick-anderson
description: Questa esercitazione illustra l'integrazione dell'autenticazione utente dell'account Google in un'app ASP.NET Core esistente.
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 02/18/2021
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
uid: security/authentication/google-logins
ms.openlocfilehash: 181ce87e8085839e0fcc0d77010c588ef7a290b1
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110131"
---
# <a name="google-external-login-setup-in-aspnet-core"></a><span data-ttu-id="06b26-103">Installazione di Google External login in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="06b26-103">Google external login setup in ASP.NET Core</span></span>

<span data-ttu-id="06b26-104">Da [Valeriy Novytskyy](https://github.com/01binary) e [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="06b26-104">By [Valeriy Novytskyy](https://github.com/01binary) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="06b26-105">Questa esercitazione illustra come consentire agli utenti di accedere con il proprio account Google usando il progetto ASP.NET Core 3,0 creato nella [pagina precedente](xref:security/authentication/social/index).</span><span class="sxs-lookup"><span data-stu-id="06b26-105">This tutorial shows you how to enable users to sign in with their Google account using the ASP.NET Core 3.0 project created on the [previous page](xref:security/authentication/social/index).</span></span>

## <a name="create-a-google-api-console-project-and-client-id"></a><span data-ttu-id="06b26-106">Creare un progetto console e un ID client di Google API</span><span class="sxs-lookup"><span data-stu-id="06b26-106">Create a Google API Console project and client ID</span></span>

* <span data-ttu-id="06b26-107">Aggiungere il pacchetto [Microsoft. AspNetCore. Authentication. Google](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Google) NuGet all'app.</span><span class="sxs-lookup"><span data-stu-id="06b26-107">Add the [Microsoft.AspNetCore.Authentication.Google](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Google) NuGet package to the app.</span></span>
* <span data-ttu-id="06b26-108">Seguire le istruzioni riportate in [integrazione di google Sign-In nell'app Web](https://developers.google.com/identity/sign-in/web/sign-in) (documentazione di Google).</span><span class="sxs-lookup"><span data-stu-id="06b26-108">Follow the guidance in [Integrating Google Sign-In into your web app](https://developers.google.com/identity/sign-in/web/sign-in) (Google documentation).</span></span>
* <span data-ttu-id="06b26-109">Nella pagina **credenziali** di [Google console](https://console.developers.google.com/apis/credentials)Selezionare **create Credentials**  >  **OAuth client ID**.</span><span class="sxs-lookup"><span data-stu-id="06b26-109">In the **Credentials** page of the [Google console](https://console.developers.google.com/apis/credentials), select **CREATE CREDENTIALS** > **OAuth client ID**.</span></span>
* <span data-ttu-id="06b26-110">Nella finestra di dialogo **tipo di applicazione** selezionare **applicazione Web**.</span><span class="sxs-lookup"><span data-stu-id="06b26-110">In the **Application type** dialog, select **Web application**.</span></span> <span data-ttu-id="06b26-111">Specificare un **nome** per l'app.</span><span class="sxs-lookup"><span data-stu-id="06b26-111">Provide a **Name** for the app.</span></span>
* <span data-ttu-id="06b26-112">Nella sezione **URI di reindirizzamento autorizzati** selezionare **Aggiungi URI** per impostare l'URI di reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="06b26-112">In the **Authorized redirect URIs** section, select **ADD URI** to set the redirect URI.</span></span> <span data-ttu-id="06b26-113">URI di reindirizzamento di esempio: `https://localhost:{PORT}/signin-google` , dove il `{PORT}` segnaposto è la porta dell'app.</span><span class="sxs-lookup"><span data-stu-id="06b26-113">Example redirect URI: `https://localhost:{PORT}/signin-google`, where the `{PORT}` placeholder is the app's port.</span></span>
* <span data-ttu-id="06b26-114">Selezionare il pulsante **Crea** .</span><span class="sxs-lookup"><span data-stu-id="06b26-114">Select the **CREATE** button.</span></span>
* <span data-ttu-id="06b26-115">Salvare l' **ID client** e il **segreto client** per l'uso nella configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="06b26-115">Save the **Client ID** and **Client Secret** for use in the app's configuration.</span></span>
* <span data-ttu-id="06b26-116">Quando si distribuisce il sito, è possibile:</span><span class="sxs-lookup"><span data-stu-id="06b26-116">When deploying the site, either:</span></span>
  * <span data-ttu-id="06b26-117">Aggiornare l'URI di reindirizzamento dell'app nella **console di Google** all'URI di reindirizzamento distribuito dell'app.</span><span class="sxs-lookup"><span data-stu-id="06b26-117">Update the app's redirect URI in the **Google Console** to the app's deployed redirect URI.</span></span>
  * <span data-ttu-id="06b26-118">Creare una nuova registrazione dell'API Google nella **console di Google** per l'app di produzione con l'URI di reindirizzamento di produzione.</span><span class="sxs-lookup"><span data-stu-id="06b26-118">Create a new Google API registration in the **Google Console** for the production app with its production redirect URI.</span></span>

## <a name="store-the-google-client-id-and-secret"></a><span data-ttu-id="06b26-119">Archiviare l'ID e il segreto del client Google</span><span class="sxs-lookup"><span data-stu-id="06b26-119">Store the Google client ID and secret</span></span>

<span data-ttu-id="06b26-120">Archiviare le impostazioni riservate, ad esempio i valori di ID client e segreto di Google con [gestione Secret](xref:security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="06b26-120">Store sensitive settings such as the Google client ID and secret values with [Secret Manager](xref:security/app-secrets).</span></span> <span data-ttu-id="06b26-121">Per questo esempio, attenersi alla procedura seguente:</span><span class="sxs-lookup"><span data-stu-id="06b26-121">For this sample, use the following steps:</span></span>

1. <span data-ttu-id="06b26-122">Inizializzare il progetto per l'archiviazione segreta in base alle istruzioni riportate in [abilitare l'archiviazione segreta](xref:security/app-secrets#enable-secret-storage).</span><span class="sxs-lookup"><span data-stu-id="06b26-122">Initialize the project for secret storage per the instructions at [Enable secret storage](xref:security/app-secrets#enable-secret-storage).</span></span>
1. <span data-ttu-id="06b26-123">Archiviare le impostazioni sensibili nell'archivio dei segreti locali con le chiavi segrete `Authentication:Google:ClientId` e `Authentication:Google:ClientSecret` :</span><span class="sxs-lookup"><span data-stu-id="06b26-123">Store the sensitive settings in the local secret store with the secret keys `Authentication:Google:ClientId` and `Authentication:Google:ClientSecret`:</span></span>

    ```dotnetcli
    dotnet user-secrets set "Authentication:Google:ClientId" "<client-id>"
    dotnet user-secrets set "Authentication:Google:ClientSecret" "<client-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

<span data-ttu-id="06b26-124">È possibile gestire le credenziali e l'utilizzo dell'API nella [console API](https://console.developers.google.com/apis/dashboard).</span><span class="sxs-lookup"><span data-stu-id="06b26-124">You can manage your API credentials and usage in the [API Console](https://console.developers.google.com/apis/dashboard).</span></span>

## <a name="configure-google-authentication"></a><span data-ttu-id="06b26-125">Configurare l'autenticazione di Google</span><span class="sxs-lookup"><span data-stu-id="06b26-125">Configure Google authentication</span></span>

<span data-ttu-id="06b26-126">Aggiungere il servizio Google a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="06b26-126">Add the Google service to `Startup.ConfigureServices`:</span></span>

[!code-csharp[](~/security/authentication/social/social-code/3.x/StartupGoogle3x.cs?highlight=11-19)]

[!INCLUDE [default settings configuration](includes/default-settings2-2.md)]

## <a name="sign-in-with-google"></a><span data-ttu-id="06b26-127">Accedere con Google</span><span class="sxs-lookup"><span data-stu-id="06b26-127">Sign in with Google</span></span>

* <span data-ttu-id="06b26-128">Eseguire l'app e fare clic su **Accedi**.</span><span class="sxs-lookup"><span data-stu-id="06b26-128">Run the app and click **Log in**.</span></span> <span data-ttu-id="06b26-129">Viene visualizzata un'opzione per accedere con Google.</span><span class="sxs-lookup"><span data-stu-id="06b26-129">An option to sign in with Google appears.</span></span>
* <span data-ttu-id="06b26-130">Fare clic sul pulsante **Google** , che reindirizza a Google per l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="06b26-130">Click the **Google** button, which redirects to Google for authentication.</span></span>
* <span data-ttu-id="06b26-131">Dopo aver immesso le credenziali Google, viene reindirizzato di nuovo al sito Web.</span><span class="sxs-lookup"><span data-stu-id="06b26-131">After entering your Google credentials, you are redirected back to the web site.</span></span>

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

<span data-ttu-id="06b26-132"><xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>Per altre informazioni sulle opzioni di configurazione supportate da Google Authentication, vedere le informazioni di riferimento sulle API.</span><span class="sxs-lookup"><span data-stu-id="06b26-132">See the <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> API reference for more information on configuration options supported by Google authentication.</span></span> <span data-ttu-id="06b26-133">Questa operazione può essere utilizzata per richiedere informazioni diverse sull'utente.</span><span class="sxs-lookup"><span data-stu-id="06b26-133">This can be used to request different information about the user.</span></span>

## <a name="change-the-default-callback-uri"></a><span data-ttu-id="06b26-134">Modificare l'URI di callback predefinito</span><span class="sxs-lookup"><span data-stu-id="06b26-134">Change the default callback URI</span></span>

<span data-ttu-id="06b26-135">Il segmento URI `/signin-google` viene impostato come callback predefinito del provider di autenticazione Google.</span><span class="sxs-lookup"><span data-stu-id="06b26-135">The URI segment `/signin-google` is set as the default callback of the Google authentication provider.</span></span> <span data-ttu-id="06b26-136">È possibile modificare l'URI di callback predefinito durante la configurazione del middleware di autenticazione di Google tramite la <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CallbackPath?displayProperty=nameWithType> proprietà ereditato della <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> classe.</span><span class="sxs-lookup"><span data-stu-id="06b26-136">You can change the default callback URI while configuring the Google authentication middleware via the inherited <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CallbackPath?displayProperty=nameWithType>) property of the <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> class.</span></span>

## <a name="troubleshooting"></a><span data-ttu-id="06b26-137">Risoluzione dei problemi</span><span class="sxs-lookup"><span data-stu-id="06b26-137">Troubleshooting</span></span>

* <span data-ttu-id="06b26-138">Se l'accesso non funziona e non si ricevono errori, passare alla modalità di sviluppo per semplificare il debug del problema.</span><span class="sxs-lookup"><span data-stu-id="06b26-138">If the sign-in doesn't work and you aren't getting any errors, switch to development mode to make the issue easier to debug.</span></span>
* <span data-ttu-id="06b26-139">Se Identity non è configurato chiamando `services.AddIdentity` in `ConfigureServices` , il tentativo di autenticare i risultati in *ArgumentException: è necessario specificare l'opzione ' SignInScheme '*.</span><span class="sxs-lookup"><span data-stu-id="06b26-139">If Identity isn't configured by calling `services.AddIdentity` in `ConfigureServices`, attempting to authenticate results in *ArgumentException: The 'SignInScheme' option must be provided*.</span></span> <span data-ttu-id="06b26-140">Il modello di progetto usato in questa esercitazione garantisce che questa operazione venga eseguita.</span><span class="sxs-lookup"><span data-stu-id="06b26-140">The project template used in this tutorial ensures that this is done.</span></span>
* <span data-ttu-id="06b26-141">Se il database del sito non è stato creato applicando la migrazione iniziale, si ottiene *un'operazione di database non riuscita durante l'elaborazione dell'* errore di richiesta.</span><span class="sxs-lookup"><span data-stu-id="06b26-141">If the site database has not been created by applying the initial migration, you get *A database operation failed while processing the request* error.</span></span> <span data-ttu-id="06b26-142">Selezionare **applica migrazioni** per creare il database e aggiornare la pagina per continuare a superare l'errore.</span><span class="sxs-lookup"><span data-stu-id="06b26-142">Select **Apply Migrations** to create the database, and refresh the page to continue past the error.</span></span>

## <a name="next-steps"></a><span data-ttu-id="06b26-143">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="06b26-143">Next steps</span></span>

* <span data-ttu-id="06b26-144">Questo articolo ha illustrato come è possibile eseguire l'autenticazione con Google.</span><span class="sxs-lookup"><span data-stu-id="06b26-144">This article showed how you can authenticate with Google.</span></span> <span data-ttu-id="06b26-145">È possibile seguire un approccio simile per l'autenticazione con altri provider elencati nella [pagina precedente](xref:security/authentication/social/index).</span><span class="sxs-lookup"><span data-stu-id="06b26-145">You can follow a similar approach to authenticate with other providers listed on the [previous page](xref:security/authentication/social/index).</span></span>
* <span data-ttu-id="06b26-146">Dopo aver pubblicato l'app in Azure, reimpostare la `ClientSecret` nella console dell'API Google.</span><span class="sxs-lookup"><span data-stu-id="06b26-146">Once you publish the app to Azure, reset the `ClientSecret` in the Google API Console.</span></span>
* <span data-ttu-id="06b26-147">Impostare `Authentication:Google:ClientId` e `Authentication:Google:ClientSecret` come impostazioni dell'applicazione nella portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="06b26-147">Set the `Authentication:Google:ClientId` and `Authentication:Google:ClientSecret` as application settings in the Azure portal.</span></span> <span data-ttu-id="06b26-148">Il sistema di configurazione è configurato per la lettura delle chiavi dalle variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="06b26-148">The configuration system is set up to read keys from environment variables.</span></span>
