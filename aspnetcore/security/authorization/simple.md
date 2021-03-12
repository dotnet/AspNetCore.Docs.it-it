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
# <a name="simple-authorization-in-aspnet-core"></a>Autorizzazione semplice in ASP.NET Core

<a name="security-authorization-simple"></a>

L'autorizzazione in ASP.NET Core viene controllata con <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> e i vari parametri. Nella sua forma più semplice, applicando l' `[Authorize]` attributo a un controller, a un'azione o a una Razor pagina, è possibile limitare l'accesso a tale componente a qualsiasi utente autenticato.

Il codice seguente, ad esempio, limita l'accesso a `AccountController` a qualsiasi utente autenticato.

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

Se si desidera applicare l'autorizzazione a un'azione anziché al controller, applicare l' `AuthorizeAttribute` attributo all'azione stessa:

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

A questo punto, solo gli utenti autenticati possono accedere alla `Logout` funzione.

È anche possibile usare l' `AllowAnonymous` attributo per consentire l'accesso da parte di utenti non autenticati a singole azioni. Ad esempio:

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

Ciò consente solo agli utenti autenticati di `AccountController` , ad eccezione dell' `Login` azione, accessibile da parte di tutti, indipendentemente dallo stato autenticato o non autenticato/anonimo.

> [!WARNING]
> `[AllowAnonymous]` Ignora tutte le istruzioni di autorizzazione. Se si combinano `[AllowAnonymous]` e qualsiasi `[Authorize]` attributo, gli `[Authorize]` attributi vengono ignorati. Se ad esempio si applica `[AllowAnonymous]` a livello di controller, tutti `[Authorize]` gli attributi sullo stesso controller (o su qualsiasi azione al suo interno) vengono ignorati.

[!INCLUDE[](~/includes/requireAuth.md)]

<a name="aarp"></a>

## <a name="authorize-attribute-and-razor-pages"></a>Autorizzare gli attributi e le Razor pagine

<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ***Non*** può essere applicato ai Razor gestori di pagine. Ad esempio, `[Authorize]` non può essere applicato a `OnGet` , `OnPost` o a qualsiasi altro gestore di pagina. Considerare l'uso di un controller ASP.NET Core MVC per le pagine con requisiti di autorizzazione diversi per gestori diversi.

Per applicare l'autorizzazione ai metodi del gestore di pagina, è possibile usare i due approcci seguenti Razor :

* Usare pagine separate per i gestori di pagine che richiedono un'autorizzazione diversa. Spostare il contenuto condiviso in una o più [visualizzazioni parziali](xref:mvc/views/partial). Quando possibile, questo è l'approccio consigliato.
* Per il contenuto che deve condividere una pagina comune, scrivere un filtro che esegua l'autorizzazione come parte di [IAsyncPageFilter. OnPageHandlerSelectionAsync](xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter.OnPageHandlerSelectionAsync%2A). Il progetto GitHub [PageHandlerAuth](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth) illustra questo approccio:
  * [AuthorizeIndexPageHandlerFilter](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs) implementa il filtro di autorizzazione:[!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs?name=snippet)]

  * L'attributo [[AuthorizePageHandler]](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs#L16) viene applicato al `OnGet` gestore di pagina: [!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs?name=snippet)]

> [!WARNING]
> L' [](https://github.com/pranavkm/PageHandlerAuth) approccio di esempio PageHandlerAuth ***non***:
> * Comporre con attributi di autorizzazione applicati alla pagina, al modello di pagina o a livello globale. La composizione di attributi di autorizzazione comporta l'autenticazione e l'autorizzazione eseguite più volte quando si dispone `AuthorizeAttribute` di un'altra `AuthorizeFilter` istanza o di istanze applicate anche alla pagina.
> * Usare insieme al resto del sistema di autenticazione e autorizzazione ASP.NET Core. È necessario verificare che l'utilizzo di questo approccio funzioni correttamente per l'applicazione.

Non sono previsti piani per il supporto `AuthorizeAttribute` dei Razor gestori di pagine. 
