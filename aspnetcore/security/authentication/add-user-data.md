---
title: Aggiungere, scaricare ed eliminare dati utente Identity in un progetto ASP.NET Core
author: rick-anderson
description: Informazioni su come aggiungere dati utente personalizzati a Identity in un progetto ASP.NET Core. Eliminare i dati per ogni GDPR.
ms.author: riande
ms.date: 03/26/2020
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
uid: security/authentication/add-user-data
ms.openlocfilehash: 2d921a0c72fb7c03cd88966077e2d33e4b19ffa1
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585916"
---
# <a name="add-download-and-delete-custom-user-data-to-identity-in-an-aspnet-core-project"></a>Aggiungere, scaricare ed eliminare dati utente personalizzati Identity in un progetto ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

Questo articolo illustra come:

* Aggiungere dati utente personalizzati a un'app Web ASP.NET Core.
* Contrassegnare il modello di dati utente personalizzato con l' <xref:Microsoft.AspNetCore.Identity.PersonalDataAttribute> attributo in modo che sia disponibile automaticamente per il download e l'eliminazione. La possibilità di scaricare ed eliminare i dati consente di soddisfare i requisiti di [GDPR](xref:security/gdpr) .

Il progetto di esempio viene creato da un' Razor app Web di pagine, ma le istruzioni sono simili per un'app Web MVC ASP.NET Core.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/add-user-data) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [](~/includes/3.0-SDK.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE [](~/includes/2.2-SDK.md)]

::: moniker-end

## <a name="create-a-razor-web-app"></a>Creare un' Razor app Web

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-3.0"

* Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**. Denominare il progetto **app Web 1** se si desidera che corrisponda allo spazio dei nomi del codice di [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) .
* Selezionare **ASP.NET Core applicazione Web** > **OK**
* Selezionare **ASP.NET Core 3,0** nell'elenco a discesa
* Selezionare l' **applicazione Web** > **OK**
* Compilare ed eseguire il progetto.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**. Denominare il progetto **app Web 1** se si desidera che corrisponda allo spazio dei nomi del codice di [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) .
* Selezionare **ASP.NET Core applicazione Web** > **OK**
* Selezionare **ASP.NET Core 2,2** nell'elenco a discesa
* Selezionare l' **applicazione Web** > **OK**
* Compilare ed eseguire il progetto.

::: moniker-end


# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

```dotnetcli
dotnet new webapp -o WebApp1
```

---

## <a name="run-the-identity-scaffolder"></a>Eseguire l' Identity impalcatura

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Da **Esplora soluzioni**, fare clic con il pulsante destro del mouse sul progetto > **Aggiungi**  >  **nuovo elemento con impalcatura**.
* Dal riquadro sinistro della finestra di dialogo **Aggiungi impalcatura** selezionare **Identity**  >  **Aggiungi**.
* Nella finestra di dialogo **Aggiungi Identity** sono disponibili le opzioni seguenti:
  * Selezionare il file di layout esistente  *~/Pages/Shared/_Layout. cshtml*
  * Selezionare i seguenti file di cui eseguire l'override:
    * **Account/registro**
    * **Account/Gestione/indice**
  * Selezionare il **+** pulsante per creare una nuova **classe del contesto dati**. Accettare il tipo (**app Web 1. Models. WebApp1Context** se il progetto è denominato **app Web 1**).
  * Selezionare il **+** pulsante per creare una nuova **classe utente**. Accettare il tipo (**WebApp1User** se il progetto è denominato **app Web 1**) > **Aggiungi**.
* Selezionare **Aggiungi**.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Se in precedenza non è stato installato il ASP.NET Core impalcatura, installarlo ora:

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

