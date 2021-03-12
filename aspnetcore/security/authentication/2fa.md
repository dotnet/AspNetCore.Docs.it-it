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
# <a name="two-factor-authentication-with-sms-in-aspnet-core"></a>Autenticazione a due fattori con SMS in ASP.NET Core

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Swiss-sviluppatori](https://github.com/Swiss-Devs)

>[!WARNING]
> Le app di autenticazione a due fattori (2FA), che usano un algoritmo di password monouso basato sul tempo (TOTP), sono l'approccio consigliato per 2FA. 2FA con TOTP è preferibile a SMS 2FA. Per altre informazioni, vedere [abilitare la generazione di codice a matrice per le app TOTP Authenticator in ASP.NET Core](xref:security/authentication/identity-enable-qrcodes) per ASP.NET Core 2,0 e versioni successive.

Questa esercitazione illustra come configurare l'autenticazione a due fattori (2FA) tramite SMS. Vengono fornite istruzioni per [Twilio](https://www.twilio.com/) e [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/), ma è possibile usare qualsiasi altro provider SMS. Prima di iniziare questa esercitazione, è consigliabile completare la [conferma dell'account e il recupero della password](xref:security/authentication/accconfirm) .

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/2fa/sample/Web2FA). [Come eseguire il download](xref:index#how-to-download-a-sample).

## <a name="create-a-new-aspnet-core-project"></a>Creare un nuovo progetto ASP.NET Core

Creare una nuova app Web di ASP.NET Core denominata `Web2FA` con singoli account utente. Seguire le istruzioni in <xref:security/enforcing-ssl> per configurare e richiedere HTTPS.

### <a name="create-an-sms-account"></a>Creare un account SMS

Creare un account SMS, ad esempio, da [Twilio](https://www.twilio.com/) o [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/). Registrare le credenziali di autenticazione (per Twilio: accountSid e authToken, per ASPSMS: userKey e password).

#### <a name="figuring-out-sms-provider-credentials"></a>Individuazione delle credenziali del provider SMS

**Twilio**

Dalla scheda Dashboard dell'account Twilio copiare il **SID dell'account** e il **token di autenticazione**.

**ASPSMS:**

Dalle impostazioni dell'account passare a **userKey** e copiarlo insieme alla **password**.

Questi valori vengono archiviati in un secondo momento con lo strumento di gestione del segreto all'interno delle chiavi `SMSAccountIdentification` e `SMSAccountPassword` .

#### <a name="specifying-senderid--originator"></a>Specifica di SenderID/originator

**Twilio:** Dalla scheda numeri copiare il **numero di telefono** Twilio.

**ASPSMS:** Nel menu Sblocca autori sbloccare uno o più autori o scegliere un originatore alfanumerico (non supportato da tutte le reti).

Questo valore verrà archiviato in un secondo momento con lo strumento di gestione del segreto all'interno della chiave `SMSAccountFrom` .

### <a name="provide-credentials-for-the-sms-service"></a>Fornire le credenziali per il servizio SMS

Per accedere all'account utente e alle impostazioni della chiave verrà usato il [modello di opzioni](xref:fundamentals/configuration/options) .

* Creare una classe per recuperare la chiave SMS sicura. Per questo esempio, la `SMSoptions` classe viene creata nel file *Services/SMSoptions. cs* .

[!code-csharp[](2fa/sample/Web2FA/Services/SMSoptions.cs)]

Impostare `SMSAccountIdentification` `SMSAccountPassword` e `SMSAccountFrom` con lo strumento di [gestione dei segreti](xref:security/app-secrets). Ad esempio:

```none
C:/Web2FA/src/WebApp1>dotnet user-secrets set SMSAccountIdentification 12345
info: Successfully saved SMSAccountIdentification = 12345 to the secret store.
```

* Aggiungere il pacchetto NuGet per il provider SMS. Dalla console di gestione pacchetti (PMC) eseguire:

**Twilio**

`Install-Package Twilio`

**ASPSMS:**

`Install-Package ASPSMS`

* Aggiungere il codice nel file *Services/MessageServices. cs* per abilitare SMS. Usare la sezione Twilio o ASPSMS:

**Twilio**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_twilio.cs)]

**ASPSMS:**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_ASPSMS.cs)]

### <a name="configure-startup-to-use-smsoptions"></a>Configurare l'avvio per l'uso `SMSoptions`

