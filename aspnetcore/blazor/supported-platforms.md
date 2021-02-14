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
# <a name="aspnet-core-blazor-supported-platforms"></a>BlazorPiattaforme supportate ASP.NET Core

::: moniker range=">= aspnetcore-5.0"

Blazor WebAssembly e Blazor Server sono supportati nei browser indicati nella tabella seguente.

| Browser                          | Versione         |
| -------------------------------- | --------------- |
| Apple Safari, incluso iOS      | Corrente&dagger; |
| Google Chrome, incluso Android | Corrente&dagger; |
| Microsoft Edge                   | Corrente&dagger; |
| Mozilla Firefox                  | Corrente&dagger; |  

&dagger;*Current* si riferisce alla versione più recente del browser.  

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## Blazor WebAssembly

| Browser                          | Versione               |
| -------------------------------- | --------------------- |
| Apple Safari, incluso iOS      | Corrente&dagger;       |
| Google Chrome, incluso Android | Corrente&dagger;       |
| Microsoft Edge                   | Corrente&dagger;       |
| Microsoft Internet Explorer      | Non supportato&Dagger; |
| Mozilla Firefox                  | Corrente&dagger;       |  

&dagger;*Current* si riferisce alla versione più recente del browser.  
&Dagger;Microsoft Internet Explorer non supporta [webassembly](https://webassembly.org).

## Blazor Server

| Browser                          | Versione         |
| -------------------------------- | --------------- |
| Apple Safari, incluso iOS      | Corrente&dagger; |
| Google Chrome, incluso Android | Corrente&dagger; |
| Microsoft Edge                   | Corrente&dagger; |
| Microsoft Internet Explorer      | 11&Dagger;      |
| Mozilla Firefox                  | Corrente&dagger; |

&dagger;*Current* si riferisce alla versione più recente del browser.  
&Dagger;Sono necessarie altre ricompilazioni. Ad esempio, le promesse possono essere aggiunte tramite un [`Polyfill.io`](https://polyfill.io/v3/) bundle.

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/hosting-models>
* <xref:signalr/supported-platforms>
