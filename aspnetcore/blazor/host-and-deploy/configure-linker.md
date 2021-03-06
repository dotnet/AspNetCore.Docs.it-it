---
title: Configurare il linker per ASP.NET Core Blazor
author: guardrex
description: Informazioni su come controllare il linker del linguaggio intermedio (IL) durante la compilazione di un' Blazor app.
monikerRange: '>= aspnetcore-3.1 < aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
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
uid: blazor/host-and-deploy/configure-linker
ms.openlocfilehash: af3c059e7192d6b0d2b0a902b6e3a6121fdf6709
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395032"
---
# <a name="configure-the-linker-for-aspnet-core-blazor"></a>Configurare il linker per ASP.NET Core Blazor

Blazor WebAssembly esegue il collegamento del [linguaggio intermedio (il)](/dotnet/standard/managed-code#intermediate-language--execution) durante una compilazione per eliminare il il non necessario dagli assembly di output dell'app. Il linker è disabilitato durante la compilazione nella configurazione di debug. Per abilitare il linker, le app devono essere compilate in configurazione versione. Quando si distribuiscono le app, è consigliabile creare in versione Blazor WebAssembly . 

Il collegamento di un'app ottimizza le dimensioni, ma può avere effetti negativi. Le app che usano la reflection o le funzionalità dinamiche correlate potrebbero interrompersi quando vengono tagliate perché il linker non è a conoscenza di questo comportamento dinamico e non può determinare in generale quali tipi sono necessari per la reflection in fase di esecuzione. Per tagliare tali app, il linker deve essere informato sui tipi necessari per la reflection nel codice e nei pacchetti o Framework da cui dipende l'app.

Per assicurarsi che l'app tagliata funzioni correttamente una volta distribuita, è importante testare le build di rilascio dell'app spesso durante lo sviluppo.

Il collegamento per Blazor le app può essere configurato usando le funzionalità di MSBuild seguenti:

* Configurare il collegamento a livello globale con una [proprietà di MSBuild](#control-linking-with-an-msbuild-property).
* Controllo del collegamento per ogni assembly con un [file di configurazione](#control-linking-with-a-configuration-file).

## <a name="control-linking-with-an-msbuild-property"></a>Controllo del collegamento con una proprietà MSBuild

Il collegamento viene abilitato quando si compila un'app nella `Release` configurazione. Per modificare questa configurazione, configurare la `BlazorWebAssemblyEnableLinking` Proprietà MSBuild nel file di progetto:

```xml
<PropertyGroup>
  <BlazorWebAssemblyEnableLinking>false</BlazorWebAssemblyEnableLinking>
</PropertyGroup>
```

## <a name="control-linking-with-a-configuration-file"></a>Controllare il collegamento con un file di configurazione

Controllare il collegamento per ogni singolo assembly usando un file di configurazione XML e specificando il file come un elemento MSBuild nel file di progetto:

```xml
<ItemGroup>
  <BlazorLinkerDescriptor Include="LinkerConfig.xml" />
</ItemGroup>
```

`LinkerConfig.xml`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
  This file specifies which parts of the BCL or Blazor packages must not be
  stripped by the IL Linker even if they aren't referenced by user code.
-->
<linker>
  <assembly fullname="mscorlib">
    <!--
      Preserve the methods in WasmRuntime because its methods are called by 
      JavaScript client-side code to implement timers.
      Fixes: https://github.com/dotnet/blazor/issues/239
    -->
    <type fullname="System.Threading.WasmRuntime" />
  </assembly>
  <assembly fullname="System.Core">
    <!--
      System.Linq.Expressions* is required by Json.NET and any 
      expression.Compile caller. The assembly isn't stripped.
    -->
    <type fullname="System.Linq.Expressions*" />
  </assembly>
  <!--
    In this example, the app's entry point assembly is listed. The assembly
    isn't stripped by the IL Linker.
  -->
  <assembly fullname="MyCoolBlazorApp" />
</linker>
```

Per altre informazioni ed esempi, vedere [formati di dati (repository GitHub mono/linker)](https://github.com/mono/linker/blob/main/docs/data-formats.md).

## <a name="add-an-xml-linker-configuration-file-to-a-library"></a>Aggiungere un file di configurazione del linker XML a una raccolta

Per configurare il linker per una libreria specifica, aggiungere un file di configurazione del linker XML nella libreria come risorsa incorporata. La risorsa incorporata deve avere lo stesso nome dell'assembly.

Nell'esempio seguente il `LinkerConfig.xml` file viene specificato come una risorsa incorporata con lo stesso nome dell'assembly della libreria:

```xml
<ItemGroup>
  <EmbeddedResource Include="LinkerConfig.xml">
    <LogicalName>$(MSBuildProjectName).xml</LogicalName>
  </EmbeddedResource>
</ItemGroup>
```

### <a name="configure-the-linker-for-internationalization"></a>Configurare il linker per l'internazionalizzazione

Per impostazione predefinita, Blazor la configurazione del linker per le Blazor WebAssembly app rimuove le informazioni di internazionalizzazione ad eccezione delle impostazioni locali richieste in modo esplicito. La rimozione di questi assembly riduce al minimo le dimensioni dell'app.

Per controllare quali assembly I18N vengono conservati, impostare la `<BlazorWebAssemblyI18NAssemblies>` Proprietà MSBuild nel file di progetto:

```xml
<PropertyGroup>
  <BlazorWebAssemblyI18NAssemblies>{all|none|REGION1,REGION2,...}</BlazorWebAssemblyI18NAssemblies>
</PropertyGroup>
```

| Valore Region     | Assembly dell'area mono    |
| ---------------- | ----------------------- |
| `all`            | Tutti gli assembly inclusi |
| `cjk`            | `I18N.CJK.dll`          |
| `mideast`        | `I18N.MidEast.dll`      |
| `none` (impostazione predefinita) | nessuno                    |
| `other`          | `I18N.Other.dll`        |
| `rare`           | `I18N.Rare.dll`         |
| `west`           | `I18N.West.dll`         |

Usare una virgola per separare più valori (ad esempio, `mideast,west` ).

Per altre informazioni, vedere [i18n: Pnetlib internationalation Framework Library (repository GitHub mono/mono)](https://github.com/mono/mono/tree/master/mcs/class/I18N).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/webassembly-performance-best-practices#intermediate-language-il-linking>
