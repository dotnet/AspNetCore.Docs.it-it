---
title: BlazorPiattaforme supportate ASP.NET Core
author: guardrex
description: Informazioni sulle piattaforme supportate per ASP.NET Core Blazor .
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/01/2020
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
uid: blazor/supported-platforms
ms.openlocfilehash: 948c3e3f66da4727731b37491ae5c5470cfb36fe
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280714"
---
# <a name="aspnet-core-blazor-supported-platforms"></a><span data-ttu-id="c6b07-103">BlazorPiattaforme supportate ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="c6b07-103">ASP.NET Core Blazor supported platforms</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="c6b07-104">Blazor WebAssembly e Blazor Server sono supportati nei browser indicati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="c6b07-104">Blazor WebAssembly and Blazor Server are supported in the browsers shown in the following table.</span></span>

| <span data-ttu-id="c6b07-105">Browser</span><span class="sxs-lookup"><span data-stu-id="c6b07-105">Browser</span></span>                          | <span data-ttu-id="c6b07-106">Versione</span><span class="sxs-lookup"><span data-stu-id="c6b07-106">Version</span></span>         |
| -------------------------------- | --------------- |
| <span data-ttu-id="c6b07-107">Apple Safari, incluso iOS</span><span class="sxs-lookup"><span data-stu-id="c6b07-107">Apple Safari, including iOS</span></span>      | <span data-ttu-id="c6b07-108">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-108">Current&dagger;</span></span> |
| <span data-ttu-id="c6b07-109">Google Chrome, incluso Android</span><span class="sxs-lookup"><span data-stu-id="c6b07-109">Google Chrome, including Android</span></span> | <span data-ttu-id="c6b07-110">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-110">Current&dagger;</span></span> |
| <span data-ttu-id="c6b07-111">Microsoft Edge</span><span class="sxs-lookup"><span data-stu-id="c6b07-111">Microsoft Edge</span></span>                   | <span data-ttu-id="c6b07-112">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-112">Current&dagger;</span></span> |
| <span data-ttu-id="c6b07-113">Mozilla Firefox</span><span class="sxs-lookup"><span data-stu-id="c6b07-113">Mozilla Firefox</span></span>                  | <span data-ttu-id="c6b07-114">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-114">Current&dagger;</span></span> |  

<span data-ttu-id="c6b07-115">&dagger;*Current* si riferisce alla versione più recente del browser.</span><span class="sxs-lookup"><span data-stu-id="c6b07-115">&dagger;*Current* refers to the latest version of the browser.</span></span>  

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## Blazor WebAssembly

| <span data-ttu-id="c6b07-116">Browser</span><span class="sxs-lookup"><span data-stu-id="c6b07-116">Browser</span></span>                          | <span data-ttu-id="c6b07-117">Versione</span><span class="sxs-lookup"><span data-stu-id="c6b07-117">Version</span></span>               |
| -------------------------------- | --------------------- |
| <span data-ttu-id="c6b07-118">Apple Safari, incluso iOS</span><span class="sxs-lookup"><span data-stu-id="c6b07-118">Apple Safari, including iOS</span></span>      | <span data-ttu-id="c6b07-119">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-119">Current&dagger;</span></span>       |
| <span data-ttu-id="c6b07-120">Google Chrome, incluso Android</span><span class="sxs-lookup"><span data-stu-id="c6b07-120">Google Chrome, including Android</span></span> | <span data-ttu-id="c6b07-121">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-121">Current&dagger;</span></span>       |
| <span data-ttu-id="c6b07-122">Microsoft Edge</span><span class="sxs-lookup"><span data-stu-id="c6b07-122">Microsoft Edge</span></span>                   | <span data-ttu-id="c6b07-123">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-123">Current&dagger;</span></span>       |
| <span data-ttu-id="c6b07-124">Microsoft Internet Explorer</span><span class="sxs-lookup"><span data-stu-id="c6b07-124">Microsoft Internet Explorer</span></span>      | <span data-ttu-id="c6b07-125">Non supportato&Dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-125">Not Supported&Dagger;</span></span> |
| <span data-ttu-id="c6b07-126">Mozilla Firefox</span><span class="sxs-lookup"><span data-stu-id="c6b07-126">Mozilla Firefox</span></span>                  | <span data-ttu-id="c6b07-127">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-127">Current&dagger;</span></span>       |  

<span data-ttu-id="c6b07-128">&dagger;*Current* si riferisce alla versione più recente del browser.</span><span class="sxs-lookup"><span data-stu-id="c6b07-128">&dagger;*Current* refers to the latest version of the browser.</span></span>  
<span data-ttu-id="c6b07-129">&Dagger;Microsoft Internet Explorer non supporta [webassembly](https://webassembly.org).</span><span class="sxs-lookup"><span data-stu-id="c6b07-129">&Dagger;Microsoft Internet Explorer doesn't support [WebAssembly](https://webassembly.org).</span></span>

## Blazor Server

| <span data-ttu-id="c6b07-130">Browser</span><span class="sxs-lookup"><span data-stu-id="c6b07-130">Browser</span></span>                          | <span data-ttu-id="c6b07-131">Versione</span><span class="sxs-lookup"><span data-stu-id="c6b07-131">Version</span></span>         |
| -------------------------------- | --------------- |
| <span data-ttu-id="c6b07-132">Apple Safari, incluso iOS</span><span class="sxs-lookup"><span data-stu-id="c6b07-132">Apple Safari, including iOS</span></span>      | <span data-ttu-id="c6b07-133">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-133">Current&dagger;</span></span> |
| <span data-ttu-id="c6b07-134">Google Chrome, incluso Android</span><span class="sxs-lookup"><span data-stu-id="c6b07-134">Google Chrome, including Android</span></span> | <span data-ttu-id="c6b07-135">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-135">Current&dagger;</span></span> |
| <span data-ttu-id="c6b07-136">Microsoft Edge</span><span class="sxs-lookup"><span data-stu-id="c6b07-136">Microsoft Edge</span></span>                   | <span data-ttu-id="c6b07-137">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-137">Current&dagger;</span></span> |
| <span data-ttu-id="c6b07-138">Microsoft Internet Explorer</span><span class="sxs-lookup"><span data-stu-id="c6b07-138">Microsoft Internet Explorer</span></span>      | <span data-ttu-id="c6b07-139">11&Dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-139">11&Dagger;</span></span>      |
| <span data-ttu-id="c6b07-140">Mozilla Firefox</span><span class="sxs-lookup"><span data-stu-id="c6b07-140">Mozilla Firefox</span></span>                  | <span data-ttu-id="c6b07-141">Corrente&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6b07-141">Current&dagger;</span></span> |

<span data-ttu-id="c6b07-142">&dagger;*Current* si riferisce alla versione più recente del browser.</span><span class="sxs-lookup"><span data-stu-id="c6b07-142">&dagger;*Current* refers to the latest version of the browser.</span></span>  
<span data-ttu-id="c6b07-143">&Dagger;Sono necessarie altre ricompilazioni.</span><span class="sxs-lookup"><span data-stu-id="c6b07-143">&Dagger;Additional polyfills are required.</span></span> <span data-ttu-id="c6b07-144">Ad esempio, le promesse possono essere aggiunte tramite un [`Polyfill.io`](https://polyfill.io/v3/) bundle.</span><span class="sxs-lookup"><span data-stu-id="c6b07-144">For example, promises can be added via a [`Polyfill.io`](https://polyfill.io/v3/) bundle.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="c6b07-145">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="c6b07-145">Additional resources</span></span>

* <xref:blazor/hosting-models>
* <xref:signalr/supported-platforms>
