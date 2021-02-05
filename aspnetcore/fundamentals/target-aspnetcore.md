---
title: Usare API di ASP.NET Core in una libreria di classi
author: scottaddie
description: Informazioni su come usare le API ASP.NET Core in una libreria di classi.
ms.author: scaddie
ms.custom: mvc
ms.date: 12/16/2019
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
uid: fundamentals/target-aspnetcore
ms.openlocfilehash: c012658a6f48247af60c8bfd56a7d987f6aa8a68
ms.sourcegitcommit: c1839f2992b003c92cd958244a2e0771ae928786
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/05/2021
ms.locfileid: "93061509"
---
# <a name="use-aspnet-core-apis-in-a-class-library"></a>Usare API di ASP.NET Core in una libreria di classi

Di [Scott Addie](https://github.com/scottaddie)

Questo documento fornisce indicazioni per l'uso di API ASP.NET Core in una libreria di classi. Per tutte le altre linee guida della libreria, vedere [linee guida per la libreria open source](/dotnet/standard/library-guidance/).

## <a name="determine-which-aspnet-core-versions-to-support"></a>Determinare quali versioni di ASP.NET Core supportare

ASP.NET Core rispetta i criteri di [supporto di .NET Core](https://dotnet.microsoft.com/platform/support/policy/dotnet-core). Per determinare quali versioni di ASP.NET Core supportare in una raccolta, consultare i criteri di supporto. Una libreria deve:

* Eseguire un tentativo di supporto per tutte le versioni di ASP.NET Core classificate come *supporto a lungo termine* (LTS).
* Non è obbligatorio supportare le versioni di ASP.NET Core classificate come *End of Life* (EOL).

Quando vengono rese disponibili le versioni di anteprima di ASP.NET Core, le modifiche di rilievo vengono pubblicate nel repository di GitHub [ASPNET/annunci](https://github.com/aspnet/Announcements/issues) . Il test di compatibilità delle librerie può essere eseguito quando si sviluppano funzionalità del Framework.

## <a name="use-the-aspnet-core-shared-framework"></a>Usare il Framework condiviso ASP.NET Core

Con il rilascio di .NET Core 3,0, molti assembly ASP.NET Core non vengono più pubblicati in NuGet come pacchetti. Gli assembly vengono invece inclusi nel `Microsoft.AspNetCore.App` Framework condiviso, che viene installato con i programmi di installazione di .NET Core SDK e Runtime. Per un elenco di pacchetti che non vengono più pubblicati, vedere [rimuovere i riferimenti ai pacchetti obsoleti](xref:migration/22-to-30#remove-obsolete-package-references).

A partire da .NET Core 3,0, i progetti che usano `Microsoft.NET.Sdk.Web` MSBuild SDK fanno riferimento in modo implicito al Framework condiviso. I progetti che usano l' `Microsoft.NET.Sdk` `Microsoft.NET.Sdk.Razor` SDK o devono fare riferimento a ASP.NET Core per usare ASP.NET Core API nel Framework condiviso.

Per fare riferimento ASP.NET Core, aggiungere il seguente `<FrameworkReference>` elemento al file di progetto:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj?highlight=8)]

Il riferimento ASP.NET Core in questo modo è supportato solo per i progetti destinati a .NET Core 3. x.

## <a name="include-blazor-extensibility"></a>Includi Blazor estendibilità

Blazor supporta i [modelli di hosting](xref:blazor/hosting-models)webassembly (WASM) e server. A meno che non esista un motivo specifico per non farlo, una libreria di [ Razor componenti](xref:blazor/components/index) deve supportare entrambi i modelli di hosting. Una Razor libreria di componenti deve usare [Microsoft. NET. Sdk. Razor SDK](xref:razor-pages/sdk).

### <a name="support-both-hosting-models"></a>Supportare entrambi i modelli di hosting

Per supportare Razor l'utilizzo dei componenti da [Blazor Server](xref:blazor/hosting-models#blazor-server) e nei progetti [ Blazor WASM](xref:blazor/hosting-models#blazor-webassembly) , usare le istruzioni seguenti per l'editor.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Usare il modello di progetto **Razor libreria di classi** . La casella di controllo **pagine e visualizzazioni del supporto** del modello deve essere deselezionata.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Eseguire il comando seguente nel terminale integrato:

```dotnetcli
dotnet new razorclasslib
```

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Usare il modello di progetto **Razor libreria di classi** .

---

Il progetto generato dal modello esegue le operazioni seguenti:

* Destinazioni .NET Standard 2,0.
* Imposta la proprietà `RazorLangVersion` su `3.0`. `3.0` è il valore predefinito per .NET Core 3. x.
* Aggiunge i riferimenti ai pacchetti seguenti:
  * [Microsoft. AspNetCore. Components](https://www.nuget.org/packages/Microsoft.AspNetCore.Components)
  * [Microsoft. AspNetCore. Components. Web](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Web)

Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-components-library.csproj)]

### <a name="support-a-specific-hosting-model"></a>Supporto di un modello di hosting specifico

È molto meno comune supportare un singolo modello di Blazor hosting. Ad esempio, per supportare l' Razor utilizzo dei componenti [Blazor Server](xref:blazor/hosting-models#blazor-server) solo da progetti:

* Destinazione .NET Core 3. x.
* Aggiungere un `<FrameworkReference>` elemento per il Framework condiviso.

Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-components-library.csproj)]

