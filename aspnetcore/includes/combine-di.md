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

<span data-ttu-id="7e2bf-101">Si consideri il `ConfigureServices` metodo seguente, che registra i servizi e configura le opzioni:</span><span class="sxs-lookup"><span data-stu-id="7e2bf-101">Consider the following `ConfigureServices` method, which registers services and configures options:</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup2.cs?name=snippet)]

<span data-ttu-id="7e2bf-102">I gruppi di registrazioni correlati possono essere spostati in un metodo di estensione per registrare i servizi.</span><span class="sxs-lookup"><span data-stu-id="7e2bf-102">Related groups of registrations can be moved to an extension method to register services.</span></span> <span data-ttu-id="7e2bf-103">I servizi di configurazione, ad esempio, vengono aggiunti alla classe seguente:</span><span class="sxs-lookup"><span data-stu-id="7e2bf-103">For example, the configuration services are added to the following class:</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Options/MyConfigServiceCollectionExtensions.cs)]

<span data-ttu-id="7e2bf-104">I servizi rimanenti sono registrati in una classe simile.</span><span class="sxs-lookup"><span data-stu-id="7e2bf-104">The remaining services are registered in a similar class.</span></span> <span data-ttu-id="7e2bf-105">Il `ConfigureServices` metodo seguente usa i nuovi metodi di estensione per registrare i servizi:</span><span class="sxs-lookup"><span data-stu-id="7e2bf-105">The following `ConfigureServices` method uses the new extension methods to register the services:</span></span>

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Startup4.cs?name=snippet)]

<span data-ttu-id="7e2bf-106">**_Nota:_** Ogni `services.Add{GROUP_NAME}` metodo di estensione aggiunge e potenzialmente configura i servizi.</span><span class="sxs-lookup"><span data-stu-id="7e2bf-106">**_Note:_** Each `services.Add{GROUP_NAME}` extension method adds and potentially configures services.</span></span> <span data-ttu-id="7e2bf-107">Ad esempio, <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllersWithViews%2A> aggiunge i controller MVC dei servizi con le visualizzazioni richieste e <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages%2A> aggiunge le Razor pagine dei servizi richiesti.</span><span class="sxs-lookup"><span data-stu-id="7e2bf-107">For example, <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllersWithViews%2A> adds the services MVC controllers with views require, and <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages%2A> adds the services Razor Pages requires.</span></span> <span data-ttu-id="7e2bf-108">È consigliabile che le app seguano questa convenzione di denominazione.</span><span class="sxs-lookup"><span data-stu-id="7e2bf-108">We recommended that apps follow this naming convention.</span></span> <span data-ttu-id="7e2bf-109">Inserire i metodi di estensione nello spazio dei nomi <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> per incapsulare i gruppi di registrazioni di servizio.</span><span class="sxs-lookup"><span data-stu-id="7e2bf-109">Place extension methods in the <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> namespace to encapsulate groups of service registrations.</span></span>
