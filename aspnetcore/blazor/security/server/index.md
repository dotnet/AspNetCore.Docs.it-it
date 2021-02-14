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
ms.openlocfilehash: 5a3d3c6e06653de7f0d01565444d37013f347a5b
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280311"
---
# <a name="secure-aspnet-core-blazor-server-apps"></a><span data-ttu-id="85b29-103">Proteggere le Blazor Server app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="85b29-103">Secure ASP.NET Core Blazor Server apps</span></span>

<span data-ttu-id="85b29-104">Blazor Server le app sono configurate per la sicurezza in modo analogo alle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="85b29-104">Blazor Server apps are configured for security in the same manner as ASP.NET Core apps.</span></span> <span data-ttu-id="85b29-105">Per ulteriori informazioni, vedere gli articoli in <xref:security/index> .</span><span class="sxs-lookup"><span data-stu-id="85b29-105">For more information, see the articles under <xref:security/index>.</span></span> <span data-ttu-id="85b29-106">Gli argomenti di questa panoramica si applicano in modo specifico a Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="85b29-106">Topics under this overview apply specifically to Blazor Server.</span></span>

## <a name="blazor-server-project-template"></a><span data-ttu-id="85b29-107">Blazor Server modello di progetto</span><span class="sxs-lookup"><span data-stu-id="85b29-107">Blazor Server project template</span></span>

<span data-ttu-id="85b29-108">Il Blazor Server modello di progetto può essere configurato per l'autenticazione quando viene creato il progetto.</span><span class="sxs-lookup"><span data-stu-id="85b29-108">The Blazor Server project template can be configured for authentication when the project is created.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="85b29-109">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="85b29-109">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="85b29-110">Seguire le istruzioni di Visual Studio in <xref:blazor/tooling> per creare un nuovo Blazor Server progetto con un meccanismo di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="85b29-110">Follow the Visual Studio guidance in <xref:blazor/tooling> to create a new Blazor Server project with an authentication mechanism.</span></span>

<span data-ttu-id="85b29-111">Dopo aver scelto il modello **Blazor Server app** nella finestra di dialogo **Crea un nuovo ASP.NET Core applicazione Web** , selezionare **Cambia** in **autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="85b29-111">After choosing the **Blazor Server App** template in the **Create a new ASP.NET Core Web Application** dialog, select **Change** under **Authentication**.</span></span>

<span data-ttu-id="85b29-112">Viene visualizzata una finestra di dialogo che offre lo stesso set di meccanismi di autenticazione disponibili per altri progetti ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="85b29-112">A dialog opens to offer the same set of authentication mechanisms available for other ASP.NET Core projects:</span></span>

* <span data-ttu-id="85b29-113">**Nessuna autenticazione**</span><span class="sxs-lookup"><span data-stu-id="85b29-113">**No Authentication**</span></span>
* <span data-ttu-id="85b29-114">**Account utente singoli**: è possibile archiviare gli account utente:</span><span class="sxs-lookup"><span data-stu-id="85b29-114">**Individual User Accounts**: User accounts can be stored:</span></span>
  * <span data-ttu-id="85b29-115">All'interno dell'app usando il [Identity](xref:security/authentication/identity) sistema di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="85b29-115">Within the app using ASP.NET Core's [Identity](xref:security/authentication/identity) system.</span></span>
  * <span data-ttu-id="85b29-116">Con [Azure AD B2C](xref:security/authentication/azure-ad-b2c).</span><span class="sxs-lookup"><span data-stu-id="85b29-116">With [Azure AD B2C](xref:security/authentication/azure-ad-b2c).</span></span>
