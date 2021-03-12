---
title: Autorizzazione semplice in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare l'attributo di autorizzazione per limitare l'accesso a controller e azioni ASP.NET Core.
ms.author: riande
ms.date: 10/14/2016
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
uid: security/authorization/simple
ms.openlocfilehash: eaf36822edc47f7238c356e167b05ac44f93175d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587567"
---
# <a name="simple-authorization-in-aspnet-core"></a><span data-ttu-id="c5059-103">Autorizzazione semplice in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="c5059-103">Simple authorization in ASP.NET Core</span></span>

<a name="security-authorization-simple"></a>

<span data-ttu-id="c5059-104">L'autorizzazione in ASP.NET Core viene controllata con <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> e i vari parametri.</span><span class="sxs-lookup"><span data-stu-id="c5059-104">Authorization in ASP.NET Core is controlled with <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> and its various parameters.</span></span> <span data-ttu-id="c5059-105">Nella sua forma più semplice, applicando l' `[Authorize]` attributo a un controller, a un'azione o a una Razor pagina, è possibile limitare l'accesso a tale componente a qualsiasi utente autenticato.</span><span class="sxs-lookup"><span data-stu-id="c5059-105">In its simplest form, applying the `[Authorize]` attribute to a controller, action, or Razor Page, limits access to that component to any authenticated user.</span></span>

<span data-ttu-id="c5059-106">Il codice seguente, ad esempio, limita l'accesso a `AccountController` a qualsiasi utente autenticato.</span><span class="sxs-lookup"><span data-stu-id="c5059-106">For example, the following code limits access to the `AccountController` to any authenticated user.</span></span>

```csharp
[Authorize]
public class AccountController : Controller
{
    public ActionResult Login()
    {
    }

    public ActionResult Logout()
    {
    }
}
```

<span data-ttu-id="c5059-107">Se si desidera applicare l'autorizzazione a un'azione anziché al controller, applicare l' `AuthorizeAttribute` attributo all'azione stessa:</span><span class="sxs-lookup"><span data-stu-id="c5059-107">If you want to apply authorization to an action rather than the controller, apply the `AuthorizeAttribute` attribute to the action itself:</span></span>

```csharp
public class AccountController : Controller
{
   public ActionResult Login()
   {
   }

   [Authorize]
   public ActionResult Logout()
   {
   }
}
```

<span data-ttu-id="c5059-108">A questo punto, solo gli utenti autenticati possono accedere alla `Logout` funzione.</span><span class="sxs-lookup"><span data-stu-id="c5059-108">Now only authenticated users can access the `Logout` function.</span></span>

<span data-ttu-id="c5059-109">È anche possibile usare l' `AllowAnonymous` attributo per consentire l'accesso da parte di utenti non autenticati a singole azioni.</span><span class="sxs-lookup"><span data-stu-id="c5059-109">You can also use the `AllowAnonymous` attribute to allow access by non-authenticated users to individual actions.</span></span> <span data-ttu-id="c5059-110">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="c5059-110">For example:</span></span>

```csharp
[Authorize]
public class AccountController : Controller
{
    [AllowAnonymous]
    public ActionResult Login()
    {
    }

    public ActionResult Logout()
    {
    }
}
```

<span data-ttu-id="c5059-111">Ciò consente solo agli utenti autenticati di `AccountController` , ad eccezione dell' `Login` azione, accessibile da parte di tutti, indipendentemente dallo stato autenticato o non autenticato/anonimo.</span><span class="sxs-lookup"><span data-stu-id="c5059-111">This would allow only authenticated users to the `AccountController`, except for the `Login` action, which is accessible by everyone, regardless of their authenticated or unauthenticated / anonymous status.</span></span>

> [!WARNING]
> <span data-ttu-id="c5059-112">`[AllowAnonymous]` Ignora tutte le istruzioni di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="c5059-112">`[AllowAnonymous]` bypasses all authorization statements.</span></span> <span data-ttu-id="c5059-113">Se si combinano `[AllowAnonymous]` e qualsiasi `[Authorize]` attributo, gli `[Authorize]` attributi vengono ignorati.</span><span class="sxs-lookup"><span data-stu-id="c5059-113">If you combine `[AllowAnonymous]` and any `[Authorize]` attribute, the `[Authorize]` attributes are ignored.</span></span> <span data-ttu-id="c5059-114">Se ad esempio si applica `[AllowAnonymous]` a livello di controller, tutti `[Authorize]` gli attributi sullo stesso controller (o su qualsiasi azione al suo interno) vengono ignorati.</span><span class="sxs-lookup"><span data-stu-id="c5059-114">For example if you apply `[AllowAnonymous]` at the controller level, any `[Authorize]` attributes on the same controller (or on any action within it) is ignored.</span></span>

[!INCLUDE[](~/includes/requireAuth.md)]

<a name="aarp"></a>

## <a name="authorize-attribute-and-razor-pages"></a><span data-ttu-id="c5059-115">Autorizzare gli attributi e le Razor pagine</span><span class="sxs-lookup"><span data-stu-id="c5059-115">Authorize attribute and Razor Pages</span></span>

