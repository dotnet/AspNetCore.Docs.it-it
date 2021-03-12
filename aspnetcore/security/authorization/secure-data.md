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
# <a name="create-an-aspnet-core-web-app-with-user-data-protected-by-authorization"></a><span data-ttu-id="d73b2-104">Creare un'app Web di ASP.NET Core con i dati utente protetti dall'autorizzazione</span><span class="sxs-lookup"><span data-stu-id="d73b2-104">Create an ASP.NET Core web app with user data protected by authorization</span></span>

<span data-ttu-id="d73b2-105">Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Joe Audette](https://twitter.com/joeaudette)</span><span class="sxs-lookup"><span data-stu-id="d73b2-105">By [Rick Anderson](https://twitter.com/RickAndMSFT) and [Joe Audette](https://twitter.com/joeaudette)</span></span>

::: moniker range="= aspnetcore-2.0"

<span data-ttu-id="d73b2-106">Vedi [questo PDF](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_July16_18.pdf)</span><span class="sxs-lookup"><span data-stu-id="d73b2-106">See [this pdf](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_July16_18.pdf)</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="d73b2-107">Questa esercitazione illustra come creare un'app Web di ASP.NET Core con i dati utente protetti dall'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-107">This tutorial shows how to create an ASP.NET Core web app with user data protected by authorization.</span></span> <span data-ttu-id="d73b2-108">Viene visualizzato un elenco di contatti creati dagli utenti autenticati (registrati).</span><span class="sxs-lookup"><span data-stu-id="d73b2-108">It displays a list of contacts that authenticated (registered) users have created.</span></span> <span data-ttu-id="d73b2-109">Sono disponibili tre gruppi di sicurezza:</span><span class="sxs-lookup"><span data-stu-id="d73b2-109">There are three security groups:</span></span>

* <span data-ttu-id="d73b2-110">**Gli utenti registrati** possono visualizzare tutti i dati approvati e possono modificare/eliminare i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-110">**Registered users** can view all the approved data and can edit/delete their own data.</span></span>
* <span data-ttu-id="d73b2-111">I **responsabili** possono approvare o rifiutare i dati di contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-111">**Managers** can approve or reject contact data.</span></span> <span data-ttu-id="d73b2-112">Solo i contatti approvati sono visibili agli utenti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-112">Only approved contacts are visible to users.</span></span>
* <span data-ttu-id="d73b2-113">Gli **amministratori** possono approvare/rifiutare e modificare/eliminare i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-113">**Administrators** can approve/reject and edit/delete any data.</span></span>

<span data-ttu-id="d73b2-114">Le immagini in questo documento non corrispondono esattamente ai modelli più recenti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-114">The images in this document don't exactly match the latest templates.</span></span>

<span data-ttu-id="d73b2-115">Nell'immagine seguente, l'utente Rick ( `rick@example.com` ) ha eseguito l'accesso.</span><span class="sxs-lookup"><span data-stu-id="d73b2-115">In the following image, user Rick (`rick@example.com`) is signed in.</span></span> <span data-ttu-id="d73b2-116">Rick può visualizzare solo i contatti approvati e **modificare** / **Delete** / **Crea nuovi** collegamenti per i contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-116">Rick can only view approved contacts and **Edit**/**Delete**/**Create New** links for his contacts.</span></span> <span data-ttu-id="d73b2-117">Solo l'ultimo record, creato da Rick, Visualizza i collegamenti **modifica** ed **Elimina** .</span><span class="sxs-lookup"><span data-stu-id="d73b2-117">Only the last record, created by Rick, displays **Edit** and **Delete** links.</span></span> <span data-ttu-id="d73b2-118">Gli altri utenti non vedranno l'ultimo record fino a quando un responsabile o un amministratore non modifica lo stato in "approvato".</span><span class="sxs-lookup"><span data-stu-id="d73b2-118">Other users won't see the last record until a manager or administrator changes the status to "Approved".</span></span>

![Screenshot che illustra l'accesso a Rick](secure-data/_static/rick.png)

<span data-ttu-id="d73b2-120">Nell'immagine seguente, `manager@contoso.com` è stato eseguito l'accesso e nel ruolo del Manager:</span><span class="sxs-lookup"><span data-stu-id="d73b2-120">In the following image, `manager@contoso.com` is signed in and in the manager's role:</span></span>

![Screenshot che mostra l' manager@contoso.com accesso](secure-data/_static/manager1.png)

<span data-ttu-id="d73b2-122">Nella figura seguente viene illustrata la visualizzazione dettagli Manager di un contatto:</span><span class="sxs-lookup"><span data-stu-id="d73b2-122">The following image shows the managers details view of a contact:</span></span>

![Visualizzazione del contatto da un responsabile](secure-data/_static/manager.png)

<span data-ttu-id="d73b2-124">I pulsanti **approva** e **rifiuta** vengono visualizzati solo per Manager e amministratori.</span><span class="sxs-lookup"><span data-stu-id="d73b2-124">The **Approve** and **Reject** buttons are only displayed for managers and administrators.</span></span>

<span data-ttu-id="d73b2-125">Nell'immagine seguente, `admin@contoso.com` è stato eseguito l'accesso e il ruolo dell'amministratore:</span><span class="sxs-lookup"><span data-stu-id="d73b2-125">In the following image, `admin@contoso.com` is signed in and in the administrator's role:</span></span>

![Screenshot che mostra l' admin@contoso.com accesso](secure-data/_static/admin.png)

<span data-ttu-id="d73b2-127">L'amministratore dispone di tutti i privilegi.</span><span class="sxs-lookup"><span data-stu-id="d73b2-127">The administrator has all privileges.</span></span> <span data-ttu-id="d73b2-128">Può leggere, modificare ed eliminare qualsiasi contatto e modificare lo stato dei contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-128">She can read/edit/delete any contact and change the status of contacts.</span></span>

<span data-ttu-id="d73b2-129">L'app è stata creata mediante l' [impalcatura](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) del `Contact` modello seguente:</span><span class="sxs-lookup"><span data-stu-id="d73b2-129">The app was created by [scaffolding](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) the following `Contact` model:</span></span>

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

<span data-ttu-id="d73b2-130">L'esempio contiene i gestori di autorizzazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-130">The sample contains the following authorization handlers:</span></span>

* <span data-ttu-id="d73b2-131">`ContactIsOwnerAuthorizationHandler`: Assicura che un utente possa modificare solo i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-131">`ContactIsOwnerAuthorizationHandler`: Ensures that a user can only edit their data.</span></span>
* <span data-ttu-id="d73b2-132">`ContactManagerAuthorizationHandler`: Consente ai responsabili di approvare o rifiutare i contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-132">`ContactManagerAuthorizationHandler`: Allows managers to approve or reject contacts.</span></span>
* <span data-ttu-id="d73b2-133">`ContactAdministratorsAuthorizationHandler`: Consente agli amministratori di approvare o rifiutare contatti e di modificare/eliminare contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-133">`ContactAdministratorsAuthorizationHandler`: Allows administrators to approve or reject contacts and to edit/delete contacts.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d73b2-134">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="d73b2-134">Prerequisites</span></span>

<span data-ttu-id="d73b2-135">Questa esercitazione è avanzata.</span><span class="sxs-lookup"><span data-stu-id="d73b2-135">This tutorial is advanced.</span></span> <span data-ttu-id="d73b2-136">È necessario avere familiarità con:</span><span class="sxs-lookup"><span data-stu-id="d73b2-136">You should be familiar with:</span></span>

* [<span data-ttu-id="d73b2-137">ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="d73b2-137">ASP.NET Core</span></span>](xref:tutorials/first-mvc-app/start-mvc)
* [<span data-ttu-id="d73b2-138">autenticazione</span><span class="sxs-lookup"><span data-stu-id="d73b2-138">Authentication</span></span>](xref:security/authentication/identity)
* [<span data-ttu-id="d73b2-139">Conferma account e recupero password</span><span class="sxs-lookup"><span data-stu-id="d73b2-139">Account Confirmation and Password Recovery</span></span>](xref:security/authentication/accconfirm)
* [<span data-ttu-id="d73b2-140">Autorizzazione</span><span class="sxs-lookup"><span data-stu-id="d73b2-140">Authorization</span></span>](xref:security/authorization/introduction)
* [<span data-ttu-id="d73b2-141">Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="d73b2-141">Entity Framework Core</span></span>](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a><span data-ttu-id="d73b2-142">App iniziale e completata</span><span class="sxs-lookup"><span data-stu-id="d73b2-142">The starter and completed app</span></span>

<span data-ttu-id="d73b2-143">[Scaricare](xref:index#how-to-download-a-sample) l'app [completata](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples) .</span><span class="sxs-lookup"><span data-stu-id="d73b2-143">[Download](xref:index#how-to-download-a-sample) the [completed](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples) app.</span></span> <span data-ttu-id="d73b2-144">[Testare](#test-the-completed-app) l'app completata in modo da acquisire familiarità con le funzionalità di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="d73b2-144">[Test](#test-the-completed-app) the completed app so you become familiar with its security features.</span></span>

### <a name="the-starter-app"></a><span data-ttu-id="d73b2-145">App iniziale</span><span class="sxs-lookup"><span data-stu-id="d73b2-145">The starter app</span></span>

<span data-ttu-id="d73b2-146">[Scaricare](xref:index#how-to-download-a-sample) l'app [Starter](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/) .</span><span class="sxs-lookup"><span data-stu-id="d73b2-146">[Download](xref:index#how-to-download-a-sample) the [starter](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/) app.</span></span>

<span data-ttu-id="d73b2-147">Eseguire l'app, toccare il collegamento **ContactManager** e verificare che sia possibile creare, modificare ed eliminare un contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-147">Run the app, tap the **ContactManager** link, and verify you can create, edit, and delete a contact.</span></span> <span data-ttu-id="d73b2-148">Per creare l'app iniziale, vedere [creare l'app iniziale](#create-the-starter-app).</span><span class="sxs-lookup"><span data-stu-id="d73b2-148">To create the starter app, see [Create the starter app](#create-the-starter-app).</span></span>

## <a name="secure-user-data"></a><span data-ttu-id="d73b2-149">Proteggere i dati utente</span><span class="sxs-lookup"><span data-stu-id="d73b2-149">Secure user data</span></span>

<span data-ttu-id="d73b2-150">Le sezioni seguenti includono tutti i passaggi principali per creare l'app Secure User Data.</span><span class="sxs-lookup"><span data-stu-id="d73b2-150">The following sections have all the major steps to create the secure user data app.</span></span> <span data-ttu-id="d73b2-151">Può risultare utile fare riferimento al progetto completato.</span><span class="sxs-lookup"><span data-stu-id="d73b2-151">You may find it helpful to refer to the completed project.</span></span>

### <a name="tie-the-contact-data-to-the-user"></a><span data-ttu-id="d73b2-152">Associare i dati di contatto all'utente</span><span class="sxs-lookup"><span data-stu-id="d73b2-152">Tie the contact data to the user</span></span>

<span data-ttu-id="d73b2-153">Usare l' [Identity](xref:security/authentication/identity) ID utente ASP.NET per assicurarsi che gli utenti possano modificare i dati, ma non i dati di altri utenti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-153">Use the ASP.NET [Identity](xref:security/authentication/identity) user ID to ensure users can edit their data, but not other users data.</span></span> <span data-ttu-id="d73b2-154">Aggiungere `OwnerID` e `ContactStatus` al `Contact` modello:</span><span class="sxs-lookup"><span data-stu-id="d73b2-154">Add `OwnerID` and `ContactStatus` to the `Contact` model:</span></span>

[!code-csharp[](secure-data/samples/final3/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

<span data-ttu-id="d73b2-155">`OwnerID` ID dell'utente della `AspNetUser` tabella nel [Identity](xref:security/authentication/identity) database.</span><span class="sxs-lookup"><span data-stu-id="d73b2-155">`OwnerID` is the user's ID from the `AspNetUser` table in the [Identity](xref:security/authentication/identity) database.</span></span> <span data-ttu-id="d73b2-156">Il `Status` campo determina se un contatto è visualizzabile dagli utenti generali.</span><span class="sxs-lookup"><span data-stu-id="d73b2-156">The `Status` field determines if a contact is viewable by general users.</span></span>

<span data-ttu-id="d73b2-157">Creare una nuova migrazione e aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="d73b2-157">Create a new migration and update the database:</span></span>

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-identity"></a><span data-ttu-id="d73b2-158">Aggiungere servizi ruolo a Identity</span><span class="sxs-lookup"><span data-stu-id="d73b2-158">Add Role services to Identity</span></span>

<span data-ttu-id="d73b2-159">Aggiungere [Aggiungi ruoli](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) per aggiungere servizi ruolo:</span><span class="sxs-lookup"><span data-stu-id="d73b2-159">Append [AddRoles](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) to add Role services:</span></span>

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet2&highlight=9)]

<a name="rau"></a>

### <a name="require-authenticated-users"></a><span data-ttu-id="d73b2-160">Richiedi utenti autenticati</span><span class="sxs-lookup"><span data-stu-id="d73b2-160">Require authenticated users</span></span>

<span data-ttu-id="d73b2-161">Impostare i criteri di autenticazione di fallback per richiedere l'autenticazione degli utenti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-161">Set the fallback authentication policy to require users to be authenticated:</span></span>

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet&highlight=13-99)]

<span data-ttu-id="d73b2-162">Il codice evidenziato precedente imposta i [criteri di autenticazione di fallback](xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy).</span><span class="sxs-lookup"><span data-stu-id="d73b2-162">The preceding highlighted code sets the [fallback authentication policy](xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy).</span></span> <span data-ttu-id="d73b2-163">Per i criteri di autenticazione di fallback è necessario autenticare ***tutti*** gli utenti, ad eccezione di Razor pagine, controller o metodi di azione con un attributo di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-163">The fallback authentication policy requires ***all*** users to be authenticated, except for Razor Pages, controllers, or action methods with an authentication attribute.</span></span> <span data-ttu-id="d73b2-164">Ad esempio, Razor pagine, controller o metodi di azione con `[AllowAnonymous]` o `[Authorize(PolicyName="MyPolicy")]` utilizzano l'attributo di autenticazione applicato anziché i criteri di autenticazione di fallback.</span><span class="sxs-lookup"><span data-stu-id="d73b2-164">For example, Razor Pages, controllers, or action methods with `[AllowAnonymous]` or `[Authorize(PolicyName="MyPolicy")]` use the applied authentication attribute rather than the fallback authentication policy.</span></span>

<span data-ttu-id="d73b2-165"><xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> aggiunge <xref:Microsoft.AspNetCore.Authorization.Infrastructure.DenyAnonymousAuthorizationRequirement> all'istanza corrente che impone che l'utente corrente sia autenticato.</span><span class="sxs-lookup"><span data-stu-id="d73b2-165"><xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> adds <xref:Microsoft.AspNetCore.Authorization.Infrastructure.DenyAnonymousAuthorizationRequirement> to the current instance, which enforces that the current user is authenticated.</span></span>

<span data-ttu-id="d73b2-166">I criteri di autenticazione di fallback:</span><span class="sxs-lookup"><span data-stu-id="d73b2-166">The fallback authentication policy:</span></span>

* <span data-ttu-id="d73b2-167">Viene applicato a tutte le richieste che non specificano in modo esplicito i criteri di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-167">Is applied to all requests that do not explicitly specify an authentication policy.</span></span> <span data-ttu-id="d73b2-168">Per le richieste gestite dal routing degli endpoint, questo includerebbe qualsiasi endpoint che non specifichi un attributo di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-168">For requests served by endpoint routing, this would include any endpoint that does not specify an authorization attribute.</span></span> <span data-ttu-id="d73b2-169">Per le richieste gestite da altri middleware dopo il middleware di autorizzazione, ad esempio [i file statici](xref:fundamentals/static-files), il criterio viene applicato a tutte le richieste.</span><span class="sxs-lookup"><span data-stu-id="d73b2-169">For requests served by other middleware after the authorization middleware, such as [static files](xref:fundamentals/static-files), this would apply the policy to all requests.</span></span>

<span data-ttu-id="d73b2-170">L'impostazione dei criteri di autenticazione di fallback per richiedere l'autenticazione degli utenti consente di proteggere le Razor pagine e i controller appena aggiunti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-170">Setting the fallback authentication policy to require users to be authenticated protects newly added Razor Pages and controllers.</span></span> <span data-ttu-id="d73b2-171">La necessità di eseguire l'autenticazione per impostazione predefinita è più sicura rispetto a quella di fare affidamento su nuovi controller e Razor pagine per includere l' `[Authorize]` attributo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-171">Having authentication required by default is more secure than relying on new controllers and Razor Pages to include the `[Authorize]` attribute.</span></span>

<span data-ttu-id="d73b2-172">La <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions> classe contiene anche <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.DefaultPolicy?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="d73b2-172">The <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions> class also contains <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.DefaultPolicy?displayProperty=nameWithType>.</span></span> <span data-ttu-id="d73b2-173">`DefaultPolicy`È il criterio utilizzato con l' `[Authorize]` attributo quando non è specificato alcun criterio.</span><span class="sxs-lookup"><span data-stu-id="d73b2-173">The `DefaultPolicy` is the policy used with the `[Authorize]` attribute when no policy is specified.</span></span> <span data-ttu-id="d73b2-174">`[Authorize]` non contiene un criterio denominato, a differenza di `[Authorize(PolicyName="MyPolicy")]` .</span><span class="sxs-lookup"><span data-stu-id="d73b2-174">`[Authorize]` doesn't contain a named policy, unlike `[Authorize(PolicyName="MyPolicy")]`.</span></span>

<span data-ttu-id="d73b2-175">Per ulteriori informazioni sui criteri, vedere <xref:security/authorization/policies> .</span><span class="sxs-lookup"><span data-stu-id="d73b2-175">For more information on policies, see <xref:security/authorization/policies>.</span></span>

<span data-ttu-id="d73b2-176">Un metodo alternativo per i controller e Razor le pagine MVC per richiedere che tutti gli utenti siano autenticati è l'aggiunta di un filtro di autorizzazione:</span><span class="sxs-lookup"><span data-stu-id="d73b2-176">An alternative way for MVC controllers and Razor Pages to require all users be authenticated is adding an authorization filter:</span></span>

[!code-csharp[](secure-data/samples/final3/Startup2.cs?name=snippet&highlight=14-99)]

<span data-ttu-id="d73b2-177">Il codice precedente usa un filtro di autorizzazione, l'impostazione dei criteri di fallback usa il routing degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="d73b2-177">The preceding code uses an authorization filter, setting the fallback policy uses endpoint routing.</span></span> <span data-ttu-id="d73b2-178">L'impostazione dei criteri di fallback è il modo migliore per richiedere l'autenticazione di tutti gli utenti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-178">Setting the fallback policy is the preferred way to require all users be authenticated.</span></span>

<span data-ttu-id="d73b2-179">Aggiungere [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) alle `Index` pagine e in `Privacy` modo che gli utenti anonimi possano ottenere informazioni sul sito prima della registrazione:</span><span class="sxs-lookup"><span data-stu-id="d73b2-179">Add [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) to the `Index` and `Privacy` pages so anonymous users can get information about the site before they register:</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Index.cshtml.cs?highlight=1,7)]

### <a name="configure-the-test-account"></a><span data-ttu-id="d73b2-180">Configurare l'account di test</span><span class="sxs-lookup"><span data-stu-id="d73b2-180">Configure the test account</span></span>

<span data-ttu-id="d73b2-181">La `SeedData` classe crea due account: Administrator e Manager.</span><span class="sxs-lookup"><span data-stu-id="d73b2-181">The `SeedData` class creates two accounts: administrator and manager.</span></span> <span data-ttu-id="d73b2-182">Utilizzare lo [strumento Gestione segreta](xref:security/app-secrets) per impostare una password per questi account.</span><span class="sxs-lookup"><span data-stu-id="d73b2-182">Use the [Secret Manager tool](xref:security/app-secrets) to set a password for these accounts.</span></span> <span data-ttu-id="d73b2-183">Impostare la password dalla directory del progetto (la directory contenente *Program.cs*):</span><span class="sxs-lookup"><span data-stu-id="d73b2-183">Set the password from the project directory (the directory containing *Program.cs*):</span></span>

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

<span data-ttu-id="d73b2-184">Se non si specifica una password complessa, viene generata un'eccezione quando `SeedData.Initialize` viene chiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-184">If a strong password is not specified, an exception is thrown when `SeedData.Initialize` is called.</span></span>

<span data-ttu-id="d73b2-185">Aggiornare `Main` per usare la password di test:</span><span class="sxs-lookup"><span data-stu-id="d73b2-185">Update `Main` to use the test password:</span></span>

[!code-csharp[](secure-data/samples/final3/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a><span data-ttu-id="d73b2-186">Creare gli account di test e aggiornare i contatti</span><span class="sxs-lookup"><span data-stu-id="d73b2-186">Create the test accounts and update the contacts</span></span>

<span data-ttu-id="d73b2-187">Aggiornare il `Initialize` metodo nella `SeedData` classe per creare gli account di test:</span><span class="sxs-lookup"><span data-stu-id="d73b2-187">Update the `Initialize` method in the `SeedData` class to create the test accounts:</span></span>

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet_Initialize)]

<span data-ttu-id="d73b2-188">Aggiungere l'ID utente amministratore e `ContactStatus` ai contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-188">Add the administrator user ID and `ContactStatus` to the contacts.</span></span> <span data-ttu-id="d73b2-189">Creare uno dei contatti "inviato" e uno "rifiutato".</span><span class="sxs-lookup"><span data-stu-id="d73b2-189">Make one of the contacts "Submitted" and one "Rejected".</span></span> <span data-ttu-id="d73b2-190">Aggiungere l'ID utente e lo stato a tutti i contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-190">Add the user ID and status to all the contacts.</span></span> <span data-ttu-id="d73b2-191">Viene visualizzato un solo contatto:</span><span class="sxs-lookup"><span data-stu-id="d73b2-191">Only one contact is shown:</span></span>

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a><span data-ttu-id="d73b2-192">Creazione di gestori di autorizzazioni proprietario, Manager e amministratore</span><span class="sxs-lookup"><span data-stu-id="d73b2-192">Create owner, manager, and administrator authorization handlers</span></span>

<span data-ttu-id="d73b2-193">Creare una `ContactIsOwnerAuthorizationHandler` classe nella cartella *authorization* .</span><span class="sxs-lookup"><span data-stu-id="d73b2-193">Create a `ContactIsOwnerAuthorizationHandler` class in the *Authorization* folder.</span></span> <span data-ttu-id="d73b2-194">`ContactIsOwnerAuthorizationHandler`Verifica che l'utente che agisce su una risorsa appartenga alla risorsa.</span><span class="sxs-lookup"><span data-stu-id="d73b2-194">The `ContactIsOwnerAuthorizationHandler` verifies that the user acting on a resource owns the resource.</span></span>

[!code-csharp[](secure-data/samples/final3/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

<span data-ttu-id="d73b2-195">`ContactIsOwnerAuthorizationHandler`Contesto delle chiamate [. Ha esito positivo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_) se l'utente autenticato corrente è il proprietario del contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-195">The `ContactIsOwnerAuthorizationHandler` calls [context.Succeed](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_) if the current authenticated user is the contact owner.</span></span> <span data-ttu-id="d73b2-196">Gestori autorizzazioni in genere:</span><span class="sxs-lookup"><span data-stu-id="d73b2-196">Authorization handlers generally:</span></span>

* <span data-ttu-id="d73b2-197">Restituisce `context.Succeed` quando vengono soddisfatti i requisiti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-197">Return `context.Succeed` when the requirements are met.</span></span>
* <span data-ttu-id="d73b2-198">Restituisce `Task.CompletedTask` quando i requisiti non vengono soddisfatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-198">Return `Task.CompletedTask` when requirements aren't met.</span></span> <span data-ttu-id="d73b2-199">`Task.CompletedTask` non ha esito positivo o negativo &mdash; consente l'esecuzione di altri gestori di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-199">`Task.CompletedTask` is not success or failure&mdash;it allows other authorization handlers to run.</span></span>

<span data-ttu-id="d73b2-200">Se è necessario eseguire in modo esplicito l'errore, restituire [context. Esito negativo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail).</span><span class="sxs-lookup"><span data-stu-id="d73b2-200">If you need to explicitly fail, return [context.Fail](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail).</span></span>

<span data-ttu-id="d73b2-201">L'app consente ai proprietari dei contatti di modificare/eliminare/creare i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-201">The app allows contact owners to edit/delete/create their own data.</span></span> <span data-ttu-id="d73b2-202">`ContactIsOwnerAuthorizationHandler` non è necessario controllare l'operazione passata nel parametro requirement.</span><span class="sxs-lookup"><span data-stu-id="d73b2-202">`ContactIsOwnerAuthorizationHandler` doesn't need to check the operation passed in the requirement parameter.</span></span>

### <a name="create-a-manager-authorization-handler"></a><span data-ttu-id="d73b2-203">Creazione di un gestore autorizzazioni di gestione</span><span class="sxs-lookup"><span data-stu-id="d73b2-203">Create a manager authorization handler</span></span>

<span data-ttu-id="d73b2-204">Creare una `ContactManagerAuthorizationHandler` classe nella cartella *authorization* .</span><span class="sxs-lookup"><span data-stu-id="d73b2-204">Create a `ContactManagerAuthorizationHandler` class in the *Authorization* folder.</span></span> <span data-ttu-id="d73b2-205">`ContactManagerAuthorizationHandler`Verifica che l'utente che agisce sulla risorsa sia un responsabile.</span><span class="sxs-lookup"><span data-stu-id="d73b2-205">The `ContactManagerAuthorizationHandler` verifies the user acting on the resource is a manager.</span></span> <span data-ttu-id="d73b2-206">Solo i responsabili possono approvare o rifiutare le modifiche al contenuto (nuove o modificate).</span><span class="sxs-lookup"><span data-stu-id="d73b2-206">Only managers can approve or reject content changes (new or changed).</span></span>

[!code-csharp[](secure-data/samples/final3/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a><span data-ttu-id="d73b2-207">Creazione di un gestore autorizzazioni Amministratore</span><span class="sxs-lookup"><span data-stu-id="d73b2-207">Create an administrator authorization handler</span></span>

<span data-ttu-id="d73b2-208">Creare una `ContactAdministratorsAuthorizationHandler` classe nella cartella *authorization* .</span><span class="sxs-lookup"><span data-stu-id="d73b2-208">Create a `ContactAdministratorsAuthorizationHandler` class in the *Authorization* folder.</span></span> <span data-ttu-id="d73b2-209">Il `ContactAdministratorsAuthorizationHandler` Verifica che l'utente che agisce sulla risorsa sia un amministratore.</span><span class="sxs-lookup"><span data-stu-id="d73b2-209">The `ContactAdministratorsAuthorizationHandler` verifies the user acting on the resource is an administrator.</span></span> <span data-ttu-id="d73b2-210">L'amministratore può eseguire tutte le operazioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-210">Administrator can do all operations.</span></span>

[!code-csharp[](secure-data/samples/final3/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a><span data-ttu-id="d73b2-211">Registrare i gestori di autorizzazioni</span><span class="sxs-lookup"><span data-stu-id="d73b2-211">Register the authorization handlers</span></span>

<span data-ttu-id="d73b2-212">I servizi che usano Entity Framework Core devono essere registrati per l' [inserimento di dipendenze](xref:fundamentals/dependency-injection) usando [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions).</span><span class="sxs-lookup"><span data-stu-id="d73b2-212">Services using Entity Framework Core must be registered for [dependency injection](xref:fundamentals/dependency-injection) using [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions).</span></span> <span data-ttu-id="d73b2-213">`ContactIsOwnerAuthorizationHandler`Usa ASP.NET Core [Identity](xref:security/authentication/identity) , che è basato su Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="d73b2-213">The `ContactIsOwnerAuthorizationHandler` uses ASP.NET Core [Identity](xref:security/authentication/identity), which is built on Entity Framework Core.</span></span> <span data-ttu-id="d73b2-214">Registrare i gestori con la raccolta di servizi in modo che siano disponibili per `ContactsController` tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="d73b2-214">Register the handlers with the service collection so they're available to the `ContactsController` through [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="d73b2-215">Aggiungere il codice seguente alla fine di `ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d73b2-215">Add the following code to the end of `ConfigureServices`:</span></span>

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet_defaultPolicy&highlight=23-99)]

<span data-ttu-id="d73b2-216">`ContactAdministratorsAuthorizationHandler` e `ContactManagerAuthorizationHandler` vengono aggiunti come singleton.</span><span class="sxs-lookup"><span data-stu-id="d73b2-216">`ContactAdministratorsAuthorizationHandler` and `ContactManagerAuthorizationHandler` are added as singletons.</span></span> <span data-ttu-id="d73b2-217">Si tratta di singleton perché non usano EF e tutte le informazioni necessarie si trovano nel `Context` parametro del `HandleRequirementAsync` metodo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-217">They're singletons because they don't use EF and all the information needed is in the `Context` parameter of the `HandleRequirementAsync` method.</span></span>

## <a name="support-authorization"></a><span data-ttu-id="d73b2-218">Autorizzazione supporto</span><span class="sxs-lookup"><span data-stu-id="d73b2-218">Support authorization</span></span>

<span data-ttu-id="d73b2-219">In questa sezione si aggiornano le Razor pagine e si aggiunge una classe dei requisiti delle operazioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-219">In this section, you update the Razor Pages and add an operations requirements class.</span></span>

### <a name="review-the-contact-operations-requirements-class"></a><span data-ttu-id="d73b2-220">Esaminare la classe dei requisiti delle operazioni di contatto</span><span class="sxs-lookup"><span data-stu-id="d73b2-220">Review the contact operations requirements class</span></span>

<span data-ttu-id="d73b2-221">Esaminare la `ContactOperations` classe.</span><span class="sxs-lookup"><span data-stu-id="d73b2-221">Review the `ContactOperations` class.</span></span> <span data-ttu-id="d73b2-222">Questa classe contiene i requisiti supportati dall'app:</span><span class="sxs-lookup"><span data-stu-id="d73b2-222">This class contains the requirements the app supports:</span></span>

[!code-csharp[](secure-data/samples/final3/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-razor-pages"></a><span data-ttu-id="d73b2-223">Creare una classe di base per le Razor pagine contatti</span><span class="sxs-lookup"><span data-stu-id="d73b2-223">Create a base class for the Contacts Razor Pages</span></span>

<span data-ttu-id="d73b2-224">Creare una classe di base che contiene i servizi utilizzati nelle Razor pagine contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-224">Create a base class that contains the services used in the contacts Razor Pages.</span></span> <span data-ttu-id="d73b2-225">La classe base inserisce il codice di inizializzazione in un'unica posizione:</span><span class="sxs-lookup"><span data-stu-id="d73b2-225">The base class puts the initialization code in one location:</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/DI_BasePageModel.cs)]

<span data-ttu-id="d73b2-226">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="d73b2-226">The preceding code:</span></span>

* <span data-ttu-id="d73b2-227">Aggiunge il `IAuthorizationService` servizio per accedere ai gestori di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-227">Adds the `IAuthorizationService` service to access to the authorization handlers.</span></span>
* <span data-ttu-id="d73b2-228">Aggiunge il Identity `UserManager` servizio.</span><span class="sxs-lookup"><span data-stu-id="d73b2-228">Adds the Identity `UserManager` service.</span></span>
* <span data-ttu-id="d73b2-229">Aggiungere l'oggetto `ApplicationDbContext`.</span><span class="sxs-lookup"><span data-stu-id="d73b2-229">Add the `ApplicationDbContext`.</span></span>

### <a name="update-the-createmodel"></a><span data-ttu-id="d73b2-230">Aggiornare CreateModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-230">Update the CreateModel</span></span>

<span data-ttu-id="d73b2-231">Aggiornare il costruttore del modello di pagina create per usare la `DI_BasePageModel` classe di base:</span><span class="sxs-lookup"><span data-stu-id="d73b2-231">Update the create page model constructor to use the `DI_BasePageModel` base class:</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

<span data-ttu-id="d73b2-232">Aggiornare il `CreateModel.OnPostAsync` metodo a:</span><span class="sxs-lookup"><span data-stu-id="d73b2-232">Update the `CreateModel.OnPostAsync` method to:</span></span>

* <span data-ttu-id="d73b2-233">Aggiungere l'ID utente al `Contact` modello.</span><span class="sxs-lookup"><span data-stu-id="d73b2-233">Add the user ID to the `Contact` model.</span></span>
* <span data-ttu-id="d73b2-234">Chiamare il gestore autorizzazioni per verificare che l'utente disponga dell'autorizzazione per la creazione di contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-234">Call the authorization handler to verify the user has permission to create contacts.</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a><span data-ttu-id="d73b2-235">Aggiornare IndexModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-235">Update the IndexModel</span></span>

<span data-ttu-id="d73b2-236">Aggiornare il `OnGetAsync` metodo in modo da visualizzare solo i contatti approvati per gli utenti generali:</span><span class="sxs-lookup"><span data-stu-id="d73b2-236">Update the `OnGetAsync` method so only approved contacts are shown to general users:</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a><span data-ttu-id="d73b2-237">Aggiornare EditModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-237">Update the EditModel</span></span>

<span data-ttu-id="d73b2-238">Aggiungere un gestore autorizzazioni per verificare che l'utente sia proprietario del contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-238">Add an authorization handler to verify the user owns the contact.</span></span> <span data-ttu-id="d73b2-239">Poiché l'autorizzazione della risorsa viene convalidata, l' `[Authorize]` attributo non è sufficiente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-239">Because resource authorization is being validated, the `[Authorize]` attribute is not enough.</span></span> <span data-ttu-id="d73b2-240">L'app non ha accesso alla risorsa quando vengono valutati gli attributi.</span><span class="sxs-lookup"><span data-stu-id="d73b2-240">The app doesn't have access to the resource when attributes are evaluated.</span></span> <span data-ttu-id="d73b2-241">L'autorizzazione basata sulle risorse deve essere imperativa.</span><span class="sxs-lookup"><span data-stu-id="d73b2-241">Resource-based authorization must be imperative.</span></span> <span data-ttu-id="d73b2-242">I controlli devono essere eseguiti una volta che l'app può accedere alla risorsa, eseguendo il caricamento nel modello di pagina o caricando l'app all'interno del gestore stesso.</span><span class="sxs-lookup"><span data-stu-id="d73b2-242">Checks must be performed once the app has access to the resource, either by loading it in the page model or by loading it within the handler itself.</span></span> <span data-ttu-id="d73b2-243">Si accede spesso alla risorsa passando la chiave della risorsa.</span><span class="sxs-lookup"><span data-stu-id="d73b2-243">You frequently access the resource by passing in the resource key.</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a><span data-ttu-id="d73b2-244">Aggiornare DeleteModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-244">Update the DeleteModel</span></span>

<span data-ttu-id="d73b2-245">Aggiornare il modello di pagina Elimina per utilizzare il gestore autorizzazione per verificare che l'utente disponga dell'autorizzazione DELETE per il contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-245">Update the delete page model to use the authorization handler to verify the user has delete permission on the contact.</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a><span data-ttu-id="d73b2-246">Inserire il servizio di autorizzazione nelle viste</span><span class="sxs-lookup"><span data-stu-id="d73b2-246">Inject the authorization service into the views</span></span>

<span data-ttu-id="d73b2-247">Attualmente, l'interfaccia utente Mostra i collegamenti modifica ed Elimina per i contatti che non possono essere modificati dall'utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-247">Currently, the UI shows edit and delete links for contacts the user can't modify.</span></span>

<span data-ttu-id="d73b2-248">Inserire il servizio di autorizzazione nel file *pages/_ViewImports. cshtml* in modo che sia disponibile per tutte le visualizzazioni:</span><span class="sxs-lookup"><span data-stu-id="d73b2-248">Inject the authorization service in the *Pages/_ViewImports.cshtml* file so it's available to all views:</span></span>

[!code-cshtml[](secure-data/samples/final3/Pages/_ViewImports.cshtml?highlight=6-99)]

<span data-ttu-id="d73b2-249">Il markup precedente aggiunge diverse `using` istruzioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-249">The preceding markup adds several `using` statements.</span></span>

<span data-ttu-id="d73b2-250">Aggiornare i collegamenti **Edit** ed **Delete** in *pages/Contacts/index. cshtml* in modo che vengano sottoposti a rendering solo per gli utenti con le autorizzazioni appropriate:</span><span class="sxs-lookup"><span data-stu-id="d73b2-250">Update the **Edit** and **Delete** links in *Pages/Contacts/Index.cshtml* so they're only rendered for users with the appropriate permissions:</span></span>

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> <span data-ttu-id="d73b2-251">Nascondere i collegamenti da utenti che non dispongono dell'autorizzazione per modificare i dati non protegge l'app.</span><span class="sxs-lookup"><span data-stu-id="d73b2-251">Hiding links from users that don't have permission to change data doesn't secure the app.</span></span> <span data-ttu-id="d73b2-252">Nascondere i collegamenti rende l'app più intuitiva visualizzando solo i collegamenti validi.</span><span class="sxs-lookup"><span data-stu-id="d73b2-252">Hiding links makes the app more user-friendly by displaying only valid links.</span></span> <span data-ttu-id="d73b2-253">Gli utenti possono hackerare gli URL generati per richiamare le operazioni di modifica ed eliminazione sui dati di cui non sono proprietari.</span><span class="sxs-lookup"><span data-stu-id="d73b2-253">Users can hack the generated URLs to invoke edit and delete operations on data they don't own.</span></span> <span data-ttu-id="d73b2-254">La Razor pagina o il controller deve applicare i controlli di accesso per proteggere i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-254">The Razor Page or controller must enforce access checks to secure the data.</span></span>

### <a name="update-details"></a><span data-ttu-id="d73b2-255">Dettagli aggiornamento</span><span class="sxs-lookup"><span data-stu-id="d73b2-255">Update Details</span></span>

<span data-ttu-id="d73b2-256">Aggiornare la visualizzazione dettagli in modo che i responsabili possano approvare o rifiutare i contatti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-256">Update the details view so managers can approve or reject contacts:</span></span>

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Details.cshtml?name=snippet)]

<span data-ttu-id="d73b2-257">Aggiornare il modello della pagina dei dettagli:</span><span class="sxs-lookup"><span data-stu-id="d73b2-257">Update the details page model:</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a><span data-ttu-id="d73b2-258">Aggiungere o rimuovere un utente in un ruolo</span><span class="sxs-lookup"><span data-stu-id="d73b2-258">Add or remove a user to a role</span></span>

<span data-ttu-id="d73b2-259">Per informazioni su, vedere [questo problema](https://github.com/dotnet/AspNetCore.Docs/issues/8502) :</span><span class="sxs-lookup"><span data-stu-id="d73b2-259">See [this issue](https://github.com/dotnet/AspNetCore.Docs/issues/8502) for information on:</span></span>

* <span data-ttu-id="d73b2-260">Rimozione dei privilegi da un utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-260">Removing privileges from a user.</span></span> <span data-ttu-id="d73b2-261">Ad esempio, disattivare un utente in un'app di chat.</span><span class="sxs-lookup"><span data-stu-id="d73b2-261">For example, muting a user in a chat app.</span></span>
* <span data-ttu-id="d73b2-262">Aggiunta di privilegi a un utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-262">Adding privileges to a user.</span></span>

<a name="challenge"></a>

## <a name="differences-between-challenge-and-forbid"></a><span data-ttu-id="d73b2-263">Differenze tra la sfida e la proibizione</span><span class="sxs-lookup"><span data-stu-id="d73b2-263">Differences between Challenge and Forbid</span></span>

<span data-ttu-id="d73b2-264">Questa app imposta i criteri predefiniti per [richiedere l'autenticazione degli utenti](#rau).</span><span class="sxs-lookup"><span data-stu-id="d73b2-264">This app sets the default policy to [require authenticated users](#rau).</span></span> <span data-ttu-id="d73b2-265">Il codice seguente consente agli utenti anonimi.</span><span class="sxs-lookup"><span data-stu-id="d73b2-265">The following code allows anonymous users.</span></span> <span data-ttu-id="d73b2-266">Gli utenti anonimi possono mostrare le differenze tra la richiesta di confronto e la proibizione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-266">Anonymous users are allowed to show the differences between Challenge vs Forbid.</span></span>

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details2.cshtml.cs?name=snippet)]

<span data-ttu-id="d73b2-267">Nel codice precedente:</span><span class="sxs-lookup"><span data-stu-id="d73b2-267">In the preceding code:</span></span>

* <span data-ttu-id="d73b2-268">Quando l'utente **non** è autenticato, `ChallengeResult` viene restituito un oggetto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-268">When the user is **not** authenticated, a `ChallengeResult` is returned.</span></span> <span data-ttu-id="d73b2-269">Quando `ChallengeResult` viene restituito un oggetto, l'utente viene reindirizzato alla pagina di accesso.</span><span class="sxs-lookup"><span data-stu-id="d73b2-269">When a `ChallengeResult` is returned, the user is redirected to the sign-in page.</span></span>
* <span data-ttu-id="d73b2-270">Quando l'utente viene autenticato, ma non autorizzato, `ForbidResult` viene restituito un oggetto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-270">When the user is authenticated, but not authorized, a `ForbidResult` is returned.</span></span> <span data-ttu-id="d73b2-271">Quando `ForbidResult` viene restituito un oggetto, l'utente viene reindirizzato alla pagina di accesso negato.</span><span class="sxs-lookup"><span data-stu-id="d73b2-271">When a `ForbidResult` is returned, the user is redirected to the access denied page.</span></span>

## <a name="test-the-completed-app"></a><span data-ttu-id="d73b2-272">Testare l'app completata</span><span class="sxs-lookup"><span data-stu-id="d73b2-272">Test the completed app</span></span>

<span data-ttu-id="d73b2-273">Se non è già stata impostata una password per gli account utente di cui è stato eseguito il seeding, usare lo [strumento Gestione segreta](xref:security/app-secrets#secret-manager) per impostare una password:</span><span class="sxs-lookup"><span data-stu-id="d73b2-273">If you haven't already set a password for seeded user accounts, use the [Secret Manager tool](xref:security/app-secrets#secret-manager) to set a password:</span></span>

* <span data-ttu-id="d73b2-274">Scegliere una password complessa: usare otto o più caratteri e almeno un carattere maiuscolo, un numero e un simbolo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-274">Choose a strong password: Use eight or more characters and at least one upper-case character, number, and symbol.</span></span> <span data-ttu-id="d73b2-275">Ad esempio, `Passw0rd!` soddisfa i requisiti per le password complesse.</span><span class="sxs-lookup"><span data-stu-id="d73b2-275">For example, `Passw0rd!` meets the strong password requirements.</span></span>
* <span data-ttu-id="d73b2-276">Eseguire il comando seguente dalla cartella del progetto, dove `<PW>` è la password:</span><span class="sxs-lookup"><span data-stu-id="d73b2-276">Execute the following command from the project's folder, where `<PW>` is the password:</span></span>

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

<span data-ttu-id="d73b2-277">Se l'app dispone di contatti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-277">If the app has contacts:</span></span>

* <span data-ttu-id="d73b2-278">Eliminare tutti i record nella `Contact` tabella.</span><span class="sxs-lookup"><span data-stu-id="d73b2-278">Delete all of the records in the `Contact` table.</span></span>
* <span data-ttu-id="d73b2-279">Riavviare l'app per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="d73b2-279">Restart the app to seed the database.</span></span>

<span data-ttu-id="d73b2-280">Un modo semplice per testare l'app completata consiste nell'avviare tre diversi browser (o sessioni in incognito/InPrivate).</span><span class="sxs-lookup"><span data-stu-id="d73b2-280">An easy way to test the completed app is to launch three different browsers (or incognito/InPrivate sessions).</span></span> <span data-ttu-id="d73b2-281">In un browser registrare un nuovo utente (ad esempio, `test@example.com` ).</span><span class="sxs-lookup"><span data-stu-id="d73b2-281">In one browser, register a new user (for example, `test@example.com`).</span></span> <span data-ttu-id="d73b2-282">Accedere a ogni browser con un altro utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-282">Sign in to each browser with a different user.</span></span> <span data-ttu-id="d73b2-283">Verificare le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-283">Verify the following operations:</span></span>

* <span data-ttu-id="d73b2-284">Gli utenti registrati possono visualizzare tutti i dati di contatto approvati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-284">Registered users can view all of the approved contact data.</span></span>
* <span data-ttu-id="d73b2-285">Gli utenti registrati possono modificare/eliminare i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-285">Registered users can edit/delete their own data.</span></span>
* <span data-ttu-id="d73b2-286">I responsabili possono approvare/rifiutare i dati di contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-286">Managers can approve/reject contact data.</span></span> <span data-ttu-id="d73b2-287">La `Details` visualizzazione Mostra i pulsanti **approva** e **rifiuta** .</span><span class="sxs-lookup"><span data-stu-id="d73b2-287">The `Details` view shows **Approve** and **Reject** buttons.</span></span>
* <span data-ttu-id="d73b2-288">Gli amministratori possono approvare/rifiutare e modificare/eliminare tutti i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-288">Administrators can approve/reject and edit/delete all data.</span></span>

| <span data-ttu-id="d73b2-289">User</span><span class="sxs-lookup"><span data-stu-id="d73b2-289">User</span></span>                | <span data-ttu-id="d73b2-290">Seeding dall'app</span><span class="sxs-lookup"><span data-stu-id="d73b2-290">Seeded by the app</span></span> | <span data-ttu-id="d73b2-291">Opzioni</span><span class="sxs-lookup"><span data-stu-id="d73b2-291">Options</span></span>                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | <span data-ttu-id="d73b2-292">No</span><span class="sxs-lookup"><span data-stu-id="d73b2-292">No</span></span>                | <span data-ttu-id="d73b2-293">Modificare/eliminare i dati personali.</span><span class="sxs-lookup"><span data-stu-id="d73b2-293">Edit/delete the own data.</span></span>                |
| manager@contoso.com | <span data-ttu-id="d73b2-294">Sì</span><span class="sxs-lookup"><span data-stu-id="d73b2-294">Yes</span></span>               | <span data-ttu-id="d73b2-295">Approva/rifiuta e modifica/elimina i dati personali.</span><span class="sxs-lookup"><span data-stu-id="d73b2-295">Approve/reject and edit/delete own data.</span></span> |
| admin@contoso.com   | <span data-ttu-id="d73b2-296">Sì</span><span class="sxs-lookup"><span data-stu-id="d73b2-296">Yes</span></span>               | <span data-ttu-id="d73b2-297">Approva/rifiuta e modifica/elimina tutti i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-297">Approve/reject and edit/delete all data.</span></span> |

<span data-ttu-id="d73b2-298">Creare un contatto nel browser dell'amministratore.</span><span class="sxs-lookup"><span data-stu-id="d73b2-298">Create a contact in the administrator's browser.</span></span> <span data-ttu-id="d73b2-299">Copiare l'URL da eliminare e modificare dal contatto amministratore.</span><span class="sxs-lookup"><span data-stu-id="d73b2-299">Copy the URL for delete and edit from the administrator contact.</span></span> <span data-ttu-id="d73b2-300">Incollare questi collegamenti nel browser dell'utente test per verificare che l'utente test non possa eseguire queste operazioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-300">Paste these links into the test user's browser to verify the test user can't perform these operations.</span></span>

## <a name="create-the-starter-app"></a><span data-ttu-id="d73b2-301">Creare l'app Starter</span><span class="sxs-lookup"><span data-stu-id="d73b2-301">Create the starter app</span></span>

* <span data-ttu-id="d73b2-302">Creare un' Razor app per le pagine denominata "ContactManager"</span><span class="sxs-lookup"><span data-stu-id="d73b2-302">Create a Razor Pages app named "ContactManager"</span></span>
  * <span data-ttu-id="d73b2-303">Creare l'app con **singoli account utente**.</span><span class="sxs-lookup"><span data-stu-id="d73b2-303">Create the app with **Individual User Accounts**.</span></span>
  * <span data-ttu-id="d73b2-304">Denominarlo "ContactManager" in modo che lo spazio dei nomi corrisponda allo spazio dei nomi usato nell'esempio.</span><span class="sxs-lookup"><span data-stu-id="d73b2-304">Name it "ContactManager" so the namespace matches the namespace used in the sample.</span></span>
  * <span data-ttu-id="d73b2-305">`-uld` Specifica il database locale anziché SQLite</span><span class="sxs-lookup"><span data-stu-id="d73b2-305">`-uld` specifies LocalDB instead of SQLite</span></span>

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* <span data-ttu-id="d73b2-306">Aggiungi *modelli/Contact. cs*:</span><span class="sxs-lookup"><span data-stu-id="d73b2-306">Add *Models/Contact.cs*:</span></span>

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* <span data-ttu-id="d73b2-307">Impalcatura del `Contact` modello.</span><span class="sxs-lookup"><span data-stu-id="d73b2-307">Scaffold the `Contact` model.</span></span>
* <span data-ttu-id="d73b2-308">Creare la migrazione iniziale e aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="d73b2-308">Create initial migration and update the database:</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
dotnet ef database drop -f
dotnet ef migrations add initial
dotnet ef database update
```

<span data-ttu-id="d73b2-309">Se si verifica un bug con il `dotnet aspnet-codegenerator razorpage` comando, vedere [questo problema di GitHub](https://github.com/aspnet/Scaffolding/issues/984).</span><span class="sxs-lookup"><span data-stu-id="d73b2-309">If you experience a bug with the `dotnet aspnet-codegenerator razorpage` command, see [this GitHub issue](https://github.com/aspnet/Scaffolding/issues/984).</span></span>

* <span data-ttu-id="d73b2-310">Aggiornare l'ancoraggio **ContactManager** nel file *pages/Shared/_Layout. cshtml* :</span><span class="sxs-lookup"><span data-stu-id="d73b2-310">Update the **ContactManager** anchor in the *Pages/Shared/_Layout.cshtml* file:</span></span>

 ```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Contacts/Index">ContactManager</a>
  ```

* <span data-ttu-id="d73b2-311">Testare l'app creando, modificando ed eliminando un contatto</span><span class="sxs-lookup"><span data-stu-id="d73b2-311">Test the app by creating, editing, and deleting a contact</span></span>

### <a name="seed-the-database"></a><span data-ttu-id="d73b2-312">Specificare il valore di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="d73b2-312">Seed the database</span></span>

<span data-ttu-id="d73b2-313">Aggiungere la classe [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/starter3/Data/SeedData.cs) alla cartella *Data* :</span><span class="sxs-lookup"><span data-stu-id="d73b2-313">Add the [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/starter3/Data/SeedData.cs) class to the *Data* folder:</span></span>

[!code-csharp[](secure-data/samples/starter3/Data/SeedData.cs)]

<span data-ttu-id="d73b2-314">Chiama `SeedData.Initialize` da `Main` :</span><span class="sxs-lookup"><span data-stu-id="d73b2-314">Call `SeedData.Initialize` from `Main`:</span></span>

[!code-csharp[](secure-data/samples/starter3/Program.cs)]

<span data-ttu-id="d73b2-315">Verificare che l'app abbia sottoposta a seeding il database.</span><span class="sxs-lookup"><span data-stu-id="d73b2-315">Test that the app seeded the database.</span></span> <span data-ttu-id="d73b2-316">Se sono presenti righe nel database Contact, il metodo seed non viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="d73b2-316">If there are any rows in the contact DB, the seed method doesn't run.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

<span data-ttu-id="d73b2-317">Questa esercitazione illustra come creare un'app Web di ASP.NET Core con i dati utente protetti dall'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-317">This tutorial shows how to create an ASP.NET Core web app with user data protected by authorization.</span></span> <span data-ttu-id="d73b2-318">Viene visualizzato un elenco di contatti creati dagli utenti autenticati (registrati).</span><span class="sxs-lookup"><span data-stu-id="d73b2-318">It displays a list of contacts that authenticated (registered) users have created.</span></span> <span data-ttu-id="d73b2-319">Sono disponibili tre gruppi di sicurezza:</span><span class="sxs-lookup"><span data-stu-id="d73b2-319">There are three security groups:</span></span>

* <span data-ttu-id="d73b2-320">**Gli utenti registrati** possono visualizzare tutti i dati approvati e possono modificare/eliminare i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-320">**Registered users** can view all the approved data and can edit/delete their own data.</span></span>
* <span data-ttu-id="d73b2-321">I **responsabili** possono approvare o rifiutare i dati di contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-321">**Managers** can approve or reject contact data.</span></span> <span data-ttu-id="d73b2-322">Solo i contatti approvati sono visibili agli utenti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-322">Only approved contacts are visible to users.</span></span>
* <span data-ttu-id="d73b2-323">Gli **amministratori** possono approvare/rifiutare e modificare/eliminare i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-323">**Administrators** can approve/reject and edit/delete any data.</span></span>

<span data-ttu-id="d73b2-324">Nell'immagine seguente, l'utente Rick ( `rick@example.com` ) ha eseguito l'accesso.</span><span class="sxs-lookup"><span data-stu-id="d73b2-324">In the following image, user Rick (`rick@example.com`) is signed in.</span></span> <span data-ttu-id="d73b2-325">Rick può visualizzare solo i contatti approvati e **modificare** / **Delete** / **Crea nuovi** collegamenti per i contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-325">Rick can only view approved contacts and **Edit**/**Delete**/**Create New** links for his contacts.</span></span> <span data-ttu-id="d73b2-326">Solo l'ultimo record, creato da Rick, Visualizza i collegamenti **modifica** ed **Elimina** .</span><span class="sxs-lookup"><span data-stu-id="d73b2-326">Only the last record, created by Rick, displays **Edit** and **Delete** links.</span></span> <span data-ttu-id="d73b2-327">Gli altri utenti non vedranno l'ultimo record fino a quando un responsabile o un amministratore non modifica lo stato in "approvato".</span><span class="sxs-lookup"><span data-stu-id="d73b2-327">Other users won't see the last record until a manager or administrator changes the status to "Approved".</span></span>

![Screenshot che illustra l'accesso a Rick](secure-data/_static/rick.png)

<span data-ttu-id="d73b2-329">Nell'immagine seguente, `manager@contoso.com` è stato eseguito l'accesso e nel ruolo del Manager:</span><span class="sxs-lookup"><span data-stu-id="d73b2-329">In the following image, `manager@contoso.com` is signed in and in the manager's role:</span></span>

![Screenshot che mostra l' manager@contoso.com accesso](secure-data/_static/manager1.png)

<span data-ttu-id="d73b2-331">Nella figura seguente viene illustrata la visualizzazione dettagli Manager di un contatto:</span><span class="sxs-lookup"><span data-stu-id="d73b2-331">The following image shows the managers details view of a contact:</span></span>

![Visualizzazione del contatto da un responsabile](secure-data/_static/manager.png)

<span data-ttu-id="d73b2-333">I pulsanti **approva** e **rifiuta** vengono visualizzati solo per Manager e amministratori.</span><span class="sxs-lookup"><span data-stu-id="d73b2-333">The **Approve** and **Reject** buttons are only displayed for managers and administrators.</span></span>

<span data-ttu-id="d73b2-334">Nell'immagine seguente, `admin@contoso.com` è stato eseguito l'accesso e il ruolo dell'amministratore:</span><span class="sxs-lookup"><span data-stu-id="d73b2-334">In the following image, `admin@contoso.com` is signed in and in the administrator's role:</span></span>

![Screenshot che mostra l' admin@contoso.com accesso](secure-data/_static/admin.png)

<span data-ttu-id="d73b2-336">L'amministratore dispone di tutti i privilegi.</span><span class="sxs-lookup"><span data-stu-id="d73b2-336">The administrator has all privileges.</span></span> <span data-ttu-id="d73b2-337">Può leggere, modificare ed eliminare qualsiasi contatto e modificare lo stato dei contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-337">She can read/edit/delete any contact and change the status of contacts.</span></span>

<span data-ttu-id="d73b2-338">L'app è stata creata mediante l' [impalcatura](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) del `Contact` modello seguente:</span><span class="sxs-lookup"><span data-stu-id="d73b2-338">The app was created by [scaffolding](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) the following `Contact` model:</span></span>

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

<span data-ttu-id="d73b2-339">L'esempio contiene i gestori di autorizzazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-339">The sample contains the following authorization handlers:</span></span>

* <span data-ttu-id="d73b2-340">`ContactIsOwnerAuthorizationHandler`: Assicura che un utente possa modificare solo i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-340">`ContactIsOwnerAuthorizationHandler`: Ensures that a user can only edit their data.</span></span>
* <span data-ttu-id="d73b2-341">`ContactManagerAuthorizationHandler`: Consente ai responsabili di approvare o rifiutare i contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-341">`ContactManagerAuthorizationHandler`: Allows managers to approve or reject contacts.</span></span>
* <span data-ttu-id="d73b2-342">`ContactAdministratorsAuthorizationHandler`: Consente agli amministratori di approvare o rifiutare contatti e di modificare/eliminare contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-342">`ContactAdministratorsAuthorizationHandler`: Allows administrators to approve or reject contacts and to edit/delete contacts.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d73b2-343">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="d73b2-343">Prerequisites</span></span>

<span data-ttu-id="d73b2-344">Questa esercitazione è avanzata.</span><span class="sxs-lookup"><span data-stu-id="d73b2-344">This tutorial is advanced.</span></span> <span data-ttu-id="d73b2-345">È necessario avere familiarità con:</span><span class="sxs-lookup"><span data-stu-id="d73b2-345">You should be familiar with:</span></span>

* [<span data-ttu-id="d73b2-346">ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="d73b2-346">ASP.NET Core</span></span>](xref:tutorials/first-mvc-app/start-mvc)
* [<span data-ttu-id="d73b2-347">autenticazione</span><span class="sxs-lookup"><span data-stu-id="d73b2-347">Authentication</span></span>](xref:security/authentication/identity)
* [<span data-ttu-id="d73b2-348">Conferma account e recupero password</span><span class="sxs-lookup"><span data-stu-id="d73b2-348">Account Confirmation and Password Recovery</span></span>](xref:security/authentication/accconfirm)
* [<span data-ttu-id="d73b2-349">Autorizzazione</span><span class="sxs-lookup"><span data-stu-id="d73b2-349">Authorization</span></span>](xref:security/authorization/introduction)
* [<span data-ttu-id="d73b2-350">Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="d73b2-350">Entity Framework Core</span></span>](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a><span data-ttu-id="d73b2-351">App iniziale e completata</span><span class="sxs-lookup"><span data-stu-id="d73b2-351">The starter and completed app</span></span>

<span data-ttu-id="d73b2-352">[Scaricare](xref:index#how-to-download-a-sample) l'app [completata](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples) .</span><span class="sxs-lookup"><span data-stu-id="d73b2-352">[Download](xref:index#how-to-download-a-sample) the [completed](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples) app.</span></span> <span data-ttu-id="d73b2-353">[Testare](#test-the-completed-app) l'app completata in modo da acquisire familiarità con le funzionalità di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="d73b2-353">[Test](#test-the-completed-app) the completed app so you become familiar with its security features.</span></span>

### <a name="the-starter-app"></a><span data-ttu-id="d73b2-354">App iniziale</span><span class="sxs-lookup"><span data-stu-id="d73b2-354">The starter app</span></span>

<span data-ttu-id="d73b2-355">[Scaricare](xref:index#how-to-download-a-sample) l'app [Starter](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/) .</span><span class="sxs-lookup"><span data-stu-id="d73b2-355">[Download](xref:index#how-to-download-a-sample) the [starter](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/) app.</span></span>

<span data-ttu-id="d73b2-356">Eseguire l'app, toccare il collegamento **ContactManager** e verificare che sia possibile creare, modificare ed eliminare un contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-356">Run the app, tap the **ContactManager** link, and verify you can create, edit, and delete a contact.</span></span>

## <a name="secure-user-data"></a><span data-ttu-id="d73b2-357">Proteggere i dati utente</span><span class="sxs-lookup"><span data-stu-id="d73b2-357">Secure user data</span></span>

<span data-ttu-id="d73b2-358">Le sezioni seguenti includono tutti i passaggi principali per creare l'app Secure User Data.</span><span class="sxs-lookup"><span data-stu-id="d73b2-358">The following sections have all the major steps to create the secure user data app.</span></span> <span data-ttu-id="d73b2-359">Può risultare utile fare riferimento al progetto completato.</span><span class="sxs-lookup"><span data-stu-id="d73b2-359">You may find it helpful to refer to the completed project.</span></span>

### <a name="tie-the-contact-data-to-the-user"></a><span data-ttu-id="d73b2-360">Associare i dati di contatto all'utente</span><span class="sxs-lookup"><span data-stu-id="d73b2-360">Tie the contact data to the user</span></span>

<span data-ttu-id="d73b2-361">Usare l' [Identity](xref:security/authentication/identity) ID utente ASP.NET per assicurarsi che gli utenti possano modificare i dati, ma non i dati di altri utenti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-361">Use the ASP.NET [Identity](xref:security/authentication/identity) user ID to ensure users can edit their data, but not other users data.</span></span> <span data-ttu-id="d73b2-362">Aggiungere `OwnerID` e `ContactStatus` al `Contact` modello:</span><span class="sxs-lookup"><span data-stu-id="d73b2-362">Add `OwnerID` and `ContactStatus` to the `Contact` model:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

<span data-ttu-id="d73b2-363">`OwnerID` ID dell'utente della `AspNetUser` tabella nel [Identity](xref:security/authentication/identity) database.</span><span class="sxs-lookup"><span data-stu-id="d73b2-363">`OwnerID` is the user's ID from the `AspNetUser` table in the [Identity](xref:security/authentication/identity) database.</span></span> <span data-ttu-id="d73b2-364">Il `Status` campo determina se un contatto è visualizzabile dagli utenti generali.</span><span class="sxs-lookup"><span data-stu-id="d73b2-364">The `Status` field determines if a contact is viewable by general users.</span></span>

<span data-ttu-id="d73b2-365">Creare una nuova migrazione e aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="d73b2-365">Create a new migration and update the database:</span></span>

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-identity"></a><span data-ttu-id="d73b2-366">Aggiungere servizi ruolo a Identity</span><span class="sxs-lookup"><span data-stu-id="d73b2-366">Add Role services to Identity</span></span>

<span data-ttu-id="d73b2-367">Aggiungere [Aggiungi ruoli](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) per aggiungere servizi ruolo:</span><span class="sxs-lookup"><span data-stu-id="d73b2-367">Append [AddRoles](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) to add Role services:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet2&highlight=11)]

### <a name="require-authenticated-users"></a><span data-ttu-id="d73b2-368">Richiedi utenti autenticati</span><span class="sxs-lookup"><span data-stu-id="d73b2-368">Require authenticated users</span></span>

<span data-ttu-id="d73b2-369">Impostare i criteri di autenticazione predefiniti per richiedere l'autenticazione degli utenti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-369">Set the default authentication policy to require users to be authenticated:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet&highlight=17-99)] 

 <span data-ttu-id="d73b2-370">È possibile rifiutare esplicitamente l'autenticazione a Razor livello di pagina, controller o metodo di azione con l' `[AllowAnonymous]` attributo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-370">You can opt out of authentication at the Razor Page, controller, or action method level with the `[AllowAnonymous]` attribute.</span></span> <span data-ttu-id="d73b2-371">L'impostazione dei criteri di autenticazione predefiniti per richiedere l'autenticazione degli utenti consente di proteggere le Razor pagine e i controller appena aggiunti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-371">Setting the default authentication policy to require users to be authenticated protects newly added Razor Pages and controllers.</span></span> <span data-ttu-id="d73b2-372">La necessità di eseguire l'autenticazione per impostazione predefinita è più sicura rispetto a quella di fare affidamento su nuovi controller e Razor pagine per includere l' `[Authorize]` attributo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-372">Having authentication required by default is more secure than relying on new controllers and Razor Pages to include the `[Authorize]` attribute.</span></span>

<span data-ttu-id="d73b2-373">Aggiungere [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) alle pagine di indice, informazioni e contatti in modo che gli utenti anonimi possano ottenere informazioni sul sito prima della registrazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-373">Add [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) to the Index, About, and Contact pages so anonymous users can get information about the site before they register.</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Index.cshtml.cs?highlight=1,6)]

### <a name="configure-the-test-account"></a><span data-ttu-id="d73b2-374">Configurare l'account di test</span><span class="sxs-lookup"><span data-stu-id="d73b2-374">Configure the test account</span></span>

<span data-ttu-id="d73b2-375">La `SeedData` classe crea due account: Administrator e Manager.</span><span class="sxs-lookup"><span data-stu-id="d73b2-375">The `SeedData` class creates two accounts: administrator and manager.</span></span> <span data-ttu-id="d73b2-376">Utilizzare lo [strumento Gestione segreta](xref:security/app-secrets) per impostare una password per questi account.</span><span class="sxs-lookup"><span data-stu-id="d73b2-376">Use the [Secret Manager tool](xref:security/app-secrets) to set a password for these accounts.</span></span> <span data-ttu-id="d73b2-377">Impostare la password dalla directory del progetto (la directory contenente *Program.cs*):</span><span class="sxs-lookup"><span data-stu-id="d73b2-377">Set the password from the project directory (the directory containing *Program.cs*):</span></span>

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

<span data-ttu-id="d73b2-378">Se non si specifica una password complessa, viene generata un'eccezione quando `SeedData.Initialize` viene chiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-378">If a strong password is not specified, an exception is thrown when `SeedData.Initialize` is called.</span></span>

<span data-ttu-id="d73b2-379">Aggiornare `Main` per usare la password di test:</span><span class="sxs-lookup"><span data-stu-id="d73b2-379">Update `Main` to use the test password:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a><span data-ttu-id="d73b2-380">Creare gli account di test e aggiornare i contatti</span><span class="sxs-lookup"><span data-stu-id="d73b2-380">Create the test accounts and update the contacts</span></span>

<span data-ttu-id="d73b2-381">Aggiornare il `Initialize` metodo nella `SeedData` classe per creare gli account di test:</span><span class="sxs-lookup"><span data-stu-id="d73b2-381">Update the `Initialize` method in the `SeedData` class to create the test accounts:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet_Initialize)]

<span data-ttu-id="d73b2-382">Aggiungere l'ID utente amministratore e `ContactStatus` ai contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-382">Add the administrator user ID and `ContactStatus` to the contacts.</span></span> <span data-ttu-id="d73b2-383">Creare uno dei contatti "inviato" e uno "rifiutato".</span><span class="sxs-lookup"><span data-stu-id="d73b2-383">Make one of the contacts "Submitted" and one "Rejected".</span></span> <span data-ttu-id="d73b2-384">Aggiungere l'ID utente e lo stato a tutti i contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-384">Add the user ID and status to all the contacts.</span></span> <span data-ttu-id="d73b2-385">Viene visualizzato un solo contatto:</span><span class="sxs-lookup"><span data-stu-id="d73b2-385">Only one contact is shown:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a><span data-ttu-id="d73b2-386">Creazione di gestori di autorizzazioni proprietario, Manager e amministratore</span><span class="sxs-lookup"><span data-stu-id="d73b2-386">Create owner, manager, and administrator authorization handlers</span></span>

<span data-ttu-id="d73b2-387">Creare una cartella di *autorizzazione* e crearvi una `ContactIsOwnerAuthorizationHandler` classe.</span><span class="sxs-lookup"><span data-stu-id="d73b2-387">Create an *Authorization* folder and create a `ContactIsOwnerAuthorizationHandler` class in it.</span></span> <span data-ttu-id="d73b2-388">`ContactIsOwnerAuthorizationHandler`Verifica che l'utente che agisce su una risorsa appartenga alla risorsa.</span><span class="sxs-lookup"><span data-stu-id="d73b2-388">The `ContactIsOwnerAuthorizationHandler` verifies that the user acting on a resource owns the resource.</span></span>

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

<span data-ttu-id="d73b2-389">`ContactIsOwnerAuthorizationHandler`Contesto delle chiamate [. Ha esito positivo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_) se l'utente autenticato corrente è il proprietario del contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-389">The `ContactIsOwnerAuthorizationHandler` calls [context.Succeed](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_) if the current authenticated user is the contact owner.</span></span> <span data-ttu-id="d73b2-390">Gestori autorizzazioni in genere:</span><span class="sxs-lookup"><span data-stu-id="d73b2-390">Authorization handlers generally:</span></span>

* <span data-ttu-id="d73b2-391">Restituisce `context.Succeed` quando vengono soddisfatti i requisiti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-391">Return `context.Succeed` when the requirements are met.</span></span>
* <span data-ttu-id="d73b2-392">Restituisce `Task.CompletedTask` quando i requisiti non vengono soddisfatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-392">Return `Task.CompletedTask` when requirements aren't met.</span></span> <span data-ttu-id="d73b2-393">`Task.CompletedTask` non ha esito positivo o negativo &mdash; consente l'esecuzione di altri gestori di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-393">`Task.CompletedTask` is not success or failure&mdash;it allows other authorization handlers to run.</span></span>

<span data-ttu-id="d73b2-394">Se è necessario eseguire in modo esplicito l'errore, restituire [context. Esito negativo](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail).</span><span class="sxs-lookup"><span data-stu-id="d73b2-394">If you need to explicitly fail, return [context.Fail](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail).</span></span>

<span data-ttu-id="d73b2-395">L'app consente ai proprietari dei contatti di modificare/eliminare/creare i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-395">The app allows contact owners to edit/delete/create their own data.</span></span> <span data-ttu-id="d73b2-396">`ContactIsOwnerAuthorizationHandler` non è necessario controllare l'operazione passata nel parametro requirement.</span><span class="sxs-lookup"><span data-stu-id="d73b2-396">`ContactIsOwnerAuthorizationHandler` doesn't need to check the operation passed in the requirement parameter.</span></span>

### <a name="create-a-manager-authorization-handler"></a><span data-ttu-id="d73b2-397">Creazione di un gestore autorizzazioni di gestione</span><span class="sxs-lookup"><span data-stu-id="d73b2-397">Create a manager authorization handler</span></span>

<span data-ttu-id="d73b2-398">Creare una `ContactManagerAuthorizationHandler` classe nella cartella *authorization* .</span><span class="sxs-lookup"><span data-stu-id="d73b2-398">Create a `ContactManagerAuthorizationHandler` class in the *Authorization* folder.</span></span> <span data-ttu-id="d73b2-399">`ContactManagerAuthorizationHandler`Verifica che l'utente che agisce sulla risorsa sia un responsabile.</span><span class="sxs-lookup"><span data-stu-id="d73b2-399">The `ContactManagerAuthorizationHandler` verifies the user acting on the resource is a manager.</span></span> <span data-ttu-id="d73b2-400">Solo i responsabili possono approvare o rifiutare le modifiche al contenuto (nuove o modificate).</span><span class="sxs-lookup"><span data-stu-id="d73b2-400">Only managers can approve or reject content changes (new or changed).</span></span>

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a><span data-ttu-id="d73b2-401">Creazione di un gestore autorizzazioni Amministratore</span><span class="sxs-lookup"><span data-stu-id="d73b2-401">Create an administrator authorization handler</span></span>

<span data-ttu-id="d73b2-402">Creare una `ContactAdministratorsAuthorizationHandler` classe nella cartella *authorization* .</span><span class="sxs-lookup"><span data-stu-id="d73b2-402">Create a `ContactAdministratorsAuthorizationHandler` class in the *Authorization* folder.</span></span> <span data-ttu-id="d73b2-403">Il `ContactAdministratorsAuthorizationHandler` Verifica che l'utente che agisce sulla risorsa sia un amministratore.</span><span class="sxs-lookup"><span data-stu-id="d73b2-403">The `ContactAdministratorsAuthorizationHandler` verifies the user acting on the resource is an administrator.</span></span> <span data-ttu-id="d73b2-404">L'amministratore può eseguire tutte le operazioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-404">Administrator can do all operations.</span></span>

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a><span data-ttu-id="d73b2-405">Registrare i gestori di autorizzazioni</span><span class="sxs-lookup"><span data-stu-id="d73b2-405">Register the authorization handlers</span></span>

<span data-ttu-id="d73b2-406">I servizi che usano Entity Framework Core devono essere registrati per l' [inserimento di dipendenze](xref:fundamentals/dependency-injection) usando [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions).</span><span class="sxs-lookup"><span data-stu-id="d73b2-406">Services using Entity Framework Core must be registered for [dependency injection](xref:fundamentals/dependency-injection) using [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions).</span></span> <span data-ttu-id="d73b2-407">`ContactIsOwnerAuthorizationHandler`Usa ASP.NET Core [Identity](xref:security/authentication/identity) , che è basato su Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="d73b2-407">The `ContactIsOwnerAuthorizationHandler` uses ASP.NET Core [Identity](xref:security/authentication/identity), which is built on Entity Framework Core.</span></span> <span data-ttu-id="d73b2-408">Registrare i gestori con la raccolta di servizi in modo che siano disponibili per `ContactsController` tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="d73b2-408">Register the handlers with the service collection so they're available to the `ContactsController` through [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="d73b2-409">Aggiungere il codice seguente alla fine di `ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d73b2-409">Add the following code to the end of `ConfigureServices`:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet_defaultPolicy&highlight=27-99)]

<span data-ttu-id="d73b2-410">`ContactAdministratorsAuthorizationHandler` e `ContactManagerAuthorizationHandler` vengono aggiunti come singleton.</span><span class="sxs-lookup"><span data-stu-id="d73b2-410">`ContactAdministratorsAuthorizationHandler` and `ContactManagerAuthorizationHandler` are added as singletons.</span></span> <span data-ttu-id="d73b2-411">Si tratta di singleton perché non usano EF e tutte le informazioni necessarie si trovano nel `Context` parametro del `HandleRequirementAsync` metodo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-411">They're singletons because they don't use EF and all the information needed is in the `Context` parameter of the `HandleRequirementAsync` method.</span></span>

## <a name="support-authorization"></a><span data-ttu-id="d73b2-412">Autorizzazione supporto</span><span class="sxs-lookup"><span data-stu-id="d73b2-412">Support authorization</span></span>

<span data-ttu-id="d73b2-413">In questa sezione si aggiornano le Razor pagine e si aggiunge una classe dei requisiti delle operazioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-413">In this section, you update the Razor Pages and add an operations requirements class.</span></span>

### <a name="review-the-contact-operations-requirements-class"></a><span data-ttu-id="d73b2-414">Esaminare la classe dei requisiti delle operazioni di contatto</span><span class="sxs-lookup"><span data-stu-id="d73b2-414">Review the contact operations requirements class</span></span>

<span data-ttu-id="d73b2-415">Esaminare la `ContactOperations` classe.</span><span class="sxs-lookup"><span data-stu-id="d73b2-415">Review the `ContactOperations` class.</span></span> <span data-ttu-id="d73b2-416">Questa classe contiene i requisiti supportati dall'app:</span><span class="sxs-lookup"><span data-stu-id="d73b2-416">This class contains the requirements the app supports:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-razor-pages"></a><span data-ttu-id="d73b2-417">Creare una classe di base per le Razor pagine contatti</span><span class="sxs-lookup"><span data-stu-id="d73b2-417">Create a base class for the Contacts Razor Pages</span></span>

<span data-ttu-id="d73b2-418">Creare una classe di base che contiene i servizi utilizzati nelle Razor pagine contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-418">Create a base class that contains the services used in the contacts Razor Pages.</span></span> <span data-ttu-id="d73b2-419">La classe base inserisce il codice di inizializzazione in un'unica posizione:</span><span class="sxs-lookup"><span data-stu-id="d73b2-419">The base class puts the initialization code in one location:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/DI_BasePageModel.cs)]

<span data-ttu-id="d73b2-420">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="d73b2-420">The preceding code:</span></span>

* <span data-ttu-id="d73b2-421">Aggiunge il `IAuthorizationService` servizio per accedere ai gestori di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-421">Adds the `IAuthorizationService` service to access to the authorization handlers.</span></span>
* <span data-ttu-id="d73b2-422">Aggiunge il Identity `UserManager` servizio.</span><span class="sxs-lookup"><span data-stu-id="d73b2-422">Adds the Identity `UserManager` service.</span></span>
* <span data-ttu-id="d73b2-423">Aggiungere l'oggetto `ApplicationDbContext`.</span><span class="sxs-lookup"><span data-stu-id="d73b2-423">Add the `ApplicationDbContext`.</span></span>

### <a name="update-the-createmodel"></a><span data-ttu-id="d73b2-424">Aggiornare CreateModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-424">Update the CreateModel</span></span>

<span data-ttu-id="d73b2-425">Aggiornare il costruttore del modello di pagina create per usare la `DI_BasePageModel` classe di base:</span><span class="sxs-lookup"><span data-stu-id="d73b2-425">Update the create page model constructor to use the `DI_BasePageModel` base class:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

<span data-ttu-id="d73b2-426">Aggiornare il `CreateModel.OnPostAsync` metodo a:</span><span class="sxs-lookup"><span data-stu-id="d73b2-426">Update the `CreateModel.OnPostAsync` method to:</span></span>

* <span data-ttu-id="d73b2-427">Aggiungere l'ID utente al `Contact` modello.</span><span class="sxs-lookup"><span data-stu-id="d73b2-427">Add the user ID to the `Contact` model.</span></span>
* <span data-ttu-id="d73b2-428">Chiamare il gestore autorizzazioni per verificare che l'utente disponga dell'autorizzazione per la creazione di contatti.</span><span class="sxs-lookup"><span data-stu-id="d73b2-428">Call the authorization handler to verify the user has permission to create contacts.</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a><span data-ttu-id="d73b2-429">Aggiornare IndexModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-429">Update the IndexModel</span></span>

<span data-ttu-id="d73b2-430">Aggiornare il `OnGetAsync` metodo in modo da visualizzare solo i contatti approvati per gli utenti generali:</span><span class="sxs-lookup"><span data-stu-id="d73b2-430">Update the `OnGetAsync` method so only approved contacts are shown to general users:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a><span data-ttu-id="d73b2-431">Aggiornare EditModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-431">Update the EditModel</span></span>

<span data-ttu-id="d73b2-432">Aggiungere un gestore autorizzazioni per verificare che l'utente sia proprietario del contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-432">Add an authorization handler to verify the user owns the contact.</span></span> <span data-ttu-id="d73b2-433">Poiché l'autorizzazione della risorsa viene convalidata, l' `[Authorize]` attributo non è sufficiente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-433">Because resource authorization is being validated, the `[Authorize]` attribute is not enough.</span></span> <span data-ttu-id="d73b2-434">L'app non ha accesso alla risorsa quando vengono valutati gli attributi.</span><span class="sxs-lookup"><span data-stu-id="d73b2-434">The app doesn't have access to the resource when attributes are evaluated.</span></span> <span data-ttu-id="d73b2-435">L'autorizzazione basata sulle risorse deve essere imperativa.</span><span class="sxs-lookup"><span data-stu-id="d73b2-435">Resource-based authorization must be imperative.</span></span> <span data-ttu-id="d73b2-436">I controlli devono essere eseguiti una volta che l'app può accedere alla risorsa, eseguendo il caricamento nel modello di pagina o caricando l'app all'interno del gestore stesso.</span><span class="sxs-lookup"><span data-stu-id="d73b2-436">Checks must be performed once the app has access to the resource, either by loading it in the page model or by loading it within the handler itself.</span></span> <span data-ttu-id="d73b2-437">Si accede spesso alla risorsa passando la chiave della risorsa.</span><span class="sxs-lookup"><span data-stu-id="d73b2-437">You frequently access the resource by passing in the resource key.</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a><span data-ttu-id="d73b2-438">Aggiornare DeleteModel</span><span class="sxs-lookup"><span data-stu-id="d73b2-438">Update the DeleteModel</span></span>

<span data-ttu-id="d73b2-439">Aggiornare il modello di pagina Elimina per utilizzare il gestore autorizzazione per verificare che l'utente disponga dell'autorizzazione DELETE per il contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-439">Update the delete page model to use the authorization handler to verify the user has delete permission on the contact.</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a><span data-ttu-id="d73b2-440">Inserire il servizio di autorizzazione nelle viste</span><span class="sxs-lookup"><span data-stu-id="d73b2-440">Inject the authorization service into the views</span></span>

<span data-ttu-id="d73b2-441">Attualmente, l'interfaccia utente Mostra i collegamenti modifica ed Elimina per i contatti che non possono essere modificati dall'utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-441">Currently, the UI shows edit and delete links for contacts the user can't modify.</span></span>

<span data-ttu-id="d73b2-442">Inserire il servizio di autorizzazione nel file *views/_ViewImports. cshtml* in modo che sia disponibile per tutte le visualizzazioni:</span><span class="sxs-lookup"><span data-stu-id="d73b2-442">Inject the authorization service in the *Views/_ViewImports.cshtml* file so it's available to all views:</span></span>

[!code-cshtml[](secure-data/samples/final2.1/Pages/_ViewImports.cshtml?highlight=6-99)]

<span data-ttu-id="d73b2-443">Il markup precedente aggiunge diverse `using` istruzioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-443">The preceding markup adds several `using` statements.</span></span>

<span data-ttu-id="d73b2-444">Aggiornare i collegamenti **Edit** ed **Delete** in *pages/Contacts/index. cshtml* in modo che vengano sottoposti a rendering solo per gli utenti con le autorizzazioni appropriate:</span><span class="sxs-lookup"><span data-stu-id="d73b2-444">Update the **Edit** and **Delete** links in *Pages/Contacts/Index.cshtml* so they're only rendered for users with the appropriate permissions:</span></span>

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> <span data-ttu-id="d73b2-445">Nascondere i collegamenti da utenti che non dispongono dell'autorizzazione per modificare i dati non protegge l'app.</span><span class="sxs-lookup"><span data-stu-id="d73b2-445">Hiding links from users that don't have permission to change data doesn't secure the app.</span></span> <span data-ttu-id="d73b2-446">Nascondere i collegamenti rende l'app più intuitiva visualizzando solo i collegamenti validi.</span><span class="sxs-lookup"><span data-stu-id="d73b2-446">Hiding links makes the app more user-friendly by displaying only valid links.</span></span> <span data-ttu-id="d73b2-447">Gli utenti possono hackerare gli URL generati per richiamare le operazioni di modifica ed eliminazione sui dati di cui non sono proprietari.</span><span class="sxs-lookup"><span data-stu-id="d73b2-447">Users can hack the generated URLs to invoke edit and delete operations on data they don't own.</span></span> <span data-ttu-id="d73b2-448">La Razor pagina o il controller deve applicare i controlli di accesso per proteggere i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-448">The Razor Page or controller must enforce access checks to secure the data.</span></span>

### <a name="update-details"></a><span data-ttu-id="d73b2-449">Dettagli aggiornamento</span><span class="sxs-lookup"><span data-stu-id="d73b2-449">Update Details</span></span>

<span data-ttu-id="d73b2-450">Aggiornare la visualizzazione dettagli in modo che i responsabili possano approvare o rifiutare i contatti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-450">Update the details view so managers can approve or reject contacts:</span></span>

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml?name=snippet)]

<span data-ttu-id="d73b2-451">Aggiornare il modello della pagina dei dettagli:</span><span class="sxs-lookup"><span data-stu-id="d73b2-451">Update the details page model:</span></span>

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a><span data-ttu-id="d73b2-452">Aggiungere o rimuovere un utente in un ruolo</span><span class="sxs-lookup"><span data-stu-id="d73b2-452">Add or remove a user to a role</span></span>

<span data-ttu-id="d73b2-453">Per informazioni su, vedere [questo problema](https://github.com/dotnet/AspNetCore.Docs/issues/8502) :</span><span class="sxs-lookup"><span data-stu-id="d73b2-453">See [this issue](https://github.com/dotnet/AspNetCore.Docs/issues/8502) for information on:</span></span>

* <span data-ttu-id="d73b2-454">Rimozione dei privilegi da un utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-454">Removing privileges from a user.</span></span> <span data-ttu-id="d73b2-455">Ad esempio, disattivare un utente in un'app di chat.</span><span class="sxs-lookup"><span data-stu-id="d73b2-455">For example, muting a user in a chat app.</span></span>
* <span data-ttu-id="d73b2-456">Aggiunta di privilegi a un utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-456">Adding privileges to a user.</span></span>

## <a name="test-the-completed-app"></a><span data-ttu-id="d73b2-457">Testare l'app completata</span><span class="sxs-lookup"><span data-stu-id="d73b2-457">Test the completed app</span></span>

<span data-ttu-id="d73b2-458">Se non è già stata impostata una password per gli account utente di cui è stato eseguito il seeding, usare lo [strumento Gestione segreta](xref:security/app-secrets#secret-manager) per impostare una password:</span><span class="sxs-lookup"><span data-stu-id="d73b2-458">If you haven't already set a password for seeded user accounts, use the [Secret Manager tool](xref:security/app-secrets#secret-manager) to set a password:</span></span>

* <span data-ttu-id="d73b2-459">Scegliere una password complessa: usare otto o più caratteri e almeno un carattere maiuscolo, un numero e un simbolo.</span><span class="sxs-lookup"><span data-stu-id="d73b2-459">Choose a strong password: Use eight or more characters and at least one upper-case character, number, and symbol.</span></span> <span data-ttu-id="d73b2-460">Ad esempio, `Passw0rd!` soddisfa i requisiti per le password complesse.</span><span class="sxs-lookup"><span data-stu-id="d73b2-460">For example, `Passw0rd!` meets the strong password requirements.</span></span>
* <span data-ttu-id="d73b2-461">Eseguire il comando seguente dalla cartella del progetto, dove `<PW>` è la password:</span><span class="sxs-lookup"><span data-stu-id="d73b2-461">Execute the following command from the project's folder, where `<PW>` is the password:</span></span>

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

* <span data-ttu-id="d73b2-462">Eliminare e aggiornare il database</span><span class="sxs-lookup"><span data-stu-id="d73b2-462">Drop and update the Database</span></span>

  ```dotnetcli
  dotnet ef database drop -f
  dotnet ef database update  
  ```

* <span data-ttu-id="d73b2-463">Riavviare l'app per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="d73b2-463">Restart the app to seed the database.</span></span>

<span data-ttu-id="d73b2-464">Un modo semplice per testare l'app completata consiste nell'avviare tre diversi browser (o sessioni in incognito/InPrivate).</span><span class="sxs-lookup"><span data-stu-id="d73b2-464">An easy way to test the completed app is to launch three different browsers (or incognito/InPrivate sessions).</span></span> <span data-ttu-id="d73b2-465">In un browser registrare un nuovo utente (ad esempio, `test@example.com` ).</span><span class="sxs-lookup"><span data-stu-id="d73b2-465">In one browser, register a new user (for example, `test@example.com`).</span></span> <span data-ttu-id="d73b2-466">Accedere a ogni browser con un altro utente.</span><span class="sxs-lookup"><span data-stu-id="d73b2-466">Sign in to each browser with a different user.</span></span> <span data-ttu-id="d73b2-467">Verificare le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="d73b2-467">Verify the following operations:</span></span>

* <span data-ttu-id="d73b2-468">Gli utenti registrati possono visualizzare tutti i dati di contatto approvati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-468">Registered users can view all of the approved contact data.</span></span>
* <span data-ttu-id="d73b2-469">Gli utenti registrati possono modificare/eliminare i propri dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-469">Registered users can edit/delete their own data.</span></span>
* <span data-ttu-id="d73b2-470">I responsabili possono approvare/rifiutare i dati di contatto.</span><span class="sxs-lookup"><span data-stu-id="d73b2-470">Managers can approve/reject contact data.</span></span> <span data-ttu-id="d73b2-471">La `Details` visualizzazione Mostra i pulsanti **approva** e **rifiuta** .</span><span class="sxs-lookup"><span data-stu-id="d73b2-471">The `Details` view shows **Approve** and **Reject** buttons.</span></span>
* <span data-ttu-id="d73b2-472">Gli amministratori possono approvare/rifiutare e modificare/eliminare tutti i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-472">Administrators can approve/reject and edit/delete all data.</span></span>

| <span data-ttu-id="d73b2-473">User</span><span class="sxs-lookup"><span data-stu-id="d73b2-473">User</span></span>                | <span data-ttu-id="d73b2-474">Seeding dall'app</span><span class="sxs-lookup"><span data-stu-id="d73b2-474">Seeded by the app</span></span> | <span data-ttu-id="d73b2-475">Opzioni</span><span class="sxs-lookup"><span data-stu-id="d73b2-475">Options</span></span>                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | <span data-ttu-id="d73b2-476">No</span><span class="sxs-lookup"><span data-stu-id="d73b2-476">No</span></span>                | <span data-ttu-id="d73b2-477">Modificare/eliminare i dati personali.</span><span class="sxs-lookup"><span data-stu-id="d73b2-477">Edit/delete the own data.</span></span>                |
| manager@contoso.com | <span data-ttu-id="d73b2-478">Sì</span><span class="sxs-lookup"><span data-stu-id="d73b2-478">Yes</span></span>               | <span data-ttu-id="d73b2-479">Approva/rifiuta e modifica/elimina i dati personali.</span><span class="sxs-lookup"><span data-stu-id="d73b2-479">Approve/reject and edit/delete own data.</span></span> |
| admin@contoso.com   | <span data-ttu-id="d73b2-480">Sì</span><span class="sxs-lookup"><span data-stu-id="d73b2-480">Yes</span></span>               | <span data-ttu-id="d73b2-481">Approva/rifiuta e modifica/elimina tutti i dati.</span><span class="sxs-lookup"><span data-stu-id="d73b2-481">Approve/reject and edit/delete all data.</span></span> |

<span data-ttu-id="d73b2-482">Creare un contatto nel browser dell'amministratore.</span><span class="sxs-lookup"><span data-stu-id="d73b2-482">Create a contact in the administrator's browser.</span></span> <span data-ttu-id="d73b2-483">Copiare l'URL da eliminare e modificare dal contatto amministratore.</span><span class="sxs-lookup"><span data-stu-id="d73b2-483">Copy the URL for delete and edit from the administrator contact.</span></span> <span data-ttu-id="d73b2-484">Incollare questi collegamenti nel browser dell'utente test per verificare che l'utente test non possa eseguire queste operazioni.</span><span class="sxs-lookup"><span data-stu-id="d73b2-484">Paste these links into the test user's browser to verify the test user can't perform these operations.</span></span>

## <a name="create-the-starter-app"></a><span data-ttu-id="d73b2-485">Creare l'app Starter</span><span class="sxs-lookup"><span data-stu-id="d73b2-485">Create the starter app</span></span>

* <span data-ttu-id="d73b2-486">Creare un' Razor app per le pagine denominata "ContactManager"</span><span class="sxs-lookup"><span data-stu-id="d73b2-486">Create a Razor Pages app named "ContactManager"</span></span>
  * <span data-ttu-id="d73b2-487">Creare l'app con **singoli account utente**.</span><span class="sxs-lookup"><span data-stu-id="d73b2-487">Create the app with **Individual User Accounts**.</span></span>
  * <span data-ttu-id="d73b2-488">Denominarlo "ContactManager" in modo che lo spazio dei nomi corrisponda allo spazio dei nomi usato nell'esempio.</span><span class="sxs-lookup"><span data-stu-id="d73b2-488">Name it "ContactManager" so the namespace matches the namespace used in the sample.</span></span>
  * <span data-ttu-id="d73b2-489">`-uld` Specifica il database locale anziché SQLite</span><span class="sxs-lookup"><span data-stu-id="d73b2-489">`-uld` specifies LocalDB instead of SQLite</span></span>

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* <span data-ttu-id="d73b2-490">Aggiungi *modelli/Contact. cs*:</span><span class="sxs-lookup"><span data-stu-id="d73b2-490">Add *Models/Contact.cs*:</span></span>

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* <span data-ttu-id="d73b2-491">Impalcatura del `Contact` modello.</span><span class="sxs-lookup"><span data-stu-id="d73b2-491">Scaffold the `Contact` model.</span></span>
* <span data-ttu-id="d73b2-492">Creare la migrazione iniziale e aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="d73b2-492">Create initial migration and update the database:</span></span>

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
  dotnet ef database drop -f
  dotnet ef migrations add initial
  dotnet ef database update
  ```

* <span data-ttu-id="d73b2-493">Aggiornare l'ancoraggio **ContactManager** nel file *pages/_Layout. cshtml* :</span><span class="sxs-lookup"><span data-stu-id="d73b2-493">Update the **ContactManager** anchor in the *Pages/_Layout.cshtml* file:</span></span>

  ```cshtml
  <a asp-page="/Contacts/Index" class="navbar-brand">ContactManager</a>
  ```

* <span data-ttu-id="d73b2-494">Testare l'app creando, modificando ed eliminando un contatto</span><span class="sxs-lookup"><span data-stu-id="d73b2-494">Test the app by creating, editing, and deleting a contact</span></span>

### <a name="seed-the-database"></a><span data-ttu-id="d73b2-495">Specificare il valore di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="d73b2-495">Seed the database</span></span>

<span data-ttu-id="d73b2-496">Aggiungere la classe [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/starter2.1/Data/SeedData.cs) alla cartella *Data* .</span><span class="sxs-lookup"><span data-stu-id="d73b2-496">Add the [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/secure-data/samples/starter2.1/Data/SeedData.cs) class to the *Data* folder.</span></span>

<span data-ttu-id="d73b2-497">Chiama `SeedData.Initialize` da `Main` :</span><span class="sxs-lookup"><span data-stu-id="d73b2-497">Call `SeedData.Initialize` from `Main`:</span></span>

[!code-csharp[](secure-data/samples/starter2.1/Program.cs?name=snippet)]

<span data-ttu-id="d73b2-498">Verificare che l'app abbia sottoposta a seeding il database.</span><span class="sxs-lookup"><span data-stu-id="d73b2-498">Test that the app seeded the database.</span></span> <span data-ttu-id="d73b2-499">Se sono presenti righe nel database Contact, il metodo seed non viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="d73b2-499">If there are any rows in the contact DB, the seed method doesn't run.</span></span>

::: moniker-end

<a name="secure-data-add-resources-label"></a>

### <a name="additional-resources"></a><span data-ttu-id="d73b2-500">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="d73b2-500">Additional resources</span></span>

* [<span data-ttu-id="d73b2-501">Creare un'app Web .NET Core e database SQL nel Servizio app di Azure</span><span class="sxs-lookup"><span data-stu-id="d73b2-501">Build a .NET Core and SQL Database web app in Azure App Service</span></span>](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)
* <span data-ttu-id="d73b2-502">[ASP.NET Core Lab di autorizzazione](https://github.com/blowdart/AspNetAuthorizationWorkshop).</span><span class="sxs-lookup"><span data-stu-id="d73b2-502">[ASP.NET Core Authorization Lab](https://github.com/blowdart/AspNetAuthorizationWorkshop).</span></span> <span data-ttu-id="d73b2-503">Questo Lab illustra in modo più dettagliato le funzionalità di sicurezza introdotte in questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="d73b2-503">This lab goes into more detail on the security features introduced in this tutorial.</span></span>
* <xref:security/authorization/introduction>
* [<span data-ttu-id="d73b2-504">Autorizzazione personalizzata basata su criteri</span><span class="sxs-lookup"><span data-stu-id="d73b2-504">Custom policy-based authorization</span></span>](xref:security/authorization/policies)