* <span data-ttu-id="85b29-117">**Account aziendali o dell'Istituto di istruzione**</span><span class="sxs-lookup"><span data-stu-id="85b29-117">**Work or School Accounts**</span></span>
* <span data-ttu-id="85b29-118">**Autenticazione di Windows**</span><span class="sxs-lookup"><span data-stu-id="85b29-118">**Windows Authentication**</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="85b29-119">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="85b29-119">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="85b29-120">Seguire le indicazioni Visual Studio Code in <xref:blazor/tooling> per creare un nuovo Blazor Server progetto con un meccanismo di autenticazione:</span><span class="sxs-lookup"><span data-stu-id="85b29-120">Follow the Visual Studio Code guidance in <xref:blazor/tooling> to create a new Blazor Server project with an authentication mechanism:</span></span>

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

<span data-ttu-id="85b29-121">I valori di autenticazione consentiti (`{AUTHENTICATION}`) sono indicati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="85b29-121">Permissible authentication values (`{AUTHENTICATION}`) are shown in the following table.</span></span>

| <span data-ttu-id="85b29-122">Meccanismo di autenticazione</span><span class="sxs-lookup"><span data-stu-id="85b29-122">Authentication mechanism</span></span> | <span data-ttu-id="85b29-123">Descrizione</span><span class="sxs-lookup"><span data-stu-id="85b29-123">Description</span></span> |
| ------------------------ | ----------- |
| <span data-ttu-id="85b29-124">`None` (impostazione predefinita)</span><span class="sxs-lookup"><span data-stu-id="85b29-124">`None` (default)</span></span>         | <span data-ttu-id="85b29-125">Nessuna autenticazione</span><span class="sxs-lookup"><span data-stu-id="85b29-125">No authentication</span></span> |
| `Individual`             | <span data-ttu-id="85b29-126">Utenti archiviati nell'app con ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="85b29-126">Users stored in the app with ASP.NET Core Identity</span></span> |
| `IndividualB2C`          | <span data-ttu-id="85b29-127">Utenti archiviati in [Azure ad B2C](xref:security/authentication/azure-ad-b2c)</span><span class="sxs-lookup"><span data-stu-id="85b29-127">Users stored in [Azure AD B2C](xref:security/authentication/azure-ad-b2c)</span></span> |
| `SingleOrg`              | <span data-ttu-id="85b29-128">Autenticazione organizzativa per un singolo tenant</span><span class="sxs-lookup"><span data-stu-id="85b29-128">Organizational authentication for a single tenant</span></span> |
| `MultiOrg`               | <span data-ttu-id="85b29-129">Autenticazione organizzativa per più tenant</span><span class="sxs-lookup"><span data-stu-id="85b29-129">Organizational authentication for multiple tenants</span></span> |
| `Windows`                | <span data-ttu-id="85b29-130">Autenticazione di Windows</span><span class="sxs-lookup"><span data-stu-id="85b29-130">Windows Authentication</span></span> |

<span data-ttu-id="85b29-131">Usando l' `-o|--output` opzione, il comando usa il valore fornito per il `{APP NAME}` segnaposto per:</span><span class="sxs-lookup"><span data-stu-id="85b29-131">Using the `-o|--output` option, the command uses the value provided for the `{APP NAME}` placeholder to:</span></span>

* <span data-ttu-id="85b29-132">Creare una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="85b29-132">Create a folder for the project.</span></span>
* <span data-ttu-id="85b29-133">Assegnare un nome al progetto.</span><span class="sxs-lookup"><span data-stu-id="85b29-133">Name the project.</span></span>

<span data-ttu-id="85b29-134">Per ulteriori informazioni, vedere il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando nella Guida di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="85b29-134">For more information, see the [`dotnet new`](/dotnet/core/tools/dotnet-new) command in the .NET Core Guide.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="85b29-135">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="85b29-135">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="85b29-136">Seguire le indicazioni Visual Studio per Mac in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="85b29-136">Follow the Visual Studio for Mac guidance in <xref:blazor/tooling>.</span></span>

1. <span data-ttu-id="85b29-137">Nel passaggio **configurare la nuova Blazor Server app** selezionare **autenticazione singola (in-app) nell'elenco a** discesa **autenticazione** .</span><span class="sxs-lookup"><span data-stu-id="85b29-137">On the **Configure your new Blazor Server App** step, select **Individual Authentication (in-app)** from the **Authentication** drop down.</span></span>

