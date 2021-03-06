---
title: Proteggere le Blazor Server app ASP.NET Core
author: guardrex
description: Informazioni su come proteggere Blazor Server le app come ASP.NET Core applicazioni.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/06/2020
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
uid: blazor/security/server/index
ms.openlocfilehash: 147ebbeb84e1755307d627ef428d92d1b0248c74
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394863"
---
# <a name="secure-aspnet-core-blazor-server-apps"></a><span data-ttu-id="56194-103">Proteggere le Blazor Server app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="56194-103">Secure ASP.NET Core Blazor Server apps</span></span>

<span data-ttu-id="56194-104">Blazor Server le app sono configurate per la sicurezza in modo analogo alle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="56194-104">Blazor Server apps are configured for security in the same manner as ASP.NET Core apps.</span></span> <span data-ttu-id="56194-105">Per ulteriori informazioni, vedere gli articoli in <xref:security/index> .</span><span class="sxs-lookup"><span data-stu-id="56194-105">For more information, see the articles under <xref:security/index>.</span></span> <span data-ttu-id="56194-106">Gli argomenti di questa panoramica si applicano in modo specifico a Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="56194-106">Topics under this overview apply specifically to Blazor Server.</span></span>

## <a name="blazor-server-project-template"></a><span data-ttu-id="56194-107">Blazor Server modello di progetto</span><span class="sxs-lookup"><span data-stu-id="56194-107">Blazor Server project template</span></span>

<span data-ttu-id="56194-108">Il [ Blazor Server modello di progetto](xref:blazor/project-structure) può essere configurato per l'autenticazione quando viene creato il progetto.</span><span class="sxs-lookup"><span data-stu-id="56194-108">The [Blazor Server project template](xref:blazor/project-structure) can be configured for authentication when the project is created.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="56194-109">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="56194-109">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="56194-110">Seguire le istruzioni di Visual Studio in <xref:blazor/tooling> per creare un nuovo Blazor Server progetto con un meccanismo di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="56194-110">Follow the Visual Studio guidance in <xref:blazor/tooling> to create a new Blazor Server project with an authentication mechanism.</span></span>

<span data-ttu-id="56194-111">Dopo aver scelto il modello **Blazor Server app** nella finestra di dialogo **Crea un nuovo ASP.NET Core applicazione Web** , selezionare **Cambia** in **autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="56194-111">After choosing the **Blazor Server App** template in the **Create a new ASP.NET Core Web Application** dialog, select **Change** under **Authentication**.</span></span>

<span data-ttu-id="56194-112">Viene visualizzata una finestra di dialogo che offre lo stesso set di meccanismi di autenticazione disponibili per altri progetti ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="56194-112">A dialog opens to offer the same set of authentication mechanisms available for other ASP.NET Core projects:</span></span>

* <span data-ttu-id="56194-113">**Nessuna autenticazione**</span><span class="sxs-lookup"><span data-stu-id="56194-113">**No Authentication**</span></span>
* <span data-ttu-id="56194-114">**Account utente singoli**: è possibile archiviare gli account utente:</span><span class="sxs-lookup"><span data-stu-id="56194-114">**Individual User Accounts**: User accounts can be stored:</span></span>
  * <span data-ttu-id="56194-115">All'interno dell'app usando il [Identity](xref:security/authentication/identity) sistema di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="56194-115">Within the app using ASP.NET Core's [Identity](xref:security/authentication/identity) system.</span></span>
  * <span data-ttu-id="56194-116">Con [Azure AD B2C](xref:security/authentication/azure-ad-b2c).</span><span class="sxs-lookup"><span data-stu-id="56194-116">With [Azure AD B2C](xref:security/authentication/azure-ad-b2c).</span></span>
