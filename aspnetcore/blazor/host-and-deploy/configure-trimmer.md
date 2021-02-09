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
# <a name="configure-the-trimmer-for-aspnet-core-blazor"></a><span data-ttu-id="02a4f-103">Configurare il trimmer per ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="02a4f-103">Configure the Trimmer for ASP.NET Core Blazor</span></span>

<span data-ttu-id="02a4f-104">Blazor WebAssembly esegue il taglio del [linguaggio intermedio (il)](/dotnet/standard/managed-code#intermediate-language--execution) per ridurre le dimensioni dell'output pubblicato.</span><span class="sxs-lookup"><span data-stu-id="02a4f-104">Blazor WebAssembly performs [Intermediate Language (IL)](/dotnet/standard/managed-code#intermediate-language--execution) trimming to reduce the size of the published output.</span></span> <span data-ttu-id="02a4f-105">Per impostazione predefinita, il trimming si verifica quando si pubblica un'app.</span><span class="sxs-lookup"><span data-stu-id="02a4f-105">By default, trimming occurs when publishing an app.</span></span>

<span data-ttu-id="02a4f-106">Il trimming può avere effetti negativi.</span><span class="sxs-lookup"><span data-stu-id="02a4f-106">Trimming may have detrimental effects.</span></span> <span data-ttu-id="02a4f-107">Nelle app che usano la reflection il trimmer spesso non può determinare i tipi richiesti per la reflection in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="02a4f-107">In apps that use reflection, the Trimmer often can't determine the required types for reflection at runtime.</span></span> <span data-ttu-id="02a4f-108">Per tagliare le app che usano la reflection, il trimmer deve essere informato sui tipi necessari per la reflection nel codice dell'app e nei pacchetti o nei Framework da cui dipende l'app.</span><span class="sxs-lookup"><span data-stu-id="02a4f-108">To trim apps that use reflection, the Trimmer must be informed about required types for reflection in both the app's code and in the packages or frameworks that the app depends on.</span></span> <span data-ttu-id="02a4f-109">Il trimmer non è inoltre in grado di reagire al comportamento dinamico di un'app in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="02a4f-109">The Trimmer is also unable to react to an app's dynamic behavior at runtime.</span></span> <span data-ttu-id="02a4f-110">Per assicurarsi che l'app tagliata funzioni correttamente una volta distribuita, testare spesso l'output pubblicato durante lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="02a4f-110">To ensure the trimmed app works correctly once deployed, test published output frequently while developing.</span></span>

<span data-ttu-id="02a4f-111">Per configurare il trimmer, vedere l'articolo relativo alle [Opzioni di taglio](/dotnet/core/deploying/trimming-options) nella documentazione dei concetti di base di .NET, che include indicazioni sugli argomenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="02a4f-111">To configure the Trimmer, see the [Trimming options](/dotnet/core/deploying/trimming-options) article in the .NET Fundamentals documentation, which includes guidance on the following subjects:</span></span>

* <span data-ttu-id="02a4f-112">Disabilitare il trimming per l'intera app con la `<PublishTrimmed>` proprietà nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="02a4f-112">Disable trimming for the entire app with the `<PublishTrimmed>` property in the project file.</span></span>
* <span data-ttu-id="02a4f-113">Consente di controllare il modo in cui il ritaglio inutilizzato viene ignorato dal trimmer.</span><span class="sxs-lookup"><span data-stu-id="02a4f-113">Control how aggressively unused IL is discarded by the Trimmer.</span></span>
* <span data-ttu-id="02a4f-114">Arrestare il trimmer tagliando assembly specifici.</span><span class="sxs-lookup"><span data-stu-id="02a4f-114">Stop the Trimmer from trimming specific assemblies.</span></span>
* <span data-ttu-id="02a4f-115">Assembly "radice" per il trimming.</span><span class="sxs-lookup"><span data-stu-id="02a4f-115">"Root" assemblies for trimming.</span></span>
* <span data-ttu-id="02a4f-116">Avvisi di superficie per i tipi riflessi impostando la `<SuppressTrimAnalysisWarnings>` proprietà su `false` nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="02a4f-116">Surface warnings for reflected types by setting the `<SuppressTrimAnalysisWarnings>` property to `false` in the project file.</span></span>
* <span data-ttu-id="02a4f-117">Controllare il taglio del simbolo e il supporto di degugger.</span><span class="sxs-lookup"><span data-stu-id="02a4f-117">Control symbol trimming and degugger support.</span></span>
* <span data-ttu-id="02a4f-118">Impostare le funzionalità trimmer per la rimozione delle funzionalità della libreria del Framework.</span><span class="sxs-lookup"><span data-stu-id="02a4f-118">Set Trimmer features for trimming framework library features.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="02a4f-119">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="02a4f-119">Additional resources</span></span>

* [<span data-ttu-id="02a4f-120">Trimming delle distribuzioni autonome e degli eseguibili</span><span class="sxs-lookup"><span data-stu-id="02a4f-120">Trim self-contained deployments and executables</span></span>](/dotnet/core/deploying/trim-self-contained)
* <xref:blazor/webassembly-performance-best-practices#intermediate-language-il-trimming>
