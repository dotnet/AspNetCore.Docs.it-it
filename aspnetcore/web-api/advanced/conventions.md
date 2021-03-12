---
title: Usare le convenzioni dell'API Web
author: pranavkm
description: Informazioni sulle convenzioni dell'API Web in ASP.NET Core.
monikerRange: '>= aspnetcore-2.2'
ms.author: scaddie
ms.custom: mvc
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
uid: web-api/advanced/conventions
ms.openlocfilehash: 1e6526f46fbd177add3699fb5b667021b741c6a4
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587840"
---
# <a name="use-web-api-conventions"></a>Usare le convenzioni dell'API Web

Di [Pranav Krishnamoorthy](https://github.com/pranavkm) e [Scott Addie](https://github.com/scottaddie)

ASP.NET Core 2.2 e versioni successive include una modalità per l'estrazione di [documentazione dell'API](xref:tutorials/web-api-help-pages-using-swagger) comune e l'applicazione della documentazione a più azioni o controller, oppure a tutti i controller inclusi in un assembly. Le convenzioni API Web sono un sostituto per la decorazione di singole azioni con [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) .

Una convenzione consente di:

* Definire i tipi restituiti più comuni e i codici di stato restituiti da un tipo di azione specifico.
* Identificare le azioni che deviano dallo standard definito.

ASP.NET Core MVC 2.2 e versioni successive include un set di convenzioni predefinite in <xref:Microsoft.AspNetCore.Mvc.DefaultApiConventions?displayProperty=fullName>. Le convenzioni sono basate sul controller (*ValuesController.cs*) disponibile nel modello di progetto **API** di ASP.NET Core. Se si seguono i criteri del modello, le convenzioni predefinite danno i risultati previsti. Se tuttavia le convenzioni predefinite non soddisfano le proprie esigenze, vedere [Creare convenzioni dell'API Web](#create-web-api-conventions).

In fase di runtime, <xref:Microsoft.AspNetCore.Mvc.ApiExplorer> riconosce le convenzioni. `ApiExplorer` è l'astrazione MVC per comunicare con i generatori di documenti [OpenAPI](https://www.openapis.org/) (noti anche come Swagger). Gli attributi della convenzione applicata vengono associati a un'azione e sono inclusi nella documentazione OpenAPI dell'azione. Anche gli [analizzatori di API](xref:web-api/advanced/analyzers) supportano le convenzioni. Se l'azione non è convenzionale (ad esempio, se restituisce un codice di stato non documentato dalla convenzione applicata), un avviso richiede di documentare il codice di stato.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/conventions/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="apply-web-api-conventions"></a>Applicare le convenzioni dell'API Web

Le convenzioni non sono componibili; ogni azione può essere associata a una sola convenzione. Le convenzioni più specifiche hanno la precedenza su quelle meno specifiche. Quando si applicano a un'azione due o più convenzioni con la stessa priorità, la selezione è non deterministica. Per applicare una convenzione a un'azione sono disponibili le opzioni seguenti, dalla più specifica alla meno specifica:

1. `Microsoft.AspNetCore.Mvc.ApiConventionMethodAttribute`&mdash;Si applica a singole azioni e specifica il tipo di convenzione e il metodo di convenzione applicabile.

    Nell'esempio seguente, il metodo della convenzione `Microsoft.AspNetCore.Mvc.DefaultApiConventions.Put` del tipo di convenzione predefinito viene applicato all'azione `Update`:

    [!code-csharp[](conventions/sample/Controllers/ContactsConventionController.cs?name=snippet_ApiConventionMethod&highlight=3)]

    Il metodo della convenzione `Microsoft.AspNetCore.Mvc.DefaultApiConventions.Put` applica i seguenti attributi all'azione:

    ```csharp
    [ProducesDefaultResponseType]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    ```

    Per altre informazioni su `[ProducesDefaultResponseType]`, vedere [Default Response](https://swagger.io/docs/specification/describing-responses/#default) (Risposta predefinita).

1. `Microsoft.AspNetCore.Mvc.ApiConventionTypeAttribute` applicato a un controller &mdash; Applica il tipo di convenzione specificato a tutte le azioni del controller. Un metodo di convenzione è contrassegnato con hint che determinano le azioni a cui si applica il metodo di convenzione. Per altre informazioni sugli hint, vedere [Creare convenzioni dell'API Web](#create-web-api-conventions).

    Nell'esempio seguente il set di convenzioni predefinito viene applicato a tutte le azioni in *ContactsConventionController*:

    [!code-csharp[](conventions/sample/Controllers/ContactsConventionController.cs?name=snippet_ApiConventionTypeAttribute&highlight=2)]

1. `Microsoft.AspNetCore.Mvc.ApiConventionTypeAttribute` applicato a un assembly &mdash; Applica il tipo di convenzione specificato a tutti i controller dell'assembly corrente. È consigliabile applicare attributi a livello di assembly nel file *Startup.cs*.

    Nell'esempio seguente il set di convenzioni predefinito viene applicato a tutti i controller dell'assembly:

    [!code-csharp[](conventions/sample/Startup.cs?name=snippet_ApiConventionTypeAttribute&highlight=1)]

## <a name="create-web-api-conventions"></a>Creare convenzioni dell'API Web

Se le convenzioni dell'API predefinite non soddisfano le proprie esigenze, creare convenzioni personalizzate. Una convenzione è:

* un tipo statico con metodi.
* in grado di definire [tipi di risposta](#response-types) e [requisiti di denominazione](#naming-requirements) per le azioni.

### <a name="response-types"></a>Tipi di risposta

Questi metodi vengono annotati con attributi `[ProducesResponseType]` o `[ProducesDefaultResponseType]`. Ad esempio:

```csharp
public static class MyAppConventions
{
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public static void Find(int id)
    {
    }
}
```

Se non sono presenti attributi dei metadati più specifici, l'applicazione di questa convenzione a un assembly impone quanto segue:

* Il metodo della convenzione viene applicato a qualsiasi azione con nome `Find`.
* Un parametro con nome `id` è presente nell'azione `Find`.

### <a name="naming-requirements"></a>Requisiti di denominazione

Gli attributi `[ApiConventionNameMatch]` e `[ApiConventionTypeMatch]` possono essere applicati al metodo convenzione che determina le azioni nelle quali vengono implementati. Ad esempio:

```csharp
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ApiConventionNameMatch(ApiConventionNameMatchBehavior.Prefix)]
public static void Find(
    [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
    int id)
{ }
```

Nell'esempio precedente:

* L'opzione `Microsoft.AspNetCore.Mvc.ApiExplorer.ApiConventionNameMatchBehavior.Prefix` applicata al metodo indica che la convenzione corrisponde a qualsiasi azione con il prefisso "Find". `Find`, `FindPet` e `FindById` sono esempi di azioni corrispondenti.
* `Microsoft.AspNetCore.Mvc.ApiExplorer.ApiConventionNameMatchBehavior.Suffix` applicato al parametro indica che la convenzione corrisponde a metodi che hanno esattamente un parametro che termina con l'identificatore del suffisso. Sono esempi parametri come `id` o `petId`. `ApiConventionTypeMatch` può essere applicato in modo analogo ai tipi, per vincolare il tipo del parametro. Un argomento `params[]` indica i parametri rimanenti, per i quali non è necessario che sia rilevata una corrispondenza esplicita.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:web-api/advanced/analyzers>
* <xref:tutorials/web-api-help-pages-using-swagger>
