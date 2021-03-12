---
title: Account confirmation and password recovery in ASP.NET Core (Conferma dell'account e recupero della password in ASP.NET Core)
author: rick-anderson
description: Informazioni su come creare un'app ASP.NET Core con la conferma della posta elettronica e la reimpostazione della password.
ms.author: riande
ms.date: 03/11/2019
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
uid: security/authentication/accconfirm
ms.openlocfilehash: f71ae5e619b875c03401fa78320582c406875401
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586111"
---
# <a name="account-confirmation-and-password-recovery-in-aspnet-core"></a><span data-ttu-id="5790d-103">Account confirmation and password recovery in ASP.NET Core (Conferma dell'account e recupero della password in ASP.NET Core)</span><span class="sxs-lookup"><span data-stu-id="5790d-103">Account confirmation and password recovery in ASP.NET Core</span></span>

<span data-ttu-id="5790d-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Ponant](https://github.com/Ponant)e [Joe Audette](https://twitter.com/joeaudette)</span><span class="sxs-lookup"><span data-stu-id="5790d-104">By [Rick Anderson](https://twitter.com/RickAndMSFT), [Ponant](https://github.com/Ponant), and [Joe Audette](https://twitter.com/joeaudette)</span></span>

<span data-ttu-id="5790d-105">Questa esercitazione illustra come compilare un'app ASP.NET Core con la conferma della posta elettronica e la reimpostazione della password.</span><span class="sxs-lookup"><span data-stu-id="5790d-105">This tutorial shows how to build an ASP.NET Core app with email confirmation and password reset.</span></span> <span data-ttu-id="5790d-106">Questa esercitazione **non** è un argomento iniziale.</span><span class="sxs-lookup"><span data-stu-id="5790d-106">This tutorial is **not** a beginning topic.</span></span> <span data-ttu-id="5790d-107">È necessario avere familiarità con:</span><span class="sxs-lookup"><span data-stu-id="5790d-107">You should be familiar with:</span></span>

* [<span data-ttu-id="5790d-108">ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="5790d-108">ASP.NET Core</span></span>](xref:tutorials/razor-pages/razor-pages-start)
* [<span data-ttu-id="5790d-109">autenticazione</span><span class="sxs-lookup"><span data-stu-id="5790d-109">Authentication</span></span>](xref:security/authentication/identity)
* [<span data-ttu-id="5790d-110">Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="5790d-110">Entity Framework Core</span></span>](xref:data/ef-mvc/intro)

<!-- see C:/Dropbox/wrk/Code/SendGridConsole/Program.cs -->

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a><span data-ttu-id="5790d-111">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="5790d-111">Prerequisites</span></span>

[<span data-ttu-id="5790d-112">.NET Core 3,0 SDK o versione successiva</span><span class="sxs-lookup"><span data-stu-id="5790d-112">.NET Core 3.0 SDK or later</span></span>](https://dotnet.microsoft.com/download/dotnet-core/3.0)

## <a name="create-and-test-a-web-app-with-authentication"></a><span data-ttu-id="5790d-113">Creare e testare un'app Web con autenticazione</span><span class="sxs-lookup"><span data-stu-id="5790d-113">Create and test a web app with authentication</span></span>

<span data-ttu-id="5790d-114">Eseguire i comandi seguenti per creare un'app Web con l'autenticazione di.</span><span class="sxs-lookup"><span data-stu-id="5790d-114">Run the following commands to create a web app with authentication.</span></span>

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet run
```

<span data-ttu-id="5790d-115">Eseguire l'app, selezionare il collegamento **Register** e registrare un utente.</span><span class="sxs-lookup"><span data-stu-id="5790d-115">Run the app, select the **Register** link, and register a user.</span></span> <span data-ttu-id="5790d-116">Una volta effettuata la registrazione, si verrà reindirizzati alla `/Identity/Account/RegisterConfirmation` pagina a contenente un collegamento per simulare la conferma della posta elettronica:</span><span class="sxs-lookup"><span data-stu-id="5790d-116">Once registered, you are redirected to the to `/Identity/Account/RegisterConfirmation` page which contains a link to simulate email confirmation:</span></span>

* <span data-ttu-id="5790d-117">Selezionare il `Click here to confirm your account` collegamento.</span><span class="sxs-lookup"><span data-stu-id="5790d-117">Select the `Click here to confirm your account` link.</span></span>
* <span data-ttu-id="5790d-118">Selezionare il collegamento di **accesso** e accedere con le stesse credenziali.</span><span class="sxs-lookup"><span data-stu-id="5790d-118">Select the **Login** link and sign-in with the same credentials.</span></span>
* <span data-ttu-id="5790d-119">Selezionare il `Hello YourEmail@provider.com!` collegamento per reindirizzare l'utente alla `/Identity/Account/Manage/PersonalData` pagina.</span><span class="sxs-lookup"><span data-stu-id="5790d-119">Select the `Hello YourEmail@provider.com!` link, which redirects you to the `/Identity/Account/Manage/PersonalData` page.</span></span>
* <span data-ttu-id="5790d-120">Selezionare la scheda **Personal Data (dati personali** ) a sinistra e quindi fare clic su **Delete (Elimina**).</span><span class="sxs-lookup"><span data-stu-id="5790d-120">Select the **Personal data** tab on the left, and then select **Delete**.</span></span>

### <a name="configure-an-email-provider"></a><span data-ttu-id="5790d-121">Configurare un provider di posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-121">Configure an email provider</span></span>

<span data-ttu-id="5790d-122">In questa esercitazione viene usato [SendGrid](https://sendgrid.com) per inviare messaggi di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-122">In this tutorial, [SendGrid](https://sendgrid.com) is used to send email.</span></span> <span data-ttu-id="5790d-123">Per inviare messaggi di posta elettronica sono necessari un account e una chiave di SendGrid.</span><span class="sxs-lookup"><span data-stu-id="5790d-123">You need a SendGrid account and key to send email.</span></span> <span data-ttu-id="5790d-124">È possibile usare altri provider di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-124">You can use other email providers.</span></span> <span data-ttu-id="5790d-125">È consigliabile usare SendGrid o un altro servizio di posta elettronica per inviare messaggi di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-125">We recommend you use SendGrid or another email service to send email.</span></span> <span data-ttu-id="5790d-126">SMTP è difficile da proteggere e configurare correttamente.</span><span class="sxs-lookup"><span data-stu-id="5790d-126">SMTP is difficult to secure and set up correctly.</span></span>

<span data-ttu-id="5790d-127">L'account SendGrid può richiedere l' [aggiunta di un mittente](https://sendgrid.com/docs/ui/sending-email/senders/).</span><span class="sxs-lookup"><span data-stu-id="5790d-127">The SendGrid account may require [adding a Sender](https://sendgrid.com/docs/ui/sending-email/senders/).</span></span>

<span data-ttu-id="5790d-128">Creare una classe per recuperare la chiave di posta elettronica sicura.</span><span class="sxs-lookup"><span data-stu-id="5790d-128">Create a class to fetch the secure email key.</span></span> <span data-ttu-id="5790d-129">Per questo esempio, creare *Servizi/AuthMessageSenderOptions. cs*:</span><span class="sxs-lookup"><span data-stu-id="5790d-129">For this sample, create *Services/AuthMessageSenderOptions.cs*:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a><span data-ttu-id="5790d-130">Configurare i segreti utente di SendGrid</span><span class="sxs-lookup"><span data-stu-id="5790d-130">Configure SendGrid user secrets</span></span>

<span data-ttu-id="5790d-131">Impostare `SendGridUser` e `SendGridKey` con lo [strumento di gestione dei segreti](xref:security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="5790d-131">Set the `SendGridUser` and `SendGridKey` with the [secret-manager tool](xref:security/app-secrets).</span></span> <span data-ttu-id="5790d-132">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="5790d-132">For example:</span></span>

```dotnetcli
dotnet user-secrets set SendGridUser RickAndMSFT
dotnet user-secrets set SendGridKey <key>

Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

<span data-ttu-id="5790d-133">In Windows, gestione Secret archivia le coppie chiave/valore in una *secrets.js* nel file nella `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` Directory.</span><span class="sxs-lookup"><span data-stu-id="5790d-133">On Windows, Secret Manager stores keys/value pairs in a *secrets.json* file in the `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` directory.</span></span>

<span data-ttu-id="5790d-134">Il contenuto del *secrets.jsnel* file non è crittografato.</span><span class="sxs-lookup"><span data-stu-id="5790d-134">The contents of the *secrets.json* file aren't encrypted.</span></span> <span data-ttu-id="5790d-135">Il markup seguente mostra il *secrets.jssu* file.</span><span class="sxs-lookup"><span data-stu-id="5790d-135">The following markup shows the *secrets.json* file.</span></span> <span data-ttu-id="5790d-136">Il `SendGridKey` valore è stato rimosso.</span><span class="sxs-lookup"><span data-stu-id="5790d-136">The `SendGridKey` value has been removed.</span></span>

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

<span data-ttu-id="5790d-137">Per ulteriori informazioni, vedere il [modello](xref:fundamentals/configuration/options) e la [configurazione](xref:fundamentals/configuration/index)delle opzioni.</span><span class="sxs-lookup"><span data-stu-id="5790d-137">For more information, see the [Options pattern](xref:fundamentals/configuration/options) and [configuration](xref:fundamentals/configuration/index).</span></span>

### <a name="install-sendgrid"></a><span data-ttu-id="5790d-138">Installare SendGrid</span><span class="sxs-lookup"><span data-stu-id="5790d-138">Install SendGrid</span></span>

<span data-ttu-id="5790d-139">Questa esercitazione illustra come aggiungere notifiche tramite posta elettronica tramite [SendGrid](https://sendgrid.com/), ma è possibile inviare messaggi di posta elettronica usando SMTP e altri meccanismi.</span><span class="sxs-lookup"><span data-stu-id="5790d-139">This tutorial shows how to add email notifications through [SendGrid](https://sendgrid.com/), but you can send email using SMTP and other mechanisms.</span></span>

<span data-ttu-id="5790d-140">Installare il `SendGrid` pacchetto NuGet:</span><span class="sxs-lookup"><span data-stu-id="5790d-140">Install the `SendGrid` NuGet package:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5790d-141">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5790d-141">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5790d-142">Dalla console di gestione pacchetti immettere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5790d-142">From the Package Manager Console, enter the following command:</span></span>

```powershell
Install-Package SendGrid
```

# <a name="net-core-cli"></a>[<span data-ttu-id="5790d-143">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="5790d-143">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="5790d-144">Dalla console di immettere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5790d-144">From the console, enter the following command:</span></span>

```dotnetcli
dotnet add package SendGrid
```

---

<span data-ttu-id="5790d-145">Per una registrazione gratuita per un account SendGrid gratuito, vedere [Introduzione a SendGrid](https://sendgrid.com/free/) .</span><span class="sxs-lookup"><span data-stu-id="5790d-145">See [Get Started with SendGrid for Free](https://sendgrid.com/free/) to register for a free SendGrid account.</span></span>

### <a name="implement-iemailsender"></a><span data-ttu-id="5790d-146">Implementare IEmailSender</span><span class="sxs-lookup"><span data-stu-id="5790d-146">Implement IEmailSender</span></span>

<span data-ttu-id="5790d-147">Per implementare `IEmailSender` , creare *Services/EmailSender. cs* con codice simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="5790d-147">To Implement `IEmailSender`, create *Services/EmailSender.cs* with code similar to the following:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a><span data-ttu-id="5790d-148">Configurare l'avvio per il supporto della posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-148">Configure startup to support email</span></span>

<span data-ttu-id="5790d-149">Aggiungere il codice seguente al `ConfigureServices` metodo nel file *Startup.cs* :</span><span class="sxs-lookup"><span data-stu-id="5790d-149">Add the following code to the `ConfigureServices` method in the *Startup.cs* file:</span></span>

* <span data-ttu-id="5790d-150">Aggiungere `EmailSender` come servizio temporaneo.</span><span class="sxs-lookup"><span data-stu-id="5790d-150">Add `EmailSender` as a transient service.</span></span>
* <span data-ttu-id="5790d-151">Registrare l' `AuthMessageSenderOptions` istanza di configurazione.</span><span class="sxs-lookup"><span data-stu-id="5790d-151">Register the `AuthMessageSenderOptions` configuration instance.</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover30/Startup.cs?name=snippet1&highlight=11-15)]

## <a name="scaffold-registerconfirmation"></a><span data-ttu-id="5790d-152">RegisterConfirmation ponteggi</span><span class="sxs-lookup"><span data-stu-id="5790d-152">Scaffold RegisterConfirmation</span></span>

<span data-ttu-id="5790d-153">Seguire le istruzioni per [impalcature Identity ](xref:security/authentication/scaffold-identity) e impalcature `RegisterConfirmation` .</span><span class="sxs-lookup"><span data-stu-id="5790d-153">Follow the instructions for [Scaffold Identity](xref:security/authentication/scaffold-identity) and scaffold `RegisterConfirmation`.</span></span>

<!-- .NET 5 fixes this, see
https://github.com/dotnet/aspnetcore/blob/main/src/Identity/UI/src/Areas/Identity/Pages/V4/Account/RegisterConfirmation.cshtml.cs#L74-L77
-->

[!INCLUDE[](~/includes/disableVer.md)]

## <a name="register-confirm-email-and-reset-password"></a><span data-ttu-id="5790d-154">Registrare, confermare la posta elettronica e reimpostare la password</span><span class="sxs-lookup"><span data-stu-id="5790d-154">Register, confirm email, and reset password</span></span>

<span data-ttu-id="5790d-155">Eseguire l'app Web e testare la conferma dell'account e il flusso di recupero della password.</span><span class="sxs-lookup"><span data-stu-id="5790d-155">Run the web app, and test the account confirmation and password recovery flow.</span></span>

* <span data-ttu-id="5790d-156">Eseguire l'app e registrare un nuovo utente</span><span class="sxs-lookup"><span data-stu-id="5790d-156">Run the app and register a new user</span></span>
* <span data-ttu-id="5790d-157">Controllare la posta elettronica per il collegamento di conferma dell'account.</span><span class="sxs-lookup"><span data-stu-id="5790d-157">Check your email for the account confirmation link.</span></span> <span data-ttu-id="5790d-158">Se non si riceve il messaggio di posta elettronica, vedere [debug](#debug) della posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-158">See [Debug email](#debug) if you don't get the email.</span></span>
* <span data-ttu-id="5790d-159">Fare clic sul collegamento per confermare la posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-159">Click the link to confirm your email.</span></span>
* <span data-ttu-id="5790d-160">Accedere con l'indirizzo di posta elettronica e la password.</span><span class="sxs-lookup"><span data-stu-id="5790d-160">Sign in with your email and password.</span></span>
* <span data-ttu-id="5790d-161">Uscire,</span><span class="sxs-lookup"><span data-stu-id="5790d-161">Sign out.</span></span>

### <a name="test-password-reset"></a><span data-ttu-id="5790d-162">Testare la reimpostazione della password</span><span class="sxs-lookup"><span data-stu-id="5790d-162">Test password reset</span></span>

* <span data-ttu-id="5790d-163">Se è stato eseguito l'accesso, selezionare **Disconnetti**.</span><span class="sxs-lookup"><span data-stu-id="5790d-163">If you're signed in, select **Logout**.</span></span>
* <span data-ttu-id="5790d-164">Selezionare il collegamento **Accedi** e selezionare il collegamento **password dimenticata?** .</span><span class="sxs-lookup"><span data-stu-id="5790d-164">Select the **Log in** link and select the **Forgot your password?** link.</span></span>
* <span data-ttu-id="5790d-165">Immettere il messaggio di posta elettronica usato per registrare l'account.</span><span class="sxs-lookup"><span data-stu-id="5790d-165">Enter the email you used to register the account.</span></span>
* <span data-ttu-id="5790d-166">Viene inviato un messaggio di posta elettronica con un collegamento per reimpostare la password.</span><span class="sxs-lookup"><span data-stu-id="5790d-166">An email with a link to reset your password is sent.</span></span> <span data-ttu-id="5790d-167">Controllare la posta elettronica e fare clic sul collegamento per reimpostare la password.</span><span class="sxs-lookup"><span data-stu-id="5790d-167">Check your email and click the link to reset your password.</span></span> <span data-ttu-id="5790d-168">Dopo che la password è stata reimpostata, è possibile accedere con l'indirizzo di posta elettronica e la nuova password.</span><span class="sxs-lookup"><span data-stu-id="5790d-168">After your password has been successfully reset, you can sign in with your email and new password.</span></span>

<a name="resend"></a>

## <a name="resend-email-confirmation"></a><span data-ttu-id="5790d-169">Invia di nuovo la conferma tramite posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-169">Resend email confirmation</span></span>

<span data-ttu-id="5790d-170">In ASP.NET Core 5,0 e versioni successive selezionare il collegamento Invia di nuovo la **conferma tramite posta elettronica** nella pagina di **accesso** .</span><span class="sxs-lookup"><span data-stu-id="5790d-170">In ASP.NET Core 5.0 and later, select the **Resend email confirmation** link on the **Login** page.</span></span>

### <a name="change-email-and-activity-timeout"></a><span data-ttu-id="5790d-171">Modificare il timeout dell'attività e del messaggio di posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-171">Change email and activity timeout</span></span>

<span data-ttu-id="5790d-172">Il timeout di inattività predefinito è di 14 giorni.</span><span class="sxs-lookup"><span data-stu-id="5790d-172">The default inactivity timeout is 14 days.</span></span> <span data-ttu-id="5790d-173">Il codice seguente imposta il timeout di inattività su 5 giorni:</span><span class="sxs-lookup"><span data-stu-id="5790d-173">The following code sets the inactivity timeout to 5 days:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a><span data-ttu-id="5790d-174">Modifica tutte le durate dei token di protezione dati</span><span class="sxs-lookup"><span data-stu-id="5790d-174">Change all data protection token lifespans</span></span>

<span data-ttu-id="5790d-175">Il codice seguente modifica il periodo di timeout di tutti i token di protezione dati in 3 ore:</span><span class="sxs-lookup"><span data-stu-id="5790d-175">The following code changes all data protection tokens timeout period to 3 hours:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAllTokens.cs?name=snippet1&highlight=11-12)]

<span data-ttu-id="5790d-176">I Identity token utente predefiniti (vedere [AspNetCore/src/ Identity /Extensions.core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) hanno un timeout di un [giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span><span class="sxs-lookup"><span data-stu-id="5790d-176">The built in Identity user tokens (see [AspNetCore/src/Identity/Extensions.Core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) )have a [one day timeout](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span></span>

### <a name="change-the-email-token-lifespan"></a><span data-ttu-id="5790d-177">Modificare la durata del token di posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-177">Change the email token lifespan</span></span>

<span data-ttu-id="5790d-178">La durata del token predefinita [dei Identity token utente](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) è [un giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span><span class="sxs-lookup"><span data-stu-id="5790d-178">The default token lifespan of [the Identity user tokens](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) is [one day](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span></span> <span data-ttu-id="5790d-179">Questa sezione illustra come modificare la durata del token di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-179">This section shows how to change the email token lifespan.</span></span>

<span data-ttu-id="5790d-180">Aggiungere un [DataProtectorTokenProvider \<TUser> ](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) personalizzato e <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions> :</span><span class="sxs-lookup"><span data-stu-id="5790d-180">Add a custom [DataProtectorTokenProvider\<TUser>](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) and <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions>:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover30/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

<span data-ttu-id="5790d-181">Aggiungere il provider personalizzato al contenitore del servizio:</span><span class="sxs-lookup"><span data-stu-id="5790d-181">Add the custom provider to the service container:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupEmail.cs?name=snippet1&highlight=10-16)]

<a name="debug"></a>

### <a name="debug-email"></a><span data-ttu-id="5790d-182">Posta elettronica di debug</span><span class="sxs-lookup"><span data-stu-id="5790d-182">Debug email</span></span>

<span data-ttu-id="5790d-183">Se non è possibile ottenere la posta elettronica:</span><span class="sxs-lookup"><span data-stu-id="5790d-183">If you can't get email working:</span></span>

* <span data-ttu-id="5790d-184">Impostare un punto di interruzione in `EmailSender.Execute` per verificare che `SendGridClient.SendEmailAsync` venga chiamato.</span><span class="sxs-lookup"><span data-stu-id="5790d-184">Set a breakpoint in `EmailSender.Execute` to verify `SendGridClient.SendEmailAsync` is called.</span></span>
* <span data-ttu-id="5790d-185">Creare un' [app console per inviare messaggi di posta elettronica](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) usando codice simile a `EmailSender.Execute` .</span><span class="sxs-lookup"><span data-stu-id="5790d-185">Create a [console app to send email](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) using similar code to `EmailSender.Execute`.</span></span>
* <span data-ttu-id="5790d-186">Esaminare la pagina [attività posta elettronica](https://sendgrid.com/docs/User_Guide/email_activity.html) .</span><span class="sxs-lookup"><span data-stu-id="5790d-186">Review the [Email Activity](https://sendgrid.com/docs/User_Guide/email_activity.html) page.</span></span>
* <span data-ttu-id="5790d-187">Controllare la cartella della posta indesiderata.</span><span class="sxs-lookup"><span data-stu-id="5790d-187">Check your spam folder.</span></span>
* <span data-ttu-id="5790d-188">Prova un altro alias di posta elettronica in un provider di posta elettronica diverso (Microsoft, Yahoo, Gmail e così via)</span><span class="sxs-lookup"><span data-stu-id="5790d-188">Try another email alias on a different email provider (Microsoft, Yahoo, Gmail, etc.)</span></span>
* <span data-ttu-id="5790d-189">Provare a inviare a diversi account di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-189">Try sending to different email accounts.</span></span>

<span data-ttu-id="5790d-190">**Una procedura di sicurezza consigliata** consiste nel **non** usare i segreti di produzione per test e sviluppo.</span><span class="sxs-lookup"><span data-stu-id="5790d-190">**A security best practice** is to **not** use production secrets in test and development.</span></span> <span data-ttu-id="5790d-191">Se si pubblica l'app in Azure, impostare i segreti SendGrid come impostazioni dell'applicazione nel portale dell'app Web di Azure.</span><span class="sxs-lookup"><span data-stu-id="5790d-191">If you publish the app to Azure, set the SendGrid secrets as application settings in the Azure Web App portal.</span></span> <span data-ttu-id="5790d-192">Il sistema di configurazione è configurato per la lettura delle chiavi dalle variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="5790d-192">The configuration system is set up to read keys from environment variables.</span></span>

## <a name="combine-social-and-local-login-accounts"></a><span data-ttu-id="5790d-193">Combinare account di accesso di social networking e locali</span><span class="sxs-lookup"><span data-stu-id="5790d-193">Combine social and local login accounts</span></span>

<span data-ttu-id="5790d-194">Per completare questa sezione, è necessario innanzitutto abilitare un provider di autenticazione esterno.</span><span class="sxs-lookup"><span data-stu-id="5790d-194">To complete this section, you must first enable an external authentication provider.</span></span> <span data-ttu-id="5790d-195">Vedere [l'autenticazione di Facebook, Google e del provider esterno](xref:security/authentication/social/index).</span><span class="sxs-lookup"><span data-stu-id="5790d-195">See [Facebook, Google, and external provider authentication](xref:security/authentication/social/index).</span></span>

<span data-ttu-id="5790d-196">È possibile combinare account locali e di social networking facendo clic sul collegamento di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-196">You can combine local and social accounts by clicking on your email link.</span></span> <span data-ttu-id="5790d-197">Nella sequenza seguente " RickAndMSFT@gmail.com " viene creato per la prima volta come account di accesso locale; tuttavia, è possibile creare prima l'account come account di accesso di social networking, quindi aggiungere un account di accesso locale.</span><span class="sxs-lookup"><span data-stu-id="5790d-197">In the following sequence, "RickAndMSFT@gmail.com" is first created as a local login; however, you can create the account as a social login first, then add a local login.</span></span>

![Applicazione Web: RickAndMSFT@gmail.com utente autenticato](accconfirm/_static/rick.png)

<span data-ttu-id="5790d-199">Fare clic sul collegamento **Gestisci** .</span><span class="sxs-lookup"><span data-stu-id="5790d-199">Click on the **Manage** link.</span></span> <span data-ttu-id="5790d-200">Si notino gli account di accesso di social network (0) esterni associati a questo account.</span><span class="sxs-lookup"><span data-stu-id="5790d-200">Note the 0 external (social logins) associated with this account.</span></span>

![Gestisci visualizzazione](accconfirm/_static/manage.png)

<span data-ttu-id="5790d-202">Fare clic sul collegamento a un altro servizio di accesso e accettare le richieste dell'app.</span><span class="sxs-lookup"><span data-stu-id="5790d-202">Click the link to another login service and accept the app requests.</span></span> <span data-ttu-id="5790d-203">Nell'immagine seguente, Facebook è il provider di autenticazione esterno:</span><span class="sxs-lookup"><span data-stu-id="5790d-203">In the following image, Facebook is the external authentication provider:</span></span>

![Gestire gli account di accesso esterni visualizzare l'elenco di Facebook](accconfirm/_static/fb.png)

<span data-ttu-id="5790d-205">I due account sono stati combinati.</span><span class="sxs-lookup"><span data-stu-id="5790d-205">The two accounts have been combined.</span></span> <span data-ttu-id="5790d-206">È possibile accedere con uno dei due account.</span><span class="sxs-lookup"><span data-stu-id="5790d-206">You are able to sign in with either account.</span></span> <span data-ttu-id="5790d-207">È possibile che gli utenti aggiungano gli account locali nel caso in cui il servizio di autenticazione per l'accesso di social networking sia inattivo o più probabilmente abbiano perso l'accesso al proprio account di social networking.</span><span class="sxs-lookup"><span data-stu-id="5790d-207">You might want your users to add local accounts in case their social login authentication service is down, or more likely they've lost access to their social account.</span></span>

## <a name="enable-account-confirmation-after-a-site-has-users"></a><span data-ttu-id="5790d-208">Abilitare la conferma dell'account dopo che un sito ha utenti</span><span class="sxs-lookup"><span data-stu-id="5790d-208">Enable account confirmation after a site has users</span></span>

<span data-ttu-id="5790d-209">L'abilitazione della conferma dell'account in un sito con utenti blocca tutti gli utenti esistenti.</span><span class="sxs-lookup"><span data-stu-id="5790d-209">Enabling account confirmation on a site with users locks out all the existing users.</span></span> <span data-ttu-id="5790d-210">Gli utenti esistenti sono bloccati perché i relativi account non sono confermati.</span><span class="sxs-lookup"><span data-stu-id="5790d-210">Existing users are locked out because their accounts aren't confirmed.</span></span> <span data-ttu-id="5790d-211">Per aggirare il blocco utente esistente, utilizzare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="5790d-211">To work around existing user lockout, use one of the following approaches:</span></span>

* <span data-ttu-id="5790d-212">Aggiornare il database per contrassegnare tutti gli utenti esistenti come confermati.</span><span class="sxs-lookup"><span data-stu-id="5790d-212">Update the database to mark all existing users as being confirmed.</span></span>
* <span data-ttu-id="5790d-213">Verificare gli utenti esistenti.</span><span class="sxs-lookup"><span data-stu-id="5790d-213">Confirm existing users.</span></span> <span data-ttu-id="5790d-214">Ad esempio, invio in batch di messaggi di posta elettronica con collegamenti di conferma.</span><span class="sxs-lookup"><span data-stu-id="5790d-214">For example, batch-send emails with confirmation links.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="prerequisites"></a><span data-ttu-id="5790d-215">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="5790d-215">Prerequisites</span></span>

[<span data-ttu-id="5790d-216">.NET Core 2,2 SDK o versione successiva</span><span class="sxs-lookup"><span data-stu-id="5790d-216">.NET Core 2.2 SDK or later</span></span>](https://dotnet.microsoft.com/download/dotnet-core)

## <a name="create-a-web--app-and-scaffold-identity"></a><span data-ttu-id="5790d-217">Creare un'app Web e un patibolo Identity</span><span class="sxs-lookup"><span data-stu-id="5790d-217">Create a web  app and scaffold Identity</span></span>

<span data-ttu-id="5790d-218">Eseguire i comandi seguenti per creare un'app Web con l'autenticazione di.</span><span class="sxs-lookup"><span data-stu-id="5790d-218">Run the following commands to create a web app with authentication.</span></span>

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator identity -dc WebPWrecover.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout;Account.ConfirmEmail"
dotnet ef database drop -f
dotnet ef database update
dotnet run
```

> [!NOTE]
> <span data-ttu-id="5790d-219">Se <xref:Microsoft.AspNetCore.Identity.PasswordOptions> sono configurati in `Startup.ConfigureServices` , la configurazione degli [ `[StringLength]` attributi](xref:System.ComponentModel.DataAnnotations.StringLengthAttribute) potrebbe essere necessaria per la `Password` proprietà nelle pagine con impalcature Identity .</span><span class="sxs-lookup"><span data-stu-id="5790d-219">If <xref:Microsoft.AspNetCore.Identity.PasswordOptions> are configured in `Startup.ConfigureServices`, [`[StringLength]` attribute](xref:System.ComponentModel.DataAnnotations.StringLengthAttribute) configuration might be required for the `Password` property in scaffolded Identity pages.</span></span> <span data-ttu-id="5790d-220">Una `InputModel` `Password` proprietà viene trovata nel `Areas/Identity/Pages/Account/Register.cshtml.cs` file dopo l'impalcatura Identity .</span><span class="sxs-lookup"><span data-stu-id="5790d-220">An `InputModel` `Password` property is found in the `Areas/Identity/Pages/Account/Register.cshtml.cs` file after scaffolding Identity.</span></span>

## <a name="test-new-user-registration"></a><span data-ttu-id="5790d-221">Testare la registrazione di un nuovo utente</span><span class="sxs-lookup"><span data-stu-id="5790d-221">Test new user registration</span></span>

<span data-ttu-id="5790d-222">Eseguire l'app, selezionare il collegamento **Register** e registrare un utente.</span><span class="sxs-lookup"><span data-stu-id="5790d-222">Run the app, select the **Register** link, and register a user.</span></span> <span data-ttu-id="5790d-223">A questo punto, l'unica convalida sul messaggio di posta elettronica è con l' [`[EmailAddress]`](/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="5790d-223">At this point, the only validation on the email is with the [`[EmailAddress]`](/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute) attribute.</span></span> <span data-ttu-id="5790d-224">Dopo aver inviato la registrazione, si è connessi all'app.</span><span class="sxs-lookup"><span data-stu-id="5790d-224">After submitting the registration, you are logged into the app.</span></span> <span data-ttu-id="5790d-225">Più avanti nell'esercitazione, il codice viene aggiornato in modo che i nuovi utenti non possano accedere fino a quando non vengono convalidati i messaggi di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-225">Later in the tutorial, the code is updated so new users can't sign in until their email is validated.</span></span>

[!INCLUDE[](~/includes/view-identity-db.md)]

<span data-ttu-id="5790d-226">Si noti che il `EmailConfirmed` campo della tabella è `False` .</span><span class="sxs-lookup"><span data-stu-id="5790d-226">Note the table's `EmailConfirmed` field is `False`.</span></span>

<span data-ttu-id="5790d-227">Potrebbe essere necessario usare nuovamente questo messaggio di posta elettronica nel passaggio successivo quando l'app invia un messaggio di posta elettronica di conferma.</span><span class="sxs-lookup"><span data-stu-id="5790d-227">You might want to use this email again in the next step when the app sends a confirmation email.</span></span> <span data-ttu-id="5790d-228">Fare clic con il pulsante destro del mouse sulla riga e scegliere **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="5790d-228">Right-click on the row and select **Delete**.</span></span> <span data-ttu-id="5790d-229">L'eliminazione dell'alias di posta elettronica rende più semplice nei passaggi seguenti.</span><span class="sxs-lookup"><span data-stu-id="5790d-229">Deleting the email alias makes it easier in the following steps.</span></span>

<a name="prevent-login-at-registration"></a>

## <a name="require-email-confirmation"></a><span data-ttu-id="5790d-230">Richiedi conferma posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-230">Require email confirmation</span></span>

<span data-ttu-id="5790d-231">È consigliabile confermare il messaggio di posta elettronica di una nuova registrazione utente.</span><span class="sxs-lookup"><span data-stu-id="5790d-231">It's a best practice to confirm the email of a new user registration.</span></span> <span data-ttu-id="5790d-232">La conferma tramite posta elettronica consente di verificare che non stiano eseguendo la rappresentazione di un altro utente, ovvero che non sono stati registrati con il messaggio di posta elettronica di un altro utente.</span><span class="sxs-lookup"><span data-stu-id="5790d-232">Email confirmation helps to verify they're not impersonating someone else (that is, they haven't registered with someone else's email).</span></span> <span data-ttu-id="5790d-233">Si supponga di avere un forum di discussione e di voler impedire che " yli@example.com " si registri come " nolivetto@contoso.com ".</span><span class="sxs-lookup"><span data-stu-id="5790d-233">Suppose you had a discussion forum, and you wanted to prevent "yli@example.com" from registering as "nolivetto@contoso.com".</span></span> <span data-ttu-id="5790d-234">Senza la conferma della posta elettronica, " nolivetto@contoso.com " potrebbe ricevere un messaggio di posta elettronica indesiderato dall'app.</span><span class="sxs-lookup"><span data-stu-id="5790d-234">Without email confirmation, "nolivetto@contoso.com" could receive unwanted email from your app.</span></span> <span data-ttu-id="5790d-235">Si supponga che l'utente sia stato accidentalmente registrato come " ylo@example.com " e non abbia notato l'errore di ortografia di "Yli".</span><span class="sxs-lookup"><span data-stu-id="5790d-235">Suppose the user accidentally registered as "ylo@example.com" and hadn't noticed the misspelling of "yli".</span></span> <span data-ttu-id="5790d-236">Non saranno in grado di usare il ripristino della password perché l'app non ha la posta elettronica corretta.</span><span class="sxs-lookup"><span data-stu-id="5790d-236">They wouldn't be able to use password recovery because the app doesn't have their correct email.</span></span> <span data-ttu-id="5790d-237">La conferma tramite posta elettronica garantisce una protezione limitata dai bot.</span><span class="sxs-lookup"><span data-stu-id="5790d-237">Email confirmation provides limited protection from bots.</span></span> <span data-ttu-id="5790d-238">La conferma tramite posta elettronica non fornisce protezione da utenti malintenzionati con molti account di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-238">Email confirmation doesn't provide protection from malicious users with many email accounts.</span></span>

<span data-ttu-id="5790d-239">In genere si vuole impedire ai nuovi utenti di inviare dati al sito Web prima di avere un messaggio di posta elettronica confermato.</span><span class="sxs-lookup"><span data-stu-id="5790d-239">You generally want to prevent new users from posting any data to your web site before they have a confirmed email.</span></span>

<span data-ttu-id="5790d-240">Aggiornamento `Startup.ConfigureServices`  per richiedere un messaggio di posta elettronica confermato:</span><span class="sxs-lookup"><span data-stu-id="5790d-240">Update `Startup.ConfigureServices`  to require a confirmed email:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=8-11)]

<span data-ttu-id="5790d-241">`config.SignIn.RequireConfirmedEmail = true;` impedisce agli utenti registrati di accedere fino a quando non viene confermata la posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-241">`config.SignIn.RequireConfirmedEmail = true;` prevents registered users from logging in until their email is confirmed.</span></span>

### <a name="configure-email-provider"></a><span data-ttu-id="5790d-242">Configurare il provider di posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-242">Configure email provider</span></span>

<span data-ttu-id="5790d-243">In questa esercitazione viene usato [SendGrid](https://sendgrid.com) per inviare messaggi di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-243">In this tutorial, [SendGrid](https://sendgrid.com) is used to send email.</span></span> <span data-ttu-id="5790d-244">Per inviare messaggi di posta elettronica sono necessari un account e una chiave di SendGrid.</span><span class="sxs-lookup"><span data-stu-id="5790d-244">You need a SendGrid account and key to send email.</span></span> <span data-ttu-id="5790d-245">È possibile usare altri provider di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-245">You can use other email providers.</span></span> <span data-ttu-id="5790d-246">ASP.NET Core 2. x include `System.Net.Mail` , che consente di inviare messaggi di posta elettronica dall'app.</span><span class="sxs-lookup"><span data-stu-id="5790d-246">ASP.NET Core 2.x includes `System.Net.Mail`, which allows you to send email from your app.</span></span> <span data-ttu-id="5790d-247">È consigliabile usare SendGrid o un altro servizio di posta elettronica per inviare messaggi di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-247">We recommend you use SendGrid or another email service to send email.</span></span> <span data-ttu-id="5790d-248">SMTP è difficile da proteggere e configurare correttamente.</span><span class="sxs-lookup"><span data-stu-id="5790d-248">SMTP is difficult to secure and set up correctly.</span></span>

<span data-ttu-id="5790d-249">Creare una classe per recuperare la chiave di posta elettronica sicura.</span><span class="sxs-lookup"><span data-stu-id="5790d-249">Create a class to fetch the secure email key.</span></span> <span data-ttu-id="5790d-250">Per questo esempio, creare *Servizi/AuthMessageSenderOptions. cs*:</span><span class="sxs-lookup"><span data-stu-id="5790d-250">For this sample, create *Services/AuthMessageSenderOptions.cs*:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a><span data-ttu-id="5790d-251">Configurare i segreti utente di SendGrid</span><span class="sxs-lookup"><span data-stu-id="5790d-251">Configure SendGrid user secrets</span></span>

<span data-ttu-id="5790d-252">Impostare `SendGridUser` e `SendGridKey` con lo [strumento di gestione dei segreti](xref:security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="5790d-252">Set the `SendGridUser` and `SendGridKey` with the [secret-manager tool](xref:security/app-secrets).</span></span> <span data-ttu-id="5790d-253">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="5790d-253">For example:</span></span>

```console
C:/WebAppl>dotnet user-secrets set SendGridUser RickAndMSFT
info: Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

<span data-ttu-id="5790d-254">In Windows, gestione Secret archivia le coppie chiave/valore in una *secrets.js* nel file nella `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` Directory.</span><span class="sxs-lookup"><span data-stu-id="5790d-254">On Windows, Secret Manager stores keys/value pairs in a *secrets.json* file in the `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` directory.</span></span>

<span data-ttu-id="5790d-255">Il contenuto del *secrets.jsnel* file non è crittografato.</span><span class="sxs-lookup"><span data-stu-id="5790d-255">The contents of the *secrets.json* file aren't encrypted.</span></span> <span data-ttu-id="5790d-256">Il markup seguente mostra il *secrets.jssu* file.</span><span class="sxs-lookup"><span data-stu-id="5790d-256">The following markup shows the *secrets.json* file.</span></span> <span data-ttu-id="5790d-257">Il `SendGridKey` valore è stato rimosso.</span><span class="sxs-lookup"><span data-stu-id="5790d-257">The `SendGridKey` value has been removed.</span></span>

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

<span data-ttu-id="5790d-258">Per ulteriori informazioni, vedere il [modello](xref:fundamentals/configuration/options) e la [configurazione](xref:fundamentals/configuration/index)delle opzioni.</span><span class="sxs-lookup"><span data-stu-id="5790d-258">For more information, see the [Options pattern](xref:fundamentals/configuration/options) and [configuration](xref:fundamentals/configuration/index).</span></span>

### <a name="install-sendgrid"></a><span data-ttu-id="5790d-259">Installare SendGrid</span><span class="sxs-lookup"><span data-stu-id="5790d-259">Install SendGrid</span></span>

<span data-ttu-id="5790d-260">Questa esercitazione illustra come aggiungere notifiche tramite posta elettronica tramite [SendGrid](https://sendgrid.com/), ma è possibile inviare messaggi di posta elettronica usando SMTP e altri meccanismi.</span><span class="sxs-lookup"><span data-stu-id="5790d-260">This tutorial shows how to add email notifications through [SendGrid](https://sendgrid.com/), but you can send email using SMTP and other mechanisms.</span></span>

<span data-ttu-id="5790d-261">Installare il `SendGrid` pacchetto NuGet:</span><span class="sxs-lookup"><span data-stu-id="5790d-261">Install the `SendGrid` NuGet package:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5790d-262">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5790d-262">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5790d-263">Dalla console di gestione pacchetti immettere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5790d-263">From the Package Manager Console, enter the following command:</span></span>

```powershell
Install-Package SendGrid
```

# <a name="net-core-cli"></a>[<span data-ttu-id="5790d-264">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="5790d-264">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="5790d-265">Dalla console di immettere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5790d-265">From the console, enter the following command:</span></span>

```dotnetcli
dotnet add package SendGrid
```

---

<span data-ttu-id="5790d-266">Per una registrazione gratuita per un account SendGrid gratuito, vedere [Introduzione a SendGrid](https://sendgrid.com/free/) .</span><span class="sxs-lookup"><span data-stu-id="5790d-266">See [Get Started with SendGrid for Free](https://sendgrid.com/free/) to register for a free SendGrid account.</span></span>

### <a name="implement-iemailsender"></a><span data-ttu-id="5790d-267">Implementare IEmailSender</span><span class="sxs-lookup"><span data-stu-id="5790d-267">Implement IEmailSender</span></span>

<span data-ttu-id="5790d-268">Per implementare `IEmailSender` , creare *Services/EmailSender. cs* con codice simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="5790d-268">To Implement `IEmailSender`, create *Services/EmailSender.cs* with code similar to the following:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a><span data-ttu-id="5790d-269">Configurare l'avvio per il supporto della posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-269">Configure startup to support email</span></span>

<span data-ttu-id="5790d-270">Aggiungere il codice seguente al `ConfigureServices` metodo nel file *Startup.cs* :</span><span class="sxs-lookup"><span data-stu-id="5790d-270">Add the following code to the `ConfigureServices` method in the *Startup.cs* file:</span></span>

* <span data-ttu-id="5790d-271">Aggiungere `EmailSender` come servizio temporaneo.</span><span class="sxs-lookup"><span data-stu-id="5790d-271">Add `EmailSender` as a transient service.</span></span>
* <span data-ttu-id="5790d-272">Registrare l' `AuthMessageSenderOptions` istanza di configurazione.</span><span class="sxs-lookup"><span data-stu-id="5790d-272">Register the `AuthMessageSenderOptions` configuration instance.</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=15-99)]

## <a name="enable-account-confirmation-and-password-recovery"></a><span data-ttu-id="5790d-273">Abilitare la conferma dell'account e il recupero della password</span><span class="sxs-lookup"><span data-stu-id="5790d-273">Enable account confirmation and password recovery</span></span>

<span data-ttu-id="5790d-274">Il modello include il codice per la conferma dell'account e il recupero della password.</span><span class="sxs-lookup"><span data-stu-id="5790d-274">The template has the code for account confirmation and password recovery.</span></span> <span data-ttu-id="5790d-275">Trovare il `OnPostAsync` metodo in *aree/ Identity /pages/account/Register.cshtml.cs*.</span><span class="sxs-lookup"><span data-stu-id="5790d-275">Find the `OnPostAsync` method in *Areas/Identity/Pages/Account/Register.cshtml.cs*.</span></span>

<span data-ttu-id="5790d-276">Impedire l'accesso automatico degli utenti appena registrati impostando come commento la riga seguente:</span><span class="sxs-lookup"><span data-stu-id="5790d-276">Prevent newly registered users from being automatically signed in by commenting out the following line:</span></span>

```csharp
await _signInManager.SignInAsync(user, isPersistent: false);
```

<span data-ttu-id="5790d-277">Il metodo completo viene visualizzato con la riga modificata evidenziata:</span><span class="sxs-lookup"><span data-stu-id="5790d-277">The complete method is shown with the changed line highlighted:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/Areas/Identity/Pages/Account/Register.cshtml.cs?highlight=22&name=snippet_Register)]

## <a name="register-confirm-email-and-reset-password"></a><span data-ttu-id="5790d-278">Registrare, confermare la posta elettronica e reimpostare la password</span><span class="sxs-lookup"><span data-stu-id="5790d-278">Register, confirm email, and reset password</span></span>

<span data-ttu-id="5790d-279">Eseguire l'app Web e testare la conferma dell'account e il flusso di recupero della password.</span><span class="sxs-lookup"><span data-stu-id="5790d-279">Run the web app, and test the account confirmation and password recovery flow.</span></span>

* <span data-ttu-id="5790d-280">Eseguire l'app e registrare un nuovo utente</span><span class="sxs-lookup"><span data-stu-id="5790d-280">Run the app and register a new user</span></span>
* <span data-ttu-id="5790d-281">Controllare la posta elettronica per il collegamento di conferma dell'account.</span><span class="sxs-lookup"><span data-stu-id="5790d-281">Check your email for the account confirmation link.</span></span> <span data-ttu-id="5790d-282">Se non si riceve il messaggio di posta elettronica, vedere [debug](#debug) della posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-282">See [Debug email](#debug) if you don't get the email.</span></span>
* <span data-ttu-id="5790d-283">Fare clic sul collegamento per confermare la posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-283">Click the link to confirm your email.</span></span>
* <span data-ttu-id="5790d-284">Accedere con l'indirizzo di posta elettronica e la password.</span><span class="sxs-lookup"><span data-stu-id="5790d-284">Sign in with your email and password.</span></span>
* <span data-ttu-id="5790d-285">Uscire,</span><span class="sxs-lookup"><span data-stu-id="5790d-285">Sign out.</span></span>

### <a name="view-the-manage-page"></a><span data-ttu-id="5790d-286">Visualizzare la pagina Gestisci</span><span class="sxs-lookup"><span data-stu-id="5790d-286">View the manage page</span></span>

<span data-ttu-id="5790d-287">Selezionare il nome utente nel browser: ![ finestra del browser con nome utente](accconfirm/_static/un.png)</span><span class="sxs-lookup"><span data-stu-id="5790d-287">Select your user name in the browser: ![browser window with user name](accconfirm/_static/un.png)</span></span>

<span data-ttu-id="5790d-288">La pagina Gestisci viene visualizzata con la scheda **profilo** selezionata.</span><span class="sxs-lookup"><span data-stu-id="5790d-288">The manage page is displayed with the **Profile** tab selected.</span></span> <span data-ttu-id="5790d-289">Viene visualizzata una casella di controllo che indica che il messaggio di **posta elettronica è** stato confermato.</span><span class="sxs-lookup"><span data-stu-id="5790d-289">The **Email** shows a check box indicating the email has been confirmed.</span></span>

### <a name="test-password-reset"></a><span data-ttu-id="5790d-290">Testare la reimpostazione della password</span><span class="sxs-lookup"><span data-stu-id="5790d-290">Test password reset</span></span>

* <span data-ttu-id="5790d-291">Se è stato eseguito l'accesso, selezionare **Disconnetti**.</span><span class="sxs-lookup"><span data-stu-id="5790d-291">If you're signed in, select **Logout**.</span></span>
* <span data-ttu-id="5790d-292">Selezionare il collegamento **Accedi** e selezionare il collegamento **password dimenticata?** .</span><span class="sxs-lookup"><span data-stu-id="5790d-292">Select the **Log in** link and select the **Forgot your password?** link.</span></span>
* <span data-ttu-id="5790d-293">Immettere il messaggio di posta elettronica usato per registrare l'account.</span><span class="sxs-lookup"><span data-stu-id="5790d-293">Enter the email you used to register the account.</span></span>
* <span data-ttu-id="5790d-294">Viene inviato un messaggio di posta elettronica con un collegamento per reimpostare la password.</span><span class="sxs-lookup"><span data-stu-id="5790d-294">An email with a link to reset your password is sent.</span></span> <span data-ttu-id="5790d-295">Controllare la posta elettronica e fare clic sul collegamento per reimpostare la password.</span><span class="sxs-lookup"><span data-stu-id="5790d-295">Check your email and click the link to reset your password.</span></span> <span data-ttu-id="5790d-296">Dopo che la password è stata reimpostata, è possibile accedere con l'indirizzo di posta elettronica e la nuova password.</span><span class="sxs-lookup"><span data-stu-id="5790d-296">After your password has been successfully reset, you can sign in with your email and new password.</span></span>

## <a name="change-email-and-activity-timeout"></a><span data-ttu-id="5790d-297">Modificare il timeout dell'attività e del messaggio di posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-297">Change email and activity timeout</span></span>

<span data-ttu-id="5790d-298">Il timeout di inattività predefinito è di 14 giorni.</span><span class="sxs-lookup"><span data-stu-id="5790d-298">The default inactivity timeout is 14 days.</span></span> <span data-ttu-id="5790d-299">Il codice seguente imposta il timeout di inattività su 5 giorni:</span><span class="sxs-lookup"><span data-stu-id="5790d-299">The following code sets the inactivity timeout to 5 days:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a><span data-ttu-id="5790d-300">Modifica tutte le durate dei token di protezione dati</span><span class="sxs-lookup"><span data-stu-id="5790d-300">Change all data protection token lifespans</span></span>

<span data-ttu-id="5790d-301">Il codice seguente modifica il periodo di timeout di tutti i token di protezione dati in 3 ore:</span><span class="sxs-lookup"><span data-stu-id="5790d-301">The following code changes all data protection tokens timeout period to 3 hours:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAllTokens.cs?name=snippet1&highlight=15-16)]

<span data-ttu-id="5790d-302">I Identity token utente predefiniti (vedere [AspNetCore/src/ Identity /Extensions.core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) hanno un timeout di un [giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span><span class="sxs-lookup"><span data-stu-id="5790d-302">The built in Identity user tokens (see [AspNetCore/src/Identity/Extensions.Core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) )have a [one day timeout](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span></span>

### <a name="change-the-email-token-lifespan"></a><span data-ttu-id="5790d-303">Modificare la durata del token di posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-303">Change the email token lifespan</span></span>

<span data-ttu-id="5790d-304">La durata del token predefinita [dei Identity token utente](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) è [un giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span><span class="sxs-lookup"><span data-stu-id="5790d-304">The default token lifespan of [the Identity user tokens](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) is [one day](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).</span></span> <span data-ttu-id="5790d-305">Questa sezione illustra come modificare la durata del token di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-305">This section shows how to change the email token lifespan.</span></span>

<span data-ttu-id="5790d-306">Aggiungere un [DataProtectorTokenProvider \<TUser> ](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) personalizzato e <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions> :</span><span class="sxs-lookup"><span data-stu-id="5790d-306">Add a custom [DataProtectorTokenProvider\<TUser>](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) and <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions>:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

<span data-ttu-id="5790d-307">Aggiungere il provider personalizzato al contenitore del servizio:</span><span class="sxs-lookup"><span data-stu-id="5790d-307">Add the custom provider to the service container:</span></span>

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupEmail.cs?name=snippet1&highlight=10-13,18)]

### <a name="resend-email-confirmation"></a><span data-ttu-id="5790d-308">Invia di nuovo la conferma tramite posta elettronica</span><span class="sxs-lookup"><span data-stu-id="5790d-308">Resend email confirmation</span></span>

<span data-ttu-id="5790d-309">Vedere [il problema in GitHub](https://github.com/dotnet/AspNetCore/issues/5410).</span><span class="sxs-lookup"><span data-stu-id="5790d-309">See [this GitHub issue](https://github.com/dotnet/AspNetCore/issues/5410).</span></span>

<a name="debug"></a>

### <a name="debug-email"></a><span data-ttu-id="5790d-310">Posta elettronica di debug</span><span class="sxs-lookup"><span data-stu-id="5790d-310">Debug email</span></span>

<span data-ttu-id="5790d-311">Se non è possibile ottenere la posta elettronica:</span><span class="sxs-lookup"><span data-stu-id="5790d-311">If you can't get email working:</span></span>

* <span data-ttu-id="5790d-312">Impostare un punto di interruzione in `EmailSender.Execute` per verificare che `SendGridClient.SendEmailAsync` venga chiamato.</span><span class="sxs-lookup"><span data-stu-id="5790d-312">Set a breakpoint in `EmailSender.Execute` to verify `SendGridClient.SendEmailAsync` is called.</span></span>
* <span data-ttu-id="5790d-313">Creare un' [app console per inviare messaggi di posta elettronica](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) usando codice simile a `EmailSender.Execute` .</span><span class="sxs-lookup"><span data-stu-id="5790d-313">Create a [console app to send email](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) using similar code to `EmailSender.Execute`.</span></span>
* <span data-ttu-id="5790d-314">Esaminare la pagina [attività posta elettronica](https://sendgrid.com/docs/User_Guide/email_activity.html) .</span><span class="sxs-lookup"><span data-stu-id="5790d-314">Review the [Email Activity](https://sendgrid.com/docs/User_Guide/email_activity.html) page.</span></span>
* <span data-ttu-id="5790d-315">Controllare la cartella della posta indesiderata.</span><span class="sxs-lookup"><span data-stu-id="5790d-315">Check your spam folder.</span></span>
* <span data-ttu-id="5790d-316">Prova un altro alias di posta elettronica in un provider di posta elettronica diverso (Microsoft, Yahoo, Gmail e così via)</span><span class="sxs-lookup"><span data-stu-id="5790d-316">Try another email alias on a different email provider (Microsoft, Yahoo, Gmail, etc.)</span></span>
* <span data-ttu-id="5790d-317">Provare a inviare a diversi account di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-317">Try sending to different email accounts.</span></span>

<span data-ttu-id="5790d-318">**Una procedura di sicurezza consigliata** consiste nel **non** usare i segreti di produzione per test e sviluppo.</span><span class="sxs-lookup"><span data-stu-id="5790d-318">**A security best practice** is to **not** use production secrets in test and development.</span></span> <span data-ttu-id="5790d-319">Se si pubblica l'app in Azure, è possibile impostare SendGrid Secrets come impostazioni dell'applicazione nel portale dell'app Web di Azure.</span><span class="sxs-lookup"><span data-stu-id="5790d-319">If you publish the app to Azure, you can set the SendGrid secrets as application settings in the Azure Web App portal.</span></span> <span data-ttu-id="5790d-320">Il sistema di configurazione è configurato per la lettura delle chiavi dalle variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="5790d-320">The configuration system is set up to read keys from environment variables.</span></span>

## <a name="combine-social-and-local-login-accounts"></a><span data-ttu-id="5790d-321">Combinare account di accesso di social networking e locali</span><span class="sxs-lookup"><span data-stu-id="5790d-321">Combine social and local login accounts</span></span>

<span data-ttu-id="5790d-322">Per completare questa sezione, è necessario innanzitutto abilitare un provider di autenticazione esterno.</span><span class="sxs-lookup"><span data-stu-id="5790d-322">To complete this section, you must first enable an external authentication provider.</span></span> <span data-ttu-id="5790d-323">Vedere [l'autenticazione di Facebook, Google e del provider esterno](xref:security/authentication/social/index).</span><span class="sxs-lookup"><span data-stu-id="5790d-323">See [Facebook, Google, and external provider authentication](xref:security/authentication/social/index).</span></span>

<span data-ttu-id="5790d-324">È possibile combinare account locali e di social networking facendo clic sul collegamento di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="5790d-324">You can combine local and social accounts by clicking on your email link.</span></span> <span data-ttu-id="5790d-325">Nella sequenza seguente " RickAndMSFT@gmail.com " viene creato per la prima volta come account di accesso locale; tuttavia, è possibile creare prima l'account come account di accesso di social networking, quindi aggiungere un account di accesso locale.</span><span class="sxs-lookup"><span data-stu-id="5790d-325">In the following sequence, "RickAndMSFT@gmail.com" is first created as a local login; however, you can create the account as a social login first, then add a local login.</span></span>

![Applicazione Web: RickAndMSFT@gmail.com utente autenticato](accconfirm/_static/rick.png)

<span data-ttu-id="5790d-327">Fare clic sul collegamento **Gestisci** .</span><span class="sxs-lookup"><span data-stu-id="5790d-327">Click on the **Manage** link.</span></span> <span data-ttu-id="5790d-328">Si notino gli account di accesso di social network (0) esterni associati a questo account.</span><span class="sxs-lookup"><span data-stu-id="5790d-328">Note the 0 external (social logins) associated with this account.</span></span>

![Gestisci visualizzazione](accconfirm/_static/manage.png)

<span data-ttu-id="5790d-330">Fare clic sul collegamento a un altro servizio di accesso e accettare le richieste dell'app.</span><span class="sxs-lookup"><span data-stu-id="5790d-330">Click the link to another login service and accept the app requests.</span></span> <span data-ttu-id="5790d-331">Nell'immagine seguente, Facebook è il provider di autenticazione esterno:</span><span class="sxs-lookup"><span data-stu-id="5790d-331">In the following image, Facebook is the external authentication provider:</span></span>

![Gestire gli account di accesso esterni visualizzare l'elenco di Facebook](accconfirm/_static/fb.png)

<span data-ttu-id="5790d-333">I due account sono stati combinati.</span><span class="sxs-lookup"><span data-stu-id="5790d-333">The two accounts have been combined.</span></span> <span data-ttu-id="5790d-334">È possibile accedere con uno dei due account.</span><span class="sxs-lookup"><span data-stu-id="5790d-334">You are able to sign in with either account.</span></span> <span data-ttu-id="5790d-335">È possibile che gli utenti aggiungano gli account locali nel caso in cui il servizio di autenticazione per l'accesso di social networking sia inattivo o più probabilmente abbiano perso l'accesso al proprio account di social networking.</span><span class="sxs-lookup"><span data-stu-id="5790d-335">You might want your users to add local accounts in case their social login authentication service is down, or more likely they've lost access to their social account.</span></span>

## <a name="enable-account-confirmation-after-a-site-has-users"></a><span data-ttu-id="5790d-336">Abilitare la conferma dell'account dopo che un sito ha utenti</span><span class="sxs-lookup"><span data-stu-id="5790d-336">Enable account confirmation after a site has users</span></span>

<span data-ttu-id="5790d-337">L'abilitazione della conferma dell'account in un sito con utenti blocca tutti gli utenti esistenti.</span><span class="sxs-lookup"><span data-stu-id="5790d-337">Enabling account confirmation on a site with users locks out all the existing users.</span></span> <span data-ttu-id="5790d-338">Gli utenti esistenti sono bloccati perché i relativi account non sono confermati.</span><span class="sxs-lookup"><span data-stu-id="5790d-338">Existing users are locked out because their accounts aren't confirmed.</span></span> <span data-ttu-id="5790d-339">Per aggirare il blocco utente esistente, utilizzare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="5790d-339">To work around existing user lockout, use one of the following approaches:</span></span>

* <span data-ttu-id="5790d-340">Aggiornare il database per contrassegnare tutti gli utenti esistenti come confermati.</span><span class="sxs-lookup"><span data-stu-id="5790d-340">Update the database to mark all existing users as being confirmed.</span></span>
* <span data-ttu-id="5790d-341">Verificare gli utenti esistenti.</span><span class="sxs-lookup"><span data-stu-id="5790d-341">Confirm existing users.</span></span> <span data-ttu-id="5790d-342">Ad esempio, invio in batch di messaggi di posta elettronica con collegamenti di conferma.</span><span class="sxs-lookup"><span data-stu-id="5790d-342">For example, batch-send emails with confirmation links.</span></span>

::: moniker-end
