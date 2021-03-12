---
title: Autenticazione a due fattori con SMS in ASP.NET Core
author: rick-anderson
description: Informazioni su come configurare l'autenticazione a due fattori (2FA) con un'app ASP.NET Core.
monikerRange: < aspnetcore-2.0
ms.author: riande
ms.date: 09/22/2018
ms.custom: mvc, seodec18
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
uid: security/authentication/2fa
ms.openlocfilehash: 1f77f3f4b7e9dd558e9869992e2f1f4d185e5b10
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586852"
---
# <a name="two-factor-authentication-with-sms-in-aspnet-core"></a><span data-ttu-id="2a2d7-103">Autenticazione a due fattori con SMS in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2a2d7-103">Two-factor authentication with SMS in ASP.NET Core</span></span>

<span data-ttu-id="2a2d7-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Swiss-sviluppatori](https://github.com/Swiss-Devs)</span><span class="sxs-lookup"><span data-stu-id="2a2d7-104">By [Rick Anderson](https://twitter.com/RickAndMSFT) and [Swiss-Devs](https://github.com/Swiss-Devs)</span></span>

>[!WARNING]
> <span data-ttu-id="2a2d7-105">Le app di autenticazione a due fattori (2FA), che usano un algoritmo di password monouso basato sul tempo (TOTP), sono l'approccio consigliato per 2FA.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-105">Two factor authentication (2FA) authenticator apps, using a Time-based One-time Password Algorithm (TOTP), are the industry recommended approach for 2FA.</span></span> <span data-ttu-id="2a2d7-106">2FA con TOTP è preferibile a SMS 2FA.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-106">2FA using TOTP is preferred to SMS 2FA.</span></span> <span data-ttu-id="2a2d7-107">Per altre informazioni, vedere [abilitare la generazione di codice a matrice per le app TOTP Authenticator in ASP.NET Core](xref:security/authentication/identity-enable-qrcodes) per ASP.NET Core 2,0 e versioni successive.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-107">For more information, see [Enable QR Code generation for TOTP authenticator apps in ASP.NET Core](xref:security/authentication/identity-enable-qrcodes) for ASP.NET Core 2.0 and later.</span></span>

<span data-ttu-id="2a2d7-108">Questa esercitazione illustra come configurare l'autenticazione a due fattori (2FA) tramite SMS.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-108">This tutorial shows how to set up two-factor authentication (2FA) using SMS.</span></span> <span data-ttu-id="2a2d7-109">Vengono fornite istruzioni per [Twilio](https://www.twilio.com/) e [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/), ma è possibile usare qualsiasi altro provider SMS.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-109">Instructions are given for [twilio](https://www.twilio.com/) and [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/), but you can use any other SMS provider.</span></span> <span data-ttu-id="2a2d7-110">Prima di iniziare questa esercitazione, è consigliabile completare la [conferma dell'account e il recupero della password](xref:security/authentication/accconfirm) .</span><span class="sxs-lookup"><span data-stu-id="2a2d7-110">We recommend you complete [Account Confirmation and Password Recovery](xref:security/authentication/accconfirm) before starting this tutorial.</span></span>

