---
title: Configurazione avanzata
author: rick-anderson
description: Configurazione avanzata con il modulo ASP.NET Core e il Internet Information Services (IIS).
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 5/7/2020
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
uid: host-and-deploy/iis/advanced
ms.openlocfilehash: de3abb7b5f89e3b84da38a8eabb183f9551cc1f9
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588919"
---
# <a name="advanced-configuration-of-the-aspnet-core-module-and-iis"></a><span data-ttu-id="d687b-103">Configurazione avanzata del modulo ASP.NET Core e di IIS</span><span class="sxs-lookup"><span data-stu-id="d687b-103">Advanced configuration of the ASP.NET Core Module and IIS</span></span>

<span data-ttu-id="d687b-104">Questo articolo illustra le opzioni di configurazione avanzate e gli scenari per il modulo ASP.NET Core e IIS.</span><span class="sxs-lookup"><span data-stu-id="d687b-104">This article covers advanced configuration options and scenarios for the ASP.NET Core Module and IIS.</span></span>

## <a name="modify-the-stack-size"></a><span data-ttu-id="d687b-105">Modificare le dimensioni dello stack</span><span class="sxs-lookup"><span data-stu-id="d687b-105">Modify the stack size</span></span>

<span data-ttu-id="d687b-106">*Si applica solo quando si usa il modello di hosting in-process.*</span><span class="sxs-lookup"><span data-stu-id="d687b-106">*Only applies when using the in-process hosting model.*</span></span>

<span data-ttu-id="d687b-107">Configurare la dimensione dello stack gestito usando l' `stackSize` impostazione in byte nel `web.config` file.</span><span class="sxs-lookup"><span data-stu-id="d687b-107">Configure the managed stack size using the `stackSize` setting in bytes in the `web.config` file.</span></span> <span data-ttu-id="d687b-108">Le dimensioni predefinite sono pari a 1.048.576 byte (1 MB).</span><span class="sxs-lookup"><span data-stu-id="d687b-108">The default size is 1,048,576 bytes (1 MB).</span></span> <span data-ttu-id="d687b-109">Nell'esempio seguente le dimensioni dello stack vengono modificate in 2 MB (2.097.152 byte):</span><span class="sxs-lookup"><span data-stu-id="d687b-109">The following example changes the stack size to 2 MB (2,097,152 bytes):</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="stackSize" value="2097152" />
  </handlerSettings>