Aggiungere `SMSoptions` al contenitore dei servizi nel `ConfigureServices` metodo in *Startup.cs*:

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet1&highlight=4)]

### <a name="enable-two-factor-authentication"></a>Abilitare l'autenticazione a due fattori

Aprire il file di visualizzazione *views/Manage/index. cshtml* Razor e rimuovere i caratteri di commento, in modo che nessun markup sia impostato come commento.

## <a name="log-in-with-two-factor-authentication"></a>Eseguire l'accesso con l'autenticazione a due fattori

* Eseguire l'app e registrare un nuovo utente

![Visualizzazione di registrazione dell'applicazione Web aperta in Microsoft Edge](2fa/_static/login2fa1.png)

* Toccare il nome utente, che attiva il `Index` metodo di azione in Gestisci controller. Toccare quindi il collegamento **Aggiungi** numero di telefono.

![Gestisci visualizzazione: toccare il collegamento "Aggiungi"](2fa/_static/login2fa2.png)

* Aggiungere un numero di telefono che riceverà il codice di verifica e toccare **Invia codice di verifica**.

![Pagina Aggiungi numero di telefono](2fa/_static/login2fa3.png)

* Viene ricevuto un messaggio di testo con il codice di verifica. Immettilo e tocca **Invia**

![Pagina Verifica numero di telefono](2fa/_static/login2fa4.png)

Se non viene visualizzato un messaggio di testo, vedere la pagina del log Twilio.

* La visualizzazione gestione mostra che il numero di telefono è stato aggiunto correttamente.

![Gestisci visualizzazione-numero di telefono aggiunto correttamente](2fa/_static/login2fa5.png)

* Toccare **Abilita** per abilitare l'autenticazione a due fattori.

![Gestisci visualizzazione: Abilita autenticazione a due fattori](2fa/_static/login2fa6.png)

### <a name="test-two-factor-authentication"></a>Testare l'autenticazione a due fattori

* Disconnettersi.

* Eseguire l'accesso.

* Per l'account utente è stata abilitata l'autenticazione a due fattori, pertanto è necessario fornire il secondo fattore di autenticazione. In questa esercitazione è stata abilitata la verifica telefonica. I modelli predefiniti consentono inoltre di configurare la posta elettronica come secondo fattore. È possibile configurare altri secondi fattori per l'autenticazione, ad esempio i codici QR. Toccare **Invia**.

![Invia visualizzazione codice di verifica](2fa/_static/login2fa7.png)

* Immettere il codice ricevuto nel messaggio SMS.

* Quando si fa clic sulla casella di controllo **memorizza questo browser** , non è necessario usare 2FA per accedere quando si usa lo stesso dispositivo e browser. Abilitazione di 2FA e facendo clic su **ricorda questo browser** fornirà una protezione 2FA avanzata da utenti malintenzionati che tentano di accedere all'account, purché non dispongano dell'accesso al dispositivo. Questa operazione può essere eseguita su qualsiasi dispositivo privato usato regolarmente. Impostando  **ricorda questo browser**, si ottiene la maggiore sicurezza di 2FA dai dispositivi che non vengono usati regolarmente e si ottiene la convenienza di non dover passare attraverso 2FA nei propri dispositivi.

![Verifica visualizzazione](2fa/_static/login2fa8.png)

## <a name="account-lockout-for-protecting-against-brute-force-attacks"></a>Blocco degli account per la protezione da attacchi di forza bruta

Il blocco degli account è consigliato con 2FA. Quando un utente accede tramite un account locale o un account di social networking, viene archiviato ogni tentativo non riuscito in 2FA. Se viene raggiunto il numero massimo di tentativi di accesso non riusciti, l'utente viene bloccato (impostazione predefinita: blocco di 5 minuti dopo 5 tentativi di accesso non riusciti). Una corretta autenticazione Reimposta il numero di tentativi di accesso non riusciti e reimposta il clock. Il numero massimo di tentativi di accesso non riusciti e l'ora di blocco possono essere impostati con [MaxFailedAccessAttempts](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.maxfailedaccessattempts) e [DefaultLockoutTimeSpan](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.defaultlockouttimespan). Di seguito viene configurato il blocco degli account per 10 minuti dopo 10 tentativi di accesso non riusciti:

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet2&highlight=13-17)]

Confermare che [PasswordSignInAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.passwordsigninasync) imposta `lockoutOnFailure` su `true` :

```csharp
var result = await _signInManager.PasswordSignInAsync(
                 Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: true);
```