<span data-ttu-id="2a2d7-111">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/2fa/sample/Web2FA).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-111">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/2fa/sample/Web2FA).</span></span> <span data-ttu-id="2a2d7-112">[Come eseguire il download](xref:index#how-to-download-a-sample).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-112">[How to download](xref:index#how-to-download-a-sample).</span></span>

## <a name="create-a-new-aspnet-core-project"></a><span data-ttu-id="2a2d7-113">Creare un nuovo progetto ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2a2d7-113">Create a new ASP.NET Core project</span></span>

<span data-ttu-id="2a2d7-114">Creare una nuova app Web di ASP.NET Core denominata `Web2FA` con singoli account utente.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-114">Create a new ASP.NET Core web app named `Web2FA` with individual user accounts.</span></span> <span data-ttu-id="2a2d7-115">Seguire le istruzioni in <xref:security/enforcing-ssl> per configurare e richiedere HTTPS.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-115">Follow the instructions in <xref:security/enforcing-ssl> to set up and require HTTPS.</span></span>

### <a name="create-an-sms-account"></a><span data-ttu-id="2a2d7-116">Creare un account SMS</span><span class="sxs-lookup"><span data-stu-id="2a2d7-116">Create an SMS account</span></span>

<span data-ttu-id="2a2d7-117">Creare un account SMS, ad esempio, da [Twilio](https://www.twilio.com/) o [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-117">Create an SMS account, for example, from [twilio](https://www.twilio.com/) or [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/).</span></span> <span data-ttu-id="2a2d7-118">Registrare le credenziali di autenticazione (per Twilio: accountSid e authToken, per ASPSMS: userKey e password).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-118">Record the authentication credentials (for twilio: accountSid and authToken, for ASPSMS: Userkey and Password).</span></span>

#### <a name="figuring-out-sms-provider-credentials"></a><span data-ttu-id="2a2d7-119">Individuazione delle credenziali del provider SMS</span><span class="sxs-lookup"><span data-stu-id="2a2d7-119">Figuring out SMS Provider credentials</span></span>

<span data-ttu-id="2a2d7-120">**Twilio**</span><span class="sxs-lookup"><span data-stu-id="2a2d7-120">**Twilio:**</span></span>

<span data-ttu-id="2a2d7-121">Dalla scheda Dashboard dell'account Twilio copiare il **SID dell'account** e il **token di autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-121">From the Dashboard tab of your Twilio account, copy the **Account SID** and **Auth token**.</span></span>

<span data-ttu-id="2a2d7-122">**ASPSMS:**</span><span class="sxs-lookup"><span data-stu-id="2a2d7-122">**ASPSMS:**</span></span>

<span data-ttu-id="2a2d7-123">Dalle impostazioni dell'account passare a **userKey** e copiarlo insieme alla **password**.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-123">From your account settings, navigate to **Userkey** and copy it together with your **Password**.</span></span>

<span data-ttu-id="2a2d7-124">Questi valori vengono archiviati in un secondo momento con lo strumento di gestione del segreto all'interno delle chiavi `SMSAccountIdentification` e `SMSAccountPassword` .</span><span class="sxs-lookup"><span data-stu-id="2a2d7-124">We will later store these values in with the secret-manager tool within the keys `SMSAccountIdentification` and `SMSAccountPassword`.</span></span>

#### <a name="specifying-senderid--originator"></a><span data-ttu-id="2a2d7-125">Specifica di SenderID/originator</span><span class="sxs-lookup"><span data-stu-id="2a2d7-125">Specifying SenderID / Originator</span></span>

<span data-ttu-id="2a2d7-126">**Twilio:** Dalla scheda numeri copiare il **numero di telefono** Twilio.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-126">**Twilio:** From the Numbers tab, copy your Twilio **phone number**.</span></span>

<span data-ttu-id="2a2d7-127">**ASPSMS:** Nel menu Sblocca autori sbloccare uno o più autori o scegliere un originatore alfanumerico (non supportato da tutte le reti).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-127">**ASPSMS:** Within the Unlock Originators Menu, unlock one or more Originators or choose an alphanumeric Originator (Not supported by all networks).</span></span>

<span data-ttu-id="2a2d7-128">Questo valore verrà archiviato in un secondo momento con lo strumento di gestione del segreto all'interno della chiave `SMSAccountFrom` .</span><span class="sxs-lookup"><span data-stu-id="2a2d7-128">We will later store this value with the secret-manager tool within the key `SMSAccountFrom`.</span></span>

### <a name="provide-credentials-for-the-sms-service"></a><span data-ttu-id="2a2d7-129">Fornire le credenziali per il servizio SMS</span><span class="sxs-lookup"><span data-stu-id="2a2d7-129">Provide credentials for the SMS service</span></span>

<span data-ttu-id="2a2d7-130">Per accedere all'account utente e alle impostazioni della chiave verrà usato il [modello di opzioni](xref:fundamentals/configuration/options) .</span><span class="sxs-lookup"><span data-stu-id="2a2d7-130">We'll use the [Options pattern](xref:fundamentals/configuration/options) to access the user account and key settings.</span></span>

* <span data-ttu-id="2a2d7-131">Creare una classe per recuperare la chiave SMS sicura.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-131">Create a class to fetch the secure SMS key.</span></span> <span data-ttu-id="2a2d7-132">Per questo esempio, la `SMSoptions` classe viene creata nel file *Services/SMSoptions. cs* .</span><span class="sxs-lookup"><span data-stu-id="2a2d7-132">For this sample, the `SMSoptions` class is created in the *Services/SMSoptions.cs* file.</span></span>

[!code-csharp[](2fa/sample/Web2FA/Services/SMSoptions.cs)]

<span data-ttu-id="2a2d7-133">Impostare `SMSAccountIdentification` `SMSAccountPassword` e `SMSAccountFrom` con lo strumento di [gestione dei segreti](xref:security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-133">Set the `SMSAccountIdentification`, `SMSAccountPassword` and `SMSAccountFrom` with the [secret-manager tool](xref:security/app-secrets).</span></span> <span data-ttu-id="2a2d7-134">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="2a2d7-134">For example:</span></span>

```none
C:/Web2FA/src/WebApp1>dotnet user-secrets set SMSAccountIdentification 12345
info: Successfully saved SMSAccountIdentification = 12345 to the secret store.
```

* <span data-ttu-id="2a2d7-135">Aggiungere il pacchetto NuGet per il provider SMS.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-135">Add the NuGet package for the SMS provider.</span></span> <span data-ttu-id="2a2d7-136">Dalla console di gestione pacchetti (PMC) eseguire:</span><span class="sxs-lookup"><span data-stu-id="2a2d7-136">From the Package Manager Console (PMC) run:</span></span>

<span data-ttu-id="2a2d7-137">**Twilio**</span><span class="sxs-lookup"><span data-stu-id="2a2d7-137">**Twilio:**</span></span>

`Install-Package Twilio`

<span data-ttu-id="2a2d7-138">**ASPSMS:**</span><span class="sxs-lookup"><span data-stu-id="2a2d7-138">**ASPSMS:**</span></span>

`Install-Package ASPSMS`

* <span data-ttu-id="2a2d7-139">Aggiungere il codice nel file *Services/MessageServices. cs* per abilitare SMS.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-139">Add code in the *Services/MessageServices.cs* file to enable SMS.</span></span> <span data-ttu-id="2a2d7-140">Usare la sezione Twilio o ASPSMS:</span><span class="sxs-lookup"><span data-stu-id="2a2d7-140">Use either the Twilio or the ASPSMS section:</span></span>

<span data-ttu-id="2a2d7-141">**Twilio**</span><span class="sxs-lookup"><span data-stu-id="2a2d7-141">**Twilio:**</span></span>  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_twilio.cs)]

<span data-ttu-id="2a2d7-142">**ASPSMS:**</span><span class="sxs-lookup"><span data-stu-id="2a2d7-142">**ASPSMS:**</span></span>  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_ASPSMS.cs)]

