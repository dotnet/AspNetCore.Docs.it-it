---
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
ms.openlocfilehash: ca743cdb39423d833902c330f6f0682b600c9833
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552156"
---
::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

> [!WARNING]
> Un parametro **catch-all** può corrispondere erroneamente alle route a causa di un [bug](https://github.com/dotnet/aspnetcore/issues/18677) nel routing. Le app interessate da questo bug hanno le seguenti caratteristiche:
>
> * Una route catch-all, ad esempio `{**slug}"`
> * La route catch-all non riesce a trovare le richieste corrispondenti.
> * La rimozione di altre route rende l'avvio del funzionamento della route catch-all.
>
> Vedere i bug di GitHub [18677](https://github.com/dotnet/aspnetcore/issues/18677) e [16579](https://github.com/dotnet/aspnetcore/issues/16579) per i casi di esempio che hanno raggiunto questo bug.
>
> Una correzione del consenso esplicito per questo bug è inclusa in [.NET Core 3.1.301 SDK e versioni successive](https://dotnet.microsoft.com/download/dotnet-core/3.1). Il codice seguente imposta un'opzione interna che corregge questo bug:
>
>```csharp
>public static void Main(string[] args)
>{
>    AppContext.SetSwitch("Microsoft.AspNetCore.Routing.UseCorrectCatchAllBehavior", 
>                          true);
>    CreateHostBuilder(args).Build().Run();
>}
>// Remaining code removed for brevity.
>```

::: moniker-end