Per ulteriori informazioni sulle librerie contenenti i Razor componenti di, vedere [ASP.NET Core Razor Components Class Libraries](xref:blazor/components/class-libraries).

## <a name="include-mvc-extensibility"></a>Includi estendibilità MVC

Questa sezione descrive le raccomandazioni per le librerie che includono:

* [Razor viste o Razor pagine](#razor-views-or-razor-pages)
* [Helper per i tag](#tag-helpers)
* [Componenti di visualizzazione](#view-components)

Questa sezione non illustra la funzionalità multitargeting per supportare più versioni di MVC. Per informazioni sul supporto di più versioni di ASP.NET Core, vedere [supportare più versioni di ASP.NET Core](#support-multiple-aspnet-core-versions).

### <a name="razor-views-or-razor-pages"></a>Razor viste o Razor pagine

Un progetto che include [ Razor viste](xref:mvc/views/overview) o [ Razor pagine](xref:razor-pages/index) deve utilizzare [Microsoft. NET. Sdk. Razor SDK](xref:razor-pages/sdk).

Se il progetto è destinato a .NET Core 3. x, è necessario:

* `AddRazorSupportForMvc`Proprietà di MSBuild impostata su `true` .
* `<FrameworkReference>`Elemento per il Framework condiviso.

Il modello di progetto **Razor libreria di classi** soddisfa i requisiti precedenti per i progetti destinati a .NET Core 3. x. Usare le istruzioni seguenti per l'editor.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Usare il modello di progetto **Razor libreria di classi** . È necessario selezionare la casella di controllo **pagine e visualizzazioni del supporto** del modello.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Eseguire il comando seguente nel terminale integrato:

```dotnetcli
dotnet new razorclasslib -s
```

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Al momento non è disponibile alcun supporto per i modelli di progetto.

---

Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-views-pages-library.csproj)]

Se il progetto è destinato .NET Standard al contrario, è necessario un riferimento al pacchetto [Microsoft. AspNetCore. Mvc](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc) . Il `Microsoft.AspNetCore.Mvc` pacchetto è stato spostato nel Framework condiviso in ASP.NET Core 3,0 e pertanto non è più pubblicato. Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-views-pages-library.csproj?highlight=8)]

### <a name="tag-helpers"></a>Helper tag

Un progetto che include gli [Helper Tag](xref:mvc/views/tag-helpers/intro) deve usare l' `Microsoft.NET.Sdk` SDK. Se la destinazione è .NET Core 3. x, aggiungere un `<FrameworkReference>` elemento per il Framework condiviso. Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

Se la destinazione .NET Standard (per supportare versioni precedenti a ASP.NET Core 3. x), aggiungere un riferimento al pacchetto a [Microsoft. AspNetCore. Mvc Razor .](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor). Il `Microsoft.AspNetCore.Mvc.Razor` pacchetto è stato spostato nel Framework condiviso e pertanto non è più pubblicato. Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-tag-helpers-library.csproj)]

### <a name="view-components"></a>Componenti di visualizzazione

