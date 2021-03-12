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
# <a name="add-download-and-delete-custom-user-data-to-identity-in-an-aspnet-core-project"></a><span data-ttu-id="f4cc3-104">Aggiungere, scaricare ed eliminare dati utente personalizzati Identity in un progetto ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="f4cc3-104">Add, download, and delete custom user data to Identity in an ASP.NET Core project</span></span>

<span data-ttu-id="f4cc3-105">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="f4cc3-105">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="f4cc3-106">Questo articolo illustra come:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-106">This article shows how to:</span></span>

* <span data-ttu-id="f4cc3-107">Aggiungere dati utente personalizzati a un'app Web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-107">Add custom user data to an ASP.NET Core web app.</span></span>
* <span data-ttu-id="f4cc3-108">Contrassegnare il modello di dati utente personalizzato con l' <xref:Microsoft.AspNetCore.Identity.PersonalDataAttribute> attributo in modo che sia disponibile automaticamente per il download e l'eliminazione.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-108">Mark the custom user data model with the <xref:Microsoft.AspNetCore.Identity.PersonalDataAttribute> attribute so it's automatically available for download and deletion.</span></span> <span data-ttu-id="f4cc3-109">La possibilità di scaricare ed eliminare i dati consente di soddisfare i requisiti di [GDPR](xref:security/gdpr) .</span><span class="sxs-lookup"><span data-stu-id="f4cc3-109">Making the data able to be downloaded and deleted helps meet [GDPR](xref:security/gdpr) requirements.</span></span>

<span data-ttu-id="f4cc3-110">Il progetto di esempio viene creato da un' Razor app Web di pagine, ma le istruzioni sono simili per un'app Web MVC ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-110">The project sample is created from a Razor Pages web app, but the instructions are similar for a ASP.NET Core MVC web app.</span></span>

<span data-ttu-id="f4cc3-111">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/add-user-data) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="f4cc3-111">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/add-user-data) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="f4cc3-112">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="f4cc3-112">Prerequisites</span></span>

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [](~/includes/3.0-SDK.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE [](~/includes/2.2-SDK.md)]

::: moniker-end

## <a name="create-a-razor-web-app"></a><span data-ttu-id="f4cc3-113">Creare un' Razor app Web</span><span class="sxs-lookup"><span data-stu-id="f4cc3-113">Create a Razor web app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="f4cc3-114">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="f4cc3-114">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-3.0"

* <span data-ttu-id="f4cc3-115">Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-115">From the Visual Studio **File** menu, select **New** > **Project**.</span></span> <span data-ttu-id="f4cc3-116">Denominare il progetto **app Web 1** se si desidera che corrisponda allo spazio dei nomi del codice di [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) .</span><span class="sxs-lookup"><span data-stu-id="f4cc3-116">Name the project **WebApp1** if you want to it match the namespace of the [download sample](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) code.</span></span>
* <span data-ttu-id="f4cc3-117">Selezionare **ASP.NET Core applicazione Web** > **OK**</span><span class="sxs-lookup"><span data-stu-id="f4cc3-117">Select **ASP.NET Core Web Application** > **OK**</span></span>
* <span data-ttu-id="f4cc3-118">Selezionare **ASP.NET Core 3,0** nell'elenco a discesa</span><span class="sxs-lookup"><span data-stu-id="f4cc3-118">Select **ASP.NET Core 3.0** in the dropdown</span></span>
* <span data-ttu-id="f4cc3-119">Selezionare l' **applicazione Web** > **OK**</span><span class="sxs-lookup"><span data-stu-id="f4cc3-119">Select **Web Application** > **OK**</span></span>
* <span data-ttu-id="f4cc3-120">Compilare ed eseguire il progetto.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-120">Build and run the project.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* <span data-ttu-id="f4cc3-121">Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-121">From the Visual Studio **File** menu, select **New** > **Project**.</span></span> <span data-ttu-id="f4cc3-122">Denominare il progetto **app Web 1** se si desidera che corrisponda allo spazio dei nomi del codice di [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) .</span><span class="sxs-lookup"><span data-stu-id="f4cc3-122">Name the project **WebApp1** if you want to it match the namespace of the [download sample](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) code.</span></span>
* <span data-ttu-id="f4cc3-123">Selezionare **ASP.NET Core applicazione Web** > **OK**</span><span class="sxs-lookup"><span data-stu-id="f4cc3-123">Select **ASP.NET Core Web Application** > **OK**</span></span>
* <span data-ttu-id="f4cc3-124">Selezionare **ASP.NET Core 2,2** nell'elenco a discesa</span><span class="sxs-lookup"><span data-stu-id="f4cc3-124">Select **ASP.NET Core 2.2** in the dropdown</span></span>
* <span data-ttu-id="f4cc3-125">Selezionare l' **applicazione Web** > **OK**</span><span class="sxs-lookup"><span data-stu-id="f4cc3-125">Select **Web Application** > **OK**</span></span>
* <span data-ttu-id="f4cc3-126">Compilare ed eseguire il progetto.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-126">Build and run the project.</span></span>

