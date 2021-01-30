---
title: Provider di archiviazione chiavi in ASP.NET Core
author: rick-anderson
description: Informazioni sui provider di archiviazione chiavi in ASP.NET Core e su come configurare i percorsi di archiviazione delle chiavi.
ms.author: riande
ms.date: 12/05/2019
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
uid: security/data-protection/implementation/key-storage-providers
ms.openlocfilehash: e4cf10d09c1629afb298aef0c2b86ad3bf7b646c
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057369"
---
# <a name="key-storage-providers-in-aspnet-core"></a>Provider di archiviazione chiavi in ASP.NET Core

Per impostazione predefinita, il sistema di protezione dei dati [utilizza un meccanismo di individuazione](xref:security/data-protection/configuration/default-settings) per determinare dove devono essere rese permanente le chiavi crittografiche. Lo sviluppatore può eseguire l'override del meccanismo di individuazione predefinito e specificare manualmente il percorso.

> [!WARNING]
> Se si specifica un percorso di persistenza della chiave esplicito, il sistema di protezione dei dati Annulla la registrazione del meccanismo di crittografia delle chiavi predefinite, in modo che le chiavi non vengano più crittografate a riposo. Si consiglia inoltre di [specificare un meccanismo di crittografia a chiave esplicito](xref:security/data-protection/implementation/key-encryption-at-rest) per le distribuzioni di produzione.

## <a name="file-system"></a>File system

Per configurare un repository di chiavi basato su file system, chiamare la routine di configurazione [PersistKeysToFileSystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem) , come illustrato di seguito. Fornire un oggetto [DirectoryInfo](/dotnet/api/system.io.directoryinfo) che punta al repository in cui devono essere archiviate le chiavi:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"c:\temp-keys\"));
}
```

Il `DirectoryInfo` può puntare a una directory nel computer locale oppure può puntare a una cartella in una condivisione di rete. Se si fa riferimento a una directory nel computer locale (e lo scenario è che solo le app nel computer locale richiedono l'accesso per usare questo repository), provare a usare [Windows DPAPI](xref:security/data-protection/implementation/key-encryption-at-rest) (in Windows) per crittografare le chiavi inattive. In caso contrario, prendere in considerazione l'uso di un [certificato X. 509](xref:security/data-protection/implementation/key-encryption-at-rest) per crittografare le chiavi inattive.

## <a name="azure-storage"></a>Archiviazione di Azure

Il pacchetto [Azure. Extensions. AspNetCore. dataprotection. Blobs](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Blobs) consente di archiviare le chiavi di protezione dei dati nell'archivio BLOB di Azure. Le chiavi possono essere condivise tra più istanze di un'app Web. Le app possono condividere la cookie protezione CSRF o di autenticazione tra più server.

Per configurare il provider di archiviazione BLOB di Azure, chiamare uno degli overload [PersistKeysToAzureBlobStorage](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.persistkeystoazureblobstorage) .

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blob URI including SAS token>"));
}
```

Se l'app Web è in esecuzione come servizio di Azure, è possibile usare la stringa di connessione per l'autenticazione nell'archiviazione di Azure usando [Azure. storage. Blobs](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.blobcontainerclient).

```csharp
string connectionString = "<connection_string>";
string containerName = "my-key-container";
BlobContainerClient container = new BlobContainerClient(connectionString, containerName);

// optional - provision the container automatically
await container.CreateIfNotExistsAsync();

services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(container, "keys.xml");
```

> [!NOTE]
> La stringa di connessione all'account di archiviazione è disponibile nel portale di Azure sotto la sezione "chiavi di accesso" oppure eseguendo il comando dell'interfaccia della riga di comando seguente: 
> ```bash
> az storage account show-connection-string --name <account_name> --resource-group <resource_group>
> ```

## <a name="redis"></a>Redis

::: moniker range=">= aspnetcore-2.2"

Il pacchetto [Microsoft. AspNetCore. dataprotection. StackExchangeRedis](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.StackExchangeRedis/) consente di archiviare le chiavi di protezione dei dati in una cache Redis. Le chiavi possono essere condivise tra più istanze di un'app Web. Le app possono condividere la cookie protezione CSRF o di autenticazione tra più server.

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Il pacchetto [Microsoft. AspNetCore. dataprotection. Redis](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Redis/) consente di archiviare le chiavi di protezione dei dati in una cache Redis. Le chiavi possono essere condivise tra più istanze di un'app Web. Le app possono condividere la cookie protezione CSRF o di autenticazione tra più server.

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

