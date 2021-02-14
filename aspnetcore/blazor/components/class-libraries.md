---
title: RazorLibrerie di classi dei componenti ASP.NET Core
author: guardrex
description: Scopri in che modo i componenti possono essere inclusi nelle Blazor app da una libreria di componenti esterna.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/12/2021
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
uid: blazor/components/class-libraries
ms.openlocfilehash: fed5d26ecd73a4710ee794c413fd51e0b4cb4913
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280198"
---
# <a name="aspnet-core-razor-components-class-libraries"></a>RazorLibrerie di classi dei componenti ASP.NET Core

I componenti possono essere condivisi in una [ Razor libreria di classi (RCL)](xref:razor-pages/ui-class) nei progetti. È possibile includere una *Razor libreria di classi Components* da:

* Un altro progetto nella soluzione.
* Un pacchetto NuGet.
* Libreria .NET A cui si fa riferimento.

Così come i componenti sono tipi .NET normali, i componenti forniti da un RCL sono assembly .NET normali.

## <a name="create-an-rcl"></a>Creare un RCL

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Creare un nuovo progetto.
1. Selezionare **Razor libreria di classi**. Selezionare **Avanti**.
1. Nella finestra di dialogo **Crea una nuova Razor libreria di classi** Selezionare **Crea**.
1. Specificare il nome di un progetto nel campo **Nome progetto** oppure accettare il nome predefinito. Gli esempi in questo argomento usano il nome del progetto `ComponentLibrary` . Selezionare **Crea**.
1. Aggiungere RCL a una soluzione:
   1. Fare clic con il pulsante destro del mouse sulla soluzione. Selezionare **Aggiungi**  >  **progetto esistente**.
   1. Passare al file di progetto di RCL.
   1. Selezionare il file di progetto di RCL ( `.csproj` ).
1. Aggiungere un riferimento a RCL dall'app:
   1. Fare clic con il pulsante destro del mouse sul progetto app. Selezionare **Aggiungi**  >  **riferimento**.
   1. Selezionare il progetto RCL. Selezionare **OK**.

> [!NOTE]
> Se la casella di controllo **pagine e visualizzazioni di supporto** è selezionata durante la generazione del RCL dal modello, aggiungere anche un `_Imports.razor` file alla radice del progetto generato con il contenuto seguente per abilitare la Razor creazione dei componenti:
>
> ```razor
> @using Microsoft.AspNetCore.Components.Web
> ```
>
> Aggiungere manualmente il file alla radice del progetto generato.

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

1. Usare il modello **Razor libreria di classi** ( `razorclasslib` ) con il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando in una shell dei comandi. Nell'esempio seguente viene creato un RCL denominato `ComponentLibrary` . La cartella che include `ComponentLibrary` viene creata automaticamente quando si esegue il comando:

   ```dotnetcli
   dotnet new razorclasslib -o ComponentLibrary
   ```

   > [!NOTE]
   > Se l' `-s|--support-pages-and-views` opzione viene utilizzata durante la generazione del RCL dal modello, aggiungere anche un `_Imports.razor` file alla radice del progetto generato con il contenuto seguente per abilitare la Razor creazione dei componenti:
   >
   > ```razor
   > @using Microsoft.AspNetCore.Components.Web
   > ```
   >
   > Aggiungere manualmente il file alla radice del progetto generato.

1. Per aggiungere la libreria a un progetto esistente, usare il [`dotnet add reference`](/dotnet/core/tools/dotnet-add-reference) comando in una shell dei comandi. Nell'esempio seguente viene aggiunto RCL all'app. Eseguire il comando seguente dalla cartella del progetto dell'app con il percorso della libreria:

   ```dotnetcli
   dotnet add reference {PATH TO LIBRARY}
   ```

---

## <a name="consume-a-library-component"></a>Utilizzare un componente di libreria

Per utilizzare i componenti definiti in una raccolta in un altro progetto, utilizzare uno degli approcci seguenti:

