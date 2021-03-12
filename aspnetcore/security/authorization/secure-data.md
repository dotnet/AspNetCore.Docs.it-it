---
title: Creare un'app ASP.NET Core con i dati utente protetti dall'autorizzazione
author: rick-anderson
description: Informazioni su come creare un'app Web di ASP.NET Core con i dati utente protetti dall'autorizzazione. Include HTTPS, autenticazione, sicurezza ASP.NET Core Identity .
ms.author: riande
ms.date: 7/18/2020
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
uid: security/authorization/secure-data
ms.openlocfilehash: 662456af59c453df66ca48139a6de40d0e2cbf0d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589192"
---
# <a name="create-an-aspnet-core-web-app-with-user-data-protected-by-authorization"></a>Creare un'app Web di ASP.NET Core con i dati utente protetti dall'autorizzazione

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Joe Audette](https://twitter.com/joeaudette)

::: moniker range="= aspnetcore-2.0"

Vedi [questo PDF](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_July16_18.pdf)

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

Questa esercitazione illustra come creare un'app Web di ASP.NET Core con i dati utente protetti dall'autorizzazione. Viene visualizzato un elenco di contatti creati dagli utenti autenticati (registrati). Sono disponibili tre gruppi di sicurezza:

* **Gli utenti registrati** possono visualizzare tutti i dati approvati e possono modificare/eliminare i propri dati.
* I **responsabili** possono approvare o rifiutare i dati di contatto. Solo i contatti approvati sono visibili agli utenti.
* Gli **amministratori** possono approvare/rifiutare e modificare/eliminare i dati.

Le immagini in questo documento non corrispondono esattamente ai modelli più recenti.

Nell'immagine seguente, l'utente Rick ( `rick@example.com` ) ha eseguito l'accesso. Rick può visualizzare solo i contatti approvati e **modificare** / **Delete** / **Crea nuovi** collegamenti per i contatti. Solo l'ultimo record, creato da Rick, Visualizza i collegamenti **modifica** ed **Elimina** . Gli altri utenti non vedranno l'ultimo record fino a quando un responsabile o un amministratore non modifica lo stato in "approvato".

![Screenshot che illustra l'accesso a Rick](secure-data/_static/rick.png)

Nell'immagine seguente, `manager@contoso.com` è stato eseguito l'accesso e nel ruolo del Manager:

![Screenshot che mostra l' manager@contoso.com accesso](secure-data/_static/manager1.png)

Nella figura seguente viene illustrata la visualizzazione dettagli Manager di un contatto:

![Visualizzazione del contatto da un responsabile](secure-data/_static/manager.png)

I pulsanti **approva** e **rifiuta** vengono visualizzati solo per Manager e amministratori.

Nell'immagine seguente, `admin@contoso.com` è stato eseguito l'accesso e il ruolo dell'amministratore:

![Screenshot che mostra l' admin@contoso.com accesso](secure-data/_static/admin.png)

L'amministratore dispone di tutti i privilegi. Può leggere, modificare ed eliminare qualsiasi contatto e modificare lo stato dei contatti.

L'app è stata creata mediante l' [impalcatura](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) del `Contact` modello seguente:

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

L'esempio contiene i gestori di autorizzazioni seguenti:

* `ContactIsOwnerAuthorizationHandler`: Assicura che un utente possa modificare solo i propri dati.
* `ContactManagerAuthorizationHandler`: Consente ai responsabili di approvare o rifiutare i contatti.
* `ContactAdministratorsAuthorizationHandler`: Consente agli amministratori di approvare o rifiutare contatti e di modificare/eliminare contatti.

## <a name="prerequisites"></a>Prerequisiti

Questa esercitazione è avanzata. È necessario avere familiarità con:

* [ASP.NET Core](xref:tutorials/first-mvc-app/start-mvc)
* [autenticazione](xref:security/authentication/identity)
* [Conferma account e recupero password](xref:security/authentication/accconfirm)
* [Autorizzazione](xref:security/authorization/introduction)
* [Entity Framework Core](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a>App iniziale e completata

[Scaricare](xref:index#how-to-download-a-sample) l'app [completata](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples) . [Testare](#test-the-completed-app) l'app completata in modo da acquisire familiarità con le funzionalità di sicurezza.

### <a name="the-starter-app"></a>App iniziale

[Scaricare](xref:index#how-to-download-a-sample) l'app [Starter](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/) .

Eseguire l'app, toccare il collegamento **ContactManager** e verificare che sia possibile creare, modificare ed eliminare un contatto. Per creare l'app iniziale, vedere [creare l'app iniziale](#create-the-starter-app).

## <a name="secure-user-data"></a>Proteggere i dati utente

Le sezioni seguenti includono tutti i passaggi principali per creare l'app Secure User Data. Può risultare utile fare riferimento al progetto completato.

### <a name="tie-the-contact-data-to-the-user"></a>Associare i dati di contatto all'utente

Usare l' [Identity](xref:security/authentication/identity) ID utente ASP.NET per assicurarsi che gli utenti possano modificare i dati, ma non i dati di altri utenti. Aggiungere `OwnerID` e `ContactStatus` al `Contact` modello:

[!code-csharp[](secure-data/samples/final3/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

`OwnerID` ID dell'utente della `AspNetUser` tabella nel [Identity](xref:security/authentication/identity) database. Il `Status` campo determina se un contatto è visualizzabile dagli utenti generali.

Creare una nuova migrazione e aggiornare il database:

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-identity"></a>Aggiungere servizi ruolo a Identity

Aggiungere [Aggiungi ruoli](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) per aggiungere servizi ruolo:

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet2&highlight=9)]

<a name="rau"></a>

### <a name="require-authenticated-users"></a>Richiedi utenti autenticati

Impostare i criteri di autenticazione di fallback per richiedere l'autenticazione degli utenti:

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet&highlight=13-99)]

Il codice evidenziato precedente imposta i [criteri di autenticazione di fallback](xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy). Per i criteri di autenticazione di fallback è necessario autenticare ***tutti*** gli utenti, ad eccezione di Razor pagine, controller o metodi di azione con un attributo di autenticazione. Ad esempio, Razor pagine, controller o metodi di azione con `[AllowAnonymous]` o `[Authorize(PolicyName="MyPolicy")]` utilizzano l'attributo di autenticazione applicato anziché i criteri di autenticazione di fallback.

<xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> aggiunge <xref:Microsoft.AspNetCore.Authorization.Infrastructure.DenyAnonymousAuthorizationRequirement> all'istanza corrente che impone che l'utente corrente sia autenticato.

I criteri di autenticazione di fallback:

* Viene applicato a tutte le richieste che non specificano in modo esplicito i criteri di autenticazione. Per le richieste gestite dal routing degli endpoint, questo includerebbe qualsiasi endpoint che non specifichi un attributo di autorizzazione. Per le richieste gestite da altri middleware dopo il middleware di autorizzazione, ad esempio [i file statici](xref:fundamentals/static-files), il criterio viene applicato a tutte le richieste.

L'impostazione dei criteri di autenticazione di fallback per richiedere l'autenticazione degli utenti consente di proteggere le Razor pagine e i controller appena aggiunti. La necessità di eseguire l'autenticazione per impostazione predefinita è più sicura rispetto a quella di fare affidamento su nuovi controller e Razor pagine per includere l' `[Authorize]` attributo.

La <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions> classe contiene anche <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.DefaultPolicy?displayProperty=nameWithType> . `DefaultPolicy`È il criterio utilizzato con l' `[Authorize]` attributo quando non è specificato alcun criterio. `[Authorize]` non contiene un criterio denominato, a differenza di `[Authorize(PolicyName="MyPolicy")]` .

Per ulteriori informazioni sui criteri, vedere <xref:security/authorization/policies> .

Un metodo alternativo per i controller e Razor le pagine MVC per richiedere che tutti gli utenti siano autenticati è l'aggiunta di un filtro di autorizzazione:

[!code-csharp[](secure-data/samples/final3/Startup2.cs?name=snippet&highlight=14-99)]

Il codice precedente usa un filtro di autorizzazione, l'impostazione dei criteri di fallback usa il routing degli endpoint. L'impostazione dei criteri di fallback è il modo migliore per richiedere l'autenticazione di tutti gli utenti.

Aggiungere [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) alle `Index` pagine e in `Privacy` modo che gli utenti anonimi possano ottenere informazioni sul sito prima della registrazione:

[!code-csharp[](secure-data/samples/final3/Pages/Index.cshtml.cs?highlight=1,7)]

### <a name="configure-the-test-account"></a>Configurare l'account di test

La `SeedData` classe crea due account: Administrator e Manager. Utilizzare lo [strumento Gestione segreta](xref:security/app-secrets) per impostare una password per questi account. Impostare la password dalla directory del progetto (la directory contenente *Program.cs*):

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

Se non si specifica una password complessa, viene generata un'eccezione quando `SeedData.Initialize` viene chiamato il metodo.

Aggiornare `Main` per usare la password di test:

[!code-csharp[](secure-data/samples/final3/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a>Creare gli account di test e aggiornare i contatti

Aggiornare il `Initialize` metodo nella `SeedData` classe per creare gli account di test:

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet_Initialize)]

Aggiungere l'ID utente amministratore e `ContactStatus` ai contatti. Creare uno dei contatti "inviato" e uno "rifiutato". Aggiungere l'ID utente e lo stato a tutti i contatti. Viene visualizzato un solo contatto:

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a>Creazione di gestori di autorizzazioni proprietario, Manager e amministratore

Creare una `ContactIsOwnerAuthorizationHandler` classe nella cartella *authorization* . `ContactIsOwnerAuthorizationHandler`Verifica che l'utente che agisce su una risorsa appartenga alla risorsa.

[!code-csharp[](secure-data/samples/final3/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

`ContactIsOwnerAuthorizationHandler`Contesto delle chiamate [. Ha esito positivo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_) se l'utente autenticato corrente è il proprietario del contatto. Gestori autorizzazioni in genere:

* Restituisce `context.Succeed` quando vengono soddisfatti i requisiti.
* Restituisce `Task.CompletedTask` quando i requisiti non vengono soddisfatti. `Task.CompletedTask` non ha esito positivo o negativo &mdash; consente l'esecuzione di altri gestori di autorizzazione.

Se è necessario eseguire in modo esplicito l'errore, restituire [context. Esito negativo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail).

L'app consente ai proprietari dei contatti di modificare/eliminare/creare i propri dati. `ContactIsOwnerAuthorizationHandler` non è necessario controllare l'operazione passata nel parametro requirement.

### <a name="create-a-manager-authorization-handler"></a>Creazione di un gestore autorizzazioni di gestione

Creare una `ContactManagerAuthorizationHandler` classe nella cartella *authorization* . `ContactManagerAuthorizationHandler`Verifica che l'utente che agisce sulla risorsa sia un responsabile. Solo i responsabili possono approvare o rifiutare le modifiche al contenuto (nuove o modificate).

[!code-csharp[](secure-data/samples/final3/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a>Creazione di un gestore autorizzazioni Amministratore

Creare una `ContactAdministratorsAuthorizationHandler` classe nella cartella *authorization* . Il `ContactAdministratorsAuthorizationHandler` Verifica che l'utente che agisce sulla risorsa sia un amministratore. L'amministratore può eseguire tutte le operazioni.

[!code-csharp[](secure-data/samples/final3/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a>Registrare i gestori di autorizzazioni

I servizi che usano Entity Framework Core devono essere registrati per l' [inserimento di dipendenze](xref:fundamentals/dependency-injection) usando [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions). `ContactIsOwnerAuthorizationHandler`Usa ASP.NET Core [Identity](xref:security/authentication/identity) , che è basato su Entity Framework Core. Registrare i gestori con la raccolta di servizi in modo che siano disponibili per `ContactsController` tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection). Aggiungere il codice seguente alla fine di `ConfigureServices` :

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet_defaultPolicy&highlight=23-99)]

`ContactAdministratorsAuthorizationHandler` e `ContactManagerAuthorizationHandler` vengono aggiunti come singleton. Si tratta di singleton perché non usano EF e tutte le informazioni necessarie si trovano nel `Context` parametro del `HandleRequirementAsync` metodo.

## <a name="support-authorization"></a>Autorizzazione supporto

In questa sezione si aggiornano le Razor pagine e si aggiunge una classe dei requisiti delle operazioni.

### <a name="review-the-contact-operations-requirements-class"></a>Esaminare la classe dei requisiti delle operazioni di contatto

Esaminare la `ContactOperations` classe. Questa classe contiene i requisiti supportati dall'app:

[!code-csharp[](secure-data/samples/final3/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-razor-pages"></a>Creare una classe di base per le Razor pagine contatti

Creare una classe di base che contiene i servizi utilizzati nelle Razor pagine contatti. La classe base inserisce il codice di inizializzazione in un'unica posizione:

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/DI_BasePageModel.cs)]

Il codice precedente:

* Aggiunge il `IAuthorizationService` servizio per accedere ai gestori di autorizzazione.
* Aggiunge il Identity `UserManager` servizio.
* Aggiungere l'oggetto `ApplicationDbContext`.

### <a name="update-the-createmodel"></a>Aggiornare CreateModel

Aggiornare il costruttore del modello di pagina create per usare la `DI_BasePageModel` classe di base:

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

Aggiornare il `CreateModel.OnPostAsync` metodo a:

* Aggiungere l'ID utente al `Contact` modello.
* Chiamare il gestore autorizzazioni per verificare che l'utente disponga dell'autorizzazione per la creazione di contatti.

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a>Aggiornare IndexModel

Aggiornare il `OnGetAsync` metodo in modo da visualizzare solo i contatti approvati per gli utenti generali:

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a>Aggiornare EditModel

Aggiungere un gestore autorizzazioni per verificare che l'utente sia proprietario del contatto. Poiché l'autorizzazione della risorsa viene convalidata, l' `[Authorize]` attributo non è sufficiente. L'app non ha accesso alla risorsa quando vengono valutati gli attributi. L'autorizzazione basata sulle risorse deve essere imperativa. I controlli devono essere eseguiti una volta che l'app può accedere alla risorsa, eseguendo il caricamento nel modello di pagina o caricando l'app all'interno del gestore stesso. Si accede spesso alla risorsa passando la chiave della risorsa.

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a>Aggiornare DeleteModel

Aggiornare il modello di pagina Elimina per utilizzare il gestore autorizzazione per verificare che l'utente disponga dell'autorizzazione DELETE per il contatto.

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a>Inserire il servizio di autorizzazione nelle viste

Attualmente, l'interfaccia utente Mostra i collegamenti modifica ed Elimina per i contatti che non possono essere modificati dall'utente.

Inserire il servizio di autorizzazione nel file *pages/_ViewImports. cshtml* in modo che sia disponibile per tutte le visualizzazioni:

[!code-cshtml[](secure-data/samples/final3/Pages/_ViewImports.cshtml?highlight=6-99)]

Il markup precedente aggiunge diverse `using` istruzioni.

Aggiornare i collegamenti **Edit** ed **Delete** in *pages/Contacts/index. cshtml* in modo che vengano sottoposti a rendering solo per gli utenti con le autorizzazioni appropriate:

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> Nascondere i collegamenti da utenti che non dispongono dell'autorizzazione per modificare i dati non protegge l'app. Nascondere i collegamenti rende l'app più intuitiva visualizzando solo i collegamenti validi. Gli utenti possono hackerare gli URL generati per richiamare le operazioni di modifica ed eliminazione sui dati di cui non sono proprietari. La Razor pagina o il controller deve applicare i controlli di accesso per proteggere i dati.

### <a name="update-details"></a>Dettagli aggiornamento

Aggiornare la visualizzazione dettagli in modo che i responsabili possano approvare o rifiutare i contatti:

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Details.cshtml?name=snippet)]

Aggiornare il modello della pagina dei dettagli:

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a>Aggiungere o rimuovere un utente in un ruolo

Per informazioni su, vedere [questo problema](https://github.com/dotnet/AspNetCore.Docs/issues/8502) :

* Rimozione dei privilegi da un utente. Ad esempio, disattivare un utente in un'app di chat.
* Aggiunta di privilegi a un utente.

<a name="challenge"></a>

## <a name="differences-between-challenge-and-forbid"></a>Differenze tra la sfida e la proibizione

Questa app imposta i criteri predefiniti per [richiedere l'autenticazione degli utenti](#rau). Il codice seguente consente agli utenti anonimi. Gli utenti anonimi possono mostrare le differenze tra la richiesta di confronto e la proibizione.

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details2.cshtml.cs?name=snippet)]

Nel codice precedente:

* Quando l'utente **non** è autenticato, `ChallengeResult` viene restituito un oggetto. Quando `ChallengeResult` viene restituito un oggetto, l'utente viene reindirizzato alla pagina di accesso.
* Quando l'utente viene autenticato, ma non autorizzato, `ForbidResult` viene restituito un oggetto. Quando `ForbidResult` viene restituito un oggetto, l'utente viene reindirizzato alla pagina di accesso negato.

## <a name="test-the-completed-app"></a>Testare l'app completata

Se non è già stata impostata una password per gli account utente di cui è stato eseguito il seeding, usare lo [strumento Gestione segreta](xref:security/app-secrets#secret-manager) per impostare una password:

* Scegliere una password complessa: usare otto o più caratteri e almeno un carattere maiuscolo, un numero e un simbolo. Ad esempio, `Passw0rd!` soddisfa i requisiti per le password complesse.
* Eseguire il comando seguente dalla cartella del progetto, dove `<PW>` è la password:

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

Se l'app dispone di contatti:

* Eliminare tutti i record nella `Contact` tabella.
* Riavviare l'app per inizializzare il database.

Un modo semplice per testare l'app completata consiste nell'avviare tre diversi browser (o sessioni in incognito/InPrivate). In un browser registrare un nuovo utente (ad esempio, `test@example.com` ). Accedere a ogni browser con un altro utente. Verificare le operazioni seguenti:

* Gli utenti registrati possono visualizzare tutti i dati di contatto approvati.
* Gli utenti registrati possono modificare/eliminare i propri dati.
* I responsabili possono approvare/rifiutare i dati di contatto. La `Details` visualizzazione Mostra i pulsanti **approva** e **rifiuta** .
* Gli amministratori possono approvare/rifiutare e modificare/eliminare tutti i dati.

| User                | Seeding dall'app | Opzioni                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | No                | Modificare/eliminare i dati personali.                |
| manager@contoso.com | Sì               | Approva/rifiuta e modifica/elimina i dati personali. |
| admin@contoso.com   | Sì               | Approva/rifiuta e modifica/elimina tutti i dati. |

Creare un contatto nel browser dell'amministratore. Copiare l'URL da eliminare e modificare dal contatto amministratore. Incollare questi collegamenti nel browser dell'utente test per verificare che l'utente test non possa eseguire queste operazioni.

## <a name="create-the-starter-app"></a>Creare l'app Starter

* Creare un' Razor app per le pagine denominata "ContactManager"
  * Creare l'app con **singoli account utente**.
  * Denominarlo "ContactManager" in modo che lo spazio dei nomi corrisponda allo spazio dei nomi usato nell'esempio.
  * `-uld` Specifica il database locale anziché SQLite

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* Aggiungi *modelli/Contact. cs*:

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* Impalcatura del `Contact` modello.
* Creare la migrazione iniziale e aggiornare il database:

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
dotnet ef database drop -f
dotnet ef migrations add initial
dotnet ef database update
```

Se si verifica un bug con il `dotnet aspnet-codegenerator razorpage` comando, vedere [questo problema di GitHub](https://github.com/aspnet/Scaffolding/issues/984).

* Aggiornare l'ancoraggio **ContactManager** nel file *pages/Shared/_Layout. cshtml* :

 ```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Contacts/Index">ContactManager</a>
  ```

* Testare l'app creando, modificando ed eliminando un contatto

### <a name="seed-the-database"></a>Specificare il valore di inizializzazione del database

Aggiungere la classe [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/starter3/Data/SeedData.cs) alla cartella *Data* :

[!code-csharp[](secure-data/samples/starter3/Data/SeedData.cs)]

Chiama `SeedData.Initialize` da `Main` :

[!code-csharp[](secure-data/samples/starter3/Program.cs)]

Verificare che l'app abbia sottoposta a seeding il database. Se sono presenti righe nel database Contact, il metodo seed non viene eseguito.

::: moniker-end

::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

Questa esercitazione illustra come creare un'app Web di ASP.NET Core con i dati utente protetti dall'autorizzazione. Viene visualizzato un elenco di contatti creati dagli utenti autenticati (registrati). Sono disponibili tre gruppi di sicurezza:

* **Gli utenti registrati** possono visualizzare tutti i dati approvati e possono modificare/eliminare i propri dati.
* I **responsabili** possono approvare o rifiutare i dati di contatto. Solo i contatti approvati sono visibili agli utenti.
* Gli **amministratori** possono approvare/rifiutare e modificare/eliminare i dati.

Nell'immagine seguente, l'utente Rick ( `rick@example.com` ) ha eseguito l'accesso. Rick può visualizzare solo i contatti approvati e **modificare** / **Delete** / **Crea nuovi** collegamenti per i contatti. Solo l'ultimo record, creato da Rick, Visualizza i collegamenti **modifica** ed **Elimina** . Gli altri utenti non vedranno l'ultimo record fino a quando un responsabile o un amministratore non modifica lo stato in "approvato".

![Screenshot che illustra l'accesso a Rick](secure-data/_static/rick.png)

Nell'immagine seguente, `manager@contoso.com` è stato eseguito l'accesso e nel ruolo del Manager:

![Screenshot che mostra l' manager@contoso.com accesso](secure-data/_static/manager1.png)

Nella figura seguente viene illustrata la visualizzazione dettagli Manager di un contatto:

![Visualizzazione del contatto da un responsabile](secure-data/_static/manager.png)

I pulsanti **approva** e **rifiuta** vengono visualizzati solo per Manager e amministratori.

Nell'immagine seguente, `admin@contoso.com` è stato eseguito l'accesso e il ruolo dell'amministratore:

![Screenshot che mostra l' admin@contoso.com accesso](secure-data/_static/admin.png)

L'amministratore dispone di tutti i privilegi. Può leggere, modificare ed eliminare qualsiasi contatto e modificare lo stato dei contatti.

L'app è stata creata mediante l' [impalcatura](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) del `Contact` modello seguente:

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

L'esempio contiene i gestori di autorizzazioni seguenti:

* `ContactIsOwnerAuthorizationHandler`: Assicura che un utente possa modificare solo i propri dati.
* `ContactManagerAuthorizationHandler`: Consente ai responsabili di approvare o rifiutare i contatti.
* `ContactAdministratorsAuthorizationHandler`: Consente agli amministratori di approvare o rifiutare contatti e di modificare/eliminare contatti.

## <a name="prerequisites"></a>Prerequisiti

Questa esercitazione è avanzata. È necessario avere familiarità con:

* [ASP.NET Core](xref:tutorials/first-mvc-app/start-mvc)
* [autenticazione](xref:security/authentication/identity)
* [Conferma account e recupero password](xref:security/authentication/accconfirm)
* [Autorizzazione](xref:security/authorization/introduction)
* [Entity Framework Core](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a>App iniziale e completata

[Scaricare](xref:index#how-to-download-a-sample) l'app [completata](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples) . [Testare](#test-the-completed-app) l'app completata in modo da acquisire familiarità con le funzionalità di sicurezza.

### <a name="the-starter-app"></a>App iniziale

[Scaricare](xref:index#how-to-download-a-sample) l'app [Starter](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/) .

Eseguire l'app, toccare il collegamento **ContactManager** e verificare che sia possibile creare, modificare ed eliminare un contatto.

## <a name="secure-user-data"></a>Proteggere i dati utente

Le sezioni seguenti includono tutti i passaggi principali per creare l'app Secure User Data. Può risultare utile fare riferimento al progetto completato.

### <a name="tie-the-contact-data-to-the-user"></a>Associare i dati di contatto all'utente

Usare l' [Identity](xref:security/authentication/identity) ID utente ASP.NET per assicurarsi che gli utenti possano modificare i dati, ma non i dati di altri utenti. Aggiungere `OwnerID` e `ContactStatus` al `Contact` modello:

[!code-csharp[](secure-data/samples/final2.1/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

`OwnerID` ID dell'utente della `AspNetUser` tabella nel [Identity](xref:security/authentication/identity) database. Il `Status` campo determina se un contatto è visualizzabile dagli utenti generali.

Creare una nuova migrazione e aggiornare il database:

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-identity"></a>Aggiungere servizi ruolo a Identity

Aggiungere [Aggiungi ruoli](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) per aggiungere servizi ruolo:

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet2&highlight=11)]

### <a name="require-authenticated-users"></a>Richiedi utenti autenticati

Impostare i criteri di autenticazione predefiniti per richiedere l'autenticazione degli utenti:

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet&highlight=17-99)] 

 È possibile rifiutare esplicitamente l'autenticazione a Razor livello di pagina, controller o metodo di azione con l' `[AllowAnonymous]` attributo. L'impostazione dei criteri di autenticazione predefiniti per richiedere l'autenticazione degli utenti consente di proteggere le Razor pagine e i controller appena aggiunti. La necessità di eseguire l'autenticazione per impostazione predefinita è più sicura rispetto a quella di fare affidamento su nuovi controller e Razor pagine per includere l' `[Authorize]` attributo.

Aggiungere [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) alle pagine di indice, informazioni e contatti in modo che gli utenti anonimi possano ottenere informazioni sul sito prima della registrazione.

[!code-csharp[](secure-data/samples/final2.1/Pages/Index.cshtml.cs?highlight=1,6)]

### <a name="configure-the-test-account"></a>Configurare l'account di test

La `SeedData` classe crea due account: Administrator e Manager. Utilizzare lo [strumento Gestione segreta](xref:security/app-secrets) per impostare una password per questi account. Impostare la password dalla directory del progetto (la directory contenente *Program.cs*):

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

Se non si specifica una password complessa, viene generata un'eccezione quando `SeedData.Initialize` viene chiamato il metodo.

Aggiornare `Main` per usare la password di test:

[!code-csharp[](secure-data/samples/final2.1/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a>Creare gli account di test e aggiornare i contatti

Aggiornare il `Initialize` metodo nella `SeedData` classe per creare gli account di test:

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet_Initialize)]

Aggiungere l'ID utente amministratore e `ContactStatus` ai contatti. Creare uno dei contatti "inviato" e uno "rifiutato". Aggiungere l'ID utente e lo stato a tutti i contatti. Viene visualizzato un solo contatto:

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a>Creazione di gestori di autorizzazioni proprietario, Manager e amministratore

Creare una cartella di *autorizzazione* e crearvi una `ContactIsOwnerAuthorizationHandler` classe. `ContactIsOwnerAuthorizationHandler`Verifica che l'utente che agisce su una risorsa appartenga alla risorsa.

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

`ContactIsOwnerAuthorizationHandler`Contesto delle chiamate [. Ha esito positivo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_) se l'utente autenticato corrente è il proprietario del contatto. Gestori autorizzazioni in genere:

* Restituisce `context.Succeed` quando vengono soddisfatti i requisiti.
* Restituisce `Task.CompletedTask` quando i requisiti non vengono soddisfatti. `Task.CompletedTask` non ha esito positivo o negativo &mdash; consente l'esecuzione di altri gestori di autorizzazione.

Se è necessario eseguire in modo esplicito l'errore, restituire [context. Esito negativo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail).

L'app consente ai proprietari dei contatti di modificare/eliminare/creare i propri dati. `ContactIsOwnerAuthorizationHandler` non è necessario controllare l'operazione passata nel parametro requirement.

### <a name="create-a-manager-authorization-handler"></a>Creazione di un gestore autorizzazioni di gestione

Creare una `ContactManagerAuthorizationHandler` classe nella cartella *authorization* . `ContactManagerAuthorizationHandler`Verifica che l'utente che agisce sulla risorsa sia un responsabile. Solo i responsabili possono approvare o rifiutare le modifiche al contenuto (nuove o modificate).

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a>Creazione di un gestore autorizzazioni Amministratore

Creare una `ContactAdministratorsAuthorizationHandler` classe nella cartella *authorization* . Il `ContactAdministratorsAuthorizationHandler` Verifica che l'utente che agisce sulla risorsa sia un amministratore. L'amministratore può eseguire tutte le operazioni.

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a>Registrare i gestori di autorizzazioni

I servizi che usano Entity Framework Core devono essere registrati per l' [inserimento di dipendenze](xref:fundamentals/dependency-injection) usando [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions). `ContactIsOwnerAuthorizationHandler`Usa ASP.NET Core [Identity](xref:security/authentication/identity) , che è basato su Entity Framework Core. Registrare i gestori con la raccolta di servizi in modo che siano disponibili per `ContactsController` tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection). Aggiungere il codice seguente alla fine di `ConfigureServices` :

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet_defaultPolicy&highlight=27-99)]

`ContactAdministratorsAuthorizationHandler` e `ContactManagerAuthorizationHandler` vengono aggiunti come singleton. Si tratta di singleton perché non usano EF e tutte le informazioni necessarie si trovano nel `Context` parametro del `HandleRequirementAsync` metodo.

## <a name="support-authorization"></a>Autorizzazione supporto

In questa sezione si aggiornano le Razor pagine e si aggiunge una classe dei requisiti delle operazioni.

### <a name="review-the-contact-operations-requirements-class"></a>Esaminare la classe dei requisiti delle operazioni di contatto

Esaminare la `ContactOperations` classe. Questa classe contiene i requisiti supportati dall'app:

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-razor-pages"></a>Creare una classe di base per le Razor pagine contatti

Creare una classe di base che contiene i servizi utilizzati nelle Razor pagine contatti. La classe base inserisce il codice di inizializzazione in un'unica posizione:

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/DI_BasePageModel.cs)]

Il codice precedente:

* Aggiunge il `IAuthorizationService` servizio per accedere ai gestori di autorizzazione.
* Aggiunge il Identity `UserManager` servizio.
* Aggiungere l'oggetto `ApplicationDbContext`.

### <a name="update-the-createmodel"></a>Aggiornare CreateModel

Aggiornare il costruttore del modello di pagina create per usare la `DI_BasePageModel` classe di base:

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

Aggiornare il `CreateModel.OnPostAsync` metodo a:

* Aggiungere l'ID utente al `Contact` modello.
* Chiamare il gestore autorizzazioni per verificare che l'utente disponga dell'autorizzazione per la creazione di contatti.

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a>Aggiornare IndexModel

Aggiornare il `OnGetAsync` metodo in modo da visualizzare solo i contatti approvati per gli utenti generali:

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a>Aggiornare EditModel

Aggiungere un gestore autorizzazioni per verificare che l'utente sia proprietario del contatto. Poiché l'autorizzazione della risorsa viene convalidata, l' `[Authorize]` attributo non è sufficiente. L'app non ha accesso alla risorsa quando vengono valutati gli attributi. L'autorizzazione basata sulle risorse deve essere imperativa. I controlli devono essere eseguiti una volta che l'app può accedere alla risorsa, eseguendo il caricamento nel modello di pagina o caricando l'app all'interno del gestore stesso. Si accede spesso alla risorsa passando la chiave della risorsa.

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a>Aggiornare DeleteModel

Aggiornare il modello di pagina Elimina per utilizzare il gestore autorizzazione per verificare che l'utente disponga dell'autorizzazione DELETE per il contatto.

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a>Inserire il servizio di autorizzazione nelle viste

Attualmente, l'interfaccia utente Mostra i collegamenti modifica ed Elimina per i contatti che non possono essere modificati dall'utente.

Inserire il servizio di autorizzazione nel file *views/_ViewImports. cshtml* in modo che sia disponibile per tutte le visualizzazioni:

[!code-cshtml[](secure-data/samples/final2.1/Pages/_ViewImports.cshtml?highlight=6-99)]

Il markup precedente aggiunge diverse `using` istruzioni.

Aggiornare i collegamenti **Edit** ed **Delete** in *pages/Contacts/index. cshtml* in modo che vengano sottoposti a rendering solo per gli utenti con le autorizzazioni appropriate:

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> Nascondere i collegamenti da utenti che non dispongono dell'autorizzazione per modificare i dati non protegge l'app. Nascondere i collegamenti rende l'app più intuitiva visualizzando solo i collegamenti validi. Gli utenti possono hackerare gli URL generati per richiamare le operazioni di modifica ed eliminazione sui dati di cui non sono proprietari. La Razor pagina o il controller deve applicare i controlli di accesso per proteggere i dati.

### <a name="update-details"></a>Dettagli aggiornamento

Aggiornare la visualizzazione dettagli in modo che i responsabili possano approvare o rifiutare i contatti:

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml?name=snippet)]

Aggiornare il modello della pagina dei dettagli:

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a>Aggiungere o rimuovere un utente in un ruolo

Per informazioni su, vedere [questo problema](https://github.com/dotnet/AspNetCore.Docs/issues/8502) :

* Rimozione dei privilegi da un utente. Ad esempio, disattivare un utente in un'app di chat.
* Aggiunta di privilegi a un utente.

## <a name="test-the-completed-app"></a>Testare l'app completata

Se non è già stata impostata una password per gli account utente di cui è stato eseguito il seeding, usare lo [strumento Gestione segreta](xref:security/app-secrets#secret-manager) per impostare una password:

* Scegliere una password complessa: usare otto o più caratteri e almeno un carattere maiuscolo, un numero e un simbolo. Ad esempio, `Passw0rd!` soddisfa i requisiti per le password complesse.
* Eseguire il comando seguente dalla cartella del progetto, dove `<PW>` è la password:

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

* Eliminare e aggiornare il database

  ```dotnetcli
  dotnet ef database drop -f
  dotnet ef database update  
  ```

* Riavviare l'app per inizializzare il database.

Un modo semplice per testare l'app completata consiste nell'avviare tre diversi browser (o sessioni in incognito/InPrivate). In un browser registrare un nuovo utente (ad esempio, `test@example.com` ). Accedere a ogni browser con un altro utente. Verificare le operazioni seguenti:

* Gli utenti registrati possono visualizzare tutti i dati di contatto approvati.
* Gli utenti registrati possono modificare/eliminare i propri dati.
* I responsabili possono approvare/rifiutare i dati di contatto. La `Details` visualizzazione Mostra i pulsanti **approva** e **rifiuta** .
* Gli amministratori possono approvare/rifiutare e modificare/eliminare tutti i dati.

| User                | Seeding dall'app | Opzioni                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | No                | Modificare/eliminare i dati personali.                |
| manager@contoso.com | Sì               | Approva/rifiuta e modifica/elimina i dati personali. |
| admin@contoso.com   | Sì               | Approva/rifiuta e modifica/elimina tutti i dati. |

Creare un contatto nel browser dell'amministratore. Copiare l'URL da eliminare e modificare dal contatto amministratore. Incollare questi collegamenti nel browser dell'utente test per verificare che l'utente test non possa eseguire queste operazioni.

## <a name="create-the-starter-app"></a>Creare l'app Starter

* Creare un' Razor app per le pagine denominata "ContactManager"
  * Creare l'app con **singoli account utente**.
  * Denominarlo "ContactManager" in modo che lo spazio dei nomi corrisponda allo spazio dei nomi usato nell'esempio.
  * `-uld` Specifica il database locale anziché SQLite

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* Aggiungi *modelli/Contact. cs*:

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* Impalcatura del `Contact` modello.
* Creare la migrazione iniziale e aggiornare il database:

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
  dotnet ef database drop -f
  dotnet ef migrations add initial
  dotnet ef database update
  ```

* Aggiornare l'ancoraggio **ContactManager** nel file *pages/_Layout. cshtml* :

  ```cshtml
  <a asp-page="/Contacts/Index" class="navbar-brand">ContactManager</a>
  ```

* Testare l'app creando, modificando ed eliminando un contatto

### <a name="seed-the-database"></a>Specificare il valore di inizializzazione del database

Aggiungere la classe [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/starter2.1/Data/SeedData.cs) alla cartella *Data* .

Chiama `SeedData.Initialize` da `Main` :

[!code-csharp[](secure-data/samples/starter2.1/Program.cs?name=snippet)]

Verificare che l'app abbia sottoposta a seeding il database. Se sono presenti righe nel database Contact, il metodo seed non viene eseguito.

::: moniker-end

<a name="secure-data-add-resources-label"></a>

### <a name="additional-resources"></a>Risorse aggiuntive

* [Creare un'app Web .NET Core e database SQL nel Servizio app di Azure](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)
* [ASP.NET Core Lab di autorizzazione](https://github.com/blowdart/AspNetAuthorizationWorkshop). Questo Lab illustra in modo più dettagliato le funzionalità di sicurezza introdotte in questa esercitazione.
* <xref:security/authorization/introduction>
* [Autorizzazione personalizzata basata su criteri](xref:security/authorization/policies)
