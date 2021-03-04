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
# <a name="google-external-login-setup-in-aspnet-core"></a>Installazione di Google External login in ASP.NET Core

Da [Valeriy Novytskyy](https://github.com/01binary) e [Rick Anderson](https://twitter.com/RickAndMSFT)

Questa esercitazione illustra come consentire agli utenti di accedere con il proprio account Google usando il progetto ASP.NET Core 3,0 creato nella [pagina precedente](xref:security/authentication/social/index).

## <a name="create-a-google-api-console-project-and-client-id"></a>Creare un progetto console e un ID client di Google API

* Aggiungere il pacchetto [Microsoft. AspNetCore. Authentication. Google](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Google) NuGet all'app.
* Seguire le istruzioni riportate in [integrazione di google Sign-In nell'app Web](https://developers.google.com/identity/sign-in/web/sign-in) (documentazione di Google).
* Nella pagina **credenziali** di [Google console](https://console.developers.google.com/apis/credentials)Selezionare **create Credentials**  >  **OAuth client ID**.
* Nella finestra di dialogo **tipo di applicazione** selezionare **applicazione Web**. Specificare un **nome** per l'app.
* Nella sezione **URI di reindirizzamento autorizzati** selezionare **Aggiungi URI** per impostare l'URI di reindirizzamento. URI di reindirizzamento di esempio: `https://localhost:{PORT}/signin-google` , dove il `{PORT}` segnaposto è la porta dell'app.
* Selezionare il pulsante **Crea** .
* Salvare l' **ID client** e il **segreto client** per l'uso nella configurazione dell'app.
* Quando si distribuisce il sito, è possibile:
  * Aggiornare l'URI di reindirizzamento dell'app nella **console di Google** all'URI di reindirizzamento distribuito dell'app.
  * Creare una nuova registrazione dell'API Google nella **console di Google** per l'app di produzione con l'URI di reindirizzamento di produzione.

## <a name="store-the-google-client-id-and-secret"></a>Archiviare l'ID e il segreto del client Google

Archiviare le impostazioni riservate, ad esempio i valori di ID client e segreto di Google con [gestione Secret](xref:security/app-secrets). Per questo esempio, attenersi alla procedura seguente:

1. Inizializzare il progetto per l'archiviazione segreta in base alle istruzioni riportate in [abilitare l'archiviazione segreta](xref:security/app-secrets#enable-secret-storage).
1. Archiviare le impostazioni sensibili nell'archivio dei segreti locali con le chiavi segrete `Authentication:Google:ClientId` e `Authentication:Google:ClientSecret` :

    ```dotnetcli
    dotnet user-secrets set "Authentication:Google:ClientId" "<client-id>"
    dotnet user-secrets set "Authentication:Google:ClientSecret" "<client-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

È possibile gestire le credenziali e l'utilizzo dell'API nella [console API](https://console.developers.google.com/apis/dashboard).

## <a name="configure-google-authentication"></a>Configurare l'autenticazione di Google

Aggiungere il servizio Google a `Startup.ConfigureServices` :

[!code-csharp[](~/security/authentication/social/social-code/3.x/StartupGoogle3x.cs?highlight=11-19)]

[!INCLUDE [default settings configuration](includes/default-settings2-2.md)]

## <a name="sign-in-with-google"></a>Accedere con Google

* Eseguire l'app e fare clic su **Accedi**. Viene visualizzata un'opzione per accedere con Google.
* Fare clic sul pulsante **Google** , che reindirizza a Google per l'autenticazione.
* Dopo aver immesso le credenziali Google, viene reindirizzato di nuovo al sito Web.

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

<xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>Per altre informazioni sulle opzioni di configurazione supportate da Google Authentication, vedere le informazioni di riferimento sulle API. Questa operazione può essere utilizzata per richiedere informazioni diverse sull'utente.

## <a name="change-the-default-callback-uri"></a>Modificare l'URI di callback predefinito

Il segmento URI `/signin-google` viene impostato come callback predefinito del provider di autenticazione Google. È possibile modificare l'URI di callback predefinito durante la configurazione del middleware di autenticazione di Google tramite la <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CallbackPath?displayProperty=nameWithType> proprietà ereditato della <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> classe.

## <a name="troubleshooting"></a>Risoluzione dei problemi

* Se l'accesso non funziona e non si ricevono errori, passare alla modalità di sviluppo per semplificare il debug del problema.
* Se Identity non è configurato chiamando `services.AddIdentity` in `ConfigureServices` , il tentativo di autenticare i risultati in *ArgumentException: è necessario specificare l'opzione ' SignInScheme '*. Il modello di progetto usato in questa esercitazione garantisce che questa operazione venga eseguita.
* Se il database del sito non è stato creato applicando la migrazione iniziale, si ottiene *un'operazione di database non riuscita durante l'elaborazione dell'* errore di richiesta. Selezionare **applica migrazioni** per creare il database e aggiornare la pagina per continuare a superare l'errore.

## <a name="next-steps"></a>Passaggi successivi

* Questo articolo ha illustrato come è possibile eseguire l'autenticazione con Google. È possibile seguire un approccio simile per l'autenticazione con altri provider elencati nella [pagina precedente](xref:security/authentication/social/index).
* Dopo aver pubblicato l'app in Azure, reimpostare la `ClientSecret` nella console dell'API Google.
* Impostare `Authentication:Google:ClientId` e `Authentication:Google:ClientSecret` come impostazioni dell'applicazione nella portale di Azure. Il sistema di configurazione è configurato per la lettura delle chiavi dalle variabili di ambiente.