::: moniker-end


# <a name="net-core-cli"></a>[<span data-ttu-id="f4cc3-127">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="f4cc3-127">.NET Core CLI</span></span>](#tab/netcore-cli)

```dotnetcli
dotnet new webapp -o WebApp1
```

---

## <a name="run-the-identity-scaffolder"></a><span data-ttu-id="f4cc3-128">Eseguire l' Identity impalcatura</span><span class="sxs-lookup"><span data-stu-id="f4cc3-128">Run the Identity scaffolder</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="f4cc3-129">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="f4cc3-129">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="f4cc3-130">Da **Esplora soluzioni**, fare clic con il pulsante destro del mouse sul progetto > **Aggiungi**  >  **nuovo elemento con impalcatura**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-130">From **Solution Explorer**, right-click on the project > **Add** > **New Scaffolded Item**.</span></span>
* <span data-ttu-id="f4cc3-131">Dal riquadro sinistro della finestra di dialogo **Aggiungi impalcatura** selezionare **Identity**  >  **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-131">From the left pane of the **Add Scaffold** dialog, select **Identity** > **Add**.</span></span>
* <span data-ttu-id="f4cc3-132">Nella finestra di dialogo **Aggiungi Identity** sono disponibili le opzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-132">In the **Add Identity** dialog, the following options:</span></span>
  * <span data-ttu-id="f4cc3-133">Selezionare il file di layout esistente  *~/Pages/Shared/_Layout. cshtml*</span><span class="sxs-lookup"><span data-stu-id="f4cc3-133">Select the existing layout  file  *~/Pages/Shared/_Layout.cshtml*</span></span>
  * <span data-ttu-id="f4cc3-134">Selezionare i seguenti file di cui eseguire l'override:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-134">Select the following files to override:</span></span>
    * <span data-ttu-id="f4cc3-135">**Account/registro**</span><span class="sxs-lookup"><span data-stu-id="f4cc3-135">**Account/Register**</span></span>
    * <span data-ttu-id="f4cc3-136">**Account/Gestione/indice**</span><span class="sxs-lookup"><span data-stu-id="f4cc3-136">**Account/Manage/Index**</span></span>
  * <span data-ttu-id="f4cc3-137">Selezionare il **+** pulsante per creare una nuova **classe del contesto dati**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-137">Select the **+** button to create a new **Data context class**.</span></span> <span data-ttu-id="f4cc3-138">Accettare il tipo (**app Web 1. Models. WebApp1Context** se il progetto è denominato **app Web 1**).</span><span class="sxs-lookup"><span data-stu-id="f4cc3-138">Accept the type (**WebApp1.Models.WebApp1Context** if the project is named **WebApp1**).</span></span>
  * <span data-ttu-id="f4cc3-139">Selezionare il **+** pulsante per creare una nuova **classe utente**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-139">Select the **+** button to create a new **User class**.</span></span> <span data-ttu-id="f4cc3-140">Accettare il tipo (**WebApp1User** se il progetto è denominato **app Web 1**) > **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-140">Accept the type (**WebApp1User** if the project is named **WebApp1**) > **Add**.</span></span>
* <span data-ttu-id="f4cc3-141">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-141">Select **Add**.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="f4cc3-142">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="f4cc3-142">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="f4cc3-143">Se in precedenza non è stato installato il ASP.NET Core impalcatura, installarlo ora:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-143">If you have not previously installed the ASP.NET Core scaffolder, install it now:</span></span>

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