1. <span data-ttu-id="85b29-138">L'app viene creata per i singoli utenti archiviati nell'app con ASP.NET Core Identity .</span><span class="sxs-lookup"><span data-stu-id="85b29-138">The app is created for individual users stored in the app with ASP.NET Core Identity.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="85b29-139">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="85b29-139">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="85b29-140">Creare un nuovo Blazor Server progetto con un meccanismo di autenticazione usando il comando seguente in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="85b29-140">Create a new Blazor Server project with an authentication mechanism using the following command in a command shell:</span></span>

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

<span data-ttu-id="85b29-141">I valori di autenticazione consentiti (`{AUTHENTICATION}`) sono indicati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="85b29-141">Permissible authentication values (`{AUTHENTICATION}`) are shown in the following table.</span></span>

| <span data-ttu-id="85b29-142">Meccanismo di autenticazione</span><span class="sxs-lookup"><span data-stu-id="85b29-142">Authentication mechanism</span></span> | <span data-ttu-id="85b29-143">Descrizione</span><span class="sxs-lookup"><span data-stu-id="85b29-143">Description</span></span> |
| ------------------------ | ----------- |
| <span data-ttu-id="85b29-144">`None` (impostazione predefinita)</span><span class="sxs-lookup"><span data-stu-id="85b29-144">`None` (default)</span></span>         | <span data-ttu-id="85b29-145">Nessuna autenticazione</span><span class="sxs-lookup"><span data-stu-id="85b29-145">No authentication</span></span> |
| `Individual`             | <span data-ttu-id="85b29-146">Utenti archiviati nell'app con ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="85b29-146">Users stored in the app with ASP.NET Core Identity</span></span> |
| `IndividualB2C`          | <span data-ttu-id="85b29-147">Utenti archiviati in [Azure ad B2C](xref:security/authentication/azure-ad-b2c)</span><span class="sxs-lookup"><span data-stu-id="85b29-147">Users stored in [Azure AD B2C](xref:security/authentication/azure-ad-b2c)</span></span> |
| `SingleOrg`              | <span data-ttu-id="85b29-148">Autenticazione organizzativa per un singolo tenant</span><span class="sxs-lookup"><span data-stu-id="85b29-148">Organizational authentication for a single tenant</span></span> |
| `MultiOrg`               | <span data-ttu-id="85b29-149">Autenticazione organizzativa per più tenant</span><span class="sxs-lookup"><span data-stu-id="85b29-149">Organizational authentication for multiple tenants</span></span> |
| `Windows`                | <span data-ttu-id="85b29-150">Autenticazione di Windows</span><span class="sxs-lookup"><span data-stu-id="85b29-150">Windows Authentication</span></span> |

<span data-ttu-id="85b29-151">Usando l' `-o|--output` opzione, il comando usa il valore fornito per il `{APP NAME}` segnaposto per:</span><span class="sxs-lookup"><span data-stu-id="85b29-151">Using the `-o|--output` option, the command uses the value provided for the `{APP NAME}` placeholder to:</span></span>

* <span data-ttu-id="85b29-152">Creare una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="85b29-152">Create a folder for the project.</span></span>
* <span data-ttu-id="85b29-153">Assegnare un nome al progetto.</span><span class="sxs-lookup"><span data-stu-id="85b29-153">Name the project.</span></span>

<span data-ttu-id="85b29-154">Per altre informazioni:</span><span class="sxs-lookup"><span data-stu-id="85b29-154">For more information:</span></span>

* <span data-ttu-id="85b29-155">Vedere il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando nella Guida di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="85b29-155">See the [`dotnet new`](/dotnet/core/tools/dotnet-new) command in the .NET Core Guide.</span></span>
* <span data-ttu-id="85b29-156">Eseguire il comando della Guida per il Blazor Server modello ( `blazorserver` ) in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="85b29-156">Execute the help command for the Blazor Server template (`blazorserver`) in a command shell:</span></span>

  ```dotnetcli
  dotnet new blazorserver --help
  ```

