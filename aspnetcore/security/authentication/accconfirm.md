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
# <a name="account-confirmation-and-password-recovery-in-aspnet-core"></a>Account confirmation and password recovery in ASP.NET Core (Conferma dell'account e recupero della password in ASP.NET Core)

Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Ponant](https://github.com/Ponant)e [Joe Audette](https://twitter.com/joeaudette)

Questa esercitazione illustra come compilare un'app ASP.NET Core con la conferma della posta elettronica e la reimpostazione della password. Questa esercitazione **non** è un argomento iniziale. È necessario avere familiarità con:

* [ASP.NET Core](xref:tutorials/razor-pages/razor-pages-start)
* [autenticazione](xref:security/authentication/identity)
* [Entity Framework Core](xref:data/ef-mvc/intro)

<!-- see C:/Dropbox/wrk/Code/SendGridConsole/Program.cs -->

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a>Prerequisiti

[.NET Core 3,0 SDK o versione successiva](https://dotnet.microsoft.com/download/dotnet-core/3.0)

## <a name="create-and-test-a-web-app-with-authentication"></a>Creare e testare un'app Web con autenticazione

Eseguire i comandi seguenti per creare un'app Web con l'autenticazione di.

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet run
```

Eseguire l'app, selezionare il collegamento **Register** e registrare un utente. Una volta effettuata la registrazione, si verrà reindirizzati alla `/Identity/Account/RegisterConfirmation` pagina a contenente un collegamento per simulare la conferma della posta elettronica:

* Selezionare il `Click here to confirm your account` collegamento.
* Selezionare il collegamento di **accesso** e accedere con le stesse credenziali.
* Selezionare il `Hello YourEmail@provider.com!` collegamento per reindirizzare l'utente alla `/Identity/Account/Manage/PersonalData` pagina.
* Selezionare la scheda **Personal Data (dati personali** ) a sinistra e quindi fare clic su **Delete (Elimina**).

### <a name="configure-an-email-provider"></a>Configurare un provider di posta elettronica

In questa esercitazione viene usato [SendGrid](https://sendgrid.com) per inviare messaggi di posta elettronica. Per inviare messaggi di posta elettronica sono necessari un account e una chiave di SendGrid. È possibile usare altri provider di posta elettronica. È consigliabile usare SendGrid o un altro servizio di posta elettronica per inviare messaggi di posta elettronica. SMTP è difficile da proteggere e configurare correttamente.

L'account SendGrid può richiedere l' [aggiunta di un mittente](https://sendgrid.com/docs/ui/sending-email/senders/).

Creare una classe per recuperare la chiave di posta elettronica sicura. Per questo esempio, creare *Servizi/AuthMessageSenderOptions. cs*:

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>Configurare i segreti utente di SendGrid

Impostare `SendGridUser` e `SendGridKey` con lo [strumento di gestione dei segreti](xref:security/app-secrets). Ad esempio:

```dotnetcli
dotnet user-secrets set SendGridUser RickAndMSFT
dotnet user-secrets set SendGridKey <key>

Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

In Windows, gestione Secret archivia le coppie chiave/valore in una *secrets.js* nel file nella `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` Directory.

Il contenuto del *secrets.jsnel* file non è crittografato. Il markup seguente mostra il *secrets.jssu* file. Il `SendGridKey` valore è stato rimosso.

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

Per ulteriori informazioni, vedere il [modello](xref:fundamentals/configuration/options) e la [configurazione](xref:fundamentals/configuration/index)delle opzioni.

### <a name="install-sendgrid"></a>Installare SendGrid

Questa esercitazione illustra come aggiungere notifiche tramite posta elettronica tramite [SendGrid](https://sendgrid.com/), ma è possibile inviare messaggi di posta elettronica usando SMTP e altri meccanismi.

Installare il `SendGrid` pacchetto NuGet:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Dalla console di gestione pacchetti immettere il comando seguente:

```powershell
Install-Package SendGrid
```

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Dalla console di immettere il comando seguente:

```dotnetcli
dotnet add package SendGrid
```

---

Per una registrazione gratuita per un account SendGrid gratuito, vedere [Introduzione a SendGrid](https://sendgrid.com/free/) .

### <a name="implement-iemailsender"></a>Implementare IEmailSender

Per implementare `IEmailSender` , creare *Services/EmailSender. cs* con codice simile al seguente:

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>Configurare l'avvio per il supporto della posta elettronica

Aggiungere il codice seguente al `ConfigureServices` metodo nel file *Startup.cs* :

* Aggiungere `EmailSender` come servizio temporaneo.
* Registrare l' `AuthMessageSenderOptions` istanza di configurazione.

[!code-csharp[](accconfirm/sample/WebPWrecover30/Startup.cs?name=snippet1&highlight=11-15)]

## <a name="scaffold-registerconfirmation"></a>RegisterConfirmation ponteggi

Seguire le istruzioni per [impalcature Identity ](xref:security/authentication/scaffold-identity) e impalcature `RegisterConfirmation` .

<!-- .NET 5 fixes this, see
https://github.com/dotnet/aspnetcore/blob/main/src/Identity/UI/src/Areas/Identity/Pages/V4/Account/RegisterConfirmation.cshtml.cs#L74-L77
-->

[!INCLUDE[](~/includes/disableVer.md)]

## <a name="register-confirm-email-and-reset-password"></a>Registrare, confermare la posta elettronica e reimpostare la password

Eseguire l'app Web e testare la conferma dell'account e il flusso di recupero della password.

* Eseguire l'app e registrare un nuovo utente
* Controllare la posta elettronica per il collegamento di conferma dell'account. Se non si riceve il messaggio di posta elettronica, vedere [debug](#debug) della posta elettronica.
* Fare clic sul collegamento per confermare la posta elettronica.
* Accedere con l'indirizzo di posta elettronica e la password.
* Uscire,

### <a name="test-password-reset"></a>Testare la reimpostazione della password

* Se è stato eseguito l'accesso, selezionare **Disconnetti**.
* Selezionare il collegamento **Accedi** e selezionare il collegamento **password dimenticata?** .
* Immettere il messaggio di posta elettronica usato per registrare l'account.
* Viene inviato un messaggio di posta elettronica con un collegamento per reimpostare la password. Controllare la posta elettronica e fare clic sul collegamento per reimpostare la password. Dopo che la password è stata reimpostata, è possibile accedere con l'indirizzo di posta elettronica e la nuova password.

<a name="resend"></a>

## <a name="resend-email-confirmation"></a>Invia di nuovo la conferma tramite posta elettronica

In ASP.NET Core 5,0 e versioni successive selezionare il collegamento Invia di nuovo la **conferma tramite posta elettronica** nella pagina di **accesso** .

### <a name="change-email-and-activity-timeout"></a>Modificare il timeout dell'attività e del messaggio di posta elettronica

Il timeout di inattività predefinito è di 14 giorni. Il codice seguente imposta il timeout di inattività su 5 giorni:

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>Modifica tutte le durate dei token di protezione dati

Il codice seguente modifica il periodo di timeout di tutti i token di protezione dati in 3 ore:

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAllTokens.cs?name=snippet1&highlight=11-12)]

I Identity token utente predefiniti (vedere [AspNetCore/src/ Identity /Extensions.core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) hanno un timeout di un [giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).

### <a name="change-the-email-token-lifespan"></a>Modificare la durata del token di posta elettronica

La durata del token predefinita [dei Identity token utente](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) è [un giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs). Questa sezione illustra come modificare la durata del token di posta elettronica.

Aggiungere un [DataProtectorTokenProvider \<TUser> ](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) personalizzato e <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions> :

[!code-csharp[](accconfirm/sample/WebPWrecover30/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

Aggiungere il provider personalizzato al contenitore del servizio:

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupEmail.cs?name=snippet1&highlight=10-16)]

<a name="debug"></a>

### <a name="debug-email"></a>Posta elettronica di debug

Se non è possibile ottenere la posta elettronica:

* Impostare un punto di interruzione in `EmailSender.Execute` per verificare che `SendGridClient.SendEmailAsync` venga chiamato.
* Creare un' [app console per inviare messaggi di posta elettronica](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) usando codice simile a `EmailSender.Execute` .
* Esaminare la pagina [attività posta elettronica](https://sendgrid.com/docs/User_Guide/email_activity.html) .
* Controllare la cartella della posta indesiderata.
* Prova un altro alias di posta elettronica in un provider di posta elettronica diverso (Microsoft, Yahoo, Gmail e così via)
* Provare a inviare a diversi account di posta elettronica.

**Una procedura di sicurezza consigliata** consiste nel **non** usare i segreti di produzione per test e sviluppo. Se si pubblica l'app in Azure, impostare i segreti SendGrid come impostazioni dell'applicazione nel portale dell'app Web di Azure. Il sistema di configurazione è configurato per la lettura delle chiavi dalle variabili di ambiente.

## <a name="combine-social-and-local-login-accounts"></a>Combinare account di accesso di social networking e locali

Per completare questa sezione, è necessario innanzitutto abilitare un provider di autenticazione esterno. Vedere [l'autenticazione di Facebook, Google e del provider esterno](xref:security/authentication/social/index).

È possibile combinare account locali e di social networking facendo clic sul collegamento di posta elettronica. Nella sequenza seguente " RickAndMSFT@gmail.com " viene creato per la prima volta come account di accesso locale; tuttavia, è possibile creare prima l'account come account di accesso di social networking, quindi aggiungere un account di accesso locale.

![Applicazione Web: RickAndMSFT@gmail.com utente autenticato](accconfirm/_static/rick.png)

Fare clic sul collegamento **Gestisci** . Si notino gli account di accesso di social network (0) esterni associati a questo account.

![Gestisci visualizzazione](accconfirm/_static/manage.png)

Fare clic sul collegamento a un altro servizio di accesso e accettare le richieste dell'app. Nell'immagine seguente, Facebook è il provider di autenticazione esterno:

![Gestire gli account di accesso esterni visualizzare l'elenco di Facebook](accconfirm/_static/fb.png)

I due account sono stati combinati. È possibile accedere con uno dei due account. È possibile che gli utenti aggiungano gli account locali nel caso in cui il servizio di autenticazione per l'accesso di social networking sia inattivo o più probabilmente abbiano perso l'accesso al proprio account di social networking.

## <a name="enable-account-confirmation-after-a-site-has-users"></a>Abilitare la conferma dell'account dopo che un sito ha utenti

L'abilitazione della conferma dell'account in un sito con utenti blocca tutti gli utenti esistenti. Gli utenti esistenti sono bloccati perché i relativi account non sono confermati. Per aggirare il blocco utente esistente, utilizzare uno degli approcci seguenti:

* Aggiornare il database per contrassegnare tutti gli utenti esistenti come confermati.
* Verificare gli utenti esistenti. Ad esempio, invio in batch di messaggi di posta elettronica con collegamenti di conferma.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="prerequisites"></a>Prerequisiti

[.NET Core 2,2 SDK o versione successiva](https://dotnet.microsoft.com/download/dotnet-core)

## <a name="create-a-web--app-and-scaffold-identity"></a>Creare un'app Web e un patibolo Identity

Eseguire i comandi seguenti per creare un'app Web con l'autenticazione di.

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
> Se <xref:Microsoft.AspNetCore.Identity.PasswordOptions> sono configurati in `Startup.ConfigureServices` , la configurazione degli [ `[StringLength]` attributi](xref:System.ComponentModel.DataAnnotations.StringLengthAttribute) potrebbe essere necessaria per la `Password` proprietà nelle pagine con impalcature Identity . Una `InputModel` `Password` proprietà viene trovata nel `Areas/Identity/Pages/Account/Register.cshtml.cs` file dopo l'impalcatura Identity .

## <a name="test-new-user-registration"></a>Testare la registrazione di un nuovo utente

Eseguire l'app, selezionare il collegamento **Register** e registrare un utente. A questo punto, l'unica convalida sul messaggio di posta elettronica è con l' [`[EmailAddress]`](/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute) attributo. Dopo aver inviato la registrazione, si è connessi all'app. Più avanti nell'esercitazione, il codice viene aggiornato in modo che i nuovi utenti non possano accedere fino a quando non vengono convalidati i messaggi di posta elettronica.

[!INCLUDE[](~/includes/view-identity-db.md)]

Si noti che il `EmailConfirmed` campo della tabella è `False` .

Potrebbe essere necessario usare nuovamente questo messaggio di posta elettronica nel passaggio successivo quando l'app invia un messaggio di posta elettronica di conferma. Fare clic con il pulsante destro del mouse sulla riga e scegliere **Elimina**. L'eliminazione dell'alias di posta elettronica rende più semplice nei passaggi seguenti.

<a name="prevent-login-at-registration"></a>

## <a name="require-email-confirmation"></a>Richiedi conferma posta elettronica

È consigliabile confermare il messaggio di posta elettronica di una nuova registrazione utente. La conferma tramite posta elettronica consente di verificare che non stiano eseguendo la rappresentazione di un altro utente, ovvero che non sono stati registrati con il messaggio di posta elettronica di un altro utente. Si supponga di avere un forum di discussione e di voler impedire che " yli@example.com " si registri come " nolivetto@contoso.com ". Senza la conferma della posta elettronica, " nolivetto@contoso.com " potrebbe ricevere un messaggio di posta elettronica indesiderato dall'app. Si supponga che l'utente sia stato accidentalmente registrato come " ylo@example.com " e non abbia notato l'errore di ortografia di "Yli". Non saranno in grado di usare il ripristino della password perché l'app non ha la posta elettronica corretta. La conferma tramite posta elettronica garantisce una protezione limitata dai bot. La conferma tramite posta elettronica non fornisce protezione da utenti malintenzionati con molti account di posta elettronica.

In genere si vuole impedire ai nuovi utenti di inviare dati al sito Web prima di avere un messaggio di posta elettronica confermato.

Aggiornamento `Startup.ConfigureServices`  per richiedere un messaggio di posta elettronica confermato:

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=8-11)]

`config.SignIn.RequireConfirmedEmail = true;` impedisce agli utenti registrati di accedere fino a quando non viene confermata la posta elettronica.

### <a name="configure-email-provider"></a>Configurare il provider di posta elettronica

In questa esercitazione viene usato [SendGrid](https://sendgrid.com) per inviare messaggi di posta elettronica. Per inviare messaggi di posta elettronica sono necessari un account e una chiave di SendGrid. È possibile usare altri provider di posta elettronica. ASP.NET Core 2. x include `System.Net.Mail` , che consente di inviare messaggi di posta elettronica dall'app. È consigliabile usare SendGrid o un altro servizio di posta elettronica per inviare messaggi di posta elettronica. SMTP è difficile da proteggere e configurare correttamente.

Creare una classe per recuperare la chiave di posta elettronica sicura. Per questo esempio, creare *Servizi/AuthMessageSenderOptions. cs*:

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>Configurare i segreti utente di SendGrid

Impostare `SendGridUser` e `SendGridKey` con lo [strumento di gestione dei segreti](xref:security/app-secrets). Ad esempio:

```console
C:/WebAppl>dotnet user-secrets set SendGridUser RickAndMSFT
info: Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

In Windows, gestione Secret archivia le coppie chiave/valore in una *secrets.js* nel file nella `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` Directory.

Il contenuto del *secrets.jsnel* file non è crittografato. Il markup seguente mostra il *secrets.jssu* file. Il `SendGridKey` valore è stato rimosso.

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

Per ulteriori informazioni, vedere il [modello](xref:fundamentals/configuration/options) e la [configurazione](xref:fundamentals/configuration/index)delle opzioni.

### <a name="install-sendgrid"></a>Installare SendGrid

Questa esercitazione illustra come aggiungere notifiche tramite posta elettronica tramite [SendGrid](https://sendgrid.com/), ma è possibile inviare messaggi di posta elettronica usando SMTP e altri meccanismi.

Installare il `SendGrid` pacchetto NuGet:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Dalla console di gestione pacchetti immettere il comando seguente:

```powershell
Install-Package SendGrid
```

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Dalla console di immettere il comando seguente:

```dotnetcli
dotnet add package SendGrid
```

---

Per una registrazione gratuita per un account SendGrid gratuito, vedere [Introduzione a SendGrid](https://sendgrid.com/free/) .

### <a name="implement-iemailsender"></a>Implementare IEmailSender

Per implementare `IEmailSender` , creare *Services/EmailSender. cs* con codice simile al seguente:

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>Configurare l'avvio per il supporto della posta elettronica

Aggiungere il codice seguente al `ConfigureServices` metodo nel file *Startup.cs* :

* Aggiungere `EmailSender` come servizio temporaneo.
* Registrare l' `AuthMessageSenderOptions` istanza di configurazione.

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=15-99)]

## <a name="enable-account-confirmation-and-password-recovery"></a>Abilitare la conferma dell'account e il recupero della password

Il modello include il codice per la conferma dell'account e il recupero della password. Trovare il `OnPostAsync` metodo in *aree/ Identity /pages/account/Register.cshtml.cs*.

Impedire l'accesso automatico degli utenti appena registrati impostando come commento la riga seguente:

```csharp
await _signInManager.SignInAsync(user, isPersistent: false);
```

Il metodo completo viene visualizzato con la riga modificata evidenziata:

[!code-csharp[](accconfirm/sample/WebPWrecover22/Areas/Identity/Pages/Account/Register.cshtml.cs?highlight=22&name=snippet_Register)]

## <a name="register-confirm-email-and-reset-password"></a>Registrare, confermare la posta elettronica e reimpostare la password

Eseguire l'app Web e testare la conferma dell'account e il flusso di recupero della password.

* Eseguire l'app e registrare un nuovo utente
* Controllare la posta elettronica per il collegamento di conferma dell'account. Se non si riceve il messaggio di posta elettronica, vedere [debug](#debug) della posta elettronica.
* Fare clic sul collegamento per confermare la posta elettronica.
* Accedere con l'indirizzo di posta elettronica e la password.
* Uscire,

### <a name="view-the-manage-page"></a>Visualizzare la pagina Gestisci

Selezionare il nome utente nel browser: ![ finestra del browser con nome utente](accconfirm/_static/un.png)

La pagina Gestisci viene visualizzata con la scheda **profilo** selezionata. Viene visualizzata una casella di controllo che indica che il messaggio di **posta elettronica è** stato confermato.

### <a name="test-password-reset"></a>Testare la reimpostazione della password

* Se è stato eseguito l'accesso, selezionare **Disconnetti**.
* Selezionare il collegamento **Accedi** e selezionare il collegamento **password dimenticata?** .
* Immettere il messaggio di posta elettronica usato per registrare l'account.
* Viene inviato un messaggio di posta elettronica con un collegamento per reimpostare la password. Controllare la posta elettronica e fare clic sul collegamento per reimpostare la password. Dopo che la password è stata reimpostata, è possibile accedere con l'indirizzo di posta elettronica e la nuova password.

## <a name="change-email-and-activity-timeout"></a>Modificare il timeout dell'attività e del messaggio di posta elettronica

Il timeout di inattività predefinito è di 14 giorni. Il codice seguente imposta il timeout di inattività su 5 giorni:

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>Modifica tutte le durate dei token di protezione dati

Il codice seguente modifica il periodo di timeout di tutti i token di protezione dati in 3 ore:

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAllTokens.cs?name=snippet1&highlight=15-16)]

I Identity token utente predefiniti (vedere [AspNetCore/src/ Identity /Extensions.core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) hanno un timeout di un [giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs).

### <a name="change-the-email-token-lifespan"></a>Modificare la durata del token di posta elettronica

La durata del token predefinita [dei Identity token utente](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) è [un giorno](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs). Questa sezione illustra come modificare la durata del token di posta elettronica.

Aggiungere un [DataProtectorTokenProvider \<TUser> ](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1) personalizzato e <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions> :

[!code-csharp[](accconfirm/sample/WebPWrecover22/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

Aggiungere il provider personalizzato al contenitore del servizio:

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupEmail.cs?name=snippet1&highlight=10-13,18)]

### <a name="resend-email-confirmation"></a>Invia di nuovo la conferma tramite posta elettronica

Vedere [il problema in GitHub](https://github.com/dotnet/AspNetCore/issues/5410).

<a name="debug"></a>

### <a name="debug-email"></a>Posta elettronica di debug

Se non è possibile ottenere la posta elettronica:

* Impostare un punto di interruzione in `EmailSender.Execute` per verificare che `SendGridClient.SendEmailAsync` venga chiamato.
* Creare un' [app console per inviare messaggi di posta elettronica](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) usando codice simile a `EmailSender.Execute` .
* Esaminare la pagina [attività posta elettronica](https://sendgrid.com/docs/User_Guide/email_activity.html) .
* Controllare la cartella della posta indesiderata.
* Prova un altro alias di posta elettronica in un provider di posta elettronica diverso (Microsoft, Yahoo, Gmail e così via)
* Provare a inviare a diversi account di posta elettronica.

**Una procedura di sicurezza consigliata** consiste nel **non** usare i segreti di produzione per test e sviluppo. Se si pubblica l'app in Azure, è possibile impostare SendGrid Secrets come impostazioni dell'applicazione nel portale dell'app Web di Azure. Il sistema di configurazione è configurato per la lettura delle chiavi dalle variabili di ambiente.

## <a name="combine-social-and-local-login-accounts"></a>Combinare account di accesso di social networking e locali

Per completare questa sezione, è necessario innanzitutto abilitare un provider di autenticazione esterno. Vedere [l'autenticazione di Facebook, Google e del provider esterno](xref:security/authentication/social/index).

È possibile combinare account locali e di social networking facendo clic sul collegamento di posta elettronica. Nella sequenza seguente " RickAndMSFT@gmail.com " viene creato per la prima volta come account di accesso locale; tuttavia, è possibile creare prima l'account come account di accesso di social networking, quindi aggiungere un account di accesso locale.

![Applicazione Web: RickAndMSFT@gmail.com utente autenticato](accconfirm/_static/rick.png)

Fare clic sul collegamento **Gestisci** . Si notino gli account di accesso di social network (0) esterni associati a questo account.

![Gestisci visualizzazione](accconfirm/_static/manage.png)

Fare clic sul collegamento a un altro servizio di accesso e accettare le richieste dell'app. Nell'immagine seguente, Facebook è il provider di autenticazione esterno:

![Gestire gli account di accesso esterni visualizzare l'elenco di Facebook](accconfirm/_static/fb.png)

I due account sono stati combinati. È possibile accedere con uno dei due account. È possibile che gli utenti aggiungano gli account locali nel caso in cui il servizio di autenticazione per l'accesso di social networking sia inattivo o più probabilmente abbiano perso l'accesso al proprio account di social networking.

## <a name="enable-account-confirmation-after-a-site-has-users"></a>Abilitare la conferma dell'account dopo che un sito ha utenti

L'abilitazione della conferma dell'account in un sito con utenti blocca tutti gli utenti esistenti. Gli utenti esistenti sono bloccati perché i relativi account non sono confermati. Per aggirare il blocco utente esistente, utilizzare uno degli approcci seguenti:

* Aggiornare il database per contrassegnare tutti gli utenti esistenti come confermati.
* Verificare gli utenti esistenti. Ad esempio, invio in batch di messaggi di posta elettronica con collegamenti di conferma.

::: moniker-end