<span data-ttu-id="f4cc3-144">Aggiungere un riferimento al pacchetto [Microsoft. VisualStudio. Web. CodeGeneration. Design](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/) al file di progetto (con estensione csproj).</span><span class="sxs-lookup"><span data-stu-id="f4cc3-144">Add a package reference to [Microsoft.VisualStudio.Web.CodeGeneration.Design](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/) to the project (.csproj) file.</span></span> <span data-ttu-id="f4cc3-145">Eseguire il comando seguente nella directory del progetto:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-145">Run the following command in the project directory:</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
```

<span data-ttu-id="f4cc3-146">Eseguire il comando seguente per elencare le Identity Opzioni dell'impalcatura:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-146">Run the following command to list the Identity scaffolder options:</span></span>

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

<span data-ttu-id="f4cc3-147">Nella cartella del progetto eseguire l' Identity impalcatura:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-147">In the project folder, run the Identity scaffolder:</span></span>

```dotnetcli
dotnet aspnet-codegenerator identity -u WebApp1User -fi Account.Register;Account.Manage.Index
```

---

<span data-ttu-id="f4cc3-148">Seguire le istruzioni in [Migrations, UseAuthentication e layout](xref:security/authentication/scaffold-identity#efm) per eseguire la procedura seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-148">Follow the instruction in [Migrations, UseAuthentication, and layout](xref:security/authentication/scaffold-identity#efm) to perform the following steps:</span></span>

* <span data-ttu-id="f4cc3-149">Creare una migrazione e aggiornare il database.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-149">Create a migration and update the database.</span></span>
* <span data-ttu-id="f4cc3-150">Aggiunta di `UseAuthentication` a `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-150">Add `UseAuthentication` to `Startup.Configure`.</span></span>
* <span data-ttu-id="f4cc3-151">Aggiungere `<partial name="_LoginPartial" />` al file di layout.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-151">Add `<partial name="_LoginPartial" />` to the layout file.</span></span>
* <span data-ttu-id="f4cc3-152">Eseguire il test dell'app:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-152">Test the app:</span></span>
  * <span data-ttu-id="f4cc3-153">Registrare un utente</span><span class="sxs-lookup"><span data-stu-id="f4cc3-153">Register a user</span></span>
  * <span data-ttu-id="f4cc3-154">Selezionare il nuovo nome utente (accanto al collegamento di **disconnessione** ).</span><span class="sxs-lookup"><span data-stu-id="f4cc3-154">Select the new user name (next to the **Logout** link).</span></span> <span data-ttu-id="f4cc3-155">Potrebbe essere necessario espandere la finestra o selezionare l'icona della barra di spostamento per visualizzare il nome utente e altri collegamenti.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-155">You might need to expand the window or select the navigation bar icon to show the user name and other links.</span></span>
  * <span data-ttu-id="f4cc3-156">Selezionare la scheda **dati personali** .</span><span class="sxs-lookup"><span data-stu-id="f4cc3-156">Select the **Personal Data** tab.</span></span>
  * <span data-ttu-id="f4cc3-157">Selezionare il pulsante **download** ed esaminare il *PersonalData.jssu* file.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-157">Select the **Download** button and examined the *PersonalData.json* file.</span></span>
  * <span data-ttu-id="f4cc3-158">Testare il pulsante **Elimina per eliminare** l'utente che ha eseguito l'accesso.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-158">Test the **Delete** button, which deletes the logged on user.</span></span>

## <a name="add-custom-user-data-to-the-identity-db"></a><span data-ttu-id="f4cc3-159">Aggiungere dati utente personalizzati al Identity database</span><span class="sxs-lookup"><span data-stu-id="f4cc3-159">Add custom user data to the Identity DB</span></span>

<span data-ttu-id="f4cc3-160">Aggiornare la `IdentityUser` classe derivata con proprietà personalizzate.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-160">Update the `IdentityUser` derived class with custom properties.</span></span> <span data-ttu-id="f4cc3-161">Se è stato denominato il progetto app Web 1, il file è denominato *aree/ Identity /Data/WebApp1User.cs*.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-161">If you named the project WebApp1, the file is named *Areas/Identity/Data/WebApp1User.cs*.</span></span> <span data-ttu-id="f4cc3-162">Aggiornare il file con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-162">Update the file with the following code:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

<span data-ttu-id="f4cc3-163">Le proprietà con l'attributo [personaldata](/dotnet/api/microsoft.aspnetcore.identity.personaldataattribute) sono:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-163">Properties with the [PersonalData](/dotnet/api/microsoft.aspnetcore.identity.personaldataattribute) attribute are:</span></span>

