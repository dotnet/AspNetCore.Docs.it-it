---
title: Introduzione a ASP.NET Core
author: rick-anderson
description: Introduzione a ASP.NET Core, un Framework multipiattaforma, ad alte prestazioni, open source per la compilazione di app moderne, abilitate per il cloud e connesse a Internet.
ms.author: riande
ms.custom: mvc
ms.date: 04/17/2020
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
uid: index
ms.openlocfilehash: 3e41336d084e25319f8b1ab4c4ab3175b758d23d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588802"
---
# <a name="introduction-to-aspnet-core"></a>Introduzione a ASP.NET Core

Di [Daniel Roth](https://github.com/danroth27), [Rick Anderson](https://twitter.com/RickAndMSFT), e [Shaun Luttin](https://twitter.com/dicshaunary)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core è un Framework multipiattaforma, ad alte prestazioni, [Open Source](https://github.com/dotnet/aspnetcore) per la compilazione di app moderne, abilitate per il cloud e connesse a Internet. Con ASP.NET Core, è possibile:

* Creazione di servizi e app Web, [Internet delle cose](https://www.microsoft.com/internet-of-things/) app e backend per dispositivi mobili.
* Usare gli strumenti di sviluppo preferiti in Windows, macOS e Linux.
* Distribuire nel cloud o in locale.
* Eseguire in [.NET Core](/dotnet/core/introduction).

## <a name="why-choose-aspnet-core"></a>Perché scegliere ASP.NET Core?

Milioni di sviluppatori usano o hanno usato [ASP.NET 4. x](/aspnet/overview) per creare app Web. ASP.NET Core è una riprogettazione di ASP.NET 4. x, incluse le modifiche architetturali che hanno come risultato un Framework più snello e modulare.

[!INCLUDE[](~/includes/benefits.md)]

## <a name="build-web-apis-and-web-ui-using-aspnet-core-mvc"></a>Compilare API web e interfaccia utente web tramite ASP.NET Core MVC

ASP.NET Core MVC offre funzionalità per la compilazione di [API Web](xref:tutorials/first-web-api) e [app Web](xref:tutorials/razor-pages/index):

* Il [Model-View-Controller (MVC)](xref:mvc/overview) consente di rendere le API web e le app web testabili.
* [ Razor Pages](xref:razor-pages/index) è un modello di programmazione basato su pagine che rende la creazione di un'interfaccia utente Web più semplice e produttiva.
* il [ Razor markup](xref:mvc/views/razor) fornisce una sintassi produttiva per le [ Razor pagine](xref:razor-pages/index) e le [visualizzazioni MVC](xref:mvc/views/overview).
* Gli [Helper Tag](xref:mvc/views/tag-helpers/intro) consentono al codice lato server di partecipare alla creazione e al rendering di elementi HTML nei Razor file.
* Il supporto incorporato per [più formati di dati e per la negoziazione del contenuto](xref:web-api/advanced/formatting) consente alle API web di raggiungere una vasta gamma di client, inclusi i browser e i dispositivi mobili.
* L'[associazione di modelli](xref:mvc/models/model-binding) esegue automaticamente il mapping dei dati dalle richieste HTTP ai parametri del metodo di azione.
* La [convalida dei modelli](xref:mvc/models/validation) esegue automaticamente la convalida sul lato server e sul lato client.

## <a name="client-side-development"></a>Sviluppo lato client

ASP.NET Core si integra facilmente con le librerie e i framework lato client più diffusi, tra cui [Blazor](xref:blazor/index) , [angolare](xref:spa/angular), [reagire](xref:spa/react)e [bootstrap](https://getbootstrap.com/). Per altre informazioni, vedere <xref:blazor/index> e gli argomenti correlati in *Sviluppo lato client*.

<a name="target-framework"></a>

## <a name="aspnet-core-target-frameworks"></a>Framework di destinazione ASP.NET Core

ASP.NET Core 3. x e versioni successive possono essere destinati solo a .NET Core. In genere, ASP.NET Core è costituito da librerie di [.NET standard](/dotnet/standard/net-standard) . Le librerie scritte con .NET Standard 2.0 supportano l'esecuzione su [qualsiasi piattaforma .NET che implementa .NET Standard 2.0](/dotnet/standard/net-standard#net-implementation-support).

Usare .NET Core come destinazione offre diversi vantaggi, che aumentano con ogni versione. Alcuni vantaggi di .NET Core in .NET Framework sono:

* Funzionamento multipiattaforma. Viene eseguito in Windows, macOS e Linux.
* prestazioni migliorate
* [Controllo delle versioni affiancato](/dotnet/standard/choosing-core-framework-server#side-by-side-net-versions-per-application-level)
* Nuove API
* Open source

## <a name="recommended-learning-path"></a>Percorso di apprendimento consigliato

Per un'introduzione allo sviluppo di app ASP.NET Core, è consigliabile usare la sequenza di esercitazioni seguente:

1. Seguire un'esercitazione per il tipo di app che si vuole sviluppare o gestire.

   |Tipo di app  |Scenario  |Esercitazione  |
   |----------|----------|----------|
   |app Web                   | Nuovo sviluppo di interfaccia utente Web sul lato server |[Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) |
   |app Web                   | Gestione di un'app MVC |[Introduzione a MVC](xref:tutorials/first-mvc-app/start-mvc)|
   |app Web                   | Sviluppo di interfaccia utente Web lato client |[Introduzione a Blazor](https://dotnet.microsoft.com/learn/aspnet/blazor-tutorial/intro) |
   |API Web                   | Servizi HTTP RESTful |[Creare un'API Web](xref:tutorials/first-web-api)&dagger; |
   |App RPC (Remote Procedure Call) | Servizi di primo contratto che usano i buffer del protocollo |[Introduzione all'uso di un servizio gRPC](xref:tutorials/grpc/grpc-start) |
   |App in tempo reale             | Comunicazione bidirezionale tra i server e i client connessi |[Introduzione a SignalR](xref:tutorials/signalr) |

1. Seguire un'esercitazione che illustra come eseguire l'accesso ai dati di base.

   |Scenario  |Esercitazione  |
   |----------|----------|
   |Nuovo sviluppo        |[Razor Pagine con Entity Framework Core](xref:data/ef-rp/intro) |
   |Gestione di un'app MVC |[MVC con Entity Framework Core](xref:data/ef-mvc/intro) |

1. Leggi una panoramica dei [concetti di base](xref:fundamentals/index) ASP.NET Core che si applicano a tutti i tipi di app.

1. Esplorare il sommario per altri argomenti di interesse.

&dagger;È disponibile anche un' [esercitazione sull'API Web interattiva](/learn/modules/build-web-api-net-core). Non è necessaria alcuna installazione locale degli strumenti di sviluppo. Il codice viene eseguito in un [Azure cloud Shell](https://azure.microsoft.com/features/cloud-shell/) nel browser e [curl](https://curl.haxx.se/) viene usato per i test.

## <a name="migrate-from-net-framework"></a>Eseguire la migrazione da .NET Framework

Per una guida di riferimento alla migrazione delle app ASP.NET 4. x ai ASP.NET Core, vedere <xref:migration/proper-to-2x/index> .

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core è un Framework multipiattaforma, ad alte prestazioni, [Open Source](https://github.com/dotnet/aspnetcore) per la compilazione di app moderne, abilitate per il cloud e connesse a Internet. Con ASP.NET Core, è possibile:

* Creazione di servizi e app Web, [Internet delle cose](https://www.microsoft.com/internet-of-things/) app e backend per dispositivi mobili.
* Usare gli strumenti di sviluppo preferiti in Windows, macOS e Linux.
* Distribuire nel cloud o in locale.
* Eseguire in [.NET Core o .NET Framework](/dotnet/articles/standard/choosing-core-framework-server).

## <a name="why-choose-aspnet-core"></a>Perché scegliere ASP.NET Core?

Milioni di sviluppatori usano o hanno usato [ASP.NET 4. x](/aspnet/overview) per creare app Web. ASP.NET Core è una riprogettazione di ASP.NET 4.x, con modifiche a livello di architettura che comportano un framework più efficiente e modulare.

[!INCLUDE[](~/includes/benefits.md)]

## <a name="build-web-apis-and-web-ui-using-aspnet-core-mvc"></a>Compilare API web e interfaccia utente web tramite ASP.NET Core MVC

ASP.NET Core MVC offre funzionalità per la compilazione di [API Web](xref:tutorials/first-web-api) e [app Web](xref:tutorials/razor-pages/index):

* Il [Model-View-Controller (MVC)](xref:mvc/overview) consente di rendere le API web e le app web testabili.
* [ Razor Pages](xref:razor-pages/index) è un modello di programmazione basato su pagine che rende la creazione di un'interfaccia utente Web più semplice e produttiva.
* il [ Razor markup](xref:mvc/views/razor) fornisce una sintassi produttiva per le [ Razor pagine](xref:razor-pages/index) e le [visualizzazioni MVC](xref:mvc/views/overview).
* Gli [Helper Tag](xref:mvc/views/tag-helpers/intro) consentono al codice lato server di partecipare alla creazione e al rendering di elementi HTML nei Razor file.
* Il supporto incorporato per [più formati di dati e per la negoziazione del contenuto](xref:web-api/advanced/formatting) consente alle API web di raggiungere una vasta gamma di client, inclusi i browser e i dispositivi mobili.
* L'[associazione di modelli](xref:mvc/models/model-binding) esegue automaticamente il mapping dei dati dalle richieste HTTP ai parametri del metodo di azione.
* La [convalida dei modelli](xref:mvc/models/validation) esegue automaticamente la convalida sul lato server e sul lato client.

## <a name="client-side-development"></a>Sviluppo lato client

ASP.NET Core si integra facilmente con le librerie e i framework lato client più diffusi, tra cui [Blazor](xref:blazor/index) , [angolare](xref:spa/angular), [reagire](xref:spa/react)e [bootstrap](https://getbootstrap.com/). Per altre informazioni, vedere <xref:blazor/index> e gli argomenti correlati in *Sviluppo lato client*.

<a name="target-framework"></a>

## <a name="aspnet-core-targeting-net-framework"></a>ASP.NET Core per .NET Framework

ASP.NET Core 2.x può avere come destinazione .NET Core o .NET Framework. Le app ASP.NET Core destinate a .NET Framework non sono multipiattaforma, ma funzionano solo in Windows. ASP.NET Core 2.x è costituito a livello generale da librerie [.NET Standard](/dotnet/standard/net-standard). Le librerie scritte con .NET Standard 2.0 supportano l'esecuzione su [qualsiasi piattaforma .NET che implementa .NET Standard 2.0](/dotnet/standard/net-standard#net-implementation-support).

ASP.NET Core 2.x è supportato nelle versioni di .NET Framework che implementano .NET Standard 2.0:

* È consigliabile .NET Framework versione più recente.
* .NET Framework 4.6.1 e versioni successive.

L'esecuzione di ASP.NET Core 3.0 e versioni successive sarà consentita solo in .NET Core. Per altri dettagli riguardanti questa modifica, vedere [A first look at changes coming in ASP.NET Core 3.0](https://blogs.msdn.microsoft.com/webdev/2018/10/29/a-first-look-at-changes-coming-in-asp-net-core-3-0/) (Una prima occhiata alle modifiche previste per ASP.NET Core 3.0).

Usare .NET Core come destinazione offre diversi vantaggi, che aumentano con ogni versione. Alcuni vantaggi di .NET Core in .NET Framework sono:

* Funzionamento multipiattaforma. Esecuzione con macOS, Linux e Windows.
* prestazioni migliorate
* [Controllo delle versioni affiancato](/dotnet/standard/choosing-core-framework-server#side-by-side-net-versions-per-application-level)
* Nuove API
* Open source

Per semplificare la divariazione dell'API da .NET Framework a .NET Core, [Windows Compatibility Pack](/dotnet/core/porting/windows-compat-pack) ha reso disponibili migliaia di API solo per Windows disponibili in .NET Core. Queste API non erano disponibili in .NET Core 1. x.

## <a name="recommended-learning-path"></a>Percorso di apprendimento consigliato

Per un'introduzione allo sviluppo delle app ASP.NET Core, è consigliabile eseguire la sequenza di esercitazioni e articoli seguente:

1. Seguire un'esercitazione per il tipo di app che si vuole sviluppare o gestire.

   |Tipo di app  |Scenario  |Esercitazione  |
   |----------|----------|----------|
   |app Web                   | Per un nuovo sviluppo        |[Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) |
   |app Web                   | Per gestire un'app MVC |[Introduzione a MVC](xref:tutorials/first-mvc-app/start-mvc)|
   |API Web                   |                            |[Creare un'API Web](xref:tutorials/first-web-api)&dagger; |
   |App in tempo reale             |                            |[Introduzione a SignalR](xref:tutorials/signalr) |

1. Seguire un'esercitazione che illustra come eseguire l'accesso ai dati di base.

   |Scenario  |Esercitazione  |
   |----------|----------|
   | Per un nuovo sviluppo        |[Razor Pagine con Entity Framework Core](xref:data/ef-rp/intro) |
   | Per gestire un'app MVC |[MVC con Entity Framework Core](xref:data/ef-mvc/intro) |

1. Leggi una panoramica dei [concetti di base](xref:fundamentals/index) ASP.NET Core che si applicano a tutti i tipi di app.

1. Esplorare il Sommario per cercare altri argomenti di interesse.

&dagger;È anche disponibile un' [esercitazione sull'API Web che segue completamente nel browser](/learn/modules/build-web-api-net-core), non è necessaria alcuna installazione dell'IDE locale. Il codice viene eseguito in un'[Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) e per il testing viene usato [curl](https://curl.haxx.se/).

## <a name="migrate-from-net-framework"></a>Eseguire la migrazione da .NET Framework

Per una guida di riferimento alla migrazione delle app ASP.NET ai ASP.NET Core, vedere <xref:migration/proper-to-2x/index> .

::: moniker-end

## <a name="how-to-download-a-sample"></a>Come scaricare un esempio

Molti articoli ed esercitazioni includono collegamenti al codice di esempio.

1. [Scaricare il file ZIP del repository ASP.NET](https://codeload.github.com/dotnet/AspNetCore.Docs/zip/main).
1. Decomprimere il file di *Docs-main.zip* .
1. Usare l'URL nel collegamento di esempio per passare alla directory di esempio.

### <a name="preprocessor-directives-in-sample-code"></a>Direttive per il preprocessore nel codice di esempio

Per illustrare più scenari, le app di esempio usano le `#define` `#if-#else/#elif-#endif` direttive per il preprocessore e per compilare ed eseguire in modo selettivo sezioni diverse del codice di esempio. Per gli esempi che usano questo approccio, impostare la `#define` direttiva all'inizio dei file C# per definire il simbolo associato allo scenario che si vuole eseguire. Alcuni esempi richiedono la definizione del simbolo nella parte superiore di più file per eseguire uno scenario.

Ad esempio, l'elenco di simboli `#define` seguente indica che sono disponibili quattro scenari, ovvero uno scenario per simbolo. La configurazione di esempio corrente esegue lo scenario `TemplateCode`:

```csharp
#define TemplateCode // or LogFromMain or ExpandDefault or FilterInCode
```

Per modificare l'esempio in modo che venga eseguito lo scenario `ExpandDefault`, definire il simbolo `ExpandDefault` e lasciare i simboli rimanenti con commento:

```csharp
#define ExpandDefault // TemplateCode or LogFromMain or FilterInCode
```

Per altre informazioni sull'uso delle [direttive del preprocessore C#](/dotnet/csharp/language-reference/preprocessor-directives/) per compilare in modo selettivo le sezioni di codice, vedere [#define (Riferimenti per C#)](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-define) e [#if (Riferimenti per C#)](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if).

### <a name="regions-in-sample-code"></a>Aree del codice di esempio

Alcune app di esempio contengono sezioni di codice racchiuse tra [#region](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-region) e [#endregion](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-endregion) direttive C#. Il sistema di compilazione della documentazione inserisce queste aree negli argomenti della documentazione visualizzabile.  

I nomi delle aree in genere contengono la parola "frammento". L'esempio seguente mostra un'area denominata `snippet_WebHostDefaults`:

```csharp
#region snippet_WebHostDefaults
Host.CreateDefaultBuilder(args)
    .ConfigureWebHostDefaults(webBuilder =>
    {
        webBuilder.UseStartup<Startup>();
    });
#endregion
```

Il file markdown dell'argomento fa riferimento al frammento di codice C# precedente con la riga seguente:

```md
[!code-csharp[](sample/SampleApp/Program.cs?name=snippet_WebHostDefaults)]
```

È possibile ignorare o rimuovere in modo sicuro le `#region` `#endregion` direttive e che racchiudono il codice. Non modificare il codice all'interno di queste direttive se si prevede di eseguire gli scenari di esempio descritti nell'argomento. È possibile modificare il codice durante la sperimentazione con altri scenari.

Per altre informazioni, vedere [Contribuire alla documentazione ASP.NET: Frammenti di codice](https://github.com/dotnet/AspNetCore.Docs/blob/main/CONTRIBUTING.md#code-snippets).

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni, vedere le seguenti risorse:

* <xref:getting-started>
* <xref:tutorials/publish-to-azure-webapp-using-vs>
* [Nozioni fondamentali su ASP.NET Core](xref:fundamentals/index)
* [La trasmissione settimanale della community di ASP.NET](https://live.asp.net/) offre una presentazione dei progressi e dei piani del team. Vengono segnalati nuovi blog e software di terze parti.