* <span data-ttu-id="56194-117">**Account aziendali o dell'Istituto di istruzione**</span><span class="sxs-lookup"><span data-stu-id="56194-117">**Work or School Accounts**</span></span>
* <span data-ttu-id="56194-118">**Autenticazione di Windows**</span><span class="sxs-lookup"><span data-stu-id="56194-118">**Windows Authentication**</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="56194-119">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="56194-119">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="56194-120">Seguire le indicazioni Visual Studio Code in <xref:blazor/tooling> per creare un nuovo Blazor Server progetto con un meccanismo di autenticazione:</span><span class="sxs-lookup"><span data-stu-id="56194-120">Follow the Visual Studio Code guidance in <xref:blazor/tooling> to create a new Blazor Server project with an authentication mechanism:</span></span>

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

<span data-ttu-id="56194-121">I valori di autenticazione consentiti (`{AUTHENTICATION}`) sono indicati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="56194-121">Permissible authentication values (`{AUTHENTICATION}`) are shown in the following table.</span></span>

| <span data-ttu-id="56194-122">Meccanismo di autenticazione</span><span class="sxs-lookup"><span data-stu-id="56194-122">Authentication mechanism</span></span> | <span data-ttu-id="56194-123">Descrizione</span><span class="sxs-lookup"><span data-stu-id="56194-123">Description</span></span> |
| ------------------------ | ----------- |
| <span data-ttu-id="56194-124">`None` (impostazione predefinita)</span><span class="sxs-lookup"><span data-stu-id="56194-124">`None` (default)</span></span>         | <span data-ttu-id="56194-125">Nessuna autenticazione</span><span class="sxs-lookup"><span data-stu-id="56194-125">No authentication</span></span> |
| `Individual`             | <span data-ttu-id="56194-126">Utenti archiviati nell'app con ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="56194-126">Users stored in the app with ASP.NET Core Identity</span></span> |
| `IndividualB2C`          | <span data-ttu-id="56194-127">Utenti archiviati in [Azure ad B2C](xref:security/authentication/azure-ad-b2c)</span><span class="sxs-lookup"><span data-stu-id="56194-127">Users stored in [Azure AD B2C](xref:security/authentication/azure-ad-b2c)</span></span> |
| `SingleOrg`              | <span data-ttu-id="56194-128">Autenticazione organizzativa per un singolo tenant</span><span class="sxs-lookup"><span data-stu-id="56194-128">Organizational authentication for a single tenant</span></span> |
| `MultiOrg`               | <span data-ttu-id="56194-129">Autenticazione organizzativa per più tenant</span><span class="sxs-lookup"><span data-stu-id="56194-129">Organizational authentication for multiple tenants</span></span> |
| `Windows`                | <span data-ttu-id="56194-130">Autenticazione di Windows</span><span class="sxs-lookup"><span data-stu-id="56194-130">Windows Authentication</span></span> |

<span data-ttu-id="56194-131">Usando l' `-o|--output` opzione, il comando usa il valore fornito per il `{APP NAME}` segnaposto per:</span><span class="sxs-lookup"><span data-stu-id="56194-131">Using the `-o|--output` option, the command uses the value provided for the `{APP NAME}` placeholder to:</span></span>

* <span data-ttu-id="56194-132">Creare una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="56194-132">Create a folder for the project.</span></span>
* <span data-ttu-id="56194-133">Assegnare un nome al progetto.</span><span class="sxs-lookup"><span data-stu-id="56194-133">Name the project.</span></span>

<span data-ttu-id="56194-134">Per ulteriori informazioni, vedere il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando nella Guida di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="56194-134">For more information, see the [`dotnet new`](/dotnet/core/tools/dotnet-new) command in the .NET Core Guide.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="56194-135">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="56194-135">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="56194-136">Seguire le indicazioni Visual Studio per Mac in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="56194-136">Follow the Visual Studio for Mac guidance in <xref:blazor/tooling>.</span></span>

1. <span data-ttu-id="56194-137">Nel passaggio **configurare la nuova Blazor Server app** selezionare **autenticazione singola (in-app) nell'elenco a** discesa **autenticazione** .</span><span class="sxs-lookup"><span data-stu-id="56194-137">On the **Configure your new Blazor Server App** step, select **Individual Authentication (in-app)** from the **Authentication** drop down.</span></span>