* <span data-ttu-id="f4cc3-164">Viene eliminato quando viene chiamata la pagina *aree/ Identity /pages/account/Manage/DeletePersonalData.cshtml* Razor `UserManager.Delete` .</span><span class="sxs-lookup"><span data-stu-id="f4cc3-164">Deleted when the *Areas/Identity/Pages/Account/Manage/DeletePersonalData.cshtml* Razor Page calls `UserManager.Delete`.</span></span>
* <span data-ttu-id="f4cc3-165">Incluso nei dati scaricati dalla pagina *aree/ Identity /pages/account/Manage/DownloadPersonalData.cshtml* Razor .</span><span class="sxs-lookup"><span data-stu-id="f4cc3-165">Included in the downloaded data by the *Areas/Identity/Pages/Account/Manage/DownloadPersonalData.cshtml* Razor Page.</span></span>

### <a name="update-the-accountmanageindexcshtml-page"></a><span data-ttu-id="f4cc3-166">Aggiornare la pagina account/Manage/index. cshtml</span><span class="sxs-lookup"><span data-stu-id="f4cc3-166">Update the Account/Manage/Index.cshtml page</span></span>

<span data-ttu-id="f4cc3-167">Aggiornare `InputModel` in *areas/ Identity /pages/account/Manage/index.cshtml.cs* con il codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-167">Update the `InputModel` in *Areas/Identity/Pages/Account/Manage/Index.cshtml.cs* with the following highlighted code:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=24-32,48-49,96-104,106)]

<span data-ttu-id="f4cc3-168">Aggiornare le *aree/ Identity /pages/account/Manage/index.cshtml* con il markup evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-168">Update the *Areas/Identity/Pages/Account/Manage/Index.cshtml* with the following highlighted markup:</span></span>

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=18-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=28-36,63-64,98-106,119)]

<span data-ttu-id="f4cc3-169">Aggiornare le *aree/ Identity /pages/account/Manage/index.cshtml* con il markup evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-169">Update the *Areas/Identity/Pages/Account/Manage/Index.cshtml* with the following highlighted markup:</span></span>

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=35-42)]

::: moniker-end

### <a name="update-the-accountregistercshtml-page"></a><span data-ttu-id="f4cc3-170">Aggiornare la pagina account/Register. cshtml</span><span class="sxs-lookup"><span data-stu-id="f4cc3-170">Update the Account/Register.cshtml page</span></span>

<span data-ttu-id="f4cc3-171">Aggiornare `InputModel` in *areas/ Identity /pages/account/Register.cshtml.cs* con il codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-171">Update the `InputModel` in *Areas/Identity/Pages/Account/Register.cshtml.cs* with the following highlighted code:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=30-38,70-71)]

<span data-ttu-id="f4cc3-172">Aggiornare le *aree/ Identity /pages/account/Register.cshtml* con il markup evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-172">Update the *Areas/Identity/Pages/Account/Register.cshtml* with the following highlighted markup:</span></span>

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=28-36,67,66)]

<span data-ttu-id="f4cc3-173">Aggiornare le *aree/ Identity /pages/account/Register.cshtml* con il markup evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-173">Update the *Areas/Identity/Pages/Account/Register.cshtml* with the following highlighted markup:</span></span>

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end


<span data-ttu-id="f4cc3-174">Compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-174">Build the project.</span></span>

### <a name="add-a-migration-for-the-custom-user-data"></a><span data-ttu-id="f4cc3-175">Aggiungere una migrazione per i dati utente personalizzati</span><span class="sxs-lookup"><span data-stu-id="f4cc3-175">Add a migration for the custom user data</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="f4cc3-176">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="f4cc3-176">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="f4cc3-177">Nella **console di gestione pacchetti** di Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-177">In the Visual Studio **Package Manager Console**:</span></span>

```powershell
Add-Migration CustomUserData
Update-Database
```

