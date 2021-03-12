---
title: Introduzione a Identity on ASP.NET Core
author: rick-anderson
description: Usare Identity con un'app ASP.NET Core. Informazioni su come impostare i requisiti per le password (RequireDigit, RequiredLength, RequiredUniqueChars e altro).
ms.author: riande
ms.date: 7/15/2020
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
uid: security/authentication/identity
ms.openlocfilehash: 7141808b3190db2c882d562d753cd68fe3f985ff
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589738"
---
# <a name="introduction-to-identity-on-aspnet-core"></a><span data-ttu-id="a5dc9-104">Introduzione a Identity on ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="a5dc9-104">Introduction to Identity on ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="a5dc9-105">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="a5dc9-105">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="a5dc9-106">ASP.NET Core Identity:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-106">ASP.NET Core Identity:</span></span>

* <span data-ttu-id="a5dc9-107">È un'API che supporta la funzionalità di accesso dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-107">Is an API that supports user interface (UI) login functionality.</span></span>
* <span data-ttu-id="a5dc9-108">Gestisce utenti, password, dati di profilo, ruoli, attestazioni, token, conferma tramite posta elettronica e altro ancora.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-108">Manages users, passwords, profile data, roles, claims, tokens, email confirmation, and more.</span></span>

<span data-ttu-id="a5dc9-109">Gli utenti possono creare un account con le informazioni di accesso archiviate in Identity o possono usare un provider di accesso esterno.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-109">Users can create an account with the login information stored in Identity or they can use an external login provider.</span></span> <span data-ttu-id="a5dc9-110">I provider di accesso esterni supportati includono [Facebook, Google, account Microsoft e Twitter](xref:security/authentication/social/index).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-110">Supported external login providers include [Facebook, Google, Microsoft Account, and Twitter](xref:security/authentication/social/index).</span></span>

[!INCLUDE[](~/includes/requireAuth.md)]

