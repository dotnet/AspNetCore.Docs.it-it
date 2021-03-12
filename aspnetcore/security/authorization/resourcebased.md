---
title: Autorizzazione basata sulle risorse in ASP.NET Core
author: scottaddie
description: Informazioni su come implementare l'autorizzazione basata sulle risorse in un'app ASP.NET Core quando un attributo di autorizzazione non è sufficiente.
ms.author: scaddie
ms.custom: mvc
ms.date: 11/15/2018
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
uid: security/authorization/resourcebased
ms.openlocfilehash: 61c97be03709f63f57a6383ab0c51ca9a511fbbb
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585773"
---
# <a name="resource-based-authorization-in-aspnet-core"></a><span data-ttu-id="50d0c-103">Autorizzazione basata sulle risorse in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="50d0c-103">Resource-based authorization in ASP.NET Core</span></span>

<span data-ttu-id="50d0c-104">La strategia di autorizzazione dipende dalla risorsa a cui si accede.</span><span class="sxs-lookup"><span data-stu-id="50d0c-104">Authorization strategy depends upon the resource being accessed.</span></span> <span data-ttu-id="50d0c-105">Si consideri un documento con una proprietà Author.</span><span class="sxs-lookup"><span data-stu-id="50d0c-105">Consider a document that has an author property.</span></span> <span data-ttu-id="50d0c-106">Solo l'autore è autorizzato ad aggiornare il documento.</span><span class="sxs-lookup"><span data-stu-id="50d0c-106">Only the author is allowed to update the document.</span></span> <span data-ttu-id="50d0c-107">Di conseguenza, il documento deve essere recuperato dall'archivio dati prima che possa essere eseguita la valutazione dell'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="50d0c-107">Consequently, the document must be retrieved from the data store before authorization evaluation can occur.</span></span>

<span data-ttu-id="50d0c-108">La valutazione degli attributi viene eseguita prima data binding e prima dell'esecuzione del gestore di pagina o dell'azione che carica il documento.</span><span class="sxs-lookup"><span data-stu-id="50d0c-108">Attribute evaluation occurs before data binding and before execution of the page handler or action that loads the document.</span></span> <span data-ttu-id="50d0c-109">Per questi motivi, l'autorizzazione dichiarativa con un `[Authorize]` attributo non è sufficiente.</span><span class="sxs-lookup"><span data-stu-id="50d0c-109">For these reasons, declarative authorization with an `[Authorize]` attribute doesn't suffice.</span></span> <span data-ttu-id="50d0c-110">In alternativa, è possibile richiamare un metodo di autorizzazione personalizzato &mdash; con uno stile noto come *autorizzazione imperativa*.</span><span class="sxs-lookup"><span data-stu-id="50d0c-110">Instead, you can invoke a custom authorization method&mdash;a style known as *imperative authorization*.</span></span>