### <a name="configure-startup-to-use-smsoptions"></a><span data-ttu-id="2a2d7-143">Configurare l'avvio per l'uso `SMSoptions`</span><span class="sxs-lookup"><span data-stu-id="2a2d7-143">Configure startup to use `SMSoptions`</span></span>

<span data-ttu-id="2a2d7-144">Aggiungere `SMSoptions` al contenitore dei servizi nel `ConfigureServices` metodo in *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="2a2d7-144">Add `SMSoptions` to the service container in the `ConfigureServices` method in the *Startup.cs*:</span></span>

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet1&highlight=4)]

### <a name="enable-two-factor-authentication"></a><span data-ttu-id="2a2d7-145">Abilitare l'autenticazione a due fattori</span><span class="sxs-lookup"><span data-stu-id="2a2d7-145">Enable two-factor authentication</span></span>

<span data-ttu-id="2a2d7-146">Aprire il file di visualizzazione *views/Manage/index. cshtml* Razor e rimuovere i caratteri di commento, in modo che nessun markup sia impostato come commento.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-146">Open the *Views/Manage/Index.cshtml* Razor view file and remove the comment characters (so no markup is commented out).</span></span>

## <a name="log-in-with-two-factor-authentication"></a><span data-ttu-id="2a2d7-147">Eseguire l'accesso con l'autenticazione a due fattori</span><span class="sxs-lookup"><span data-stu-id="2a2d7-147">Log in with two-factor authentication</span></span>