Un progetto che include [componenti di visualizzazione](xref:mvc/views/view-components) deve usare l' `Microsoft.NET.Sdk` SDK. Se la destinazione è .NET Core 3. x, aggiungere un `<FrameworkReference>` elemento per il Framework condiviso. Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

Se la destinazione .NET Standard (per supportare versioni precedenti a ASP.NET Core 3. x), aggiungere un riferimento al pacchetto a [Microsoft. AspNetCore. Mvc. ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures). Il `Microsoft.AspNetCore.Mvc.ViewFeatures` pacchetto è stato spostato nel Framework condiviso e pertanto non è più pubblicato. Ad esempio:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-view-components-library.csproj)]

## <a name="support-multiple-aspnet-core-versions"></a>Supporto di più versioni di ASP.NET Core

Il multitargeting è necessario per creare una libreria che supporti più varianti di ASP.NET Core. Si consideri uno scenario in cui una libreria helper tag deve supportare le varianti di ASP.NET Core seguenti:

* ASP.NET Core 2,1 targeting .NET Framework 4.6.1
* ASP.NET Core 2. x destinato a .NET Core 2. x
* ASP.NET Core 3. x destinato a .NET Core 3. x

Il file di progetto seguente supporta queste varianti tramite la `TargetFrameworks` proprietà:

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

Con il file di progetto precedente:

* Il `Markdig` pacchetto viene aggiunto a tutti i consumer.
* Riferimento a [Microsoft. AspNetCore. Mvc. Razor ](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor) viene aggiunto per i consumer destinati .NET Framework 4.6.1 o versione successiva o .NET Core 2. x. La versione 2.1.0 del pacchetto funziona con ASP.NET Core 2,2, a causa della compatibilità con le versioni precedenti.
* Al Framework condiviso viene fatto riferimento per i consumer destinati a .NET Core 3. x. Il `Microsoft.AspNetCore.Mvc.Razor` pacchetto è incluso nel Framework condiviso.

In alternativa, è possibile indirizzare .NET Standard 2,0 anziché indirizzare sia a .NET Core 2,1 sia a .NET Framework 4.6.1:

[!code-xml[](target-aspnetcore/samples/multi-tfm/alternative-tag-helpers-library.csproj?highlight=4)]

Con il file di progetto precedente, sono presenti le avvertenze seguenti:

* Poiché la libreria contiene solo Helper tag, è più semplice usare come destinazione le piattaforme specifiche in cui viene eseguito ASP.NET Core: .NET Core e .NET Framework. Gli helper tag non possono essere usati da altri Framework di destinazione compatibili con .NET Standard 2,0, ad esempio Unity, UWP e Novell.
* L'uso di .NET Standard 2.0 da .NET Framework presenta alcuni problemi che sono stati risolti in .NET Framework 4.7.2. È possibile migliorare l'esperienza per gli utenti che usano .NET Framework 4.6.1 tramite 4.7.1 impostando come destinazione .NET Framework 4.6.1.

Se la libreria deve chiamare le API specifiche della piattaforma, destinare implementazioni .NET specifiche anziché .NET Standard. Per ulteriori informazioni, vedere [multitargeting](/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting).

## <a name="use-an-api-that-hasnt-changed"></a>Usare un'API che non è stata modificata

Si immagini uno scenario in cui si sta aggiornando una libreria middleware da .NET Core 2,2 a 3,0. Le API middleware ASP.NET Core utilizzate nella libreria non sono state modificate tra ASP.NET Core 2,2 e 3,0. Per continuare a supportare la libreria middleware in .NET Core 3,0, seguire questa procedura:

* Seguire le [indicazioni della libreria standard](/dotnet/standard/library-guidance/).
* Aggiungere un riferimento al pacchetto per ogni pacchetto NuGet dell'API se l'assembly corrispondente non esiste nel Framework condiviso.

## <a name="use-an-api-that-changed"></a>Usare un'API modificata

Si immagini uno scenario in cui si sta aggiornando una libreria da .NET Core 2,2 a .NET Core 3,0. Un'API ASP.NET Core utilizzata nella libreria presenta una [modifica sostanziale](/dotnet/core/compatibility/breaking-changes) in ASP.NET Core 3,0. Valutare se la libreria può essere riscritta in modo da non usare l'API interruppe in tutte le versioni.