<span data-ttu-id="a5dc9-111">Il [ Identity codice sorgente](https://github.com/dotnet/AspNetCore/tree/main/src/Identity) è disponibile in GitHub.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-111">The [Identity source code](https://github.com/dotnet/AspNetCore/tree/main/src/Identity) is available on GitHub.</span></span> <span data-ttu-id="a5dc9-112">[Impalcatura Identity ](xref:security/authentication/scaffold-identity) e visualizzare i file generati per esaminare l'interazione del modello con Identity .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-112">[Scaffold Identity](xref:security/authentication/scaffold-identity) and view the generated files to review the template interaction with Identity.</span></span>

<span data-ttu-id="a5dc9-113">Identity viene in genere configurato utilizzando un database di SQL Server per archiviare i nomi utente, le password e i dati di profilo.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-113">Identity is typically configured using a SQL Server database to store user names, passwords, and profile data.</span></span> <span data-ttu-id="a5dc9-114">In alternativa, è possibile usare un altro archivio permanente, ad esempio archiviazione tabelle di Azure.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-114">Alternatively, another persistent store can be used, for example, Azure Table Storage.</span></span>

<span data-ttu-id="a5dc9-115">In questo argomento si apprenderà come usare Identity per registrare, accedere e disconnettere un utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-115">In this topic, you learn how to use Identity to register, log in, and log out a user.</span></span> <span data-ttu-id="a5dc9-116">Nota: i modelli considerano il nome utente e il messaggio di posta elettronica come lo stesso per gli utenti.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-116">Note: the templates treat username and email as the same for users.</span></span> <span data-ttu-id="a5dc9-117">Per istruzioni più dettagliate sulla creazione di app che usano Identity , vedere [passaggi successivi](#next).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-117">For more detailed instructions about creating apps that use Identity, see [Next Steps](#next).</span></span>

<span data-ttu-id="a5dc9-118">[Piattaforma di identità Microsoft](/azure/active-directory/develop/) :</span><span class="sxs-lookup"><span data-stu-id="a5dc9-118">[Microsoft identity platform](/azure/active-directory/develop/) is:</span></span>

* <span data-ttu-id="a5dc9-119">Evoluzione della piattaforma per sviluppatori Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-119">An evolution of the Azure Active Directory (Azure AD) developer platform.</span></span>
* <span data-ttu-id="a5dc9-120">Non correlato a ASP.NET Core Identity .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-120">Unrelated to ASP.NET Core Identity.</span></span>

[!INCLUDE[](~/includes/IdentityServer4.md)]

<span data-ttu-id="a5dc9-121">Consente di [visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/identity/sample) ([come scaricare](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-121">[View or download the sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/identity/sample) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<a name="adi"></a>

## <a name="create-a-web-app-with-authentication"></a><span data-ttu-id="a5dc9-122">Creare un'app Web con l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="a5dc9-122">Create a Web app with authentication</span></span>

<span data-ttu-id="a5dc9-123">Creare un progetto di applicazione Web di ASP.NET Core con singoli account utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-123">Create an ASP.NET Core Web Application project with Individual User Accounts.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a5dc9-124">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a5dc9-124">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="a5dc9-125">Selezionare **file** > **nuovo** > **progetto**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-125">Select **File** > **New** > **Project**.</span></span>
* <span data-ttu-id="a5dc9-126">Selezionare **Applicazione Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-126">Select **ASP.NET Core Web Application**.</span></span> <span data-ttu-id="a5dc9-127">Denominare il progetto **app Web 1** in modo che abbia lo stesso spazio dei nomi del download del progetto.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-127">Name the project **WebApp1** to have the same namespace as the project download.</span></span> <span data-ttu-id="a5dc9-128">Fare clic su **OK**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-128">Click **OK**.</span></span>
* <span data-ttu-id="a5dc9-129">Selezionare un' **applicazione Web** ASP.NET Core, quindi selezionare **Modifica autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-129">Select an ASP.NET Core **Web Application**, then select **Change Authentication**.</span></span>
* <span data-ttu-id="a5dc9-130">Selezionare **singoli account utente** e fare clic su **OK**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-130">Select **Individual User Accounts** and click **OK**.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a5dc9-131">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a5dc9-131">.NET Core CLI</span></span>](#tab/netcore-cli)

```dotnetcli
dotnet new webapp --auth Individual -o WebApp1
```

<span data-ttu-id="a5dc9-132">Il comando precedente crea un' Razor app Web con SQLite.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-132">The preceding command creates a Razor web app using SQLite.</span></span> <span data-ttu-id="a5dc9-133">Per creare l'app Web con il database locale, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-133">To create the web app with LocalDB, run the following command:</span></span>

```dotnetcli
dotnet new webapp --auth Individual -uld -o WebApp1
```

---

<span data-ttu-id="a5dc9-134">Il progetto generato fornisce [ASP.NET Core Identity](xref:security/authentication/identity) una [ Razor libreria di classi](xref:razor-pages/ui-class).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-134">The generated project provides [ASP.NET Core Identity](xref:security/authentication/identity) as a [Razor Class Library](xref:razor-pages/ui-class).</span></span> <span data-ttu-id="a5dc9-135">La Identity Razor libreria di classi espone gli endpoint con l' `Identity` area.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-135">The Identity Razor Class Library exposes endpoints with the `Identity` area.</span></span> <span data-ttu-id="a5dc9-136">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-136">For example:</span></span>

* <span data-ttu-id="a5dc9-137">/Identity/Account/Login</span><span class="sxs-lookup"><span data-stu-id="a5dc9-137">/Identity/Account/Login</span></span>
* <span data-ttu-id="a5dc9-138">/Identity/Account/Logout</span><span class="sxs-lookup"><span data-stu-id="a5dc9-138">/Identity/Account/Logout</span></span>
* <span data-ttu-id="a5dc9-139">/Identity/Account/Manage</span><span class="sxs-lookup"><span data-stu-id="a5dc9-139">/Identity/Account/Manage</span></span>

### <a name="apply-migrations"></a><span data-ttu-id="a5dc9-140">Applicare le migrazioni</span><span class="sxs-lookup"><span data-stu-id="a5dc9-140">Apply migrations</span></span>

<span data-ttu-id="a5dc9-141">Applicare le migrazioni per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-141">Apply the migrations to initialize the database.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a5dc9-142">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a5dc9-142">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="a5dc9-143">Eseguire il comando seguente nella console di gestione pacchetti (PMC):</span><span class="sxs-lookup"><span data-stu-id="a5dc9-143">Run the following command in the Package Manager Console (PMC):</span></span>

`PM> Update-Database`

# <a name="net-core-cli"></a>[<span data-ttu-id="a5dc9-144">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a5dc9-144">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="a5dc9-145">Le migrazioni non sono necessarie in questo passaggio quando si usa SQLite.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-145">Migrations are not necessary at this step when using SQLite.</span></span>

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

<span data-ttu-id="a5dc9-146">Per il database locale, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-146">For LocalDB, run the following command:</span></span>

```dotnetcli
dotnet ef database update
```

---

### <a name="test-register-and-login"></a><span data-ttu-id="a5dc9-147">Registro di test e account di accesso</span><span class="sxs-lookup"><span data-stu-id="a5dc9-147">Test Register and Login</span></span>

<span data-ttu-id="a5dc9-148">Eseguire l'app e registrare un utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-148">Run the app and register a user.</span></span> <span data-ttu-id="a5dc9-149">A seconda delle dimensioni dello schermo, potrebbe essere necessario selezionare l'interruttore di spostamento per visualizzare i collegamenti di **Registro** e di **accesso** .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-149">Depending on your screen size, you might need to select the navigation toggle button to see the **Register** and **Login** links.</span></span>

[!INCLUDE[](~/includes/view-identity-db.md)]

<a name="pw"></a>

### <a name="configure-identity-services"></a><span data-ttu-id="a5dc9-150">Configurare i Identity Servizi</span><span class="sxs-lookup"><span data-stu-id="a5dc9-150">Configure Identity services</span></span>

<span data-ttu-id="a5dc9-151">I servizi vengono aggiunti in `ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-151">Services are added in `ConfigureServices`.</span></span> <span data-ttu-id="a5dc9-152">Il modello tipico consiste nel chiamare tutti i metodi `Add{Service}` e quindi chiamare tutti i metodi `services.Configure{Service}`.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-152">The typical pattern is to call all the `Add{Service}` methods, and then call all the `services.Configure{Service}` methods.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

[!code-csharp[](identity/sample/WebApp3/Startup.cs?name=snippet_configureservices&highlight=11-99)]

<span data-ttu-id="a5dc9-153">Il codice evidenziato precedente viene configurato Identity con i valori di opzione predefiniti.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-153">The preceding highlighted code configures Identity with default option values.</span></span> <span data-ttu-id="a5dc9-154">I servizi vengono resi disponibili per l'applicazione tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-154">Services are made available to the app through [dependency injection](xref:fundamentals/dependency-injection).</span></span>

<span data-ttu-id="a5dc9-155">Identity viene abilitato chiamando <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-155">Identity is enabled by calling <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*>.</span></span> <span data-ttu-id="a5dc9-156">`UseAuthentication` aggiunge il [middleware](xref:fundamentals/middleware/index) di autenticazione alla pipeline della richiesta.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-156">`UseAuthentication` adds authentication [middleware](xref:fundamentals/middleware/index) to the request pipeline.</span></span>

[!code-csharp[](identity/sample/WebApp3/Startup.cs?name=snippet_configure&highlight=19)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](identity/sample/WebApp5x/Startup.cs?name=snippet_configureservices&highlight=12-99)]

<span data-ttu-id="a5dc9-157">Il codice precedente Configura Identity con i valori di opzione predefiniti.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-157">The preceding code configures Identity with default option values.</span></span> <span data-ttu-id="a5dc9-158">I servizi vengono resi disponibili per l'applicazione tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-158">Services are made available to the app through [dependency injection](xref:fundamentals/dependency-injection).</span></span>

<span data-ttu-id="a5dc9-159">Identity viene abilitato chiamando [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-159">Identity is enabled by calling [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span></span> <span data-ttu-id="a5dc9-160">`UseAuthentication` aggiunge il [middleware](xref:fundamentals/middleware/index) di autenticazione alla pipeline della richiesta.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-160">`UseAuthentication` adds authentication [middleware](xref:fundamentals/middleware/index) to the request pipeline.</span></span>

[!code-csharp[](identity/sample/WebApp5x/Startup.cs?name=snippet_configure&highlight=19)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="a5dc9-161">L'app generata dal modello non usa l' [autorizzazione](xref:security/authorization/secure-data).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-161">The template-generated app doesn't use [authorization](xref:security/authorization/secure-data).</span></span> <span data-ttu-id="a5dc9-162">`app.UseAuthorization` è incluso per assicurarsi che venga aggiunto nell'ordine corretto se l'app aggiunge l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-162">`app.UseAuthorization` is included to ensure it's added in the correct order should the app add authorization.</span></span> <span data-ttu-id="a5dc9-163">`UseRouting`, `UseAuthentication` , `UseAuthorization` e `UseEndpoints` devono essere chiamati nell'ordine indicato nel codice precedente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-163">`UseRouting`, `UseAuthentication`, `UseAuthorization`, and `UseEndpoints` must be called in the order shown in the preceding code.</span></span>

<span data-ttu-id="a5dc9-164">Per ulteriori informazioni su `IdentityOptions` e `Startup` , vedere <xref:Microsoft.AspNetCore.Identity.IdentityOptions> e [avvio dell'applicazione](xref:fundamentals/startup).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-164">For more information on `IdentityOptions` and `Startup`, see <xref:Microsoft.AspNetCore.Identity.IdentityOptions> and [Application Startup](xref:fundamentals/startup).</span></span>

## <a name="scaffold-register-login-logout-and-registerconfirmation"></a><span data-ttu-id="a5dc9-165">Registro di ponteggi, account di accesso, disconnessione e RegisterConfirmation</span><span class="sxs-lookup"><span data-stu-id="a5dc9-165">Scaffold Register, Login, LogOut, and RegisterConfirmation</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a5dc9-166">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a5dc9-166">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="a5dc9-167">Aggiungere i `Register` `Login` file,, `LogOut` e `RegisterConfirmation` .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-167">Add the `Register`, `Login`, `LogOut`, and `RegisterConfirmation` files.</span></span> <span data-ttu-id="a5dc9-168">Seguire l' [identità del patibolo in un Razor progetto con](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization) le istruzioni di autorizzazione per generare il codice illustrato in questa sezione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-168">Follow the [Scaffold identity into a Razor project with authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization) instructions to generate the code shown in this section.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a5dc9-169">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a5dc9-169">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="a5dc9-170">Se il progetto è stato creato con il nome **app Web 1**, eseguire i comandi seguenti.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-170">If you created the project with name **WebApp1**, run the following commands.</span></span> <span data-ttu-id="a5dc9-171">In caso contrario, utilizzare lo spazio dei nomi corretto per `ApplicationDbContext` :</span><span class="sxs-lookup"><span data-stu-id="a5dc9-171">Otherwise, use the correct namespace for the `ApplicationDbContext`:</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc WebApp1.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout;Account.RegisterConfirmation"
```

<span data-ttu-id="a5dc9-172">PowerShell usa il punto e virgola come separatore di comandi.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-172">PowerShell uses semicolon as a command separator.</span></span> <span data-ttu-id="a5dc9-173">Quando si usa PowerShell, usare il carattere di escape per il punto e virgola nell'elenco dei file o inserire l'elenco di file tra virgolette doppie, come illustrato nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-173">When using PowerShell, escape the semicolons in the file list or put the file list in double quotes, as the preceding example shows.</span></span>

<span data-ttu-id="a5dc9-174">Per ulteriori informazioni sull'impalcatura Identity , vedere la pagina relativa all' [identità del patibolo in un Razor progetto con autorizzazione](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-174">For more information on scaffolding Identity, see [Scaffold identity into a Razor project with authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization).</span></span>

---

### <a name="examine-register"></a><span data-ttu-id="a5dc9-175">Esaminare il registro</span><span class="sxs-lookup"><span data-stu-id="a5dc9-175">Examine Register</span></span>

<span data-ttu-id="a5dc9-176">Quando un utente fa clic sul pulsante **registra** nella `Register` pagina, `RegisterModel.OnPostAsync` viene richiamata l'azione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-176">When a user clicks the **Register** button on the `Register` page, the `RegisterModel.OnPostAsync` action is invoked.</span></span> <span data-ttu-id="a5dc9-177">L'utente viene creato da [CreateAsync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_) nell' `_userManager` oggetto:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-177">The user is created by [CreateAsync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_) on the `_userManager` object:</span></span>

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=9)]

<!-- .NET 5 fixes this, see
https://github.com/dotnet/aspnetcore/blob/main/src/Identity/UI/src/Areas/Identity/Pages/V4/Account/RegisterConfirmation.cshtml.cs#L74-L77
-->
[!INCLUDE[](~/includes/disableVer.md)]

### <a name="log-in"></a><span data-ttu-id="a5dc9-178">Accesso</span><span class="sxs-lookup"><span data-stu-id="a5dc9-178">Log in</span></span>

<span data-ttu-id="a5dc9-179">Il modulo di accesso viene visualizzato quando:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-179">The Login form is displayed when:</span></span>

* <span data-ttu-id="a5dc9-180">Il collegamento **Accedi** è selezionato.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-180">The **Log in** link is selected.</span></span>
* <span data-ttu-id="a5dc9-181">Un utente tenta di accedere a una pagina con restrizioni che non è autorizzata ad accedere **o** quando non è stata autenticata dal sistema.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-181">A user attempts to access a restricted page that they aren't authorized to access **or** when they haven't been authenticated by the system.</span></span>

<span data-ttu-id="a5dc9-182">Quando viene inviato il modulo nella pagina di accesso, `OnPostAsync` viene chiamata l'azione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-182">When the form on the Login page is submitted, the `OnPostAsync` action is called.</span></span> <span data-ttu-id="a5dc9-183">`PasswordSignInAsync` viene chiamato sull' `_signInManager` oggetto.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-183">`PasswordSignInAsync` is called on the `_signInManager` object.</span></span>

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Login.cshtml.cs?name=snippet&highlight=10-11)]

<span data-ttu-id="a5dc9-184">Per informazioni su come prendere decisioni relative alle autorizzazioni, vedere <xref:security/authorization/introduction> .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-184">For information on how to make authorization decisions, see <xref:security/authorization/introduction>.</span></span>

### <a name="log-out"></a><span data-ttu-id="a5dc9-185">Effettuare la disconnessione</span><span class="sxs-lookup"><span data-stu-id="a5dc9-185">Log out</span></span>

<span data-ttu-id="a5dc9-186">Il collegamento **Disconnetti** richiama l' `LogoutModel.OnPost` azione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-186">The **Log out** link invokes the `LogoutModel.OnPost` action.</span></span> 

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Logout.cshtml.cs?highlight=36)]

