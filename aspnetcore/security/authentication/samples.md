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
# <a name="authentication-samples-for-aspnet-core"></a>Esempi di autenticazione per ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

Il [repository ASP.NET Core](https://github.com/dotnet/aspnetcore) contiene i seguenti esempi di autenticazione ( `main` Branch):

* [Trasformazione delle attestazioni](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/ClaimsTransformation)
* [Cookie autenticazione](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Cookies)
* [Risposta di errore di autorizzazione personalizzata](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/CustomAuthorizationFailureResponse)
* [Provider di criteri personalizzati-IAuthorizationPolicyProvider](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/CustomPolicyProvider)
* [Opzioni e schemi di autenticazione dinamici](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/DynamicSchemes)
* [Attestazioni esterne](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/Identity.ExternalClaims)
* [Selezione tra cookie e un altro schema di autenticazione in base alla richiesta](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/PathSchemeSelection)
* [Limita l'accesso ai file statici](https://github.com/dotnet/aspnetcore/tree/main/src/Security/samples/StaticFilesAuth)

## <a name="obtain-and-run-the-samples"></a>Ottenere ed eseguire gli esempi

I collegamenti di esempio forniti in questo articolo forniscono esempi per la prossima versione di ASP.NET Core. Per ottenere un esempio per la versione corrente o una versione precedente, seguire questa procedura:

* Selezionare il ramo di rilascio del [repository ASP.NET Core](https://github.com/dotnet/aspnetcore)] ( https://github.com/dotnet/aspnetcore) . Ad esempio, il `release/5.0` Branch contiene gli esempi per la versione di ASP.NET Core 5,0.
* Clonare o scaricare il repository ASP.NET Core.
* Nel sistema locale verificare l'installazione della versione [.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core) corrispondente al clone del repository ASP.NET Core.
* Passare a un esempio nella `aspnetcore/src/Security/samples` cartella ed eseguire l'esempio con il [ `dotnet run` comando](/dotnet/core/tools/dotnet-run).
