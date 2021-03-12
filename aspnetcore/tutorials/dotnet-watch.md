---
title: Sviluppare app ASP.NET Core usando un watcher per file
author: rick-anderson
description: Questa esercitazione illustra come installare e usare lo strumento watcher per file (dotnet watch) dell'interfaccia della riga di comando di .NET Core in un'app ASP.NET Core.
ms.author: riande
ms.date: 05/31/2018
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
uid: tutorials/dotnet-watch
ms.openlocfilehash: 84cae3b3babe28c2ebf6dba50023b020112d1bb3
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587580"
---
# <a name="develop-aspnet-core-apps-using-a-file-watcher"></a>Sviluppare app ASP.NET Core usando un watcher per file

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Victor Hurdugaci](https://twitter.com/victorhurdugaci)

`dotnet watch` è uno strumento che esegue un comando [interfaccia della riga di comando di .NET Core](/dotnet/core/tools) quando i file di origine cambiano. Ad esempio, una modifica di file può attivare la compilazione, l'esecuzione di test o la distribuzione.

Questa esercitazione usa un'API Web esistente con due endpoint, di cui uno restituisce una somma e l'altro un prodotto. Il metodo del prodotto ha un bug, che verrà corretto in questa esercitazione.

Scaricare l' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/dotnet-watch/sample). Questa è costituita da due progetti: *WebApp* (un'API Web di ASP.NET Core) e *WebAppTests* (unit test per l'API Web).

In una shell dei comandi passare alla cartella *WebApp*. Eseguire il comando seguente:

```dotnetcli
dotnet run
```

> [!NOTE]
> È possibile usare `dotnet run --project <PROJECT>` per specificare un progetto da eseguire. Ad esempio, l'esecuzione di `dotnet run --project WebApp` dalla radice dell'app di esempio consentirà di eseguire anche il progetto *WebApp*.

L'output della console visualizza messaggi simili al seguente che indicano che l'app è in esecuzione e in attesa di richieste:

```console
$ dotnet run
Hosting environment: Development
Content root path: C:/Docs/aspnetcore/tutorials/dotnet-watch/sample/WebApp
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

In un Web browser passare a `http://localhost:<port number>/api/math/sum?a=4&b=5`. Dovrebbe essere visualizzato il risultato `9`.

Passare all'API del prodotto (`http://localhost:<port number>/api/math/product?a=4&b=5`). Restituisce `9` e non `20` come previsto. Questo problema verrà corretto più avanti nell'esercitazione.

::: moniker range="<= aspnetcore-2.0"

## <a name="add-dotnet-watch-to-a-project"></a>Aggiungere `dotnet watch` a un progetto

Lo strumento watcher per file `dotnet watch` è incluso nella versione 2.1.300 di .NET Core SDK. Se si usa una versione precedente di .NET Core SDK, i passaggi seguenti sono obbligatori.

1. Aggiungere un riferimento al pacchetto `Microsoft.DotNet.Watcher.Tools` nel file *.csproj*:

    ```xml
    <ItemGroup>
        <DotNetCliToolReference Include="Microsoft.DotNet.Watcher.Tools" Version="2.0.0" />
    </ItemGroup>
    ```

1. Installare il pacchetto `Microsoft.DotNet.Watcher.Tools` eseguendo il comando seguente:

    ```dotnetcli
    dotnet restore
    ```

::: moniker-end

## <a name="run-net-core-cli-commands-using-dotnet-watch"></a>Eseguire i comandi dell'interfaccia della riga di comando di .NET Core con `dotnet watch`

Qualsiasi [comando dell'interfaccia della riga di comando di .NET Core](/dotnet/core/tools#cli-commands) può essere eseguito con `dotnet watch`. Ad esempio:

| Comando | Comando con watch |
| ---- | ----- |
| dotnet run | dotnet watch run |
| esecuzione DotNet-f netcoreapp 3.1 | esecuzione di DotNet Watch-f netcoreapp 3.1 |
| DotNet Run-f netcoreapp 3.1----arg1 | esecuzione di DotNet Watch-f netcoreapp 3.1----arg1 |
| dotnet test | dotnet watch test |

Eseguire `dotnet watch run` nella cartella *WebApp*. L'output della console indica che `watch` è stato avviato.

::: moniker range=">= aspnetcore-5.0"
In esecuzione `dotnet watch run` in un'app Web viene avviato un browser che consente di passare all'URL dell'app al termine della preparazione. `dotnet watch` a tale scopo, leggere l'output della console dell'app e attendere il messaggio pronto visualizzato da <xref:Microsoft.AspNetCore.WebHost> .

`dotnet watch` Aggiorna il browser quando rileva le modifiche apportate ai file osservati. A tale scopo, il comando Watch inserisce un middleware nell'app che modifica le risposte HTML create dall'app. Il middleware aggiunge un blocco di script JavaScript alla pagina che consente `dotnet watch` di impostare il browser per l'aggiornamento. Attualmente, le modifiche apportate a tutti i file controllati, incluso il contenuto statico, ad esempio i file *HTML* e *CSS* , determinano la ricompilazione dell'app.

`dotnet watch`:

* Controlla solo i file che incidono sulle compilazioni per impostazione predefinita.
* Eventuali altri file controllati (tramite configurazione) continuano a generare una compilazione.

Per ulteriori informazioni sulla configurazione, vedere la pagina relativa alla [configurazione di DotNet-Watch](#dotnet-watch-configuration) in questo documento.

::: moniker-end

> [!NOTE]
> È possibile usare `dotnet watch --project <PROJECT>` per specificare un progetto da controllare. Ad esempio, l'esecuzione di `dotnet watch --project WebApp run` dalla radice dell'app di esempio consentirà di eseguire e controllare anche il progetto *WebApp*.

## <a name="make-changes-with-dotnet-watch"></a>Apportare modifiche con `dotnet watch`

Assicurarsi che `dotnet watch` sia in esecuzione.

Correggere il bug nel metodo `Product` di *MathController.cs* in modo che restituisca il prodotto e non la somma:

```csharp
public static int Product(int a, int b)
{
    return a * b;
}
```

Salvare il file. L'output della console indica che `dotnet watch` ha rilevato una modifica del file e ha riavviato l'app.

Verificare che `http://localhost:<port number>/api/math/product?a=4&b=5` restituisca il risultato corretto.

## <a name="run-tests-using-dotnet-watch"></a>Eseguire test con `dotnet watch`

1. Modificare il metodo `Product` di *MathController.cs* in modo che restituisca la somma. Salvare il file.
1. In una shell dei comandi passare alla cartella *WebAppTests*.
1. Eseguire [dotnet restore](/dotnet/core/tools/dotnet-restore).
1. Eseguire `dotnet watch test`. L'output indica che un test non è stato superato e che il watcher è in attesa di modifiche ai file:

     ```console
     Total tests: 2. Passed: 1. Failed: 1. Skipped: 0.
     Test Run Failed.
     ```

1. Correggere il codice del metodo `Product` in modo che restituisca il prodotto. Salvare il file.

`dotnet watch` rileva la modifica ai file ed esegue di nuovo i test. L'output della console indica che i test sono stati superati.

## <a name="customize-files-list-to-watch"></a>Personalizzare l'elenco dei file da controllare

Per impostazione predefinita, `dotnet-watch` tiene traccia di tutti i file che soddisfano i criteri GLOB seguenti:

* `**/*.cs`
* `*.csproj`
* `**/*.resx`

È possibile aggiungere altri elementi all'elenco di controllo modificando il file con estensione *csproj*. Gli elementi possono essere specificati singolarmente o tramite criteri GLOB.

```xml
<ItemGroup>
    <!-- extends watching group to include *.js files -->
    <Watch Include="**\*.js" Exclude="node_modules\**\*;**\*.js.map;obj\**\*;bin\**\*" />
</ItemGroup>
```

## <a name="opt-out-of-files-to-be-watched"></a>Esclusione di file dal controllo

È possibile configurare `dotnet-watch` in modo che ignori le impostazioni predefinite. Per ignorare file specifici, aggiungere l'attributo `Watch="false"` alla definizione di un elemento nel file con estensione *csproj*:

```xml
<ItemGroup>
    <!-- exclude Generated.cs from dotnet-watch -->
    <Compile Include="Generated.cs" Watch="false" />

    <!-- exclude Strings.resx from dotnet-watch -->
    <EmbeddedResource Include="Strings.resx" Watch="false" />

    <!-- exclude changes in this referenced project -->
    <ProjectReference Include="..\ClassLibrary1\ClassLibrary1.csproj" Watch="false" />
</ItemGroup>
```

## <a name="custom-watch-projects"></a>Progetti di controllo personalizzati

`dotnet-watch` non è limitato a progetti C#. È possibile creare progetti di controllo personalizzati per gestire scenari diversi. Si consideri il layout di progetto seguente:

* **test**
  * *UnitTests/UnitTests.csproj*
  * *IntegrationTests/IntegrationTests.csproj*

Se l'obiettivo consiste nel controllare entrambi i progetti, creare un file di progetto personalizzato configurato per controllarli entrambi:

```xml
<Project>
    <ItemGroup>
        <TestProjects Include="**\*.csproj" />
        <Watch Include="**\*.cs" />
    </ItemGroup>

    <Target Name="Test">
        <MSBuild Targets="VSTest" Projects="@(TestProjects)" />
    </Target>

    <Import Project="$(MSBuildExtensionsPath)\Microsoft.Common.targets" />
</Project>
```

Per avviare il controllo dei file per entrambi i progetti, passare alla cartella *test*. Eseguire il comando seguente:

```dotnetcli
dotnet watch msbuild /t:Test
```

VSTest viene eseguito quando un qualsiasi file viene modificato in uno dei progetti di test.

## <a name="dotnet-watch-configuration"></a>configurazione di DotNet-Watch

Alcune opzioni di configurazione possono essere passate a `dotnet watch` tramite le variabili di ambiente. Le variabili disponibili sono:

| Impostazione  | Descrizione |
| ------------- | ------------- |
| `DOTNET_USE_POLLING_FILE_WATCHER`                | Se impostato su "1" o "true", `dotnet watch` utilizza un Watcher del file di polling anziché CoreFx `FileSystemWatcher` . Usato quando si controllano i file nelle condivisioni di rete o nei volumi montati docker.                       |
| `DOTNET_WATCH_SUPPRESS_MSBUILD_INCREMENTALISM`   | Per impostazione predefinita, `dotnet watch` ottimizza la compilazione evitando determinate operazioni, ad esempio l'esecuzione del ripristino o la rivalutazione del set di file controllati per ogni modifica apportata al file. Se impostato su "1" o "true", queste ottimizzazioni sono disabilitate. |
| `DOTNET_WATCH_SUPPRESS_LAUNCH_BROWSER`   | `dotnet watch run` tenta di avviare browser per le app Web con `launchBrowser` configurato in *launchSettings.js*. Se impostato su "1" o "true", questo comportamento viene eliminato. |
| `DOTNET_WATCH_SUPPRESS_BROWSER_REFRESH`   | `dotnet watch run` tenta di aggiornare i browser quando rileva le modifiche al file. Se impostato su "1" o "true", questo comportamento viene eliminato. Questo comportamento viene anche eliminato se `DOTNET_WATCH_SUPPRESS_LAUNCH_BROWSER` è impostato. |