</aspNetCore>
```

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a><span data-ttu-id="d687b-110">La configurazione del proxy usa il protocollo HTTP e un token di associazione</span><span class="sxs-lookup"><span data-stu-id="d687b-110">Proxy configuration uses HTTP protocol and a pairing token</span></span>

<span data-ttu-id="d687b-111">*Si applica solo all'hosting out-of-process.*</span><span class="sxs-lookup"><span data-stu-id="d687b-111">*Only applies to out-of-process hosting.*</span></span>

<span data-ttu-id="d687b-112">Il proxy creato tra il modulo ASP.NET Core e Kestrel usa il protocollo HTTP.</span><span class="sxs-lookup"><span data-stu-id="d687b-112">The proxy created between the ASP.NET Core Module and Kestrel uses the HTTP protocol.</span></span> <span data-ttu-id="d687b-113">Non vi è alcun rischio di intercettazione del traffico tra il modulo e Kestrel da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="d687b-113">There's no risk of eavesdropping the traffic between the module and Kestrel from a location off of the server.</span></span>

<span data-ttu-id="d687b-114">Viene usato un token di associazione per garantire che le richieste ricevute da Kestrel siano state trasmesse tramite proxy da IIS e non provengano da un'altra origine.</span><span class="sxs-lookup"><span data-stu-id="d687b-114">A pairing token is used to guarantee that the requests received by Kestrel were proxied by IIS and didn't come from some other source.</span></span> <span data-ttu-id="d687b-115">Il token di associazione viene creato e impostato in una variabile di ambiente (`ASPNETCORE_TOKEN`) dal modulo.</span><span class="sxs-lookup"><span data-stu-id="d687b-115">The pairing token is created and set into an environment variable (`ASPNETCORE_TOKEN`) by the module.</span></span> <span data-ttu-id="d687b-116">Il token di associazione viene impostato anche in un'intestazione (`MS-ASPNETCORE-TOKEN`) in tutte le richieste trasmesse tramite proxy.</span><span class="sxs-lookup"><span data-stu-id="d687b-116">The pairing token is also set into a header (`MS-ASPNETCORE-TOKEN`) on every proxied request.</span></span> <span data-ttu-id="d687b-117">Il middleware di IIS controlla ogni richiesta che riceve in modo da confermare che il valore dell'intestazione del token di associazione corrisponda al valore della variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="d687b-117">IIS Middleware checks each request it receives to confirm that the pairing token header value matches the environment variable value.</span></span> <span data-ttu-id="d687b-118">Se i valori del token non corrispondono, la richiesta viene registrata e rifiutata.</span><span class="sxs-lookup"><span data-stu-id="d687b-118">If the token values are mismatched, the request is logged and rejected.</span></span> <span data-ttu-id="d687b-119">La variabile di ambiente del token di associazione e il traffico tra il modulo e Kestrel non sono accessibili da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="d687b-119">The pairing token environment variable and the traffic between the module and Kestrel aren't accessible from a location off of the server.</span></span> <span data-ttu-id="d687b-120">Senza conoscere il valore del token di associazione, un utente malintenzionato non può inviare richieste che ignorano il controllo nel middleware di IIS.</span><span class="sxs-lookup"><span data-stu-id="d687b-120">Without knowing the pairing token value, an attacker can't submit requests that bypass the check in the IIS Middleware.</span></span>

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a><span data-ttu-id="d687b-121">Modulo ASP.NET Core con configurazione condivisa di IIS</span><span class="sxs-lookup"><span data-stu-id="d687b-121">ASP.NET Core Module with an IIS Shared Configuration</span></span>

<span data-ttu-id="d687b-122">Il programma di installazione del modulo ASP.NET Core viene eseguito con i privilegi dell' `TrustedInstaller` account.</span><span class="sxs-lookup"><span data-stu-id="d687b-122">The ASP.NET Core Module installer runs with the privileges of the `TrustedInstaller` account.</span></span> <span data-ttu-id="d687b-123">Poiché l'account di sistema locale non dispone dell'autorizzazione di modifica per il percorso di condivisione utilizzato dalla configurazione condivisa di IIS, il programma di installazione genera un errore di accesso negato quando si tenta di configurare le impostazioni del modulo nel `applicationHost.config` file nella condivisione.</span><span class="sxs-lookup"><span data-stu-id="d687b-123">Because the local system account doesn't have modify permission for the share path used by the IIS Shared Configuration, the installer throws an access denied error when attempting to configure the module settings in the `applicationHost.config` file on the share.</span></span>

<span data-ttu-id="d687b-124">Quando si usa una configurazione condivisa di IIS nello stesso computer dell'installazione di IIS, eseguire il programma di installazione del bundle di hosting ASP.NET Core con il parametro `OPT_NO_SHARED_CONFIG_CHECK` impostato su `1`:</span><span class="sxs-lookup"><span data-stu-id="d687b-124">When using an IIS Shared Configuration on the same machine as the IIS installation, run the ASP.NET Core Hosting Bundle installer with the `OPT_NO_SHARED_CONFIG_CHECK` parameter set to `1`:</span></span>

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

<span data-ttu-id="d687b-125">Quando il percorso della configurazione condivisa non è nello stesso computer dell'installazione di IIS, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="d687b-125">When the path to the shared configuration isn't on the same machine as the IIS installation, follow these steps:</span></span>

1. <span data-ttu-id="d687b-126">Disabilitare la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="d687b-126">Disable the IIS Shared Configuration.</span></span>
1. <span data-ttu-id="d687b-127">Eseguire il programma di installazione.</span><span class="sxs-lookup"><span data-stu-id="d687b-127">Run the installer.</span></span>
1. <span data-ttu-id="d687b-128">Esportare il file aggiornato nella `applicationHost.config` condivisione file.</span><span class="sxs-lookup"><span data-stu-id="d687b-128">Export the updated `applicationHost.config` file to the file share.</span></span>
1. <span data-ttu-id="d687b-129">Abilitare di nuovo la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="d687b-129">Re-enable the IIS Shared Configuration.</span></span>

## <a name="data-protection"></a><span data-ttu-id="d687b-130">Protezione dei dati</span><span class="sxs-lookup"><span data-stu-id="d687b-130">Data protection</span></span>

<span data-ttu-id="d687b-131">Lo [stack di protezione dei dati di ASP.NET Core](xref:security/data-protection/introduction) viene usato da diversi [middleware](xref:fundamentals/middleware/index) di ASP.NET Core, inclusi quelli usati nell'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d687b-131">The [ASP.NET Core Data Protection stack](xref:security/data-protection/introduction) is used by several ASP.NET Core [middlewares](xref:fundamentals/middleware/index), including middleware used in authentication.</span></span> <span data-ttu-id="d687b-132">Anche se le DPAPI (Data Protection API) non vengono chiamate dal codice dell'utente, è consigliabile configurare la protezione dati per la creazione di un [archivio di chiavi](xref:security/data-protection/implementation/key-management) crittografiche permanente con uno script di distribuzione o nel codice dell'utente.</span><span class="sxs-lookup"><span data-stu-id="d687b-132">Even if Data Protection APIs aren't called by user code, data protection should be configured with a deployment script or in user code to create a persistent cryptographic [key store](xref:security/data-protection/implementation/key-management).</span></span> <span data-ttu-id="d687b-133">Se non si configura la protezione dei dati, le chiavi vengono mantenute in memoria ed eliminate al riavvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="d687b-133">If data protection isn't configured, the keys are held in memory and discarded when the app restarts.</span></span>

<span data-ttu-id="d687b-134">Se l'anello della chiave di protezione dei dati viene archiviato in memoria quando l'app viene riavviata:</span><span class="sxs-lookup"><span data-stu-id="d687b-134">If the Data Protection key ring is stored in memory when the app restarts:</span></span>

* <span data-ttu-id="d687b-135">Tutti i cookie token di autenticazione basati su vengono invalidati.</span><span class="sxs-lookup"><span data-stu-id="d687b-135">All cookie-based authentication tokens are invalidated.</span></span> 
* <span data-ttu-id="d687b-136">Gli utenti devono ripetere l'accesso alla richiesta successiva.</span><span class="sxs-lookup"><span data-stu-id="d687b-136">Users are required to sign in again on their next request.</span></span> 
* <span data-ttu-id="d687b-137">Tutti i dati protetti con il gruppo di chiavi non possono più essere decrittografati.</span><span class="sxs-lookup"><span data-stu-id="d687b-137">Any data protected with the key ring can no longer be decrypted.</span></span> <span data-ttu-id="d687b-138">Questo può includere [i token CSRF](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) e [ASP.NET Core TempData MVC cookie ](xref:fundamentals/app-state#tempdata).</span><span class="sxs-lookup"><span data-stu-id="d687b-138">This may include [CSRF tokens](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) and [ASP.NET Core MVC TempData cookies](xref:fundamentals/app-state#tempdata).</span></span>

<span data-ttu-id="d687b-139">Per configurare la protezione dati in IIS in modo da rendere permanente il gruppo di chiavi, usare **uno** degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="d687b-139">To configure data protection under IIS to persist the key ring, use **one** of the following approaches:</span></span>

* <span data-ttu-id="d687b-140">**Creare chiavi del registro di sistema per la protezione dei dati**</span><span class="sxs-lookup"><span data-stu-id="d687b-140">**Create Data Protection Registry keys**</span></span>

  <span data-ttu-id="d687b-141">Le chiavi di protezione dei dati usate dalle app ASP.NET Core vengono archiviate nel registro di sistema esterno alle app.</span><span class="sxs-lookup"><span data-stu-id="d687b-141">Data Protection keys used by ASP.NET Core apps are stored in the registry external to the apps.</span></span> <span data-ttu-id="d687b-142">Per salvare in modo permanente le chiavi per una determinata app, creare le chiavi del registro di sistema per il pool di app.</span><span class="sxs-lookup"><span data-stu-id="d687b-142">To persist the keys for a given app, create Registry keys for the app pool.</span></span>

  <span data-ttu-id="d687b-143">Per le installazioni IIS autonome non web farm è possibile usare lo [script PowerShell Data Protection Provision-AutoGenKeys.ps1 (Provisioning di protezione dati-AutoGenKeys.ps1)](https://github.com/dotnet/AspNetCore/blob/main/src/DataProtection/Provision-AutoGenKeys.ps1) per ogni pool di app usato con un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d687b-143">For standalone, non-webfarm IIS installations, the [Data Protection Provision-AutoGenKeys.ps1 PowerShell script](https://github.com/dotnet/AspNetCore/blob/main/src/DataProtection/Provision-AutoGenKeys.ps1) can be used for each app pool used with an ASP.NET Core app.</span></span> <span data-ttu-id="d687b-144">Questo script crea una chiave del registro di sistema nel registro di sistema HKLM accessibile solo all'account del processo di lavoro del pool di applicazioni dell'app.</span><span class="sxs-lookup"><span data-stu-id="d687b-144">This script creates a Registry key in the HKLM registry that's accessible only to the worker process account of the app's app pool.</span></span> <span data-ttu-id="d687b-145">Le chiavi vengono crittografate a riposo tramite DPAPI con una chiave a livello del computer.</span><span class="sxs-lookup"><span data-stu-id="d687b-145">Keys are encrypted at rest using DPAPI with a machine-wide key.</span></span>

  <span data-ttu-id="d687b-146">In Web farm scenari, un'app può essere configurata in modo da usare un percorso UNC per archiviare il relativo anello chiave di protezione dati.</span><span class="sxs-lookup"><span data-stu-id="d687b-146">In web farm scenarios, an app can be configured to use a UNC path to store its Data Protection key ring.</span></span> <span data-ttu-id="d687b-147">Per impostazione predefinita, le chiavi non sono crittografate.</span><span class="sxs-lookup"><span data-stu-id="d687b-147">By default, the keys aren't encrypted.</span></span> <span data-ttu-id="d687b-148">Assicurarsi che le autorizzazioni per i file per la condivisione di rete siano limitate all'account di Windows con cui viene eseguita l'app.</span><span class="sxs-lookup"><span data-stu-id="d687b-148">Ensure that the file permissions for the network share are limited to the Windows account that the app runs under.</span></span> <span data-ttu-id="d687b-149">È possibile usare un certificato X509 per la protezione delle chiavi inattive.</span><span class="sxs-lookup"><span data-stu-id="d687b-149">An X509 certificate can be used to protect keys at rest.</span></span> <span data-ttu-id="d687b-150">Si consideri un meccanismo per consentire agli utenti di caricare i certificati.</span><span class="sxs-lookup"><span data-stu-id="d687b-150">Consider a mechanism to allow users to upload certificates.</span></span> <span data-ttu-id="d687b-151">Posizionare i certificati nell'archivio certificati attendibili dell'utente e assicurarsi che siano disponibili in tutti i computer in cui viene eseguita l'app dell'utente.</span><span class="sxs-lookup"><span data-stu-id="d687b-151">Place certificates into the user's trusted certificate store and ensure they're available on all machines where the user's app runs.</span></span> <span data-ttu-id="d687b-152">Per altre informazioni, vedere <xref:security/data-protection/configuration/overview>.</span><span class="sxs-lookup"><span data-stu-id="d687b-152">For more information, see <xref:security/data-protection/configuration/overview>.</span></span>

* <span data-ttu-id="d687b-153">**Configurare il pool di applicazioni IIS per caricare il profilo utente**</span><span class="sxs-lookup"><span data-stu-id="d687b-153">**Configure the IIS Application Pool to load the user profile**</span></span>

  <span data-ttu-id="d687b-154">Questa impostazione si trova nella sezione **Modello del processo** in **Impostazioni avanzate** per il pool di app.</span><span class="sxs-lookup"><span data-stu-id="d687b-154">This setting is in the **Process Model** section under the **Advanced Settings** for the app pool.</span></span> <span data-ttu-id="d687b-155">Impostare **Caricamento del profilo utente** su `True`.</span><span class="sxs-lookup"><span data-stu-id="d687b-155">Set **Load User Profile** to `True`.</span></span> <span data-ttu-id="d687b-156">Con l'impostazione `True` le chiavi vengono archiviate nella directory del profilo utente e protette tramite DPAPI con una chiave specifica per l'account utente.</span><span class="sxs-lookup"><span data-stu-id="d687b-156">When set to `True`, keys are stored in the user profile directory and protected using DPAPI with a key specific to the user account.</span></span> <span data-ttu-id="d687b-157">Le chiavi vengono salvate in modo permanente nella `%LOCALAPPDATA%/ASP.NET/DataProtection-Keys` cartella.</span><span class="sxs-lookup"><span data-stu-id="d687b-157">Keys are persisted to the `%LOCALAPPDATA%/ASP.NET/DataProtection-Keys` folder.</span></span>

  <span data-ttu-id="d687b-158">È necessario abilitare anche l' [ `setProfileEnvironment` attributo](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration) del pool di app.</span><span class="sxs-lookup"><span data-stu-id="d687b-158">The app pool's [`setProfileEnvironment` attribute](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration) must also be enabled.</span></span> <span data-ttu-id="d687b-159">Il valore predefinito di `setProfileEnvironment` è `true`.</span><span class="sxs-lookup"><span data-stu-id="d687b-159">The default value of `setProfileEnvironment` is `true`.</span></span> <span data-ttu-id="d687b-160">In alcuni scenari (ad esempio, per il sistema operativo Windows), `setProfileEnvironment` è impostato su `false`.</span><span class="sxs-lookup"><span data-stu-id="d687b-160">In some scenarios (for example, Windows OS), `setProfileEnvironment` is set to `false`.</span></span> <span data-ttu-id="d687b-161">Se le chiavi non vengono archiviate nella directory del profilo utente come previsto:</span><span class="sxs-lookup"><span data-stu-id="d687b-161">If keys aren't stored in the user profile directory as expected:</span></span>

  1. <span data-ttu-id="d687b-162">Passa alla cartella `%windir%/system32/inetsrv/config`.</span><span class="sxs-lookup"><span data-stu-id="d687b-162">Navigate to the `%windir%/system32/inetsrv/config` folder.</span></span>
  1. <span data-ttu-id="d687b-163">Aprire il file `applicationHost.config`.</span><span class="sxs-lookup"><span data-stu-id="d687b-163">Open the `applicationHost.config` file.</span></span>
  1. <span data-ttu-id="d687b-164">Individuare l'elemento `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>`.</span><span class="sxs-lookup"><span data-stu-id="d687b-164">Locate the `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` element.</span></span>
  1. <span data-ttu-id="d687b-165">Verificare che l'attributo `setProfileEnvironment` non sia presente, condizione che corrisponde all'impostazione predefinita `true`, o impostare in modo esplicito il valore dell'attributo su `true`.</span><span class="sxs-lookup"><span data-stu-id="d687b-165">Confirm that the `setProfileEnvironment` attribute isn't present, which defaults the value to `true`, or explicitly set the attribute's value to `true`.</span></span>

* <span data-ttu-id="d687b-166">**Usare il file system come archivio del gruppo di chiavi**</span><span class="sxs-lookup"><span data-stu-id="d687b-166">**Use the file system as a key ring store**</span></span>

  <span data-ttu-id="d687b-167">Modificare il codice dell'app per [usare il file system come archivio del gruppo di chiavi](xref:security/data-protection/configuration/overview).</span><span class="sxs-lookup"><span data-stu-id="d687b-167">Adjust the app code to [use the file system as a key ring store](xref:security/data-protection/configuration/overview).</span></span> <span data-ttu-id="d687b-168">Usare un certificato X509 per proteggere il gruppo di chiavi e verificare che sia un certificato attendibile.</span><span class="sxs-lookup"><span data-stu-id="d687b-168">Use an X509 certificate to protect the key ring and ensure the certificate is a trusted certificate.</span></span> <span data-ttu-id="d687b-169">Se il certificato è autofirmato, inserirlo nell'archivio radice attendibile.</span><span class="sxs-lookup"><span data-stu-id="d687b-169">If the certificate is self-signed, place the certificate in the Trusted Root store.</span></span>

  <span data-ttu-id="d687b-170">Quando si usa IIS in una web farm:</span><span class="sxs-lookup"><span data-stu-id="d687b-170">When using IIS in a web farm:</span></span>

  * <span data-ttu-id="d687b-171">Usare una condivisione di file a cui possono accedere tutti i computer.</span><span class="sxs-lookup"><span data-stu-id="d687b-171">Use a file share that all machines can access.</span></span>
  * <span data-ttu-id="d687b-172">Distribuire un certificato X509 in ogni computer.</span><span class="sxs-lookup"><span data-stu-id="d687b-172">Deploy an X509 certificate to each machine.</span></span> <span data-ttu-id="d687b-173">Configurare la [protezione dei dati nel codice](xref:security/data-protection/configuration/overview).</span><span class="sxs-lookup"><span data-stu-id="d687b-173">Configure [Data Protection in code](xref:security/data-protection/configuration/overview).</span></span>

* <span data-ttu-id="d687b-174">**Impostare criteri a livello di computer per la protezione dei dati**</span><span class="sxs-lookup"><span data-stu-id="d687b-174">**Set a machine-wide policy for Data Protection**</span></span>

  <span data-ttu-id="d687b-175">Il sistema di protezione dei dati ha un supporto limitato per l'impostazione di [criteri a livello di computer](xref:security/data-protection/configuration/machine-wide-policy) predefiniti per tutte le app che utilizzano le API di protezione dei dati.</span><span class="sxs-lookup"><span data-stu-id="d687b-175">The Data Protection system has limited support for setting a default [machine-wide policy](xref:security/data-protection/configuration/machine-wide-policy) for all apps that consume the Data Protection APIs.</span></span> <span data-ttu-id="d687b-176">Per altre informazioni, vedere <xref:security/data-protection/introduction>.</span><span class="sxs-lookup"><span data-stu-id="d687b-176">For more information, see <xref:security/data-protection/introduction>.</span></span>

## <a name="iis-configuration"></a><span data-ttu-id="d687b-177">Configurazione di IIS</span><span class="sxs-lookup"><span data-stu-id="d687b-177">IIS configuration</span></span>

<span data-ttu-id="d687b-178">**Sistemi operativi Windows Server**</span><span class="sxs-lookup"><span data-stu-id="d687b-178">**Windows Server operating systems**</span></span>

<span data-ttu-id="d687b-179">Abilitare il ruolo del server **Server Web (IIS)** e stabilire i servizi di ruolo.</span><span class="sxs-lookup"><span data-stu-id="d687b-179">Enable the **Web Server (IIS)** server role and establish role services.</span></span>

1. <span data-ttu-id="d687b-180">Usare la procedura guidata **Aggiungi ruoli e funzionalità** accessibile tramite il menu **Gestisci** o il collegamento in **Server Manager**.</span><span class="sxs-lookup"><span data-stu-id="d687b-180">Use the **Add Roles and Features** wizard from the **Manage** menu or the link in **Server Manager**.</span></span> <span data-ttu-id="d687b-181">Nel passaggio **Ruoli del server** selezionare la casella per **Server Web (IIS)**.</span><span class="sxs-lookup"><span data-stu-id="d687b-181">On the **Server Roles** step, check the box for **Web Server (IIS)**.</span></span>

   ![Il ruolo Server Web IIS viene selezionato nel passaggio dei ruoli del server Selezionare.](index/_static/server-roles-ws2016.png)

1. <span data-ttu-id="d687b-183">Dopo il passaggio **Funzionalità**, viene caricato il passaggio **Servizi ruolo** per Server Web (IIS).</span><span class="sxs-lookup"><span data-stu-id="d687b-183">After the **Features** step, the **Role services** step loads for Web Server (IIS).</span></span> <span data-ttu-id="d687b-184">Selezionare i servizi ruolo IIS desiderati o accettare i servizi ruolo predefiniti specificati.</span><span class="sxs-lookup"><span data-stu-id="d687b-184">Select the IIS role services desired or accept the default role services provided.</span></span>

   ![I servizi ruolo predefiniti vengono selezionati nel passaggio Selezionare i servizi ruolo.](index/_static/role-services-ws2016.png)

   <span data-ttu-id="d687b-186">**Autenticazione di Windows (facoltativa)**</span><span class="sxs-lookup"><span data-stu-id="d687b-186">**Windows Authentication (Optional)**</span></span>  
   <span data-ttu-id="d687b-187">Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: sicurezza del **server Web**  >  .</span><span class="sxs-lookup"><span data-stu-id="d687b-187">To enable Windows Authentication, expand the following nodes: **Web Server** > **Security**.</span></span> <span data-ttu-id="d687b-188">Selezionare la funzionalità **Autenticazione di Windows**.</span><span class="sxs-lookup"><span data-stu-id="d687b-188">Select the **Windows Authentication** feature.</span></span> <span data-ttu-id="d687b-189">Per ulteriori informazioni, vedere [ `<windowsAuthentication>` autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).</span><span class="sxs-lookup"><span data-stu-id="d687b-189">For more information, see [Windows Authentication `<windowsAuthentication>`](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) and [Configure Windows authentication](xref:security/authentication/windowsauth).</span></span>

   <span data-ttu-id="d687b-190">**WebSockets (facoltativo)**</span><span class="sxs-lookup"><span data-stu-id="d687b-190">**WebSockets (Optional)**</span></span>  
   <span data-ttu-id="d687b-191">WebSockets è supportato con ASP.NET Core 1.1 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="d687b-191">WebSockets is supported with ASP.NET Core 1.1 or later.</span></span> <span data-ttu-id="d687b-192">Per abilitare i WebSocket, espandere i nodi seguenti: sviluppo di applicazioni **server Web**  >  .</span><span class="sxs-lookup"><span data-stu-id="d687b-192">To enable WebSockets, expand the following nodes: **Web Server** > **Application Development**.</span></span> <span data-ttu-id="d687b-193">Selezionare la funzionalità **Protocollo WebSocket**.</span><span class="sxs-lookup"><span data-stu-id="d687b-193">Select the **WebSocket Protocol** feature.</span></span> <span data-ttu-id="d687b-194">Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).</span><span class="sxs-lookup"><span data-stu-id="d687b-194">For more information, see [WebSockets](xref:fundamentals/websockets).</span></span>

1. <span data-ttu-id="d687b-195">Procedere con il passaggio **Conferma** per installare il ruolo del server web e i servizi.</span><span class="sxs-lookup"><span data-stu-id="d687b-195">Proceed through the **Confirmation** step to install the web server role and services.</span></span> <span data-ttu-id="d687b-196">Dopo l'installazione del ruolo **server Web (IIS)** non è necessario riavviare il server o IIS.</span><span class="sxs-lookup"><span data-stu-id="d687b-196">A server/IIS restart isn't required after installing the **Web Server (IIS)** role.</span></span>

<span data-ttu-id="d687b-197">**Sistemi operativi desktop di Windows**</span><span class="sxs-lookup"><span data-stu-id="d687b-197">**Windows desktop operating systems**</span></span>

<span data-ttu-id="d687b-198">Abilitare **Console di gestione IIS** e **Servizi Web**.</span><span class="sxs-lookup"><span data-stu-id="d687b-198">Enable the **IIS Management Console** and **World Wide Web Services**.</span></span>

1. <span data-ttu-id="d687b-199">Passare a **Pannello di controllo**  >  **programmi** programmi  >  **e funzionalità**  >  **attiva o disattiva funzionalità Windows** (lato sinistro dello schermo).</span><span class="sxs-lookup"><span data-stu-id="d687b-199">Navigate to **Control Panel** > **Programs** > **Programs and Features** > **Turn Windows features on or off** (left side of the screen).</span></span>

1. <span data-ttu-id="d687b-200">Aprire il nodo **Internet Information Services**.</span><span class="sxs-lookup"><span data-stu-id="d687b-200">Open the **Internet Information Services** node.</span></span> <span data-ttu-id="d687b-201">Aprire il nodo **Strumenti di gestione Web**.</span><span class="sxs-lookup"><span data-stu-id="d687b-201">Open the **Web Management Tools** node.</span></span>

1. <span data-ttu-id="d687b-202">Selezionare la casella per **Console di gestione IIS**.</span><span class="sxs-lookup"><span data-stu-id="d687b-202">Check the box for **IIS Management Console**.</span></span>

1. <span data-ttu-id="d687b-203">Selezionare la casella per **Servizi World Wide Web**.</span><span class="sxs-lookup"><span data-stu-id="d687b-203">Check the box for **World Wide Web Services**.</span></span>

1. <span data-ttu-id="d687b-204">Accettare le funzionalità predefinite per **Servizi World Wide Web** o personalizzare le funzionalità IIS.</span><span class="sxs-lookup"><span data-stu-id="d687b-204">Accept the default features for **World Wide Web Services** or customize the IIS features.</span></span>

   <span data-ttu-id="d687b-205">**Autenticazione di Windows (facoltativa)**</span><span class="sxs-lookup"><span data-stu-id="d687b-205">**Windows Authentication (Optional)**</span></span>  
   <span data-ttu-id="d687b-206">Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: **World Wide Web sicurezza dei servizi**  >  .</span><span class="sxs-lookup"><span data-stu-id="d687b-206">To enable Windows Authentication, expand the following nodes: **World Wide Web Services** > **Security**.</span></span> <span data-ttu-id="d687b-207">Selezionare la funzionalità **Autenticazione di Windows**.</span><span class="sxs-lookup"><span data-stu-id="d687b-207">Select the **Windows Authentication** feature.</span></span> <span data-ttu-id="d687b-208">Per ulteriori informazioni, vedere [ `<windowsAuthentication>` autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).</span><span class="sxs-lookup"><span data-stu-id="d687b-208">For more information, see [Windows Authentication `<windowsAuthentication>`](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) and [Configure Windows authentication](xref:security/authentication/windowsauth).</span></span>

   <span data-ttu-id="d687b-209">**WebSockets (facoltativo)**</span><span class="sxs-lookup"><span data-stu-id="d687b-209">**WebSockets (Optional)**</span></span>  
   <span data-ttu-id="d687b-210">WebSockets è supportato con ASP.NET Core 1.1 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="d687b-210">WebSockets is supported with ASP.NET Core 1.1 or later.</span></span> <span data-ttu-id="d687b-211">Per abilitare i WebSocket, espandere i nodi seguenti: funzionalità di sviluppo di applicazioni di **World Wide Web Services**  >  .</span><span class="sxs-lookup"><span data-stu-id="d687b-211">To enable WebSockets, expand the following nodes: **World Wide Web Services** > **Application Development Features**.</span></span> <span data-ttu-id="d687b-212">Selezionare la funzionalità **Protocollo WebSocket**.</span><span class="sxs-lookup"><span data-stu-id="d687b-212">Select the **WebSocket Protocol** feature.</span></span> <span data-ttu-id="d687b-213">Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).</span><span class="sxs-lookup"><span data-stu-id="d687b-213">For more information, see [WebSockets](xref:fundamentals/websockets).</span></span>

1. <span data-ttu-id="d687b-214">Se l'installazione di IIS richiede un riavvio, riavviare il sistema.</span><span class="sxs-lookup"><span data-stu-id="d687b-214">If the IIS installation requires a restart, restart the system.</span></span>

![La console di gestione IIS e servizi World Wide Web sono selezionati nelle funzionalità di Windows.](index/_static/windows-features-win10.png)

## <a name="virtual-directories"></a><span data-ttu-id="d687b-216">Directory virtuali</span><span class="sxs-lookup"><span data-stu-id="d687b-216">Virtual Directories</span></span>

<span data-ttu-id="d687b-217">Le [directory virtuali IIS](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories) non sono supportate con le app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d687b-217">[IIS Virtual Directories](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories) aren't supported with ASP.NET Core apps.</span></span> <span data-ttu-id="d687b-218">Un'app può essere ospitata come [applicazione secondaria](#sub-applications).</span><span class="sxs-lookup"><span data-stu-id="d687b-218">An app can be hosted as a [sub-application](#sub-applications).</span></span>

## <a name="sub-applications"></a><span data-ttu-id="d687b-219">Applicazioni secondarie</span><span class="sxs-lookup"><span data-stu-id="d687b-219">Sub-applications</span></span>

<span data-ttu-id="d687b-220">Un'app ASP.NET Core può essere ospitata come [applicazione secondaria di IIS (app secondaria)](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications).</span><span class="sxs-lookup"><span data-stu-id="d687b-220">An ASP.NET Core app can be hosted as an [IIS sub-application (sub-app)](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications).</span></span> <span data-ttu-id="d687b-221">Il percorso dell'app secondaria diventa parte dell'URL dell'app radice.</span><span class="sxs-lookup"><span data-stu-id="d687b-221">The sub-app's path becomes part of the root app's URL.</span></span>

<span data-ttu-id="d687b-222">I collegamenti ad asset statici all'interno dell'app secondaria devono usare la notazione con tilde-barra (`~/`).</span><span class="sxs-lookup"><span data-stu-id="d687b-222">Static asset links within the sub-app should use tilde-slash (`~/`) notation.</span></span> <span data-ttu-id="d687b-223">La notazione con tilde-barra attiva un [helper tag](xref:mvc/views/tag-helpers/intro) per anteporre la base del percorso dell'app secondaria al collegamento relativo sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="d687b-223">Tilde-slash notation triggers a [Tag Helper](xref:mvc/views/tag-helpers/intro) to prepend the sub-app's pathbase to the rendered relative link.</span></span> <span data-ttu-id="d687b-224">Per un'app secondaria in `/subapp_path`, il rendering di un'immagine collegata con `src="~/image.png"` viene eseguito come `src="/subapp_path/image.png"`.</span><span class="sxs-lookup"><span data-stu-id="d687b-224">For a sub-app at `/subapp_path`, an image linked with `src="~/image.png"` is rendered as `src="/subapp_path/image.png"`.</span></span> <span data-ttu-id="d687b-225">Il middleware dei file statici dell'app radice non elabora la richiesta di file statici.</span><span class="sxs-lookup"><span data-stu-id="d687b-225">The root app's Static File Middleware doesn't process the static file request.</span></span> <span data-ttu-id="d687b-226">La richiesta viene elaborata dal middleware dei file statici dell'app secondaria.</span><span class="sxs-lookup"><span data-stu-id="d687b-226">The request is processed by the sub-app's Static File Middleware.</span></span>

<span data-ttu-id="d687b-227">Se l'attributo `src` di un asset statico viene impostato su un percorso assoluto (ad esempio, `src="/image.png"`), il rendering del collegamento viene eseguito senza la base del percorso dell'app secondaria.</span><span class="sxs-lookup"><span data-stu-id="d687b-227">If a static asset's `src` attribute is set to an absolute path (for example, `src="/image.png"`), the link is rendered without the sub-app's pathbase.</span></span> <span data-ttu-id="d687b-228">Il middleware dei file statici dell'app radice prova a servire l'asset dalla [radice Web](xref:fundamentals/index#web-root) dell'app radice con conseguente risposta *404 - Non trovato*, a meno che l'asset statico non sia disponibile dal'app radice.</span><span class="sxs-lookup"><span data-stu-id="d687b-228">The root app's Static File Middleware attempts to serve the asset from the root app's [web root](xref:fundamentals/index#web-root), which results in a *404 - Not Found* response unless the static asset is available from the root app.</span></span>

<span data-ttu-id="d687b-229">Per ospitare un'app ASP.NET Core come app secondaria in un'altra app ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="d687b-229">To host an ASP.NET Core app as a sub-app under another ASP.NET Core app:</span></span>

1. <span data-ttu-id="d687b-230">Definire un pool di app per l'app secondaria.</span><span class="sxs-lookup"><span data-stu-id="d687b-230">Establish an app pool for the sub-app.</span></span> <span data-ttu-id="d687b-231">Impostare **Versione .NET CLR** su **Nessun codice gestito** perché Core Common Language Runtime (CoreCLR) for .NET Core viene avviato per ospitare l'app nel processo di lavoro e non CLR desktop (.NET CLR).</span><span class="sxs-lookup"><span data-stu-id="d687b-231">Set the **.NET CLR Version** to **No Managed Code** because the Core Common Language Runtime (CoreCLR) for .NET Core is booted to host the app in the worker process, not the desktop CLR (.NET CLR).</span></span>

1. <span data-ttu-id="d687b-232">Aggiungere il sito radice in Gestione IIS con l'applicazione secondaria in una cartella all'interno del sito principale.</span><span class="sxs-lookup"><span data-stu-id="d687b-232">Add the root site in IIS Manager with the sub-app in a folder under the root site.</span></span>

1. <span data-ttu-id="d687b-233">Fare clic con il pulsante destro del mouse sulla cartella dell'applicazione secondaria in Gestione IIS e scegliere **Converti in applicazione**.</span><span class="sxs-lookup"><span data-stu-id="d687b-233">Right-click the sub-app folder in IIS Manager and select **Convert to Application**.</span></span>

1. <span data-ttu-id="d687b-234">Nella finestra di dialogo **Aggiungi applicazione** usare il pulsante **Seleziona** per **Pool di applicazioni** per assegnare il pool di app creato per l'app secondaria.</span><span class="sxs-lookup"><span data-stu-id="d687b-234">In the **Add Application** dialog, use the **Select** button for the **Application Pool** to assign the app pool that you created for the sub-app.</span></span> <span data-ttu-id="d687b-235">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="d687b-235">Select **OK**.</span></span>

<span data-ttu-id="d687b-236">L'assegnazione di un pool di app separato all'app secondaria è un requisito quando si usa il modello di hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="d687b-236">The assignment of a separate app pool to the sub-app is a requirement when using the in-process hosting model.</span></span>

<span data-ttu-id="d687b-237">Per ulteriori informazioni sul modello di hosting in-process e sulla configurazione del modulo ASP.NET Core, vedere <xref:host-and-deploy/aspnet-core-module> .</span><span class="sxs-lookup"><span data-stu-id="d687b-237">For more information on the in-process hosting model and configuring the ASP.NET Core Module, see <xref:host-and-deploy/aspnet-core-module>.</span></span>

## <a name="application-pools"></a><span data-ttu-id="d687b-238">Pool di applicazioni</span><span class="sxs-lookup"><span data-stu-id="d687b-238">Application Pools</span></span>

<span data-ttu-id="d687b-239">L'isolamento dei pool di app è determinato dal modello di hosting:</span><span class="sxs-lookup"><span data-stu-id="d687b-239">App pool isolation is determined by the hosting model:</span></span>

* <span data-ttu-id="d687b-240">Hosting in-process: le app sono necessarie per l'esecuzione in pool di applicazioni separati.</span><span class="sxs-lookup"><span data-stu-id="d687b-240">In-process hosting: Apps are required to run in separate app pools.</span></span>
* <span data-ttu-id="d687b-241">Hosting out-of-process: è consigliabile isolare le app le une dalle altre eseguendo ogni app nel proprio pool di applicazioni.</span><span class="sxs-lookup"><span data-stu-id="d687b-241">Out-of-process hosting: We recommend isolating the apps from each other by running each app in its own app pool.</span></span>

<span data-ttu-id="d687b-242">Nella finestra di dialogo **Aggiungi sito Web** è selezionato per impostazione predefinita un singolo pool di app per ogni app.</span><span class="sxs-lookup"><span data-stu-id="d687b-242">The IIS **Add Website** dialog defaults to a single app pool per app.</span></span> <span data-ttu-id="d687b-243">Quando si specifica un valore in **Nome del sito**, il testo viene automaticamente trasferito alla casella di testo **Pool di applicazioni**.</span><span class="sxs-lookup"><span data-stu-id="d687b-243">When a **Site name** is provided, the text is automatically transferred to the **Application pool** textbox.</span></span> <span data-ttu-id="d687b-244">Quando si aggiunge il sito viene creato un nuovo pool di applicazioni con il nome del sito.</span><span class="sxs-lookup"><span data-stu-id="d687b-244">A new app pool is created using the site name when the site is added.</span></span>

## <a name="application-pool-identity"></a><span data-ttu-id="d687b-245">Pool di applicazioni Identity</span><span class="sxs-lookup"><span data-stu-id="d687b-245">Application Pool Identity</span></span>

<span data-ttu-id="d687b-246">Un account di identità del pool di app consente di eseguire un'app in un account univoco, senza dover creare e gestire domini o account locali.</span><span class="sxs-lookup"><span data-stu-id="d687b-246">An app pool identity account allows an app to run under a unique account without having to create and manage domains or local accounts.</span></span> <span data-ttu-id="d687b-247">In IIS 8.0 o versioni successive il processo di lavoro amministrazione IIS (WAS) crea un account virtuale con il nome del nuovo pool di applicazioni ed esegue i processi di lavoro del pool di applicazioni inclusi nell'account per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="d687b-247">On IIS 8.0 or later, the IIS Admin Worker Process (WAS) creates a virtual account with the name of the new app pool and runs the app pool's worker processes under this account by default.</span></span> <span data-ttu-id="d687b-248">Nella console di gestione IIS in **Impostazioni avanzate** per il pool di applicazioni, verificare che **Identity** sia impostato su Usa `ApplicationPoolIdentity` :</span><span class="sxs-lookup"><span data-stu-id="d687b-248">In the IIS Management Console under **Advanced Settings** for the app pool, ensure that the **Identity** is set to use `ApplicationPoolIdentity`:</span></span>

![Finestra di dialogo Impostazioni avanzate del pool di applicazione](index/_static/apppool-identity.png)

<span data-ttu-id="d687b-250">Il processo di gestione IIS crea un identificatore sicuro con il nome del pool di app nel sistema di sicurezza di Windows.</span><span class="sxs-lookup"><span data-stu-id="d687b-250">The IIS management process creates a secure identifier with the name of the app pool in the Windows Security System.</span></span> <span data-ttu-id="d687b-251">Le risorse possono essere protette mediante questa identità,</span><span class="sxs-lookup"><span data-stu-id="d687b-251">Resources can be secured using this identity.</span></span> <span data-ttu-id="d687b-252">che tuttavia non è un account utente reale e non viene visualizzata nella console di gestione utenti di Windows.</span><span class="sxs-lookup"><span data-stu-id="d687b-252">However, this identity isn't a real user account and doesn't show up in the Windows User Management Console.</span></span>

<span data-ttu-id="d687b-253">Se il processo di lavoro IIS richiede l'accesso con privilegi elevati all'app, modificare l'elenco di controllo di accesso (ACL) della directory contenente l'app:</span><span class="sxs-lookup"><span data-stu-id="d687b-253">If the IIS worker process requires elevated access to the app, modify the Access Control List (ACL) for the directory containing the app:</span></span>

1. <span data-ttu-id="d687b-254">Aprire Esplora risorse, quindi spostarsi nella directory.</span><span class="sxs-lookup"><span data-stu-id="d687b-254">Open Windows Explorer and navigate to the directory.</span></span>

1. <span data-ttu-id="d687b-255">Fare clic con il pulsante destro del mouse sulla directory e scegliere **Proprietà**.</span><span class="sxs-lookup"><span data-stu-id="d687b-255">Right-click on the directory and select **Properties**.</span></span>

1. <span data-ttu-id="d687b-256">Nella scheda **Sicurezza** selezionare il pulsante **Modifica** e quindi il pulsante **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="d687b-256">Under the **Security** tab, select the **Edit** button and then the **Add** button.</span></span>

1. <span data-ttu-id="d687b-257">Fare clic sul pulsante **Percorsi** e verificare che il sistema sia selezionato.</span><span class="sxs-lookup"><span data-stu-id="d687b-257">Select the **Locations** button and make sure the system is selected.</span></span>

1. <span data-ttu-id="d687b-258">Immettere il `IIS AppPool\{APP POOL NAME}` formato, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, in **immettere i nomi degli oggetti da selezionare** .</span><span class="sxs-lookup"><span data-stu-id="d687b-258">Enter `IIS AppPool\{APP POOL NAME}` format, where the placeholder `{APP POOL NAME}` is the app pool name, in **Enter the object names to select** area.</span></span> <span data-ttu-id="d687b-259">Fare clic sul pulsante **Controlla nomi**.</span><span class="sxs-lookup"><span data-stu-id="d687b-259">Select the **Check Names** button.</span></span> <span data-ttu-id="d687b-260">Per *DefaultAppPool* controllare i nomi usando `IIS AppPool\DefaultAppPool` .</span><span class="sxs-lookup"><span data-stu-id="d687b-260">For the *DefaultAppPool* check the names using `IIS AppPool\DefaultAppPool`.</span></span> <span data-ttu-id="d687b-261">Quando si seleziona il pulsante **Controlla nomi** , il valore `DefaultAppPool` è indicato nell'area dei nomi degli oggetti.</span><span class="sxs-lookup"><span data-stu-id="d687b-261">When the **Check Names** button is selected, a value of `DefaultAppPool` is indicated in the object names area.</span></span> <span data-ttu-id="d687b-262">Non è possibile immettere il nome del pool di applicazioni direttamente nell'area dei nomi degli oggetti.</span><span class="sxs-lookup"><span data-stu-id="d687b-262">It isn't possible to enter the app pool name directly into the object names area.</span></span> <span data-ttu-id="d687b-263">Usare il `IIS AppPool\{APP POOL NAME}` formato, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, quando si verifica il nome dell'oggetto.</span><span class="sxs-lookup"><span data-stu-id="d687b-263">Use the `IIS AppPool\{APP POOL NAME}` format, where the placeholder `{APP POOL NAME}` is the app pool name, when checking for the object name.</span></span>

   ![Finestra di dialogo Seleziona utenti o gruppi per la cartella dell'app: il nome del pool di applicazioni "DefaultAppPool" viene aggiunto in coda a "IIS AppPool\" nell'area dei nomi degli oggetti prima di selezionare "Controlla nomi".](index/_static/select-users-or-groups-1.png)

1. <span data-ttu-id="d687b-265">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="d687b-265">Select **OK**.</span></span>

   ![Finestra di dialogo Seleziona utenti o gruppi per la cartella dell'app: dopo aver selezionato "Controlla nomi", il nome dell'oggetto "DefaultAppPool" viene visualizzato nell'area dei nomi degli oggetti.](index/_static/select-users-or-groups-2.png)

1. <span data-ttu-id="d687b-267">Le autorizzazioni di lettura ed esecuzione devono essere concesse per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="d687b-267">Read &amp; execute permissions should be granted by default.</span></span> <span data-ttu-id="d687b-268">Fornire autorizzazioni aggiuntive in base alle necessità.</span><span class="sxs-lookup"><span data-stu-id="d687b-268">Provide additional permissions as needed.</span></span>

<span data-ttu-id="d687b-269">L'accesso può essere concesso anche dal prompt dei comandi usando lo strumento **ICACLS**.</span><span class="sxs-lookup"><span data-stu-id="d687b-269">Access can also be granted at a command prompt using the **ICACLS** tool.</span></span> <span data-ttu-id="d687b-270">Usando *DefaultAppPool* come esempio, viene eseguito il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="d687b-270">Using the *DefaultAppPool* as an example, the following command is used:</span></span>

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

<span data-ttu-id="d687b-271">Per altre informazioni, vedere l'argomento [icacls](/windows-server/administration/windows-commands/icacls).</span><span class="sxs-lookup"><span data-stu-id="d687b-271">For more information, see the [icacls](/windows-server/administration/windows-commands/icacls) topic.</span></span>

## <a name="http2-support"></a><span data-ttu-id="d687b-272">Supporto HTTP/2</span><span class="sxs-lookup"><span data-stu-id="d687b-272">HTTP/2 support</span></span>

<span data-ttu-id="d687b-273">[HTTP/2](https://httpwg.org/specs/rfc7540.html) è supportato con ASP.NET Core negli scenari di distribuzione IIS seguenti:</span><span class="sxs-lookup"><span data-stu-id="d687b-273">[HTTP/2](https://httpwg.org/specs/rfc7540.html) is supported with ASP.NET Core in the following IIS deployment scenarios:</span></span>

* <span data-ttu-id="d687b-274">In-Process</span><span class="sxs-lookup"><span data-stu-id="d687b-274">In-process</span></span>
  * <span data-ttu-id="d687b-275">Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="d687b-275">Windows Server 2016/Windows 10 or later; IIS 10 or later</span></span>
  * <span data-ttu-id="d687b-276">Connessione TLS 1.2 o successiva</span><span class="sxs-lookup"><span data-stu-id="d687b-276">TLS 1.2 or later connection</span></span>
* <span data-ttu-id="d687b-277">Out-of-process</span><span class="sxs-lookup"><span data-stu-id="d687b-277">Out-of-process</span></span>
  * <span data-ttu-id="d687b-278">Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="d687b-278">Windows Server 2016/Windows 10 or later; IIS 10 or later</span></span>
  * <span data-ttu-id="d687b-279">Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso al [server Kestrel](xref:fundamentals/servers/kestrel) usa HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="d687b-279">Public-facing edge server connections use HTTP/2, but the reverse proxy connection to the [Kestrel server](xref:fundamentals/servers/kestrel) uses HTTP/1.1.</span></span>
  * <span data-ttu-id="d687b-280">Connessione TLS 1.2 o successiva</span><span class="sxs-lookup"><span data-stu-id="d687b-280">TLS 1.2 or later connection</span></span>

<span data-ttu-id="d687b-281">Per una distribuzione in-process quando viene stabilita una connessione HTTP/2, i [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) report `HTTP/2` .</span><span class="sxs-lookup"><span data-stu-id="d687b-281">For an in-process deployment when an HTTP/2 connection is established, [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) reports `HTTP/2`.</span></span> <span data-ttu-id="d687b-282">Per una distribuzione out-of-process quando viene stabilita una connessione HTTP/2, i [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) report `HTTP/1.1` .</span><span class="sxs-lookup"><span data-stu-id="d687b-282">For an out-of-process deployment when an HTTP/2 connection is established, [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) reports `HTTP/1.1`.</span></span>

<span data-ttu-id="d687b-283">Per altre informazioni sui modelli di hosting in-process e out-of-process, vedere <xref:host-and-deploy/aspnet-core-module>.</span><span class="sxs-lookup"><span data-stu-id="d687b-283">For more information on the in-process and out-of-process hosting models, see <xref:host-and-deploy/aspnet-core-module>.</span></span>

<span data-ttu-id="d687b-284">HTTP/2 è abilitato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="d687b-284">HTTP/2 is enabled by default.</span></span> <span data-ttu-id="d687b-285">Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="d687b-285">Connections fall back to HTTP/1.1 if an HTTP/2 connection isn't established.</span></span> <span data-ttu-id="d687b-286">Per altre informazioni sulla configurazione di HTTP/2 con le distribuzioni IIS, vedere [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS).</span><span class="sxs-lookup"><span data-stu-id="d687b-286">For more information on HTTP/2 configuration with IIS deployments, see [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis).</span></span>

## <a name="cors-preflight-requests"></a><span data-ttu-id="d687b-287">Richieste preliminari CORS</span><span class="sxs-lookup"><span data-stu-id="d687b-287">CORS preflight requests</span></span>

<span data-ttu-id="d687b-288">*Questa sezione si applica solo alle app ASP.NET Core destinate a .NET Framework.*</span><span class="sxs-lookup"><span data-stu-id="d687b-288">*This section only applies to ASP.NET Core apps that target the .NET Framework.*</span></span>

<span data-ttu-id="d687b-289">Per un'app ASP.NET Core destinata a .NET Framework, le richieste OPTIONS non vengono passate all'app per impostazione predefinita in IIS.</span><span class="sxs-lookup"><span data-stu-id="d687b-289">For an ASP.NET Core app that targets the .NET Framework, OPTIONS requests aren't passed to the app by default in IIS.</span></span> <span data-ttu-id="d687b-290">Per informazioni su come configurare i gestori IIS dell'app in `web.config` per passare le richieste di opzioni, vedere [abilitare le richieste tra le origini in API Web ASP.NET 2:](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)funzionamento di CORS.</span><span class="sxs-lookup"><span data-stu-id="d687b-290">To learn how to configure the app's IIS handlers in `web.config` to pass OPTIONS requests, see [Enable cross-origin requests in ASP.NET Web API 2: How CORS Works](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works).</span></span>

## <a name="application-initialization-module-and-idle-timeout"></a><span data-ttu-id="d687b-291">Modulo di inizializzazione dell'applicazione e timeout di inattività</span><span class="sxs-lookup"><span data-stu-id="d687b-291">Application Initialization Module and Idle Timeout</span></span>

<span data-ttu-id="d687b-292">Per l'hosting in IIS dal modulo ASP.NET Core versione 2:</span><span class="sxs-lookup"><span data-stu-id="d687b-292">When hosted in IIS by the ASP.NET Core Module version 2:</span></span>

* <span data-ttu-id="d687b-293">[Modulo di inizializzazione dell'applicazione](#application-initialization-module): l'app è ospitata [in-process](xref:host-and-deploy/iis/in-process-hosting) o [out-of-process](xref:host-and-deploy/iis/out-of-process-hosting) può essere configurata per l'avvio automatico in un processo di lavoro o il riavvio del server.</span><span class="sxs-lookup"><span data-stu-id="d687b-293">[Application Initialization Module](#application-initialization-module): App's hosted [in-process](xref:host-and-deploy/iis/in-process-hosting) or [out-of-process](xref:host-and-deploy/iis/out-of-process-hosting) can be configured to start automatically on a worker process restart or server restart.</span></span>
* <span data-ttu-id="d687b-294">[Timeout di inattività](#idle-timeout): è possibile configurare l'app ospitata [in-process in](xref:host-and-deploy/iis/in-process-hosting) modo che non si timeout durante i periodi di inattività.</span><span class="sxs-lookup"><span data-stu-id="d687b-294">[Idle Timeout](#idle-timeout): App's hosted [in-process](xref:host-and-deploy/iis/in-process-hosting) can be configured not to time out during periods of inactivity.</span></span>

### <a name="application-initialization-module"></a><span data-ttu-id="d687b-295">Modulo Inizializzazione applicazione</span><span class="sxs-lookup"><span data-stu-id="d687b-295">Application Initialization Module</span></span>

<span data-ttu-id="d687b-296">*Si applica alle app ospitate in-process e out-of-process.*</span><span class="sxs-lookup"><span data-stu-id="d687b-296">*Applies to apps hosted in-process and out-of-process.*</span></span>

<span data-ttu-id="d687b-297">[Inizializzazione applicazione di IIS](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) è una funzionalità di IIS che invia una richiesta HTTP all'app quando il pool di app viene avviato o riciclato.</span><span class="sxs-lookup"><span data-stu-id="d687b-297">[IIS Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) is an IIS feature that sends an HTTP request to the app when the app pool starts or is recycled.</span></span> <span data-ttu-id="d687b-298">La richiesta attiva l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="d687b-298">The request triggers the app to start.</span></span> <span data-ttu-id="d687b-299">Per impostazione predefinita, IIS invia una richiesta all'URL radice dell'app (`/`) per inizializzare l'app (vedere le [risorse aggiuntive](#application-initialization-module-and-idle-timeout-additional-resources) per altri dettagli sulla configurazione).</span><span class="sxs-lookup"><span data-stu-id="d687b-299">By default, IIS issues a request to the app's root URL (`/`) to initialize the app (see the [additional resources](#application-initialization-module-and-idle-timeout-additional-resources) for more details on configuration).</span></span>

<span data-ttu-id="d687b-300">Verificare che la funzionalità del ruolo Inizializzazione applicazione di IIS sia abilitata:</span><span class="sxs-lookup"><span data-stu-id="d687b-300">Confirm that the IIS Application Initialization role feature in enabled:</span></span>

<span data-ttu-id="d687b-301">Nei sistemi desktop Windows 7 o versioni successive quando si usa IIS in locale:</span><span class="sxs-lookup"><span data-stu-id="d687b-301">On Windows 7 or later desktop systems when using IIS locally:</span></span>

1. <span data-ttu-id="d687b-302">Passare a **Pannello di controllo**  >  **programmi** programmi  >  **e funzionalità**  >  **attiva o disattiva funzionalità Windows** (lato sinistro dello schermo).</span><span class="sxs-lookup"><span data-stu-id="d687b-302">Navigate to **Control Panel** > **Programs** > **Programs and Features** > **Turn Windows features on or off** (left side of the screen).</span></span>
1. <span data-ttu-id="d687b-303">Aprire **Internet Information Services**  >  funzionalità di sviluppo di applicazioni **World Wide Web Services**  >  .</span><span class="sxs-lookup"><span data-stu-id="d687b-303">Open **Internet Information Services** > **World Wide Web Services** > **Application Development Features**.</span></span>
1. <span data-ttu-id="d687b-304">Selezionare la casella di controllo **Inizializzazione applicazione**.</span><span class="sxs-lookup"><span data-stu-id="d687b-304">Select the check box for **Application Initialization**.</span></span>

<span data-ttu-id="d687b-305">In Windows Server 2008 R2 o versioni successive:</span><span class="sxs-lookup"><span data-stu-id="d687b-305">On Windows Server 2008 R2 or later:</span></span>

1. <span data-ttu-id="d687b-306">Aprire l'**Aggiunta guidata ruoli e funzionalità**.</span><span class="sxs-lookup"><span data-stu-id="d687b-306">Open the **Add Roles and Features Wizard**.</span></span>
1. <span data-ttu-id="d687b-307">Nel pannello **Selezione servizi ruolo** aprire il nodo **Sviluppo applicazioni**.</span><span class="sxs-lookup"><span data-stu-id="d687b-307">In the **Select role services** panel, open the **Application Development** node.</span></span>
1. <span data-ttu-id="d687b-308">Selezionare la casella di controllo **Inizializzazione applicazione**.</span><span class="sxs-lookup"><span data-stu-id="d687b-308">Select the check box for **Application Initialization**.</span></span>

<span data-ttu-id="d687b-309">Per abilitare il modulo Inizializzazione applicazione per il sito, usare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="d687b-309">Use either of the following approaches to enable the Application Initialization Module for the site:</span></span>

* <span data-ttu-id="d687b-310">In Gestione IIS:</span><span class="sxs-lookup"><span data-stu-id="d687b-310">Using IIS Manager:</span></span>

  1. <span data-ttu-id="d687b-311">Selezionare **Pool di applicazioni** nel pannello **Connessioni**.</span><span class="sxs-lookup"><span data-stu-id="d687b-311">Select **Application Pools** in the **Connections** panel.</span></span>
  1. <span data-ttu-id="d687b-312">Fare clic con il pulsante destro del mouse sul pool di app dell'app nell'elenco e scegliere **Impostazioni avanzate**.</span><span class="sxs-lookup"><span data-stu-id="d687b-312">Right-click the app's app pool in the list and select **Advanced Settings**.</span></span>
  1. <span data-ttu-id="d687b-313">La **modalità di avvio** predefinita è `OnDemand` .</span><span class="sxs-lookup"><span data-stu-id="d687b-313">The default **Start Mode** is `OnDemand`.</span></span> <span data-ttu-id="d687b-314">Impostare la **modalità di avvio** su `AlwaysRunning` .</span><span class="sxs-lookup"><span data-stu-id="d687b-314">Set the **Start Mode** to `AlwaysRunning`.</span></span> <span data-ttu-id="d687b-315">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="d687b-315">Select **OK**.</span></span>
  1. <span data-ttu-id="d687b-316">Aprire il nodo **Siti** nel pannello **Connessioni**.</span><span class="sxs-lookup"><span data-stu-id="d687b-316">Open the **Sites** node in the **Connections** panel.</span></span>
  1. <span data-ttu-id="d687b-317">Fare clic con il pulsante destro del mouse sull'app e scegliere **Gestisci**  >  **Impostazioni avanzate** sito Web.</span><span class="sxs-lookup"><span data-stu-id="d687b-317">Right-click the app and select **Manage Website** > **Advanced Settings**.</span></span>
  1. <span data-ttu-id="d687b-318">L'impostazione predefinita di **precaricamento abilitato** è `False` .</span><span class="sxs-lookup"><span data-stu-id="d687b-318">The default **Preload Enabled** setting is `False`.</span></span> <span data-ttu-id="d687b-319">Impostare **preload abilitato** su `True` .</span><span class="sxs-lookup"><span data-stu-id="d687b-319">Set **Preload Enabled** to `True`.</span></span> <span data-ttu-id="d687b-320">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="d687b-320">Select **OK**.</span></span>

* <span data-ttu-id="d687b-321">Usando `web.config` , aggiungere l' `<applicationInitialization>` elemento con `doAppInitAfterRestart` impostato su `true` agli `<system.webServer>` elementi nel file dell'app `web.config` :</span><span class="sxs-lookup"><span data-stu-id="d687b-321">Using `web.config`, add the `<applicationInitialization>` element with `doAppInitAfterRestart` set to `true` to the `<system.webServer>` elements in the app's `web.config` file:</span></span>

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <applicationInitialization doAppInitAfterRestart="true" />
      </system.webServer>
    </location>
  </configuration>
  ```

