---
title: Schemi di criteri in ASP.NET Core
author: rick-anderson
description: Gli schemi dei criteri di autenticazione rendono più semplice avere un unico schema di autenticazione logica
ms.author: riande
ms.date: 12/05/2019
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
uid: security/authentication/policyschemes
ms.openlocfilehash: 63d931c926c9660f5d68d5a2ce292bf57efdb49c
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053228"
---
# <a name="policy-schemes-in-aspnet-core"></a>Schemi di criteri in ASP.NET Core

Gli schemi dei criteri di autenticazione rendono più semplice disporre di un singolo schema di autenticazione logica che potenzialmente utilizza diversi approcci. Ad esempio, uno schema di criteri può utilizzare l'autenticazione Google per le richieste e cookie l'autenticazione per tutti gli altri elementi. Gli schemi dei criteri di autenticazione lo rendono:

* Facile da inviare qualsiasi azione di autenticazione a un altro schema.
* In avanti in modo dinamico in base alla richiesta.

Tutti gli schemi di autenticazione che usano Derived <xref:Microsoft.AspNetCore.Authentication.AuthenticationSchemeOptions> e il [AuthenticationHandler \<TOptions> ](/dotnet/api/microsoft.aspnetcore.authentication.authenticationhandler-1)associato:

* Gli schemi di criteri sono automaticamente in ASP.NET Core 2,1 e versioni successive.
* Può essere abilitato tramite la configurazione delle opzioni dello schema.

[!code-csharp[sample](policyschemes/samples/AuthenticationSchemeOptions.cs?name=snippet)]

## <a name="examples"></a>Esempio

Nell'esempio seguente viene illustrato uno schema di livello superiore che combina schemi di livello inferiore. Google Authentication viene usato per le richieste e cookie l'autenticazione viene usata per tutte le altre operazioni:

[!code-csharp[sample](policyschemes/samples/Startup.cs?name=snippet1)]

Nell'esempio seguente viene abilitata la selezione dinamica degli schemi in base alle singole richieste. Ovvero come combinare cookie s e l'autenticazione API:

 <!-- REVIEW, missing If set in public Func<HttpContext, string> ForwardDefaultSelector -->

[!code-csharp[sample](policyschemes/samples/Startup.cs?name=snippet2)]