* <span data-ttu-id="2a2d7-148">Eseguire l'app e registrare un nuovo utente</span><span class="sxs-lookup"><span data-stu-id="2a2d7-148">Run the app and register a new user</span></span>

![Visualizzazione di registrazione dell'applicazione Web aperta in Microsoft Edge](2fa/_static/login2fa1.png)

* <span data-ttu-id="2a2d7-150">Toccare il nome utente, che attiva il `Index` metodo di azione in Gestisci controller.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-150">Tap on your user name, which activates the `Index` action method in Manage controller.</span></span> <span data-ttu-id="2a2d7-151">Toccare quindi il collegamento **Aggiungi** numero di telefono.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-151">Then tap the phone number **Add** link.</span></span>

![Gestisci visualizzazione: toccare il collegamento "Aggiungi"](2fa/_static/login2fa2.png)

* <span data-ttu-id="2a2d7-153">Aggiungere un numero di telefono che riceverà il codice di verifica e toccare **Invia codice di verifica**.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-153">Add a phone number that will receive the verification code, and tap **Send verification code**.</span></span>

![Pagina Aggiungi numero di telefono](2fa/_static/login2fa3.png)

* <span data-ttu-id="2a2d7-155">Viene ricevuto un messaggio di testo con il codice di verifica.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-155">You will get a text message with the verification code.</span></span> <span data-ttu-id="2a2d7-156">Immettilo e tocca **Invia**</span><span class="sxs-lookup"><span data-stu-id="2a2d7-156">Enter it and tap **Submit**</span></span>

![Pagina Verifica numero di telefono](2fa/_static/login2fa4.png)

<span data-ttu-id="2a2d7-158">Se non viene visualizzato un messaggio di testo, vedere la pagina del log Twilio.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-158">If you don't get a text message, see twilio log page.</span></span>

* <span data-ttu-id="2a2d7-159">La visualizzazione gestione mostra che il numero di telefono è stato aggiunto correttamente.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-159">The Manage view shows your phone number was added successfully.</span></span>

![Gestisci visualizzazione-numero di telefono aggiunto correttamente](2fa/_static/login2fa5.png)

* <span data-ttu-id="2a2d7-161">Toccare **Abilita** per abilitare l'autenticazione a due fattori.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-161">Tap **Enable** to enable two-factor authentication.</span></span>

![Gestisci visualizzazione: Abilita autenticazione a due fattori](2fa/_static/login2fa6.png)

### <a name="test-two-factor-authentication"></a><span data-ttu-id="2a2d7-163">Testare l'autenticazione a due fattori</span><span class="sxs-lookup"><span data-stu-id="2a2d7-163">Test two-factor authentication</span></span>

* <span data-ttu-id="2a2d7-164">Disconnettersi.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-164">Log off.</span></span>

* <span data-ttu-id="2a2d7-165">Eseguire l'accesso.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-165">Log in.</span></span>

* <span data-ttu-id="2a2d7-166">Per l'account utente è stata abilitata l'autenticazione a due fattori, pertanto è necessario fornire il secondo fattore di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-166">The user account has enabled two-factor authentication, so you have to provide the second factor of authentication .</span></span> <span data-ttu-id="2a2d7-167">In questa esercitazione è stata abilitata la verifica telefonica.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-167">In this tutorial you have enabled phone verification.</span></span> <span data-ttu-id="2a2d7-168">I modelli predefiniti consentono inoltre di configurare la posta elettronica come secondo fattore.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-168">The built in templates also allow you to set up email as the second factor.</span></span> <span data-ttu-id="2a2d7-169">È possibile configurare altri secondi fattori per l'autenticazione, ad esempio i codici QR.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-169">You can set up additional second factors for authentication such as QR codes.</span></span> <span data-ttu-id="2a2d7-170">Toccare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-170">Tap **Submit**.</span></span>

![Invia visualizzazione codice di verifica](2fa/_static/login2fa7.png)

* <span data-ttu-id="2a2d7-172">Immettere il codice ricevuto nel messaggio SMS.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-172">Enter the code you get in the SMS message.</span></span>

* <span data-ttu-id="2a2d7-173">Quando si fa clic sulla casella di controllo **memorizza questo browser** , non è necessario usare 2FA per accedere quando si usa lo stesso dispositivo e browser.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-173">Clicking on the **Remember this browser** check box will exempt you from needing to use 2FA to log on when using the same device and browser.</span></span> <span data-ttu-id="2a2d7-174">Abilitazione di 2FA e facendo clic su **ricorda questo browser** fornirà una protezione 2FA avanzata da utenti malintenzionati che tentano di accedere all'account, purché non dispongano dell'accesso al dispositivo.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-174">Enabling 2FA and clicking on **Remember this browser** will provide you with strong 2FA protection from malicious users trying to access your account, as long as they don't have access to your device.</span></span> <span data-ttu-id="2a2d7-175">Questa operazione può essere eseguita su qualsiasi dispositivo privato usato regolarmente.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-175">You can do this on any private device you regularly use.</span></span> <span data-ttu-id="2a2d7-176">Impostando  **ricorda questo browser**, si ottiene la maggiore sicurezza di 2FA dai dispositivi che non vengono usati regolarmente e si ottiene la convenienza di non dover passare attraverso 2FA nei propri dispositivi.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-176">By setting  **Remember this browser**, you get the added security of 2FA from devices you don't regularly use, and you get the convenience on not having to go through 2FA on your own devices.</span></span>

![Verifica visualizzazione](2fa/_static/login2fa8.png)

## <a name="account-lockout-for-protecting-against-brute-force-attacks"></a><span data-ttu-id="2a2d7-178">Blocco degli account per la protezione da attacchi di forza bruta</span><span class="sxs-lookup"><span data-stu-id="2a2d7-178">Account lockout for protecting against brute force attacks</span></span>

<span data-ttu-id="2a2d7-179">Il blocco degli account è consigliato con 2FA.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-179">Account lockout is recommended with 2FA.</span></span> <span data-ttu-id="2a2d7-180">Quando un utente accede tramite un account locale o un account di social networking, viene archiviato ogni tentativo non riuscito in 2FA.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-180">Once a user signs in through a local account or social account, each failed attempt at 2FA is stored.</span></span> <span data-ttu-id="2a2d7-181">Se viene raggiunto il numero massimo di tentativi di accesso non riusciti, l'utente viene bloccato (impostazione predefinita: blocco di 5 minuti dopo 5 tentativi di accesso non riusciti).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-181">If the maximum failed access attempts is reached, the user is locked out (default: 5 minute lockout after 5 failed access attempts).</span></span> <span data-ttu-id="2a2d7-182">Una corretta autenticazione Reimposta il numero di tentativi di accesso non riusciti e reimposta il clock.</span><span class="sxs-lookup"><span data-stu-id="2a2d7-182">A successful authentication resets the failed access attempts count and resets the clock.</span></span> <span data-ttu-id="2a2d7-183">Il numero massimo di tentativi di accesso non riusciti e l'ora di blocco possono essere impostati con [MaxFailedAccessAttempts](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.maxfailedaccessattempts) e [DefaultLockoutTimeSpan](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.defaultlockouttimespan).</span><span class="sxs-lookup"><span data-stu-id="2a2d7-183">The maximum failed access attempts and lockout time can be set with [MaxFailedAccessAttempts](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.maxfailedaccessattempts) and [DefaultLockoutTimeSpan](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.defaultlockouttimespan).</span></span> <span data-ttu-id="2a2d7-184">Di seguito viene configurato il blocco degli account per 10 minuti dopo 10 tentativi di accesso non riusciti:</span><span class="sxs-lookup"><span data-stu-id="2a2d7-184">The following configures account lockout for 10 minutes after 10 failed access attempts:</span></span>

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet2&highlight=13-17)]

<span data-ttu-id="2a2d7-185">Confermare che [PasswordSignInAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.passwordsigninasync) imposta `lockoutOnFailure` su `true` :</span><span class="sxs-lookup"><span data-stu-id="2a2d7-185">Confirm that [PasswordSignInAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.passwordsigninasync) sets `lockoutOnFailure` to `true`:</span></span>

```csharp
var result = await _signInManager.PasswordSignInAsync(
                 Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: true);
```
