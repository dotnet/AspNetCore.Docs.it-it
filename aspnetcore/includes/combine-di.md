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
ms.openlocfilehash: 08d212b48c3f97531bea34b11d5b8c9a9069ae34
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536297"
---
<a name="csc"></a>

Si consideri il `ConfigureServices` metodo seguente, che registra i servizi e configura le opzioni:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup2.cs?name=snippet)]

I gruppi di registrazioni correlati possono essere spostati in un metodo di estensione per registrare i servizi. I servizi di configurazione, ad esempio, vengono aggiunti alla classe seguente:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Options/MyConfigServiceCollectionExtensions.cs)]

I servizi rimanenti sono registrati in una classe simile. Il `ConfigureServices` metodo seguente usa i nuovi metodi di estensione per registrare i servizi:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup4.cs?name=snippet)]

**_Nota:_** Ogni `services.Add{GROUP_NAME}` metodo di estensione aggiunge e potenzialmente configura i servizi. Ad esempio, <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllersWithViews%2A> aggiunge i controller MVC dei servizi con le visualizzazioni richieste e <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages%2A> aggiunge le Razor pagine dei servizi richiesti. È consigliabile che le app seguano questa convenzione di denominazione. Inserire i metodi di estensione nello spazio dei nomi <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> per incapsulare i gruppi di registrazioni di servizio.