<span data-ttu-id="a5dc9-187">Nel codice precedente, il codice `return RedirectToPage();` deve essere un reindirizzamento in modo che il browser esegua una nuova richiesta e venga aggiornata l'identità dell'utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-187">In the preceding code, the code `return RedirectToPage();` needs to be a redirect so that the browser performs a new request and the identity for the user gets updated.</span></span>

<span data-ttu-id="a5dc9-188">[SignOutAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync) Cancella le attestazioni dell'utente archiviate in un oggetto cookie .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-188">[SignOutAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync) clears the user's claims stored in a cookie.</span></span>

<span data-ttu-id="a5dc9-189">Post viene specificato nelle *pagine/Shared/_LoginPartial. cshtml*:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-189">Post is specified in the *Pages/Shared/_LoginPartial.cshtml*:</span></span>

[!code-cshtml[](identity/sample/WebApp3/Pages/Shared/_LoginPartial.cshtml?highlight=15)]

## <a name="test-identity"></a><span data-ttu-id="a5dc9-190">Test Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-190">Test Identity</span></span>

<span data-ttu-id="a5dc9-191">I modelli di progetto Web predefiniti consentono l'accesso anonimo alle Home page.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-191">The default web project templates allow anonymous access to the home pages.</span></span> <span data-ttu-id="a5dc9-192">Per eseguire il test Identity , aggiungere [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) :</span><span class="sxs-lookup"><span data-stu-id="a5dc9-192">To test Identity, add [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute):</span></span>

