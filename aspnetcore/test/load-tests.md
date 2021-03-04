---
title: ASP.NET Core test di carico/stress
author: Jeremy-Meng
description: Informazioni sui diversi strumenti e approcci rilevanti per test di carico e test di stress ASP.NET Core app.
ms.author: riande
ms.custom: mvc
ms.date: 4/05/2019
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
uid: test/loadtests
ms.openlocfilehash: 5a860fe398dff2e12671b6854a80bbab3120499b
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109481"
---
# <a name="aspnet-core-loadstress-testing"></a>ASP.NET Core test di carico/stress

Test di carico e test di stress sono importanti per garantire che un'app Web sia efficiente e scalabile. I rispettivi obiettivi sono diversi anche se condividono spesso test simili.

**Test di carico**: verificare se l'app è in grado di gestire un carico specificato di utenti per un determinato scenario soddisfacendo comunque l'obiettivo della risposta. L'app viene eseguita in condizioni normali.

**Test di stress**: verifica la stabilità dell'app quando viene eseguita in condizioni estreme, spesso per un lungo periodo di tempo. I test inseriscono un carico utente elevato, picchi o aumento graduale del carico, sull'app o limitano le risorse di elaborazione dell'app.

I test di stress determinano se un'app in stress può recuperare da un errore e restituire normalmente il comportamento previsto. In condizioni di stress, l'app non viene eseguita in condizioni normali.

Visual Studio 2019 ha annunciato piani per [deprecare il test di carico](https://devblogs.microsoft.com/devops/cloud-based-load-testing-service-eol/). Il servizio di test di carico basato sul cloud di Azure DevOps corrispondente è stato chiuso.

## <a name="third-party-tools"></a>Strumenti di terzi

L'elenco seguente contiene gli strumenti per le prestazioni Web di terze parti con diversi set di funzionalità:

* [Apache JMeter](https://jmeter.apache.org/)
* [ApacheBench (AB)](https://httpd.apache.org/docs/2.4/programs/ab.html)
* [Gatling](https://gatling.io/)
* [K6](https://k6.io)
* [Locusta](https://locust.io/)
* [Websurge di Wind West](https://websurge.west-wind.com/)
* [Netling](https://github.com/hallatore/Netling)
* [Vegeta](https://github.com/tsenart/vegeta)
* [NBomber](https://github.com/PragmaticFlow/NBomber)
