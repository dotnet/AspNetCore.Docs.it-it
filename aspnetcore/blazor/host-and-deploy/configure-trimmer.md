---
title: Configurare il trimmer per ASP.NET Core Blazor
author: guardrex
description: Informazioni su come controllare il linker linguaggio intermedio (IL) (trimmer) quando si compila un' Blazor app.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 02/08/2021
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
uid: blazor/host-and-deploy/configure-trimmer
ms.openlocfilehash: 41887638f13a08d375075e8377da19d1d0098c4b
ms.sourcegitcommit: ef8d8c79993a6608bf597ad036edcf30b231843f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975215"
---
# <a name="configure-the-trimmer-for-aspnet-core-blazor"></a>Configurare il trimmer per ASP.NET Core Blazor

Blazor WebAssembly esegue il taglio del [linguaggio intermedio (il)](/dotnet/standard/managed-code#intermediate-language--execution) per ridurre le dimensioni dell'output pubblicato. Per impostazione predefinita, il trimming si verifica quando si pubblica un'app.

Il trimming può avere effetti negativi. Nelle app che usano la reflection il trimmer spesso non può determinare i tipi richiesti per la reflection in fase di esecuzione. Per tagliare le app che usano la reflection, il trimmer deve essere informato sui tipi necessari per la reflection nel codice dell'app e nei pacchetti o nei Framework da cui dipende l'app. Il trimmer non è inoltre in grado di reagire al comportamento dinamico di un'app in fase di esecuzione. Per assicurarsi che l'app tagliata funzioni correttamente una volta distribuita, testare spesso l'output pubblicato durante lo sviluppo.

Per configurare il trimmer, vedere l'articolo relativo alle [Opzioni di taglio](/dotnet/core/deploying/trimming-options) nella documentazione dei concetti di base di .NET, che include indicazioni sugli argomenti seguenti:

* Disabilitare il trimming per l'intera app con la `<PublishTrimmed>` proprietà nel file di progetto.
* Consente di controllare il modo in cui il ritaglio inutilizzato viene ignorato dal trimmer.
* Arrestare il trimmer tagliando assembly specifici.
* Assembly "radice" per il trimming.
* Avvisi di superficie per i tipi riflessi impostando la `<SuppressTrimAnalysisWarnings>` proprietà su `false` nel file di progetto.
* Controllare il taglio del simbolo e il supporto di degugger.
* Impostare le funzionalità trimmer per la rimozione delle funzionalità della libreria del Framework.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Trimming delle distribuzioni autonome e degli eseguibili](/dotnet/core/deploying/trim-self-contained)
* <xref:blazor/webassembly-performance-best-practices#intermediate-language-il-trimming>