Se è possibile riscrivere la libreria, effettuare questa operazione e continuare a usare un Framework di destinazione precedente (ad esempio, .NET Standard 2,0 o .NET Framework 4.6.1) con i riferimenti al pacchetto.

Se non è possibile riscrivere la libreria, seguire questa procedura:

* Aggiungere una destinazione per .NET Core 3,0.
* Aggiungere un `<FrameworkReference>` elemento per il Framework condiviso.
* Usare la [direttiva per il preprocessore #if](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if) con il simbolo del Framework di destinazione appropriato per compilare il codice in modo condizionale.

Le letture e le scritture sincrone sui flussi di richiesta e risposta HTTP, ad esempio, sono disabilitate per impostazione predefinita a partire da ASP.NET Core 3,0. ASP.NET Core 2,2 supporta il comportamento sincrono per impostazione predefinita. Si consideri una libreria middleware in cui le operazioni di lettura e scrittura sincrone devono essere abilitate quando si verifica l'I/O. La libreria deve racchiudere il codice per abilitare le funzionalità sincrone nella direttiva per il preprocessore appropriata. Ad esempio:

[!code-csharp[](target-aspnetcore/samples/middleware.cs?highlight=9-24)]

## <a name="use-an-api-introduced-in-30"></a>Usare un'API introdotta in 3,0

Si supponga di voler usare un'API ASP.NET Core introdotta in ASP.NET Core 3,0. Prendere in considerazione le domande seguenti:

1. La libreria richiede funzionalmente la nuova API?
1. La libreria può implementare questa funzionalità in modo diverso?

Se la libreria richiede a livello funzionale l'API e non è possibile implementarla in modo inattivo:

* Solo .NET Core 3. x di destinazione.
* Aggiungere un `<FrameworkReference>` elemento per il Framework condiviso.

Se la libreria può implementare la funzionalità in modo diverso:

* Aggiungere .NET Core 3. x come Framework di destinazione.
* Aggiungere un `<FrameworkReference>` elemento per il Framework condiviso.
* Usare la [direttiva per il preprocessore #if](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if) con il simbolo del Framework di destinazione appropriato per compilare il codice in modo condizionale.

Ad esempio, l'helper tag seguente usa l' <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> interfaccia introdotta in ASP.NET Core 3,0. I consumer destinati a .NET Core 3,0 eseguono il percorso del codice definito dal `NETCOREAPP3_0` simbolo del Framework di destinazione. Il tipo di parametro del costruttore dell'helper Tag passa a <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment> per i consumer di .NET Core 2,1 e .NET Framework 4.6.1. Questa modifica era necessaria perché ASP.NET Core 3,0 è stata contrassegnata `IHostingEnvironment` come obsoleta e consigliata `IWebHostEnvironment` come sostituzione.

```csharp
[HtmlTargetElement("script", Attributes = "asp-inline")]
public class ScriptInliningTagHelper : TagHelper
{
    private readonly IFileProvider _wwwroot;

#if NETCOREAPP3_0
    public ScriptInliningTagHelper(IWebHostEnvironment env)
#else
    public ScriptInliningTagHelper(IHostingEnvironment env)
#endif
    {
        _wwwroot = env.WebRootFileProvider;
    }

    // code omitted for brevity
}
```

Il file di progetto multitargeting seguente supporta questo scenario di helper Tag:

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

## <a name="use-an-api-removed-from-the-shared-framework"></a>Usare un'API rimossa dal Framework condiviso

Per usare un assembly ASP.NET Core rimosso dal Framework condiviso, aggiungere il riferimento al pacchetto appropriato. Per un elenco di pacchetti rimossi dal Framework condiviso in ASP.NET Core 3,0, vedere [rimuovere i riferimenti ai pacchetti obsoleti](xref:migration/22-to-30#remove-obsolete-package-references).

Ad esempio, per aggiungere il client dell'API Web:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNet.WebApi.Client" Version="5.2.7" />
  </ItemGroup>

</Project>
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:razor-pages/ui-class>
* <xref:blazor/components/class-libraries>
* [Supporto per le implementazioni di .NET](/dotnet/standard/net-standard#net-implementation-support)
* [Criteri di supporto .NET](https://dotnet.microsoft.com/platform/support/policy)