1. <span data-ttu-id="56194-138">L'app viene creata per i singoli utenti archiviati nell'app con ASP.NET Core Identity .</span><span class="sxs-lookup"><span data-stu-id="56194-138">The app is created for individual users stored in the app with ASP.NET Core Identity.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="56194-139">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="56194-139">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="56194-140">Creare un nuovo Blazor Server progetto con un meccanismo di autenticazione usando il comando seguente in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="56194-140">Create a new Blazor Server project with an authentication mechanism using the following command in a command shell:</span></span>

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

<span data-ttu-id="56194-141">I valori di autenticazione consentiti (`{AUTHENTICATION}`) sono indicati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="56194-141">Permissible authentication values (`{AUTHENTICATION}`) are shown in the following table.</span></span>

| <span data-ttu-id="56194-142">Meccanismo di autenticazione</span><span class="sxs-lookup"><span data-stu-id="56194-142">Authentication mechanism</span></span> | <span data-ttu-id="56194-143">Descrizione</span><span class="sxs-lookup"><span data-stu-id="56194-143">Description</span></span> |
| ------------------------ | ----------- |
| <span data-ttu-id="56194-144">`None` (impostazione predefinita)</span><span class="sxs-lookup"><span data-stu-id="56194-144">`None` (default)</span></span>         | <span data-ttu-id="56194-145">Nessuna autenticazione</span><span class="sxs-lookup"><span data-stu-id="56194-145">No authentication</span></span> |
| `Individual`             | <span data-ttu-id="56194-146">Utenti archiviati nell'app con ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="56194-146">Users stored in the app with ASP.NET Core Identity</span></span> |
| `IndividualB2C`          | <span data-ttu-id="56194-147">Utenti archiviati in [Azure ad B2C](xref:security/authentication/azure-ad-b2c)</span><span class="sxs-lookup"><span data-stu-id="56194-147">Users stored in [Azure AD B2C](xref:security/authentication/azure-ad-b2c)</span></span> |
| `SingleOrg`              | <span data-ttu-id="56194-148">Autenticazione organizzativa per un singolo tenant</span><span class="sxs-lookup"><span data-stu-id="56194-148">Organizational authentication for a single tenant</span></span> |
| `MultiOrg`               | <span data-ttu-id="56194-149">Autenticazione organizzativa per più tenant</span><span class="sxs-lookup"><span data-stu-id="56194-149">Organizational authentication for multiple tenants</span></span> |
| `Windows`                | <span data-ttu-id="56194-150">Autenticazione di Windows</span><span class="sxs-lookup"><span data-stu-id="56194-150">Windows Authentication</span></span> |

<span data-ttu-id="56194-151">Usando l' `-o|--output` opzione, il comando usa il valore fornito per il `{APP NAME}` segnaposto per:</span><span class="sxs-lookup"><span data-stu-id="56194-151">Using the `-o|--output` option, the command uses the value provided for the `{APP NAME}` placeholder to:</span></span>

* <span data-ttu-id="56194-152">Creare una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="56194-152">Create a folder for the project.</span></span>
* <span data-ttu-id="56194-153">Assegnare un nome al progetto.</span><span class="sxs-lookup"><span data-stu-id="56194-153">Name the project.</span></span>

<span data-ttu-id="56194-154">Per altre informazioni:</span><span class="sxs-lookup"><span data-stu-id="56194-154">For more information:</span></span>

* <span data-ttu-id="56194-155">Vedere il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando nella Guida di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="56194-155">See the [`dotnet new`](/dotnet/core/tools/dotnet-new) command in the .NET Core Guide.</span></span>
* <span data-ttu-id="56194-156">Eseguire il comando della Guida per il Blazor Server modello ( `blazorserver` ) in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="56194-156">Execute the help command for the Blazor Server template (`blazorserver`) in a command shell:</span></span>

  ```dotnetcli
  dotnet new blazorserver --help
  ```

---

## <a name="scaffold-identity"></a><span data-ttu-id="56194-157">Scaffold Identity</span><span class="sxs-lookup"><span data-stu-id="56194-157">Scaffold Identity</span></span>