---

## <a name="scaffold-identity"></a><span data-ttu-id="85b29-157">Scaffold Identity</span><span class="sxs-lookup"><span data-stu-id="85b29-157">Scaffold Identity</span></span>

<span data-ttu-id="85b29-158">Impalcatura Identity in un Blazor Server progetto:</span><span class="sxs-lookup"><span data-stu-id="85b29-158">Scaffold Identity into a Blazor Server project:</span></span>

* <span data-ttu-id="85b29-159">[Senza autorizzazione esistente](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-without-existing-authorization).</span><span class="sxs-lookup"><span data-stu-id="85b29-159">[Without existing authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-without-existing-authorization).</span></span>
* <span data-ttu-id="85b29-160">[Con autorizzazione](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-with-authorization).</span><span class="sxs-lookup"><span data-stu-id="85b29-160">[With authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-with-authorization).</span></span>

## <a name="azure-app-service-on-linux-with-identity-server"></a><span data-ttu-id="85b29-161">Servizio app Azure in Linux con Identity Server</span><span class="sxs-lookup"><span data-stu-id="85b29-161">Azure App Service on Linux with Identity Server</span></span>

<span data-ttu-id="85b29-162">Specificare l'autorità emittente in modo esplicito quando si distribuisce in app Azure servizio in Linux con Identity Server.</span><span class="sxs-lookup"><span data-stu-id="85b29-162">Specify the issuer explicitly when deploying to Azure App Service on Linux with Identity Server.</span></span> <span data-ttu-id="85b29-163">Per altre informazioni, vedere <xref:security/authentication/identity/spa#azure-app-service-on-linux>.</span><span class="sxs-lookup"><span data-stu-id="85b29-163">For more information, see <xref:security/authentication/identity/spa#azure-app-service-on-linux>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="85b29-164">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="85b29-164">Additional resources</span></span>

* [<span data-ttu-id="85b29-165">Avvio rapido: Aggiungere l'accesso con Microsoft a un'app Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="85b29-165">Quickstart: Add sign-in with Microsoft to an ASP.NET Core web app</span></span>](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp)
* [<span data-ttu-id="85b29-166">Avvio rapido: Proteggere un'API Web ASP.NET Core con Microsoft Identity Platform</span><span class="sxs-lookup"><span data-stu-id="85b29-166">Quickstart: Protect an ASP.NET Core web API with Microsoft identity platform</span></span>](/azure/active-directory/develop/quickstart-v2-aspnet-core-web-api)
* <span data-ttu-id="85b29-167"><xref:host-and-deploy/proxy-load-balancer>: Include informazioni aggiuntive su:</span><span class="sxs-lookup"><span data-stu-id="85b29-167"><xref:host-and-deploy/proxy-load-balancer>: Includes guidance on:</span></span>
  * <span data-ttu-id="85b29-168">Uso del middleware intestazioni con inoltro per mantenere le informazioni sullo schema HTTPS tra i server proxy e le reti interne.</span><span class="sxs-lookup"><span data-stu-id="85b29-168">Using Forwarded Headers Middleware to preserve HTTPS scheme information across proxy servers and internal networks.</span></span>
  * <span data-ttu-id="85b29-169">Scenari e casi d'uso aggiuntivi, tra cui la configurazione manuale dello schema, la modifica del percorso della richiesta per il routing delle richieste corretto e l'inoltro dello schema di richiesta per i proxy inversi Linux e non IIS.</span><span class="sxs-lookup"><span data-stu-id="85b29-169">Additional scenarios and use cases, including manual scheme configuration, request path changes for correct request routing, and forwarding the request scheme for Linux and non-IIS reverse proxies.</span></span>
