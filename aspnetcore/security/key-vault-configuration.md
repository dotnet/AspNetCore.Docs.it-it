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
# <a name="azure-key-vault-configuration-provider-in-aspnet-core"></a>Provider di configurazione di Azure Key Vault in ASP.NET Core

Di [Andrew Stanton-Nurse](https://github.com/anurse)

::: moniker range=">= aspnetcore-3.0"

Questo documento illustra come usare il provider di configurazione [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) per caricare i valori di configurazione dell'app da Azure Key Vault segreti. Azure Key Vault è un servizio basato sul cloud che aiuta a proteggere le chiavi crittografiche e i segreti usati da app e servizi. Gli scenari comuni per l'uso di Azure Key Vault con le app ASP.NET Core includono:

* Controllo dell'accesso ai dati di configurazione riservati.
* Soddisfare i requisiti per i moduli di protezione hardware convalidati FIPS 140-2 Level 2 (HSM) quando si archiviano i dati di configurazione.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/key-vault-configuration/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="packages"></a>Pacchetti

Aggiungere i riferimenti ai pacchetti per i pacchetti seguenti:

* [Azure.Extensions.AspNetCore.Configuration. Segreti](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.Configuration.Secrets)
* [Azure.Identity](https://www.nuget.org/packages/Azure.Identity)

## <a name="sample-app"></a>App di esempio

L'app di esempio viene eseguita in una delle due modalità determinate dall' `#define` istruzione all'inizio del file *Program.cs* :

* `Certificate`: Illustra l'uso di un ID client Azure Key Vault e di un certificato X. 509 per accedere ai segreti archiviati nel Azure Key Vault. Questa versione dell'esempio può essere eseguita da qualsiasi posizione, distribuita nel servizio app Azure o in qualsiasi host in grado di servire un'app ASP.NET Core.
* `Managed`: Illustra come usare le [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview) per autenticare l'app Azure Key Vault con Azure ad autenticazione senza credenziali archiviate nel codice o nella configurazione dell'app. Quando si usano identità gestite per l'autenticazione, non è necessario un Azure AD ID applicazione e una password (segreto client). La `Managed` versione dell'esempio deve essere distribuita in Azure. Seguire le istruzioni riportate nella sezione [usare le identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) .

Per ulteriori informazioni su come configurare un'app di esempio utilizzando le direttive per il preprocessore ( `#define` ), vedere <xref:index#preprocessor-directives-in-sample-code> .

## <a name="secret-storage-in-the-development-environment"></a>Archiviazione segreta nell'ambiente di sviluppo

Impostare i segreti in locale usando lo [strumento di gestione dei segreti](xref:security/app-secrets). Quando l'app di esempio viene eseguita nel computer locale nell'ambiente di sviluppo, i segreti vengono caricati dall'archivio dei segreti dell'utente locale.

Lo strumento di gestione dei segreti richiede una `<UserSecretsId>` proprietà nel file di progetto dell'app. Impostare il valore della proprietà ( `{GUID}` ) su un GUID univoco:

```xml
<PropertyGroup>
  <UserSecretsId>{GUID}</UserSecretsId>
</PropertyGroup>
```

I segreti vengono creati come coppie nome/valore. I valori gerarchici (sezioni di configurazione) utilizzano un `:` (due punti) come separatore nei nomi delle chiavi di [configurazione ASP.NET Core](xref:fundamentals/configuration/index) .

Il gestore del segreto viene usato da una shell dei comandi aperta alla [radice del contenuto](xref:fundamentals/index#content-root)del progetto, dove `{SECRET NAME}` è il nome e `{SECRET VALUE}` è il valore:

```dotnetcli
dotnet user-secrets set "{SECRET NAME}" "{SECRET VALUE}"
```

Eseguire i comandi seguenti in una shell dei comandi dalla radice del [contenuto](xref:fundamentals/index#content-root) del progetto per impostare i segreti per l'app di esempio:

```dotnetcli
dotnet user-secrets set "SecretName" "secret_value_1_dev"
dotnet user-secrets set "Section:SecretName" "secret_value_2_dev"
```

Quando questi segreti vengono archiviati in Azure Key Vault nell' [archiviazione segreta nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione, il `_dev` suffisso viene modificato in `_prod` . Il suffisso fornisce un segnale visivo nell'output dell'app che indica l'origine dei valori di configurazione.

## <a name="secret-storage-in-the-production-environment-with-azure-key-vault"></a>Archiviazione segreta nell'ambiente di produzione con Azure Key Vault

Le istruzioni fornite nell'argomento [avvio rapido: impostare e recuperare un segreto da Azure Key Vault usando l'interfaccia](/azure/key-vault/quick-create-cli) della riga di comando di Azure sono riepilogate qui per la creazione di un Azure Key Vault e l'archiviazione di segreti usati dall'app di esempio. Per ulteriori informazioni, vedere l'argomento.

1. Aprire Azure cloud shell usando uno dei metodi seguenti nel [portale di Azure](https://portal.azure.com/):

   * Selezionare **Prova** nell'angolo superiore destro di un blocco di codice. Usare la stringa di ricerca "interfaccia della riga di comando di Azure" nella casella di testo.
   * Aprire Cloud Shell nel browser con il pulsante **avvia cloud Shell** .
   * Selezionare il pulsante **Cloud Shell** nel menu nell'angolo in alto a destra del portale di Azure.

   Per altre informazioni, vedere l'interfaccia della riga di comando di [Azure](/cli/azure/) e [Panoramica di Azure cloud Shell](/azure/cloud-shell/overview).

1. Se non è già stato eseguito l'autenticazione, effettuare l'accesso con il `az login` comando.

1. Creare un gruppo di risorse con il comando seguente, dove `{RESOURCE GROUP NAME}` è il nome del gruppo di risorse per il nuovo gruppo di risorse e `{LOCATION}` è l'area di Azure (Datacenter):

   ```azurecli
   az group create --name "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. Creare un insieme di credenziali delle chiavi nel gruppo di risorse con il comando seguente, dove è il nome del nuovo insieme di credenziali delle `{KEY VAULT NAME}` chiavi e `{LOCATION}` è l'area di Azure (Datacenter):

   ```azurecli
   az keyvault create --name {KEY VAULT NAME} --resource-group "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. Creare segreti nell'insieme di credenziali delle chiavi come coppie nome/valore.

   Azure Key Vault nomi di segreto sono limitati a caratteri alfanumerici e trattini. I valori gerarchici (sezioni di configurazione) usano `--` (due trattini) come separatore. I due punti, che in genere vengono usati per delimitare una sezione da una sottochiave in [ASP.NET Core configurazione](xref:fundamentals/configuration/index), non sono consentiti nei nomi dei segreti di Key Vault. Pertanto, vengono usati due trattini e scambiati per i due punti quando i segreti vengono caricati nella configurazione dell'app.

   I segreti seguenti sono da usare con l'app di esempio. I valori includono un `_prod` suffisso per distinguerli dai `_dev` valori dei suffissi caricati nell'ambiente di sviluppo da segreti utente. Sostituire `{KEY VAULT NAME}` con il nome dell'insieme di credenziali delle chiavi creato nel passaggio precedente:

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "SecretName" --value "secret_value_1_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "Section--SecretName" --value "secret_value_2_prod"
   ```

## <a name="use-application-id-and-x509-certificate-for-non-azure-hosted-apps"></a>Usare l'ID applicazione e il certificato X. 509 per le app non ospitate in Azure

Configurare Azure AD, Azure Key Vault e l'app per usare un ID applicazione Azure Active Directory e un certificato X. 509 per l'autenticazione in un insieme di credenziali **delle chiavi quando l'app è ospitata all'esterno di Azure**. Per i dettagli, vedere l'articolo relativo alle [informazioni su chiavi, segreti e certificati](/azure/key-vault/about-keys-secrets-and-certificates).

> [!NOTE]
> Sebbene l'uso di un ID applicazione e di un certificato X. 509 sia supportato per le app ospitate in Azure, è consigliabile usare [identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) quando si ospita un'app in Azure. Le identità gestite non richiedono l'archiviazione di un certificato nell'app o nell'ambiente di sviluppo.

L'app di esempio usa un ID applicazione e un certificato X. 509 quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Certificate` .

1. Creare un certificato di archivio PKCS # 12 (*. pfx*). Le opzioni per la creazione di certificati includono [Makecert in Windows](/windows/desktop/seccrypto/makecert) e [openssl](https://www.openssl.org/).
1. Installare il certificato nell'archivio certificati personale dell'utente corrente. Contrassegnare la chiave come esportabile è facoltativa. Prendere nota dell'identificazione personale del certificato, che verrà usato più avanti in questo processo.
1. Esportare il certificato di archiviazione PKCS # 12 (*. pfx*) come certificato con codifica der (*. cer*).
1. Registrare l'app con Azure AD (**registrazioni app**).
1. Caricare il certificato con codifica DER (*. cer*) in Azure ad:
   1. Selezionare l'app in Azure AD.
   1. Passare a **certificati & segreti**.
   1. Selezionare **Carica certificato** per caricare il certificato, che contiene la chiave pubblica. Un certificato con *estensione cer*, *PEM* o *CRT* è accettabile.
1. Archiviare il nome dell'insieme di credenziali delle chiavi, l'ID applicazione e l'identificazione personale del certificato nel file dell'app *appsettings.json* .
1. Passare a insiemi di credenziali delle **chiavi** nella portale di Azure.
1. Selezionare l'insieme di credenziali delle chiavi creato nell' [Archivio Secret nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione.
1. Selezionare **Criteri di accesso**.
1. Selezionare **Aggiungi criterio di accesso**.
1. Aprire **autorizzazioni segrete** e fornire all'app le autorizzazioni **Get** ed **List** .
1. Selezionare **Seleziona entità** e selezionare l'app registrata in base al nome. Fare clic sul pulsante **Seleziona**.
1. Selezionare **OK**.
1. Selezionare **Salva**.
1. Distribuire l'app.

L' `Certificate` app di esempio ottiene i valori di configurazione da `IConfigurationRoot` con lo stesso nome del nome del segreto:

* Valori non gerarchici: il valore per `SecretName` viene ottenuto con `config["SecretName"]` .
* Valori gerarchici (sezioni): `:` notazione use (due punti) o `GetSection` metodo di estensione. Per ottenere il valore di configurazione, usare uno di questi approcci:
  * `config["Section:SecretName"]`
  * `config.GetSection("Section")["SecretName"]`

Il certificato X. 509 è gestito dal sistema operativo. L'app chiama **AddAzureKeyVault** con i valori forniti dal *appsettings.json* file:

[!code-csharp[](key-vault-configuration/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=46-49)]

Valori di esempio

* Nome dell'insieme di credenziali delle chiavi: `contosovault`
* ID applicazione: `627e911e-43cc-61d4-992e-12db9c81b413`
* Identificazione personale del certificato: `fe14593dd66b2406c5269d742d04b6e1ab03adb1`

*appsettings.json*:

[!code-json[](key-vault-configuration/samples/3.x/SampleApp/appsettings.json?highlight=10-12)]

Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati. Nell'ambiente di sviluppo, i valori Secret vengono caricati con il `_dev` suffisso. Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso.

## <a name="use-managed-identities-for-azure-resources"></a>Usare identità gestite per le risorse di Azure

**Un'app distribuita in Azure** può trarre vantaggio dalle [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview), che consente all'app di eseguire l'autenticazione con Azure Key Vault usando Azure ad autenticazione senza credenziali (ID applicazione e password/segreto client) archiviate nell'app.

L'app di esempio usa le identità gestite per le risorse di Azure quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Managed` .

Immettere il nome dell'insieme di credenziali nel *appsettings.json* file dell'app. L'app di esempio non richiede un ID applicazione e una password (segreto client) quando è impostata sulla `Managed` versione, in modo che sia possibile ignorare tali voci di configurazione. L'app viene distribuita in Azure e Azure autentica l'app per accedere Azure Key Vault solo usando il nome dell'insieme di credenziali archiviato nel *appsettings.json* file.

Distribuire l'app di esempio nel servizio app Azure.

Un'app distribuita nel servizio app Azure viene registrata automaticamente con Azure AD quando viene creato il servizio. Ottenere l'ID oggetto dalla distribuzione per utilizzarlo nel comando seguente. L'ID oggetto viene visualizzato nella portale di Azure nel **Identity** Pannello del servizio app.

Usando l'interfaccia della riga di comando di Azure e l'ID oggetto dell'app, fornire all'app le `list` `get` autorizzazioni e per accedere all'insieme di credenziali delle chiavi:

```azurecli
az keyvault set-policy --name {KEY VAULT NAME} --object-id {OBJECT ID} --secret-permissions get list
```

**Riavviare l'app** usando l'interfaccia della riga di comando di Azure, PowerShell o la portale di Azure.

App di esempio:

* Crea un'istanza della `DefaultAzureCredential` classe, la credenziale tenta di ottenere un token di accesso dall'ambiente per le risorse di Azure.
* [`Azure.Security.KeyVault.Secrets.Secrets`](/dotnet/api/azure.security.keyvault.secrets)Viene creata una nuova istanza di con l' `DefaultAzureCredential` istanza di.
* L' `Azure.Security.KeyVault.Secrets.Secrets` istanza viene utilizzata con un'implementazione predefinita di `Azure.Extensions.AspNetCore.Configuration.Secrets` che carica tutti i valori del segreto e sostituisce i doppi trattini ( `--` ) con i due punti ( `:` ) nei nomi delle chiavi.

[!code-csharp[](key-vault-configuration/samples/3.x/SampleApp/Program.cs?name=snippet2&highlight=12-14)]

Valore di esempio del nome dell'insieme di credenziali delle chiavi: `contosovault`

*appsettings.json*:

```json
{
  "KeyVaultName": "Key Vault Name"
}
```

Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati. Nell'ambiente di sviluppo i valori Secret hanno il `_dev` suffisso perché sono forniti da segreti utente. Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso perché sono forniti da Azure Key Vault.

Se viene visualizzato un `Access denied` errore, verificare che l'app sia registrata con Azure ad e che l'accesso sia stato fornito all'insieme di credenziali delle chiavi. Confermare di aver riavviato il servizio in Azure.

Per informazioni sull'uso del provider con un'identità gestita e una pipeline di Azure DevOps, vedere [creare una connessione del servizio Azure Resource Manager a una macchina virtuale con un'identità del servizio gestito](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity).

## <a name="configuration-options"></a>Opzioni di configurazione

AddAzureKeyVault può accettare un AzureKeyVaultConfigurationOptions:

```csharp
config.AddAzureKeyVault(new SecretClient(new URI("Your Key Vault Endpoint"), new DefaultAzureCredential()),
                        new AzureKeyVaultConfigurationOptions())
    {
        ...
    });
```

| Proprietà         | Descrizione |
| ---------------- | ----------- |
| `Manager`        | `Azure.Extensions.AspNetCore.Configuration.Secrets` istanza di utilizzata per controllare il caricamento del segreto. |
| `ReloadInterval` | `Timespan` per attendere tra i tentativi di polling dell'insieme di credenziali delle chiavi per le modifiche. Il valore predefinito è `null` (la configurazione non viene ricaricata). |

## <a name="use-a-key-name-prefix"></a>Usa prefisso nome chiave

AddAzureKeyVault fornisce un overload che accetta un'implementazione di `Azure.Extensions.AspNetCore.Configuration.Secrets` , che consente di controllare il modo in cui i segreti dell'insieme di credenziali delle chiavi vengono convertiti in chiavi di configurazione. Ad esempio, è possibile implementare l'interfaccia per caricare i valori dei segreti in base a un valore di prefisso fornito all'avvio dell'app. Questo consente, ad esempio, di caricare i segreti in base alla versione dell'app.

> [!WARNING]
> Non usare i prefissi sui segreti dell'insieme di credenziali delle chiavi per collocare i segreti per più app nello stesso insieme di credenziali delle chiavi o per collocare i segreti ambientali (ad esempio, *sviluppo* rispetto ai segreti di *produzione* ) nello stesso insieme di credenziali. È consigliabile che app e ambienti di sviluppo/produzione diversi usino insiemi di credenziali delle chiavi distinti per isolare gli ambienti app per il massimo livello di sicurezza.

Nell'esempio seguente viene stabilito un segreto nell'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti per l'ambiente di sviluppo) per `5000-AppSecret` (i periodi non sono consentiti nei nomi dei segreti dell'insieme di credenziali delle chiavi). Questo segreto rappresenta un segreto app per la versione 5.0.0.0 dell'app. Per un'altra versione dell'app, 5.1.0.0, viene aggiunto un segreto all'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti) per `5100-AppSecret` . Ogni versione dell'app carica il valore del segreto con versione nella configurazione come `AppSecret` , rimuovendo la versione durante il caricamento del segreto.

AddAzureKeyVault viene chiamato con un oggetto personalizzato `Azure.Extensions.AspNetCore.Configuration.Secrets` :

[!code-csharp[](key-vault-configuration/samples_snapshot/Program.cs)]

L' `Azure.Extensions.AspNetCore.Configuration.Secrets` implementazione reagisce ai prefissi di versione dei segreti per caricare il segreto appropriato nella configurazione:

* `Load` carica un segreto quando il nome inizia con il prefisso. Non vengono caricati altri segreti.
* `GetKey`:
  * Rimuove il prefisso dal nome del segreto.
  * Sostituisce due trattini in qualsiasi nome con `KeyDelimiter` , che è il delimitatore usato nella configurazione (in genere i due punti). Azure Key Vault non consente i due punti nei nomi dei segreti.

[!code-csharp[](key-vault-configuration/samples_snapshot/Startup.cs)]

Il `Load` metodo viene chiamato da un algoritmo del provider che scorre i segreti dell'insieme di credenziali per trovare quelli con il prefisso della versione. Quando viene trovato un prefisso di versione con `Load` , l'algoritmo usa il `GetKey` metodo per restituire il nome della configurazione del nome del segreto. Rimuove il prefisso della versione dal nome del segreto e restituisce il resto del nome del segreto per il caricamento nelle coppie nome-valore della configurazione dell'app.

Quando viene implementato questo approccio:

1. La versione dell'app specificata nel file di progetto dell'app. Nell'esempio seguente la versione dell'app è impostata su `5.0.0.0` :

   ```xml
   <PropertyGroup>
     <Version>5.0.0.0</Version>
   </PropertyGroup>
   ```

1. Verificare che `<UserSecretsId>` sia presente una proprietà nel file di progetto dell'app, dove `{GUID}` è un GUID fornito dall'utente:

   ```xml
   <PropertyGroup>
     <UserSecretsId>{GUID}</UserSecretsId>
   </PropertyGroup>
   ```

   Salvare i segreti seguenti localmente con lo [strumento di gestione dei segreti](xref:security/app-secrets):

   ```dotnetcli
   dotnet user-secrets set "5000-AppSecret" "5.0.0.0_secret_value_dev"
   dotnet user-secrets set "5100-AppSecret" "5.1.0.0_secret_value_dev"
   ```

1. I segreti vengono salvati in Azure Key Vault usando i comandi dell'interfaccia della riga di comando di Azure seguenti:

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5000-AppSecret" --value "5.0.0.0_secret_value_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5100-AppSecret" --value "5.1.0.0_secret_value_prod"
   ```

1. Quando l'app viene eseguita, vengono caricati i segreti dell'insieme di credenziali delle chiavi. La stringa segreta per `5000-AppSecret` viene confrontata con la versione dell'app specificata nel file di progetto dell'app ( `5.0.0.0` ).

1. La versione `5000` (con il trattino) viene rimossa dal nome della chiave. In tutta l'app, la lettura della configurazione con la chiave `AppSecret` carica il valore del segreto.

1. Se la versione dell'app viene modificata nel file di progetto in `5.1.0.0` e l'app viene nuovamente eseguita, il valore del segreto restituito è `5.1.0.0_secret_value_dev` nell'ambiente di sviluppo e `5.1.0.0_secret_value_prod` in produzione.

> [!NOTE]
> È anche possibile fornire un' <xref:Azure.Security.KeyVault.Secrets.SecretClient> implementazione personalizzata a AddAzureKeyVault. Un client personalizzato consente la condivisione di una singola istanza del client nell'app.

## <a name="bind-an-array-to-a-class"></a>Associare una matrice a una classe

Il provider è in grado di leggere i valori di configurazione in una matrice per l'associazione a una matrice POCO.

Quando si legge da un'origine di configurazione che consente alle chiavi di contenere separatori di due punti ( `:` ), viene usato un segmento di chiave numerico per distinguere le chiavi che costituiscono una matrice ( `:0:` , `:1:` , &hellip; `:{n}:` ). Per altre informazioni, vedere [Configuration: associare una matrice a una classe](xref:fundamentals/configuration/index#bind-an-array-to-a-class).

Azure Key Vault chiavi non possono utilizzare i due punti come separatore. L'approccio descritto in questo argomento USA trattini doppi ( `--` ) come separatore per i valori gerarchici (sezioni). Le chiavi di matrice vengono archiviate in Azure Key Vault con trattini doppi e segmenti di chiave numerica ( `--0--` , `--1--` , &hellip; `--{n}--` ).

Esaminare la seguente configurazione del provider di registrazione [Serilog](https://serilog.net/) fornita da un file JSON. Nella matrice sono definiti due valori letterali di oggetto `WriteTo` che riflettono due *sink* Serilog, che descrivono le destinazioni per la registrazione dell'output:

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

La configurazione mostrata nel file JSON precedente viene archiviata in Azure Key Vault usando la notazione a doppio trattino ( `--` ) e i segmenti numerici:

| Chiave | Valore |
| --- | ----- |
| `Serilog--WriteTo--0--Name` | `AzureTableStorage` |
| `Serilog--WriteTo--0--Args--storageTableName` | `logs` |
| `Serilog--WriteTo--0--Args--connectionString` | `DefaultEnd...ountKey=Eby8...GMGw==` |
| `Serilog--WriteTo--1--Name` | `AzureDocumentDB` |
| `Serilog--WriteTo--1--Args--endpointUrl` | `https://contoso.documents.azure.com:443` |
| `Serilog--WriteTo--1--Args--authorizationKey` | `Eby8...GMGw==` |

## <a name="reload-secrets"></a>Ricarica segreti

I segreti vengono memorizzati nella cache finché non `IConfigurationRoot.Reload()` viene chiamato. I segreti scaduti, disabilitati e aggiornati nell'insieme di credenziali delle chiavi non vengono rispettati dall'app fino a quando non `Reload` viene eseguito.

```csharp
Configuration.Reload();
```

## <a name="disabled-and-expired-secrets"></a>Segreti disabilitati e scaduti

I segreti disabilitati e scaduti generano un' <xref:Azure.RequestFailedException> . Per impedire che l'app venga generata, fornire la configurazione usando un provider di configurazione diverso o aggiornare il segreto disabilitato o scaduto.

## <a name="troubleshoot"></a>Risolvere problemi

Quando l'app non riesce a caricare la configurazione usando il provider, viene scritto un messaggio di errore nell' [infrastruttura di registrazione ASP.NET Core](xref:fundamentals/logging/index). Le condizioni seguenti impediranno il caricamento della configurazione:

* L'app o il certificato non è configurato correttamente in Azure Active Directory.
* L'insieme di credenziali delle chiavi non esiste in Azure Key Vault.
* L'app non è autorizzata ad accedere all'insieme di credenziali delle chiavi.
* Il criterio di accesso non include le `Get` `List` autorizzazioni e.
* Nell'insieme di credenziali delle chiavi i dati di configurazione (coppia nome-valore) sono denominati, mancanti, disabilitati o scaduti in modo errato.
* L'app presenta il nome dell'insieme di credenziali delle chiavi ( `KeyVaultName` ), Azure ad ID applicazione () o l'identificazione `AzureADApplicationId` personale del certificato Azure ad ( `AzureADCertThumbprint` ) o Azure ad DirectoryId ( `AzureADDirectoryId` ).
* La chiave di configurazione (nome) non è corretta nell'app per il valore che si sta provando a caricare.
* Quando si aggiungono i criteri di accesso per l'app all'insieme di credenziali delle chiavi, il criterio è stato creato, ma il pulsante **Salva** non è stato selezionato nell'interfaccia utente dei **criteri di accesso** .

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/configuration/index>
* [Microsoft Azure: Key Vault](https://azure.microsoft.com/services/key-vault/)
* [Microsoft Azure: Key Vault documentazione](/azure/key-vault/)
* [Come generare e trasferire chiavi HSM protette per Azure Key Vault](/azure/key-vault/key-vault-hsm-protected-keys)
* [Classe KeyVaultClient](/dotnet/api/microsoft.azure.keyvault.keyvaultclient)
* [Guida introduttiva: Impostare e recuperare un segreto da Azure Key Vault tramite un'app Web .NET](/azure/key-vault/quick-create-net)
* [Esercitazione: Come usare Azure Key Vault con una macchina virtuale Windows di Azure in .NET](/azure/key-vault/tutorial-net-windows-virtual-machine)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Questo documento illustra come usare il provider di configurazione [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/) per caricare i valori di configurazione dell'app da Azure Key Vault segreti. Azure Key Vault è un servizio basato sul cloud che aiuta a proteggere le chiavi crittografiche e i segreti usati da app e servizi. Gli scenari comuni per l'uso di Azure Key Vault con le app ASP.NET Core includono:

* Controllo dell'accesso ai dati di configurazione riservati.
* Soddisfare i requisiti per i moduli di protezione hardware convalidati FIPS 140-2 Level 2 (HSM) quando si archiviano i dati di configurazione.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/key-vault-configuration/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="packages"></a>Pacchetti

Aggiungere un riferimento al pacchetto [Microsoft.Extensions.Configuration. ](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/) Pacchetto di AzureKeyVault.

## <a name="sample-app"></a>App di esempio

L'app di esempio viene eseguita in una delle due modalità determinate dall' `#define` istruzione all'inizio del file *Program.cs* :

* `Certificate`: Illustra l'uso di un ID client Azure Key Vault e di un certificato X. 509 per accedere ai segreti archiviati nel Azure Key Vault. Questa versione dell'esempio può essere eseguita da qualsiasi posizione, distribuita nel servizio app Azure o in qualsiasi host in grado di servire un'app ASP.NET Core.
* `Managed`: Illustra come usare le [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview) per autenticare l'app Azure Key Vault con Azure ad autenticazione senza credenziali archiviate nel codice o nella configurazione dell'app. Quando si usano identità gestite per l'autenticazione, non è necessario un Azure AD ID applicazione e una password (segreto client). La `Managed` versione dell'esempio deve essere distribuita in Azure. Seguire le istruzioni riportate nella sezione [usare le identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) .

Per ulteriori informazioni su come configurare un'app di esempio utilizzando le direttive per il preprocessore ( `#define` ), vedere <xref:index#preprocessor-directives-in-sample-code> .

## <a name="secret-storage-in-the-development-environment"></a>Archiviazione segreta nell'ambiente di sviluppo

Impostare i segreti in locale usando lo [strumento di gestione dei segreti](xref:security/app-secrets). Quando l'app di esempio viene eseguita nel computer locale nell'ambiente di sviluppo, i segreti vengono caricati dall'archivio dei segreti dell'utente locale.

Lo strumento di gestione dei segreti richiede una `<UserSecretsId>` proprietà nel file di progetto dell'app. Impostare il valore della proprietà ( `{GUID}` ) su un GUID univoco:

```xml
<PropertyGroup>
  <UserSecretsId>{GUID}</UserSecretsId>
</PropertyGroup>
```

I segreti vengono creati come coppie nome/valore. I valori gerarchici (sezioni di configurazione) utilizzano un `:` (due punti) come separatore nei nomi delle chiavi di [configurazione ASP.NET Core](xref:fundamentals/configuration/index) .

Il gestore del segreto viene usato da una shell dei comandi aperta alla [radice del contenuto](xref:fundamentals/index#content-root)del progetto, dove `{SECRET NAME}` è il nome e `{SECRET VALUE}` è il valore:

```dotnetcli
dotnet user-secrets set "{SECRET NAME}" "{SECRET VALUE}"
```

Eseguire i comandi seguenti in una shell dei comandi dalla radice del [contenuto](xref:fundamentals/index#content-root) del progetto per impostare i segreti per l'app di esempio:

```dotnetcli
dotnet user-secrets set "SecretName" "secret_value_1_dev"
dotnet user-secrets set "Section:SecretName" "secret_value_2_dev"
```

Quando questi segreti vengono archiviati in Azure Key Vault nell' [archiviazione segreta nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione, il `_dev` suffisso viene modificato in `_prod` . Il suffisso fornisce un segnale visivo nell'output dell'app che indica l'origine dei valori di configurazione.

## <a name="secret-storage-in-the-production-environment-with-azure-key-vault"></a>Archiviazione segreta nell'ambiente di produzione con Azure Key Vault

Le istruzioni fornite nell'argomento [avvio rapido: impostare e recuperare un segreto da Azure Key Vault usando l'interfaccia](/azure/key-vault/quick-create-cli) della riga di comando di Azure sono riepilogate qui per la creazione di un Azure Key Vault e l'archiviazione di segreti usati dall'app di esempio. Per ulteriori informazioni, vedere l'argomento.

1. Aprire Azure cloud shell usando uno dei metodi seguenti nel [portale di Azure](https://portal.azure.com/):

   * Selezionare **Prova** nell'angolo superiore destro di un blocco di codice. Usare la stringa di ricerca "interfaccia della riga di comando di Azure" nella casella di testo.
   * Aprire Cloud Shell nel browser con il pulsante **avvia cloud Shell** .
   * Selezionare il pulsante **Cloud Shell** nel menu nell'angolo in alto a destra del portale di Azure.

   Per altre informazioni, vedere l'interfaccia della riga di comando di [Azure](/cli/azure/) e [Panoramica di Azure cloud Shell](/azure/cloud-shell/overview).

1. Se non è già stato eseguito l'autenticazione, effettuare l'accesso con il `az login` comando.

1. Creare un gruppo di risorse con il comando seguente, dove `{RESOURCE GROUP NAME}` è il nome del gruppo di risorse per il nuovo gruppo di risorse e `{LOCATION}` è l'area di Azure (Datacenter):

   ```azurecli
   az group create --name "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. Creare un insieme di credenziali delle chiavi nel gruppo di risorse con il comando seguente, dove è il nome del nuovo insieme di credenziali delle `{KEY VAULT NAME}` chiavi e `{LOCATION}` è l'area di Azure (Datacenter):

   ```azurecli
   az keyvault create --name {KEY VAULT NAME} --resource-group "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. Creare segreti nell'insieme di credenziali delle chiavi come coppie nome/valore.

   Azure Key Vault nomi di segreto sono limitati a caratteri alfanumerici e trattini. I valori gerarchici (sezioni di configurazione) usano `--` (due trattini) come separatore. I due punti, che in genere vengono usati per delimitare una sezione da una sottochiave in [ASP.NET Core configurazione](xref:fundamentals/configuration/index), non sono consentiti nei nomi dei segreti di Key Vault. Pertanto, vengono usati due trattini e scambiati per i due punti quando i segreti vengono caricati nella configurazione dell'app.

   I segreti seguenti sono da usare con l'app di esempio. I valori includono un `_prod` suffisso per distinguerli dai `_dev` valori dei suffissi caricati nell'ambiente di sviluppo da segreti utente. Sostituire `{KEY VAULT NAME}` con il nome dell'insieme di credenziali delle chiavi creato nel passaggio precedente:

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "SecretName" --value "secret_value_1_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "Section--SecretName" --value "secret_value_2_prod"
   ```

## <a name="use-application-id-and-x509-certificate-for-non-azure-hosted-apps"></a>Usare l'ID applicazione e il certificato X. 509 per le app non ospitate in Azure

Configurare Azure AD, Azure Key Vault e l'app per usare un ID applicazione Azure Active Directory e un certificato X. 509 per l'autenticazione in un insieme di credenziali **delle chiavi quando l'app è ospitata all'esterno di Azure**. Per i dettagli, vedere l'articolo relativo alle [informazioni su chiavi, segreti e certificati](/azure/key-vault/about-keys-secrets-and-certificates).

> [!NOTE]
> Sebbene l'uso di un ID applicazione e di un certificato X. 509 sia supportato per le app ospitate in Azure, è consigliabile usare [identità gestite per le risorse di Azure](#use-managed-identities-for-azure-resources) quando si ospita un'app in Azure. Le identità gestite non richiedono l'archiviazione di un certificato nell'app o nell'ambiente di sviluppo.

L'app di esempio usa un ID applicazione e un certificato X. 509 quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Certificate` .

1. Creare un certificato di archivio PKCS # 12 (*. pfx*). Le opzioni per la creazione di certificati includono [Makecert in Windows](/windows/desktop/seccrypto/makecert) e [openssl](https://www.openssl.org/).
1. Installare il certificato nell'archivio certificati personale dell'utente corrente. Contrassegnare la chiave come esportabile è facoltativa. Prendere nota dell'identificazione personale del certificato, che verrà usato più avanti in questo processo.
1. Esportare il certificato di archiviazione PKCS # 12 (*. pfx*) come certificato con codifica der (*. cer*).
1. Registrare l'app con Azure AD (**registrazioni app**).
1. Caricare il certificato con codifica DER (*. cer*) in Azure ad:
   1. Selezionare l'app in Azure AD.
   1. Passare a **certificati & segreti**.
   1. Selezionare **Carica certificato** per caricare il certificato, che contiene la chiave pubblica. Un certificato con *estensione cer*, *PEM* o *CRT* è accettabile.
1. Archiviare il nome dell'insieme di credenziali delle chiavi, l'ID applicazione e l'identificazione personale del certificato nel file dell'app *appsettings.json* .
1. Passare a insiemi di credenziali delle **chiavi** nella portale di Azure.
1. Selezionare l'insieme di credenziali delle chiavi creato nell' [Archivio Secret nell'ambiente di produzione con Azure Key Vault](#secret-storage-in-the-production-environment-with-azure-key-vault) sezione.
1. Selezionare **Criteri di accesso**.
1. Selezionare **Aggiungi criterio di accesso**.
1. Aprire **autorizzazioni segrete** e fornire all'app le autorizzazioni **Get** ed **List** .
1. Selezionare **Seleziona entità** e selezionare l'app registrata in base al nome. Fare clic sul pulsante **Seleziona**.
1. Selezionare **OK**.
1. Selezionare **Salva**.
1. Distribuire l'app.

L' `Certificate` app di esempio ottiene i valori di configurazione da `IConfigurationRoot` con lo stesso nome del nome del segreto:

* Valori non gerarchici: il valore per `SecretName` viene ottenuto con `config["SecretName"]` .
* Valori gerarchici (sezioni): `:` notazione use (due punti) o `GetSection` metodo di estensione. Per ottenere il valore di configurazione, usare uno di questi approcci:
  * `config["Section:SecretName"]`
  * `config.GetSection("Section")["SecretName"]`

Il certificato X. 509 è gestito dal sistema operativo. L'app chiama <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> con i valori forniti dal *appsettings.json* file:

[!code-csharp[](key-vault-configuration/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=20-23)]

Valori di esempio

* Nome dell'insieme di credenziali delle chiavi: `contosovault`
* ID applicazione: `627e911e-43cc-61d4-992e-12db9c81b413`
* Identificazione personale del certificato: `fe14593dd66b2406c5269d742d04b6e1ab03adb1`

*appsettings.json*:

[!code-json[](key-vault-configuration/samples/2.x/SampleApp/appsettings.json?highlight=10-12)]

Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati. Nell'ambiente di sviluppo, i valori Secret vengono caricati con il `_dev` suffisso. Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso.

## <a name="use-managed-identities-for-azure-resources"></a>Usare identità gestite per le risorse di Azure

**Un'app distribuita in Azure** può trarre vantaggio dalle [identità gestite per le risorse di Azure](/azure/active-directory/managed-identities-azure-resources/overview), che consente all'app di eseguire l'autenticazione con Azure Key Vault usando Azure ad autenticazione senza credenziali (ID applicazione e password/segreto client) archiviate nell'app.

L'app di esempio usa le identità gestite per le risorse di Azure quando l' `#define` istruzione all'inizio del file *Program.cs* è impostata su `Managed` .

Immettere il nome dell'insieme di credenziali nel *appsettings.json* file dell'app. L'app di esempio non richiede un ID applicazione e una password (segreto client) quando è impostata sulla `Managed` versione, in modo che sia possibile ignorare tali voci di configurazione. L'app viene distribuita in Azure e Azure autentica l'app per accedere Azure Key Vault solo usando il nome dell'insieme di credenziali archiviato nel *appsettings.json* file.

Distribuire l'app di esempio nel servizio app Azure.

Un'app distribuita nel servizio app Azure viene registrata automaticamente con Azure AD quando viene creato il servizio. Ottenere l'ID oggetto dalla distribuzione per utilizzarlo nel comando seguente. L'ID oggetto viene visualizzato nella portale di Azure nel **Identity** Pannello del servizio app.

Usando l'interfaccia della riga di comando di Azure e l'ID oggetto dell'app, fornire all'app le `list` `get` autorizzazioni e per accedere all'insieme di credenziali delle chiavi:

```azurecli
az keyvault set-policy --name {KEY VAULT NAME} --object-id {OBJECT ID} --secret-permissions get list
```

**Riavviare l'app** usando l'interfaccia della riga di comando di Azure, PowerShell o la portale di Azure.

App di esempio:

* Crea un'istanza della `AzureServiceTokenProvider` classe senza una stringa di connessione. Quando non viene specificata una stringa di connessione, il provider tenta di ottenere un token di accesso dalle identità gestite per le risorse di Azure.
* Viene creato un nuovo oggetto <xref:Microsoft.Azure.KeyVault.KeyVaultClient> con il `AzureServiceTokenProvider` callback del token dell'istanza.
* L' <xref:Microsoft.Azure.KeyVault.KeyVaultClient> istanza viene utilizzata con un'implementazione predefinita di <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> che carica tutti i valori del segreto e sostituisce i doppi trattini ( `--` ) con i due punti ( `:` ) nei nomi delle chiavi.

[!code-csharp[](key-vault-configuration/samples/2.x/SampleApp/Program.cs?name=snippet2&highlight=13-21)]

Valore di esempio del nome dell'insieme di credenziali delle chiavi: `contosovault`
    
*appsettings.json*:

```json
{
  "KeyVaultName": "Key Vault Name"
}
```

Quando si esegue l'app, in una pagina Web vengono visualizzati i valori dei segreti caricati. Nell'ambiente di sviluppo i valori Secret hanno il `_dev` suffisso perché sono forniti da segreti utente. Nell'ambiente di produzione, i valori vengono caricati con il `_prod` suffisso perché sono forniti da Azure Key Vault.

Se viene visualizzato un `Access denied` errore, verificare che l'app sia registrata con Azure ad e che l'accesso sia stato fornito all'insieme di credenziali delle chiavi. Confermare di aver riavviato il servizio in Azure.

Per informazioni sull'uso del provider con un'identità gestita e una pipeline di Azure DevOps, vedere [creare una connessione del servizio Azure Resource Manager a una macchina virtuale con un'identità del servizio gestito](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity).

## <a name="use-a-key-name-prefix"></a>Usa prefisso nome chiave

<xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> fornisce un overload che accetta un'implementazione di <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> , che consente di controllare la modalità di conversione dei segreti dell'insieme di credenziali delle chiavi in chiavi di configurazione. Ad esempio, è possibile implementare l'interfaccia per caricare i valori dei segreti in base a un valore di prefisso fornito all'avvio dell'app. Questo consente, ad esempio, di caricare i segreti in base alla versione dell'app.

> [!WARNING]
> Non usare i prefissi sui segreti dell'insieme di credenziali delle chiavi per collocare i segreti per più app nello stesso insieme di credenziali delle chiavi o per collocare i segreti ambientali (ad esempio, *sviluppo* rispetto ai segreti di *produzione* ) nello stesso insieme di credenziali. È consigliabile che app e ambienti di sviluppo/produzione diversi usino insiemi di credenziali delle chiavi distinti per isolare gli ambienti app per il massimo livello di sicurezza.

Nell'esempio seguente viene stabilito un segreto nell'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti per l'ambiente di sviluppo) per `5000-AppSecret` (i periodi non sono consentiti nei nomi dei segreti dell'insieme di credenziali delle chiavi). Questo segreto rappresenta un segreto app per la versione 5.0.0.0 dell'app. Per un'altra versione dell'app, 5.1.0.0, viene aggiunto un segreto all'insieme di credenziali delle chiavi (e usando lo strumento di gestione dei segreti) per `5100-AppSecret` . Ogni versione dell'app carica il valore del segreto con versione nella configurazione come `AppSecret` , rimuovendo la versione durante il caricamento del segreto.

<xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> viene chiamato con un oggetto personalizzato <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> :

[!code-csharp[](key-vault-configuration/samples_snapshot/Program.cs)]

L' <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> implementazione reagisce ai prefissi di versione dei segreti per caricare il segreto appropriato nella configurazione:

* `Load` carica un segreto quando il nome inizia con il prefisso. Non vengono caricati altri segreti.
* `GetKey`:
  * Rimuove il prefisso dal nome del segreto.
  * Sostituisce due trattini in qualsiasi nome con `KeyDelimiter` , che è il delimitatore usato nella configurazione (in genere i due punti). Azure Key Vault non consente i due punti nei nomi dei segreti.

[!code-csharp[](key-vault-configuration/samples_snapshot/Startup.cs)]

Il `Load` metodo viene chiamato da un algoritmo del provider che scorre i segreti dell'insieme di credenziali per trovare quelli con il prefisso della versione. Quando viene trovato un prefisso di versione con `Load` , l'algoritmo usa il `GetKey` metodo per restituire il nome della configurazione del nome del segreto. Rimuove il prefisso della versione dal nome del segreto e restituisce il resto del nome del segreto per il caricamento nelle coppie nome-valore della configurazione dell'app.

Quando viene implementato questo approccio:

1. La versione dell'app specificata nel file di progetto dell'app. Nell'esempio seguente la versione dell'app è impostata su `5.0.0.0` :

   ```xml
   <PropertyGroup>
     <Version>5.0.0.0</Version>
   </PropertyGroup>
   ```

1. Verificare che `<UserSecretsId>` sia presente una proprietà nel file di progetto dell'app, dove `{GUID}` è un GUID fornito dall'utente:

   ```xml
   <PropertyGroup>
     <UserSecretsId>{GUID}</UserSecretsId>
   </PropertyGroup>
   ```

   Salvare i segreti seguenti localmente con lo [strumento di gestione dei segreti](xref:security/app-secrets):

   ```dotnetcli
   dotnet user-secrets set "5000-AppSecret" "5.0.0.0_secret_value_dev"
   dotnet user-secrets set "5100-AppSecret" "5.1.0.0_secret_value_dev"
   ```

1. I segreti vengono salvati in Azure Key Vault usando i comandi dell'interfaccia della riga di comando di Azure seguenti:

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5000-AppSecret" --value "5.0.0.0_secret_value_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5100-AppSecret" --value "5.1.0.0_secret_value_prod"
   ```

1. Quando l'app viene eseguita, vengono caricati i segreti dell'insieme di credenziali delle chiavi. La stringa segreta per `5000-AppSecret` viene confrontata con la versione dell'app specificata nel file di progetto dell'app ( `5.0.0.0` ).

1. La versione `5000` (con il trattino) viene rimossa dal nome della chiave. In tutta l'app, la lettura della configurazione con la chiave `AppSecret` carica il valore del segreto.

1. Se la versione dell'app viene modificata nel file di progetto in `5.1.0.0` e l'app viene nuovamente eseguita, il valore del segreto restituito è `5.1.0.0_secret_value_dev` nell'ambiente di sviluppo e `5.1.0.0_secret_value_prod` in produzione.

> [!NOTE]
> È anche possibile fornire un' <xref:Microsoft.Azure.KeyVault.KeyVaultClient> implementazione personalizzata a <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> . Un client personalizzato consente la condivisione di una singola istanza del client nell'app.

## <a name="bind-an-array-to-a-class"></a>Associare una matrice a una classe

Il provider è in grado di leggere i valori di configurazione in una matrice per l'associazione a una matrice POCO.

Quando si legge da un'origine di configurazione che consente alle chiavi di contenere separatori di due punti ( `:` ), viene usato un segmento di chiave numerico per distinguere le chiavi che costituiscono una matrice ( `:0:` , `:1:` , &hellip; `:{n}:` ). Per altre informazioni, vedere [Configuration: associare una matrice a una classe](xref:fundamentals/configuration/index#bind-an-array-to-a-class).

Azure Key Vault chiavi non possono utilizzare i due punti come separatore. L'approccio descritto in questo argomento USA trattini doppi ( `--` ) come separatore per i valori gerarchici (sezioni). Le chiavi di matrice vengono archiviate in Azure Key Vault con trattini doppi e segmenti di chiave numerica ( `--0--` , `--1--` , &hellip; `--{n}--` ).

Esaminare la seguente configurazione del provider di registrazione [Serilog](https://serilog.net/) fornita da un file JSON. Nella matrice sono definiti due valori letterali di oggetto `WriteTo` che riflettono due *sink* Serilog, che descrivono le destinazioni per la registrazione dell'output:

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

La configurazione mostrata nel file JSON precedente viene archiviata in Azure Key Vault usando la notazione a doppio trattino ( `--` ) e i segmenti numerici:

| Chiave | Valore |
| --- | ----- |
| `Serilog--WriteTo--0--Name` | `AzureTableStorage` |
| `Serilog--WriteTo--0--Args--storageTableName` | `logs` |
| `Serilog--WriteTo--0--Args--connectionString` | `DefaultEnd...ountKey=Eby8...GMGw==` |
| `Serilog--WriteTo--1--Name` | `AzureDocumentDB` |
| `Serilog--WriteTo--1--Args--endpointUrl` | `https://contoso.documents.azure.com:443` |
| `Serilog--WriteTo--1--Args--authorizationKey` | `Eby8...GMGw==` |

## <a name="reload-secrets"></a>Ricarica segreti

I segreti vengono memorizzati nella cache finché non `IConfigurationRoot.Reload()` viene chiamato. I segreti scaduti, disabilitati e aggiornati nell'insieme di credenziali delle chiavi non vengono rispettati dall'app fino a quando non `Reload` viene eseguito.

```csharp
Configuration.Reload();
```

## <a name="disabled-and-expired-secrets"></a>Segreti disabilitati e scaduti

I segreti disabilitati e scaduti generano un' <xref:Microsoft.Azure.KeyVault.Models.KeyVaultErrorException> . Per impedire che l'app venga generata, fornire la configurazione usando un provider di configurazione diverso o aggiornare il segreto disabilitato o scaduto.

## <a name="troubleshoot"></a>Risolvere problemi

Quando l'app non riesce a caricare la configurazione usando il provider, viene scritto un messaggio di errore nell' [infrastruttura di registrazione ASP.NET Core](xref:fundamentals/logging/index). Le condizioni seguenti impediranno il caricamento della configurazione:

* L'app o il certificato non è configurato correttamente in Azure Active Directory.
* L'insieme di credenziali delle chiavi non esiste in Azure Key Vault.
* L'app non è autorizzata ad accedere all'insieme di credenziali delle chiavi.
* Il criterio di accesso non include le `Get` `List` autorizzazioni e.
* Nell'insieme di credenziali delle chiavi i dati di configurazione (coppia nome-valore) sono denominati, mancanti, disabilitati o scaduti in modo errato.
* L'app presenta il nome dell'insieme di credenziali delle chiavi errato ( `KeyVaultName` ), Azure ad ID applicazione ( `AzureADApplicationId` ) o l'identificazione personale del certificato Azure ad ( `AzureADCertThumbprint` ).
* La chiave di configurazione (nome) non è corretta nell'app per il valore che si sta provando a caricare.
* Quando si aggiungono i criteri di accesso per l'app all'insieme di credenziali delle chiavi, il criterio è stato creato, ma il pulsante **Salva** non è stato selezionato nell'interfaccia utente dei **criteri di accesso** .

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/configuration/index>
* [Microsoft Azure: Key Vault](https://azure.microsoft.com/services/key-vault/)
* [Microsoft Azure: Key Vault documentazione](/azure/key-vault/)
* [Come generare e trasferire chiavi HSM protette per Azure Key Vault](/azure/key-vault/key-vault-hsm-protected-keys)
* [Classe KeyVaultClient](/dotnet/api/microsoft.azure.keyvault.keyvaultclient)
* [Guida introduttiva: Impostare e recuperare un segreto da Azure Key Vault tramite un'app Web .NET](/azure/key-vault/quick-create-net)
* [Esercitazione: Come usare Azure Key Vault con una macchina virtuale Windows di Azure in .NET](/azure/key-vault/tutorial-net-windows-virtual-machine)

::: moniker-end
