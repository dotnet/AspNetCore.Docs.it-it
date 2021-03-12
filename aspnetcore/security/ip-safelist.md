---
title: Indirizzi IP attendibili del client per ASP.NET Core
author: damienbod
description: Informazioni su come scrivere filtri middleware o azione per convalidare gli indirizzi IP remoti rispetto a un elenco di indirizzi IP approvati.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/12/2020
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
uid: security/ip-safelist
ms.openlocfilehash: f89191cff1faa5b55ee9031a6a6d0749a74e43c0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587541"
---
# <a name="client-ip-safelist-for-aspnet-core"></a>Indirizzi IP attendibili del client per ASP.NET Core

Di [Damien Bowden](https://twitter.com/damien_bod) e [Tom Dykstra](https://github.com/tdykstra)
 
Questo articolo illustra tre modi per implementare un elenco di indirizzi IP attendibili (noto anche come elenco di accesso consentito) in un'app ASP.NET Core. Un'app di esempio associata illustra tutti e tre gli approcci. È possibile usare:

* Middleware per controllare l'indirizzo IP remoto di ogni richiesta.
* Filtri azione MVC per controllare l'indirizzo IP remoto delle richieste per controller o metodi di azione specifici.
* Razor Filtra le pagine per controllare l'indirizzo IP remoto delle richieste di Razor pagine.

In ogni caso, una stringa contenente gli indirizzi IP client approvati viene archiviata in un'impostazione dell'app. Il middleware o il filtro:

* Analizza la stringa in una matrice. 
* Verifica se l'indirizzo IP remoto è presente nella matrice.

L'accesso è consentito se la matrice contiene l'indirizzo IP. In caso contrario, viene restituito un codice di stato HTTP 403 Forbidden.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/ip-safelist/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="ip-address-safelist"></a>Indirizzi IP attendibili

Nell'app di esempio, l'indirizzo IP attendibile è:

* Definito dalla `AdminSafeList` proprietà nel *appsettings.json* file.
* Stringa delimitata da punti e virgola che può contenere indirizzi IP [versione 4 (IPv4)](https://wikipedia.org/wiki/IPv4) e [IPv6 (Internet Protocol versione 6)](https://wikipedia.org/wiki/IPv6) .

[!code-json[](ip-safelist/samples/3.x/ClientIpAspNetCore/appsettings.json?range=1-3&highlight=2)]

Nell'esempio precedente `127.0.0.1` sono consentiti gli indirizzi IPv4 di e `192.168.1.5` e l'indirizzo di loopback IPv6 `::1` (formato compresso per `0:0:0:0:0:0:0:1` ).

## <a name="middleware"></a>Middleware

Il `Startup.Configure` metodo aggiunge il `AdminSafeListMiddleware` tipo di middleware personalizzato alla pipeline delle richieste dell'app. L'oggetto safename viene recuperato con il provider di configurazione .NET Core e viene passato come parametro del costruttore.

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureAddMiddleware)]

Il middleware analizza la stringa in una matrice e cerca l'indirizzo IP remoto nella matrice. Se l'indirizzo IP remoto non viene trovato, il middleware restituisce HTTP 403 non consentito. Questo processo di convalida viene ignorato per le richieste HTTP GET.

[!code-csharp[](ip-safelist/samples/Shared/ClientIpSafelistComponents/Middlewares/AdminSafeListMiddleware.cs?name=snippet_ClassOnly)]

## <a name="action-filter"></a>Filtro azioni

Se si desidera il controllo degli accessi basato su attendibili per controller MVC o metodi di azione specifici, utilizzare un filtro azioni. Ad esempio:

[!code-csharp[](ip-safelist/samples/Shared/ClientIpSafelistComponents/Filters/ClientIpCheckActionFilter.cs?name=snippet_ClassOnly)]

In `Startup.ConfigureServices` aggiungere il filtro azioni alla raccolta di filtri MVC. Nell'esempio seguente `ClientIpCheckActionFilter` viene aggiunto un filtro azioni. Un oggetto safe e un'istanza del logger della console vengono passati come parametri del costruttore.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesActionFilter)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesActionFilter)]

::: moniker-end

Il filtro azioni può quindi essere applicato a un controller o a un metodo di azione con l'attributo [[ServiceFilter]](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute) :

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Controllers/ValuesController.cs?name=snippet_ActionFilter&highlight=1)]

Nell'app di esempio, il filtro azioni viene applicato al metodo di azione del controller `Get` . Quando si esegue il test dell'app inviando:

* Una richiesta HTTP GET, l' `[ServiceFilter]` attributo convalida l'indirizzo IP del client. Se è consentito l'accesso al `Get` metodo di azione, viene generata una variante del seguente output della console dal filtro azioni e dal metodo di azione:

    ```
    dbug: ClientIpSafelistComponents.Filters.ClientIpCheckActionFilter[0]
          Remote IpAddress: ::1
    dbug: ClientIpAspNetCore.Controllers.ValuesController[0]
          successful HTTP GET    
    ```

* Un verbo di richiesta HTTP diverso da GET, il `AdminSafeListMiddleware` middleware convalida l'indirizzo IP del client.

## <a name="razor-pages-filter"></a>Razor Filtro pagine

Se si desidera il controllo degli accessi basato su attendibilità per un' Razor app pagine, utilizzare un Razor filtro pagine. Ad esempio:

[!code-csharp[](ip-safelist/samples/Shared/ClientIpSafelistComponents/Filters/ClientIpCheckPageFilter.cs?name=snippet_ClassOnly)]

In `Startup.ConfigureServices` abilitare il Razor filtro pagine aggiungendolo alla raccolta di filtri MVC. Nell'esempio seguente `ClientIpCheckPageFilter` Razor viene aggiunto un filtro Pages. Un oggetto safe e un'istanza del logger della console vengono passati come parametri del costruttore.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesPageFilter)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesPageFilter)]

::: moniker-end

Quando viene richiesta la pagina di *Indice* dell'app di esempio Razor , il Razor filtro pagine convalida l'indirizzo IP del client. Il filtro produce una variante dell'output della console seguente:

```
dbug: ClientIpSafelistComponents.Filters.ClientIpCheckPageFilter[0]
      Remote IpAddress: ::1
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/middleware/index>
* [Filtri azione](xref:mvc/controllers/filters#action-filters)
* <xref:razor-pages/filter>