::: moniker range=">= aspnetcore-3.0"
<span data-ttu-id="50d0c-111">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/resourcebased/samples/3_0) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="50d0c-111">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/resourcebased/samples/3_0) ([how to download](xref:index#how-to-download-a-sample)).</span></span>
::: moniker-end

 ::: moniker range=">= aspnetcore-2.0 < aspnetcore-3.0"
<span data-ttu-id="50d0c-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/resourcebased/samples/2_2) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="50d0c-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/resourcebased/samples/2_2) ([how to download](xref:index#how-to-download-a-sample)).</span></span>
::: moniker-end

::: moniker range="<= aspnetcore-1.1"
<span data-ttu-id="50d0c-113">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/resourcebased/samples/1_1) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="50d0c-113">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authorization/resourcebased/samples/1_1) ([how to download](xref:index#how-to-download-a-sample)).</span></span>
::: moniker-end

<span data-ttu-id="50d0c-114">[Creare un'app ASP.NET Core con i dati utente protetti dall'autorizzazione](xref:security/authorization/secure-data) contiene un'app di esempio che usa l'autorizzazione basata sulle risorse.</span><span class="sxs-lookup"><span data-stu-id="50d0c-114">[Create an ASP.NET Core app with user data protected by authorization](xref:security/authorization/secure-data) contains a sample app that uses resource-based authorization.</span></span>

## <a name="use-imperative-authorization"></a><span data-ttu-id="50d0c-115">USA autorizzazione imperativa</span><span class="sxs-lookup"><span data-stu-id="50d0c-115">Use imperative authorization</span></span>

<span data-ttu-id="50d0c-116">L'autorizzazione viene implementata come servizio [IAuthorizationService](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice) e viene registrata nella raccolta di servizi all'interno della `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="50d0c-116">Authorization is implemented as an [IAuthorizationService](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice) service and is registered in the service collection within the `Startup` class.</span></span> <span data-ttu-id="50d0c-117">Il servizio viene reso disponibile tramite l' [inserimento di dipendenze](xref:fundamentals/dependency-injection) in gestori di pagine o azioni.</span><span class="sxs-lookup"><span data-stu-id="50d0c-117">The service is made available via [dependency injection](xref:fundamentals/dependency-injection) to page handlers or actions.</span></span>

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Controllers/DocumentController.cs?name=snippet_IAuthServiceDI&highlight=6)]

<span data-ttu-id="50d0c-118">`IAuthorizationService` dispone di due `AuthorizeAsync` Overload del metodo: uno accetta la risorsa e il nome del criterio e l'altro accetta la risorsa e un elenco di requisiti da valutare.</span><span class="sxs-lookup"><span data-stu-id="50d0c-118">`IAuthorizationService` has two `AuthorizeAsync` method overloads: one accepting the resource and the policy name and the other accepting the resource and a list of requirements to evaluate.</span></span>

::: moniker range=">= aspnetcore-2.0"

```csharp
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

```csharp
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

<a name="security-authorization-resource-based-imperative"></a>

<span data-ttu-id="50d0c-119">Nell'esempio seguente la risorsa da proteggere viene caricata in un `Document` oggetto personalizzato.</span><span class="sxs-lookup"><span data-stu-id="50d0c-119">In the following example, the resource to be secured is loaded into a custom `Document` object.</span></span> <span data-ttu-id="50d0c-120">`AuthorizeAsync`Viene richiamato un overload per determinare se l'utente corrente è autorizzato a modificare il documento specificato.</span><span class="sxs-lookup"><span data-stu-id="50d0c-120">An `AuthorizeAsync` overload is invoked to determine whether the current user is allowed to edit the provided document.</span></span> <span data-ttu-id="50d0c-121">I criteri di autorizzazione "EditPolicy" personalizzati vengono presi in considerazione nella decisione.</span><span class="sxs-lookup"><span data-stu-id="50d0c-121">A custom "EditPolicy" authorization policy is factored into the decision.</span></span> <span data-ttu-id="50d0c-122">Per altre informazioni sulla creazione di criteri di autorizzazione, vedere [autorizzazione personalizzata basata su criteri](xref:security/authorization/policies) .</span><span class="sxs-lookup"><span data-stu-id="50d0c-122">See [Custom policy-based authorization](xref:security/authorization/policies) for more on creating authorization policies.</span></span>

> [!NOTE]
> <span data-ttu-id="50d0c-123">Negli esempi di codice seguenti si presuppone che l'autenticazione sia stata eseguita e che sia stata impostata la `User` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="50d0c-123">The following code samples assume authentication has run and set the `User` property.</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Pages/Document/Edit.cshtml.cs?name=snippet_DocumentEditHandler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentEditAction)]

::: moniker-end

## <a name="write-a-resource-based-handler"></a><span data-ttu-id="50d0c-124">Scrivere un gestore basato su risorse</span><span class="sxs-lookup"><span data-stu-id="50d0c-124">Write a resource-based handler</span></span>

<span data-ttu-id="50d0c-125">La scrittura di un gestore per l'autorizzazione basata sulle risorse non è molto diversa dalla [scrittura di un gestore di requisiti semplici](xref:security/authorization/policies#security-authorization-policies-based-authorization-handler).</span><span class="sxs-lookup"><span data-stu-id="50d0c-125">Writing a handler for resource-based authorization isn't much different than [writing a plain requirements handler](xref:security/authorization/policies#security-authorization-policies-based-authorization-handler).</span></span> <span data-ttu-id="50d0c-126">Creare una classe di requisiti personalizzata e implementare una classe del gestore di requisiti.</span><span class="sxs-lookup"><span data-stu-id="50d0c-126">Create a custom requirement class, and implement a requirement handler class.</span></span> <span data-ttu-id="50d0c-127">Per ulteriori informazioni sulla creazione di una classe requirement, vedere [requirements](xref:security/authorization/policies#requirements).</span><span class="sxs-lookup"><span data-stu-id="50d0c-127">For more information on creating a requirement class, see [Requirements](xref:security/authorization/policies#requirements).</span></span>

<span data-ttu-id="50d0c-128">La classe handler specifica sia il requisito che il tipo di risorsa.</span><span class="sxs-lookup"><span data-stu-id="50d0c-128">The handler class specifies both the requirement and resource type.</span></span> <span data-ttu-id="50d0c-129">Ad esempio, un gestore che usa un `SameAuthorRequirement` e una `Document` risorsa segue:</span><span class="sxs-lookup"><span data-stu-id="50d0c-129">For example, a handler utilizing a `SameAuthorRequirement` and a `Document` resource follows:</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

<span data-ttu-id="50d0c-130">Nell'esempio precedente, si supponga che `SameAuthorRequirement` sia un caso speciale di una classe più generica `SpecificAuthorRequirement` .</span><span class="sxs-lookup"><span data-stu-id="50d0c-130">In the preceding example, imagine that `SameAuthorRequirement` is a special case of a more generic `SpecificAuthorRequirement` class.</span></span> <span data-ttu-id="50d0c-131">La `SpecificAuthorRequirement` classe (non mostrata) contiene una `Name` proprietà che rappresenta il nome dell'autore.</span><span class="sxs-lookup"><span data-stu-id="50d0c-131">The `SpecificAuthorRequirement` class (not shown) contains a `Name` property representing the name of the author.</span></span> <span data-ttu-id="50d0c-132">`Name`È possibile impostare la proprietà sull'utente corrente.</span><span class="sxs-lookup"><span data-stu-id="50d0c-132">The `Name` property could be set to the current user.</span></span>

<span data-ttu-id="50d0c-133">Registrare il requisito e il gestore in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="50d0c-133">Register the requirement and handler in `Startup.ConfigureServices`:</span></span>

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Startup.cs?name=snippet_ConfigureServicesSample&highlight=4-8,10)]
::: moniker-end

 ::: moniker range=">= aspnetcore-2.0 < aspnetcore-3.0"
[!code-csharp[](resourcebased/samples/2_2/ResourceBasedAuthApp2/Startup.cs?name=snippet_ConfigureServicesSample&highlight=3-7,9)]
::: moniker-end

::: moniker range="<= aspnetcore-1.1"
[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Startup.cs?name=snippet_ConfigureServicesSample&highlight=3-7,9)]
::: moniker-end

### <a name="operational-requirements"></a><span data-ttu-id="50d0c-134">Requisiti operativi</span><span class="sxs-lookup"><span data-stu-id="50d0c-134">Operational requirements</span></span>

<span data-ttu-id="50d0c-135">Se si stanno prendendo decisioni in base ai risultati delle operazioni CRUD (create, Read, Update, Delete), usare la classe helper [OperationAuthorizationRequirement](/dotnet/api/microsoft.aspnetcore.authorization.infrastructure.operationauthorizationrequirement) .</span><span class="sxs-lookup"><span data-stu-id="50d0c-135">If you're making decisions based on the outcomes of CRUD (Create, Read, Update, Delete) operations, use the [OperationAuthorizationRequirement](/dotnet/api/microsoft.aspnetcore.authorization.infrastructure.operationauthorizationrequirement) helper class.</span></span> <span data-ttu-id="50d0c-136">Questa classe consente di scrivere un singolo gestore anziché una singola classe per ogni tipo di operazione.</span><span class="sxs-lookup"><span data-stu-id="50d0c-136">This class enables you to write a single handler instead of an individual class for each operation type.</span></span> <span data-ttu-id="50d0c-137">Per usarlo, specificare alcuni nomi di operazione:</span><span class="sxs-lookup"><span data-stu-id="50d0c-137">To use it, provide some operation names:</span></span>

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_OperationsClass)]

<span data-ttu-id="50d0c-138">Il gestore viene implementato come segue, usando un `OperationAuthorizationRequirement` requisito e una `Document` risorsa:</span><span class="sxs-lookup"><span data-stu-id="50d0c-138">The handler is implemented as follows, using an `OperationAuthorizationRequirement` requirement and a `Document` resource:</span></span>

 ::: moniker range=">= aspnetcore-2.0"
[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

<span data-ttu-id="50d0c-139">Il gestore precedente convalida l'operazione usando la risorsa, l'identità dell'utente e la proprietà del requisito `Name` .</span><span class="sxs-lookup"><span data-stu-id="50d0c-139">The preceding handler validates the operation using the resource, the user's identity, and the requirement's `Name` property.</span></span>

## <a name="challenge-and-forbid-with-an-operational-resource-handler"></a><span data-ttu-id="50d0c-140">Verifica e Impedisci un gestore di risorse operativo</span><span class="sxs-lookup"><span data-stu-id="50d0c-140">Challenge and forbid with an operational resource handler</span></span>

<span data-ttu-id="50d0c-141">In questa sezione viene illustrato il modo in cui vengono elaborati i risultati dell'azione di richiesta e proibisce e il modo in cui le</span><span class="sxs-lookup"><span data-stu-id="50d0c-141">This section shows how the challenge and forbid action results are processed and how challenge and forbid differ.</span></span>

<span data-ttu-id="50d0c-142">Per chiamare un gestore di risorse operative, specificare l'operazione quando si richiama `AuthorizeAsync` nel gestore o nell'azione della pagina.</span><span class="sxs-lookup"><span data-stu-id="50d0c-142">To call an operational resource handler, specify the operation when invoking `AuthorizeAsync` in your page handler or action.</span></span> <span data-ttu-id="50d0c-143">Nell'esempio seguente viene determinato se l'utente autenticato è autorizzato a visualizzare il documento specificato.</span><span class="sxs-lookup"><span data-stu-id="50d0c-143">The following example determines whether the authenticated user is permitted to view the provided document.</span></span>

> [!NOTE]
> <span data-ttu-id="50d0c-144">Negli esempi di codice seguenti si presuppone che l'autenticazione sia stata eseguita e che sia stata impostata la `User` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="50d0c-144">The following code samples assume authentication has run and set the `User` property.</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Pages/Document/View.cshtml.cs?name=snippet_DocumentViewHandler&highlight=10-11)]

<span data-ttu-id="50d0c-145">Se l'autorizzazione ha esito positivo, viene restituita la pagina per la visualizzazione del documento.</span><span class="sxs-lookup"><span data-stu-id="50d0c-145">If authorization succeeds, the page for viewing the document is returned.</span></span> <span data-ttu-id="50d0c-146">Se l'autorizzazione ha esito negativo, ma l'utente viene autenticato, restituendo `ForbidResult` informa qualsiasi middleware di autenticazione che l'autorizzazione non è riuscita.</span><span class="sxs-lookup"><span data-stu-id="50d0c-146">If authorization fails but the user is authenticated, returning `ForbidResult` informs any authentication middleware that authorization failed.</span></span> <span data-ttu-id="50d0c-147">`ChallengeResult`Viene restituito un oggetto quando è necessario eseguire l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="50d0c-147">A `ChallengeResult` is returned when authentication must be performed.</span></span> <span data-ttu-id="50d0c-148">Per i client del browser interattivo, potrebbe essere opportuno reindirizzare l'utente a una pagina di accesso.</span><span class="sxs-lookup"><span data-stu-id="50d0c-148">For interactive browser clients, it may be appropriate to redirect the user to a login page.</span></span>

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentViewAction&highlight=11-12)]

<span data-ttu-id="50d0c-149">Se l'autorizzazione ha esito positivo, viene restituita la vista per il documento.</span><span class="sxs-lookup"><span data-stu-id="50d0c-149">If authorization succeeds, the view for the document is returned.</span></span> <span data-ttu-id="50d0c-150">Se l'autorizzazione ha esito negativo, restituendo `ChallengeResult` informa il middleware di autenticazione che l'autorizzazione non è riuscita e il middleware può prendere la risposta appropriata.</span><span class="sxs-lookup"><span data-stu-id="50d0c-150">If authorization fails, returning `ChallengeResult` informs any authentication middleware that authorization failed, and the middleware can take the appropriate response.</span></span> <span data-ttu-id="50d0c-151">Una risposta appropriata potrebbe restituire un codice di stato 401 o 403.</span><span class="sxs-lookup"><span data-stu-id="50d0c-151">An appropriate response could be returning a 401 or 403 status code.</span></span> <span data-ttu-id="50d0c-152">Per i client del browser interattivo, potrebbe significare il reindirizzamento dell'utente a una pagina di accesso.</span><span class="sxs-lookup"><span data-stu-id="50d0c-152">For interactive browser clients, it could mean redirecting the user to a login page.</span></span>

::: moniker-end
