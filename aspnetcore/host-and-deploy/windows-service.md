---
title: Ospitare ASP.NET Core in un servizio Windows
author: rick-anderson
description: Informazioni su come ospitare un'app ASP.NET Core in un servizio Windows.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: host-and-deploy/windows-service
ms.openlocfilehash: e5c7dd0e52f1246d3ac6ad9622573db4c276654b
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588815"
---
# <a name="host-aspnet-core-in-a-windows-service"></a><span data-ttu-id="588df-103">Ospitare ASP.NET Core in un servizio Windows</span><span class="sxs-lookup"><span data-stu-id="588df-103">Host ASP.NET Core in a Windows Service</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="588df-104">È possibile ospitare un'app ASP.NET Core in Windows come [servizio Windows](/dotnet/framework/windows-services/introduction-to-windows-service-applications) senza usare IIS.</span><span class="sxs-lookup"><span data-stu-id="588df-104">An ASP.NET Core app can be hosted on Windows as a [Windows Service](/dotnet/framework/windows-services/introduction-to-windows-service-applications) without using IIS.</span></span> <span data-ttu-id="588df-105">Quando è ospitata come servizio di Windows, l'app viene avviata automaticamente dopo il riavvio del server.</span><span class="sxs-lookup"><span data-stu-id="588df-105">When hosted as a Windows Service, the app automatically starts after server reboots.</span></span>

<span data-ttu-id="588df-106">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="588df-106">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="588df-107">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="588df-107">Prerequisites</span></span>