Per eseguire la configurazione in Redis, chiamare uno degli overload di [PersistKeysToStackExchangeRedis](/dotnet/api/microsoft.aspnetcore.dataprotection.stackexchangeredisdataprotectionbuilderextensions.persistkeystostackexchangeredis) :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var redis = ConnectionMultiplexer.Connect("<URI>");
    services.AddDataProtection()
        .PersistKeysToStackExchangeRedis(redis, "DataProtection-Keys");
}
```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Per eseguire la configurazione in Redis, chiamare uno degli overload di [PersistKeysToRedis](/dotnet/api/microsoft.aspnetcore.dataprotection.redisdataprotectionbuilderextensions.persistkeystoredis) :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var redis = ConnectionMultiplexer.Connect("<URI>");
    services.AddDataProtection()
        .PersistKeysToRedis(redis, "DataProtection-Keys");
}
```

::: moniker-end

Per altre informazioni, vedere i seguenti argomenti:

* [StackExchange. Redis ConnectionMultiplexer](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Basics.md)
* [Cache Redis di Azure](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache#connect-to-the-cache)
* [Esempi di ASP.NET Core DataProtection](https://github.com/dotnet/AspNetCore/tree/2.2.0/src/DataProtection/samples)

## <a name="registry"></a>Registro

**Si applica solo alle distribuzioni di Windows.**

In alcuni casi l'app potrebbe non avere accesso in scrittura alla file system. Si consideri uno scenario in cui un'app è in esecuzione come account del servizio virtuale, ad esempio l'identità del pool di app *w3wp.exe*. In questi casi, l'amministratore può eseguire il provisioning di una chiave del registro di sistema accessibile dall'identità dell'account del servizio. Chiamare il metodo di estensione [PersistKeysToRegistry](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystoregistry) , come illustrato di seguito. Specificare un [RegistryKey](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.registryxmlrepository.registrykey) che punta alla posizione in cui devono essere archiviate le chiavi crittografiche:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToRegistry(Registry.CurrentUser.OpenSubKey(@"SOFTWARE\Sample\keys", true));
}
```

> [!IMPORTANT]
> Si consiglia di utilizzare [Windows DPAPI](xref:security/data-protection/implementation/key-encryption-at-rest) per crittografare le chiavi inattive.

::: moniker range=">= aspnetcore-2.2"

## <a name="entity-framework-core"></a>Entity Framework Core

Il pacchetto [Microsoft. AspNetCore. dataprotection. EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.EntityFrameworkCore/) fornisce un meccanismo per archiviare le chiavi di protezione dati in un database usando Entity Framework Core. Il `Microsoft.AspNetCore.DataProtection.EntityFrameworkCore` pacchetto NuGet deve essere aggiunto al file di progetto e non è incluso nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).

Con questo pacchetto, le chiavi possono essere condivise tra più istanze di un'app Web.

Per configurare il provider di EF Core, chiamare il metodo [PersistKeysToDbContext \<TContext> ](/dotnet/api/microsoft.aspnetcore.dataprotection.entityframeworkcoredataprotectionextensions.persistkeystodbcontext) :

[!code-csharp[Main](key-storage-providers/sample/Startup.cs?name=snippet&highlight=13-20)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

Il parametro generico, `TContext` , deve ereditare da [DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) e implementare [IDataProtectionKeyContext](/dotnet/api/microsoft.aspnetcore.dataprotection.entityframeworkcore.idataprotectionkeycontext):

[!code-csharp[Main](key-storage-providers/sample/MyKeysContext.cs)]

Creare la `DataProtectionKeys` tabella.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Eseguire i comandi seguenti nella finestra **console di gestione pacchetti** (PMC):

```powershell
Add-Migration AddDataProtectionKeys -Context MyKeysContext
Update-Database -Context MyKeysContext
```

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

Eseguire i comandi seguenti in una shell dei comandi:

```dotnetcli
dotnet ef migrations add AddDataProtectionKeys --context MyKeysContext
dotnet ef database update --context MyKeysContext
```

---

`MyKeysContext` è l'oggetto `DbContext` definito nel codice di esempio precedente. Se si usa un `DbContext` con un nome diverso, sostituire con il `DbContext` nome `MyKeysContext` .

La `DataProtectionKeys` classe/entità adotta la struttura illustrata nella tabella seguente.

| Proprietà/campo | Tipo CLR | Tipo SQL              |
| -------------- | -------- | --------------------- |
| `Id`           | `int`    | `int`, PK, `IDENTITY(1,1)` , not null   |
| `FriendlyName` | `string` | `nvarchar(MAX)`, null |
| `Xml`          | `string` | `nvarchar(MAX)`, null |

::: moniker-end

## <a name="custom-key-repository"></a>Repository di chiavi personalizzate

Se i meccanismi predefiniti non sono appropriati, lo sviluppatore può specificare il proprio meccanismo di persistenza della chiave fornendo un [IXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.ixmlrepository)personalizzato.
