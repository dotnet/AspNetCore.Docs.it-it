---
title: ASP.NET Core e Entity Framework 6
author: rick-anderson
description: Entity Framework 6,3 e versioni successive funzionano con ASP.NET Core 3,1 e versioni successive.
ms.author: riande
ms.custom: mvc
ms.date: 7/14/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: data/entity-framework-6
ms.openlocfilehash: 44211ac7fa2acc7a7a9471ef362cff02f94fa2b6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588269"
---
# <a name="aspnet-core-and-entity-framework-6"></a>ASP.NET Core e Entity Framework 6
::: moniker range=">= aspnetcore-3.0"

Di [Patrick Goode](https://github.com/attrib75)

## <a name="using-entity-framework-6-with-aspnet-core"></a>Uso di Entity Framework 6 con ASP.NET Core

Per lo sviluppo è necessario usare [Entity Framework Core](/ef/) . L' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/3.xsample) USA [Entity Framework 6 (EF6)](/ef/ef6), che può essere usato per eseguire la migrazione di app in uscita da ASP.NET Core.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Entity Framework - configurazione basata su codice](/ef/ef6/fundamentals/configuring/code-based)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Paweł Grudzień](https://github.com/pgrudzien12), [Damien Pontifex](https://github.com/DamienPontifex) e [Tom Dykstra](https://github.com/tdykstra)

In questo articolo viene illustrato come usare Entity Framework 6 in un'applicazione ASP.NET Core.    

## <a name="overview"></a>Panoramica 

Per l'uso di Entity Framework 6, il progetto deve eseguire la compilazione con .NET Framework, poiché Entity Framework 6 non supporta .NET Core. Se sono richieste funzionalità multipiattaforma, è necessario eseguire l'aggiornamento a [Entity Framework Core](/ef/).  

Il modo consigliato per usare Entity Framework 6 in un'applicazione ASP.NET Core consiste nell'inserire il contesto EF6 e le classi del modello in un progetto di libreria di classi destinato .NET Framework. Aggiungere un riferimento alla libreria di classi dal progetto ASP.NET Core. Vedere l'esempio [soluzione di Visual Studio con progetti EF6 e ASP.NET Core](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/sample/).    

Non è possibile inserire un contesto EF6 in un progetto ASP.NET poiché i progetti .NET Core non supportano tutte le funzionalità che i comandi EF6, come *Enable-Migrations*, richiedono.    

Indipendentemente dal tipo di progetto in cui si trova il contesto EF6, solo gli strumenti da riga di comando EF6 funzionano con un contesto EF6. Ad esempio, `Scaffold-DbContext` è disponibile solo in Entity Framework Core. Se è necessario eseguire reverse engineering di un database in un modello EF6, vedere <https://docs.microsoft.com/ef/ef6/modeling/code-first/workflows/existing-database> .    

## <a name="reference-full-framework-and-ef6-in-the-aspnet-core-project"></a>Framework completo di riferimento e EF6 nel progetto ASP.NET Core 

Il progetto ASP.NET Core deve avere come destinazione .NET Framework e fare riferimento a EF6. Ad esempio, il file *csproj* del progetto ASP.NET Core avrà un aspetto simile a quanto segue (sono visualizzate solo le parti pertinenti del file).    

[!code-xml[](entity-framework-6/sample/MVCCore/MVCCore.csproj?range=3-9&highlight=2)]   

Quando si crea un nuovo progetto, usare il modello **Applicazione Web di ASP.NET Core (.NET Framework)**.    

## <a name="handle-connection-strings"></a>Gestire le stringhe di connessione    

Gli strumenti da riga di comando EF6 che vengono usati nel progetto della libreria di classi EF6 richiedono un costruttore predefinito, così da poter creare un'istanza del contesto. Tuttavia è opportuno specificare la stringa di connessione da usare nel progetto ASP.NET Core. In questo caso il costruttore di contesto deve avere un parametro che consente di passare alla stringa di connessione. Ecco un esempio.   

[!code-csharp[](entity-framework-6/sample/EF6/SchoolContext.cs?name=snippet_Constructor)]   

Poiché il contesto EF6 non ha un costruttore senza parametri, il progetto EF6 deve fornire un'implementazione di <https://docs.microsoft.com/dotnet/api/system.data.entity.infrastructure.idbcontextfactory-1?view=entity-framework-6.2.0> . Gli strumenti da riga di comando EF6 trovano e usano detta implementazione, così da poter creare un'istanza del contesto. Ecco un esempio.   

[!code-csharp[](entity-framework-6/sample/EF6/SchoolContextFactory.cs?name=snippet_IDbContextFactory)]  

In questo codice di esempio, l'implementazione `IDbContextFactory` passa in una stringa di connessione hardcoded. Questa è la stringa di connessione che gli strumenti da riga di comando useranno. È opportuno implementare una strategia per garantire che la libreria di classi usi la stessa stringa di connessione usata dall'applicazione chiamante. Ad esempio, è possibile ottenere il valore dalla variabile di un ambiente in entrambi i progetti.   

## <a name="set-up-dependency-injection-in-the-aspnet-core-project"></a>Impostare l'inserimento di dipendenze nel progetto ASP.NET Core  

Nel file *Startup.cs* del progetto Core, impostare il contesto di EF6 per l'inserimento di dipendenze in `ConfigureServices`. Gli oggetti di contesto di Entity Framework devono essere limitati a una durata per richiesta.   

[!code-csharp[](entity-framework-6/sample/MVCCore/Startup.cs?name=snippet_ConfigureServices&highlight=5)]   

È quindi possibile ottenere un'istanza del contesto nei controller usando l'inserimento di dipendenze. Il codice è simile a quanto si scriverebbe per un contesto EF Core:    

[!code-csharp[](entity-framework-6/sample/MVCCore/Controllers/StudentsController.cs?name=snippet_ContextInController)]  

## <a name="sample-application"></a>Applicazione di esempio   

Per un'applicazione di esempio funzionante, vedere la [soluzione di Visual Studio di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/sample/) che accompagna questo articolo.    

Questo esempio può essere creato da zero tramite i passaggi seguenti in Visual Studio:    

* Creare una soluzione.    

* **Aggiungi** > **Nuovo progetto** > **Web** > **Applicazione Web ASP.NET Core**    
  * Nella finestra di dialogo di selezione del modello di progetto selezionare API e .NET Framework nell'elenco a discesa 

* **Aggiungi** > **Nuovo progetto** > **Windows Desktop** > **Libreria di classi (.NET Framework)**  

* Nella **Console di Gestione pacchetti** eseguire il comando per entrambi i progetti`Install-Package Entityframework`.    

* Nel progetto di libreria di classi, creare le classi di modelli di dati e una classe di contesto e un'implementazione di `IDbContextFactory`.    

* Nella Console di Gestione pacchetti del progetto di libreria di classi, eseguire i comandi `Enable-Migrations` e `Add-Migration Initial`. Se il progetto ASP.NET Core è stato impostato come progetto di avvio, aggiungere `-StartupProjectName EF6` a questi comandi. 

* Nel progetto Core aggiungere un riferimento al progetto di libreria di classi.    

* Nel progetto Core, in *Startup.cs*, registrare il contesto per l'inserimento delle dipendenze.    

* Nel progetto principale, in *appsettings.json* , aggiungere la stringa di connessione.  

* Nel progetto Core, aggiungere un controller e le visualizzazioni per verificare che è possibile leggere e scrivere i dati. Si noti che lo scaffolding di ASP.NET MVC di base non funzionerà con il contesto EF6 a cui viene fatto riferimento dalla libreria di classi.

::: moniker-end
