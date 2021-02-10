---
title: ASP.NET Core Blazor file statici
author: guardrex
description: Informazioni su come configurare e gestire i file statici per le Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/27/2021
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
uid: blazor/fundamentals/static-files
ms.openlocfilehash: 709250251113014027fe86c6a373e58a9c2a5009
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100110033"
---
# <a name="aspnet-core-blazor-static-files"></a>ASP.NET Core Blazor file statici

*Questo articolo si applica a Blazor Server .*

Per creare altri mapping di file con <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> o configurare altri <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> , usare **uno** degli approcci seguenti. Negli esempi seguenti, il `{EXTENSION}` segnaposto è l'estensione di file e il `{CONTENT TYPE}` segnaposto è il tipo di contenuto.

* Configurare le opzioni tramite l' [inserimento di dipendenze](xref:blazor/fundamentals/dependency-injection) in `Startup.ConfigureServices` ( `Startup.cs` ) utilizzando <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> :

  ```csharp
  using Microsoft.AspNetCore.StaticFiles;

  ...

  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  services.Configure<StaticFileOptions>(options =>
  {
      options.ContentTypeProvider = provider;
  });
  ```

  Poiché questo approccio configura lo stesso provider di file usato per gestire `blazor.server.js` , assicurarsi che la configurazione personalizzata non interferisca con il servizio `blazor.server.js` . Ad esempio, non rimuovere il mapping per i file JavaScript configurando il provider con `provider.Mappings.Remove(".js")` .

* Usare due chiamate a <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> in `Startup.Configure` ( `Startup.cs` ):
  * Configurare il provider di file personalizzato nella prima chiamata con <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> .
  * Il secondo middleware serve `blazor.server.js` , che usa la configurazione predefinita dei file statici fornita dal Blazor Framework.

  ```csharp
  using Microsoft.AspNetCore.StaticFiles;

  ...

  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  app.UseStaticFiles(new StaticFileOptions { ContentTypeProvider = provider });
  app.UseStaticFiles();
  ```

* È possibile evitare di interferire con `_framework/blazor.server.js` il servizio utilizzando <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> per eseguire un middleware di file statico personalizzato:

  ```csharp
  app.MapWhen(ctx => !ctx.Request.Path
      .StartsWithSegments("_framework/blazor.server.js", 
          subApp => subApp.UseStaticFiles(new StaticFileOptions(){ ... })));
  ```
