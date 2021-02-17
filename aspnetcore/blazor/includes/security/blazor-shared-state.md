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
ms.openlocfilehash: 5ff4e4368d9e6d7c8525ae4ef0625d176a256a85
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551997"
---
<span data-ttu-id="be3f7-101">Blazor le app Server risiedono nella memoria del server.</span><span class="sxs-lookup"><span data-stu-id="be3f7-101">Blazor server apps live in server memory.</span></span> <span data-ttu-id="be3f7-102">Ciò significa che sono presenti più app ospitate all'interno dello stesso processo.</span><span class="sxs-lookup"><span data-stu-id="be3f7-102">That means that there are multiple apps hosted within the same process.</span></span> <span data-ttu-id="be3f7-103">Per ogni sessione dell'app, Blazor avvia un circuito con il proprio ambito del contenitore di.</span><span class="sxs-lookup"><span data-stu-id="be3f7-103">For each app session, Blazor starts a circuit with its own DI container scope.</span></span> <span data-ttu-id="be3f7-104">Ciò significa che i servizi con ambito sono univoci per ogni Blazor sessione.</span><span class="sxs-lookup"><span data-stu-id="be3f7-104">That means that scoped services are unique per Blazor session.</span></span>

> [!WARNING]
> <span data-ttu-id="be3f7-105">Non sono consigliate le app nello stesso stato di condivisione server usando i servizi singleton, a meno che non venga presa una particolare attenzione, in quanto questo può introdurre vulnerabilità di sicurezza, ad esempio la perdita dello stato utente tra i circuiti.</span><span class="sxs-lookup"><span data-stu-id="be3f7-105">We don't recommend apps on the same server share state using singleton services unless extreme care is taken, as this can introduce security vulnerabilities, such as leaking user state across circuits.</span></span>

<span data-ttu-id="be3f7-106">È possibile usare i servizi singleton con stato nelle Blazor app se sono appositamente progettati.</span><span class="sxs-lookup"><span data-stu-id="be3f7-106">You can use stateful singleton services in Blazor apps if they are specifically designed for it.</span></span> <span data-ttu-id="be3f7-107">Ad esempio, è possibile usare una cache in memoria come singleton perché richiede una chiave per accedere a una determinata voce, supponendo che gli utenti non abbiano il controllo delle chiavi di cache usate.</span><span class="sxs-lookup"><span data-stu-id="be3f7-107">For example, it's ok to use a memory cache as a singleton because it requires a key to access a given entry, assuming users don't have control of what cache keys are used.</span></span>

<span data-ttu-id="be3f7-108">**Inoltre, per motivi di sicurezza, non è necessario usare <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor> all'interno delle Blazor app.**</span><span class="sxs-lookup"><span data-stu-id="be3f7-108">**Additionally, again for security reasons, you must not use <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor> within Blazor apps.**</span></span> <span data-ttu-id="be3f7-109">Blazor le app vengono eseguite all'esterno del contesto della pipeline ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="be3f7-109">Blazor apps run outside of the context of the ASP.NET Core pipeline.</span></span> <span data-ttu-id="be3f7-110">Non è garantito che sia <xref:Microsoft.AspNetCore.Http.HttpContext> disponibile all'interno di <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor> , né che contenga il contesto che ha avviato l' Blazor app.</span><span class="sxs-lookup"><span data-stu-id="be3f7-110">The <xref:Microsoft.AspNetCore.Http.HttpContext> isn't guaranteed to be available within the <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor>, nor is it guaranteed to be holding the context that started the Blazor app.</span></span>

<span data-ttu-id="be3f7-111">Il modo consigliato per passare lo stato della richiesta all'app consiste nell'usare i Blazor parametri per il componente radice nel rendering iniziale dell'app:</span><span class="sxs-lookup"><span data-stu-id="be3f7-111">The recommended way to pass request state to the Blazor app is through parameters to the root component in the initial rendering of the app:</span></span>

* <span data-ttu-id="be3f7-112">Definire una classe con tutti i dati che si desidera passare all' Blazor app.</span><span class="sxs-lookup"><span data-stu-id="be3f7-112">Define a class with all the data you want to pass to the Blazor app.</span></span>
* <span data-ttu-id="be3f7-113">Inserire i dati dalla Razor pagina utilizzando la <xref:Microsoft.AspNetCore.Http.HttpContext> disponibile in quel momento.</span><span class="sxs-lookup"><span data-stu-id="be3f7-113">Populate that data from the Razor page using the <xref:Microsoft.AspNetCore.Http.HttpContext> available at that time.</span></span>
* <span data-ttu-id="be3f7-114">Passare i dati all' Blazor app come parametro al componente radice (app).</span><span class="sxs-lookup"><span data-stu-id="be3f7-114">Pass the data to the Blazor app as a parameter to the root component (App).</span></span>
* <span data-ttu-id="be3f7-115">Definire un parametro nel componente radice per conservare i dati passati all'app.</span><span class="sxs-lookup"><span data-stu-id="be3f7-115">Define a parameter in the root component to hold the data being passed to the app.</span></span>
* <span data-ttu-id="be3f7-116">Usare i dati specifici dell'utente all'interno dell'app; in alternativa, copiare i dati in un servizio con ambito in in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> modo che possa essere usato nell'app.</span><span class="sxs-lookup"><span data-stu-id="be3f7-116">Use the user-specific data within the app; or alternatively, copy that data into a scoped service within <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> so that it can be used across the app.</span></span>

<span data-ttu-id="be3f7-117">Per ulteriori informazioni e un codice di esempio, vedere <xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app>.</span><span class="sxs-lookup"><span data-stu-id="be3f7-117">For more information and example code, see <xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app>.</span></span>