Aggiungere un riferimento al pacchetto [Microsoft. VisualStudio. Web. CodeGeneration. Design](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/) al file di progetto (con estensione csproj). Eseguire il comando seguente nella directory del progetto:

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
```

Eseguire il comando seguente per elencare le Identity Opzioni dell'impalcatura:

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

Nella cartella del progetto eseguire l' Identity impalcatura:

```dotnetcli
dotnet aspnet-codegenerator identity -u WebApp1User -fi Account.Register;Account.Manage.Index
```

---

Seguire le istruzioni in [Migrations, UseAuthentication e layout](xref:security/authentication/scaffold-identity#efm) per eseguire la procedura seguente:

* Creare una migrazione e aggiornare il database.
* Aggiunta di `UseAuthentication` a `Startup.Configure`.
* Aggiungere `<partial name="_LoginPartial" />` al file di layout.
* Eseguire il test dell'app:
  * Registrare un utente
  * Selezionare il nuovo nome utente (accanto al collegamento di **disconnessione** ). Potrebbe essere necessario espandere la finestra o selezionare l'icona della barra di spostamento per visualizzare il nome utente e altri collegamenti.
  * Selezionare la scheda **dati personali** .
  * Selezionare il pulsante **download** ed esaminare il *PersonalData.jssu* file.
  * Testare il pulsante **Elimina per eliminare** l'utente che ha eseguito l'accesso.

## <a name="add-custom-user-data-to-the-identity-db"></a>Aggiungere dati utente personalizzati al Identity database

Aggiornare la `IdentityUser` classe derivata con proprietà personalizzate. Se è stato denominato il progetto app Web 1, il file è denominato *aree/ Identity /Data/WebApp1User.cs*. Aggiornare il file con il codice seguente:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

Le proprietà con l'attributo [personaldata](/dotnet/api/microsoft.aspnetcore.identity.personaldataattribute) sono:

* Viene eliminato quando viene chiamata la pagina *aree/ Identity /pages/account/Manage/DeletePersonalData.cshtml* Razor `UserManager.Delete` .
* Incluso nei dati scaricati dalla pagina *aree/ Identity /pages/account/Manage/DownloadPersonalData.cshtml* Razor .

### <a name="update-the-accountmanageindexcshtml-page"></a>Aggiornare la pagina account/Manage/index. cshtml

Aggiornare `InputModel` in *areas/ Identity /pages/account/Manage/index.cshtml.cs* con il codice evidenziato seguente:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=24-32,48-49,96-104,106)]

Aggiornare le *aree/ Identity /pages/account/Manage/index.cshtml* con il markup evidenziato seguente:

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=18-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=28-36,63-64,98-106,119)]

Aggiornare le *aree/ Identity /pages/account/Manage/index.cshtml* con il markup evidenziato seguente:

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=35-42)]

::: moniker-end

### <a name="update-the-accountregistercshtml-page"></a>Aggiornare la pagina account/Register. cshtml

Aggiornare `InputModel` in *areas/ Identity /pages/account/Register.cshtml.cs* con il codice evidenziato seguente:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=30-38,70-71)]

Aggiornare le *aree/ Identity /pages/account/Register.cshtml* con il markup evidenziato seguente:

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=28-36,67,66)]

Aggiornare le *aree/ Identity /pages/account/Register.cshtml* con il markup evidenziato seguente:

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end


Compilare il progetto.

### <a name="add-a-migration-for-the-custom-user-data"></a>Aggiungere una migrazione per i dati utente personalizzati

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Nella **console di gestione pacchetti** di Visual Studio:

```powershell
Add-Migration CustomUserData
Update-Database
```

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CustomUserData
dotnet ef database update
```

---

## <a name="test-create-view-download-delete-custom-user-data"></a>Test creare, visualizzare, scaricare, eliminare dati utente personalizzati

Eseguire il test dell'app:

* Registrare un nuovo utente.
* Visualizzare i dati utente personalizzati nella `/Identity/Account/Manage` pagina.
* Scaricare e visualizzare i dati personali degli utenti dalla `/Identity/Account/Manage/PersonalData` pagina.

## <a name="add-claims-to-identity-using-iuserclaimsprincipalfactoryapplicationuser"></a>Aggiungere attestazioni all' Identity uso di IUserClaimsPrincipalFactory<ApplicationUser>

> [!NOTE]
> Questa sezione non è un'estensione dell'esercitazione precedente. Per applicare i passaggi seguenti all'app compilata con l'esercitazione, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/18797).

È possibile aggiungere attestazioni aggiuntive a utilizzando ASP.NET Core Identity l' `IUserClaimsPrincipalFactory<T>` interfaccia. Questa classe può essere aggiunta all'app nel `Startup.ConfigureServices` metodo. Aggiungere l'implementazione personalizzata della classe come indicato di seguito:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddScoped<IUserClaimsPrincipalFactory<ApplicationUser>, 
        AdditionalUserClaimsPrincipalFactory>();
```

Il codice dimostrativo usa la `ApplicationUser` classe. Questa classe aggiunge una `IsAdmin` proprietà che viene utilizzata per aggiungere l'attestazione aggiuntiva.

```csharp
public class ApplicationUser : IdentityUser
{
    public bool IsAdmin { get; set; }
}
```

L'oggetto `AdditionalUserClaimsPrincipalFactory` implementa l'interfaccia `UserClaimsPrincipalFactory`. Viene aggiunta una nuova attestazione di ruolo a `ClaimsPrincipal` .

```csharp
public class AdditionalUserClaimsPrincipalFactory 
        : UserClaimsPrincipalFactory<ApplicationUser, IdentityRole>
{
    public AdditionalUserClaimsPrincipalFactory( 
        UserManager<ApplicationUser> userManager,
        RoleManager<IdentityRole> roleManager, 
        IOptions<IdentityOptions> optionsAccessor) 
        : base(userManager, roleManager, optionsAccessor)
    {}

    public async override Task<ClaimsPrincipal> CreateAsync(ApplicationUser user)
    {
        var principal = await base.CreateAsync(user);
        var identity = (ClaimsIdentity)principal.Identity;

        var claims = new List<Claim>();
        if (user.IsAdmin)
        {
            claims.Add(new Claim(JwtClaimTypes.Role, "admin"));
        }
        else
        {
            claims.Add(new Claim(JwtClaimTypes.Role, "user"));
        }

        identity.AddClaims(claims);
        return principal;
    }
}
```

L'attestazione aggiuntiva può quindi essere usata nell'app. In una Razor pagina, l' `IAuthorizationService` istanza di può essere utilizzata per accedere al valore dell'attestazione.

```cshtml
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

@if ((await AuthorizationService.AuthorizeAsync(User, "IsAdmin")).Succeeded)
{
    <ul class="mr-auto navbar-nav">
        <li class="nav-item">
            <a class="nav-link" asp-controller="Admin" asp-action="Index">ADMIN</a>
        </li>
    </ul>
}
```