### <a name="idle-timeout"></a><span data-ttu-id="d687b-322">Timeout di inattività</span><span class="sxs-lookup"><span data-stu-id="d687b-322">Idle Timeout</span></span>

<span data-ttu-id="d687b-323">*Si applica solo alle app ospitate in-process.*</span><span class="sxs-lookup"><span data-stu-id="d687b-323">*Only applies to apps hosted in-process.*</span></span>

<span data-ttu-id="d687b-324">Per evitare l'inattività dell'app, impostare il timeout di inattività del pool di app tramite Gestione IIS:</span><span class="sxs-lookup"><span data-stu-id="d687b-324">To prevent the app from idling, set the app pool's idle timeout using IIS Manager:</span></span>

1. <span data-ttu-id="d687b-325">Selezionare **Pool di applicazioni** nel pannello **Connessioni**.</span><span class="sxs-lookup"><span data-stu-id="d687b-325">Select **Application Pools** in the **Connections** panel.</span></span>
1. <span data-ttu-id="d687b-326">Fare clic con il pulsante destro del mouse sul pool di app dell'app nell'elenco e scegliere **Impostazioni avanzate**.</span><span class="sxs-lookup"><span data-stu-id="d687b-326">Right-click the app's app pool in the list and select **Advanced Settings**.</span></span>
1. <span data-ttu-id="d687b-327">Il **timeout di inattività predefinito (minuti)** è `20` minuti.</span><span class="sxs-lookup"><span data-stu-id="d687b-327">The default **Idle Time-out (minutes)** is `20` minutes.</span></span> <span data-ttu-id="d687b-328">Imposta il **timeout di inattività (minuti)** su `0` (zero).</span><span class="sxs-lookup"><span data-stu-id="d687b-328">Set the **Idle Time-out (minutes)** to `0` (zero).</span></span> <span data-ttu-id="d687b-329">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="d687b-329">Select **OK**.</span></span>
1. <span data-ttu-id="d687b-330">Riciclare il processo di lavoro.</span><span class="sxs-lookup"><span data-stu-id="d687b-330">Recycle the worker process.</span></span>