* [<span data-ttu-id="588df-108">ASP.NET Core SDK 2.1 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="588df-108">ASP.NET Core SDK 2.1 or later</span></span>](https://dotnet.microsoft.com/download)
* [<span data-ttu-id="588df-109">PowerShell 6,2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="588df-109">PowerShell 6.2 or later</span></span>](https://github.com/PowerShell/PowerShell)

## <a name="worker-service-template"></a><span data-ttu-id="588df-110">Modello di servizio di ruolo di lavoro</span><span class="sxs-lookup"><span data-stu-id="588df-110">Worker Service template</span></span>

<span data-ttu-id="588df-111">Il modello di servizio di ruolo di lavoro di ASP.NET Core rappresenta un punto di partenza per la scrittura di app di servizi a esecuzione prolungata.</span><span class="sxs-lookup"><span data-stu-id="588df-111">The ASP.NET Core Worker Service template provides a starting point for writing long running service apps.</span></span> <span data-ttu-id="588df-112">Per usare il modello come base per un'app di servizio Windows:</span><span class="sxs-lookup"><span data-stu-id="588df-112">To use the template as a basis for a Windows Service app:</span></span>

1. <span data-ttu-id="588df-113">Creare un'app di servizio di ruolo di lavoro dal modello .NET Core.</span><span class="sxs-lookup"><span data-stu-id="588df-113">Create a Worker Service app from the .NET Core template.</span></span>
1. <span data-ttu-id="588df-114">Seguire le indicazioni nella sezione [Configurazione dell'app](#app-configuration) per aggiornare l'app di servizio di ruolo di lavoro affinché venga eseguita come servizio Windows.</span><span class="sxs-lookup"><span data-stu-id="588df-114">Follow the guidance in the [App configuration](#app-configuration) section to update the Worker Service app so that it can run as a Windows Service.</span></span>

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="app-configuration"></a><span data-ttu-id="588df-115">Configurazione delle app</span><span class="sxs-lookup"><span data-stu-id="588df-115">App configuration</span></span>

<span data-ttu-id="588df-116">L'app richiede un riferimento al pacchetto per [Microsoft. Extensions. Hosting. WindowsServices](https://www.nuget.org/packages/Microsoft.Extensions.Hosting.WindowsServices).</span><span class="sxs-lookup"><span data-stu-id="588df-116">The app requires a package reference for [Microsoft.Extensions.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.Extensions.Hosting.WindowsServices).</span></span>

<span data-ttu-id="588df-117">`IHostBuilder.UseWindowsService` viene chiamato durante la compilazione dell'host.</span><span class="sxs-lookup"><span data-stu-id="588df-117">`IHostBuilder.UseWindowsService` is called when building the host.</span></span> <span data-ttu-id="588df-118">Se l'app è in esecuzione come servizio di Windows, il metodo:</span><span class="sxs-lookup"><span data-stu-id="588df-118">If the app is running as a Windows Service, the method:</span></span>

* <span data-ttu-id="588df-119">Imposta la durata dell'host su `WindowsServiceLifetime`.</span><span class="sxs-lookup"><span data-stu-id="588df-119">Sets the host lifetime to `WindowsServiceLifetime`.</span></span>
* <span data-ttu-id="588df-120">Imposta la [radice del contenuto](xref:fundamentals/index#content-root) su [AppContext. BaseDirectory](xref:System.AppContext.BaseDirectory).</span><span class="sxs-lookup"><span data-stu-id="588df-120">Sets the [content root](xref:fundamentals/index#content-root) to [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory).</span></span> <span data-ttu-id="588df-121">Per altre informazioni, vedere la sezione [Directory corrente e radice del contenuto](#current-directory-and-content-root).</span><span class="sxs-lookup"><span data-stu-id="588df-121">For more information, see the [Current directory and content root](#current-directory-and-content-root) section.</span></span>
* <span data-ttu-id="588df-122">Abilita la registrazione nel registro eventi:</span><span class="sxs-lookup"><span data-stu-id="588df-122">Enables logging to the event log:</span></span>
  * <span data-ttu-id="588df-123">Il nome dell'applicazione viene usato come nome di origine predefinito.</span><span class="sxs-lookup"><span data-stu-id="588df-123">The application name is used as the default source name.</span></span>
  * <span data-ttu-id="588df-124">Il livello di registrazione predefinito è *avviso* o superiore per un'app basata su un modello di ASP.NET Core che chiama `CreateDefaultBuilder` per compilare l'host.</span><span class="sxs-lookup"><span data-stu-id="588df-124">The default log level is *Warning* or higher for an app based on an ASP.NET Core template that calls `CreateDefaultBuilder` to build the host.</span></span>
  * <span data-ttu-id="588df-125">Eseguire l'override del livello di registrazione predefinito con la `Logging:EventLog:LogLevel:Default` chiave in *appsettings.json* / *appSettings. { Environment}. JSON* o un altro provider di configurazione.</span><span class="sxs-lookup"><span data-stu-id="588df-125">Override the default log level with the `Logging:EventLog:LogLevel:Default` key in *appsettings.json*/*appsettings.{Environment}.json* or other configuration provider.</span></span>
  * <span data-ttu-id="588df-126">Solo gli amministratori possono creare nuove origini eventi.</span><span class="sxs-lookup"><span data-stu-id="588df-126">Only administrators can create new event sources.</span></span> <span data-ttu-id="588df-127">Quando non è possibile creare un'origine evento usando il nome dell'applicazione, viene registrato un avviso nell'origine *Applicazione* e i log eventi vengono disabilitati.</span><span class="sxs-lookup"><span data-stu-id="588df-127">When an event source can't be created using the application name, a warning is logged to the *Application* source and event logs are disabled.</span></span>

<span data-ttu-id="588df-128">In `CreateHostBuilder` di *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="588df-128">In `CreateHostBuilder` of *Program.cs*:</span></span>

```csharp
Host.CreateDefaultBuilder(args)
    .UseWindowsService()
    ...
```

<span data-ttu-id="588df-129">Questo argomento è accompagnato dalle app di esempio seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-129">The following sample apps accompany this topic:</span></span>

* <span data-ttu-id="588df-130">Esempio di servizio di lavoro in background: un esempio di app non Web basato sul [modello di servizio](#worker-service-template) del ruolo di lavoro che usa i [servizi ospitati](xref:fundamentals/host/hosted-services) per le attività in background.</span><span class="sxs-lookup"><span data-stu-id="588df-130">Background Worker Service Sample: A non-web app sample based on the [Worker Service template](#worker-service-template) that uses [hosted services](xref:fundamentals/host/hosted-services) for background tasks.</span></span>
* <span data-ttu-id="588df-131">Esempio di servizio app Web: un Razor esempio di app Web di pagine che viene eseguito come servizio Windows con [servizi ospitati](xref:fundamentals/host/hosted-services) per le attività in background.</span><span class="sxs-lookup"><span data-stu-id="588df-131">Web App Service Sample: A Razor Pages web app sample that runs as a Windows Service with [hosted services](xref:fundamentals/host/hosted-services) for background tasks.</span></span>

<span data-ttu-id="588df-132">Per informazioni aggiuntive su MVC, vedere gli articoli in <xref:mvc/overview> e <xref:migration/22-to-30> .</span><span class="sxs-lookup"><span data-stu-id="588df-132">For MVC guidance, see the articles under <xref:mvc/overview> and <xref:migration/22-to-30>.</span></span>

## <a name="deployment-type"></a><span data-ttu-id="588df-133">Tipo di distribuzione</span><span class="sxs-lookup"><span data-stu-id="588df-133">Deployment type</span></span>

<span data-ttu-id="588df-134">Per informazioni e consigli sugli scenari di distribuzione, vedere [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/).</span><span class="sxs-lookup"><span data-stu-id="588df-134">For information and advice on deployment scenarios, see [.NET Core application deployment](/dotnet/core/deploying/).</span></span>

### <a name="sdk"></a><span data-ttu-id="588df-135">SDK</span><span class="sxs-lookup"><span data-stu-id="588df-135">SDK</span></span>

<span data-ttu-id="588df-136">Per un servizio basato su app Web che usa le Razor pagine o i framework MVC, specificare Web SDK nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="588df-136">For a web app-based service that uses the Razor Pages or MVC frameworks, specify the Web SDK in the project file:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

<span data-ttu-id="588df-137">Se il servizio esegue solo attività in background, ad esempio [servizi ospitati](xref:fundamentals/host/hosted-services), specificare l'SDK del ruolo di lavoro nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="588df-137">If the service only executes background tasks (for example, [hosted services](xref:fundamentals/host/hosted-services)), specify the Worker SDK in the project file:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a><span data-ttu-id="588df-138">Distribuzione dipendente dal framework</span><span class="sxs-lookup"><span data-stu-id="588df-138">Framework-dependent deployment (FDD)</span></span>

<span data-ttu-id="588df-139">La distribuzione dipendente dal framework si basa sulla presenza di una versione condivisa a livello di sistema di .NET Core nel sistema di destinazione.</span><span class="sxs-lookup"><span data-stu-id="588df-139">Framework-dependent deployment (FDD) relies on the presence of a shared system-wide version of .NET Core on the target system.</span></span> <span data-ttu-id="588df-140">Quando lo scenario di distribuzione dipendente dal framework viene implementato in base alle indicazioni di questo articolo, l'SDK genera un file eseguibile (con estensione *exe*) detto *eseguibile dipendente dal framework*.</span><span class="sxs-lookup"><span data-stu-id="588df-140">When the FDD scenario is adopted following the guidance in this article, the SDK produces an executable (*.exe*), called a *framework-dependent executable*.</span></span>

<span data-ttu-id="588df-141">Se si usa l' [SDK Web](#sdk), un file di *web.config* , che in genere viene prodotto durante la pubblicazione di un'app ASP.NET Core, non è necessario per un'app dei servizi Windows.</span><span class="sxs-lookup"><span data-stu-id="588df-141">If using the [Web SDK](#sdk), a *web.config* file, which is normally produced when publishing an ASP.NET Core app, is unnecessary for a Windows Services app.</span></span> <span data-ttu-id="588df-142">Per disabilitare la creazione del file *web.config*, aggiungere la proprietà `<IsTransformWebConfigDisabled>` impostata su `true`.</span><span class="sxs-lookup"><span data-stu-id="588df-142">To disable the creation of the *web.config* file, add the `<IsTransformWebConfigDisabled>` property set to `true`.</span></span>

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp3.0</TargetFramework>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a><span data-ttu-id="588df-143">Distribuzione autonoma</span><span class="sxs-lookup"><span data-stu-id="588df-143">Self-contained deployment (SCD)</span></span>

<span data-ttu-id="588df-144">Una distribuzione autonoma non si basa sulla presenza di un framework condiviso nel sistema host.</span><span class="sxs-lookup"><span data-stu-id="588df-144">Self-contained deployment (SCD) doesn't rely on the presence of a shared framework on the host system.</span></span> <span data-ttu-id="588df-145">Il runtime e le dipendenze dell'app vengono distribuiti con l'app.</span><span class="sxs-lookup"><span data-stu-id="588df-145">The runtime and the app's dependencies are deployed with the app.</span></span>

<span data-ttu-id="588df-146">Un [identificatore di runtime (RID)](/dotnet/core/rid-catalog) di Windows viene incluso nel `<PropertyGroup>` che contiene il framework di destinazione:</span><span class="sxs-lookup"><span data-stu-id="588df-146">A Windows [Runtime Identifier (RID)](/dotnet/core/rid-catalog) is included in the `<PropertyGroup>` that contains the target framework:</span></span>

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

<span data-ttu-id="588df-147">Per eseguire la pubblicazione per più identificatori di runtime:</span><span class="sxs-lookup"><span data-stu-id="588df-147">To publish for multiple RIDs:</span></span>

* <span data-ttu-id="588df-148">Specificare gli identificatori di runtime in un elenco delimitato da punto e virgola.</span><span class="sxs-lookup"><span data-stu-id="588df-148">Provide the RIDs in a semicolon-delimited list.</span></span>
* <span data-ttu-id="588df-149">Usare il nome della proprietà [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plurale).</span><span class="sxs-lookup"><span data-stu-id="588df-149">Use the property name [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plural).</span></span>

<span data-ttu-id="588df-150">Per altre informazioni, vedere il [Catalogo RID di .NET Core](/dotnet/core/rid-catalog).</span><span class="sxs-lookup"><span data-stu-id="588df-150">For more information, see [.NET Core RID Catalog](/dotnet/core/rid-catalog).</span></span>

## <a name="service-user-account"></a><span data-ttu-id="588df-151">Account utente del servizio</span><span class="sxs-lookup"><span data-stu-id="588df-151">Service user account</span></span>

<span data-ttu-id="588df-152">Per creare un account utente per un servizio, usare il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) da una shell dei comandi di PowerShell 6 amministrativa.</span><span class="sxs-lookup"><span data-stu-id="588df-152">To create a user account for a service, use the [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet from an administrative PowerShell 6 command shell.</span></span>

<span data-ttu-id="588df-153">In Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763) o versioni successive:</span><span class="sxs-lookup"><span data-stu-id="588df-153">On Windows 10 October 2018 Update (version 1809/build 10.0.17763) or later:</span></span>

```powershell
New-LocalUser -Name {SERVICE NAME}
```

<span data-ttu-id="588df-154">In un sistema operativo Windows precedente ad Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763):</span><span class="sxs-lookup"><span data-stu-id="588df-154">On Windows OS earlier than the Windows 10 October 2018 Update (version 1809/build 10.0.17763):</span></span>

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

<span data-ttu-id="588df-155">Fornire una [password complessa](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) quando richiesto.</span><span class="sxs-lookup"><span data-stu-id="588df-155">Provide a [strong password](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) when prompted.</span></span>

<span data-ttu-id="588df-156">Se non viene specificato il parametro `-AccountExpires` per il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) con un valore <xref:System.DateTime> di scadenza, l'account non scade.</span><span class="sxs-lookup"><span data-stu-id="588df-156">Unless the `-AccountExpires` parameter is supplied to the [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet with an expiration <xref:System.DateTime>, the account doesn't expire.</span></span>

<span data-ttu-id="588df-157">Per altre informazioni, vedere [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) e [Service User Accounts](/windows/desktop/services/service-user-accounts) (Account utente di servizio).</span><span class="sxs-lookup"><span data-stu-id="588df-157">For more information, see [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) and [Service User Accounts](/windows/desktop/services/service-user-accounts).</span></span>

<span data-ttu-id="588df-158">Un approccio alternativo per la gestione degli utenti quando si usa Active Directory consiste nell'usare account del servizio gestito.</span><span class="sxs-lookup"><span data-stu-id="588df-158">An alternative approach to managing users when using Active Directory is to use Managed Service Accounts.</span></span> <span data-ttu-id="588df-159">Per altre informazioni, vedere [Panoramica degli account del servizio gestito del gruppo](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).</span><span class="sxs-lookup"><span data-stu-id="588df-159">For more information, see [Group Managed Service Accounts Overview](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).</span></span>

## <a name="log-on-as-a-service-rights"></a><span data-ttu-id="588df-160">Diritti Accesso come servizio</span><span class="sxs-lookup"><span data-stu-id="588df-160">Log on as a service rights</span></span>

<span data-ttu-id="588df-161">Per stabilire i diritti *Accesso come servizio* per un account utente di servizio:</span><span class="sxs-lookup"><span data-stu-id="588df-161">To establish *Log on as a service* rights for a service user account:</span></span>

1. <span data-ttu-id="588df-162">Aprire l'editor Criteri di sicurezza locali eseguendo *secpol.msc*.</span><span class="sxs-lookup"><span data-stu-id="588df-162">Open the Local Security Policy editor by running *secpol.msc*.</span></span>
1. <span data-ttu-id="588df-163">Espandere il nodo **Criteri locali** e selezionare **Assegnazione diritti utente**.</span><span class="sxs-lookup"><span data-stu-id="588df-163">Expand the **Local Policies** node and select **User Rights Assignment**.</span></span>
1. <span data-ttu-id="588df-164">Aprire il criterio **Accesso come servizio**.</span><span class="sxs-lookup"><span data-stu-id="588df-164">Open the **Log on as a service** policy.</span></span>
1. <span data-ttu-id="588df-165">Selezionare **Aggiungi utente o gruppo**.</span><span class="sxs-lookup"><span data-stu-id="588df-165">Select **Add User or Group**.</span></span>
1. <span data-ttu-id="588df-166">Specificare il nome oggetto (account utente) in uno dei modi seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-166">Provide the object name (user account) using either of the following approaches:</span></span>
   1. <span data-ttu-id="588df-167">Digitare l'account utente (`{DOMAIN OR COMPUTER NAME\USER}`) nel campo del nome oggetto e scegliere **OK** per aggiungere l'utente al criterio.</span><span class="sxs-lookup"><span data-stu-id="588df-167">Type the user account (`{DOMAIN OR COMPUTER NAME\USER}`) in the object name field and select **OK** to add the user to the policy.</span></span>
   1. <span data-ttu-id="588df-168">Fare clic su **Advanced** (Avanzate).</span><span class="sxs-lookup"><span data-stu-id="588df-168">Select **Advanced**.</span></span> <span data-ttu-id="588df-169">Selezionare **Trova**.</span><span class="sxs-lookup"><span data-stu-id="588df-169">Select **Find Now**.</span></span> <span data-ttu-id="588df-170">Selezionare l'account utente dall'elenco.</span><span class="sxs-lookup"><span data-stu-id="588df-170">Select the user account from the list.</span></span> <span data-ttu-id="588df-171">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="588df-171">Select **OK**.</span></span> <span data-ttu-id="588df-172">Scegliere di nuovo **OK** per aggiungere l'utente al criterio.</span><span class="sxs-lookup"><span data-stu-id="588df-172">Select **OK** again to add the user to the policy.</span></span>
1. <span data-ttu-id="588df-173">Scegliere **OK** o **Applica** per accettare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="588df-173">Select **OK** or **Apply** to accept the changes.</span></span>

## <a name="create-and-manage-the-windows-service"></a><span data-ttu-id="588df-174">Creare e gestire il servizio di Windows</span><span class="sxs-lookup"><span data-stu-id="588df-174">Create and manage the Windows Service</span></span>

### <a name="create-a-service"></a><span data-ttu-id="588df-175">Creare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-175">Create a service</span></span>

<span data-ttu-id="588df-176">Usare i comandi di PowerShell per registrare un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-176">Use PowerShell commands to register a service.</span></span> <span data-ttu-id="588df-177">Da una shell dei comandi di PowerShell 6 amministrativa eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-177">From an administrative PowerShell 6 command shell, execute the following commands:</span></span>

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = "{DOMAIN OR COMPUTER NAME\USER}", "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName "{EXE FILE PATH}" -Credential "{DOMAIN OR COMPUTER NAME\USER}" -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* <span data-ttu-id="588df-178">`{EXE PATH}`: Percorso della cartella dell'app nell'host (ad esempio, `d:\myservice` ).</span><span class="sxs-lookup"><span data-stu-id="588df-178">`{EXE PATH}`: Path to the app's folder on the host (for example, `d:\myservice`).</span></span> <span data-ttu-id="588df-179">Non includere l'eseguibile dell'app nel percorso.</span><span class="sxs-lookup"><span data-stu-id="588df-179">Don't include the app's executable in the path.</span></span> <span data-ttu-id="588df-180">Non è necessario aggiungere una barra finale.</span><span class="sxs-lookup"><span data-stu-id="588df-180">A trailing slash isn't required.</span></span>
* <span data-ttu-id="588df-181">`{DOMAIN OR COMPUTER NAME\USER}`: Account utente del servizio (ad esempio, `Contoso\ServiceUser` ).</span><span class="sxs-lookup"><span data-stu-id="588df-181">`{DOMAIN OR COMPUTER NAME\USER}`: Service user account (for example, `Contoso\ServiceUser`).</span></span>
* <span data-ttu-id="588df-182">`{SERVICE NAME}`: Nome del servizio (ad esempio, `MyService` ).</span><span class="sxs-lookup"><span data-stu-id="588df-182">`{SERVICE NAME}`: Service name (for example, `MyService`).</span></span>
* <span data-ttu-id="588df-183">`{EXE FILE PATH}`: Percorso eseguibile dell'app (ad esempio, `d:\myservice\myservice.exe` ).</span><span class="sxs-lookup"><span data-stu-id="588df-183">`{EXE FILE PATH}`: The app's executable path (for example, `d:\myservice\myservice.exe`).</span></span> <span data-ttu-id="588df-184">Includere il nome del file eseguibile con l'estensione.</span><span class="sxs-lookup"><span data-stu-id="588df-184">Include the executable's file name with extension.</span></span>
* <span data-ttu-id="588df-185">`{DESCRIPTION}`: Descrizione del servizio (ad esempio, `My sample service` ).</span><span class="sxs-lookup"><span data-stu-id="588df-185">`{DESCRIPTION}`: Service description (for example, `My sample service`).</span></span>
* <span data-ttu-id="588df-186">`{DISPLAY NAME}`: Nome visualizzato del servizio (ad esempio, `My Service` ).</span><span class="sxs-lookup"><span data-stu-id="588df-186">`{DISPLAY NAME}`: Service display name (for example, `My Service`).</span></span>

### <a name="start-a-service"></a><span data-ttu-id="588df-187">Avviare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-187">Start a service</span></span>

<span data-ttu-id="588df-188">Per avviare un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-188">Start a service with the following PowerShell 6 command:</span></span>

```powershell
Start-Service -Name {SERVICE NAME}
```

<span data-ttu-id="588df-189">L'avvio del servizio richiede alcuni secondi.</span><span class="sxs-lookup"><span data-stu-id="588df-189">The command takes a few seconds to start the service.</span></span>

### <a name="determine-a-services-status"></a><span data-ttu-id="588df-190">Determinare lo stato di un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-190">Determine a service's status</span></span>

<span data-ttu-id="588df-191">Per verificare lo stato di un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-191">To check the status of a service, use the following PowerShell 6 command:</span></span>

```powershell
Get-Service -Name {SERVICE NAME}
```

<span data-ttu-id="588df-192">Lo stato viene indicato con uno dei valori seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-192">The status is reported as one of the following values:</span></span>

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a><span data-ttu-id="588df-193">Arrestare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-193">Stop a service</span></span>

<span data-ttu-id="588df-194">Per arrestare un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-194">Stop a service with the following Powershell 6 command:</span></span>

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a><span data-ttu-id="588df-195">Rimuovere un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-195">Remove a service</span></span>

<span data-ttu-id="588df-196">Dopo il breve lasso di tempo necessario per arrestare un servizio, rimuovere il servizio con il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-196">After a short delay to stop a service, remove a service with the following Powershell 6 command:</span></span>

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="proxy-server-and-load-balancer-scenarios"></a><span data-ttu-id="588df-197">Scenari con server proxy e servizi di bilanciamento del carico</span><span class="sxs-lookup"><span data-stu-id="588df-197">Proxy server and load balancer scenarios</span></span>

<span data-ttu-id="588df-198">I servizi che interagiscono con le richieste da Internet o da una rete aziendale e si trovano dietro un proxy o un servizio di bilanciamento del carico potrebbero richiedere una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="588df-198">Services that interact with requests from the Internet or a corporate network and are behind a proxy or load balancer might require additional configuration.</span></span> <span data-ttu-id="588df-199">Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.</span><span class="sxs-lookup"><span data-stu-id="588df-199">For more information, see <xref:host-and-deploy/proxy-load-balancer>.</span></span>

## <a name="configure-endpoints"></a><span data-ttu-id="588df-200">Configurare gli endpoint</span><span class="sxs-lookup"><span data-stu-id="588df-200">Configure endpoints</span></span>

<span data-ttu-id="588df-201">Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`.</span><span class="sxs-lookup"><span data-stu-id="588df-201">By default, ASP.NET Core binds to `http://localhost:5000`.</span></span> <span data-ttu-id="588df-202">Configurare l'URL e la porta impostando la `ASPNETCORE_URLS` variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="588df-202">Configure the URL and port by setting the `ASPNETCORE_URLS` environment variable.</span></span>

<span data-ttu-id="588df-203">Per ulteriori approcci alla configurazione di porte e URL, vedere l'articolo relativo al server pertinente:</span><span class="sxs-lookup"><span data-stu-id="588df-203">For additional URL and port configuration approaches, see the relevant server article:</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* <xref:fundamentals/servers/kestrel/endpoints>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

* <xref:fundamentals/servers/kestrel#endpoint-configuration>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

* <xref:fundamentals/servers/httpsys#configure-windows-server>

<span data-ttu-id="588df-204">Le linee guida precedenti riguardano il supporto per gli endpoint HTTPS.</span><span class="sxs-lookup"><span data-stu-id="588df-204">The preceding guidance covers support for HTTPS endpoints.</span></span> <span data-ttu-id="588df-205">Ad esempio, configurare l'app per HTTPS quando si usa l'autenticazione con un servizio Windows.</span><span class="sxs-lookup"><span data-stu-id="588df-205">For example, configure the app for HTTPS when authentication is used with a Windows Service.</span></span>

> [!NOTE]
> <span data-ttu-id="588df-206">L'uso del certificato di sviluppo ASP.NET Core HTTPS per proteggere un endpoint del servizio non è supportato.</span><span class="sxs-lookup"><span data-stu-id="588df-206">Use of the ASP.NET Core HTTPS development certificate to secure a service endpoint isn't supported.</span></span>

## <a name="current-directory-and-content-root"></a><span data-ttu-id="588df-207">Directory corrente e radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="588df-207">Current directory and content root</span></span>

<span data-ttu-id="588df-208">La directory di lavoro corrente restituita chiamando <xref:System.IO.Directory.GetCurrentDirectory%2A> per un servizio Windows è la cartella *C:\\WINDOWS\\system32*.</span><span class="sxs-lookup"><span data-stu-id="588df-208">The current working directory returned by calling <xref:System.IO.Directory.GetCurrentDirectory%2A> for a Windows Service is the *C:\\WINDOWS\\system32* folder.</span></span> <span data-ttu-id="588df-209">La cartella *system32* non è un percorso appropriato per archiviare i file di un servizio, ad esempio i file di impostazioni.</span><span class="sxs-lookup"><span data-stu-id="588df-209">The *system32* folder isn't a suitable location to store a service's files (for example, settings files).</span></span> <span data-ttu-id="588df-210">Usare uno degli approcci seguenti per gestire e accedere agli asset e ai file di impostazioni di un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-210">Use one of the following approaches to maintain and access a service's assets and settings files.</span></span>

### <a name="use-contentrootpath-or-contentrootfileprovider"></a><span data-ttu-id="588df-211">Usare ContentRootPath o ContentRootFileProvider</span><span class="sxs-lookup"><span data-stu-id="588df-211">Use ContentRootPath or ContentRootFileProvider</span></span>

<span data-ttu-id="588df-212">Usare [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath) o <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootFileProvider> per individuare le risorse di un'app.</span><span class="sxs-lookup"><span data-stu-id="588df-212">Use [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath) or <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootFileProvider> to locate an app's resources.</span></span>

<span data-ttu-id="588df-213">Quando l'app viene eseguita come servizio, <xref:Microsoft.Extensions.Hosting.WindowsServiceLifetimeHostBuilderExtensions.UseWindowsService%2A> imposta <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath> su [AppContext. BaseDirectory](xref:System.AppContext.BaseDirectory).</span><span class="sxs-lookup"><span data-stu-id="588df-213">When the app runs as a service, <xref:Microsoft.Extensions.Hosting.WindowsServiceLifetimeHostBuilderExtensions.UseWindowsService%2A> sets the <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath> to [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory).</span></span>

<span data-ttu-id="588df-214">I file di impostazioni predefinite dell'app *appsettings.json* e *appSettings. { Environment}. JSON*, viene caricato dalla radice del contenuto dell'app chiamando [CreateDefaultBuilder durante la costruzione dell'host](xref:fundamentals/host/generic-host#set-up-a-host).</span><span class="sxs-lookup"><span data-stu-id="588df-214">The app's default settings files, *appsettings.json* and *appsettings.{Environment}.json*, are loaded from the app's content root by calling [CreateDefaultBuilder during host construction](xref:fundamentals/host/generic-host#set-up-a-host).</span></span>

<span data-ttu-id="588df-215">Per gli altri file di impostazioni caricati dal codice Developer in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A> , non è necessario chiamare <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A> .</span><span class="sxs-lookup"><span data-stu-id="588df-215">For other settings files loaded by developer code in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A>, there's no need to call <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A>.</span></span> <span data-ttu-id="588df-216">Nell'esempio seguente, il *custom_settings.js* nel file è presente nella radice del contenuto dell'app e viene caricato senza impostare esplicitamente un percorso di base:</span><span class="sxs-lookup"><span data-stu-id="588df-216">In the following example, the *custom_settings.json* file exists in the app's content root and is loaded without explicitly setting a base path:</span></span>

[!code-csharp[](windows-service/samples_snapshot/CustomSettingsExample.cs?highlight=13)]

<span data-ttu-id="588df-217">Non tentare di usare <xref:System.IO.Directory.GetCurrentDirectory%2A> per ottenere un percorso di risorsa perché un'app di servizio Windows restituisce la cartella *C: \\ Windows \\ system32* come directory corrente.</span><span class="sxs-lookup"><span data-stu-id="588df-217">Don't attempt to use <xref:System.IO.Directory.GetCurrentDirectory%2A> to obtain a resource path because a Windows Service app returns the *C:\\WINDOWS\\system32* folder as its current directory.</span></span>

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a><span data-ttu-id="588df-218">Archiviare i file di un servizio in un percorso appropriato nel disco</span><span class="sxs-lookup"><span data-stu-id="588df-218">Store a service's files in a suitable location on disk</span></span>

<span data-ttu-id="588df-219">Specificare un percorso assoluto con <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A> quando si usa un <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per la cartella contenente i file.</span><span class="sxs-lookup"><span data-stu-id="588df-219">Specify an absolute path with <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A> when using an <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> to the folder containing the files.</span></span>

## <a name="troubleshoot"></a><span data-ttu-id="588df-220">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="588df-220">Troubleshoot</span></span>

<span data-ttu-id="588df-221">Per risolvere i problemi relativi a un'app di servizio Windows, vedere <xref:test/troubleshoot> .</span><span class="sxs-lookup"><span data-stu-id="588df-221">To troubleshoot a Windows Service app, see <xref:test/troubleshoot>.</span></span>

### <a name="common-errors"></a><span data-ttu-id="588df-222">Errori comuni</span><span class="sxs-lookup"><span data-stu-id="588df-222">Common errors</span></span>

* <span data-ttu-id="588df-223">È in uso una versione precedente o provvisoria di PowerShell.</span><span class="sxs-lookup"><span data-stu-id="588df-223">An old or pre-release version of PowerShell is in use.</span></span>
* <span data-ttu-id="588df-224">Il servizio registrato non usa l'output **pubblicato** dell'app dal comando [DotNet Publish](/dotnet/core/tools/dotnet-publish) .</span><span class="sxs-lookup"><span data-stu-id="588df-224">The registered service doesn't use the app's **published** output from the [dotnet publish](/dotnet/core/tools/dotnet-publish) command.</span></span> <span data-ttu-id="588df-225">L'output del comando [DotNet Build](/dotnet/core/tools/dotnet-build) non è supportato per la distribuzione di app.</span><span class="sxs-lookup"><span data-stu-id="588df-225">Output of the [dotnet build](/dotnet/core/tools/dotnet-build) command isn't supported for app deployment.</span></span> <span data-ttu-id="588df-226">Gli asset pubblicati si trovano in una delle cartelle seguenti, a seconda del tipo di distribuzione:</span><span class="sxs-lookup"><span data-stu-id="588df-226">Published assets are found in either of the following folders depending on the deployment type:</span></span>
  * <span data-ttu-id="588df-227">*bin/Release/{Target Framework}/Publish* (FDD)</span><span class="sxs-lookup"><span data-stu-id="588df-227">*bin/Release/{TARGET FRAMEWORK}/publish* (FDD)</span></span>
  * <span data-ttu-id="588df-228">*bin/Release/{Target Framework}/{Runtime Identifier}/Publish* (SCD)</span><span class="sxs-lookup"><span data-stu-id="588df-228">*bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)</span></span>
* <span data-ttu-id="588df-229">Il servizio non è nello stato in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="588df-229">The service isn't in the RUNNING state.</span></span>
* <span data-ttu-id="588df-230">I percorsi delle risorse utilizzate dall'app, ad esempio i certificati, non sono corretti.</span><span class="sxs-lookup"><span data-stu-id="588df-230">The paths to resources that the app uses (for example, certificates) are incorrect.</span></span> <span data-ttu-id="588df-231">Il percorso di base di un servizio Windows è *c: \\ Windows \\ system32*.</span><span class="sxs-lookup"><span data-stu-id="588df-231">The base path of a Windows Service is *c:\\Windows\\System32*.</span></span>
* <span data-ttu-id="588df-232">L'utente non dispone *di diritti di accesso come servizio* .</span><span class="sxs-lookup"><span data-stu-id="588df-232">The user doesn't have *Log on as a service* rights.</span></span>
* <span data-ttu-id="588df-233">La password dell'utente è scaduta o non è stata passata correttamente durante l'esecuzione del `New-Service` comando di PowerShell.</span><span class="sxs-lookup"><span data-stu-id="588df-233">The user's password is expired or incorrectly passed when executing the `New-Service` PowerShell command.</span></span>
* <span data-ttu-id="588df-234">L'app richiede l'autenticazione ASP.NET Core ma non è configurata per le connessioni protette (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="588df-234">The app requires ASP.NET Core authentication but isn't configured for secure connections (HTTPS).</span></span>
* <span data-ttu-id="588df-235">La porta dell'URL della richiesta non è corretta o non è configurata correttamente nell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-235">The request URL port is incorrect or not configured correctly in the app.</span></span>

### <a name="system-and-application-event-logs"></a><span data-ttu-id="588df-236">Log eventi di sistema e dell'applicazione</span><span class="sxs-lookup"><span data-stu-id="588df-236">System and Application Event Logs</span></span>

<span data-ttu-id="588df-237">Accedere ai registri eventi di sistema e dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="588df-237">Access the System and Application Event Logs:</span></span>

1. <span data-ttu-id="588df-238">Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .</span><span class="sxs-lookup"><span data-stu-id="588df-238">Open the Start menu, search for *Event Viewer*, and select the **Event Viewer** app.</span></span>
1. <span data-ttu-id="588df-239">In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.</span><span class="sxs-lookup"><span data-stu-id="588df-239">In **Event Viewer**, open the **Windows Logs** node.</span></span>
1. <span data-ttu-id="588df-240">Selezionare **sistema** per aprire il registro eventi di sistema.</span><span class="sxs-lookup"><span data-stu-id="588df-240">Select **System** to open the System Event Log.</span></span> <span data-ttu-id="588df-241">Selezionare **Applicazione** per aprire il log eventi dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="588df-241">Select **Application** to open the Application Event Log.</span></span>
1. <span data-ttu-id="588df-242">Cercare gli errori associati all'app in cui si è verificato il problema.</span><span class="sxs-lookup"><span data-stu-id="588df-242">Search for errors associated with the failing app.</span></span>

### <a name="run-the-app-at-a-command-prompt"></a><span data-ttu-id="588df-243">Eseguire l'app da un prompt dei comandi</span><span class="sxs-lookup"><span data-stu-id="588df-243">Run the app at a command prompt</span></span>

<span data-ttu-id="588df-244">Molti errori di avvio non producono informazioni utili nei log eventi.</span><span class="sxs-lookup"><span data-stu-id="588df-244">Many startup errors don't produce useful information in the event logs.</span></span> <span data-ttu-id="588df-245">È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host.</span><span class="sxs-lookup"><span data-stu-id="588df-245">You can find the cause of some errors by running the app at a command prompt on the hosting system.</span></span> <span data-ttu-id="588df-246">Per registrare dettagli aggiuntivi dall'app, abbassare il [livello di registrazione](xref:fundamentals/logging/index#log-level) o eseguire l'app nell' [ambiente di sviluppo](xref:fundamentals/environments).</span><span class="sxs-lookup"><span data-stu-id="588df-246">To log additional detail from the app, lower the [log level](xref:fundamentals/logging/index#log-level) or run the app in the [Development environment](xref:fundamentals/environments).</span></span>

### <a name="clear-package-caches"></a><span data-ttu-id="588df-247">Cancella cache di pacchetti</span><span class="sxs-lookup"><span data-stu-id="588df-247">Clear package caches</span></span>

<span data-ttu-id="588df-248">Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-248">A functioning app may fail immediately after upgrading either the .NET Core SDK on the development machine or changing package versions within the app.</span></span> <span data-ttu-id="588df-249">In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali.</span><span class="sxs-lookup"><span data-stu-id="588df-249">In some cases, incoherent packages may break an app when performing major upgrades.</span></span> <span data-ttu-id="588df-250">La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-250">Most of these issues can be fixed by following these instructions:</span></span>

1. <span data-ttu-id="588df-251">Eliminare le cartelle *bin* e *obj*.</span><span class="sxs-lookup"><span data-stu-id="588df-251">Delete the *bin* and *obj* folders.</span></span>
1. <span data-ttu-id="588df-252">Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="588df-252">Clear the package caches by executing [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) from a command shell.</span></span>

   <span data-ttu-id="588df-253">La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` .</span><span class="sxs-lookup"><span data-stu-id="588df-253">Clearing package caches can also be accomplished with the [nuget.exe](https://www.nuget.org/downloads) tool and executing the command `nuget locals all -clear`.</span></span> <span data-ttu-id="588df-254">*nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).</span><span class="sxs-lookup"><span data-stu-id="588df-254">*nuget.exe* isn't a bundled install with the Windows desktop operating system and must be obtained separately from the [NuGet website](https://www.nuget.org/downloads).</span></span>

1. <span data-ttu-id="588df-255">Ripristinare e ricompilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="588df-255">Restore and rebuild the project.</span></span>
1. <span data-ttu-id="588df-256">Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.</span><span class="sxs-lookup"><span data-stu-id="588df-256">Delete all of the files in the deployment folder on the server prior to redeploying the app.</span></span>

### <a name="slow-or-hanging-app"></a><span data-ttu-id="588df-257">App lenta o bloccata</span><span class="sxs-lookup"><span data-stu-id="588df-257">Slow or hanging app</span></span>

<span data-ttu-id="588df-258">Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.</span><span class="sxs-lookup"><span data-stu-id="588df-258">A *crash dump* is a snapshot of the system's memory and can help determine the cause of an app crash, startup failure, or slow app.</span></span>

#### <a name="app-crashes-or-encounters-an-exception"></a><span data-ttu-id="588df-259">Arresto anomalo o eccezione di un'app</span><span class="sxs-lookup"><span data-stu-id="588df-259">App crashes or encounters an exception</span></span>

<span data-ttu-id="588df-260">Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):</span><span class="sxs-lookup"><span data-stu-id="588df-260">Obtain and analyze a dump from [Windows Error Reporting (WER)](/windows/desktop/wer/windows-error-reporting):</span></span>

1. <span data-ttu-id="588df-261">Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`.</span><span class="sxs-lookup"><span data-stu-id="588df-261">Create a folder to hold crash dump files at `c:\dumps`.</span></span>
1. <span data-ttu-id="588df-262">Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/samples/scripts/EnableDumps.ps1) con il nome dell'eseguibile dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="588df-262">Run the [EnableDumps PowerShell script](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/samples/scripts/EnableDumps.ps1) with the application executable name:</span></span>

   ```powershell
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. <span data-ttu-id="588df-263">Eseguire l'app nelle condizioni che causano l'arresto anomalo.</span><span class="sxs-lookup"><span data-stu-id="588df-263">Run the app under the conditions that cause the crash to occur.</span></span>
1. <span data-ttu-id="588df-264">Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/samples/scripts/DisableDumps.ps1):</span><span class="sxs-lookup"><span data-stu-id="588df-264">After the crash has occurred, run the [DisableDumps PowerShell script](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/samples/scripts/DisableDumps.ps1):</span></span>

   ```powershell
   .\DisableDumps {APPLICATION EXE}
   ```

<span data-ttu-id="588df-265">Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente.</span><span class="sxs-lookup"><span data-stu-id="588df-265">After an app crashes and dump collection is complete, the app is allowed to terminate normally.</span></span> <span data-ttu-id="588df-266">Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.</span><span class="sxs-lookup"><span data-stu-id="588df-266">The PowerShell script configures WER to collect up to five dumps per app.</span></span>

> [!WARNING]
> <span data-ttu-id="588df-267">I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).</span><span class="sxs-lookup"><span data-stu-id="588df-267">Crash dumps might take up a large amount of disk space (up to several gigabytes each).</span></span>

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a><span data-ttu-id="588df-268">L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente</span><span class="sxs-lookup"><span data-stu-id="588df-268">App hangs, fails during startup, or runs normally</span></span>

<span data-ttu-id="588df-269">Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.</span><span class="sxs-lookup"><span data-stu-id="588df-269">When an app *hangs* (stops responding but doesn't crash), fails during startup, or runs normally, see [User-Mode Dump Files: Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) to select an appropriate tool to produce the dump.</span></span>

#### <a name="analyze-the-dump"></a><span data-ttu-id="588df-270">Analizzare il dump</span><span class="sxs-lookup"><span data-stu-id="588df-270">Analyze the dump</span></span>

<span data-ttu-id="588df-271">È possibile analizzare un dump usando diversi approcci.</span><span class="sxs-lookup"><span data-stu-id="588df-271">A dump can be analyzed using several approaches.</span></span> <span data-ttu-id="588df-272">Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).</span><span class="sxs-lookup"><span data-stu-id="588df-272">For more information, see [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="588df-273">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="588df-273">Additional resources</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-5.0"
* <span data-ttu-id="588df-274">[Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel/endpoints) (include la configurazione HTTPS e il supporto SNI)</span><span class="sxs-lookup"><span data-stu-id="588df-274">[Kestrel endpoint configuration](xref:fundamentals/servers/kestrel/endpoints) (includes HTTPS configuration and SNI support)</span></span>
::: moniker-end
::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"
* <span data-ttu-id="588df-275">[Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel#endpoint-configuration) (include la configurazione HTTPS e il supporto SNI)</span><span class="sxs-lookup"><span data-stu-id="588df-275">[Kestrel endpoint configuration](xref:fundamentals/servers/kestrel#endpoint-configuration) (includes HTTPS configuration and SNI support)</span></span>
::: moniker-end

::: moniker range=">= aspnetcore-3.0"
* <xref:fundamentals/host/generic-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="588df-276">È possibile ospitare un'app ASP.NET Core in Windows come [servizio Windows](/dotnet/framework/windows-services/introduction-to-windows-service-applications) senza usare IIS.</span><span class="sxs-lookup"><span data-stu-id="588df-276">An ASP.NET Core app can be hosted on Windows as a [Windows Service](/dotnet/framework/windows-services/introduction-to-windows-service-applications) without using IIS.</span></span> <span data-ttu-id="588df-277">Quando è ospitata come servizio di Windows, l'app viene avviata automaticamente dopo il riavvio del server.</span><span class="sxs-lookup"><span data-stu-id="588df-277">When hosted as a Windows Service, the app automatically starts after server reboots.</span></span>

<span data-ttu-id="588df-278">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="588df-278">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="588df-279">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="588df-279">Prerequisites</span></span>

* [<span data-ttu-id="588df-280">ASP.NET Core SDK 2.1 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="588df-280">ASP.NET Core SDK 2.1 or later</span></span>](https://dotnet.microsoft.com/download)
* [<span data-ttu-id="588df-281">PowerShell 6,2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="588df-281">PowerShell 6.2 or later</span></span>](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a><span data-ttu-id="588df-282">Configurazione delle app</span><span class="sxs-lookup"><span data-stu-id="588df-282">App configuration</span></span>

<span data-ttu-id="588df-283">L'app richiede i riferimenti ai pacchetti [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) e [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog).</span><span class="sxs-lookup"><span data-stu-id="588df-283">The app requires package references for [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) and [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog).</span></span>

<span data-ttu-id="588df-284">Per eseguire test e debug durante l'esecuzione all'esterno di un servizio, aggiungere il codice per determinare se l'app è in esecuzione come servizio o è un'app console.</span><span class="sxs-lookup"><span data-stu-id="588df-284">To test and debug when running outside of a service, add code to determine if the app is running as a service or a console app.</span></span> <span data-ttu-id="588df-285">Controllare se il debugger è collegato o se è presente un'opzione `--console`.</span><span class="sxs-lookup"><span data-stu-id="588df-285">Inspect if the debugger is attached or a `--console` switch is present.</span></span> <span data-ttu-id="588df-286">Se una delle due condizioni è vera (l'app non è eseguita come servizio), chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>.</span><span class="sxs-lookup"><span data-stu-id="588df-286">If either condition is true (the app isn't run as a service), call <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>.</span></span> <span data-ttu-id="588df-287">Se le condizioni sono false (l'app è eseguita come servizio):</span><span class="sxs-lookup"><span data-stu-id="588df-287">If the conditions are false (the app is run as a service):</span></span>

* <span data-ttu-id="588df-288">Chiamare <xref:System.IO.Directory.SetCurrentDirectory*> e usare un percorso per la posizione di pubblicazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-288">Call <xref:System.IO.Directory.SetCurrentDirectory*> and use a path to the app's published location.</span></span> <span data-ttu-id="588df-289">Non chiamare <xref:System.IO.Directory.GetCurrentDirectory*> per ottenere il percorso perché un'app servizio Windows restituisce la cartella *C:\\WINDOWS\\system32* quando viene chiamato <xref:System.IO.Directory.GetCurrentDirectory*>.</span><span class="sxs-lookup"><span data-stu-id="588df-289">Don't call <xref:System.IO.Directory.GetCurrentDirectory*> to obtain the path because a Windows Service app returns the *C:\\WINDOWS\\system32* folder when <xref:System.IO.Directory.GetCurrentDirectory*> is called.</span></span> <span data-ttu-id="588df-290">Per altre informazioni, vedere la sezione [Directory corrente e radice del contenuto](#current-directory-and-content-root).</span><span class="sxs-lookup"><span data-stu-id="588df-290">For more information, see the [Current directory and content root](#current-directory-and-content-root) section.</span></span> <span data-ttu-id="588df-291">Questo passaggio viene eseguito prima che l'app venga configurata in `CreateWebHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="588df-291">This step is performed before the app is configured in `CreateWebHostBuilder`.</span></span>
* <span data-ttu-id="588df-292">Chiamare <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per eseguire l'app come servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-292">Call <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> to run the app as a service.</span></span>

<span data-ttu-id="588df-293">Dato che il [provider di configurazione della riga di comando](xref:fundamentals/configuration/index#command-line-configuration-provider) richiede coppie nome-valore per gli argomenti della riga di comando, l'opzione `--console` viene rimossa dagli argomenti prima che <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> riceva gli argomenti.</span><span class="sxs-lookup"><span data-stu-id="588df-293">Because the [Command-line Configuration Provider](xref:fundamentals/configuration/index#command-line-configuration-provider) requires name-value pairs for command-line arguments, the `--console` switch is removed from the arguments before <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> receives the arguments.</span></span>

<span data-ttu-id="588df-294">Per scrivere nel registro eventi di Windows, aggiungere il provider EventLog a <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>.</span><span class="sxs-lookup"><span data-stu-id="588df-294">To write to the Windows Event Log, add the EventLog provider to <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>.</span></span> <span data-ttu-id="588df-295">Impostare il livello di registrazione con la chiave `Logging:LogLevel:Default` nel file *appsettings.Production.json*.</span><span class="sxs-lookup"><span data-stu-id="588df-295">Set the logging level with the `Logging:LogLevel:Default` key in the *appsettings.Production.json* file.</span></span>

<span data-ttu-id="588df-296">Nel seguente esempio dell'app di esempio, viene chiamato `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per gestire gli eventi di durata all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-296">In the following example from the sample app, `RunAsCustomService` is called instead of <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in order to handle lifetime events within the app.</span></span> <span data-ttu-id="588df-297">Per altre informazioni, vedere la sezione [Gestire gli eventi di avvio e arresto](#handle-starting-and-stopping-events).</span><span class="sxs-lookup"><span data-stu-id="588df-297">For more information, see the [Handle starting and stopping events](#handle-starting-and-stopping-events) section.</span></span>

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a><span data-ttu-id="588df-298">Tipo di distribuzione</span><span class="sxs-lookup"><span data-stu-id="588df-298">Deployment type</span></span>

<span data-ttu-id="588df-299">Per informazioni e consigli sugli scenari di distribuzione, vedere [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/).</span><span class="sxs-lookup"><span data-stu-id="588df-299">For information and advice on deployment scenarios, see [.NET Core application deployment](/dotnet/core/deploying/).</span></span>

### <a name="sdk"></a><span data-ttu-id="588df-300">SDK</span><span class="sxs-lookup"><span data-stu-id="588df-300">SDK</span></span>

<span data-ttu-id="588df-301">Per un servizio basato su app Web che usa le Razor pagine o i framework MVC, specificare Web SDK nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="588df-301">For a web app-based service that uses the Razor Pages or MVC frameworks, specify the Web SDK in the project file:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

<span data-ttu-id="588df-302">Se il servizio esegue solo attività in background, ad esempio [servizi ospitati](xref:fundamentals/host/hosted-services), specificare l'SDK del ruolo di lavoro nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="588df-302">If the service only executes background tasks (for example, [hosted services](xref:fundamentals/host/hosted-services)), specify the Worker SDK in the project file:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a><span data-ttu-id="588df-303">Distribuzione dipendente dal framework</span><span class="sxs-lookup"><span data-stu-id="588df-303">Framework-dependent deployment (FDD)</span></span>

<span data-ttu-id="588df-304">La distribuzione dipendente dal framework si basa sulla presenza di una versione condivisa a livello di sistema di .NET Core nel sistema di destinazione.</span><span class="sxs-lookup"><span data-stu-id="588df-304">Framework-dependent deployment (FDD) relies on the presence of a shared system-wide version of .NET Core on the target system.</span></span> <span data-ttu-id="588df-305">Quando lo scenario di distribuzione dipendente dal framework viene implementato in base alle indicazioni di questo articolo, l'SDK genera un file eseguibile (con estensione *exe*) detto *eseguibile dipendente dal framework*.</span><span class="sxs-lookup"><span data-stu-id="588df-305">When the FDD scenario is adopted following the guidance in this article, the SDK produces an executable (*.exe*), called a *framework-dependent executable*.</span></span>

<span data-ttu-id="588df-306">L'identificatore di Windows [Runtime (RID)](/dotnet/core/rid-catalog) ( [\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier) ) contiene il Framework di destinazione.</span><span class="sxs-lookup"><span data-stu-id="588df-306">The Windows [Runtime Identifier (RID)](/dotnet/core/rid-catalog) ([\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier)) contains the target framework.</span></span> <span data-ttu-id="588df-307">Nell'esempio seguente il RID è impostato su `win7-x64`.</span><span class="sxs-lookup"><span data-stu-id="588df-307">In the following example, the RID is set to `win7-x64`.</span></span> <span data-ttu-id="588df-308">La proprietà `<SelfContained>` è impostata su `false`.</span><span class="sxs-lookup"><span data-stu-id="588df-308">The `<SelfContained>` property is set to `false`.</span></span> <span data-ttu-id="588df-309">Queste proprietà indicano all'SDK di generare un file eseguibile (con estensione *exe*) per Windows e un'app che dipende dal framework .NET Core condiviso.</span><span class="sxs-lookup"><span data-stu-id="588df-309">These properties instruct the SDK to generate an executable (*.exe*) file for Windows and an app that depends on the shared .NET Core framework.</span></span>

<span data-ttu-id="588df-310">Un file *web.config*, che viene normalmente generato quando si pubblica un'app ASP.NET Core, non è necessario per un'app di servizi Windows.</span><span class="sxs-lookup"><span data-stu-id="588df-310">A *web.config* file, which is normally produced when publishing an ASP.NET Core app, is unnecessary for a Windows Services app.</span></span> <span data-ttu-id="588df-311">Per disabilitare la creazione del file *web.config*, aggiungere la proprietà `<IsTransformWebConfigDisabled>` impostata su `true`.</span><span class="sxs-lookup"><span data-stu-id="588df-311">To disable the creation of the *web.config* file, add the `<IsTransformWebConfigDisabled>` property set to `true`.</span></span>

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a><span data-ttu-id="588df-312">Distribuzione autonoma</span><span class="sxs-lookup"><span data-stu-id="588df-312">Self-contained deployment (SCD)</span></span>

<span data-ttu-id="588df-313">Una distribuzione autonoma non si basa sulla presenza di un framework condiviso nel sistema host.</span><span class="sxs-lookup"><span data-stu-id="588df-313">Self-contained deployment (SCD) doesn't rely on the presence of a shared framework on the host system.</span></span> <span data-ttu-id="588df-314">Il runtime e le dipendenze dell'app vengono distribuiti con l'app.</span><span class="sxs-lookup"><span data-stu-id="588df-314">The runtime and the app's dependencies are deployed with the app.</span></span>

<span data-ttu-id="588df-315">Un [identificatore di runtime (RID)](/dotnet/core/rid-catalog) di Windows viene incluso nel `<PropertyGroup>` che contiene il framework di destinazione:</span><span class="sxs-lookup"><span data-stu-id="588df-315">A Windows [Runtime Identifier (RID)](/dotnet/core/rid-catalog) is included in the `<PropertyGroup>` that contains the target framework:</span></span>

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

<span data-ttu-id="588df-316">Per eseguire la pubblicazione per più identificatori di runtime:</span><span class="sxs-lookup"><span data-stu-id="588df-316">To publish for multiple RIDs:</span></span>

* <span data-ttu-id="588df-317">Specificare gli identificatori di runtime in un elenco delimitato da punto e virgola.</span><span class="sxs-lookup"><span data-stu-id="588df-317">Provide the RIDs in a semicolon-delimited list.</span></span>
* <span data-ttu-id="588df-318">Usare il nome della proprietà [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plurale).</span><span class="sxs-lookup"><span data-stu-id="588df-318">Use the property name [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plural).</span></span>

<span data-ttu-id="588df-319">Per altre informazioni, vedere il [Catalogo RID di .NET Core](/dotnet/core/rid-catalog).</span><span class="sxs-lookup"><span data-stu-id="588df-319">For more information, see [.NET Core RID Catalog](/dotnet/core/rid-catalog).</span></span>

<span data-ttu-id="588df-320">Una proprietà `<SelfContained>` è impostata su `true`:</span><span class="sxs-lookup"><span data-stu-id="588df-320">A `<SelfContained>` property is set to `true`:</span></span>

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a><span data-ttu-id="588df-321">Account utente del servizio</span><span class="sxs-lookup"><span data-stu-id="588df-321">Service user account</span></span>

<span data-ttu-id="588df-322">Per creare un account utente per un servizio, usare il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) da una shell dei comandi di PowerShell 6 amministrativa.</span><span class="sxs-lookup"><span data-stu-id="588df-322">To create a user account for a service, use the [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet from an administrative PowerShell 6 command shell.</span></span>

<span data-ttu-id="588df-323">In Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763) o versioni successive:</span><span class="sxs-lookup"><span data-stu-id="588df-323">On Windows 10 October 2018 Update (version 1809/build 10.0.17763) or later:</span></span>

```powershell
New-LocalUser -Name {SERVICE NAME}
```

<span data-ttu-id="588df-324">In un sistema operativo Windows precedente ad Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763):</span><span class="sxs-lookup"><span data-stu-id="588df-324">On Windows OS earlier than the Windows 10 October 2018 Update (version 1809/build 10.0.17763):</span></span>

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

<span data-ttu-id="588df-325">Fornire una [password complessa](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) quando richiesto.</span><span class="sxs-lookup"><span data-stu-id="588df-325">Provide a [strong password](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) when prompted.</span></span>

<span data-ttu-id="588df-326">Se non viene specificato il parametro `-AccountExpires` per il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) con un valore <xref:System.DateTime> di scadenza, l'account non scade.</span><span class="sxs-lookup"><span data-stu-id="588df-326">Unless the `-AccountExpires` parameter is supplied to the [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet with an expiration <xref:System.DateTime>, the account doesn't expire.</span></span>

<span data-ttu-id="588df-327">Per altre informazioni, vedere [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) e [Service User Accounts](/windows/desktop/services/service-user-accounts) (Account utente di servizio).</span><span class="sxs-lookup"><span data-stu-id="588df-327">For more information, see [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) and [Service User Accounts](/windows/desktop/services/service-user-accounts).</span></span>

<span data-ttu-id="588df-328">Un approccio alternativo per la gestione degli utenti quando si usa Active Directory consiste nell'usare account del servizio gestito.</span><span class="sxs-lookup"><span data-stu-id="588df-328">An alternative approach to managing users when using Active Directory is to use Managed Service Accounts.</span></span> <span data-ttu-id="588df-329">Per altre informazioni, vedere [Panoramica degli account del servizio gestito del gruppo](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).</span><span class="sxs-lookup"><span data-stu-id="588df-329">For more information, see [Group Managed Service Accounts Overview](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).</span></span>

## <a name="log-on-as-a-service-rights"></a><span data-ttu-id="588df-330">Diritti Accesso come servizio</span><span class="sxs-lookup"><span data-stu-id="588df-330">Log on as a service rights</span></span>

<span data-ttu-id="588df-331">Per stabilire i diritti *Accesso come servizio* per un account utente di servizio:</span><span class="sxs-lookup"><span data-stu-id="588df-331">To establish *Log on as a service* rights for a service user account:</span></span>

1. <span data-ttu-id="588df-332">Aprire l'editor Criteri di sicurezza locali eseguendo *secpol.msc*.</span><span class="sxs-lookup"><span data-stu-id="588df-332">Open the Local Security Policy editor by running *secpol.msc*.</span></span>
1. <span data-ttu-id="588df-333">Espandere il nodo **Criteri locali** e selezionare **Assegnazione diritti utente**.</span><span class="sxs-lookup"><span data-stu-id="588df-333">Expand the **Local Policies** node and select **User Rights Assignment**.</span></span>
1. <span data-ttu-id="588df-334">Aprire il criterio **Accesso come servizio**.</span><span class="sxs-lookup"><span data-stu-id="588df-334">Open the **Log on as a service** policy.</span></span>
1. <span data-ttu-id="588df-335">Selezionare **Aggiungi utente o gruppo**.</span><span class="sxs-lookup"><span data-stu-id="588df-335">Select **Add User or Group**.</span></span>
1. <span data-ttu-id="588df-336">Specificare il nome oggetto (account utente) in uno dei modi seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-336">Provide the object name (user account) using either of the following approaches:</span></span>
   1. <span data-ttu-id="588df-337">Digitare l'account utente (`{DOMAIN OR COMPUTER NAME\USER}`) nel campo del nome oggetto e scegliere **OK** per aggiungere l'utente al criterio.</span><span class="sxs-lookup"><span data-stu-id="588df-337">Type the user account (`{DOMAIN OR COMPUTER NAME\USER}`) in the object name field and select **OK** to add the user to the policy.</span></span>
   1. <span data-ttu-id="588df-338">Fare clic su **Advanced** (Avanzate).</span><span class="sxs-lookup"><span data-stu-id="588df-338">Select **Advanced**.</span></span> <span data-ttu-id="588df-339">Selezionare **Trova**.</span><span class="sxs-lookup"><span data-stu-id="588df-339">Select **Find Now**.</span></span> <span data-ttu-id="588df-340">Selezionare l'account utente dall'elenco.</span><span class="sxs-lookup"><span data-stu-id="588df-340">Select the user account from the list.</span></span> <span data-ttu-id="588df-341">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="588df-341">Select **OK**.</span></span> <span data-ttu-id="588df-342">Scegliere di nuovo **OK** per aggiungere l'utente al criterio.</span><span class="sxs-lookup"><span data-stu-id="588df-342">Select **OK** again to add the user to the policy.</span></span>
1. <span data-ttu-id="588df-343">Scegliere **OK** o **Applica** per accettare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="588df-343">Select **OK** or **Apply** to accept the changes.</span></span>

## <a name="create-and-manage-the-windows-service"></a><span data-ttu-id="588df-344">Creare e gestire il servizio di Windows</span><span class="sxs-lookup"><span data-stu-id="588df-344">Create and manage the Windows Service</span></span>

### <a name="create-a-service"></a><span data-ttu-id="588df-345">Creare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-345">Create a service</span></span>

<span data-ttu-id="588df-346">Usare i comandi di PowerShell per registrare un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-346">Use PowerShell commands to register a service.</span></span> <span data-ttu-id="588df-347">Da una shell dei comandi di PowerShell 6 amministrativa eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-347">From an administrative PowerShell 6 command shell, execute the following commands:</span></span>

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* <span data-ttu-id="588df-348">`{EXE PATH}`: Percorso della cartella dell'app nell'host (ad esempio, `d:\myservice` ).</span><span class="sxs-lookup"><span data-stu-id="588df-348">`{EXE PATH}`: Path to the app's folder on the host (for example, `d:\myservice`).</span></span> <span data-ttu-id="588df-349">Non includere l'eseguibile dell'app nel percorso.</span><span class="sxs-lookup"><span data-stu-id="588df-349">Don't include the app's executable in the path.</span></span> <span data-ttu-id="588df-350">Non è necessario aggiungere una barra finale.</span><span class="sxs-lookup"><span data-stu-id="588df-350">A trailing slash isn't required.</span></span>
* <span data-ttu-id="588df-351">`{DOMAIN OR COMPUTER NAME\USER}`: Account utente del servizio (ad esempio, `Contoso\ServiceUser` ).</span><span class="sxs-lookup"><span data-stu-id="588df-351">`{DOMAIN OR COMPUTER NAME\USER}`: Service user account (for example, `Contoso\ServiceUser`).</span></span>
* <span data-ttu-id="588df-352">`{SERVICE NAME}`: Nome del servizio (ad esempio, `MyService` ).</span><span class="sxs-lookup"><span data-stu-id="588df-352">`{SERVICE NAME}`: Service name (for example, `MyService`).</span></span>
* <span data-ttu-id="588df-353">`{EXE FILE PATH}`: Percorso eseguibile dell'app (ad esempio, `d:\myservice\myservice.exe` ).</span><span class="sxs-lookup"><span data-stu-id="588df-353">`{EXE FILE PATH}`: The app's executable path (for example, `d:\myservice\myservice.exe`).</span></span> <span data-ttu-id="588df-354">Includere il nome del file eseguibile con l'estensione.</span><span class="sxs-lookup"><span data-stu-id="588df-354">Include the executable's file name with extension.</span></span>
* <span data-ttu-id="588df-355">`{DESCRIPTION}`: Descrizione del servizio (ad esempio, `My sample service` ).</span><span class="sxs-lookup"><span data-stu-id="588df-355">`{DESCRIPTION}`: Service description (for example, `My sample service`).</span></span>
* <span data-ttu-id="588df-356">`{DISPLAY NAME}`: Nome visualizzato del servizio (ad esempio, `My Service` ).</span><span class="sxs-lookup"><span data-stu-id="588df-356">`{DISPLAY NAME}`: Service display name (for example, `My Service`).</span></span>

### <a name="start-a-service"></a><span data-ttu-id="588df-357">Avviare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-357">Start a service</span></span>

<span data-ttu-id="588df-358">Per avviare un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-358">Start a service with the following PowerShell 6 command:</span></span>

```powershell
Start-Service -Name {SERVICE NAME}
```

<span data-ttu-id="588df-359">L'avvio del servizio richiede alcuni secondi.</span><span class="sxs-lookup"><span data-stu-id="588df-359">The command takes a few seconds to start the service.</span></span>

### <a name="determine-a-services-status"></a><span data-ttu-id="588df-360">Determinare lo stato di un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-360">Determine a service's status</span></span>

<span data-ttu-id="588df-361">Per verificare lo stato di un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-361">To check the status of a service, use the following PowerShell 6 command:</span></span>

```powershell
Get-Service -Name {SERVICE NAME}
```

<span data-ttu-id="588df-362">Lo stato viene indicato con uno dei valori seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-362">The status is reported as one of the following values:</span></span>

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a><span data-ttu-id="588df-363">Arrestare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-363">Stop a service</span></span>

<span data-ttu-id="588df-364">Per arrestare un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-364">Stop a service with the following Powershell 6 command:</span></span>

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a><span data-ttu-id="588df-365">Rimuovere un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-365">Remove a service</span></span>

<span data-ttu-id="588df-366">Dopo il breve lasso di tempo necessario per arrestare un servizio, rimuovere il servizio con il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-366">After a short delay to stop a service, remove a service with the following Powershell 6 command:</span></span>

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a><span data-ttu-id="588df-367">Gestire gli eventi di avvio e arresto</span><span class="sxs-lookup"><span data-stu-id="588df-367">Handle starting and stopping events</span></span>

<span data-ttu-id="588df-368">Per gestire gli eventi <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>, <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*> e <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*>:</span><span class="sxs-lookup"><span data-stu-id="588df-368">To handle <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>, <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*>, and <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*> events:</span></span>

1. <span data-ttu-id="588df-369">Creare una classe che deriva da <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> con i metodi `OnStarting`, `OnStarted` e `OnStopping`:</span><span class="sxs-lookup"><span data-stu-id="588df-369">Create a class that derives from <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> with the `OnStarting`, `OnStarted`, and `OnStopping` methods:</span></span>

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. <span data-ttu-id="588df-370">Creare un metodo di estensione per <xref:Microsoft.AspNetCore.Hosting.IWebHost> che passa `CustomWebHostService` a <xref:System.ServiceProcess.ServiceBase.Run*>:</span><span class="sxs-lookup"><span data-stu-id="588df-370">Create an extension method for <xref:Microsoft.AspNetCore.Hosting.IWebHost> that passes the `CustomWebHostService` to <xref:System.ServiceProcess.ServiceBase.Run*>:</span></span>

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. <span data-ttu-id="588df-371">In `Program.Main` chiamare il metodo di estensione `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>:</span><span class="sxs-lookup"><span data-stu-id="588df-371">In `Program.Main`, call the `RunAsCustomService` extension method instead of <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>:</span></span>

   ```csharp
   host.RunAsCustomService();
   ```

   <span data-ttu-id="588df-372">Per visualizzare il percorso di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in `Program.Main`, fare riferimento all'esempio di codice illustrato nella sezione [Tipo di distribuzione](#deployment-type).</span><span class="sxs-lookup"><span data-stu-id="588df-372">To see the location of <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in `Program.Main`, refer to the code sample shown in the [Deployment type](#deployment-type) section.</span></span>

## <a name="proxy-server-and-load-balancer-scenarios"></a><span data-ttu-id="588df-373">Scenari con server proxy e servizi di bilanciamento del carico</span><span class="sxs-lookup"><span data-stu-id="588df-373">Proxy server and load balancer scenarios</span></span>

<span data-ttu-id="588df-374">I servizi che interagiscono con le richieste da Internet o da una rete aziendale e si trovano dietro un proxy o un servizio di bilanciamento del carico potrebbero richiedere una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="588df-374">Services that interact with requests from the Internet or a corporate network and are behind a proxy or load balancer might require additional configuration.</span></span> <span data-ttu-id="588df-375">Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.</span><span class="sxs-lookup"><span data-stu-id="588df-375">For more information, see <xref:host-and-deploy/proxy-load-balancer>.</span></span>

## <a name="configure-endpoints"></a><span data-ttu-id="588df-376">Configurare gli endpoint</span><span class="sxs-lookup"><span data-stu-id="588df-376">Configure endpoints</span></span>

<span data-ttu-id="588df-377">Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`.</span><span class="sxs-lookup"><span data-stu-id="588df-377">By default, ASP.NET Core binds to `http://localhost:5000`.</span></span> <span data-ttu-id="588df-378">Configurare l'URL e la porta impostando la `ASPNETCORE_URLS` variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="588df-378">Configure the URL and port by setting the `ASPNETCORE_URLS` environment variable.</span></span>

<span data-ttu-id="588df-379">Per ulteriori approcci alla configurazione di porte e URL, vedere l'articolo relativo al server pertinente:</span><span class="sxs-lookup"><span data-stu-id="588df-379">For additional URL and port configuration approaches, see the relevant server article:</span></span>

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

<span data-ttu-id="588df-380">Le linee guida precedenti riguardano il supporto per gli endpoint HTTPS.</span><span class="sxs-lookup"><span data-stu-id="588df-380">The preceding guidance covers support for HTTPS endpoints.</span></span> <span data-ttu-id="588df-381">Ad esempio, configurare l'app per HTTPS quando si usa l'autenticazione con un servizio Windows.</span><span class="sxs-lookup"><span data-stu-id="588df-381">For example, configure the app for HTTPS when authentication is used with a Windows Service.</span></span>

> [!NOTE]
> <span data-ttu-id="588df-382">L'uso del certificato di sviluppo ASP.NET Core HTTPS per proteggere un endpoint del servizio non è supportato.</span><span class="sxs-lookup"><span data-stu-id="588df-382">Use of the ASP.NET Core HTTPS development certificate to secure a service endpoint isn't supported.</span></span>

## <a name="current-directory-and-content-root"></a><span data-ttu-id="588df-383">Directory corrente e radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="588df-383">Current directory and content root</span></span>

<span data-ttu-id="588df-384">La directory di lavoro corrente restituita chiamando <xref:System.IO.Directory.GetCurrentDirectory*> per un servizio Windows è la cartella *C:\\WINDOWS\\system32*.</span><span class="sxs-lookup"><span data-stu-id="588df-384">The current working directory returned by calling <xref:System.IO.Directory.GetCurrentDirectory*> for a Windows Service is the *C:\\WINDOWS\\system32* folder.</span></span> <span data-ttu-id="588df-385">La cartella *system32* non è un percorso appropriato per archiviare i file di un servizio, ad esempio i file di impostazioni.</span><span class="sxs-lookup"><span data-stu-id="588df-385">The *system32* folder isn't a suitable location to store a service's files (for example, settings files).</span></span> <span data-ttu-id="588df-386">Usare uno degli approcci seguenti per gestire e accedere agli asset e ai file di impostazioni di un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-386">Use one of the following approaches to maintain and access a service's assets and settings files.</span></span>

### <a name="set-the-content-root-path-to-the-apps-folder"></a><span data-ttu-id="588df-387">Impostare il percorso radice del contenuto sulla cartella dell'app</span><span class="sxs-lookup"><span data-stu-id="588df-387">Set the content root path to the app's folder</span></span>

<span data-ttu-id="588df-388"><xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> è lo stesso percorso fornito all'argomento `binPath` durante la creazione di un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-388">The <xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> is the same path provided to the `binPath` argument when a service is created.</span></span> <span data-ttu-id="588df-389">Anziché chiamare `GetCurrentDirectory` per creare percorsi per i file di impostazioni, chiamare <xref:System.IO.Directory.SetCurrentDirectory*> con il percorso della radice del [contenuto](xref:fundamentals/index#content-root)dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-389">Instead of calling `GetCurrentDirectory` to create paths to settings files, call <xref:System.IO.Directory.SetCurrentDirectory*> with the path to the app's [content root](xref:fundamentals/index#content-root).</span></span>

<span data-ttu-id="588df-390">In `Program.Main`, determinare il percorso della cartella dell'eseguibile del servizio e usare il percorso per stabilire la radice del contenuto dell'app:</span><span class="sxs-lookup"><span data-stu-id="588df-390">In `Program.Main`, determine the path to the folder of the service's executable and use the path to establish the app's content root:</span></span>

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a><span data-ttu-id="588df-391">Archiviare i file di un servizio in un percorso appropriato nel disco</span><span class="sxs-lookup"><span data-stu-id="588df-391">Store a service's files in a suitable location on disk</span></span>

<span data-ttu-id="588df-392">Specificare un percorso assoluto con <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> quando si usa un <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per la cartella contenente i file.</span><span class="sxs-lookup"><span data-stu-id="588df-392">Specify an absolute path with <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> when using an <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> to the folder containing the files.</span></span>

## <a name="troubleshoot"></a><span data-ttu-id="588df-393">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="588df-393">Troubleshoot</span></span>

<span data-ttu-id="588df-394">Per risolvere i problemi relativi a un'app di servizio Windows, vedere <xref:test/troubleshoot> .</span><span class="sxs-lookup"><span data-stu-id="588df-394">To troubleshoot a Windows Service app, see <xref:test/troubleshoot>.</span></span>

### <a name="common-errors"></a><span data-ttu-id="588df-395">Errori comuni</span><span class="sxs-lookup"><span data-stu-id="588df-395">Common errors</span></span>

* <span data-ttu-id="588df-396">È in uso una versione precedente o provvisoria di PowerShell.</span><span class="sxs-lookup"><span data-stu-id="588df-396">An old or pre-release version of PowerShell is in use.</span></span>
* <span data-ttu-id="588df-397">Il servizio registrato non usa l'output **pubblicato** dell'app dal comando [DotNet Publish](/dotnet/core/tools/dotnet-publish) .</span><span class="sxs-lookup"><span data-stu-id="588df-397">The registered service doesn't use the app's **published** output from the [dotnet publish](/dotnet/core/tools/dotnet-publish) command.</span></span> <span data-ttu-id="588df-398">L'output del comando [DotNet Build](/dotnet/core/tools/dotnet-build) non è supportato per la distribuzione di app.</span><span class="sxs-lookup"><span data-stu-id="588df-398">Output of the [dotnet build](/dotnet/core/tools/dotnet-build) command isn't supported for app deployment.</span></span> <span data-ttu-id="588df-399">Gli asset pubblicati si trovano in una delle cartelle seguenti, a seconda del tipo di distribuzione:</span><span class="sxs-lookup"><span data-stu-id="588df-399">Published assets are found in either of the following folders depending on the deployment type:</span></span>
  * <span data-ttu-id="588df-400">*bin/Release/{Target Framework}/Publish* (FDD)</span><span class="sxs-lookup"><span data-stu-id="588df-400">*bin/Release/{TARGET FRAMEWORK}/publish* (FDD)</span></span>
  * <span data-ttu-id="588df-401">*bin/Release/{Target Framework}/{Runtime Identifier}/Publish* (SCD)</span><span class="sxs-lookup"><span data-stu-id="588df-401">*bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)</span></span>
* <span data-ttu-id="588df-402">Il servizio non è nello stato in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="588df-402">The service isn't in the RUNNING state.</span></span>
* <span data-ttu-id="588df-403">I percorsi delle risorse utilizzate dall'app, ad esempio i certificati, non sono corretti.</span><span class="sxs-lookup"><span data-stu-id="588df-403">The paths to resources that the app uses (for example, certificates) are incorrect.</span></span> <span data-ttu-id="588df-404">Il percorso di base di un servizio Windows è *c: \\ Windows \\ system32*.</span><span class="sxs-lookup"><span data-stu-id="588df-404">The base path of a Windows Service is *c:\\Windows\\System32*.</span></span>
* <span data-ttu-id="588df-405">L'utente non dispone *di diritti di accesso come servizio* .</span><span class="sxs-lookup"><span data-stu-id="588df-405">The user doesn't have *Log on as a service* rights.</span></span>
* <span data-ttu-id="588df-406">La password dell'utente è scaduta o non è stata passata correttamente durante l'esecuzione del `New-Service` comando di PowerShell.</span><span class="sxs-lookup"><span data-stu-id="588df-406">The user's password is expired or incorrectly passed when executing the `New-Service` PowerShell command.</span></span>
* <span data-ttu-id="588df-407">L'app richiede l'autenticazione ASP.NET Core ma non è configurata per le connessioni protette (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="588df-407">The app requires ASP.NET Core authentication but isn't configured for secure connections (HTTPS).</span></span>
* <span data-ttu-id="588df-408">La porta dell'URL della richiesta non è corretta o non è configurata correttamente nell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-408">The request URL port is incorrect or not configured correctly in the app.</span></span>

### <a name="system-and-application-event-logs"></a><span data-ttu-id="588df-409">Log eventi di sistema e dell'applicazione</span><span class="sxs-lookup"><span data-stu-id="588df-409">System and Application Event Logs</span></span>

<span data-ttu-id="588df-410">Accedere ai registri eventi di sistema e dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="588df-410">Access the System and Application Event Logs:</span></span>

1. <span data-ttu-id="588df-411">Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .</span><span class="sxs-lookup"><span data-stu-id="588df-411">Open the Start menu, search for *Event Viewer*, and select the **Event Viewer** app.</span></span>
1. <span data-ttu-id="588df-412">In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.</span><span class="sxs-lookup"><span data-stu-id="588df-412">In **Event Viewer**, open the **Windows Logs** node.</span></span>
1. <span data-ttu-id="588df-413">Selezionare **sistema** per aprire il registro eventi di sistema.</span><span class="sxs-lookup"><span data-stu-id="588df-413">Select **System** to open the System Event Log.</span></span> <span data-ttu-id="588df-414">Selezionare **Applicazione** per aprire il log eventi dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="588df-414">Select **Application** to open the Application Event Log.</span></span>
1. <span data-ttu-id="588df-415">Cercare gli errori associati all'app in cui si è verificato il problema.</span><span class="sxs-lookup"><span data-stu-id="588df-415">Search for errors associated with the failing app.</span></span>

### <a name="run-the-app-at-a-command-prompt"></a><span data-ttu-id="588df-416">Eseguire l'app da un prompt dei comandi</span><span class="sxs-lookup"><span data-stu-id="588df-416">Run the app at a command prompt</span></span>

<span data-ttu-id="588df-417">Molti errori di avvio non producono informazioni utili nei log eventi.</span><span class="sxs-lookup"><span data-stu-id="588df-417">Many startup errors don't produce useful information in the event logs.</span></span> <span data-ttu-id="588df-418">È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host.</span><span class="sxs-lookup"><span data-stu-id="588df-418">You can find the cause of some errors by running the app at a command prompt on the hosting system.</span></span> <span data-ttu-id="588df-419">Per registrare dettagli aggiuntivi dall'app, abbassare il [livello di registrazione](xref:fundamentals/logging/index#log-level) o eseguire l'app nell' [ambiente di sviluppo](xref:fundamentals/environments).</span><span class="sxs-lookup"><span data-stu-id="588df-419">To log additional detail from the app, lower the [log level](xref:fundamentals/logging/index#log-level) or run the app in the [Development environment](xref:fundamentals/environments).</span></span>

### <a name="clear-package-caches"></a><span data-ttu-id="588df-420">Cancella cache di pacchetti</span><span class="sxs-lookup"><span data-stu-id="588df-420">Clear package caches</span></span>

<span data-ttu-id="588df-421">Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-421">A functioning app may fail immediately after upgrading either the .NET Core SDK on the development machine or changing package versions within the app.</span></span> <span data-ttu-id="588df-422">In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali.</span><span class="sxs-lookup"><span data-stu-id="588df-422">In some cases, incoherent packages may break an app when performing major upgrades.</span></span> <span data-ttu-id="588df-423">La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-423">Most of these issues can be fixed by following these instructions:</span></span>

1. <span data-ttu-id="588df-424">Eliminare le cartelle *bin* e *obj*.</span><span class="sxs-lookup"><span data-stu-id="588df-424">Delete the *bin* and *obj* folders.</span></span>
1. <span data-ttu-id="588df-425">Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="588df-425">Clear the package caches by executing [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) from a command shell.</span></span>

   <span data-ttu-id="588df-426">La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` .</span><span class="sxs-lookup"><span data-stu-id="588df-426">Clearing package caches can also be accomplished with the [nuget.exe](https://www.nuget.org/downloads) tool and executing the command `nuget locals all -clear`.</span></span> <span data-ttu-id="588df-427">*nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).</span><span class="sxs-lookup"><span data-stu-id="588df-427">*nuget.exe* isn't a bundled install with the Windows desktop operating system and must be obtained separately from the [NuGet website](https://www.nuget.org/downloads).</span></span>

1. <span data-ttu-id="588df-428">Ripristinare e ricompilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="588df-428">Restore and rebuild the project.</span></span>
1. <span data-ttu-id="588df-429">Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.</span><span class="sxs-lookup"><span data-stu-id="588df-429">Delete all of the files in the deployment folder on the server prior to redeploying the app.</span></span>

### <a name="slow-or-hanging-app"></a><span data-ttu-id="588df-430">App lenta o bloccata</span><span class="sxs-lookup"><span data-stu-id="588df-430">Slow or hanging app</span></span>

<span data-ttu-id="588df-431">Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.</span><span class="sxs-lookup"><span data-stu-id="588df-431">A *crash dump* is a snapshot of the system's memory and can help determine the cause of an app crash, startup failure, or slow app.</span></span>

#### <a name="app-crashes-or-encounters-an-exception"></a><span data-ttu-id="588df-432">Arresto anomalo o eccezione di un'app</span><span class="sxs-lookup"><span data-stu-id="588df-432">App crashes or encounters an exception</span></span>

<span data-ttu-id="588df-433">Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):</span><span class="sxs-lookup"><span data-stu-id="588df-433">Obtain and analyze a dump from [Windows Error Reporting (WER)](/windows/desktop/wer/windows-error-reporting):</span></span>

1. <span data-ttu-id="588df-434">Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`.</span><span class="sxs-lookup"><span data-stu-id="588df-434">Create a folder to hold crash dump files at `c:\dumps`.</span></span>
1. <span data-ttu-id="588df-435">Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1) con il nome dell'eseguibile dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="588df-435">Run the [EnableDumps PowerShell script](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1) with the application executable name:</span></span>

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. <span data-ttu-id="588df-436">Eseguire l'app nelle condizioni che causano l'arresto anomalo.</span><span class="sxs-lookup"><span data-stu-id="588df-436">Run the app under the conditions that cause the crash to occur.</span></span>
1. <span data-ttu-id="588df-437">Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1):</span><span class="sxs-lookup"><span data-stu-id="588df-437">After the crash has occurred, run the [DisableDumps PowerShell script](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1):</span></span>

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

<span data-ttu-id="588df-438">Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente.</span><span class="sxs-lookup"><span data-stu-id="588df-438">After an app crashes and dump collection is complete, the app is allowed to terminate normally.</span></span> <span data-ttu-id="588df-439">Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.</span><span class="sxs-lookup"><span data-stu-id="588df-439">The PowerShell script configures WER to collect up to five dumps per app.</span></span>

> [!WARNING]
> <span data-ttu-id="588df-440">I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).</span><span class="sxs-lookup"><span data-stu-id="588df-440">Crash dumps might take up a large amount of disk space (up to several gigabytes each).</span></span>

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a><span data-ttu-id="588df-441">L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente</span><span class="sxs-lookup"><span data-stu-id="588df-441">App hangs, fails during startup, or runs normally</span></span>

<span data-ttu-id="588df-442">Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.</span><span class="sxs-lookup"><span data-stu-id="588df-442">When an app *hangs* (stops responding but doesn't crash), fails during startup, or runs normally, see [User-Mode Dump Files: Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) to select an appropriate tool to produce the dump.</span></span>

#### <a name="analyze-the-dump"></a><span data-ttu-id="588df-443">Analizzare il dump</span><span class="sxs-lookup"><span data-stu-id="588df-443">Analyze the dump</span></span>

<span data-ttu-id="588df-444">È possibile analizzare un dump usando diversi approcci.</span><span class="sxs-lookup"><span data-stu-id="588df-444">A dump can be analyzed using several approaches.</span></span> <span data-ttu-id="588df-445">Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).</span><span class="sxs-lookup"><span data-stu-id="588df-445">For more information, see [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="588df-446">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="588df-446">Additional resources</span></span>

* <span data-ttu-id="588df-447">[Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel#endpoint-configuration) (include la configurazione HTTPS e il supporto SNI)</span><span class="sxs-lookup"><span data-stu-id="588df-447">[Kestrel endpoint configuration](xref:fundamentals/servers/kestrel#endpoint-configuration) (includes HTTPS configuration and SNI support)</span></span>
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

<span data-ttu-id="588df-448">È possibile ospitare un'app ASP.NET Core in Windows come [servizio Windows](/dotnet/framework/windows-services/introduction-to-windows-service-applications) senza usare IIS.</span><span class="sxs-lookup"><span data-stu-id="588df-448">An ASP.NET Core app can be hosted on Windows as a [Windows Service](/dotnet/framework/windows-services/introduction-to-windows-service-applications) without using IIS.</span></span> <span data-ttu-id="588df-449">Quando è ospitata come servizio di Windows, l'app viene avviata automaticamente dopo il riavvio del server.</span><span class="sxs-lookup"><span data-stu-id="588df-449">When hosted as a Windows Service, the app automatically starts after server reboots.</span></span>

<span data-ttu-id="588df-450">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="588df-450">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="588df-451">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="588df-451">Prerequisites</span></span>

* [<span data-ttu-id="588df-452">ASP.NET Core SDK 2.1 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="588df-452">ASP.NET Core SDK 2.1 or later</span></span>](https://dotnet.microsoft.com/download)
* [<span data-ttu-id="588df-453">PowerShell 6,2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="588df-453">PowerShell 6.2 or later</span></span>](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a><span data-ttu-id="588df-454">Configurazione delle app</span><span class="sxs-lookup"><span data-stu-id="588df-454">App configuration</span></span>

<span data-ttu-id="588df-455">L'app richiede i riferimenti ai pacchetti [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) e [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog).</span><span class="sxs-lookup"><span data-stu-id="588df-455">The app requires package references for [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) and [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog).</span></span>

<span data-ttu-id="588df-456">Per eseguire test e debug durante l'esecuzione all'esterno di un servizio, aggiungere il codice per determinare se l'app è in esecuzione come servizio o è un'app console.</span><span class="sxs-lookup"><span data-stu-id="588df-456">To test and debug when running outside of a service, add code to determine if the app is running as a service or a console app.</span></span> <span data-ttu-id="588df-457">Controllare se il debugger è collegato o se è presente un'opzione `--console`.</span><span class="sxs-lookup"><span data-stu-id="588df-457">Inspect if the debugger is attached or a `--console` switch is present.</span></span> <span data-ttu-id="588df-458">Se una delle due condizioni è vera (l'app non è eseguita come servizio), chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>.</span><span class="sxs-lookup"><span data-stu-id="588df-458">If either condition is true (the app isn't run as a service), call <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>.</span></span> <span data-ttu-id="588df-459">Se le condizioni sono false (l'app è eseguita come servizio):</span><span class="sxs-lookup"><span data-stu-id="588df-459">If the conditions are false (the app is run as a service):</span></span>

* <span data-ttu-id="588df-460">Chiamare <xref:System.IO.Directory.SetCurrentDirectory*> e usare un percorso per la posizione di pubblicazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-460">Call <xref:System.IO.Directory.SetCurrentDirectory*> and use a path to the app's published location.</span></span> <span data-ttu-id="588df-461">Non chiamare <xref:System.IO.Directory.GetCurrentDirectory*> per ottenere il percorso perché un'app servizio Windows restituisce la cartella *C:\\WINDOWS\\system32* quando viene chiamato <xref:System.IO.Directory.GetCurrentDirectory*>.</span><span class="sxs-lookup"><span data-stu-id="588df-461">Don't call <xref:System.IO.Directory.GetCurrentDirectory*> to obtain the path because a Windows Service app returns the *C:\\WINDOWS\\system32* folder when <xref:System.IO.Directory.GetCurrentDirectory*> is called.</span></span> <span data-ttu-id="588df-462">Per altre informazioni, vedere la sezione [Directory corrente e radice del contenuto](#current-directory-and-content-root).</span><span class="sxs-lookup"><span data-stu-id="588df-462">For more information, see the [Current directory and content root](#current-directory-and-content-root) section.</span></span> <span data-ttu-id="588df-463">Questo passaggio viene eseguito prima che l'app venga configurata in `CreateWebHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="588df-463">This step is performed before the app is configured in `CreateWebHostBuilder`.</span></span>
* <span data-ttu-id="588df-464">Chiamare <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per eseguire l'app come servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-464">Call <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> to run the app as a service.</span></span>

<span data-ttu-id="588df-465">Dato che il [provider di configurazione della riga di comando](xref:fundamentals/configuration/index#command-line-configuration-provider) richiede coppie nome-valore per gli argomenti della riga di comando, l'opzione `--console` viene rimossa dagli argomenti prima che <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> riceva gli argomenti.</span><span class="sxs-lookup"><span data-stu-id="588df-465">Because the [Command-line Configuration Provider](xref:fundamentals/configuration/index#command-line-configuration-provider) requires name-value pairs for command-line arguments, the `--console` switch is removed from the arguments before <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> receives the arguments.</span></span>

<span data-ttu-id="588df-466">Per scrivere nel registro eventi di Windows, aggiungere il provider EventLog a <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>.</span><span class="sxs-lookup"><span data-stu-id="588df-466">To write to the Windows Event Log, add the EventLog provider to <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>.</span></span> <span data-ttu-id="588df-467">Impostare il livello di registrazione con la chiave `Logging:LogLevel:Default` nel file *appsettings.Production.json*.</span><span class="sxs-lookup"><span data-stu-id="588df-467">Set the logging level with the `Logging:LogLevel:Default` key in the *appsettings.Production.json* file.</span></span>

<span data-ttu-id="588df-468">Nel seguente esempio dell'app di esempio, viene chiamato `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per gestire gli eventi di durata all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-468">In the following example from the sample app, `RunAsCustomService` is called instead of <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in order to handle lifetime events within the app.</span></span> <span data-ttu-id="588df-469">Per altre informazioni, vedere la sezione [Gestire gli eventi di avvio e arresto](#handle-starting-and-stopping-events).</span><span class="sxs-lookup"><span data-stu-id="588df-469">For more information, see the [Handle starting and stopping events](#handle-starting-and-stopping-events) section.</span></span>

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a><span data-ttu-id="588df-470">Tipo di distribuzione</span><span class="sxs-lookup"><span data-stu-id="588df-470">Deployment type</span></span>

<span data-ttu-id="588df-471">Per informazioni e consigli sugli scenari di distribuzione, vedere [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/).</span><span class="sxs-lookup"><span data-stu-id="588df-471">For information and advice on deployment scenarios, see [.NET Core application deployment](/dotnet/core/deploying/).</span></span>

### <a name="sdk"></a><span data-ttu-id="588df-472">SDK</span><span class="sxs-lookup"><span data-stu-id="588df-472">SDK</span></span>

<span data-ttu-id="588df-473">Per un servizio basato su app Web che usa le Razor pagine o i framework MVC, specificare Web SDK nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="588df-473">For a web app-based service that uses the Razor Pages or MVC frameworks, specify the Web SDK in the project file:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

<span data-ttu-id="588df-474">Se il servizio esegue solo attività in background, ad esempio [servizi ospitati](xref:fundamentals/host/hosted-services), specificare l'SDK del ruolo di lavoro nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="588df-474">If the service only executes background tasks (for example, [hosted services](xref:fundamentals/host/hosted-services)), specify the Worker SDK in the project file:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a><span data-ttu-id="588df-475">Distribuzione dipendente dal framework</span><span class="sxs-lookup"><span data-stu-id="588df-475">Framework-dependent deployment (FDD)</span></span>

<span data-ttu-id="588df-476">La distribuzione dipendente dal framework si basa sulla presenza di una versione condivisa a livello di sistema di .NET Core nel sistema di destinazione.</span><span class="sxs-lookup"><span data-stu-id="588df-476">Framework-dependent deployment (FDD) relies on the presence of a shared system-wide version of .NET Core on the target system.</span></span> <span data-ttu-id="588df-477">Quando lo scenario di distribuzione dipendente dal framework viene implementato in base alle indicazioni di questo articolo, l'SDK genera un file eseguibile (con estensione *exe*) detto *eseguibile dipendente dal framework*.</span><span class="sxs-lookup"><span data-stu-id="588df-477">When the FDD scenario is adopted following the guidance in this article, the SDK produces an executable (*.exe*), called a *framework-dependent executable*.</span></span>

<span data-ttu-id="588df-478">L'identificatore di Windows [Runtime (RID)](/dotnet/core/rid-catalog) ( [\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier) ) contiene il Framework di destinazione.</span><span class="sxs-lookup"><span data-stu-id="588df-478">The Windows [Runtime Identifier (RID)](/dotnet/core/rid-catalog) ([\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier)) contains the target framework.</span></span> <span data-ttu-id="588df-479">Nell'esempio seguente il RID è impostato su `win7-x64`.</span><span class="sxs-lookup"><span data-stu-id="588df-479">In the following example, the RID is set to `win7-x64`.</span></span> <span data-ttu-id="588df-480">La proprietà `<SelfContained>` è impostata su `false`.</span><span class="sxs-lookup"><span data-stu-id="588df-480">The `<SelfContained>` property is set to `false`.</span></span> <span data-ttu-id="588df-481">Queste proprietà indicano all'SDK di generare un file eseguibile (con estensione *exe*) per Windows e un'app che dipende dal framework .NET Core condiviso.</span><span class="sxs-lookup"><span data-stu-id="588df-481">These properties instruct the SDK to generate an executable (*.exe*) file for Windows and an app that depends on the shared .NET Core framework.</span></span>

<span data-ttu-id="588df-482">La proprietà `<UseAppHost>` è impostata su `true`.</span><span class="sxs-lookup"><span data-stu-id="588df-482">The `<UseAppHost>` property is set to `true`.</span></span> <span data-ttu-id="588df-483">Questa proprietà fornisce il servizio con un percorso di attivazione (un file eseguibile, *.exe*) per una distribuzione dipendente dal framework.</span><span class="sxs-lookup"><span data-stu-id="588df-483">This property provides the service with an activation path (an executable, *.exe*) for an FDD.</span></span>

<span data-ttu-id="588df-484">Un file *web.config*, che viene normalmente generato quando si pubblica un'app ASP.NET Core, non è necessario per un'app di servizi Windows.</span><span class="sxs-lookup"><span data-stu-id="588df-484">A *web.config* file, which is normally produced when publishing an ASP.NET Core app, is unnecessary for a Windows Services app.</span></span> <span data-ttu-id="588df-485">Per disabilitare la creazione del file *web.config*, aggiungere la proprietà `<IsTransformWebConfigDisabled>` impostata su `true`.</span><span class="sxs-lookup"><span data-stu-id="588df-485">To disable the creation of the *web.config* file, add the `<IsTransformWebConfigDisabled>` property set to `true`.</span></span>

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <UseAppHost>true</UseAppHost>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a><span data-ttu-id="588df-486">Distribuzione autonoma</span><span class="sxs-lookup"><span data-stu-id="588df-486">Self-contained deployment (SCD)</span></span>

<span data-ttu-id="588df-487">Una distribuzione autonoma non si basa sulla presenza di un framework condiviso nel sistema host.</span><span class="sxs-lookup"><span data-stu-id="588df-487">Self-contained deployment (SCD) doesn't rely on the presence of a shared framework on the host system.</span></span> <span data-ttu-id="588df-488">Il runtime e le dipendenze dell'app vengono distribuiti con l'app.</span><span class="sxs-lookup"><span data-stu-id="588df-488">The runtime and the app's dependencies are deployed with the app.</span></span>

<span data-ttu-id="588df-489">Un [identificatore di runtime (RID)](/dotnet/core/rid-catalog) di Windows viene incluso nel `<PropertyGroup>` che contiene il framework di destinazione:</span><span class="sxs-lookup"><span data-stu-id="588df-489">A Windows [Runtime Identifier (RID)](/dotnet/core/rid-catalog) is included in the `<PropertyGroup>` that contains the target framework:</span></span>

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

<span data-ttu-id="588df-490">Per eseguire la pubblicazione per più identificatori di runtime:</span><span class="sxs-lookup"><span data-stu-id="588df-490">To publish for multiple RIDs:</span></span>

* <span data-ttu-id="588df-491">Specificare gli identificatori di runtime in un elenco delimitato da punto e virgola.</span><span class="sxs-lookup"><span data-stu-id="588df-491">Provide the RIDs in a semicolon-delimited list.</span></span>
* <span data-ttu-id="588df-492">Usare il nome della proprietà [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plurale).</span><span class="sxs-lookup"><span data-stu-id="588df-492">Use the property name [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plural).</span></span>

<span data-ttu-id="588df-493">Per altre informazioni, vedere il [Catalogo RID di .NET Core](/dotnet/core/rid-catalog).</span><span class="sxs-lookup"><span data-stu-id="588df-493">For more information, see [.NET Core RID Catalog](/dotnet/core/rid-catalog).</span></span>

<span data-ttu-id="588df-494">Una proprietà `<SelfContained>` è impostata su `true`:</span><span class="sxs-lookup"><span data-stu-id="588df-494">A `<SelfContained>` property is set to `true`:</span></span>

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a><span data-ttu-id="588df-495">Account utente del servizio</span><span class="sxs-lookup"><span data-stu-id="588df-495">Service user account</span></span>

<span data-ttu-id="588df-496">Per creare un account utente per un servizio, usare il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) da una shell dei comandi di PowerShell 6 amministrativa.</span><span class="sxs-lookup"><span data-stu-id="588df-496">To create a user account for a service, use the [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet from an administrative PowerShell 6 command shell.</span></span>

<span data-ttu-id="588df-497">In Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763) o versioni successive:</span><span class="sxs-lookup"><span data-stu-id="588df-497">On Windows 10 October 2018 Update (version 1809/build 10.0.17763) or later:</span></span>

```powershell
New-LocalUser -Name {SERVICE NAME}
```

<span data-ttu-id="588df-498">In un sistema operativo Windows precedente ad Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763):</span><span class="sxs-lookup"><span data-stu-id="588df-498">On Windows OS earlier than the Windows 10 October 2018 Update (version 1809/build 10.0.17763):</span></span>

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

<span data-ttu-id="588df-499">Fornire una [password complessa](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) quando richiesto.</span><span class="sxs-lookup"><span data-stu-id="588df-499">Provide a [strong password](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) when prompted.</span></span>

<span data-ttu-id="588df-500">Se non viene specificato il parametro `-AccountExpires` per il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) con un valore <xref:System.DateTime> di scadenza, l'account non scade.</span><span class="sxs-lookup"><span data-stu-id="588df-500">Unless the `-AccountExpires` parameter is supplied to the [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet with an expiration <xref:System.DateTime>, the account doesn't expire.</span></span>

<span data-ttu-id="588df-501">Per altre informazioni, vedere [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) e [Service User Accounts](/windows/desktop/services/service-user-accounts) (Account utente di servizio).</span><span class="sxs-lookup"><span data-stu-id="588df-501">For more information, see [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) and [Service User Accounts](/windows/desktop/services/service-user-accounts).</span></span>

<span data-ttu-id="588df-502">Un approccio alternativo per la gestione degli utenti quando si usa Active Directory consiste nell'usare account del servizio gestito.</span><span class="sxs-lookup"><span data-stu-id="588df-502">An alternative approach to managing users when using Active Directory is to use Managed Service Accounts.</span></span> <span data-ttu-id="588df-503">Per altre informazioni, vedere [Panoramica degli account del servizio gestito del gruppo](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).</span><span class="sxs-lookup"><span data-stu-id="588df-503">For more information, see [Group Managed Service Accounts Overview](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).</span></span>

## <a name="log-on-as-a-service-rights"></a><span data-ttu-id="588df-504">Diritti Accesso come servizio</span><span class="sxs-lookup"><span data-stu-id="588df-504">Log on as a service rights</span></span>

<span data-ttu-id="588df-505">Per stabilire i diritti *Accesso come servizio* per un account utente di servizio:</span><span class="sxs-lookup"><span data-stu-id="588df-505">To establish *Log on as a service* rights for a service user account:</span></span>

1. <span data-ttu-id="588df-506">Aprire l'editor Criteri di sicurezza locali eseguendo *secpol.msc*.</span><span class="sxs-lookup"><span data-stu-id="588df-506">Open the Local Security Policy editor by running *secpol.msc*.</span></span>
1. <span data-ttu-id="588df-507">Espandere il nodo **Criteri locali** e selezionare **Assegnazione diritti utente**.</span><span class="sxs-lookup"><span data-stu-id="588df-507">Expand the **Local Policies** node and select **User Rights Assignment**.</span></span>
1. <span data-ttu-id="588df-508">Aprire il criterio **Accesso come servizio**.</span><span class="sxs-lookup"><span data-stu-id="588df-508">Open the **Log on as a service** policy.</span></span>
1. <span data-ttu-id="588df-509">Selezionare **Aggiungi utente o gruppo**.</span><span class="sxs-lookup"><span data-stu-id="588df-509">Select **Add User or Group**.</span></span>
1. <span data-ttu-id="588df-510">Specificare il nome oggetto (account utente) in uno dei modi seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-510">Provide the object name (user account) using either of the following approaches:</span></span>
   1. <span data-ttu-id="588df-511">Digitare l'account utente (`{DOMAIN OR COMPUTER NAME\USER}`) nel campo del nome oggetto e scegliere **OK** per aggiungere l'utente al criterio.</span><span class="sxs-lookup"><span data-stu-id="588df-511">Type the user account (`{DOMAIN OR COMPUTER NAME\USER}`) in the object name field and select **OK** to add the user to the policy.</span></span>
   1. <span data-ttu-id="588df-512">Fare clic su **Advanced** (Avanzate).</span><span class="sxs-lookup"><span data-stu-id="588df-512">Select **Advanced**.</span></span> <span data-ttu-id="588df-513">Selezionare **Trova**.</span><span class="sxs-lookup"><span data-stu-id="588df-513">Select **Find Now**.</span></span> <span data-ttu-id="588df-514">Selezionare l'account utente dall'elenco.</span><span class="sxs-lookup"><span data-stu-id="588df-514">Select the user account from the list.</span></span> <span data-ttu-id="588df-515">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="588df-515">Select **OK**.</span></span> <span data-ttu-id="588df-516">Scegliere di nuovo **OK** per aggiungere l'utente al criterio.</span><span class="sxs-lookup"><span data-stu-id="588df-516">Select **OK** again to add the user to the policy.</span></span>
1. <span data-ttu-id="588df-517">Scegliere **OK** o **Applica** per accettare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="588df-517">Select **OK** or **Apply** to accept the changes.</span></span>

## <a name="create-and-manage-the-windows-service"></a><span data-ttu-id="588df-518">Creare e gestire il servizio di Windows</span><span class="sxs-lookup"><span data-stu-id="588df-518">Create and manage the Windows Service</span></span>

### <a name="create-a-service"></a><span data-ttu-id="588df-519">Creare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-519">Create a service</span></span>

<span data-ttu-id="588df-520">Usare i comandi di PowerShell per registrare un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-520">Use PowerShell commands to register a service.</span></span> <span data-ttu-id="588df-521">Da una shell dei comandi di PowerShell 6 amministrativa eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-521">From an administrative PowerShell 6 command shell, execute the following commands:</span></span>

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* <span data-ttu-id="588df-522">`{EXE PATH}`: Percorso della cartella dell'app nell'host (ad esempio, `d:\myservice` ).</span><span class="sxs-lookup"><span data-stu-id="588df-522">`{EXE PATH}`: Path to the app's folder on the host (for example, `d:\myservice`).</span></span> <span data-ttu-id="588df-523">Non includere l'eseguibile dell'app nel percorso.</span><span class="sxs-lookup"><span data-stu-id="588df-523">Don't include the app's executable in the path.</span></span> <span data-ttu-id="588df-524">Non è necessario aggiungere una barra finale.</span><span class="sxs-lookup"><span data-stu-id="588df-524">A trailing slash isn't required.</span></span>
* <span data-ttu-id="588df-525">`{DOMAIN OR COMPUTER NAME\USER}`: Account utente del servizio (ad esempio, `Contoso\ServiceUser` ).</span><span class="sxs-lookup"><span data-stu-id="588df-525">`{DOMAIN OR COMPUTER NAME\USER}`: Service user account (for example, `Contoso\ServiceUser`).</span></span>
* <span data-ttu-id="588df-526">`{SERVICE NAME}`: Nome del servizio (ad esempio, `MyService` ).</span><span class="sxs-lookup"><span data-stu-id="588df-526">`{SERVICE NAME}`: Service name (for example, `MyService`).</span></span>
* <span data-ttu-id="588df-527">`{EXE FILE PATH}`: Percorso eseguibile dell'app (ad esempio, `d:\myservice\myservice.exe` ).</span><span class="sxs-lookup"><span data-stu-id="588df-527">`{EXE FILE PATH}`: The app's executable path (for example, `d:\myservice\myservice.exe`).</span></span> <span data-ttu-id="588df-528">Includere il nome del file eseguibile con l'estensione.</span><span class="sxs-lookup"><span data-stu-id="588df-528">Include the executable's file name with extension.</span></span>
* <span data-ttu-id="588df-529">`{DESCRIPTION}`: Descrizione del servizio (ad esempio, `My sample service` ).</span><span class="sxs-lookup"><span data-stu-id="588df-529">`{DESCRIPTION}`: Service description (for example, `My sample service`).</span></span>
* <span data-ttu-id="588df-530">`{DISPLAY NAME}`: Nome visualizzato del servizio (ad esempio, `My Service` ).</span><span class="sxs-lookup"><span data-stu-id="588df-530">`{DISPLAY NAME}`: Service display name (for example, `My Service`).</span></span>

### <a name="start-a-service"></a><span data-ttu-id="588df-531">Avviare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-531">Start a service</span></span>

<span data-ttu-id="588df-532">Per avviare un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-532">Start a service with the following PowerShell 6 command:</span></span>

```powershell
Start-Service -Name {SERVICE NAME}
```

<span data-ttu-id="588df-533">L'avvio del servizio richiede alcuni secondi.</span><span class="sxs-lookup"><span data-stu-id="588df-533">The command takes a few seconds to start the service.</span></span>

### <a name="determine-a-services-status"></a><span data-ttu-id="588df-534">Determinare lo stato di un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-534">Determine a service's status</span></span>

<span data-ttu-id="588df-535">Per verificare lo stato di un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-535">To check the status of a service, use the following PowerShell 6 command:</span></span>

```powershell
Get-Service -Name {SERVICE NAME}
```

<span data-ttu-id="588df-536">Lo stato viene indicato con uno dei valori seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-536">The status is reported as one of the following values:</span></span>

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a><span data-ttu-id="588df-537">Arrestare un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-537">Stop a service</span></span>

<span data-ttu-id="588df-538">Per arrestare un servizio, usare il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-538">Stop a service with the following Powershell 6 command:</span></span>

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a><span data-ttu-id="588df-539">Rimuovere un servizio</span><span class="sxs-lookup"><span data-stu-id="588df-539">Remove a service</span></span>

<span data-ttu-id="588df-540">Dopo il breve lasso di tempo necessario per arrestare un servizio, rimuovere il servizio con il comando di PowerShell 6 seguente:</span><span class="sxs-lookup"><span data-stu-id="588df-540">After a short delay to stop a service, remove a service with the following Powershell 6 command:</span></span>

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a><span data-ttu-id="588df-541">Gestire gli eventi di avvio e arresto</span><span class="sxs-lookup"><span data-stu-id="588df-541">Handle starting and stopping events</span></span>

<span data-ttu-id="588df-542">Per gestire gli eventi <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>, <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*> e <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*>:</span><span class="sxs-lookup"><span data-stu-id="588df-542">To handle <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>, <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*>, and <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*> events:</span></span>

1. <span data-ttu-id="588df-543">Creare una classe che deriva da <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> con i metodi `OnStarting`, `OnStarted` e `OnStopping`:</span><span class="sxs-lookup"><span data-stu-id="588df-543">Create a class that derives from <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> with the `OnStarting`, `OnStarted`, and `OnStopping` methods:</span></span>

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. <span data-ttu-id="588df-544">Creare un metodo di estensione per <xref:Microsoft.AspNetCore.Hosting.IWebHost> che passa `CustomWebHostService` a <xref:System.ServiceProcess.ServiceBase.Run*>:</span><span class="sxs-lookup"><span data-stu-id="588df-544">Create an extension method for <xref:Microsoft.AspNetCore.Hosting.IWebHost> that passes the `CustomWebHostService` to <xref:System.ServiceProcess.ServiceBase.Run*>:</span></span>

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. <span data-ttu-id="588df-545">In `Program.Main` chiamare il metodo di estensione `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>:</span><span class="sxs-lookup"><span data-stu-id="588df-545">In `Program.Main`, call the `RunAsCustomService` extension method instead of <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>:</span></span>

   ```csharp
   host.RunAsCustomService();
   ```

   <span data-ttu-id="588df-546">Per visualizzare il percorso di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in `Program.Main`, fare riferimento all'esempio di codice illustrato nella sezione [Tipo di distribuzione](#deployment-type).</span><span class="sxs-lookup"><span data-stu-id="588df-546">To see the location of <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in `Program.Main`, refer to the code sample shown in the [Deployment type](#deployment-type) section.</span></span>

## <a name="proxy-server-and-load-balancer-scenarios"></a><span data-ttu-id="588df-547">Scenari con server proxy e servizi di bilanciamento del carico</span><span class="sxs-lookup"><span data-stu-id="588df-547">Proxy server and load balancer scenarios</span></span>

<span data-ttu-id="588df-548">I servizi che interagiscono con le richieste da Internet o da una rete aziendale e si trovano dietro un proxy o un servizio di bilanciamento del carico potrebbero richiedere una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="588df-548">Services that interact with requests from the Internet or a corporate network and are behind a proxy or load balancer might require additional configuration.</span></span> <span data-ttu-id="588df-549">Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.</span><span class="sxs-lookup"><span data-stu-id="588df-549">For more information, see <xref:host-and-deploy/proxy-load-balancer>.</span></span>

## <a name="configure-endpoints"></a><span data-ttu-id="588df-550">Configurare gli endpoint</span><span class="sxs-lookup"><span data-stu-id="588df-550">Configure endpoints</span></span>

<span data-ttu-id="588df-551">Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`.</span><span class="sxs-lookup"><span data-stu-id="588df-551">By default, ASP.NET Core binds to `http://localhost:5000`.</span></span> <span data-ttu-id="588df-552">Configurare l'URL e la porta impostando la `ASPNETCORE_URLS` variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="588df-552">Configure the URL and port by setting the `ASPNETCORE_URLS` environment variable.</span></span>

<span data-ttu-id="588df-553">Per ulteriori approcci alla configurazione di porte e URL, vedere l'articolo relativo al server pertinente:</span><span class="sxs-lookup"><span data-stu-id="588df-553">For additional URL and port configuration approaches, see the relevant server article:</span></span>

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

<span data-ttu-id="588df-554">Le linee guida precedenti riguardano il supporto per gli endpoint HTTPS.</span><span class="sxs-lookup"><span data-stu-id="588df-554">The preceding guidance covers support for HTTPS endpoints.</span></span> <span data-ttu-id="588df-555">Ad esempio, configurare l'app per HTTPS quando si usa l'autenticazione con un servizio Windows.</span><span class="sxs-lookup"><span data-stu-id="588df-555">For example, configure the app for HTTPS when authentication is used with a Windows Service.</span></span>

> [!NOTE]
> <span data-ttu-id="588df-556">L'uso del certificato di sviluppo ASP.NET Core HTTPS per proteggere un endpoint del servizio non è supportato.</span><span class="sxs-lookup"><span data-stu-id="588df-556">Use of the ASP.NET Core HTTPS development certificate to secure a service endpoint isn't supported.</span></span>

## <a name="current-directory-and-content-root"></a><span data-ttu-id="588df-557">Directory corrente e radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="588df-557">Current directory and content root</span></span>

<span data-ttu-id="588df-558">La directory di lavoro corrente restituita chiamando <xref:System.IO.Directory.GetCurrentDirectory*> per un servizio Windows è la cartella *C:\\WINDOWS\\system32*.</span><span class="sxs-lookup"><span data-stu-id="588df-558">The current working directory returned by calling <xref:System.IO.Directory.GetCurrentDirectory*> for a Windows Service is the *C:\\WINDOWS\\system32* folder.</span></span> <span data-ttu-id="588df-559">La cartella *system32* non è un percorso appropriato per archiviare i file di un servizio, ad esempio i file di impostazioni.</span><span class="sxs-lookup"><span data-stu-id="588df-559">The *system32* folder isn't a suitable location to store a service's files (for example, settings files).</span></span> <span data-ttu-id="588df-560">Usare uno degli approcci seguenti per gestire e accedere agli asset e ai file di impostazioni di un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-560">Use one of the following approaches to maintain and access a service's assets and settings files.</span></span>

### <a name="set-the-content-root-path-to-the-apps-folder"></a><span data-ttu-id="588df-561">Impostare il percorso radice del contenuto sulla cartella dell'app</span><span class="sxs-lookup"><span data-stu-id="588df-561">Set the content root path to the app's folder</span></span>

<span data-ttu-id="588df-562"><xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> è lo stesso percorso fornito all'argomento `binPath` durante la creazione di un servizio.</span><span class="sxs-lookup"><span data-stu-id="588df-562">The <xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> is the same path provided to the `binPath` argument when a service is created.</span></span> <span data-ttu-id="588df-563">Anziché chiamare `GetCurrentDirectory` per creare percorsi per i file di impostazioni, chiamare <xref:System.IO.Directory.SetCurrentDirectory*> con il percorso della radice del [contenuto](xref:fundamentals/index#content-root)dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-563">Instead of calling `GetCurrentDirectory` to create paths to settings files, call <xref:System.IO.Directory.SetCurrentDirectory*> with the path to the app's [content root](xref:fundamentals/index#content-root).</span></span>

<span data-ttu-id="588df-564">In `Program.Main`, determinare il percorso della cartella dell'eseguibile del servizio e usare il percorso per stabilire la radice del contenuto dell'app:</span><span class="sxs-lookup"><span data-stu-id="588df-564">In `Program.Main`, determine the path to the folder of the service's executable and use the path to establish the app's content root:</span></span>

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a><span data-ttu-id="588df-565">Archiviare i file di un servizio in un percorso appropriato nel disco</span><span class="sxs-lookup"><span data-stu-id="588df-565">Store a service's files in a suitable location on disk</span></span>

<span data-ttu-id="588df-566">Specificare un percorso assoluto con <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> quando si usa un <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per la cartella contenente i file.</span><span class="sxs-lookup"><span data-stu-id="588df-566">Specify an absolute path with <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> when using an <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> to the folder containing the files.</span></span>

## <a name="troubleshoot"></a><span data-ttu-id="588df-567">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="588df-567">Troubleshoot</span></span>

<span data-ttu-id="588df-568">Per risolvere i problemi relativi a un'app di servizio Windows, vedere <xref:test/troubleshoot> .</span><span class="sxs-lookup"><span data-stu-id="588df-568">To troubleshoot a Windows Service app, see <xref:test/troubleshoot>.</span></span>

### <a name="common-errors"></a><span data-ttu-id="588df-569">Errori comuni</span><span class="sxs-lookup"><span data-stu-id="588df-569">Common errors</span></span>

* <span data-ttu-id="588df-570">È in uso una versione precedente o provvisoria di PowerShell.</span><span class="sxs-lookup"><span data-stu-id="588df-570">An old or pre-release version of PowerShell is in use.</span></span>
* <span data-ttu-id="588df-571">Il servizio registrato non usa l'output **pubblicato** dell'app dal comando [DotNet Publish](/dotnet/core/tools/dotnet-publish) .</span><span class="sxs-lookup"><span data-stu-id="588df-571">The registered service doesn't use the app's **published** output from the [dotnet publish](/dotnet/core/tools/dotnet-publish) command.</span></span> <span data-ttu-id="588df-572">L'output del comando [DotNet Build](/dotnet/core/tools/dotnet-build) non è supportato per la distribuzione di app.</span><span class="sxs-lookup"><span data-stu-id="588df-572">Output of the [dotnet build](/dotnet/core/tools/dotnet-build) command isn't supported for app deployment.</span></span> <span data-ttu-id="588df-573">Gli asset pubblicati si trovano in una delle cartelle seguenti, a seconda del tipo di distribuzione:</span><span class="sxs-lookup"><span data-stu-id="588df-573">Published assets are found in either of the following folders depending on the deployment type:</span></span>
  * <span data-ttu-id="588df-574">*bin/Release/{Target Framework}/Publish* (FDD)</span><span class="sxs-lookup"><span data-stu-id="588df-574">*bin/Release/{TARGET FRAMEWORK}/publish* (FDD)</span></span>
  * <span data-ttu-id="588df-575">*bin/Release/{Target Framework}/{Runtime Identifier}/Publish* (SCD)</span><span class="sxs-lookup"><span data-stu-id="588df-575">*bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)</span></span>
* <span data-ttu-id="588df-576">Il servizio non è nello stato in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="588df-576">The service isn't in the RUNNING state.</span></span>
* <span data-ttu-id="588df-577">I percorsi delle risorse utilizzate dall'app, ad esempio i certificati, non sono corretti.</span><span class="sxs-lookup"><span data-stu-id="588df-577">The paths to resources that the app uses (for example, certificates) are incorrect.</span></span> <span data-ttu-id="588df-578">Il percorso di base di un servizio Windows è *c: \\ Windows \\ system32*.</span><span class="sxs-lookup"><span data-stu-id="588df-578">The base path of a Windows Service is *c:\\Windows\\System32*.</span></span>
* <span data-ttu-id="588df-579">L'utente non dispone *di diritti di accesso come servizio* .</span><span class="sxs-lookup"><span data-stu-id="588df-579">The user doesn't have *Log on as a service* rights.</span></span>
* <span data-ttu-id="588df-580">La password dell'utente è scaduta o non è stata passata correttamente durante l'esecuzione del `New-Service` comando di PowerShell.</span><span class="sxs-lookup"><span data-stu-id="588df-580">The user's password is expired or incorrectly passed when executing the `New-Service` PowerShell command.</span></span>
* <span data-ttu-id="588df-581">L'app richiede l'autenticazione ASP.NET Core ma non è configurata per le connessioni protette (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="588df-581">The app requires ASP.NET Core authentication but isn't configured for secure connections (HTTPS).</span></span>
* <span data-ttu-id="588df-582">La porta dell'URL della richiesta non è corretta o non è configurata correttamente nell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-582">The request URL port is incorrect or not configured correctly in the app.</span></span>

### <a name="system-and-application-event-logs"></a><span data-ttu-id="588df-583">Log eventi di sistema e dell'applicazione</span><span class="sxs-lookup"><span data-stu-id="588df-583">System and Application Event Logs</span></span>

<span data-ttu-id="588df-584">Accedere ai registri eventi di sistema e dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="588df-584">Access the System and Application Event Logs:</span></span>

1. <span data-ttu-id="588df-585">Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .</span><span class="sxs-lookup"><span data-stu-id="588df-585">Open the Start menu, search for *Event Viewer*, and select the **Event Viewer** app.</span></span>
1. <span data-ttu-id="588df-586">In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.</span><span class="sxs-lookup"><span data-stu-id="588df-586">In **Event Viewer**, open the **Windows Logs** node.</span></span>
1. <span data-ttu-id="588df-587">Selezionare **sistema** per aprire il registro eventi di sistema.</span><span class="sxs-lookup"><span data-stu-id="588df-587">Select **System** to open the System Event Log.</span></span> <span data-ttu-id="588df-588">Selezionare **Applicazione** per aprire il log eventi dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="588df-588">Select **Application** to open the Application Event Log.</span></span>
1. <span data-ttu-id="588df-589">Cercare gli errori associati all'app in cui si è verificato il problema.</span><span class="sxs-lookup"><span data-stu-id="588df-589">Search for errors associated with the failing app.</span></span>

### <a name="run-the-app-at-a-command-prompt"></a><span data-ttu-id="588df-590">Eseguire l'app da un prompt dei comandi</span><span class="sxs-lookup"><span data-stu-id="588df-590">Run the app at a command prompt</span></span>

<span data-ttu-id="588df-591">Molti errori di avvio non producono informazioni utili nei log eventi.</span><span class="sxs-lookup"><span data-stu-id="588df-591">Many startup errors don't produce useful information in the event logs.</span></span> <span data-ttu-id="588df-592">È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host.</span><span class="sxs-lookup"><span data-stu-id="588df-592">You can find the cause of some errors by running the app at a command prompt on the hosting system.</span></span> <span data-ttu-id="588df-593">Per registrare dettagli aggiuntivi dall'app, abbassare il [livello di registrazione](xref:fundamentals/logging/index#log-level) o eseguire l'app nell' [ambiente di sviluppo](xref:fundamentals/environments).</span><span class="sxs-lookup"><span data-stu-id="588df-593">To log additional detail from the app, lower the [log level](xref:fundamentals/logging/index#log-level) or run the app in the [Development environment](xref:fundamentals/environments).</span></span>

### <a name="clear-package-caches"></a><span data-ttu-id="588df-594">Cancella cache di pacchetti</span><span class="sxs-lookup"><span data-stu-id="588df-594">Clear package caches</span></span>

<span data-ttu-id="588df-595">Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="588df-595">A functioning app may fail immediately after upgrading either the .NET Core SDK on the development machine or changing package versions within the app.</span></span> <span data-ttu-id="588df-596">In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali.</span><span class="sxs-lookup"><span data-stu-id="588df-596">In some cases, incoherent packages may break an app when performing major upgrades.</span></span> <span data-ttu-id="588df-597">La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="588df-597">Most of these issues can be fixed by following these instructions:</span></span>

1. <span data-ttu-id="588df-598">Eliminare le cartelle *bin* e *obj*.</span><span class="sxs-lookup"><span data-stu-id="588df-598">Delete the *bin* and *obj* folders.</span></span>
1. <span data-ttu-id="588df-599">Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="588df-599">Clear the package caches by executing [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) from a command shell.</span></span>

   <span data-ttu-id="588df-600">La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` .</span><span class="sxs-lookup"><span data-stu-id="588df-600">Clearing package caches can also be accomplished with the [nuget.exe](https://www.nuget.org/downloads) tool and executing the command `nuget locals all -clear`.</span></span> <span data-ttu-id="588df-601">*nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).</span><span class="sxs-lookup"><span data-stu-id="588df-601">*nuget.exe* isn't a bundled install with the Windows desktop operating system and must be obtained separately from the [NuGet website](https://www.nuget.org/downloads).</span></span>

1. <span data-ttu-id="588df-602">Ripristinare e ricompilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="588df-602">Restore and rebuild the project.</span></span>
1. <span data-ttu-id="588df-603">Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.</span><span class="sxs-lookup"><span data-stu-id="588df-603">Delete all of the files in the deployment folder on the server prior to redeploying the app.</span></span>

### <a name="slow-or-hanging-app"></a><span data-ttu-id="588df-604">App lenta o bloccata</span><span class="sxs-lookup"><span data-stu-id="588df-604">Slow or hanging app</span></span>

<span data-ttu-id="588df-605">Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.</span><span class="sxs-lookup"><span data-stu-id="588df-605">A *crash dump* is a snapshot of the system's memory and can help determine the cause of an app crash, startup failure, or slow app.</span></span>

#### <a name="app-crashes-or-encounters-an-exception"></a><span data-ttu-id="588df-606">Arresto anomalo o eccezione di un'app</span><span class="sxs-lookup"><span data-stu-id="588df-606">App crashes or encounters an exception</span></span>

<span data-ttu-id="588df-607">Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):</span><span class="sxs-lookup"><span data-stu-id="588df-607">Obtain and analyze a dump from [Windows Error Reporting (WER)](/windows/desktop/wer/windows-error-reporting):</span></span>

1. <span data-ttu-id="588df-608">Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`.</span><span class="sxs-lookup"><span data-stu-id="588df-608">Create a folder to hold crash dump files at `c:\dumps`.</span></span>
1. <span data-ttu-id="588df-609">Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1) con il nome dell'eseguibile dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="588df-609">Run the [EnableDumps PowerShell script](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1) with the application executable name:</span></span>

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. <span data-ttu-id="588df-610">Eseguire l'app nelle condizioni che causano l'arresto anomalo.</span><span class="sxs-lookup"><span data-stu-id="588df-610">Run the app under the conditions that cause the crash to occur.</span></span>
1. <span data-ttu-id="588df-611">Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1):</span><span class="sxs-lookup"><span data-stu-id="588df-611">After the crash has occurred, run the [DisableDumps PowerShell script](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1):</span></span>

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

<span data-ttu-id="588df-612">Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente.</span><span class="sxs-lookup"><span data-stu-id="588df-612">After an app crashes and dump collection is complete, the app is allowed to terminate normally.</span></span> <span data-ttu-id="588df-613">Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.</span><span class="sxs-lookup"><span data-stu-id="588df-613">The PowerShell script configures WER to collect up to five dumps per app.</span></span>

> [!WARNING]
> <span data-ttu-id="588df-614">I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).</span><span class="sxs-lookup"><span data-stu-id="588df-614">Crash dumps might take up a large amount of disk space (up to several gigabytes each).</span></span>

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a><span data-ttu-id="588df-615">L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente</span><span class="sxs-lookup"><span data-stu-id="588df-615">App hangs, fails during startup, or runs normally</span></span>

<span data-ttu-id="588df-616">Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.</span><span class="sxs-lookup"><span data-stu-id="588df-616">When an app *hangs* (stops responding but doesn't crash), fails during startup, or runs normally, see [User-Mode Dump Files: Choosing the Best Tool](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) to select an appropriate tool to produce the dump.</span></span>

#### <a name="analyze-the-dump"></a><span data-ttu-id="588df-617">Analizzare il dump</span><span class="sxs-lookup"><span data-stu-id="588df-617">Analyze the dump</span></span>

<span data-ttu-id="588df-618">È possibile analizzare un dump usando diversi approcci.</span><span class="sxs-lookup"><span data-stu-id="588df-618">A dump can be analyzed using several approaches.</span></span> <span data-ttu-id="588df-619">Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).</span><span class="sxs-lookup"><span data-stu-id="588df-619">For more information, see [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="588df-620">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="588df-620">Additional resources</span></span>

* <span data-ttu-id="588df-621">[Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel#endpoint-configuration) (include la configurazione HTTPS e il supporto SNI)</span><span class="sxs-lookup"><span data-stu-id="588df-621">[Kestrel endpoint configuration](xref:fundamentals/servers/kestrel#endpoint-configuration) (includes HTTPS configuration and SNI support)</span></span>
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end
