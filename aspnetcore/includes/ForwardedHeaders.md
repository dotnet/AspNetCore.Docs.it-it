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
ms.openlocfilehash: ddc6e2abf60577644a38b0b6ad0c74a734205ef5
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551346"
---
Il middleware delle intestazioni con inoltri deve essere eseguito prima di altri middleware. Questo ordine garantisce che il middleware basato sulle intestazioni inoltrate possa usare i valori di intestazione per l'elaborazione. Per eseguire il middleware delle intestazioni in avanti dopo la diagnostica e il middleware di gestione degli errori, vedere l' [ordine del middleware delle intestazioni inoltri](xref:host-and-deploy/proxy-load-balancer#fhmo).