<span data-ttu-id="d687b-331">Per evitare il timeout delle app ospitate [out-of-process](xref:host-and-deploy/iis/out-of-process-hosting), usare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="d687b-331">To prevent apps hosted [out-of-process](xref:host-and-deploy/iis/out-of-process-hosting) from timing out, use either of the following approaches:</span></span>

* <span data-ttu-id="d687b-332">Effettuare il ping dell'app da un servizio esterno per mantenerla in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="d687b-332">Ping the app from an external service in order to keep it running.</span></span>
* <span data-ttu-id="d687b-333">Se l'app ospita solo servizi in background, evitare l'hosting IIS e usare un [servizio Windows per ospitare l'app ASP.NET Core](xref:host-and-deploy/windows-service).</span><span class="sxs-lookup"><span data-stu-id="d687b-333">If the app only hosts background services, avoid IIS hosting and use a [Windows Service to host the ASP.NET Core app](xref:host-and-deploy/windows-service).</span></span>

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a><span data-ttu-id="d687b-334">Risorse aggiuntive per il modulo Inizializzazione applicazione e il timeout di inattività</span><span class="sxs-lookup"><span data-stu-id="d687b-334">Application Initialization Module and Idle Timeout additional resources</span></span>

* <span data-ttu-id="d687b-335">[IIS 8.0 Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) (Inizializzazione delle applicazioni in IIS 8.0)</span><span class="sxs-lookup"><span data-stu-id="d687b-335">[IIS 8.0 Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)</span></span>
* <span data-ttu-id="d687b-336">[Inizializzazione `<applicationInitialization>` dell'applicazione ](/iis/configuration/system.webserver/applicationinitialization/).</span><span class="sxs-lookup"><span data-stu-id="d687b-336">[Application Initialization `<applicationInitialization>`](/iis/configuration/system.webserver/applicationinitialization/).</span></span>
* <span data-ttu-id="d687b-337">[Impostazioni del modello di processo per un `<processModel>` pool di applicazioni ](/iis/configuration/system.applicationhost/applicationpools/add/processmodel).</span><span class="sxs-lookup"><span data-stu-id="d687b-337">[Process Model Settings for an Application Pool `<processModel>`](/iis/configuration/system.applicationhost/applicationpools/add/processmodel).</span></span>

## <a name="module-schema-and-configuration-file-locations"></a><span data-ttu-id="d687b-338">Percorsi dei file di modulo, schema e configurazione</span><span class="sxs-lookup"><span data-stu-id="d687b-338">Module, schema, and configuration file locations</span></span>

### <a name="module"></a><span data-ttu-id="d687b-339">Modulo</span><span class="sxs-lookup"><span data-stu-id="d687b-339">Module</span></span>

<span data-ttu-id="d687b-340">**IIS (x86/amd64)**:</span><span class="sxs-lookup"><span data-stu-id="d687b-340">**IIS (x86/amd64)**:</span></span>

* `%windir%\System32\inetsrv\aspnetcore.dll`

* `%windir%\SysWOW64\inetsrv\aspnetcore.dll`

* `%ProgramFiles%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

<span data-ttu-id="d687b-341">**IIS Express (x86/amd64)**:</span><span class="sxs-lookup"><span data-stu-id="d687b-341">**IIS Express (x86/amd64)**:</span></span>

* `%ProgramFiles%\IIS Express\aspnetcore.dll`

* `%ProgramFiles(x86)%\IIS Express\aspnetcore.dll`

* `%ProgramFiles%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

### <a name="schema"></a><span data-ttu-id="d687b-342">SCHEMA</span><span class="sxs-lookup"><span data-stu-id="d687b-342">Schema</span></span>

<span data-ttu-id="d687b-343">**IIS**</span><span class="sxs-lookup"><span data-stu-id="d687b-343">**IIS**</span></span>

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml`

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema_v2.xml`

<span data-ttu-id="d687b-344">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="d687b-344">**IIS Express**</span></span>

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml`

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema_v2.xml`

### <a name="configuration"></a><span data-ttu-id="d687b-345">Configurazione</span><span class="sxs-lookup"><span data-stu-id="d687b-345">Configuration</span></span>

<span data-ttu-id="d687b-346">**IIS**</span><span class="sxs-lookup"><span data-stu-id="d687b-346">**IIS**</span></span>

* `%windir%\System32\inetsrv\config\applicationHost.config`

<span data-ttu-id="d687b-347">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="d687b-347">**IIS Express**</span></span>

* <span data-ttu-id="d687b-348">Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`</span><span class="sxs-lookup"><span data-stu-id="d687b-348">Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`</span></span>

* <span data-ttu-id="d687b-349">*iisexpress.exe* CLI `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`</span><span class="sxs-lookup"><span data-stu-id="d687b-349">*iisexpress.exe* CLI: `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`</span></span>

<span data-ttu-id="d687b-350">È possibile trovare i file eseguendo una ricerca `aspnetcore` nel `applicationHost.config` file.</span><span class="sxs-lookup"><span data-stu-id="d687b-350">The files can be found by searching for `aspnetcore` in the `applicationHost.config` file.</span></span>