# <a name="net-core-cli"></a>[<span data-ttu-id="f4cc3-178">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="f4cc3-178">.NET Core CLI</span></span>](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CustomUserData
dotnet ef database update
```

---

## <a name="test-create-view-download-delete-custom-user-data"></a><span data-ttu-id="f4cc3-179">Test creare, visualizzare, scaricare, eliminare dati utente personalizzati</span><span class="sxs-lookup"><span data-stu-id="f4cc3-179">Test create, view, download, delete custom user data</span></span>

<span data-ttu-id="f4cc3-180">Eseguire il test dell'app:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-180">Test the app:</span></span>

* <span data-ttu-id="f4cc3-181">Registrare un nuovo utente.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-181">Register a new user.</span></span>
* <span data-ttu-id="f4cc3-182">Visualizzare i dati utente personalizzati nella `/Identity/Account/Manage` pagina.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-182">View the custom user data on the `/Identity/Account/Manage` page.</span></span>
* <span data-ttu-id="f4cc3-183">Scaricare e visualizzare i dati personali degli utenti dalla `/Identity/Account/Manage/PersonalData` pagina.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-183">Download and view the users personal data from the `/Identity/Account/Manage/PersonalData` page.</span></span>

## <a name="add-claims-to-identity-using-iuserclaimsprincipalfactoryapplicationuser"></a><span data-ttu-id="f4cc3-184">Aggiungere attestazioni all' Identity uso di IUserClaimsPrincipalFactory<ApplicationUser></span><span class="sxs-lookup"><span data-stu-id="f4cc3-184">Add claims to Identity using IUserClaimsPrincipalFactory<ApplicationUser></span></span>

> [!NOTE]
> <span data-ttu-id="f4cc3-185">Questa sezione non è un'estensione dell'esercitazione precedente.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-185">This section isn't an extension of the previous tutorial.</span></span> <span data-ttu-id="f4cc3-186">Per applicare i passaggi seguenti all'app compilata con l'esercitazione, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/18797).</span><span class="sxs-lookup"><span data-stu-id="f4cc3-186">To apply the following steps to the app built using the tutorial, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/18797).</span></span>

<span data-ttu-id="f4cc3-187">È possibile aggiungere attestazioni aggiuntive a utilizzando ASP.NET Core Identity l' `IUserClaimsPrincipalFactory<T>` interfaccia.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-187">Additional claims can be added to ASP.NET Core Identity by using the `IUserClaimsPrincipalFactory<T>` interface.</span></span> <span data-ttu-id="f4cc3-188">Questa classe può essere aggiunta all'app nel `Startup.ConfigureServices` metodo.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-188">This class can be added to the app in the `Startup.ConfigureServices` method.</span></span> <span data-ttu-id="f4cc3-189">Aggiungere l'implementazione personalizzata della classe come indicato di seguito:</span><span class="sxs-lookup"><span data-stu-id="f4cc3-189">Add the custom implementation of the class as follows:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddScoped<IUserClaimsPrincipalFactory<ApplicationUser>, 
        AdditionalUserClaimsPrincipalFactory>();
```

<span data-ttu-id="f4cc3-190">Il codice dimostrativo usa la `ApplicationUser` classe.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-190">The demo code uses the `ApplicationUser` class.</span></span> <span data-ttu-id="f4cc3-191">Questa classe aggiunge una `IsAdmin` proprietà che viene utilizzata per aggiungere l'attestazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-191">This class adds an `IsAdmin` property which is used to add the additional claim.</span></span>

```csharp
public class ApplicationUser : IdentityUser
{
    public bool IsAdmin { get; set; }
}
```

<span data-ttu-id="f4cc3-192">L'oggetto `AdditionalUserClaimsPrincipalFactory` implementa l'interfaccia `UserClaimsPrincipalFactory`.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-192">The `AdditionalUserClaimsPrincipalFactory` implements the `UserClaimsPrincipalFactory` interface.</span></span> <span data-ttu-id="f4cc3-193">Viene aggiunta una nuova attestazione di ruolo a `ClaimsPrincipal` .</span><span class="sxs-lookup"><span data-stu-id="f4cc3-193">A new role claim is added to the `ClaimsPrincipal`.</span></span>

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

<span data-ttu-id="f4cc3-194">L'attestazione aggiuntiva può quindi essere usata nell'app.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-194">The additional claim can then be used in the app.</span></span> <span data-ttu-id="f4cc3-195">In una Razor pagina, l' `IAuthorizationService` istanza di può essere utilizzata per accedere al valore dell'attestazione.</span><span class="sxs-lookup"><span data-stu-id="f4cc3-195">In a Razor Page, the `IAuthorizationService` instance can be used to access the claim value.</span></span>

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
