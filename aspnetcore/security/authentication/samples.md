---
title: Esempi di autenticazione per ASP.NET Core
author: rick-anderson
description: Fornisce collegamenti agli esempi di autenticazione nel repository ASP.NET Core.
ms.author: riande
ms.date: 02/21/2021
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
uid: security/authentication/samples
ms.openlocfilehash: e7fb2ac32f57cf4ecd3c5db294bd0df8e186b6c6
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110118"
---
# <a name="authentication-samples-for-aspnet-core"></a><span data-ttu-id="52112-103">Esempi di autenticazione per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="52112-103">Authentication samples for ASP.NET Core</span></span>

<span data-ttu-id="52112-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="52112-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="52112-105">Il [repository ASP.NET Core](https://github.com/dotnet/aspnetcore) contiene i seguenti esempi di autenticazione ( `main` Branch):</span><span class="sxs-lookup"><span data-stu-id="52112-105">The [ASP.NET Core repository](https://github.com/dotnet/aspnetcore) contains the following authentication samples (`main` branch):</span></span>

* [<span data-ttu-id="52112-106">Trasformazione delle attestazioni</span><span class="sxs-lookup"><span data-stu-id="52112-106">Claims transformation</span></span>](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/ClaimsTransformation)
* <span data-ttu-id="52112-107">[Cookie autenticazione](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Cookies)</span><span class="sxs-lookup"><span data-stu-id="52112-107">[Cookie authentication](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Cookies)</span></span>
* [<span data-ttu-id="52112-108">Risposta di errore di autorizzazione personalizzata</span><span class="sxs-lookup"><span data-stu-id="52112-108">Custom authorization failure response</span></span>](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/CustomAuthorizationFailureResponse)
* [<span data-ttu-id="52112-109">Provider di criteri personalizzati-IAuthorizationPolicyProvider</span><span class="sxs-lookup"><span data-stu-id="52112-109">Custom policy provider - IAuthorizationPolicyProvider</span></span>](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/CustomPolicyProvider)
* [<span data-ttu-id="52112-110">Opzioni e schemi di autenticazione dinamici</span><span class="sxs-lookup"><span data-stu-id="52112-110">Dynamic authentication schemes and options</span></span>](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/DynamicSchemes)
* <span data-ttu-id="52112-111">[Attestazioni esterne](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Identity.ExternalClaims)</span><span class="sxs-lookup"><span data-stu-id="52112-111">[External claims](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Identity.ExternalClaims)</span></span>
* [<span data-ttu-id="52112-112">Selezione tra cookie e un altro schema di autenticazione in base alla richiesta</span><span class="sxs-lookup"><span data-stu-id="52112-112">Selecting between cookie and another authentication scheme based on the request</span></span>](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/PathSchemeSelection)
* [<span data-ttu-id="52112-113">Limita l'accesso ai file statici</span><span class="sxs-lookup"><span data-stu-id="52112-113">Restricts access to static files</span></span>](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/StaticFilesAuth)

## <a name="obtain-and-run-the-samples"></a><span data-ttu-id="52112-114">Ottenere ed eseguire gli esempi</span><span class="sxs-lookup"><span data-stu-id="52112-114">Obtain and run the samples</span></span>

<span data-ttu-id="52112-115">I collegamenti di esempio forniti in questo articolo forniscono esempi per la prossima versione di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="52112-115">The sample links provided in this article provide samples for the upcoming release of ASP.NET Core.</span></span> <span data-ttu-id="52112-116">Per ottenere un esempio per la versione corrente o una versione precedente, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="52112-116">To obtain a sample for the current release or a prior release, perform the following steps:</span></span>

* <span data-ttu-id="52112-117">Selezionare il ramo di rilascio del [repository ASP.NET Core](https://github.com/dotnet/aspnetcore)] ( https://github.com/dotnet/aspnetcore) .</span><span class="sxs-lookup"><span data-stu-id="52112-117">Select the release branch of the [ASP.NET Core repository](https://github.com/dotnet/aspnetcore)](https://github.com/dotnet/aspnetcore).</span></span> <span data-ttu-id="52112-118">Ad esempio, il `release/5.0` Branch contiene gli esempi per la versione di ASP.NET Core 5,0.</span><span class="sxs-lookup"><span data-stu-id="52112-118">For example, the `release/5.0` branch contains the samples for the ASP.NET Core 5.0 release.</span></span>
* <span data-ttu-id="52112-119">Clonare o scaricare il repository ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="52112-119">Clone or download the ASP.NET Core repository.</span></span>
* <span data-ttu-id="52112-120">Nel sistema locale verificare l'installazione della versione [.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core) corrispondente al clone del repository ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="52112-120">On your local system, verify installation of the [.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core) version matching the clone of the ASP.NET Core repository.</span></span>
* <span data-ttu-id="52112-121">Passare a un esempio nella `aspnetcore/src/Security/samples` cartella ed eseguire l'esempio con il [ `dotnet run` comando](/dotnet/core/tools/dotnet-run).</span><span class="sxs-lookup"><span data-stu-id="52112-121">Navigate to a sample in `aspnetcore/src/Security/samples` folder and run the sample with the [`dotnet run` command](/dotnet/core/tools/dotnet-run).</span></span>