<span data-ttu-id="56194-158">Impalcatura Identity in un Blazor Server progetto:</span><span class="sxs-lookup"><span data-stu-id="56194-158">Scaffold Identity into a Blazor Server project:</span></span>

* <span data-ttu-id="56194-159">[Senza autorizzazione esistente](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-without-existing-authorization).</span><span class="sxs-lookup"><span data-stu-id="56194-159">[Without existing authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-without-existing-authorization).</span></span>
* <span data-ttu-id="56194-160">[Con autorizzazione](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-with-authorization).</span><span class="sxs-lookup"><span data-stu-id="56194-160">[With authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-with-authorization).</span></span>

## <a name="additional-claims-and-tokens-from-external-providers"></a><span data-ttu-id="56194-161">Attestazioni e token aggiuntivi da provider esterni</span><span class="sxs-lookup"><span data-stu-id="56194-161">Additional claims and tokens from external providers</span></span>

<span data-ttu-id="56194-162">Per archiviare attestazioni aggiuntive da provider esterni, vedere <xref:security/authentication/social/additional-claims> .</span><span class="sxs-lookup"><span data-stu-id="56194-162">To store additional claims from external providers, see <xref:security/authentication/social/additional-claims>.</span></span>

## <a name="azure-app-service-on-linux-with-identity-server"></a><span data-ttu-id="56194-163">Servizio app Azure in Linux con Identity Server</span><span class="sxs-lookup"><span data-stu-id="56194-163">Azure App Service on Linux with Identity Server</span></span>

<span data-ttu-id="56194-164">Specificare l'autorità emittente in modo esplicito quando si distribuisce in app Azure servizio in Linux con Identity Server.</span><span class="sxs-lookup"><span data-stu-id="56194-164">Specify the issuer explicitly when deploying to Azure App Service on Linux with Identity Server.</span></span> <span data-ttu-id="56194-165">Per altre informazioni, vedere <xref:security/authentication/identity/spa#azure-app-service-on-linux>.</span><span class="sxs-lookup"><span data-stu-id="56194-165">For more information, see <xref:security/authentication/identity/spa#azure-app-service-on-linux>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="56194-166">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="56194-166">Additional resources</span></span>

* [<span data-ttu-id="56194-167">Avvio rapido: Aggiungere l'accesso con Microsoft a un'app Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="56194-167">Quickstart: Add sign-in with Microsoft to an ASP.NET Core web app</span></span>](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp)
* [<span data-ttu-id="56194-168">Avvio rapido: Proteggere un'API Web ASP.NET Core con Microsoft Identity Platform</span><span class="sxs-lookup"><span data-stu-id="56194-168">Quickstart: Protect an ASP.NET Core web API with Microsoft identity platform</span></span>](/azure/active-directory/develop/quickstart-v2-aspnet-core-web-api)
* <span data-ttu-id="56194-169"><xref:host-and-deploy/proxy-load-balancer>: Include informazioni aggiuntive su:</span><span class="sxs-lookup"><span data-stu-id="56194-169"><xref:host-and-deploy/proxy-load-balancer>: Includes guidance on:</span></span>
  * <span data-ttu-id="56194-170">Uso del middleware intestazioni con inoltro per mantenere le informazioni sullo schema HTTPS tra i server proxy e le reti interne.</span><span class="sxs-lookup"><span data-stu-id="56194-170">Using Forwarded Headers Middleware to preserve HTTPS scheme information across proxy servers and internal networks.</span></span>
  * <span data-ttu-id="56194-171">Scenari e casi d'uso aggiuntivi, tra cui la configurazione manuale dello schema, la modifica del percorso della richiesta per il routing delle richieste corretto e l'inoltro dello schema di richiesta per i proxy inversi Linux e non IIS.</span><span class="sxs-lookup"><span data-stu-id="56194-171">Additional scenarios and use cases, including manual scheme configuration, request path changes for correct request routing, and forwarding the request scheme for Linux and non-IIS reverse proxies.</span></span>