<span data-ttu-id="c5059-116"><xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ***Non*** può essere applicato ai Razor gestori di pagine.</span><span class="sxs-lookup"><span data-stu-id="c5059-116">The <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> can ***not*** be applied to Razor Page handlers.</span></span> <span data-ttu-id="c5059-117">Ad esempio, `[Authorize]` non può essere applicato a `OnGet` , `OnPost` o a qualsiasi altro gestore di pagina.</span><span class="sxs-lookup"><span data-stu-id="c5059-117">For example, `[Authorize]` can't be applied to `OnGet`, `OnPost`, or any other page handler.</span></span> <span data-ttu-id="c5059-118">Considerare l'uso di un controller ASP.NET Core MVC per le pagine con requisiti di autorizzazione diversi per gestori diversi.</span><span class="sxs-lookup"><span data-stu-id="c5059-118">Consider using an ASP.NET Core MVC controller for pages with different authorization requirements for different handlers.</span></span>

<span data-ttu-id="c5059-119">Per applicare l'autorizzazione ai metodi del gestore di pagina, è possibile usare i due approcci seguenti Razor :</span><span class="sxs-lookup"><span data-stu-id="c5059-119">The following two approaches can be used to apply authorization to Razor Page handler methods:</span></span>

* <span data-ttu-id="c5059-120">Usare pagine separate per i gestori di pagine che richiedono un'autorizzazione diversa.</span><span class="sxs-lookup"><span data-stu-id="c5059-120">Use separate pages for page handlers requiring different authorization.</span></span> <span data-ttu-id="c5059-121">Spostare il contenuto condiviso in una o più [visualizzazioni parziali](xref:mvc/views/partial).</span><span class="sxs-lookup"><span data-stu-id="c5059-121">Move shared content into one or more [partial views](xref:mvc/views/partial).</span></span> <span data-ttu-id="c5059-122">Quando possibile, questo è l'approccio consigliato.</span><span class="sxs-lookup"><span data-stu-id="c5059-122">When possible, this is the recommended approach.</span></span>
* <span data-ttu-id="c5059-123">Per il contenuto che deve condividere una pagina comune, scrivere un filtro che esegua l'autorizzazione come parte di [IAsyncPageFilter. OnPageHandlerSelectionAsync](xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter.OnPageHandlerSelectionAsync%2A).</span><span class="sxs-lookup"><span data-stu-id="c5059-123">For content that must share a common page, write a filter that performs authorization as part of [IAsyncPageFilter.OnPageHandlerSelectionAsync](xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter.OnPageHandlerSelectionAsync%2A).</span></span> <span data-ttu-id="c5059-124">Il progetto GitHub [PageHandlerAuth](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth) illustra questo approccio:</span><span class="sxs-lookup"><span data-stu-id="c5059-124">The [PageHandlerAuth](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth) GitHub project demonstrates this approach:</span></span>
  * <span data-ttu-id="c5059-125">[AuthorizeIndexPageHandlerFilter](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs) implementa il filtro di autorizzazione:[!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs?name=snippet)]</span><span class="sxs-lookup"><span data-stu-id="c5059-125">The [AuthorizeIndexPageHandlerFilter](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs) implements the authorization filter: [!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs?name=snippet)]</span></span>

  * <span data-ttu-id="c5059-126">L'attributo [[AuthorizePageHandler]](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs#L16) viene applicato al `OnGet` gestore di pagina: [!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs?name=snippet)]</span><span class="sxs-lookup"><span data-stu-id="c5059-126">The [[AuthorizePageHandler]](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs#L16) attribute is applied to the `OnGet` page handler: [!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs?name=snippet)]</span></span>

> [!WARNING]
> <span data-ttu-id="c5059-127">L' [](https://github.com/pranavkm/PageHandlerAuth) approccio di esempio PageHandlerAuth ***non***:</span><span class="sxs-lookup"><span data-stu-id="c5059-127">The [PageHandlerAuth](https://github.com/pranavkm/PageHandlerAuth) sample approach does ***not***:</span></span>
> * <span data-ttu-id="c5059-128">Comporre con attributi di autorizzazione applicati alla pagina, al modello di pagina o a livello globale.</span><span class="sxs-lookup"><span data-stu-id="c5059-128">Compose with authorization attributes applied to the page, page model, or globally.</span></span> <span data-ttu-id="c5059-129">La composizione di attributi di autorizzazione comporta l'autenticazione e l'autorizzazione eseguite più volte quando si dispone `AuthorizeAttribute` di un'altra `AuthorizeFilter` istanza o di istanze applicate anche alla pagina.</span><span class="sxs-lookup"><span data-stu-id="c5059-129">Composing authorization attributes results in authentication and authorization executing multiple times when you have one more `AuthorizeAttribute` or `AuthorizeFilter` instances also applied to the page.</span></span>
> * <span data-ttu-id="c5059-130">Usare insieme al resto del sistema di autenticazione e autorizzazione ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="c5059-130">Work in conjunction with the rest of ASP.NET Core authentication and authorization system.</span></span> <span data-ttu-id="c5059-131">È necessario verificare che l'utilizzo di questo approccio funzioni correttamente per l'applicazione.</span><span class="sxs-lookup"><span data-stu-id="c5059-131">You must verify using this approach works correctly for your application.</span></span>

<span data-ttu-id="c5059-132">Non sono previsti piani per il supporto `AuthorizeAttribute` dei Razor gestori di pagine.</span><span class="sxs-lookup"><span data-stu-id="c5059-132">There are no plans to support the `AuthorizeAttribute` on Razor Page handlers.</span></span> 
