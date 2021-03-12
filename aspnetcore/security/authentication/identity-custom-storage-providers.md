---
title: Provider di archiviazione personalizzati per ASP.NET Core Identity
author: ardalis
description: Informazioni su come configurare i provider di archiviazione personalizzati per ASP.NET Core Identity .
ms.author: riande
ms.custom: mvc
ms.date: 07/23/2019
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
uid: security/authentication/identity-custom-storage-providers
ms.openlocfilehash: f1c4366e4e4afa3dd86a816a649ad0a8b2ce817b
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588627"
---
# <a name="custom-storage-providers-for-aspnet-core-identity"></a>Provider di archiviazione personalizzati per ASP.NET Core Identity

Di [Steve Smith](https://ardalis.com/)

ASP.NET Core Identity è un sistema estensibile che consente di creare un provider di archiviazione personalizzato e connetterlo all'app. In questo argomento viene descritto come creare un provider di archiviazione personalizzato per ASP.NET Core Identity . Vengono illustrati i concetti importanti per la creazione di un provider di archiviazione, ma non una procedura dettagliata.

[Visualizzare o scaricare un esempio da GitHub](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/identity-custom-storage-providers/sample/CustomIdentityProviderSample).

## <a name="introduction"></a>Introduzione

Per impostazione predefinita, il ASP.NET Core Identity sistema archivia le informazioni sugli utenti in un database SQL Server usando Entity Framework Core. Per molte app, questo approccio funziona correttamente. Tuttavia, può essere preferibile utilizzare un meccanismo di persistenza o uno schema di dati diverso. Ad esempio:

* Si usa l' [archiviazione tabelle di Azure](/azure/storage/) o un altro archivio dati.
* Le tabelle di database hanno una struttura diversa. 
* È possibile usare un approccio di accesso ai dati diverso, ad esempio [elegante](https://github.com/StackExchange/Dapper). 

In ognuno di questi casi, è possibile scrivere un provider personalizzato per il meccanismo di archiviazione e collegare tale provider nell'app.

ASP.NET Core Identity è incluso nei modelli di progetto di Visual Studio con l'opzione "singoli account utente".

Quando si usa il interfaccia della riga di comando di .NET Core, aggiungere `-au Individual` :

```dotnetcli
dotnet new mvc -au Individual
```

## <a name="the-aspnet-core-identity-architecture"></a>ASP.NET Core IdentityArchitettura

ASP.NET Core Identity è costituito da classi denominate Manager e archivi. I *responsabili* sono classi di alto livello che lo sviluppatore di app usa per eseguire operazioni, ad esempio la creazione di un Identity utente. Gli *archivi* sono classi di livello inferiore che specificano il modo in cui le entità, ad esempio utenti e ruoli, vengono rese permanente. Gli archivi seguono il modello di repository e sono strettamente collegati al meccanismo di persistenza. I Manager vengono separati dagli archivi, ovvero è possibile sostituire il meccanismo di persistenza senza modificare il codice dell'applicazione (ad eccezione della configurazione).

Il diagramma seguente illustra il modo in cui un'app Web interagisce con i Manager, mentre gli archivi interagiscono con il livello di accesso ai dati.

![Le app ASP.NET Core funzionano con i Manager, ad esempio ' UserManager ',' RoleManager '. I manager lavorano con i negozi (ad esempio, "userStore ambito") che comunicano con un'origine dati usando una libreria come Entity Framework Core.](identity-custom-storage-providers/_static/identity-architecture-diagram.png)

Per creare un provider di archiviazione personalizzato, creare l'origine dati, il livello di accesso ai dati e le classi di archivio che interagiscono con questo livello di accesso ai dati (le caselle verde e grigio nel diagramma precedente). Non è necessario personalizzare i Manager o il codice dell'app che interagisce con essi (le caselle blu sopra).

Quando si crea una nuova istanza di `UserManager` o `RoleManager` si fornisce il tipo della classe utente e si passa un'istanza della classe Store come argomento. Questo approccio consente di inserire le classi personalizzate in ASP.NET Core. 

[Riconfigurare l'app per l'uso di un nuovo provider di archiviazione](#reconfigure-app-to-use-a-new-storage-provider) Mostra come creare un'istanza di `UserManager` e `RoleManager` con un archivio personalizzato.

## <a name="aspnet-core-identity-stores-data-types"></a>ASP.NET Core Identity Archivia i tipi di dati

[ASP.NET Core Identity](https://github.com/aspnet/identity) i tipi di dati sono descritti in dettaglio nelle sezioni seguenti:

### <a name="users"></a>Utenti

Utenti registrati del sito Web. Il tipo di [ Identity utente](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuser) può essere esteso o usato come esempio per un tipo personalizzato. Non è necessario ereditare da un particolare tipo per implementare una soluzione di archiviazione delle identità personalizzata.

### <a name="user-claims"></a>Attestazioni utente

Set di istruzioni (o [attestazioni](/dotnet/api/system.security.claims.claim)) sull'utente che rappresenta l'identità dell'utente. Consente di abilitare un'espressione maggiore dell'identità dell'utente che può essere eseguita tramite i ruoli.

### <a name="user-logins"></a>Account di accesso utente

Informazioni sul provider di autenticazione esterno, ad esempio Facebook o un account Microsoft, da usare per la registrazione di un utente. [Esempio](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuserlogin)

### <a name="roles"></a>Ruoli

Gruppi di autorizzazione per il sito. Include l'ID del ruolo e il nome del ruolo (ad esempio, "admin" o "Employee"). [Esempio](/dotnet/api/microsoft.aspnet.identity.corecompat.identityrole)

## <a name="the-data-access-layer"></a>Livello di accesso ai dati

In questo argomento si presuppone che l'utente abbia familiarità con il meccanismo di persistenza che si intende usare e come creare entità per tale meccanismo. In questo argomento non vengono fornite informazioni dettagliate su come creare i repository o le classi di accesso ai dati. fornisce alcuni suggerimenti sulle decisioni di progettazione quando si utilizza ASP.NET Core Identity .

Quando si progetta il livello di accesso ai dati per un provider di archivio personalizzato, è possibile avere una grande libertà. È sufficiente creare meccanismi di persistenza per le funzionalità che si intende usare nell'app. Se, ad esempio, non si usano i ruoli nell'app, non è necessario creare spazio di archiviazione per i ruoli o le associazioni ruolo utente. La tecnologia e l'infrastruttura esistente possono richiedere una struttura molto diversa dall'implementazione predefinita di ASP.NET Core Identity . Nel livello di accesso ai dati fornire la logica per lavorare con la struttura dell'implementazione dell'archiviazione.

Il livello di accesso ai dati fornisce la logica per salvare i dati da ASP.NET Core Identity a un'origine dati. Il livello di accesso ai dati per il provider di archiviazione personalizzato potrebbe includere le classi seguenti per archiviare le informazioni relative a utenti e ruoli.

### <a name="context-class"></a>Context (classe)

Incapsula le informazioni per la connessione al meccanismo di persistenza e l'esecuzione di query. Per diverse classi di dati è necessaria un'istanza di questa classe, generalmente fornita tramite l'inserimento di dipendenze. [Esempio](/dotnet/api/microsoft.aspnet.identity.corecompat.identitydbcontext-1).

### <a name="user-storage"></a>Archiviazione utente

Archivia e recupera le informazioni utente, ad esempio il nome utente e l'hash della password. [Esempio](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

### <a name="role-storage"></a>Archiviazione del ruolo

Archivia e recupera le informazioni sui ruoli, ad esempio il nome del ruolo. [Esempio](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.rolestore-1)

### <a name="userclaims-storage"></a>Archiviazione UserClaims

Archivia e recupera le informazioni sull'attestazione utente, ad esempio il tipo di attestazione e il valore. [Esempio](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

### <a name="userlogins-storage"></a>Archiviazione UserLogins

Archivia e recupera le informazioni di accesso dell'utente, ad esempio un provider di autenticazione esterno. [Esempio](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

### <a name="userrole-storage"></a>Archiviazione UserRole

Archivia e recupera i ruoli assegnati a quali utenti. [Esempio](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

**Suggerimento:** Implementare solo le classi che si intende usare nell'app.

Nelle classi di accesso ai dati fornire codice per eseguire operazioni sui dati per il meccanismo di persistenza. Ad esempio, all'interno di un provider personalizzato, è possibile che sia presente il codice seguente per creare un nuovo utente nella classe *Store* :

[!code-csharp[](identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/CustomUserStore.cs?name=createuser&highlight=7)]

La logica di implementazione per la creazione dell'utente è nel `_usersTable.CreateAsync` metodo, illustrato di seguito.

## <a name="customize-the-user-class"></a>Personalizzare la classe utente

Quando si implementa un provider di archiviazione, creare una classe utente equivalente alla [ Identity classe User](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuser).

Come minimo, la classe utente deve includere un oggetto `Id` e una `UserName` Proprietà.

La `IdentityUser` classe definisce le proprietà chiamate da `UserManager` durante l'esecuzione di operazioni richieste. Il tipo predefinito della `Id` proprietà è una stringa, ma è possibile ereditare da `IdentityUser<TKey, TUserClaim, TUserRole, TUserLogin, TUserToken>` e specificare un tipo diverso. Il Framework prevede che l'implementazione dell'archiviazione gestisca le conversioni dei tipi di dati.

## <a name="customize-the-user-store"></a>Personalizzare l'archivio utenti

Creare una `UserStore` classe che fornisce i metodi per tutte le operazioni sui dati dell'utente. Questa classe è equivalente alla classe [ &lt; &gt; TUser di userStore ambito](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.userstore-1) . Nella `UserStore` classe implementare `IUserStore<TUser>` e le interfacce facoltative necessarie. È possibile selezionare le interfacce facoltative da implementare in base alle funzionalità fornite nell'app.

### <a name="optional-interfaces"></a>Interfacce facoltative

* [IUserRoleStore](/dotnet/api/microsoft.aspnetcore.identity.iuserrolestore-1)
* [IUserClaimStore](/dotnet/api/microsoft.aspnetcore.identity.iuserclaimstore-1)
* [IUserPasswordStore](/dotnet/api/microsoft.aspnetcore.identity.iuserpasswordstore-1)
* [IUserSecurityStampStore](/dotnet/api/microsoft.aspnetcore.identity.iusersecuritystampstore-1)
* [IUserEmailStore](/dotnet/api/microsoft.aspnetcore.identity.iuseremailstore-1)
* [IUserPhoneNumberStore](/dotnet/api/microsoft.aspnetcore.identity.iuserphonenumberstore-1)
* [IQueryableUserStore](/dotnet/api/microsoft.aspnetcore.identity.iqueryableuserstore-1)
* [IUserLoginStore](/dotnet/api/microsoft.aspnetcore.identity.iuserloginstore-1)
* [IUserTwoFactorStore](/dotnet/api/microsoft.aspnetcore.identity.iusertwofactorstore-1)
* [IUserLockoutStore](/dotnet/api/microsoft.aspnetcore.identity.iuserlockoutstore-1)

Le interfacce opzionali ereditano da `IUserStore<TUser>` . Nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/security/authentication/identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/CustomUserStore.cs)è possibile visualizzare un archivio utenti di esempio parzialmente implementato.

All'interno della `UserStore` classe si utilizzano le classi di accesso ai dati create per eseguire operazioni. Questi vengono passati usando l'inserimento di dipendenze. Ad esempio, nel SQL Server con l'implementazione di elegante, la `UserStore` classe ha il `CreateAsync` metodo che usa un'istanza di `DapperUsersTable` per inserire un nuovo record:

[!code-csharp[](identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/DapperUsersTable.cs?name=createuser&highlight=7)]

### <a name="interfaces-to-implement-when-customizing-user-store"></a>Interfacce da implementare quando si Personalizza l'archivio utenti

* **IUserStore**  
 L' [interfaccia &lt; TUser &gt; di IUserStore](/dotnet/api/microsoft.aspnetcore.identity.iuserstore-1) è l'unica interfaccia che è necessario implementare nell'archivio utente. Definisce i metodi per la creazione, l'aggiornamento, l'eliminazione e il recupero degli utenti.
* **IUserClaimStore**  
 L' [interfaccia &lt; IUserClaimStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iuserclaimstore-1) definisce i metodi implementati per abilitare le attestazioni utente. Contiene i metodi per l'aggiunta, la rimozione e il recupero di attestazioni utente.
* **IUserLoginStore**  
 Il [ &lt; &gt; TUser IUserLoginStore](/dotnet/api/microsoft.aspnetcore.identity.iuserloginstore-1) definisce i metodi implementati per abilitare i provider di autenticazione esterni. Contiene i metodi per l'aggiunta, la rimozione e il recupero degli account di accesso utente e un metodo per il recupero di un utente in base alle informazioni di accesso.
* **IUserRoleStore**  
 L' [interfaccia &lt; IUserRoleStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iuserrolestore-1) definisce i metodi implementati per eseguire il mapping di un utente a un ruolo. Contiene i metodi per aggiungere, rimuovere e recuperare i ruoli di un utente e un metodo per verificare se un utente è assegnato a un ruolo.
* **IUserPasswordStore**  
 L' [interfaccia &lt; IUserPasswordStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iuserpasswordstore-1) definisce i metodi implementati per rendere permanente le password con hash. Contiene i metodi per ottenere e impostare la password con hash e un metodo che indica se l'utente ha impostato una password.
* **IUserSecurityStampStore**  
 L' [interfaccia &lt; IUserSecurityStampStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iusersecuritystampstore-1) definisce i metodi implementati per usare un indicatore di sicurezza per indicare se le informazioni sull'account dell'utente sono state modificate. Questo timbro viene aggiornato quando un utente modifica la password o aggiunge o rimuove gli account di accesso. Contiene i metodi per ottenere e impostare l'indicatore di sicurezza.
* **IUserTwoFactorStore**  
 L' [interfaccia &lt; IUserTwoFactorStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iusertwofactorstore-1) definisce i metodi implementati per supportare l'autenticazione a due fattori. Contiene i metodi per ottenere e impostare se per un utente è abilitata l'autenticazione a due fattori.
* **IUserPhoneNumberStore**  
 L' [interfaccia &lt; IUserPhoneNumberStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iuserphonenumberstore-1) definisce i metodi implementati per archiviare i numeri di telefono dell'utente. Contiene i metodi per ottenere e impostare il numero di telefono e se il numero di telefono è confermato.
* **IUserEmailStore**  
 L' [interfaccia &lt; IUserEmailStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iuseremailstore-1) definisce i metodi implementati per archiviare gli indirizzi di posta elettronica dell'utente. Contiene i metodi per ottenere e impostare l'indirizzo di posta elettronica e verificare se il messaggio di posta elettronica è confermato.
* **IUserLockoutStore**  
 L' [interfaccia &lt; IUserLockoutStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iuserlockoutstore-1) definisce i metodi implementati per archiviare informazioni sul blocco di un account. Contiene i metodi per tenere traccia dei tentativi di accesso e dei blocchi non riusciti.
* **IQueryableUserStore**  
 L' [interfaccia &lt; IQueryableUserStore &gt; TUser](/dotnet/api/microsoft.aspnetcore.identity.iqueryableuserstore-1) definisce i membri implementati per fornire un archivio utenti Queryable.

Si implementano solo le interfacce necessarie nell'app. Ad esempio:

```csharp
public class UserStore : IUserStore<IdentityUser>,
                         IUserClaimStore<IdentityUser>,
                         IUserLoginStore<IdentityUser>,
                         IUserRoleStore<IdentityUser>,
                         IUserPasswordStore<IdentityUser>,
                         IUserSecurityStampStore<IdentityUser>
{
    // interface implementations not shown
}
```

### <a name="identityuserclaim-identityuserlogin-and-identityuserrole"></a>IdentityUserClaim, Identity userlogin e Identity userrole

Lo `Microsoft.AspNet.Identity.EntityFramework` spazio dei nomi contiene implementazioni delle classi [ Identity UserClaim](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.identityuserclaim-1), [ Identity userlogin](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuserlogin)e [ Identity userrole](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.identityuserrole-1) . Se si usano queste funzionalità, è possibile creare versioni personalizzate di queste classi e definire le proprietà per l'app. Tuttavia, a volte è più efficiente non caricare queste entità in memoria durante l'esecuzione di operazioni di base, ad esempio l'aggiunta o la rimozione di un'attestazione dell'utente. Al contrario, le classi dell'archivio back-end possono eseguire queste operazioni direttamente sull'origine dati. Il metodo, ad esempio, `UserStore.GetClaimsAsync` può chiamare il `userClaimTable.FindByUserId(user.Id)` metodo per eseguire direttamente una query sulla tabella e restituire un elenco di attestazioni.

## <a name="customize-the-role-class"></a>Personalizzare la classe Role

Quando si implementa un provider di archiviazione del ruolo, è possibile creare un tipo di ruolo personalizzato. Non è necessario implementare un'interfaccia specifica, ma deve avere un oggetto `Id` e in genere avrà una `Name` Proprietà.

Di seguito è riportato un esempio di classe Role:

[!code-csharp[](identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/ApplicationRole.cs)]

## <a name="customize-the-role-store"></a>Personalizzare l'archivio dei ruoli

È possibile creare una `RoleStore` classe che fornisce i metodi per tutte le operazioni di dati sui ruoli. Questa classe è equivalente alla classe [ &lt; &gt; TRole di nell'rolestore](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.rolestore-1) . Nella `RoleStore` classe si implementa `IRoleStore<TRole>` e, facoltativamente, l' `IQueryableRoleStore<TRole>` interfaccia.

* **&lt;TRole IRoleStore&gt;**  
 L' [interfaccia &lt; IRoleStore &gt; TRole](/dotnet/api/microsoft.aspnetcore.identity.irolestore-1) definisce i metodi da implementare nella classe dell'archivio ruoli. Contiene i metodi per la creazione, l'aggiornamento, l'eliminazione e il recupero di ruoli.
* **&lt;TRole nell'rolestore&gt;**  
 Per personalizzare `RoleStore` , creare una classe che implementi l' `IRoleStore<TRole>` interfaccia. 

## <a name="reconfigure-app-to-use-a-new-storage-provider"></a>Riconfigurare l'app per l'uso di un nuovo provider di archiviazione

Dopo aver implementato un provider di archiviazione, è necessario configurare l'app per usarla. Se l'app usa il provider predefinito, sostituirla con il provider personalizzato.

1. Rimuovere il `Microsoft.AspNetCore.EntityFramework.Identity` pacchetto NuGet.
1. Se il provider di archiviazione si trova in un progetto o un pacchetto separato, aggiungervi un riferimento.
1. Sostituire tutti i riferimenti a `Microsoft.AspNetCore.EntityFramework.Identity` con un'istruzione using per lo spazio dei nomi del provider di archiviazione.
1. Nel `ConfigureServices` Metodo modificare il metodo in `AddIdentity` modo da usare i tipi personalizzati. A questo scopo, è possibile creare metodi di estensione personalizzati. Per un esempio, vedere [ Identity ServiceCollectionExtensions](https://github.com/aspnet/Identity/blob/rel/1.1.0/src/Microsoft.AspNetCore.Identity/IdentityServiceCollectionExtensions.cs) .
1. Se si usano i ruoli, aggiornare `RoleManager` per usare la `RoleStore` classe.
1. Aggiornare la stringa di connessione e le credenziali alla configurazione dell'app.

Esempio:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add identity types
    services.AddIdentity<ApplicationUser, ApplicationRole>()
        .AddDefaultTokenProviders();

    // Identity Services
    services.AddTransient<IUserStore<ApplicationUser>, CustomUserStore>();
    services.AddTransient<IRoleStore<ApplicationRole>, CustomRoleStore>();
    string connectionString = Configuration.GetConnectionString("DefaultConnection");
    services.AddTransient<SqlConnection>(e => new SqlConnection(connectionString));
    services.AddTransient<DapperUsersTable>();

    // additional configuration
}
```

## <a name="references"></a>Riferimenti

* [Provider di archiviazione personalizzati per ASP.NET 4. x Identity](/aspnet/identity/overview/extensibility/overview-of-custom-storage-providers-for-aspnet-identity)
* [ASP.NET Core Identity](https://github.com/dotnet/AspNetCore/tree/main/src/Identity): Questo repository include collegamenti a provider di archivi gestiti dalla community.