[!code-csharp[](identity/sample/WebApp3/Pages/Privacy.cshtml.cs?highlight=7)]

<span data-ttu-id="a5dc9-193">Se è stato eseguito l'accesso, disconnettersi. Eseguire l'app e selezionare il collegamento per la **privacy** .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-193">If you are signed in, sign out. Run the app and select the **Privacy** link.</span></span> <span data-ttu-id="a5dc9-194">Si verrà reindirizzati alla pagina di accesso.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-194">You are redirected to the login page.</span></span>

### <a name="explore-identity"></a><span data-ttu-id="a5dc9-195">Esplorare Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-195">Explore Identity</span></span>

<span data-ttu-id="a5dc9-196">Per esplorare Identity in modo più dettagliato:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-196">To explore Identity in more detail:</span></span>

* [<span data-ttu-id="a5dc9-197">Crea origine interfaccia utente con identità completa</span><span class="sxs-lookup"><span data-stu-id="a5dc9-197">Create full identity UI source</span></span>](xref:security/authentication/scaffold-identity#create-full-identity-ui-source)
* <span data-ttu-id="a5dc9-198">Esaminare l'origine di ogni pagina ed eseguire un'istruzione alla volta nel debugger.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-198">Examine the source of each page and step through the debugger.</span></span>

## <a name="identity-components"></a><span data-ttu-id="a5dc9-199">Identity Componenti</span><span class="sxs-lookup"><span data-stu-id="a5dc9-199">Identity Components</span></span>

<span data-ttu-id="a5dc9-200">Tutti i Identity pacchetti NuGet dipendenti da sono inclusi nel [framework condiviso ASP.NET Core](xref:aspnetcore-3.0#use-the-aspnet-core-shared-framework).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-200">All the Identity-dependent NuGet packages are included in the [ASP.NET Core shared framework](xref:aspnetcore-3.0#use-the-aspnet-core-shared-framework).</span></span>

<span data-ttu-id="a5dc9-201">Il pacchetto primario per Identity è [Microsoft. AspNetCore. Identity ](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/)</span><span class="sxs-lookup"><span data-stu-id="a5dc9-201">The primary package for Identity is [Microsoft.AspNetCore.Identity](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/).</span></span> <span data-ttu-id="a5dc9-202">Questo pacchetto contiene il set principale di interfacce per ASP.NET Core Identity ed è incluso in `Microsoft.AspNetCore.Identity.EntityFrameworkCore` .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-202">This package contains the core set of interfaces for ASP.NET Core Identity, and is included by `Microsoft.AspNetCore.Identity.EntityFrameworkCore`.</span></span>

## <a name="migrating-to-aspnet-core-identity"></a><span data-ttu-id="a5dc9-203">Migrazione a ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-203">Migrating to ASP.NET Core Identity</span></span>

<span data-ttu-id="a5dc9-204">Per ulteriori informazioni e istruzioni sulla migrazione dell' Identity archivio esistente, vedere [eseguire la migrazione Identity dell'autenticazione e ](xref:migration/identity).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-204">For more information and guidance on migrating your existing Identity store, see [Migrate Authentication and Identity](xref:migration/identity).</span></span>

## <a name="setting-password-strength"></a><span data-ttu-id="a5dc9-205">Impostazione della complessità della password</span><span class="sxs-lookup"><span data-stu-id="a5dc9-205">Setting password strength</span></span>

<span data-ttu-id="a5dc9-206">Vedere [configurazione](#pw) per un esempio che consente di impostare i requisiti minimi per le password.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-206">See [Configuration](#pw) for a sample that sets the minimum password requirements.</span></span>

## <a name="adddefaultidentity-and-addidentity"></a><span data-ttu-id="a5dc9-207">AddDefault Identity e AggiungiIdentity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-207">AddDefaultIdentity and AddIdentity</span></span>

<span data-ttu-id="a5dc9-208"><xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> è stato introdotto in ASP.NET Core 2,1.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-208"><xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> was introduced in ASP.NET Core 2.1.</span></span> <span data-ttu-id="a5dc9-209">`AddDefaultIdentity`La chiamata a è simile alla chiamata a quanto segue:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-209">Calling `AddDefaultIdentity` is similar to calling the following:</span></span>

* <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentity*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderExtensions.AddDefaultTokenProviders*>

<span data-ttu-id="a5dc9-210">Per altre informazioni, vedere [ Identity origine AddDefault](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-210">See [AddDefaultIdentity source](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) for more information.</span></span>

## <a name="prevent-publish-of-static-identity-assets"></a><span data-ttu-id="a5dc9-211">Impedisci la pubblicazione di Identity Asset statici</span><span class="sxs-lookup"><span data-stu-id="a5dc9-211">Prevent publish of static Identity assets</span></span>

<span data-ttu-id="a5dc9-212">Per evitare la pubblicazione Identity di risorse statiche (fogli di stile e file JavaScript per Identity l'interfaccia utente) nella radice Web, aggiungere la `ResolveStaticWebAssetsInputsDependsOn` proprietà e la `RemoveIdentityAssets` destinazione seguenti al file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-212">To prevent publishing static Identity assets (stylesheets and JavaScript files for Identity UI) to the web root, add the following `ResolveStaticWebAssetsInputsDependsOn` property and `RemoveIdentityAssets` target to the app's project file:</span></span>

```xml
<PropertyGroup>
  <ResolveStaticWebAssetsInputsDependsOn>RemoveIdentityAssets</ResolveStaticWebAssetsInputsDependsOn>
</PropertyGroup>

<Target Name="RemoveIdentityAssets">
  <ItemGroup>
    <StaticWebAsset Remove="@(StaticWebAsset)" Condition="%(SourceId) == 'Microsoft.AspNetCore.Identity.UI'" />
  </ItemGroup>
</Target>
```

<a name="next"></a>

## <a name="next-steps"></a><span data-ttu-id="a5dc9-213">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="a5dc9-213">Next Steps</span></span>

* <span data-ttu-id="a5dc9-214">[ASP.NET Core Identity codice sorgente](https://github.com/dotnet/aspnetcore/tree/main/src/Identity)</span><span class="sxs-lookup"><span data-stu-id="a5dc9-214">[ASP.NET Core Identity source code](https://github.com/dotnet/aspnetcore/tree/main/src/Identity)</span></span>
* <span data-ttu-id="a5dc9-215">Per informazioni sulla configurazione di using SQLite, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/5131) Identity .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-215">See [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/5131) for information on configuring Identity using SQLite.</span></span>
* [<span data-ttu-id="a5dc9-216">Configurare Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-216">Configure Identity</span></span>](xref:security/authentication/identity-configuration)
* <xref:security/authorization/secure-data>
* <xref:security/authentication/add-user-data>
* <xref:security/authentication/identity-enable-qrcodes>
* <xref:migration/identity>
* <xref:security/authentication/accconfirm>
* <xref:security/authentication/2fa>
* <xref:host-and-deploy/web-farm>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="a5dc9-217">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="a5dc9-217">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="a5dc9-218">ASP.NET Core Identity è un sistema di appartenenza che aggiunge la funzionalità di accesso alle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-218">ASP.NET Core Identity is a membership system that adds login functionality to ASP.NET Core apps.</span></span> <span data-ttu-id="a5dc9-219">Gli utenti possono creare un account con le informazioni di accesso archiviate in Identity o possono usare un provider di accesso esterno.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-219">Users can create an account with the login information stored in Identity or they can use an external login provider.</span></span> <span data-ttu-id="a5dc9-220">I provider di accesso esterni supportati includono [Facebook, Google, account Microsoft e Twitter](xref:security/authentication/social/index).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-220">Supported external login providers include [Facebook, Google, Microsoft Account, and Twitter](xref:security/authentication/social/index).</span></span>

<span data-ttu-id="a5dc9-221">Identity può essere configurato usando un database di SQL Server per archiviare i nomi utente, le password e i dati di profilo.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-221">Identity can be configured using a SQL Server database to store user names, passwords, and profile data.</span></span> <span data-ttu-id="a5dc9-222">In alternativa, è possibile usare un altro archivio permanente, ad esempio archiviazione tabelle di Azure.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-222">Alternatively, another persistent store can be used, for example, Azure Table Storage.</span></span>

<span data-ttu-id="a5dc9-223">Consente di [visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/identity/sample/src/ASPNETCore-IdentityDemoComplete/) ([come scaricare](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-223">[View or download the sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/identity/sample/src/ASPNETCore-IdentityDemoComplete/) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="a5dc9-224">In questo argomento si apprenderà come usare Identity per registrare, accedere e disconnettere un utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-224">In this topic, you learn how to use Identity to register, log in, and log out a user.</span></span> <span data-ttu-id="a5dc9-225">Per istruzioni più dettagliate sulla creazione di app che usano Identity , vedere la sezione passaggi successivi alla fine di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-225">For more detailed instructions about creating apps that use Identity, see the Next Steps section at the end of this article.</span></span>

<a name="adi"></a>

## <a name="adddefaultidentity-and-addidentity"></a><span data-ttu-id="a5dc9-226">AddDefault Identity e AggiungiIdentity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-226">AddDefaultIdentity and AddIdentity</span></span>

<span data-ttu-id="a5dc9-227"><xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> è stato introdotto in ASP.NET Core 2,1.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-227"><xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> was introduced in ASP.NET Core 2.1.</span></span> <span data-ttu-id="a5dc9-228">`AddDefaultIdentity`La chiamata a è simile alla chiamata a quanto segue:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-228">Calling `AddDefaultIdentity` is similar to calling the following:</span></span>

* <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentity*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderExtensions.AddDefaultTokenProviders*>

<span data-ttu-id="a5dc9-229">Per altre informazioni, vedere [ Identity origine AddDefault](https://github.com/dotnet/AspNetCore/blob/release/2.1/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-229">See [AddDefaultIdentity source](https://github.com/dotnet/AspNetCore/blob/release/2.1/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) for more information.</span></span>

## <a name="create-a-web-app-with-authentication"></a><span data-ttu-id="a5dc9-230">Creare un'app Web con l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="a5dc9-230">Create a Web app with authentication</span></span>

<span data-ttu-id="a5dc9-231">Creare un progetto di applicazione Web di ASP.NET Core con singoli account utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-231">Create an ASP.NET Core Web Application project with Individual User Accounts.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a5dc9-232">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a5dc9-232">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="a5dc9-233">Selezionare **file** > **nuovo** > **progetto**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-233">Select **File** > **New** > **Project**.</span></span>
* <span data-ttu-id="a5dc9-234">Selezionare **Applicazione Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-234">Select **ASP.NET Core Web Application**.</span></span> <span data-ttu-id="a5dc9-235">Denominare il progetto **app Web 1** in modo che abbia lo stesso spazio dei nomi del download del progetto.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-235">Name the project **WebApp1** to have the same namespace as the project download.</span></span> <span data-ttu-id="a5dc9-236">Fare clic su **OK**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-236">Click **OK**.</span></span>
* <span data-ttu-id="a5dc9-237">Selezionare un' **applicazione Web** ASP.NET Core, quindi selezionare **Modifica autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-237">Select an ASP.NET Core **Web Application**, then select **Change Authentication**.</span></span>
* <span data-ttu-id="a5dc9-238">Selezionare **singoli account utente** e fare clic su **OK**.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-238">Select **Individual User Accounts** and click **OK**.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a5dc9-239">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a5dc9-239">.NET Core CLI</span></span>](#tab/netcore-cli)

```dotnetcli
dotnet new webapp --auth Individual -o WebApp1
```

---

<span data-ttu-id="a5dc9-240">Il progetto generato fornisce [ASP.NET Core Identity](xref:security/authentication/identity) una [ Razor libreria di classi](xref:razor-pages/ui-class).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-240">The generated project provides [ASP.NET Core Identity](xref:security/authentication/identity) as a [Razor Class Library](xref:razor-pages/ui-class).</span></span> <span data-ttu-id="a5dc9-241">La Identity Razor libreria di classi espone gli endpoint con l' `Identity` area.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-241">The Identity Razor Class Library exposes endpoints with the `Identity` area.</span></span> <span data-ttu-id="a5dc9-242">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-242">For example:</span></span>

* <span data-ttu-id="a5dc9-243">/Identity/Account/Login</span><span class="sxs-lookup"><span data-stu-id="a5dc9-243">/Identity/Account/Login</span></span>
* <span data-ttu-id="a5dc9-244">/Identity/Account/Logout</span><span class="sxs-lookup"><span data-stu-id="a5dc9-244">/Identity/Account/Logout</span></span>
* <span data-ttu-id="a5dc9-245">/Identity/Account/Manage</span><span class="sxs-lookup"><span data-stu-id="a5dc9-245">/Identity/Account/Manage</span></span>

### <a name="apply-migrations"></a><span data-ttu-id="a5dc9-246">Applicare le migrazioni</span><span class="sxs-lookup"><span data-stu-id="a5dc9-246">Apply migrations</span></span>

<span data-ttu-id="a5dc9-247">Applicare le migrazioni per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-247">Apply the migrations to initialize the database.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a5dc9-248">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a5dc9-248">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="a5dc9-249">Eseguire il comando seguente nella console di gestione pacchetti (PMC):</span><span class="sxs-lookup"><span data-stu-id="a5dc9-249">Run the following command in the Package Manager Console (PMC):</span></span>

```powershell
Update-Database
```

# <a name="net-core-cli"></a>[<span data-ttu-id="a5dc9-250">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a5dc9-250">.NET Core CLI</span></span>](#tab/netcore-cli)

```dotnetcli
dotnet ef database update
```

---

### <a name="test-register-and-login"></a><span data-ttu-id="a5dc9-251">Registro di test e account di accesso</span><span class="sxs-lookup"><span data-stu-id="a5dc9-251">Test Register and Login</span></span>

<span data-ttu-id="a5dc9-252">Eseguire l'app e registrare un utente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-252">Run the app and register a user.</span></span> <span data-ttu-id="a5dc9-253">A seconda delle dimensioni dello schermo, potrebbe essere necessario selezionare l'interruttore di spostamento per visualizzare i collegamenti di **Registro** e di **accesso** .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-253">Depending on your screen size, you might need to select the navigation toggle button to see the **Register** and **Login** links.</span></span>

[!INCLUDE[](~/includes/view-identity-db.md)]

<a name="pw"></a>

### <a name="configure-identity-services"></a><span data-ttu-id="a5dc9-254">Configurare i Identity Servizi</span><span class="sxs-lookup"><span data-stu-id="a5dc9-254">Configure Identity services</span></span>

<span data-ttu-id="a5dc9-255">I servizi vengono aggiunti in `ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-255">Services are added in `ConfigureServices`.</span></span> <span data-ttu-id="a5dc9-256">Il modello tipico consiste nel chiamare tutti i metodi `Add{Service}` e quindi chiamare tutti i metodi `services.Configure{Service}`.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-256">The typical pattern is to call all the `Add{Service}` methods, and then call all the `services.Configure{Service}` methods.</span></span>

<span data-ttu-id="a5dc9-257">Il codice precedente Configura Identity con i valori di opzione predefiniti.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-257">The preceding code configures Identity with default option values.</span></span> <span data-ttu-id="a5dc9-258">I servizi vengono resi disponibili per l'applicazione tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-258">Services are made available to the app through [dependency injection](xref:fundamentals/dependency-injection).</span></span>

<span data-ttu-id="a5dc9-259">Identity viene abilitato chiamando [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-259">Identity is enabled by calling [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span></span> <span data-ttu-id="a5dc9-260">`UseAuthentication` aggiunge il [middleware](xref:fundamentals/middleware/index) di autenticazione alla pipeline della richiesta.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-260">`UseAuthentication` adds authentication [middleware](xref:fundamentals/middleware/index) to the request pipeline.</span></span>

[!code-csharp[](identity/sample/WebApp1/Startup.cs?name=snippet_configure&highlight=18)]

<span data-ttu-id="a5dc9-261">Per ulteriori informazioni, vedere la [ Identity classe Options](/dotnet/api/microsoft.aspnetcore.identity.identityoptions) e l' [avvio dell'applicazione](xref:fundamentals/startup).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-261">For more information, see the [IdentityOptions Class](/dotnet/api/microsoft.aspnetcore.identity.identityoptions) and [Application Startup](xref:fundamentals/startup).</span></span>

## <a name="scaffold-register-login-and-logout"></a><span data-ttu-id="a5dc9-262">Registrazione, accesso e disconnessione del patibolo</span><span class="sxs-lookup"><span data-stu-id="a5dc9-262">Scaffold Register, Login, and LogOut</span></span>

<span data-ttu-id="a5dc9-263">Seguire l' [identità del patibolo in un Razor progetto con](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization) le istruzioni di autorizzazione per generare il codice illustrato in questa sezione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-263">Follow the [Scaffold identity into a Razor project with authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization) instructions to generate the code shown in this section.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a5dc9-264">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a5dc9-264">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="a5dc9-265">Aggiungere i file di registro, di accesso e di disconnessione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-265">Add the Register, Login, and LogOut files.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a5dc9-266">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a5dc9-266">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="a5dc9-267">Se il progetto è stato creato con il nome **app Web 1**, eseguire i comandi seguenti.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-267">If you created the project with name **WebApp1**, run the following commands.</span></span> <span data-ttu-id="a5dc9-268">In caso contrario, utilizzare lo spazio dei nomi corretto per `ApplicationDbContext` :</span><span class="sxs-lookup"><span data-stu-id="a5dc9-268">Otherwise, use the correct namespace for the `ApplicationDbContext`:</span></span>

<span data-ttu-id="a5dc9-269">Quando si usa SQLite, `--useSqLite` è necessario specificare:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-269">When using SQLite, `--useSqLite` must be specified:</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc WebApp1.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout" --useSqLite
```

<span data-ttu-id="a5dc9-270">Con SQL Express, usare i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-270">With SQL Express, use the following commands:</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc WebApp1.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

<span data-ttu-id="a5dc9-271">PowerShell usa il punto e virgola come separatore di comandi.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-271">PowerShell uses semicolon as a command separator.</span></span> <span data-ttu-id="a5dc9-272">Quando si usa PowerShell, usare il carattere di escape per il punto e virgola nell'elenco dei file o inserire l'elenco di file tra virgolette doppie, come illustrato nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-272">When using PowerShell, escape the semicolons in the file list or put the file list in double quotes, as the preceding example shows.</span></span>

---

### <a name="examine-register"></a><span data-ttu-id="a5dc9-273">Esaminare il registro</span><span class="sxs-lookup"><span data-stu-id="a5dc9-273">Examine Register</span></span>

<span data-ttu-id="a5dc9-274">Quando un utente fa clic sul collegamento **Register** , `RegisterModel.OnPostAsync` viene richiamata l'azione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-274">When a user clicks the **Register** link, the `RegisterModel.OnPostAsync` action is invoked.</span></span> <span data-ttu-id="a5dc9-275">L'utente viene creato da [CreateAsync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_) nell' `_userManager` oggetto:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-275">The user is created by [CreateAsync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_) on the `_userManager` object:</span></span>

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=7)]

<span data-ttu-id="a5dc9-276">Se l'utente è stato creato correttamente, l'utente è connesso dalla chiamata a `_signInManager.SignInAsync` .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-276">If the user was created successfully, the user is logged in by the call to `_signInManager.SignInAsync`.</span></span>

<span data-ttu-id="a5dc9-277">**Nota:** Vedere la [conferma dell'account](xref:security/authentication/accconfirm#prevent-login-at-registration) per i passaggi per impedire l'accesso immediato alla registrazione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-277">**Note:** See [account confirmation](xref:security/authentication/accconfirm#prevent-login-at-registration) for steps to prevent immediate login at registration.</span></span>

### <a name="log-in"></a><span data-ttu-id="a5dc9-278">Accesso</span><span class="sxs-lookup"><span data-stu-id="a5dc9-278">Log in</span></span>

<span data-ttu-id="a5dc9-279">Il modulo di accesso viene visualizzato quando:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-279">The Login form is displayed when:</span></span>

* <span data-ttu-id="a5dc9-280">Il collegamento **Accedi** è selezionato.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-280">The **Log in** link is selected.</span></span>
* <span data-ttu-id="a5dc9-281">Un utente tenta di accedere a una pagina con restrizioni che non è autorizzata ad accedere **o** quando non è stata autenticata dal sistema.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-281">A user attempts to access a restricted page that they aren't authorized to access **or** when they haven't been authenticated by the system.</span></span>

<span data-ttu-id="a5dc9-282">Quando viene inviato il modulo nella pagina di accesso, `OnPostAsync` viene chiamata l'azione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-282">When the form on the Login page is submitted, the `OnPostAsync` action is called.</span></span> <span data-ttu-id="a5dc9-283">`PasswordSignInAsync` viene chiamato sull' `_signInManager` oggetto.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-283">`PasswordSignInAsync` is called on the `_signInManager` object.</span></span>

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Login.cshtml.cs?name=snippet&highlight=10-11)]

<span data-ttu-id="a5dc9-284">Per informazioni su come prendere decisioni relative alle autorizzazioni, vedere <xref:security/authorization/introduction> .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-284">For information on how to make authorization decisions, see <xref:security/authorization/introduction>.</span></span>

### <a name="log-out"></a><span data-ttu-id="a5dc9-285">Effettuare la disconnessione</span><span class="sxs-lookup"><span data-stu-id="a5dc9-285">Log out</span></span>

<span data-ttu-id="a5dc9-286">Il collegamento **Disconnetti** richiama l' `LogoutModel.OnPost` azione.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-286">The **Log out** link invokes the `LogoutModel.OnPost` action.</span></span> 

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Logout.cshtml.cs)]

<span data-ttu-id="a5dc9-287">[SignOutAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync) Cancella le attestazioni dell'utente archiviate in un oggetto cookie .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-287">[SignOutAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync) clears the user's claims stored in a cookie.</span></span>

<span data-ttu-id="a5dc9-288">Post viene specificato nelle *pagine/Shared/_LoginPartial. cshtml*:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-288">Post is specified in the *Pages/Shared/_LoginPartial.cshtml*:</span></span>

[!code-cshtml[](identity/sample/WebApp1/Pages/Shared/_LoginPartial.cshtml?highlight=16)]

## <a name="test-identity"></a><span data-ttu-id="a5dc9-289">Test Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-289">Test Identity</span></span>

<span data-ttu-id="a5dc9-290">I modelli di progetto Web predefiniti consentono l'accesso anonimo alle Home page.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-290">The default web project templates allow anonymous access to the home pages.</span></span> <span data-ttu-id="a5dc9-291">Per eseguire Identity il test, aggiungere [`[Authorize]`](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) alla pagina privacy.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-291">To test Identity, add [`[Authorize]`](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) to the Privacy page.</span></span>

[!code-csharp[](identity/sample/WebApp1/Pages/Privacy.cshtml.cs?highlight=7)]

<span data-ttu-id="a5dc9-292">Se è stato eseguito l'accesso, disconnettersi. Eseguire l'app e selezionare il collegamento per la **privacy** .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-292">If you are signed in, sign out. Run the app and select the **Privacy** link.</span></span> <span data-ttu-id="a5dc9-293">Si verrà reindirizzati alla pagina di accesso.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-293">You are redirected to the login page.</span></span>

### <a name="explore-identity"></a><span data-ttu-id="a5dc9-294">Esplorare Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-294">Explore Identity</span></span>

<span data-ttu-id="a5dc9-295">Per esplorare Identity in modo più dettagliato:</span><span class="sxs-lookup"><span data-stu-id="a5dc9-295">To explore Identity in more detail:</span></span>

* [<span data-ttu-id="a5dc9-296">Crea origine interfaccia utente con identità completa</span><span class="sxs-lookup"><span data-stu-id="a5dc9-296">Create full identity UI source</span></span>](xref:security/authentication/scaffold-identity#create-full-identity-ui-source)
* <span data-ttu-id="a5dc9-297">Esaminare l'origine di ogni pagina ed eseguire un'istruzione alla volta nel debugger.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-297">Examine the source of each page and step through the debugger.</span></span>

## <a name="identity-components"></a><span data-ttu-id="a5dc9-298">Identity Componenti</span><span class="sxs-lookup"><span data-stu-id="a5dc9-298">Identity Components</span></span>

<span data-ttu-id="a5dc9-299">Tutti i Identity pacchetti NuGet dipendenti sono inclusi nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-299">All the Identity dependent NuGet packages are included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app).</span></span>

<span data-ttu-id="a5dc9-300">Il pacchetto primario per Identity è [Microsoft. AspNetCore. Identity ](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/)</span><span class="sxs-lookup"><span data-stu-id="a5dc9-300">The primary package for Identity is [Microsoft.AspNetCore.Identity](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/).</span></span> <span data-ttu-id="a5dc9-301">Questo pacchetto contiene il set principale di interfacce per ASP.NET Core Identity ed è incluso in `Microsoft.AspNetCore.Identity.EntityFrameworkCore` .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-301">This package contains the core set of interfaces for ASP.NET Core Identity, and is included by `Microsoft.AspNetCore.Identity.EntityFrameworkCore`.</span></span>

## <a name="migrating-to-aspnet-core-identity"></a><span data-ttu-id="a5dc9-302">Migrazione a ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-302">Migrating to ASP.NET Core Identity</span></span>

<span data-ttu-id="a5dc9-303">Per ulteriori informazioni e istruzioni sulla migrazione dell' Identity archivio esistente, vedere [eseguire la migrazione Identity dell'autenticazione e ](xref:migration/identity).</span><span class="sxs-lookup"><span data-stu-id="a5dc9-303">For more information and guidance on migrating your existing Identity store, see [Migrate Authentication and Identity](xref:migration/identity).</span></span>

## <a name="setting-password-strength"></a><span data-ttu-id="a5dc9-304">Impostazione della complessità della password</span><span class="sxs-lookup"><span data-stu-id="a5dc9-304">Setting password strength</span></span>

<span data-ttu-id="a5dc9-305">Vedere [configurazione](#pw) per un esempio che consente di impostare i requisiti minimi per le password.</span><span class="sxs-lookup"><span data-stu-id="a5dc9-305">See [Configuration](#pw) for a sample that sets the minimum password requirements.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a5dc9-306">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="a5dc9-306">Next Steps</span></span>

* <span data-ttu-id="a5dc9-307">Per informazioni sulla configurazione di using SQLite, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/5131) Identity .</span><span class="sxs-lookup"><span data-stu-id="a5dc9-307">See [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/5131) for information on configuring Identity using SQLite.</span></span>
* [<span data-ttu-id="a5dc9-308">Configurare Identity</span><span class="sxs-lookup"><span data-stu-id="a5dc9-308">Configure Identity</span></span>](xref:security/authentication/identity-configuration)
* <xref:security/authorization/secure-data>
* <xref:security/authentication/add-user-data>
* <xref:security/authentication/identity-enable-qrcodes>
* <xref:migration/identity>
* <xref:security/authentication/accconfirm>
* <xref:security/authentication/2fa>
* <xref:host-and-deploy/web-farm>

::: moniker-end