* Usare il nome completo del tipo con lo spazio dei nomi.
* Usare Razor la [`@using`](xref:mvc/views/razor#using) direttiva. I singoli componenti possono essere aggiunti in base al nome.

Negli esempi seguenti `ComponentLibrary` è una libreria di componenti contenente il `Component1` componente ( `Component1.razor` ). Il `Component1` componente è un componente di esempio aggiunto automaticamente dal modello di progetto RCL al momento della creazione della libreria.

Fare riferimento al `Component1` componente utilizzando il relativo spazio dei nomi:

```razor
<h1>Hello, world!</h1>

Welcome to your new app.

<ComponentLibrary.Component1 />
```

In alternativa, portare la libreria nell'ambito con una [`@using`](xref:mvc/views/razor#using) direttiva e usare il componente senza il relativo spazio dei nomi:

```razor
@using ComponentLibrary

<h1>Hello, world!</h1>

Welcome to your new app.

<Component1 />
```

Facoltativamente, includere la `@using ComponentLibrary` direttiva nel file di primo livello `_Import.razor` per rendere disponibili i componenti della libreria a un intero progetto. Aggiungere la direttiva a un `_Import.razor` file a qualsiasi livello per applicare lo spazio dei nomi a un singolo componente o a un set di componenti all'interno di una cartella.

::: moniker range=">= aspnetcore-5.0"

Per i componenti di libreria che usano l' [isolamento CSS](xref:blazor/components/css-isolation), non è necessario collegare in modo esplicito i fogli di stile dei singoli componenti della libreria nell'app che utilizza la libreria. Gli stili dei componenti vengono resi disponibili automaticamente per l'app di consumo.

<!-- REACTIVATE WHEN HEAD COMPONENTS COME BACK AT 6.0

To provide additional library component styles from stylesheets in the library's `wwwroot` folder, link the stylesheets using the framework's [`Link` component](xref:blazor/fundamentals/signalr#influence-html-head-tag-elements) in `Component1.razor`:

```razor
<div class="my-component">
    <Link href="_content/ComponentLibrary/styles.css" rel="stylesheet" />

    <p>
        This Blazor component is defined in the <strong>ComponentLibrary</strong> package.
    </p>
</div>
```

NEXT PARAGRAPH: RECAST TO 'CAN ALSO ADOPT ...'

-->

Per fornire stili di componenti di libreria aggiuntivi dai fogli di stile nella `wwwroot` cartella della libreria, collegare i fogli di stile nel `wwwroot/index.html` file () o nel file dell'app che lo utilizza () Blazor WebAssembly `Pages/_Host.cshtml` Blazor Server :

```html
<head>
    ...
    <link href="_content/ComponentLibrary/additionalStyles.css" rel="stylesheet" />
</head>
```

<!-- REACTIVATE WHEN HEAD COMPONENTS COME BACK AT 6.0

When the `Link` component is used in a child component, the linked asset is also available to any other child component of the parent component as long as the child with the `Link` component is rendered. The distinction between using the `Link` component in a child component and placing a `<link>` HTML tag in `wwwroot/index.html` or `Pages/_Host.cshtml` is that a framework component's rendered HTML tag:

* Can be modified by application state. A hard-coded `<link>` HTML tag can't be modified by application state.
* Is removed from the HTML `<head>` when the parent component is no longer rendered.

-->

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Per specificare `Component1` la `my-component` classe CSS, collegarsi al foglio di stile della libreria nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel file dell'app `Pages/_Host.cshtml` ( Blazor Server ):

```html
<head>
    ...
    <link href="_content/ComponentLibrary/styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

## <a name="create-a-razor-components-class-library-with-static-assets"></a>Creare una Razor libreria di classi di componenti con asset statici

Un RCL può includere asset statici. Gli asset statici sono disponibili per qualsiasi app che usa la libreria. Per altre informazioni, vedere <xref:razor-pages/ui-class#create-an-rcl-with-static-assets>.

## <a name="supply-components-and-static-assets-to-multiple-hosted-blazor-apps"></a>Fornire componenti e risorse statiche a più app ospitate Blazor

Per altre informazioni, vedere <xref:blazor/host-and-deploy/webassembly#static-assets-and-class-libraries>.

::: moniker range=">= aspnetcore-5.0"

## <a name="browser-compatibility-analyzer-for-blazor-webassembly"></a>Analizzatore compatibilità browser per Blazor WebAssembly

Blazor WebAssembly le app sono destinate alla superficie di attacco dell'API .NET completa, ma non tutte le API .NET sono supportate nel webassembly a causa dei vincoli del sandbox del browser. Le API non supportate vengono generate <xref:System.PlatformNotSupportedException> durante l'esecuzione su webassembly. Un analizzatore della compatibilità della piattaforma avvisa lo sviluppatore quando l'app usa API non supportate dalle piattaforme di destinazione dell'app. Per le Blazor WebAssembly app, questo significa verificare che le API siano supportate nei browser. L'annotazione delle API di .NET Framework per Compatibility Analyzer è un processo in corso, quindi non tutte le API .NET Framework sono attualmente annotate.

Blazor WebAssembly e Razor i progetti libreria di classi abilitano *automaticamente* i controlli compatibilità del browser aggiungendo `browser` come piattaforma supportata con l' `SupportedPlatform` elemento MSBuild. Gli sviluppatori di librerie possono aggiungere manualmente l' `SupportedPlatform` elemento al file di progetto di una raccolta per abilitare la funzionalità:

```xml
<ItemGroup>
  <SupportedPlatform Include="browser" />
</ItemGroup>
```

Quando si crea una libreria, indicare che un'API specifica non è supportata nei browser specificando `browser` <xref:System.Runtime.Versioning.UnsupportedOSPlatformAttribute> :

```csharp
[UnsupportedOSPlatform("browser")]
private static string GetLoggingDirectory()
{
    ...
}
```

Per altre informazioni, vedere [annotazione delle API come non supportate in piattaforme specifiche (archivio GitHub DotNet/Designs](https://github.com/dotnet/designs/blob/main/accepted/2020/platform-exclusion/platform-exclusion.md#build-configuration-for-platforms).

## <a name="blazor-javascript-isolation-and-object-references"></a>Blazor Isolamento JavaScript e riferimenti a oggetti

Blazor Abilita l'isolamento JavaScript nei [moduli JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)standard. L'isolamento JavaScript offre i vantaggi seguenti:

* Il codice JavaScript importato non inquina più lo spazio dei nomi globale.
* I consumer della libreria e dei componenti non devono importare manualmente il codice JavaScript correlato.

Per altre informazioni, vedere <xref:blazor/call-javascript-from-dotnet#blazor-javascript-isolation-and-object-references>.

::: moniker-end

## <a name="build-pack-and-ship-to-nuget"></a>Compilare, comprimere e spedire a NuGet

Poiché le librerie dei componenti sono librerie .NET standard, la creazione di pacchetti e la spedizione a NuGet non è diversa dalla creazione di pacchetti e dalla distribuzione di qualsiasi libreria a NuGet. La creazione di pacchetti viene eseguita usando il [`dotnet pack`](/dotnet/core/tools/dotnet-pack) comando in una shell dei comandi:

```dotnetcli
dotnet pack
```

Caricare il pacchetto in NuGet usando il [`dotnet nuget push`](/dotnet/core/tools/dotnet-nuget-push) comando in una shell dei comandi.

## <a name="additional-resources"></a>Risorse aggiuntive

::: moniker range=">= aspnetcore-5.0"

* <xref:razor-pages/ui-class>
* [Aggiungere un file di configurazione del trimmer XML Intermediate Language (IL) a una raccolta](xref:blazor/host-and-deploy/configure-trimmer)
* [Supporto dell'isolamento CSS con le Razor librerie di classi](xref:blazor/components/css-isolation#razor-class-library-rcl-support)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <xref:razor-pages/ui-class>
* [Aggiungere un file di configurazione del linker (IL) XML Intermediate Language (IL) a una raccolta](xref:blazor/host-and-deploy/configure-linker#add-an-xml-linker-configuration-file-to-a-library)

::: moniker-end
