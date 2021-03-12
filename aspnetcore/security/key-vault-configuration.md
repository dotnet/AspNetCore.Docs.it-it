---
title: Provider di configurazione di Azure Key Vault in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare il provider di configurazione Azure Key Vault per configurare un'app usando coppie nome-valore caricate in fase di esecuzione.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc, devx-track-azurecli
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
uid: security/key-vault-configuration
ms.openlocfilehash: ab3a462a3e09113e96c6bdd0c034bff3e0bebdfa
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588295"
---
# <a name="azure-key-vault-configuration-provider-in-aspnet-core"></a><span data-ttu-id="02dbd-103">Provider di configurazione di Azure Key Vault in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="02dbd-103">Azure Key Vault Configuration Provider in ASP.NET Core</span></span>

<span data-ttu-id="02dbd-104">Di [Andrew Stanton-Nurse](https://github.com/anurse)</span><span class="sxs-lookup"><span data-stu-id="02dbd-104">By [Andrew Stanton-Nurse](https://github.com/anurse)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="02dbd-105">Questo documento illustra come usare il provider di configurazione [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) per caricare i valori di configurazione dell'app da Azure Key Vault segreti.</span><span class="sxs-lookup"><span data-stu-id="02dbd-105">This document explains how to use the [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) Configuration Provider to load app configuration values from Azure Key Vault secrets.</span></span> <span data-ttu-id="02dbd-106">Azure Key Vault è un servizio basato sul cloud che aiuta a proteggere le chiavi crittografiche e i segreti usati da app e servizi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-106">Azure Key Vault is a cloud-based service that assists in safeguarding cryptographic keys and secrets used by apps and services.</span></span> <span data-ttu-id="02dbd-107">Gli scenari comuni per l'uso di Azure Key Vault con le app ASP.NET Core includono:</span><span class="sxs-lookup"><span data-stu-id="02dbd-107">Common scenarios for using Azure Key Vault with ASP.NET Core apps include:</span></span>

* <span data-ttu-id="02dbd-108">Controllo dell'accesso ai dati di configurazione riservati.</span><span class="sxs-lookup"><span data-stu-id="02dbd-108">Controlling access to sensitive configuration data.</span></span>
* <span data-ttu-id="02dbd-109">Soddisfare i requisiti per i moduli di protezione hardware convalidati FIPS 140-2 Level 2 (HSM) quando si archiviano i dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-109">Meeting the requirement for FIPS 140-2 Level 2 validated Hardware Security Modules (HSM's) when storing configuration data.</span></span>

<span data-ttu-id="02dbd-110">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/key-vault-configuration/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="02dbd-110">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/key-vault-configuration/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="packages"></a><span data-ttu-id="02dbd-111">Pacchetti</span><span class="sxs-lookup"><span data-stu-id="02dbd-111">Packages</span></span>

<span data-ttu-id="02dbd-112">Aggiungere i riferimenti ai pacchetti per i pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="02dbd-112">Add package references for the following packages:</span></span>

* [<span data-ttu-id="02dbd-113">Azure.Extensions.AspNetCore.Configuration. Segreti</span><span class="sxs-lookup"><span data-stu-id="02dbd-113">Azure.Extensions.AspNetCore.Configuration.Secrets</span></span>](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.Configuration.Secrets)
* <span data-ttu-id="02dbd-114">[Azure.Identity](https://www.nuget.org/packages/Azure.Identity)</span><span class="sxs-lookup"><span data-stu-id="02dbd-114">[Azure.Identity](https://www.nuget.org/packages/Azure.Identity)</span></span>

## <a name="sample-app"></a><span data-ttu-id="02dbd-115">App di esempio</span><span class="sxs-lookup"><span data-stu-id="02dbd-115">Sample app</span></span>

<span data-ttu-id="02dbd-116">L'app di esempio viene eseguita in una delle due modalità determinate dall' `#define` istruzione all'inizio del file *Program.cs* :</span><span class="sxs-lookup"><span data-stu-id="02dbd-116">The sample app runs in either of two modes determined by the `#define` statement at the top of the *Program.cs* file:</span></span>

* <span data-ttu-id="02dbd-117">`Certificate`: Illustra l'uso di un ID client Azure Key Vault e di un certificato X. 509 per accedere ai segreti archiviati nel Azure Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-117">`Certificate`: Demonstrates the use of an Azure Key Vault Client ID and X.509 certificate to access secrets stored in Azure Key Vault.</span></span> <span data-ttu-id="02dbd-118">Questa versione dell'esempio può essere eseguita da qualsiasi posizione, distribuita nel servizio app Azure o in qualsiasi host in grado di servire un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="02dbd-118">This version of the sample can be run from any location, deployed to Azure App Service or any host capable of serving an ASP.NET Core app.</span></span>
* <span data-ttu-id="02dbd-119">`Managed`: Illustra come usare le [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview) per autenticare l'app Azure Key Vault con Azure ad autenticazione senza credenziali archiviate nel codice o nella configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-119">`Managed`: Demonstrates how to use [Managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview) to authenticate the app to Azure Key Vault with Azure AD authentication without credentials stored in the app's code or configuration.</span></span> <span data-ttu-id="02dbd-120">Quando si usano identità gestite per l'autenticazione, non è necessario un Azure AD ID applicazione e una password (segreto client).</span><span class="sxs-lookup"><span data-stu-id="02dbd-120">When using managed identities to authenticate, an Azure AD Application ID and Password (Client Secret) aren't required.</span></span> <span data-ttu-id="02dbd-121">La `Managed` versione dell'esempio deve essere distribuita in Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-121">The `Managed` version of the sample must be deployed to Azure.</span></span> <span data-ttu-id="02dbd-122">Seguire le istruzioni riportate nella sezione [usare le identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) .</span><span class="sxs-lookup"><span data-stu-id="02dbd-122">Follow the guidance in the [Use the Managed identities for Azure resources](#use-managed-identities-for-azure-resources) section.</span></span>

<span data-ttu-id="02dbd-123">Per ulteriori informazioni su come configurare un'app di esempio utilizzando le direttive per il preprocessore ( `#define` ), vedere <xref:index#preprocessor-directives-in-sample-code> .</span><span class="sxs-lookup"><span data-stu-id="02dbd-123">For more information on how to configure a sample app using preprocessor directives (`#define`), see <xref:index#preprocessor-directives-in-sample-code>.</span></span>

## <a name="secret-storage-in-the-development-environment"></a><span data-ttu-id="02dbd-124">Archiviazione segreta nell'ambiente di sviluppo</span><span class="sxs-lookup"><span data-stu-id="02dbd-124">Secret storage in the Development environment</span></span>

<span data-ttu-id="02dbd-125">Impostare i segreti in locale usando lo [strumento di gestione dei segreti](xref:security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="02dbd-125">Set secrets locally using the [Secret Manager tool](xref:security/app-secrets).</span></span> <span data-ttu-id="02dbd-126">Quando l'app di esempio viene eseguita nel computer locale nell'ambiente di sviluppo, i segreti vengono caricati dall'archivio dei segreti dell'utente locale.</span><span class="sxs-lookup"><span data-stu-id="02dbd-126">When the sample app runs on the local machine in the Development environment, secrets are loaded from the local user secrets store.</span></span>

<span data-ttu-id="02dbd-127">Lo strumento di gestione dei segreti richiede una `<UserSecretsId>` proprietà nel file di progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-127">The Secret Manager tool requires a `<UserSecretsId>` property in the app's project file.</span></span> <span data-ttu-id="02dbd-128">Impostare il valore della proprietà ( `{GUID}` ) su un GUID univoco:</span><span class="sxs-lookup"><span data-stu-id="02dbd-128">Set the property value (`{GUID}`) to any unique GUID:</span></span>

```xml
<PropertyGroup>
  <UserSecretsId>{GUID}</UserSecretsId>
</PropertyGroup>
```

<span data-ttu-id="02dbd-129">I segreti vengono creati come coppie nome/valore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-129">Secrets are created as name-value pairs.</span></span> <span data-ttu-id="02dbd-130">I valori gerarchici (sezioni di configurazione) utilizzano un `:` (due punti) come separatore nei nomi delle chiavi di [configurazione ASP.NET Core](xref:fundamentals/configuration/index) .</span><span class="sxs-lookup"><span data-stu-id="02dbd-130">Hierarchical values (configuration sections) use a `:` (colon) as a separator in [ASP.NET Core configuration](xref:fundamentals/configuration/index) key names.</span></span>

<span data-ttu-id="02dbd-131">Il gestore del segreto viene usato da una shell dei comandi aperta alla [radice del contenuto](xref:fundamentals/index#content-root)del progetto, dove `{SECRET NAME}` è il nome e `{SECRET VALUE}` è il valore:</span><span class="sxs-lookup"><span data-stu-id="02dbd-131">The Secret Manager is used from a command shell opened to the project's [content root](xref:fundamentals/index#content-root), where `{SECRET NAME}` is the name and `{SECRET VALUE}` is the value:</span></span>

```dotnetcli
dotnet user-secrets set "{SECRET NAME}" "{SECRET VALUE}"
```

<span data-ttu-id="02dbd-132">Eseguire i comandi seguenti in una shell dei comandi dalla radice del [contenuto](xref:fundamentals/index#content-root) del progetto per impostare i segreti per l'app di esempio:</span><span class="sxs-lookup"><span data-stu-id="02dbd-132">Execute the following commands in a command shell from the project's [content root](xref:fundamentals/index#content-root) to set the secrets for the sample app:</span></span>

```dotnetcli
dotnet user-secrets set "SecretName" "secret_value_1_dev"
dotnet user-secrets set "Section:SecretName" "secret_value_2_dev"
```

<span data-ttu-id="02dbd-133">Quando questi segreti vengono archiviati in Azure Key Vault nell' [archiviazione segreta nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione, il `_dev` suffisso viene modificato in `_prod` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-133">When these secrets are stored in Azure Key Vault in the [Secret storage in the Production environment with Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) section, the `_dev` suffix is changed to `_prod`.</span></span> <span data-ttu-id="02dbd-134">Il suffisso fornisce un segnale visivo nell'output dell'app che indica l'origine dei valori di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-134">The suffix provides a visual cue in the app's output indicating the source of the configuration values.</span></span>

## <a name="secret-storage-in-the-production-environment-with-azure-key-vault"></a><span data-ttu-id="02dbd-135">Archiviazione segreta nell'ambiente di produzione con Azure Key Vault</span><span class="sxs-lookup"><span data-stu-id="02dbd-135">Secret storage in the Production environment with Azure Key Vault</span></span>

<span data-ttu-id="02dbd-136">Le istruzioni fornite nell'argomento [avvio rapido: impostare e recuperare un segreto da Azure Key Vault usando l'interfaccia](/azure/key-vault/quick-create-cli) della riga di comando di Azure sono riepilogate qui per la creazione di un Azure Key Vault e l'archiviazione di segreti usati dall'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="02dbd-136">The instructions provided by the [Quickstart: Set and retrieve a secret from Azure Key Vault using Azure CLI](/azure/key-vault/quick-create-cli) topic are summarized here for creating an Azure Key Vault and storing secrets used by the sample app.</span></span> <span data-ttu-id="02dbd-137">Per ulteriori informazioni, vedere l'argomento.</span><span class="sxs-lookup"><span data-stu-id="02dbd-137">Refer to the topic for further details.</span></span>

1. <span data-ttu-id="02dbd-138">Aprire Azure cloud shell usando uno dei metodi seguenti nel [portale di Azure](https://portal.azure.com/):</span><span class="sxs-lookup"><span data-stu-id="02dbd-138">Open Azure Cloud shell using any one of the following methods in the [Azure portal](https://portal.azure.com/):</span></span>

   * <span data-ttu-id="02dbd-139">Selezionare **Prova** nell'angolo superiore destro di un blocco di codice.</span><span class="sxs-lookup"><span data-stu-id="02dbd-139">Select **Try It** in the upper-right corner of a code block.</span></span> <span data-ttu-id="02dbd-140">Usare la stringa di ricerca "interfaccia della riga di comando di Azure" nella casella di testo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-140">Use the search string "Azure CLI" in the text box.</span></span>
   * <span data-ttu-id="02dbd-141">Aprire Cloud Shell nel browser con il pulsante **avvia cloud Shell** .</span><span class="sxs-lookup"><span data-stu-id="02dbd-141">Open Cloud Shell in your browser with the **Launch Cloud Shell** button.</span></span>
   * <span data-ttu-id="02dbd-142">Selezionare il pulsante **Cloud Shell** nel menu nell'angolo in alto a destra del portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-142">Select the **Cloud Shell** button on the menu in the upper-right corner of the Azure portal.</span></span>

   <span data-ttu-id="02dbd-143">Per altre informazioni, vedere l'interfaccia della riga di comando di [Azure](/cli/azure/) e [Panoramica di Azure cloud Shell](/azure/cloud-shell/overview).</span><span class="sxs-lookup"><span data-stu-id="02dbd-143">For more information, see [Azure CLI](/cli/azure/) and [Overview of Azure Cloud Shell](/azure/cloud-shell/overview).</span></span>

1. <span data-ttu-id="02dbd-144">Se non è già stato eseguito l'autenticazione, effettuare l'accesso con il `az login` comando.</span><span class="sxs-lookup"><span data-stu-id="02dbd-144">If you aren't already authenticated, sign in with the `az login` command.</span></span>

1. <span data-ttu-id="02dbd-145">Creare un gruppo di risorse con il comando seguente, dove `{RESOURCE GROUP NAME}` è il nome del gruppo di risorse per il nuovo gruppo di risorse e `{LOCATION}` è l'area di Azure (Datacenter):</span><span class="sxs-lookup"><span data-stu-id="02dbd-145">Create a resource group with the following command, where `{RESOURCE GROUP NAME}` is the resource group name for the new resource group and `{LOCATION}` is the Azure region (datacenter):</span></span>

   ```azurecli
   az group create --name "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. <span data-ttu-id="02dbd-146">Creare un insieme di credenziali delle chiavi nel gruppo di risorse con il comando seguente, dove è il nome del nuovo insieme di credenziali delle `{KEY VAULT NAME}` chiavi e `{LOCATION}` è l'area di Azure (Datacenter):</span><span class="sxs-lookup"><span data-stu-id="02dbd-146">Create a key vault in the resource group with the following command, where `{KEY VAULT NAME}` is the name for the new key vault and `{LOCATION}` is the Azure region (datacenter):</span></span>

   ```azurecli
   az keyvault create --name {KEY VAULT NAME} --resource-group "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. <span data-ttu-id="02dbd-147">Creare segreti nell'insieme di credenziali delle chiavi come coppie nome/valore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-147">Create secrets in the key vault as name-value pairs.</span></span>

   <span data-ttu-id="02dbd-148">Azure Key Vault nomi di segreto sono limitati a caratteri alfanumerici e trattini.</span><span class="sxs-lookup"><span data-stu-id="02dbd-148">Azure Key Vault secret names are limited to alphanumeric characters and dashes.</span></span> <span data-ttu-id="02dbd-149">I valori gerarchici (sezioni di configurazione) usano `--` (due trattini) come separatore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-149">Hierarchical values (configuration sections) use `--` (two dashes) as a separator.</span></span> <span data-ttu-id="02dbd-150">I due punti, che in genere vengono usati per delimitare una sezione da una sottochiave in [ASP.NET Core configurazione](xref:fundamentals/configuration/index), non sono consentiti nei nomi dei segreti di Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-150">Colons, which are normally used to delimit a section from a subkey in [ASP.NET Core configuration](xref:fundamentals/configuration/index), aren't allowed in key vault secret names.</span></span> <span data-ttu-id="02dbd-151">Pertanto, vengono usati due trattini e scambiati per i due punti quando i segreti vengono caricati nella configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-151">Therefore, two dashes are used and swapped for a colon when the secrets are loaded into the app's configuration.</span></span>

   <span data-ttu-id="02dbd-152">I segreti seguenti sono da usare con l'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="02dbd-152">The following secrets are for use with the sample app.</span></span> <span data-ttu-id="02dbd-153">I valori includono un `_prod` suffisso per distinguerli dai `_dev` valori dei suffissi caricati nell'ambiente di sviluppo da segreti utente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-153">The values include a `_prod` suffix to distinguish them from the `_dev` suffix values loaded in the Development environment from User Secrets.</span></span> <span data-ttu-id="02dbd-154">Sostituire `{KEY VAULT NAME}` con il nome dell'insieme di credenziali delle chiavi creato nel passaggio precedente:</span><span class="sxs-lookup"><span data-stu-id="02dbd-154">Replace `{KEY VAULT NAME}` with the name of the key vault that you created in the prior step:</span></span>

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "SecretName" --value "secret_value_1_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "Section--SecretName" --value "secret_value_2_prod"
   ```

## <a name="use-application-id-and-x509-certificate-for-non-azure-hosted-apps"></a><span data-ttu-id="02dbd-155">Usare l'ID applicazione e il certificato X. 509 per le app non ospitate in Azure</span><span class="sxs-lookup"><span data-stu-id="02dbd-155">Use Application ID and X.509 certificate for non-Azure-hosted apps</span></span>

<span data-ttu-id="02dbd-156">Configurare Azure AD, Azure Key Vault e l'app per usare un ID applicazione Azure Active Directory e un certificato X. 509 per l'autenticazione in un insieme di credenziali **delle chiavi quando l'app è ospitata all'esterno di Azure**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-156">Configure Azure AD, Azure Key Vault, and the app to use an Azure Active Directory Application ID and X.509 certificate to authenticate to a key vault **when the app is hosted outside of Azure**.</span></span> <span data-ttu-id="02dbd-157">Per i dettagli, vedere l'articolo relativo alle [informazioni su chiavi, segreti e certificati](/azure/key-vault/about-keys-secrets-and-certificates).</span><span class="sxs-lookup"><span data-stu-id="02dbd-157">For more information, see [About keys, secrets, and certificates](/azure/key-vault/about-keys-secrets-and-certificates).</span></span>

> [!NOTE]
> <span data-ttu-id="02dbd-158">Sebbene l'uso di un ID applicazione e di un certificato X. 509 sia supportato per le app ospitate in Azure, è consigliabile usare [identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) quando si ospita un'app in Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-158">Although using an Application ID and X.509 certificate is supported for apps hosted in Azure, we recommend using [Managed identities for Azure resources](#use-managed-identities-for-azure-resources) when hosting an app in Azure.</span></span> <span data-ttu-id="02dbd-159">Le identità gestite non richiedono l'archiviazione di un certificato nell'app o nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-159">Managed identities don't require storing a certificate in the app or in the development environment.</span></span>

<span data-ttu-id="02dbd-160">L'app di esempio usa un ID applicazione e un certificato X. 509 quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Certificate` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-160">The sample app uses an Application ID and X.509 certificate when the `#define` statement at the top of the *Program.cs* file is set to `Certificate`.</span></span>

1. <span data-ttu-id="02dbd-161">Creare un certificato di archivio PKCS # 12 (*. pfx*).</span><span class="sxs-lookup"><span data-stu-id="02dbd-161">Create a PKCS#12 archive (*.pfx*) certificate.</span></span> <span data-ttu-id="02dbd-162">Le opzioni per la creazione di certificati includono [Makecert in Windows](/windows/desktop/seccrypto/makecert) e [openssl](https://www.openssl.org/).</span><span class="sxs-lookup"><span data-stu-id="02dbd-162">Options for creating certificates include [MakeCert on Windows](/windows/desktop/seccrypto/makecert) and [OpenSSL](https://www.openssl.org/).</span></span>
1. <span data-ttu-id="02dbd-163">Installare il certificato nell'archivio certificati personale dell'utente corrente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-163">Install the certificate into the current user's personal certificate store.</span></span> <span data-ttu-id="02dbd-164">Contrassegnare la chiave come esportabile è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="02dbd-164">Marking the key as exportable is optional.</span></span> <span data-ttu-id="02dbd-165">Prendere nota dell'identificazione personale del certificato, che verrà usato più avanti in questo processo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-165">Note the certificate's thumbprint, which is used later in this process.</span></span>
1. <span data-ttu-id="02dbd-166">Esportare il certificato di archiviazione PKCS # 12 (*. pfx*) come certificato con codifica der (*. cer*).</span><span class="sxs-lookup"><span data-stu-id="02dbd-166">Export the PKCS#12 archive (*.pfx*) certificate as a DER-encoded certificate (*.cer*).</span></span>
1. <span data-ttu-id="02dbd-167">Registrare l'app con Azure AD (**registrazioni app**).</span><span class="sxs-lookup"><span data-stu-id="02dbd-167">Register the app with Azure AD (**App registrations**).</span></span>
1. <span data-ttu-id="02dbd-168">Caricare il certificato con codifica DER (*. cer*) in Azure ad:</span><span class="sxs-lookup"><span data-stu-id="02dbd-168">Upload the DER-encoded certificate (*.cer*) to Azure AD:</span></span>
   1. <span data-ttu-id="02dbd-169">Selezionare l'app in Azure AD.</span><span class="sxs-lookup"><span data-stu-id="02dbd-169">Select the app in Azure AD.</span></span>
   1. <span data-ttu-id="02dbd-170">Passare a **certificati & segreti**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-170">Navigate to **Certificates & secrets**.</span></span>
   1. <span data-ttu-id="02dbd-171">Selezionare **Carica certificato** per caricare il certificato, che contiene la chiave pubblica.</span><span class="sxs-lookup"><span data-stu-id="02dbd-171">Select **Upload certificate** to upload the certificate, which contains the public key.</span></span> <span data-ttu-id="02dbd-172">Un certificato con *estensione cer*, *PEM* o *CRT* è accettabile.</span><span class="sxs-lookup"><span data-stu-id="02dbd-172">A *.cer*, *.pem*, or *.crt* certificate is acceptable.</span></span>
1. <span data-ttu-id="02dbd-173">Archiviare il nome dell'insieme di credenziali delle chiavi, l'ID applicazione e l'identificazione personale del certificato nel file dell'app *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="02dbd-173">Store the key vault name, Application ID, and certificate thumbprint in the app's *appsettings.json* file.</span></span>
1. <span data-ttu-id="02dbd-174">Passare a insiemi di credenziali delle **chiavi** nella portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-174">Navigate to **Key vaults** in the Azure portal.</span></span>
1. <span data-ttu-id="02dbd-175">Selezionare l'insieme di credenziali delle chiavi creato nell' [Archivio Secret nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-175">Select the key vault that you created in the [Secret storage in the Production environment with Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) section.</span></span>
1. <span data-ttu-id="02dbd-176">Selezionare **Criteri di accesso**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-176">Select **Access policies**.</span></span>
1. <span data-ttu-id="02dbd-177">Selezionare **Aggiungi criterio di accesso**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-177">Select **Add Access Policy**.</span></span>
1. <span data-ttu-id="02dbd-178">Aprire **autorizzazioni segrete** e fornire all'app le autorizzazioni **Get** ed **List** .</span><span class="sxs-lookup"><span data-stu-id="02dbd-178">Open **Secret permissions** and provide the app with **Get** and **List** permissions.</span></span>
1. <span data-ttu-id="02dbd-179">Selezionare **Seleziona entità** e selezionare l'app registrata in base al nome.</span><span class="sxs-lookup"><span data-stu-id="02dbd-179">Select **Select principal** and select the registered app by name.</span></span> <span data-ttu-id="02dbd-180">Fare clic sul pulsante **Seleziona**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-180">Select the **Select** button.</span></span>
1. <span data-ttu-id="02dbd-181">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-181">Select **OK**.</span></span>
1. <span data-ttu-id="02dbd-182">Selezionare **Salva**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-182">Select **Save**.</span></span>
1. <span data-ttu-id="02dbd-183">Distribuire l'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-183">Deploy the app.</span></span>

<span data-ttu-id="02dbd-184">L' `Certificate` app di esempio ottiene i valori di configurazione da `IConfigurationRoot` con lo stesso nome del nome del segreto:</span><span class="sxs-lookup"><span data-stu-id="02dbd-184">The `Certificate` sample app obtains its configuration values from `IConfigurationRoot` with the same name as the secret name:</span></span>

* <span data-ttu-id="02dbd-185">Valori non gerarchici: il valore per `SecretName` viene ottenuto con `config["SecretName"]` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-185">Non-hierarchical values: The value for `SecretName` is obtained with `config["SecretName"]`.</span></span>
* <span data-ttu-id="02dbd-186">Valori gerarchici (sezioni): `:` notazione use (due punti) o `GetSection` metodo di estensione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-186">Hierarchical values (sections): Use `:` (colon) notation or the `GetSection` extension method.</span></span> <span data-ttu-id="02dbd-187">Per ottenere il valore di configurazione, usare uno di questi approcci:</span><span class="sxs-lookup"><span data-stu-id="02dbd-187">Use either of these approaches to obtain the configuration value:</span></span>
  * `config["Section:SecretName"]`
  * `config.GetSection("Section")["SecretName"]`

<span data-ttu-id="02dbd-188">Il certificato X. 509 è gestito dal sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-188">The X.509 certificate is managed by the OS.</span></span> <span data-ttu-id="02dbd-189">L'app chiama **AddAzureKeyVault** con i valori forniti dal *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="02dbd-189">The app calls **AddAzureKeyVault** with values supplied by the *appsettings.json* file:</span></span>

[!code-csharp[](key-vault-configuration/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=46-49)]

<span data-ttu-id="02dbd-190">Valori di esempio</span><span class="sxs-lookup"><span data-stu-id="02dbd-190">Example values:</span></span>

* <span data-ttu-id="02dbd-191">Nome dell'insieme di credenziali delle chiavi: `contosovault`</span><span class="sxs-lookup"><span data-stu-id="02dbd-191">Key vault name: `contosovault`</span></span>
* <span data-ttu-id="02dbd-192">ID applicazione: `627e911e-43cc-61d4-992e-12db9c81b413`</span><span class="sxs-lookup"><span data-stu-id="02dbd-192">Application ID: `627e911e-43cc-61d4-992e-12db9c81b413`</span></span>
* <span data-ttu-id="02dbd-193">Identificazione personale del certificato: `fe14593dd66b2406c5269d742d04b6e1ab03adb1`</span><span class="sxs-lookup"><span data-stu-id="02dbd-193">Certificate thumbprint: `fe14593dd66b2406c5269d742d04b6e1ab03adb1`</span></span>

<span data-ttu-id="02dbd-194">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="02dbd-194">*appsettings.json*:</span></span>

[!code-json[](key-vault-configuration/samples/3.x/SampleApp/appsettings.json?highlight=10-12)]

<span data-ttu-id="02dbd-195">Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati.</span><span class="sxs-lookup"><span data-stu-id="02dbd-195">When you run the app, a webpage shows the loaded secret values.</span></span> <span data-ttu-id="02dbd-196">Nell'ambiente di sviluppo, i valori Secret vengono caricati con il `_dev` suffisso.</span><span class="sxs-lookup"><span data-stu-id="02dbd-196">In the Development environment, secret values load with the `_dev` suffix.</span></span> <span data-ttu-id="02dbd-197">Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso.</span><span class="sxs-lookup"><span data-stu-id="02dbd-197">In the Production environment, the values load with the `_prod` suffix.</span></span>

## <a name="use-managed-identities-for-azure-resources"></a><span data-ttu-id="02dbd-198">Usare identità gestite per le risorse di Azure</span><span class="sxs-lookup"><span data-stu-id="02dbd-198">Use Managed identities for Azure resources</span></span>

<span data-ttu-id="02dbd-199">**Un'app distribuita in Azure** può trarre vantaggio dalle [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview), che consente all'app di eseguire l'autenticazione con Azure Key Vault usando Azure ad autenticazione senza credenziali (ID applicazione e password/segreto client) archiviate nell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-199">**An app deployed to Azure** can take advantage of [Managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview), which allows the app to authenticate with Azure Key Vault using Azure AD authentication without credentials (Application ID and Password/Client Secret) stored in the app.</span></span>

<span data-ttu-id="02dbd-200">L'app di esempio usa le identità gestite per le risorse di Azure quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Managed` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-200">The sample app uses Managed identities for Azure resources when the `#define` statement at the top of the *Program.cs* file is set to `Managed`.</span></span>

<span data-ttu-id="02dbd-201">Immettere il nome dell'insieme di credenziali nel *appsettings.json* file dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-201">Enter the vault name into the app's *appsettings.json* file.</span></span> <span data-ttu-id="02dbd-202">L'app di esempio non richiede un ID applicazione e una password (segreto client) quando è impostata sulla `Managed` versione, in modo che sia possibile ignorare tali voci di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-202">The sample app doesn't require an Application ID and Password (Client Secret) when set to the `Managed` version, so you can ignore those configuration entries.</span></span> <span data-ttu-id="02dbd-203">L'app viene distribuita in Azure e Azure autentica l'app per accedere Azure Key Vault solo usando il nome dell'insieme di credenziali archiviato nel *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="02dbd-203">The app is deployed to Azure, and Azure authenticates the app to access Azure Key Vault only using the vault name stored in the *appsettings.json* file.</span></span>

<span data-ttu-id="02dbd-204">Distribuire l'app di esempio nel servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-204">Deploy the sample app to Azure App Service.</span></span>

<span data-ttu-id="02dbd-205">Un'app distribuita nel servizio app Azure viene registrata automaticamente con Azure AD quando viene creato il servizio.</span><span class="sxs-lookup"><span data-stu-id="02dbd-205">An app deployed to Azure App Service is automatically registered with Azure AD when the service is created.</span></span> <span data-ttu-id="02dbd-206">Ottenere l'ID oggetto dalla distribuzione per utilizzarlo nel comando seguente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-206">Obtain the Object ID from the deployment for use in the following command.</span></span> <span data-ttu-id="02dbd-207">L'ID oggetto viene visualizzato nella portale di Azure nel **Identity** Pannello del servizio app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-207">The Object ID is shown in the Azure portal on the **Identity** panel of the App Service.</span></span>

<span data-ttu-id="02dbd-208">Usando l'interfaccia della riga di comando di Azure e l'ID oggetto dell'app, fornire all'app le `list` `get` autorizzazioni e per accedere all'insieme di credenziali delle chiavi:</span><span class="sxs-lookup"><span data-stu-id="02dbd-208">Using Azure CLI and the app's Object ID, provide the app with `list` and `get` permissions to access the key vault:</span></span>

```azurecli
az keyvault set-policy --name {KEY VAULT NAME} --object-id {OBJECT ID} --secret-permissions get list
```

<span data-ttu-id="02dbd-209">**Riavviare l'app** usando l'interfaccia della riga di comando di Azure, PowerShell o la portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-209">**Restart the app** using Azure CLI, PowerShell, or the Azure portal.</span></span>

<span data-ttu-id="02dbd-210">App di esempio:</span><span class="sxs-lookup"><span data-stu-id="02dbd-210">The sample app:</span></span>

* <span data-ttu-id="02dbd-211">Crea un'istanza della `DefaultAzureCredential` classe, la credenziale tenta di ottenere un token di accesso dall'ambiente per le risorse di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-211">Creates an instance of the `DefaultAzureCredential` class, the credential attempts to obtain an access token from environment for Azure resources.</span></span>
* <span data-ttu-id="02dbd-212">[`Azure.Security.KeyVault.Secrets.Secrets`](/dotnet/api/azure.security.keyvault.secrets)Viene creata una nuova istanza di con l' `DefaultAzureCredential` istanza di.</span><span class="sxs-lookup"><span data-stu-id="02dbd-212">A new [`Azure.Security.KeyVault.Secrets.Secrets`](/dotnet/api/azure.security.keyvault.secrets) is created with the `DefaultAzureCredential` instance.</span></span>
* <span data-ttu-id="02dbd-213">L' `Azure.Security.KeyVault.Secrets.Secrets` istanza viene utilizzata con un'implementazione predefinita di `Azure.Extensions.AspNetCore.Configuration.Secrets` che carica tutti i valori del segreto e sostituisce i doppi trattini ( `--` ) con i due punti ( `:` ) nei nomi delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-213">The `Azure.Security.KeyVault.Secrets.Secrets` instance is used with a default implementation of `Azure.Extensions.AspNetCore.Configuration.Secrets` that loads all secret values and replaces double-dashes (`--`) with colons (`:`) in key names.</span></span>

[!code-csharp[](key-vault-configuration/samples/3.x/SampleApp/Program.cs?name=snippet2&highlight=12-14)]

<span data-ttu-id="02dbd-214">Valore di esempio del nome dell'insieme di credenziali delle chiavi: `contosovault`</span><span class="sxs-lookup"><span data-stu-id="02dbd-214">Key vault name example value: `contosovault`</span></span>

<span data-ttu-id="02dbd-215">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="02dbd-215">*appsettings.json*:</span></span>

```json
{
  "KeyVaultName": "Key Vault Name"
}
```

<span data-ttu-id="02dbd-216">Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati.</span><span class="sxs-lookup"><span data-stu-id="02dbd-216">When you run the app, a webpage shows the loaded secret values.</span></span> <span data-ttu-id="02dbd-217">Nell'ambiente di sviluppo i valori Secret hanno il `_dev` suffisso perché sono forniti da segreti utente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-217">In the Development environment, secret values have the `_dev` suffix because they're provided by User Secrets.</span></span> <span data-ttu-id="02dbd-218">Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso perché sono forniti da Azure Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-218">In the Production environment, the values load with the `_prod` suffix because they're provided by Azure Key Vault.</span></span>

<span data-ttu-id="02dbd-219">Se viene visualizzato un `Access denied` errore, verificare che l'app sia registrata con Azure ad e che l'accesso sia stato fornito all'insieme di credenziali delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-219">If you receive an `Access denied` error, confirm that the app is registered with Azure AD and provided access to the key vault.</span></span> <span data-ttu-id="02dbd-220">Confermare di aver riavviato il servizio in Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-220">Confirm that you've restarted the service in Azure.</span></span>

<span data-ttu-id="02dbd-221">Per informazioni sull'uso del provider con un'identità gestita e una pipeline di Azure DevOps, vedere [creare una connessione del servizio Azure Resource Manager a una macchina virtuale con un'identità del servizio gestito](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity).</span><span class="sxs-lookup"><span data-stu-id="02dbd-221">For information on using the provider with a managed identity and an Azure DevOps pipeline, see [Create an Azure Resource Manager service connection to a VM with a managed service identity](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity).</span></span>

## <a name="configuration-options"></a><span data-ttu-id="02dbd-222">Opzioni di configurazione</span><span class="sxs-lookup"><span data-stu-id="02dbd-222">Configuration options</span></span>

<span data-ttu-id="02dbd-223">AddAzureKeyVault può accettare un AzureKeyVaultConfigurationOptions:</span><span class="sxs-lookup"><span data-stu-id="02dbd-223">AddAzureKeyVault can accept an AzureKeyVaultConfigurationOptions:</span></span>

```csharp
config.AddAzureKeyVault(new SecretClient(new URI("Your Key Vault Endpoint"), new DefaultAzureCredential()),
                        new AzureKeyVaultConfigurationOptions())
    {
        ...
    });
```

| <span data-ttu-id="02dbd-224">Proprietà</span><span class="sxs-lookup"><span data-stu-id="02dbd-224">Property</span></span>         | <span data-ttu-id="02dbd-225">Descrizione</span><span class="sxs-lookup"><span data-stu-id="02dbd-225">Description</span></span> |
| ---------------- | ----------- |
| `Manager`        | <span data-ttu-id="02dbd-226">`Azure.Extensions.AspNetCore.Configuration.Secrets` istanza di utilizzata per controllare il caricamento del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-226">`Azure.Extensions.AspNetCore.Configuration.Secrets` instance used to control secret loading.</span></span> |
| `ReloadInterval` | <span data-ttu-id="02dbd-227">`Timespan` per attendere tra i tentativi di polling dell'insieme di credenziali delle chiavi per le modifiche.</span><span class="sxs-lookup"><span data-stu-id="02dbd-227">`Timespan` to wait between attempts at polling the key vault for changes.</span></span> <span data-ttu-id="02dbd-228">Il valore predefinito è `null` (la configurazione non viene ricaricata).</span><span class="sxs-lookup"><span data-stu-id="02dbd-228">The default value is `null` (configuration isn't reloaded).</span></span> |

## <a name="use-a-key-name-prefix"></a><span data-ttu-id="02dbd-229">Usa prefisso nome chiave</span><span class="sxs-lookup"><span data-stu-id="02dbd-229">Use a key name prefix</span></span>

<span data-ttu-id="02dbd-230">AddAzureKeyVault fornisce un overload che accetta un'implementazione di `Azure.Extensions.AspNetCore.Configuration.Secrets` , che consente di controllare il modo in cui i segreti dell'insieme di credenziali delle chiavi vengono convertiti in chiavi di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-230">AddAzureKeyVault provides an overload that accepts an implementation of `Azure.Extensions.AspNetCore.Configuration.Secrets`, which allows you to control how key vault secrets are converted into configuration keys.</span></span> <span data-ttu-id="02dbd-231">Ad esempio, è possibile implementare l'interfaccia per caricare i valori dei segreti in base a un valore di prefisso fornito all'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-231">For example, you can implement the interface to load secret values based on a prefix value you provide at app startup.</span></span> <span data-ttu-id="02dbd-232">Questo consente, ad esempio, di caricare i segreti in base alla versione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-232">This allows you, for example, to load secrets based on the version of the app.</span></span>

> [!WARNING]
> <span data-ttu-id="02dbd-233">Non usare i prefissi sui segreti dell'insieme di credenziali delle chiavi per collocare i segreti per più app nello stesso insieme di credenziali delle chiavi o per collocare i segreti ambientali (ad esempio, *sviluppo* rispetto ai segreti di *produzione* ) nello stesso insieme di credenziali.</span><span class="sxs-lookup"><span data-stu-id="02dbd-233">Don't use prefixes on key vault secrets to place secrets for multiple apps into the same key vault or to place environmental secrets (for example, *development* versus *production* secrets) into the same vault.</span></span> <span data-ttu-id="02dbd-234">È consigliabile che app e ambienti di sviluppo/produzione diversi usino insiemi di credenziali delle chiavi distinti per isolare gli ambienti app per il massimo livello di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="02dbd-234">We recommend that different apps and development/production environments use separate key vaults to isolate app environments for the highest level of security.</span></span>

<span data-ttu-id="02dbd-235">Nell'esempio seguente viene stabilito un segreto nell'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti per l'ambiente di sviluppo) per `5000-AppSecret` (i periodi non sono consentiti nei nomi dei segreti dell'insieme di credenziali delle chiavi).</span><span class="sxs-lookup"><span data-stu-id="02dbd-235">In the following example, a secret is established in the key vault (and using the Secret Manager tool for the Development environment) for `5000-AppSecret` (periods aren't allowed in key vault secret names).</span></span> <span data-ttu-id="02dbd-236">Questo segreto rappresenta un segreto app per la versione 5.0.0.0 dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-236">This secret represents an app secret for version 5.0.0.0 of the app.</span></span> <span data-ttu-id="02dbd-237">Per un'altra versione dell'app, 5.1.0.0, viene aggiunto un segreto all'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti) per `5100-AppSecret` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-237">For another version of the app, 5.1.0.0, a secret is added to the key vault (and using the Secret Manager tool) for `5100-AppSecret`.</span></span> <span data-ttu-id="02dbd-238">Ogni versione dell'app carica il valore del segreto con versione nella configurazione come `AppSecret` , rimuovendo la versione durante il caricamento del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-238">Each app version loads its versioned secret value into its configuration as `AppSecret`, stripping off the version as it loads the secret.</span></span>

<span data-ttu-id="02dbd-239">AddAzureKeyVault viene chiamato con un oggetto personalizzato `Azure.Extensions.AspNetCore.Configuration.Secrets` :</span><span class="sxs-lookup"><span data-stu-id="02dbd-239">AddAzureKeyVault is called with a custom `Azure.Extensions.AspNetCore.Configuration.Secrets`:</span></span>

[!code-csharp[](key-vault-configuration/samples_snapshot/Program.cs)]

<span data-ttu-id="02dbd-240">L' `Azure.Extensions.AspNetCore.Configuration.Secrets` implementazione reagisce ai prefissi di versione dei segreti per caricare il segreto appropriato nella configurazione:</span><span class="sxs-lookup"><span data-stu-id="02dbd-240">The `Azure.Extensions.AspNetCore.Configuration.Secrets` implementation reacts to the version prefixes of secrets to load the proper secret into configuration:</span></span>

* <span data-ttu-id="02dbd-241">`Load` carica un segreto quando il nome inizia con il prefisso.</span><span class="sxs-lookup"><span data-stu-id="02dbd-241">`Load` loads a secret when its name starts with the prefix.</span></span> <span data-ttu-id="02dbd-242">Non vengono caricati altri segreti.</span><span class="sxs-lookup"><span data-stu-id="02dbd-242">Other secrets aren't loaded.</span></span>
* <span data-ttu-id="02dbd-243">`GetKey`:</span><span class="sxs-lookup"><span data-stu-id="02dbd-243">`GetKey`:</span></span>
  * <span data-ttu-id="02dbd-244">Rimuove il prefisso dal nome del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-244">Removes the prefix from the secret name.</span></span>
  * <span data-ttu-id="02dbd-245">Sostituisce due trattini in qualsiasi nome con `KeyDelimiter` , che è il delimitatore usato nella configurazione (in genere i due punti).</span><span class="sxs-lookup"><span data-stu-id="02dbd-245">Replaces two dashes in any name with the `KeyDelimiter`, which is the delimiter used in configuration (usually a colon).</span></span> <span data-ttu-id="02dbd-246">Azure Key Vault non consente i due punti nei nomi dei segreti.</span><span class="sxs-lookup"><span data-stu-id="02dbd-246">Azure Key Vault doesn't allow a colon in secret names.</span></span>

[!code-csharp[](key-vault-configuration/samples_snapshot/Startup.cs)]

<span data-ttu-id="02dbd-247">Il `Load` metodo viene chiamato da un algoritmo del provider che scorre i segreti dell'insieme di credenziali per trovare quelli con il prefisso della versione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-247">The `Load` method is called by a provider algorithm that iterates through the vault secrets to find the ones that have the version prefix.</span></span> <span data-ttu-id="02dbd-248">Quando viene trovato un prefisso di versione con `Load` , l'algoritmo usa il `GetKey` metodo per restituire il nome della configurazione del nome del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-248">When a version prefix is found with `Load`, the algorithm uses the `GetKey` method to return the configuration name of the secret name.</span></span> <span data-ttu-id="02dbd-249">Rimuove il prefisso della versione dal nome del segreto e restituisce il resto del nome del segreto per il caricamento nelle coppie nome-valore della configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-249">It strips off the version prefix from the secret's name and returns the rest of the secret name for loading into the app's configuration name-value pairs.</span></span>

<span data-ttu-id="02dbd-250">Quando viene implementato questo approccio:</span><span class="sxs-lookup"><span data-stu-id="02dbd-250">When this approach is implemented:</span></span>

1. <span data-ttu-id="02dbd-251">La versione dell'app specificata nel file di progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-251">The app's version specified in the app's project file.</span></span> <span data-ttu-id="02dbd-252">Nell'esempio seguente la versione dell'app è impostata su `5.0.0.0` :</span><span class="sxs-lookup"><span data-stu-id="02dbd-252">In the following example, the app's version is set to `5.0.0.0`:</span></span>

   ```xml
   <PropertyGroup>
     <Version>5.0.0.0</Version>
   </PropertyGroup>
   ```

1. <span data-ttu-id="02dbd-253">Verificare che `<UserSecretsId>` sia presente una proprietà nel file di progetto dell'app, dove `{GUID}` è un GUID fornito dall'utente:</span><span class="sxs-lookup"><span data-stu-id="02dbd-253">Confirm that a `<UserSecretsId>` property is present in the app's project file, where `{GUID}` is a user-supplied GUID:</span></span>

   ```xml
   <PropertyGroup>
     <UserSecretsId>{GUID}</UserSecretsId>
   </PropertyGroup>
   ```

   <span data-ttu-id="02dbd-254">Salvare i segreti seguenti localmente con lo [strumento di gestione dei segreti](xref:security/app-secrets):</span><span class="sxs-lookup"><span data-stu-id="02dbd-254">Save the following secrets locally with the [Secret Manager tool](xref:security/app-secrets):</span></span>

   ```dotnetcli
   dotnet user-secrets set "5000-AppSecret" "5.0.0.0_secret_value_dev"
   dotnet user-secrets set "5100-AppSecret" "5.1.0.0_secret_value_dev"
   ```

1. <span data-ttu-id="02dbd-255">I segreti vengono salvati in Azure Key Vault usando i comandi dell'interfaccia della riga di comando di Azure seguenti:</span><span class="sxs-lookup"><span data-stu-id="02dbd-255">Secrets are saved in Azure Key Vault using the following Azure CLI commands:</span></span>

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5000-AppSecret" --value "5.0.0.0_secret_value_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5100-AppSecret" --value "5.1.0.0_secret_value_prod"
   ```

1. <span data-ttu-id="02dbd-256">Quando l'app viene eseguita, vengono caricati i segreti dell'insieme di credenziali delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-256">When the app is run, the key vault secrets are loaded.</span></span> <span data-ttu-id="02dbd-257">La stringa segreta per `5000-AppSecret` viene confrontata con la versione dell'app specificata nel file di progetto dell'app ( `5.0.0.0` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-257">The string secret for `5000-AppSecret` is matched to the app's version specified in the app's project file (`5.0.0.0`).</span></span>

1. <span data-ttu-id="02dbd-258">La versione `5000` (con il trattino) viene rimossa dal nome della chiave.</span><span class="sxs-lookup"><span data-stu-id="02dbd-258">The version, `5000` (with the dash), is stripped from the key name.</span></span> <span data-ttu-id="02dbd-259">In tutta l'app, la lettura della configurazione con la chiave `AppSecret` carica il valore del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-259">Throughout the app, reading configuration with the key `AppSecret` loads the secret value.</span></span>

1. <span data-ttu-id="02dbd-260">Se la versione dell'app viene modificata nel file di progetto in `5.1.0.0` e l'app viene nuovamente eseguita, il valore del segreto restituito è `5.1.0.0_secret_value_dev` nell'ambiente di sviluppo e `5.1.0.0_secret_value_prod` in produzione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-260">If the app's version is changed in the project file to `5.1.0.0` and the app is run again, the secret value returned is `5.1.0.0_secret_value_dev` in the Development environment and `5.1.0.0_secret_value_prod` in Production.</span></span>

> [!NOTE]
> <span data-ttu-id="02dbd-261">È anche possibile fornire un' <xref:Azure.Security.KeyVault.Secrets.SecretClient> implementazione personalizzata a AddAzureKeyVault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-261">You can also provide your own <xref:Azure.Security.KeyVault.Secrets.SecretClient> implementation to AddAzureKeyVault.</span></span> <span data-ttu-id="02dbd-262">Un client personalizzato consente la condivisione di una singola istanza del client nell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-262">A custom client permits sharing a single instance of the client across the app.</span></span>

## <a name="bind-an-array-to-a-class"></a><span data-ttu-id="02dbd-263">Associare una matrice a una classe</span><span class="sxs-lookup"><span data-stu-id="02dbd-263">Bind an array to a class</span></span>

<span data-ttu-id="02dbd-264">Il provider è in grado di leggere i valori di configurazione in una matrice per l'associazione a una matrice POCO.</span><span class="sxs-lookup"><span data-stu-id="02dbd-264">The provider is capable of reading configuration values into an array for binding to a POCO array.</span></span>

<span data-ttu-id="02dbd-265">Quando si legge da un'origine di configurazione che consente alle chiavi di contenere separatori di due punti ( `:` ), viene usato un segmento di chiave numerico per distinguere le chiavi che costituiscono una matrice ( `:0:` , `:1:` , &hellip; `:{n}:` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-265">When reading from a configuration source that allows keys to contain colon (`:`) separators, a numeric key segment is used to distinguish the keys that make up an array (`:0:`, `:1:`, &hellip; `:{n}:`).</span></span> <span data-ttu-id="02dbd-266">Per altre informazioni, vedere [Configuration: associare una matrice a una classe](xref:fundamentals/configuration/index#bind-an-array-to-a-class).</span><span class="sxs-lookup"><span data-stu-id="02dbd-266">For more information, see [Configuration: Bind an array to a class](xref:fundamentals/configuration/index#bind-an-array-to-a-class).</span></span>

<span data-ttu-id="02dbd-267">Azure Key Vault chiavi non possono utilizzare i due punti come separatore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-267">Azure Key Vault keys can't use a colon as a separator.</span></span> <span data-ttu-id="02dbd-268">L'approccio descritto in questo argomento USA trattini doppi ( `--` ) come separatore per i valori gerarchici (sezioni).</span><span class="sxs-lookup"><span data-stu-id="02dbd-268">The approach described in this topic uses double dashes (`--`) as a separator for hierarchical values (sections).</span></span> <span data-ttu-id="02dbd-269">Le chiavi di matrice vengono archiviate in Azure Key Vault con trattini doppi e segmenti di chiave numerica ( `--0--` , `--1--` , &hellip; `--{n}--` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-269">Array keys are stored in Azure Key Vault with double dashes and numeric key segments (`--0--`, `--1--`, &hellip; `--{n}--`).</span></span>

<span data-ttu-id="02dbd-270">Esaminare la seguente configurazione del provider di registrazione [Serilog](https://serilog.net/) fornita da un file JSON.</span><span class="sxs-lookup"><span data-stu-id="02dbd-270">Examine the following [Serilog](https://serilog.net/) logging provider configuration provided by a JSON file.</span></span> <span data-ttu-id="02dbd-271">Nella matrice sono definiti due valori letterali di oggetto `WriteTo` che riflettono due *sink* Serilog, che descrivono le destinazioni per la registrazione dell'output:</span><span class="sxs-lookup"><span data-stu-id="02dbd-271">There are two object literals defined in the `WriteTo` array that reflect two Serilog *sinks*, which describe destinations for logging output:</span></span>

```json
"Serilog": {
  "WriteTo": [
    {
      "Name": "AzureTableStorage",
      "Args": {
        "storageTableName": "logs",
        "connectionString": "DefaultEnd...ountKey=Eby8...GMGw=="
      }
    },
    {
      "Name": "AzureDocumentDB",
      "Args": {
        "endpointUrl": "https://contoso.documents.azure.com:443",
        "authorizationKey": "Eby8...GMGw=="
      }
    }
  ]
}
```

<span data-ttu-id="02dbd-272">La configurazione mostrata nel file JSON precedente viene archiviata in Azure Key Vault usando la notazione a doppio trattino ( `--` ) e i segmenti numerici:</span><span class="sxs-lookup"><span data-stu-id="02dbd-272">The configuration shown in the preceding JSON file is stored in Azure Key Vault using double dash (`--`) notation and numeric segments:</span></span>

| <span data-ttu-id="02dbd-273">Chiave</span><span class="sxs-lookup"><span data-stu-id="02dbd-273">Key</span></span> | <span data-ttu-id="02dbd-274">Valore</span><span class="sxs-lookup"><span data-stu-id="02dbd-274">Value</span></span> |
| --- | ----- |
| `Serilog--WriteTo--0--Name` | `AzureTableStorage` |
| `Serilog--WriteTo--0--Args--storageTableName` | `logs` |
| `Serilog--WriteTo--0--Args--connectionString` | `DefaultEnd...ountKey=Eby8...GMGw==` |
| `Serilog--WriteTo--1--Name` | `AzureDocumentDB` |
| `Serilog--WriteTo--1--Args--endpointUrl` | `https://contoso.documents.azure.com:443` |
| `Serilog--WriteTo--1--Args--authorizationKey` | `Eby8...GMGw==` |

## <a name="reload-secrets"></a><span data-ttu-id="02dbd-275">Ricarica segreti</span><span class="sxs-lookup"><span data-stu-id="02dbd-275">Reload secrets</span></span>

<span data-ttu-id="02dbd-276">I segreti vengono memorizzati nella cache finché non `IConfigurationRoot.Reload()` viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="02dbd-276">Secrets are cached until `IConfigurationRoot.Reload()` is called.</span></span> <span data-ttu-id="02dbd-277">I segreti scaduti, disabilitati e aggiornati nell'insieme di credenziali delle chiavi non vengono rispettati dall'app fino a quando non `Reload` viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="02dbd-277">Expired, disabled, and updated secrets in the key vault are not respected by the app until `Reload` is executed.</span></span>

```csharp
Configuration.Reload();
```

## <a name="disabled-and-expired-secrets"></a><span data-ttu-id="02dbd-278">Segreti disabilitati e scaduti</span><span class="sxs-lookup"><span data-stu-id="02dbd-278">Disabled and expired secrets</span></span>

<span data-ttu-id="02dbd-279">I segreti disabilitati e scaduti generano un' <xref:Azure.RequestFailedException> .</span><span class="sxs-lookup"><span data-stu-id="02dbd-279">Disabled and expired secrets throw a <xref:Azure.RequestFailedException>.</span></span> <span data-ttu-id="02dbd-280">Per impedire che l'app venga generata, fornire la configurazione usando un provider di configurazione diverso o aggiornare il segreto disabilitato o scaduto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-280">To prevent the app from throwing, provide the configuration using a different configuration provider or update the disabled or expired secret.</span></span>

## <a name="troubleshoot"></a><span data-ttu-id="02dbd-281">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="02dbd-281">Troubleshoot</span></span>

<span data-ttu-id="02dbd-282">Quando l'app non riesce a caricare la configurazione usando il provider, viene scritto un messaggio di errore nell' [infrastruttura di registrazione ASP.NET Core](xref:fundamentals/logging/index).</span><span class="sxs-lookup"><span data-stu-id="02dbd-282">When the app fails to load configuration using the provider, an error message is written to the [ASP.NET Core Logging infrastructure](xref:fundamentals/logging/index).</span></span> <span data-ttu-id="02dbd-283">Le condizioni seguenti impediranno il caricamento della configurazione:</span><span class="sxs-lookup"><span data-stu-id="02dbd-283">The following conditions will prevent configuration from loading:</span></span>

* <span data-ttu-id="02dbd-284">L'app o il certificato non è configurato correttamente in Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="02dbd-284">The app or certificate isn't configured correctly in Azure Active Directory.</span></span>
* <span data-ttu-id="02dbd-285">L'insieme di credenziali delle chiavi non esiste in Azure Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-285">The key vault doesn't exist in Azure Key Vault.</span></span>
* <span data-ttu-id="02dbd-286">L'app non è autorizzata ad accedere all'insieme di credenziali delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-286">The app isn't authorized to access the key vault.</span></span>
* <span data-ttu-id="02dbd-287">Il criterio di accesso non include le `Get` `List` autorizzazioni e.</span><span class="sxs-lookup"><span data-stu-id="02dbd-287">The access policy doesn't include `Get` and `List` permissions.</span></span>
* <span data-ttu-id="02dbd-288">Nell'insieme di credenziali delle chiavi i dati di configurazione (coppia nome-valore) sono denominati, mancanti, disabilitati o scaduti in modo errato.</span><span class="sxs-lookup"><span data-stu-id="02dbd-288">In the key vault, the configuration data (name-value pair) is incorrectly named, missing, disabled, or expired.</span></span>
* <span data-ttu-id="02dbd-289">L'app presenta il nome dell'insieme di credenziali delle chiavi ( `KeyVaultName` ), Azure ad ID applicazione () o l'identificazione `AzureADApplicationId` personale del certificato Azure ad ( `AzureADCertThumbprint` ) o Azure ad DirectoryId ( `AzureADDirectoryId` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-289">The app has the wrong key vault name (`KeyVaultName`), Azure AD Application Id (`AzureADApplicationId`), or Azure AD certificate thumbprint (`AzureADCertThumbprint`), or Azure AD DirectoryId (`AzureADDirectoryId`).</span></span>
* <span data-ttu-id="02dbd-290">La chiave di configurazione (nome) non è corretta nell'app per il valore che si sta provando a caricare.</span><span class="sxs-lookup"><span data-stu-id="02dbd-290">The configuration key (name) is incorrect in the app for the value you're trying to load.</span></span>
* <span data-ttu-id="02dbd-291">Quando si aggiungono i criteri di accesso per l'app all'insieme di credenziali delle chiavi, il criterio è stato creato, ma il pulsante **Salva** non è stato selezionato nell'interfaccia utente dei **criteri di accesso** .</span><span class="sxs-lookup"><span data-stu-id="02dbd-291">When adding the access policy for the app to the key vault, the policy was created, but the **Save** button wasn't selected in the **Access policies** UI.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="02dbd-292">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="02dbd-292">Additional resources</span></span>

* <xref:fundamentals/configuration/index>
* [<span data-ttu-id="02dbd-293">Microsoft Azure: Key Vault</span><span class="sxs-lookup"><span data-stu-id="02dbd-293">Microsoft Azure: Key Vault</span></span>](https://azure.microsoft.com/services/key-vault/)
* [<span data-ttu-id="02dbd-294">Microsoft Azure: Key Vault documentazione</span><span class="sxs-lookup"><span data-stu-id="02dbd-294">Microsoft Azure: Key Vault Documentation</span></span>](/azure/key-vault/)
* [<span data-ttu-id="02dbd-295">Come generare e trasferire chiavi HSM protette per Azure Key Vault</span><span class="sxs-lookup"><span data-stu-id="02dbd-295">How to generate and transfer HSM-protected keys for Azure Key Vault</span></span>](/azure/key-vault/key-vault-hsm-protected-keys)
* [<span data-ttu-id="02dbd-296">Classe KeyVaultClient</span><span class="sxs-lookup"><span data-stu-id="02dbd-296">KeyVaultClient Class</span></span>](/dotnet/api/microsoft.azure.keyvault.keyvaultclient)
* [<span data-ttu-id="02dbd-297">Guida introduttiva: Impostare e recuperare un segreto da Azure Key Vault tramite un'app Web .NET</span><span class="sxs-lookup"><span data-stu-id="02dbd-297">Quickstart: Set and retrieve a secret from Azure Key Vault by using a .NET web app</span></span>](/azure/key-vault/quick-create-net)
* [<span data-ttu-id="02dbd-298">Esercitazione: Come usare Azure Key Vault con una macchina virtuale Windows di Azure in .NET</span><span class="sxs-lookup"><span data-stu-id="02dbd-298">Tutorial: How to use Azure Key Vault with Azure Windows Virtual Machine in .NET</span></span>](/azure/key-vault/tutorial-net-windows-virtual-machine)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="02dbd-299">Questo documento illustra come usare il provider di configurazione [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/) per caricare i valori di configurazione dell'app da Azure Key Vault segreti.</span><span class="sxs-lookup"><span data-stu-id="02dbd-299">This document explains how to use the [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/) Configuration Provider to load app configuration values from Azure Key Vault secrets.</span></span> <span data-ttu-id="02dbd-300">Azure Key Vault è un servizio basato sul cloud che aiuta a proteggere le chiavi crittografiche e i segreti usati da app e servizi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-300">Azure Key Vault is a cloud-based service that assists in safeguarding cryptographic keys and secrets used by apps and services.</span></span> <span data-ttu-id="02dbd-301">Gli scenari comuni per l'uso di Azure Key Vault con le app ASP.NET Core includono:</span><span class="sxs-lookup"><span data-stu-id="02dbd-301">Common scenarios for using Azure Key Vault with ASP.NET Core apps include:</span></span>

* <span data-ttu-id="02dbd-302">Controllo dell'accesso ai dati di configurazione riservati.</span><span class="sxs-lookup"><span data-stu-id="02dbd-302">Controlling access to sensitive configuration data.</span></span>
* <span data-ttu-id="02dbd-303">Soddisfare i requisiti per i moduli di protezione hardware convalidati FIPS 140-2 Level 2 (HSM) quando si archiviano i dati di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-303">Meeting the requirement for FIPS 140-2 Level 2 validated Hardware Security Modules (HSM's) when storing configuration data.</span></span>

<span data-ttu-id="02dbd-304">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/key-vault-configuration/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="02dbd-304">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/key-vault-configuration/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="packages"></a><span data-ttu-id="02dbd-305">Pacchetti</span><span class="sxs-lookup"><span data-stu-id="02dbd-305">Packages</span></span>

<span data-ttu-id="02dbd-306">Aggiungere un riferimento al pacchetto [Microsoft.Extensions.Configuration. ](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/) Pacchetto di AzureKeyVault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-306">Add a package reference to the [Microsoft.Extensions.Configuration.AzureKeyVault](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/) package.</span></span>

## <a name="sample-app"></a><span data-ttu-id="02dbd-307">App di esempio</span><span class="sxs-lookup"><span data-stu-id="02dbd-307">Sample app</span></span>

<span data-ttu-id="02dbd-308">L'app di esempio viene eseguita in una delle due modalità determinate dall' `#define` istruzione all'inizio del file *Program.cs* :</span><span class="sxs-lookup"><span data-stu-id="02dbd-308">The sample app runs in either of two modes determined by the `#define` statement at the top of the *Program.cs* file:</span></span>

* <span data-ttu-id="02dbd-309">`Certificate`: Illustra l'uso di un ID client Azure Key Vault e di un certificato X. 509 per accedere ai segreti archiviati nel Azure Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-309">`Certificate`: Demonstrates the use of an Azure Key Vault Client ID and X.509 certificate to access secrets stored in Azure Key Vault.</span></span> <span data-ttu-id="02dbd-310">Questa versione dell'esempio può essere eseguita da qualsiasi posizione, distribuita nel servizio app Azure o in qualsiasi host in grado di servire un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="02dbd-310">This version of the sample can be run from any location, deployed to Azure App Service or any host capable of serving an ASP.NET Core app.</span></span>
* <span data-ttu-id="02dbd-311">`Managed`: Illustra come usare le [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview) per autenticare l'app Azure Key Vault con Azure ad autenticazione senza credenziali archiviate nel codice o nella configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-311">`Managed`: Demonstrates how to use [Managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview) to authenticate the app to Azure Key Vault with Azure AD authentication without credentials stored in the app's code or configuration.</span></span> <span data-ttu-id="02dbd-312">Quando si usano identità gestite per l'autenticazione, non è necessario un Azure AD ID applicazione e una password (segreto client).</span><span class="sxs-lookup"><span data-stu-id="02dbd-312">When using managed identities to authenticate, an Azure AD Application ID and Password (Client Secret) aren't required.</span></span> <span data-ttu-id="02dbd-313">La `Managed` versione dell'esempio deve essere distribuita in Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-313">The `Managed` version of the sample must be deployed to Azure.</span></span> <span data-ttu-id="02dbd-314">Seguire le istruzioni riportate nella sezione [usare le identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) .</span><span class="sxs-lookup"><span data-stu-id="02dbd-314">Follow the guidance in the [Use the Managed identities for Azure resources](#use-managed-identities-for-azure-resources) section.</span></span>

<span data-ttu-id="02dbd-315">Per ulteriori informazioni su come configurare un'app di esempio utilizzando le direttive per il preprocessore ( `#define` ), vedere <xref:index#preprocessor-directives-in-sample-code> .</span><span class="sxs-lookup"><span data-stu-id="02dbd-315">For more information on how to configure a sample app using preprocessor directives (`#define`), see <xref:index#preprocessor-directives-in-sample-code>.</span></span>

## <a name="secret-storage-in-the-development-environment"></a><span data-ttu-id="02dbd-316">Archiviazione segreta nell'ambiente di sviluppo</span><span class="sxs-lookup"><span data-stu-id="02dbd-316">Secret storage in the Development environment</span></span>

<span data-ttu-id="02dbd-317">Impostare i segreti in locale usando lo [strumento di gestione dei segreti](xref:security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="02dbd-317">Set secrets locally using the [Secret Manager tool](xref:security/app-secrets).</span></span> <span data-ttu-id="02dbd-318">Quando l'app di esempio viene eseguita nel computer locale nell'ambiente di sviluppo, i segreti vengono caricati dall'archivio dei segreti dell'utente locale.</span><span class="sxs-lookup"><span data-stu-id="02dbd-318">When the sample app runs on the local machine in the Development environment, secrets are loaded from the local user secrets store.</span></span>

<span data-ttu-id="02dbd-319">Lo strumento di gestione dei segreti richiede una `<UserSecretsId>` proprietà nel file di progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-319">The Secret Manager tool requires a `<UserSecretsId>` property in the app's project file.</span></span> <span data-ttu-id="02dbd-320">Impostare il valore della proprietà ( `{GUID}` ) su un GUID univoco:</span><span class="sxs-lookup"><span data-stu-id="02dbd-320">Set the property value (`{GUID}`) to any unique GUID:</span></span>

```xml
<PropertyGroup>
  <UserSecretsId>{GUID}</UserSecretsId>
</PropertyGroup>
```

<span data-ttu-id="02dbd-321">I segreti vengono creati come coppie nome/valore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-321">Secrets are created as name-value pairs.</span></span> <span data-ttu-id="02dbd-322">I valori gerarchici (sezioni di configurazione) utilizzano un `:` (due punti) come separatore nei nomi delle chiavi di [configurazione ASP.NET Core](xref:fundamentals/configuration/index) .</span><span class="sxs-lookup"><span data-stu-id="02dbd-322">Hierarchical values (configuration sections) use a `:` (colon) as a separator in [ASP.NET Core configuration](xref:fundamentals/configuration/index) key names.</span></span>

<span data-ttu-id="02dbd-323">Il gestore del segreto viene usato da una shell dei comandi aperta alla [radice del contenuto](xref:fundamentals/index#content-root)del progetto, dove `{SECRET NAME}` è il nome e `{SECRET VALUE}` è il valore:</span><span class="sxs-lookup"><span data-stu-id="02dbd-323">The Secret Manager is used from a command shell opened to the project's [content root](xref:fundamentals/index#content-root), where `{SECRET NAME}` is the name and `{SECRET VALUE}` is the value:</span></span>

```dotnetcli
dotnet user-secrets set "{SECRET NAME}" "{SECRET VALUE}"
```

<span data-ttu-id="02dbd-324">Eseguire i comandi seguenti in una shell dei comandi dalla radice del [contenuto](xref:fundamentals/index#content-root) del progetto per impostare i segreti per l'app di esempio:</span><span class="sxs-lookup"><span data-stu-id="02dbd-324">Execute the following commands in a command shell from the project's [content root](xref:fundamentals/index#content-root) to set the secrets for the sample app:</span></span>

```dotnetcli
dotnet user-secrets set "SecretName" "secret_value_1_dev"
dotnet user-secrets set "Section:SecretName" "secret_value_2_dev"
```

<span data-ttu-id="02dbd-325">Quando questi segreti vengono archiviati in Azure Key Vault nell' [archiviazione segreta nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione, il `_dev` suffisso viene modificato in `_prod` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-325">When these secrets are stored in Azure Key Vault in the [Secret storage in the Production environment with Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) section, the `_dev` suffix is changed to `_prod`.</span></span> <span data-ttu-id="02dbd-326">Il suffisso fornisce un segnale visivo nell'output dell'app che indica l'origine dei valori di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-326">The suffix provides a visual cue in the app's output indicating the source of the configuration values.</span></span>

## <a name="secret-storage-in-the-production-environment-with-azure-key-vault"></a><span data-ttu-id="02dbd-327">Archiviazione segreta nell'ambiente di produzione con Azure Key Vault</span><span class="sxs-lookup"><span data-stu-id="02dbd-327">Secret storage in the Production environment with Azure Key Vault</span></span>

<span data-ttu-id="02dbd-328">Le istruzioni fornite nell'argomento [avvio rapido: impostare e recuperare un segreto da Azure Key Vault usando l'interfaccia](/azure/key-vault/quick-create-cli) della riga di comando di Azure sono riepilogate qui per la creazione di un Azure Key Vault e l'archiviazione di segreti usati dall'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="02dbd-328">The instructions provided by the [Quickstart: Set and retrieve a secret from Azure Key Vault using Azure CLI](/azure/key-vault/quick-create-cli) topic are summarized here for creating an Azure Key Vault and storing secrets used by the sample app.</span></span> <span data-ttu-id="02dbd-329">Per ulteriori informazioni, vedere l'argomento.</span><span class="sxs-lookup"><span data-stu-id="02dbd-329">Refer to the topic for further details.</span></span>

1. <span data-ttu-id="02dbd-330">Aprire Azure cloud shell usando uno dei metodi seguenti nel [portale di Azure](https://portal.azure.com/):</span><span class="sxs-lookup"><span data-stu-id="02dbd-330">Open Azure Cloud shell using any one of the following methods in the [Azure portal](https://portal.azure.com/):</span></span>

   * <span data-ttu-id="02dbd-331">Selezionare **Prova** nell'angolo superiore destro di un blocco di codice.</span><span class="sxs-lookup"><span data-stu-id="02dbd-331">Select **Try It** in the upper-right corner of a code block.</span></span> <span data-ttu-id="02dbd-332">Usare la stringa di ricerca "interfaccia della riga di comando di Azure" nella casella di testo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-332">Use the search string "Azure CLI" in the text box.</span></span>
   * <span data-ttu-id="02dbd-333">Aprire Cloud Shell nel browser con il pulsante **avvia cloud Shell** .</span><span class="sxs-lookup"><span data-stu-id="02dbd-333">Open Cloud Shell in your browser with the **Launch Cloud Shell** button.</span></span>
   * <span data-ttu-id="02dbd-334">Selezionare il pulsante **Cloud Shell** nel menu nell'angolo in alto a destra del portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-334">Select the **Cloud Shell** button on the menu in the upper-right corner of the Azure portal.</span></span>

   <span data-ttu-id="02dbd-335">Per altre informazioni, vedere l'interfaccia della riga di comando di [Azure](/cli/azure/) e [Panoramica di Azure cloud Shell](/azure/cloud-shell/overview).</span><span class="sxs-lookup"><span data-stu-id="02dbd-335">For more information, see [Azure CLI](/cli/azure/) and [Overview of Azure Cloud Shell](/azure/cloud-shell/overview).</span></span>

1. <span data-ttu-id="02dbd-336">Se non è già stato eseguito l'autenticazione, effettuare l'accesso con il `az login` comando.</span><span class="sxs-lookup"><span data-stu-id="02dbd-336">If you aren't already authenticated, sign in with the `az login` command.</span></span>

1. <span data-ttu-id="02dbd-337">Creare un gruppo di risorse con il comando seguente, dove `{RESOURCE GROUP NAME}` è il nome del gruppo di risorse per il nuovo gruppo di risorse e `{LOCATION}` è l'area di Azure (Datacenter):</span><span class="sxs-lookup"><span data-stu-id="02dbd-337">Create a resource group with the following command, where `{RESOURCE GROUP NAME}` is the resource group name for the new resource group and `{LOCATION}` is the Azure region (datacenter):</span></span>

   ```azurecli
   az group create --name "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. <span data-ttu-id="02dbd-338">Creare un insieme di credenziali delle chiavi nel gruppo di risorse con il comando seguente, dove è il nome del nuovo insieme di credenziali delle `{KEY VAULT NAME}` chiavi e `{LOCATION}` è l'area di Azure (Datacenter):</span><span class="sxs-lookup"><span data-stu-id="02dbd-338">Create a key vault in the resource group with the following command, where `{KEY VAULT NAME}` is the name for the new key vault and `{LOCATION}` is the Azure region (datacenter):</span></span>

   ```azurecli
   az keyvault create --name {KEY VAULT NAME} --resource-group "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. <span data-ttu-id="02dbd-339">Creare segreti nell'insieme di credenziali delle chiavi come coppie nome/valore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-339">Create secrets in the key vault as name-value pairs.</span></span>

   <span data-ttu-id="02dbd-340">Azure Key Vault nomi di segreto sono limitati a caratteri alfanumerici e trattini.</span><span class="sxs-lookup"><span data-stu-id="02dbd-340">Azure Key Vault secret names are limited to alphanumeric characters and dashes.</span></span> <span data-ttu-id="02dbd-341">I valori gerarchici (sezioni di configurazione) usano `--` (due trattini) come separatore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-341">Hierarchical values (configuration sections) use `--` (two dashes) as a separator.</span></span> <span data-ttu-id="02dbd-342">I due punti, che in genere vengono usati per delimitare una sezione da una sottochiave in [ASP.NET Core configurazione](xref:fundamentals/configuration/index), non sono consentiti nei nomi dei segreti di Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-342">Colons, which are normally used to delimit a section from a subkey in [ASP.NET Core configuration](xref:fundamentals/configuration/index), aren't allowed in key vault secret names.</span></span> <span data-ttu-id="02dbd-343">Pertanto, vengono usati due trattini e scambiati per i due punti quando i segreti vengono caricati nella configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-343">Therefore, two dashes are used and swapped for a colon when the secrets are loaded into the app's configuration.</span></span>

   <span data-ttu-id="02dbd-344">I segreti seguenti sono da usare con l'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="02dbd-344">The following secrets are for use with the sample app.</span></span> <span data-ttu-id="02dbd-345">I valori includono un `_prod` suffisso per distinguerli dai `_dev` valori dei suffissi caricati nell'ambiente di sviluppo da segreti utente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-345">The values include a `_prod` suffix to distinguish them from the `_dev` suffix values loaded in the Development environment from User Secrets.</span></span> <span data-ttu-id="02dbd-346">Sostituire `{KEY VAULT NAME}` con il nome dell'insieme di credenziali delle chiavi creato nel passaggio precedente:</span><span class="sxs-lookup"><span data-stu-id="02dbd-346">Replace `{KEY VAULT NAME}` with the name of the key vault that you created in the prior step:</span></span>

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "SecretName" --value "secret_value_1_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "Section--SecretName" --value "secret_value_2_prod"
   ```

## <a name="use-application-id-and-x509-certificate-for-non-azure-hosted-apps"></a><span data-ttu-id="02dbd-347">Usare l'ID applicazione e il certificato X. 509 per le app non ospitate in Azure</span><span class="sxs-lookup"><span data-stu-id="02dbd-347">Use Application ID and X.509 certificate for non-Azure-hosted apps</span></span>

<span data-ttu-id="02dbd-348">Configurare Azure AD, Azure Key Vault e l'app per usare un ID applicazione Azure Active Directory e un certificato X. 509 per l'autenticazione in un insieme di credenziali **delle chiavi quando l'app è ospitata all'esterno di Azure**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-348">Configure Azure AD, Azure Key Vault, and the app to use an Azure Active Directory Application ID and X.509 certificate to authenticate to a key vault **when the app is hosted outside of Azure**.</span></span> <span data-ttu-id="02dbd-349">Per i dettagli, vedere l'articolo relativo alle [informazioni su chiavi, segreti e certificati](/azure/key-vault/about-keys-secrets-and-certificates).</span><span class="sxs-lookup"><span data-stu-id="02dbd-349">For more information, see [About keys, secrets, and certificates](/azure/key-vault/about-keys-secrets-and-certificates).</span></span>

> [!NOTE]
> <span data-ttu-id="02dbd-350">Sebbene l'uso di un ID applicazione e di un certificato X. 509 sia supportato per le app ospitate in Azure, è consigliabile usare [identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) quando si ospita un'app in Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-350">Although using an Application ID and X.509 certificate is supported for apps hosted in Azure, we recommend using [Managed identities for Azure resources](#use-managed-identities-for-azure-resources) when hosting an app in Azure.</span></span> <span data-ttu-id="02dbd-351">Le identità gestite non richiedono l'archiviazione di un certificato nell'app o nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-351">Managed identities don't require storing a certificate in the app or in the development environment.</span></span>

<span data-ttu-id="02dbd-352">L'app di esempio usa un ID applicazione e un certificato X. 509 quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Certificate` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-352">The sample app uses an Application ID and X.509 certificate when the `#define` statement at the top of the *Program.cs* file is set to `Certificate`.</span></span>

1. <span data-ttu-id="02dbd-353">Creare un certificato di archivio PKCS # 12 (*. pfx*).</span><span class="sxs-lookup"><span data-stu-id="02dbd-353">Create a PKCS#12 archive (*.pfx*) certificate.</span></span> <span data-ttu-id="02dbd-354">Le opzioni per la creazione di certificati includono [Makecert in Windows](/windows/desktop/seccrypto/makecert) e [openssl](https://www.openssl.org/).</span><span class="sxs-lookup"><span data-stu-id="02dbd-354">Options for creating certificates include [MakeCert on Windows](/windows/desktop/seccrypto/makecert) and [OpenSSL](https://www.openssl.org/).</span></span>
1. <span data-ttu-id="02dbd-355">Installare il certificato nell'archivio certificati personale dell'utente corrente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-355">Install the certificate into the current user's personal certificate store.</span></span> <span data-ttu-id="02dbd-356">Contrassegnare la chiave come esportabile è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="02dbd-356">Marking the key as exportable is optional.</span></span> <span data-ttu-id="02dbd-357">Prendere nota dell'identificazione personale del certificato, che verrà usato più avanti in questo processo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-357">Note the certificate's thumbprint, which is used later in this process.</span></span>
1. <span data-ttu-id="02dbd-358">Esportare il certificato di archiviazione PKCS # 12 (*. pfx*) come certificato con codifica der (*. cer*).</span><span class="sxs-lookup"><span data-stu-id="02dbd-358">Export the PKCS#12 archive (*.pfx*) certificate as a DER-encoded certificate (*.cer*).</span></span>
1. <span data-ttu-id="02dbd-359">Registrare l'app con Azure AD (**registrazioni app**).</span><span class="sxs-lookup"><span data-stu-id="02dbd-359">Register the app with Azure AD (**App registrations**).</span></span>
1. <span data-ttu-id="02dbd-360">Caricare il certificato con codifica DER (*. cer*) in Azure ad:</span><span class="sxs-lookup"><span data-stu-id="02dbd-360">Upload the DER-encoded certificate (*.cer*) to Azure AD:</span></span>
   1. <span data-ttu-id="02dbd-361">Selezionare l'app in Azure AD.</span><span class="sxs-lookup"><span data-stu-id="02dbd-361">Select the app in Azure AD.</span></span>
   1. <span data-ttu-id="02dbd-362">Passare a **certificati & segreti**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-362">Navigate to **Certificates & secrets**.</span></span>
   1. <span data-ttu-id="02dbd-363">Selezionare **Carica certificato** per caricare il certificato, che contiene la chiave pubblica.</span><span class="sxs-lookup"><span data-stu-id="02dbd-363">Select **Upload certificate** to upload the certificate, which contains the public key.</span></span> <span data-ttu-id="02dbd-364">Un certificato con *estensione cer*, *PEM* o *CRT* è accettabile.</span><span class="sxs-lookup"><span data-stu-id="02dbd-364">A *.cer*, *.pem*, or *.crt* certificate is acceptable.</span></span>
1. <span data-ttu-id="02dbd-365">Archiviare il nome dell'insieme di credenziali delle chiavi, l'ID applicazione e l'identificazione personale del certificato nel file dell'app *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="02dbd-365">Store the key vault name, Application ID, and certificate thumbprint in the app's *appsettings.json* file.</span></span>
1. <span data-ttu-id="02dbd-366">Passare a insiemi di credenziali delle **chiavi** nella portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-366">Navigate to **Key vaults** in the Azure portal.</span></span>
1. <span data-ttu-id="02dbd-367">Selezionare l'insieme di credenziali delle chiavi creato nell' [Archivio Secret nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-367">Select the key vault that you created in the [Secret storage in the Production environment with Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) section.</span></span>
1. <span data-ttu-id="02dbd-368">Selezionare **Criteri di accesso**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-368">Select **Access policies**.</span></span>
1. <span data-ttu-id="02dbd-369">Selezionare **Aggiungi criterio di accesso**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-369">Select **Add Access Policy**.</span></span>
1. <span data-ttu-id="02dbd-370">Aprire **autorizzazioni segrete** e fornire all'app le autorizzazioni **Get** ed **List** .</span><span class="sxs-lookup"><span data-stu-id="02dbd-370">Open **Secret permissions** and provide the app with **Get** and **List** permissions.</span></span>
1. <span data-ttu-id="02dbd-371">Selezionare **Seleziona entità** e selezionare l'app registrata in base al nome.</span><span class="sxs-lookup"><span data-stu-id="02dbd-371">Select **Select principal** and select the registered app by name.</span></span> <span data-ttu-id="02dbd-372">Fare clic sul pulsante **Seleziona**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-372">Select the **Select** button.</span></span>
1. <span data-ttu-id="02dbd-373">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-373">Select **OK**.</span></span>
1. <span data-ttu-id="02dbd-374">Selezionare **Salva**.</span><span class="sxs-lookup"><span data-stu-id="02dbd-374">Select **Save**.</span></span>
1. <span data-ttu-id="02dbd-375">Distribuire l'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-375">Deploy the app.</span></span>

<span data-ttu-id="02dbd-376">L' `Certificate` app di esempio ottiene i valori di configurazione da `IConfigurationRoot` con lo stesso nome del nome del segreto:</span><span class="sxs-lookup"><span data-stu-id="02dbd-376">The `Certificate` sample app obtains its configuration values from `IConfigurationRoot` with the same name as the secret name:</span></span>

* <span data-ttu-id="02dbd-377">Valori non gerarchici: il valore per `SecretName` viene ottenuto con `config["SecretName"]` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-377">Non-hierarchical values: The value for `SecretName` is obtained with `config["SecretName"]`.</span></span>
* <span data-ttu-id="02dbd-378">Valori gerarchici (sezioni): `:` notazione use (due punti) o `GetSection` metodo di estensione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-378">Hierarchical values (sections): Use `:` (colon) notation or the `GetSection` extension method.</span></span> <span data-ttu-id="02dbd-379">Per ottenere il valore di configurazione, usare uno di questi approcci:</span><span class="sxs-lookup"><span data-stu-id="02dbd-379">Use either of these approaches to obtain the configuration value:</span></span>
  * `config["Section:SecretName"]`
  * `config.GetSection("Section")["SecretName"]`

<span data-ttu-id="02dbd-380">Il certificato X. 509 è gestito dal sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="02dbd-380">The X.509 certificate is managed by the OS.</span></span> <span data-ttu-id="02dbd-381">L'app chiama <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> con i valori forniti dal *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="02dbd-381">The app calls <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> with values supplied by the *appsettings.json* file:</span></span>

[!code-csharp[](key-vault-configuration/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=20-23)]

<span data-ttu-id="02dbd-382">Valori di esempio</span><span class="sxs-lookup"><span data-stu-id="02dbd-382">Example values:</span></span>

* <span data-ttu-id="02dbd-383">Nome dell'insieme di credenziali delle chiavi: `contosovault`</span><span class="sxs-lookup"><span data-stu-id="02dbd-383">Key vault name: `contosovault`</span></span>
* <span data-ttu-id="02dbd-384">ID applicazione: `627e911e-43cc-61d4-992e-12db9c81b413`</span><span class="sxs-lookup"><span data-stu-id="02dbd-384">Application ID: `627e911e-43cc-61d4-992e-12db9c81b413`</span></span>
* <span data-ttu-id="02dbd-385">Identificazione personale del certificato: `fe14593dd66b2406c5269d742d04b6e1ab03adb1`</span><span class="sxs-lookup"><span data-stu-id="02dbd-385">Certificate thumbprint: `fe14593dd66b2406c5269d742d04b6e1ab03adb1`</span></span>

<span data-ttu-id="02dbd-386">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="02dbd-386">*appsettings.json*:</span></span>

[!code-json[](key-vault-configuration/samples/2.x/SampleApp/appsettings.json?highlight=10-12)]

<span data-ttu-id="02dbd-387">Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati.</span><span class="sxs-lookup"><span data-stu-id="02dbd-387">When you run the app, a webpage shows the loaded secret values.</span></span> <span data-ttu-id="02dbd-388">Nell'ambiente di sviluppo, i valori Secret vengono caricati con il `_dev` suffisso.</span><span class="sxs-lookup"><span data-stu-id="02dbd-388">In the Development environment, secret values load with the `_dev` suffix.</span></span> <span data-ttu-id="02dbd-389">Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso.</span><span class="sxs-lookup"><span data-stu-id="02dbd-389">In the Production environment, the values load with the `_prod` suffix.</span></span>

## <a name="use-managed-identities-for-azure-resources"></a><span data-ttu-id="02dbd-390">Usare identità gestite per le risorse di Azure</span><span class="sxs-lookup"><span data-stu-id="02dbd-390">Use Managed identities for Azure resources</span></span>

<span data-ttu-id="02dbd-391">**Un'app distribuita in Azure** può trarre vantaggio dalle [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview), che consente all'app di eseguire l'autenticazione con Azure Key Vault usando Azure ad autenticazione senza credenziali (ID applicazione e password/segreto client) archiviate nell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-391">**An app deployed to Azure** can take advantage of [Managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview), which allows the app to authenticate with Azure Key Vault using Azure AD authentication without credentials (Application ID and Password/Client Secret) stored in the app.</span></span>

<span data-ttu-id="02dbd-392">L'app di esempio usa le identità gestite per le risorse di Azure quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Managed` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-392">The sample app uses Managed identities for Azure resources when the `#define` statement at the top of the *Program.cs* file is set to `Managed`.</span></span>

<span data-ttu-id="02dbd-393">Immettere il nome dell'insieme di credenziali nel *appsettings.json* file dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-393">Enter the vault name into the app's *appsettings.json* file.</span></span> <span data-ttu-id="02dbd-394">L'app di esempio non richiede un ID applicazione e una password (segreto client) quando è impostata sulla `Managed` versione, in modo che sia possibile ignorare tali voci di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-394">The sample app doesn't require an Application ID and Password (Client Secret) when set to the `Managed` version, so you can ignore those configuration entries.</span></span> <span data-ttu-id="02dbd-395">L'app viene distribuita in Azure e Azure autentica l'app per accedere Azure Key Vault solo usando il nome dell'insieme di credenziali archiviato nel *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="02dbd-395">The app is deployed to Azure, and Azure authenticates the app to access Azure Key Vault only using the vault name stored in the *appsettings.json* file.</span></span>

<span data-ttu-id="02dbd-396">Distribuire l'app di esempio nel servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-396">Deploy the sample app to Azure App Service.</span></span>

<span data-ttu-id="02dbd-397">Un'app distribuita nel servizio app Azure viene registrata automaticamente con Azure AD quando viene creato il servizio.</span><span class="sxs-lookup"><span data-stu-id="02dbd-397">An app deployed to Azure App Service is automatically registered with Azure AD when the service is created.</span></span> <span data-ttu-id="02dbd-398">Ottenere l'ID oggetto dalla distribuzione per utilizzarlo nel comando seguente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-398">Obtain the Object ID from the deployment for use in the following command.</span></span> <span data-ttu-id="02dbd-399">L'ID oggetto viene visualizzato nella portale di Azure nel **Identity** Pannello del servizio app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-399">The Object ID is shown in the Azure portal on the **Identity** panel of the App Service.</span></span>

<span data-ttu-id="02dbd-400">Usando l'interfaccia della riga di comando di Azure e l'ID oggetto dell'app, fornire all'app le `list` `get` autorizzazioni e per accedere all'insieme di credenziali delle chiavi:</span><span class="sxs-lookup"><span data-stu-id="02dbd-400">Using Azure CLI and the app's Object ID, provide the app with `list` and `get` permissions to access the key vault:</span></span>

```azurecli
az keyvault set-policy --name {KEY VAULT NAME} --object-id {OBJECT ID} --secret-permissions get list
```

<span data-ttu-id="02dbd-401">**Riavviare l'app** usando l'interfaccia della riga di comando di Azure, PowerShell o la portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-401">**Restart the app** using Azure CLI, PowerShell, or the Azure portal.</span></span>

<span data-ttu-id="02dbd-402">App di esempio:</span><span class="sxs-lookup"><span data-stu-id="02dbd-402">The sample app:</span></span>

* <span data-ttu-id="02dbd-403">Crea un'istanza della `AzureServiceTokenProvider` classe senza una stringa di connessione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-403">Creates an instance of the `AzureServiceTokenProvider` class without a connection string.</span></span> <span data-ttu-id="02dbd-404">Quando non viene specificata una stringa di connessione, il provider tenta di ottenere un token di accesso dalle identità gestite per le risorse di Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-404">When a connection string isn't provided, the provider attempts to obtain an access token from Managed identities for Azure resources.</span></span>
* <span data-ttu-id="02dbd-405">Viene creato un nuovo oggetto <xref:Microsoft.Azure.KeyVault.KeyVaultClient> con il `AzureServiceTokenProvider` callback del token dell'istanza.</span><span class="sxs-lookup"><span data-stu-id="02dbd-405">A new <xref:Microsoft.Azure.KeyVault.KeyVaultClient> is created with the `AzureServiceTokenProvider` instance token callback.</span></span>
* <span data-ttu-id="02dbd-406">L' <xref:Microsoft.Azure.KeyVault.KeyVaultClient> istanza viene utilizzata con un'implementazione predefinita di <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> che carica tutti i valori del segreto e sostituisce i doppi trattini ( `--` ) con i due punti ( `:` ) nei nomi delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-406">The <xref:Microsoft.Azure.KeyVault.KeyVaultClient> instance is used with a default implementation of <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> that loads all secret values and replaces double-dashes (`--`) with colons (`:`) in key names.</span></span>

[!code-csharp[](key-vault-configuration/samples/2.x/SampleApp/Program.cs?name=snippet2&highlight=13-21)]

<span data-ttu-id="02dbd-407">Valore di esempio del nome dell'insieme di credenziali delle chiavi: `contosovault`</span><span class="sxs-lookup"><span data-stu-id="02dbd-407">Key vault name example value: `contosovault`</span></span>
    
<span data-ttu-id="02dbd-408">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="02dbd-408">*appsettings.json*:</span></span>

```json
{
  "KeyVaultName": "Key Vault Name"
}
```

<span data-ttu-id="02dbd-409">Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati.</span><span class="sxs-lookup"><span data-stu-id="02dbd-409">When you run the app, a webpage shows the loaded secret values.</span></span> <span data-ttu-id="02dbd-410">Nell'ambiente di sviluppo i valori Secret hanno il `_dev` suffisso perché sono forniti da segreti utente.</span><span class="sxs-lookup"><span data-stu-id="02dbd-410">In the Development environment, secret values have the `_dev` suffix because they're provided by User Secrets.</span></span> <span data-ttu-id="02dbd-411">Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso perché sono forniti da Azure Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-411">In the Production environment, the values load with the `_prod` suffix because they're provided by Azure Key Vault.</span></span>

<span data-ttu-id="02dbd-412">Se viene visualizzato un `Access denied` errore, verificare che l'app sia registrata con Azure ad e che l'accesso sia stato fornito all'insieme di credenziali delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-412">If you receive an `Access denied` error, confirm that the app is registered with Azure AD and provided access to the key vault.</span></span> <span data-ttu-id="02dbd-413">Confermare di aver riavviato il servizio in Azure.</span><span class="sxs-lookup"><span data-stu-id="02dbd-413">Confirm that you've restarted the service in Azure.</span></span>

<span data-ttu-id="02dbd-414">Per informazioni sull'uso del provider con un'identità gestita e una pipeline di Azure DevOps, vedere [creare una connessione del servizio Azure Resource Manager a una macchina virtuale con un'identità del servizio gestito](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity).</span><span class="sxs-lookup"><span data-stu-id="02dbd-414">For information on using the provider with a managed identity and an Azure DevOps pipeline, see [Create an Azure Resource Manager service connection to a VM with a managed service identity](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity).</span></span>

## <a name="use-a-key-name-prefix"></a><span data-ttu-id="02dbd-415">Usa prefisso nome chiave</span><span class="sxs-lookup"><span data-stu-id="02dbd-415">Use a key name prefix</span></span>

<span data-ttu-id="02dbd-416"><xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> fornisce un overload che accetta un'implementazione di <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> , che consente di controllare la modalità di conversione dei segreti dell'insieme di credenziali delle chiavi in chiavi di configurazione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-416"><xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> provides an overload that accepts an implementation of <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager>, which allows you to control how key vault secrets are converted into configuration keys.</span></span> <span data-ttu-id="02dbd-417">Ad esempio, è possibile implementare l'interfaccia per caricare i valori dei segreti in base a un valore di prefisso fornito all'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-417">For example, you can implement the interface to load secret values based on a prefix value you provide at app startup.</span></span> <span data-ttu-id="02dbd-418">Questo consente, ad esempio, di caricare i segreti in base alla versione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-418">This allows you, for example, to load secrets based on the version of the app.</span></span>

> [!WARNING]
> <span data-ttu-id="02dbd-419">Non usare i prefissi sui segreti dell'insieme di credenziali delle chiavi per collocare i segreti per più app nello stesso insieme di credenziali delle chiavi o per collocare i segreti ambientali (ad esempio, *sviluppo* rispetto ai segreti di *produzione* ) nello stesso insieme di credenziali.</span><span class="sxs-lookup"><span data-stu-id="02dbd-419">Don't use prefixes on key vault secrets to place secrets for multiple apps into the same key vault or to place environmental secrets (for example, *development* versus *production* secrets) into the same vault.</span></span> <span data-ttu-id="02dbd-420">È consigliabile che app e ambienti di sviluppo/produzione diversi usino insiemi di credenziali delle chiavi distinti per isolare gli ambienti app per il massimo livello di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="02dbd-420">We recommend that different apps and development/production environments use separate key vaults to isolate app environments for the highest level of security.</span></span>

<span data-ttu-id="02dbd-421">Nell'esempio seguente viene stabilito un segreto nell'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti per l'ambiente di sviluppo) per `5000-AppSecret` (i periodi non sono consentiti nei nomi dei segreti dell'insieme di credenziali delle chiavi).</span><span class="sxs-lookup"><span data-stu-id="02dbd-421">In the following example, a secret is established in the key vault (and using the Secret Manager tool for the Development environment) for `5000-AppSecret` (periods aren't allowed in key vault secret names).</span></span> <span data-ttu-id="02dbd-422">Questo segreto rappresenta un segreto app per la versione 5.0.0.0 dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-422">This secret represents an app secret for version 5.0.0.0 of the app.</span></span> <span data-ttu-id="02dbd-423">Per un'altra versione dell'app, 5.1.0.0, viene aggiunto un segreto all'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti) per `5100-AppSecret` .</span><span class="sxs-lookup"><span data-stu-id="02dbd-423">For another version of the app, 5.1.0.0, a secret is added to the key vault (and using the Secret Manager tool) for `5100-AppSecret`.</span></span> <span data-ttu-id="02dbd-424">Ogni versione dell'app carica il valore del segreto con versione nella configurazione come `AppSecret` , rimuovendo la versione durante il caricamento del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-424">Each app version loads its versioned secret value into its configuration as `AppSecret`, stripping off the version as it loads the secret.</span></span>

<span data-ttu-id="02dbd-425"><xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> viene chiamato con un oggetto personalizzato <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> :</span><span class="sxs-lookup"><span data-stu-id="02dbd-425"><xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> is called with a custom <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager>:</span></span>

[!code-csharp[](key-vault-configuration/samples_snapshot/Program.cs)]

<span data-ttu-id="02dbd-426">L' <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> implementazione reagisce ai prefissi di versione dei segreti per caricare il segreto appropriato nella configurazione:</span><span class="sxs-lookup"><span data-stu-id="02dbd-426">The <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> implementation reacts to the version prefixes of secrets to load the proper secret into configuration:</span></span>

* <span data-ttu-id="02dbd-427">`Load` carica un segreto quando il nome inizia con il prefisso.</span><span class="sxs-lookup"><span data-stu-id="02dbd-427">`Load` loads a secret when its name starts with the prefix.</span></span> <span data-ttu-id="02dbd-428">Non vengono caricati altri segreti.</span><span class="sxs-lookup"><span data-stu-id="02dbd-428">Other secrets aren't loaded.</span></span>
* <span data-ttu-id="02dbd-429">`GetKey`:</span><span class="sxs-lookup"><span data-stu-id="02dbd-429">`GetKey`:</span></span>
  * <span data-ttu-id="02dbd-430">Rimuove il prefisso dal nome del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-430">Removes the prefix from the secret name.</span></span>
  * <span data-ttu-id="02dbd-431">Sostituisce due trattini in qualsiasi nome con `KeyDelimiter` , che è il delimitatore usato nella configurazione (in genere i due punti).</span><span class="sxs-lookup"><span data-stu-id="02dbd-431">Replaces two dashes in any name with the `KeyDelimiter`, which is the delimiter used in configuration (usually a colon).</span></span> <span data-ttu-id="02dbd-432">Azure Key Vault non consente i due punti nei nomi dei segreti.</span><span class="sxs-lookup"><span data-stu-id="02dbd-432">Azure Key Vault doesn't allow a colon in secret names.</span></span>

[!code-csharp[](key-vault-configuration/samples_snapshot/Startup.cs)]

<span data-ttu-id="02dbd-433">Il `Load` metodo viene chiamato da un algoritmo del provider che scorre i segreti dell'insieme di credenziali per trovare quelli con il prefisso della versione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-433">The `Load` method is called by a provider algorithm that iterates through the vault secrets to find the ones that have the version prefix.</span></span> <span data-ttu-id="02dbd-434">Quando viene trovato un prefisso di versione con `Load` , l'algoritmo usa il `GetKey` metodo per restituire il nome della configurazione del nome del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-434">When a version prefix is found with `Load`, the algorithm uses the `GetKey` method to return the configuration name of the secret name.</span></span> <span data-ttu-id="02dbd-435">Rimuove il prefisso della versione dal nome del segreto e restituisce il resto del nome del segreto per il caricamento nelle coppie nome-valore della configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-435">It strips off the version prefix from the secret's name and returns the rest of the secret name for loading into the app's configuration name-value pairs.</span></span>

<span data-ttu-id="02dbd-436">Quando viene implementato questo approccio:</span><span class="sxs-lookup"><span data-stu-id="02dbd-436">When this approach is implemented:</span></span>

1. <span data-ttu-id="02dbd-437">La versione dell'app specificata nel file di progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-437">The app's version specified in the app's project file.</span></span> <span data-ttu-id="02dbd-438">Nell'esempio seguente la versione dell'app è impostata su `5.0.0.0` :</span><span class="sxs-lookup"><span data-stu-id="02dbd-438">In the following example, the app's version is set to `5.0.0.0`:</span></span>

   ```xml
   <PropertyGroup>
     <Version>5.0.0.0</Version>
   </PropertyGroup>
   ```

1. <span data-ttu-id="02dbd-439">Verificare che `<UserSecretsId>` sia presente una proprietà nel file di progetto dell'app, dove `{GUID}` è un GUID fornito dall'utente:</span><span class="sxs-lookup"><span data-stu-id="02dbd-439">Confirm that a `<UserSecretsId>` property is present in the app's project file, where `{GUID}` is a user-supplied GUID:</span></span>

   ```xml
   <PropertyGroup>
     <UserSecretsId>{GUID}</UserSecretsId>
   </PropertyGroup>
   ```

   <span data-ttu-id="02dbd-440">Salvare i segreti seguenti localmente con lo [strumento di gestione dei segreti](xref:security/app-secrets):</span><span class="sxs-lookup"><span data-stu-id="02dbd-440">Save the following secrets locally with the [Secret Manager tool](xref:security/app-secrets):</span></span>

   ```dotnetcli
   dotnet user-secrets set "5000-AppSecret" "5.0.0.0_secret_value_dev"
   dotnet user-secrets set "5100-AppSecret" "5.1.0.0_secret_value_dev"
   ```

1. <span data-ttu-id="02dbd-441">I segreti vengono salvati in Azure Key Vault usando i comandi dell'interfaccia della riga di comando di Azure seguenti:</span><span class="sxs-lookup"><span data-stu-id="02dbd-441">Secrets are saved in Azure Key Vault using the following Azure CLI commands:</span></span>

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5000-AppSecret" --value "5.0.0.0_secret_value_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5100-AppSecret" --value "5.1.0.0_secret_value_prod"
   ```

1. <span data-ttu-id="02dbd-442">Quando l'app viene eseguita, vengono caricati i segreti dell'insieme di credenziali delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-442">When the app is run, the key vault secrets are loaded.</span></span> <span data-ttu-id="02dbd-443">La stringa segreta per `5000-AppSecret` viene confrontata con la versione dell'app specificata nel file di progetto dell'app ( `5.0.0.0` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-443">The string secret for `5000-AppSecret` is matched to the app's version specified in the app's project file (`5.0.0.0`).</span></span>

1. <span data-ttu-id="02dbd-444">La versione `5000` (con il trattino) viene rimossa dal nome della chiave.</span><span class="sxs-lookup"><span data-stu-id="02dbd-444">The version, `5000` (with the dash), is stripped from the key name.</span></span> <span data-ttu-id="02dbd-445">In tutta l'app, la lettura della configurazione con la chiave `AppSecret` carica il valore del segreto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-445">Throughout the app, reading configuration with the key `AppSecret` loads the secret value.</span></span>

1. <span data-ttu-id="02dbd-446">Se la versione dell'app viene modificata nel file di progetto in `5.1.0.0` e l'app viene nuovamente eseguita, il valore del segreto restituito è `5.1.0.0_secret_value_dev` nell'ambiente di sviluppo e `5.1.0.0_secret_value_prod` in produzione.</span><span class="sxs-lookup"><span data-stu-id="02dbd-446">If the app's version is changed in the project file to `5.1.0.0` and the app is run again, the secret value returned is `5.1.0.0_secret_value_dev` in the Development environment and `5.1.0.0_secret_value_prod` in Production.</span></span>

> [!NOTE]
> <span data-ttu-id="02dbd-447">È anche possibile fornire un' <xref:Microsoft.Azure.KeyVault.KeyVaultClient> implementazione personalizzata a <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> .</span><span class="sxs-lookup"><span data-stu-id="02dbd-447">You can also provide your own <xref:Microsoft.Azure.KeyVault.KeyVaultClient> implementation to <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*>.</span></span> <span data-ttu-id="02dbd-448">Un client personalizzato consente la condivisione di una singola istanza del client nell'app.</span><span class="sxs-lookup"><span data-stu-id="02dbd-448">A custom client permits sharing a single instance of the client across the app.</span></span>

## <a name="bind-an-array-to-a-class"></a><span data-ttu-id="02dbd-449">Associare una matrice a una classe</span><span class="sxs-lookup"><span data-stu-id="02dbd-449">Bind an array to a class</span></span>

<span data-ttu-id="02dbd-450">Il provider è in grado di leggere i valori di configurazione in una matrice per l'associazione a una matrice POCO.</span><span class="sxs-lookup"><span data-stu-id="02dbd-450">The provider is capable of reading configuration values into an array for binding to a POCO array.</span></span>

<span data-ttu-id="02dbd-451">Quando si legge da un'origine di configurazione che consente alle chiavi di contenere separatori di due punti ( `:` ), viene usato un segmento di chiave numerico per distinguere le chiavi che costituiscono una matrice ( `:0:` , `:1:` , &hellip; `:{n}:` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-451">When reading from a configuration source that allows keys to contain colon (`:`) separators, a numeric key segment is used to distinguish the keys that make up an array (`:0:`, `:1:`, &hellip; `:{n}:`).</span></span> <span data-ttu-id="02dbd-452">Per altre informazioni, vedere [Configuration: associare una matrice a una classe](xref:fundamentals/configuration/index#bind-an-array-to-a-class).</span><span class="sxs-lookup"><span data-stu-id="02dbd-452">For more information, see [Configuration: Bind an array to a class](xref:fundamentals/configuration/index#bind-an-array-to-a-class).</span></span>

<span data-ttu-id="02dbd-453">Azure Key Vault chiavi non possono utilizzare i due punti come separatore.</span><span class="sxs-lookup"><span data-stu-id="02dbd-453">Azure Key Vault keys can't use a colon as a separator.</span></span> <span data-ttu-id="02dbd-454">L'approccio descritto in questo argomento USA trattini doppi ( `--` ) come separatore per i valori gerarchici (sezioni).</span><span class="sxs-lookup"><span data-stu-id="02dbd-454">The approach described in this topic uses double dashes (`--`) as a separator for hierarchical values (sections).</span></span> <span data-ttu-id="02dbd-455">Le chiavi di matrice vengono archiviate in Azure Key Vault con trattini doppi e segmenti di chiave numerica ( `--0--` , `--1--` , &hellip; `--{n}--` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-455">Array keys are stored in Azure Key Vault with double dashes and numeric key segments (`--0--`, `--1--`, &hellip; `--{n}--`).</span></span>

<span data-ttu-id="02dbd-456">Esaminare la seguente configurazione del provider di registrazione [Serilog](https://serilog.net/) fornita da un file JSON.</span><span class="sxs-lookup"><span data-stu-id="02dbd-456">Examine the following [Serilog](https://serilog.net/) logging provider configuration provided by a JSON file.</span></span> <span data-ttu-id="02dbd-457">Nella matrice sono definiti due valori letterali di oggetto `WriteTo` che riflettono due *sink* Serilog, che descrivono le destinazioni per la registrazione dell'output:</span><span class="sxs-lookup"><span data-stu-id="02dbd-457">There are two object literals defined in the `WriteTo` array that reflect two Serilog *sinks*, which describe destinations for logging output:</span></span>

```json
"Serilog": {
  "WriteTo": [
    {
      "Name": "AzureTableStorage",
      "Args": {
        "storageTableName": "logs",
        "connectionString": "DefaultEnd...ountKey=Eby8...GMGw=="
      }
    },
    {
      "Name": "AzureDocumentDB",
      "Args": {
        "endpointUrl": "https://contoso.documents.azure.com:443",
        "authorizationKey": "Eby8...GMGw=="
      }
    }
  ]
}
```

<span data-ttu-id="02dbd-458">La configurazione mostrata nel file JSON precedente viene archiviata in Azure Key Vault usando la notazione a doppio trattino ( `--` ) e i segmenti numerici:</span><span class="sxs-lookup"><span data-stu-id="02dbd-458">The configuration shown in the preceding JSON file is stored in Azure Key Vault using double dash (`--`) notation and numeric segments:</span></span>

| <span data-ttu-id="02dbd-459">Chiave</span><span class="sxs-lookup"><span data-stu-id="02dbd-459">Key</span></span> | <span data-ttu-id="02dbd-460">Valore</span><span class="sxs-lookup"><span data-stu-id="02dbd-460">Value</span></span> |
| --- | ----- |
| `Serilog--WriteTo--0--Name` | `AzureTableStorage` |
| `Serilog--WriteTo--0--Args--storageTableName` | `logs` |
| `Serilog--WriteTo--0--Args--connectionString` | `DefaultEnd...ountKey=Eby8...GMGw==` |
| `Serilog--WriteTo--1--Name` | `AzureDocumentDB` |
| `Serilog--WriteTo--1--Args--endpointUrl` | `https://contoso.documents.azure.com:443` |
| `Serilog--WriteTo--1--Args--authorizationKey` | `Eby8...GMGw==` |

## <a name="reload-secrets"></a><span data-ttu-id="02dbd-461">Ricarica segreti</span><span class="sxs-lookup"><span data-stu-id="02dbd-461">Reload secrets</span></span>

<span data-ttu-id="02dbd-462">I segreti vengono memorizzati nella cache finché non `IConfigurationRoot.Reload()` viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="02dbd-462">Secrets are cached until `IConfigurationRoot.Reload()` is called.</span></span> <span data-ttu-id="02dbd-463">I segreti scaduti, disabilitati e aggiornati nell'insieme di credenziali delle chiavi non vengono rispettati dall'app fino a quando non `Reload` viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="02dbd-463">Expired, disabled, and updated secrets in the key vault are not respected by the app until `Reload` is executed.</span></span>

```csharp
Configuration.Reload();
```

## <a name="disabled-and-expired-secrets"></a><span data-ttu-id="02dbd-464">Segreti disabilitati e scaduti</span><span class="sxs-lookup"><span data-stu-id="02dbd-464">Disabled and expired secrets</span></span>

<span data-ttu-id="02dbd-465">I segreti disabilitati e scaduti generano un' <xref:Microsoft.Azure.KeyVault.Models.KeyVaultErrorException> .</span><span class="sxs-lookup"><span data-stu-id="02dbd-465">Disabled and expired secrets throw a <xref:Microsoft.Azure.KeyVault.Models.KeyVaultErrorException>.</span></span> <span data-ttu-id="02dbd-466">Per impedire che l'app venga generata, fornire la configurazione usando un provider di configurazione diverso o aggiornare il segreto disabilitato o scaduto.</span><span class="sxs-lookup"><span data-stu-id="02dbd-466">To prevent the app from throwing, provide the configuration using a different configuration provider or update the disabled or expired secret.</span></span>

## <a name="troubleshoot"></a><span data-ttu-id="02dbd-467">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="02dbd-467">Troubleshoot</span></span>

<span data-ttu-id="02dbd-468">Quando l'app non riesce a caricare la configurazione usando il provider, viene scritto un messaggio di errore nell' [infrastruttura di registrazione ASP.NET Core](xref:fundamentals/logging/index).</span><span class="sxs-lookup"><span data-stu-id="02dbd-468">When the app fails to load configuration using the provider, an error message is written to the [ASP.NET Core Logging infrastructure](xref:fundamentals/logging/index).</span></span> <span data-ttu-id="02dbd-469">Le condizioni seguenti impediranno il caricamento della configurazione:</span><span class="sxs-lookup"><span data-stu-id="02dbd-469">The following conditions will prevent configuration from loading:</span></span>

* <span data-ttu-id="02dbd-470">L'app o il certificato non è configurato correttamente in Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="02dbd-470">The app or certificate isn't configured correctly in Azure Active Directory.</span></span>
* <span data-ttu-id="02dbd-471">L'insieme di credenziali delle chiavi non esiste in Azure Key Vault.</span><span class="sxs-lookup"><span data-stu-id="02dbd-471">The key vault doesn't exist in Azure Key Vault.</span></span>
* <span data-ttu-id="02dbd-472">L'app non è autorizzata ad accedere all'insieme di credenziali delle chiavi.</span><span class="sxs-lookup"><span data-stu-id="02dbd-472">The app isn't authorized to access the key vault.</span></span>
* <span data-ttu-id="02dbd-473">Il criterio di accesso non include le `Get` `List` autorizzazioni e.</span><span class="sxs-lookup"><span data-stu-id="02dbd-473">The access policy doesn't include `Get` and `List` permissions.</span></span>
* <span data-ttu-id="02dbd-474">Nell'insieme di credenziali delle chiavi i dati di configurazione (coppia nome-valore) sono denominati, mancanti, disabilitati o scaduti in modo errato.</span><span class="sxs-lookup"><span data-stu-id="02dbd-474">In the key vault, the configuration data (name-value pair) is incorrectly named, missing, disabled, or expired.</span></span>
* <span data-ttu-id="02dbd-475">L'app presenta il nome dell'insieme di credenziali delle chiavi errato ( `KeyVaultName` ), Azure ad ID applicazione ( `AzureADApplicationId` ) o l'identificazione personale del certificato Azure ad ( `AzureADCertThumbprint` ).</span><span class="sxs-lookup"><span data-stu-id="02dbd-475">The app has the wrong key vault name (`KeyVaultName`), Azure AD Application Id (`AzureADApplicationId`), or Azure AD certificate thumbprint (`AzureADCertThumbprint`).</span></span>
* <span data-ttu-id="02dbd-476">La chiave di configurazione (nome) non è corretta nell'app per il valore che si sta provando a caricare.</span><span class="sxs-lookup"><span data-stu-id="02dbd-476">The configuration key (name) is incorrect in the app for the value you're trying to load.</span></span>
* <span data-ttu-id="02dbd-477">Quando si aggiungono i criteri di accesso per l'app all'insieme di credenziali delle chiavi, il criterio è stato creato, ma il pulsante **Salva** non è stato selezionato nell'interfaccia utente dei **criteri di accesso** .</span><span class="sxs-lookup"><span data-stu-id="02dbd-477">When adding the access policy for the app to the key vault, the policy was created, but the **Save** button wasn't selected in the **Access policies** UI.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="02dbd-478">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="02dbd-478">Additional resources</span></span>

* <xref:fundamentals/configuration/index>
* [<span data-ttu-id="02dbd-479">Microsoft Azure: Key Vault</span><span class="sxs-lookup"><span data-stu-id="02dbd-479">Microsoft Azure: Key Vault</span></span>](https://azure.microsoft.com/services/key-vault/)
* [<span data-ttu-id="02dbd-480">Microsoft Azure: Key Vault documentazione</span><span class="sxs-lookup"><span data-stu-id="02dbd-480">Microsoft Azure: Key Vault Documentation</span></span>](/azure/key-vault/)
* [<span data-ttu-id="02dbd-481">Come generare e trasferire chiavi HSM protette per Azure Key Vault</span><span class="sxs-lookup"><span data-stu-id="02dbd-481">How to generate and transfer HSM-protected keys for Azure Key Vault</span></span>](/azure/key-vault/key-vault-hsm-protected-keys)
* [<span data-ttu-id="02dbd-482">Classe KeyVaultClient</span><span class="sxs-lookup"><span data-stu-id="02dbd-482">KeyVaultClient Class</span></span>](/dotnet/api/microsoft.azure.keyvault.keyvaultclient)
* [<span data-ttu-id="02dbd-483">Guida introduttiva: Impostare e recuperare un segreto da Azure Key Vault tramite un'app Web .NET</span><span class="sxs-lookup"><span data-stu-id="02dbd-483">Quickstart: Set and retrieve a secret from Azure Key Vault by using a .NET web app</span></span>](/azure/key-vault/quick-create-net)
* [<span data-ttu-id="02dbd-484">Esercitazione: Come usare Azure Key Vault con una macchina virtuale Windows di Azure in .NET</span><span class="sxs-lookup"><span data-stu-id="02dbd-484">Tutorial: How to use Azure Key Vault with Azure Windows Virtual Machine in .NET</span></span>](/azure/key-vault/tutorial-net-windows-virtual-machine)

::: moniker-end
