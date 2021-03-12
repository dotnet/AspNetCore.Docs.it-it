---
title: Novità di ASP.NET Core 2.0
author: rick-anderson
description: Informazioni sulle nuove funzionalità in ASP.NET Core 2.0.
ms.author: riande
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
uid: aspnetcore-2.0
ms.openlocfilehash: b7515bcd8b15199770a4245469d00d10da5566f8
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605686"
---
# <a name="whats-new-in-aspnet-core-20"></a><span data-ttu-id="37bed-103">Novità di ASP.NET Core 2.0</span><span class="sxs-lookup"><span data-stu-id="37bed-103">What's new in ASP.NET Core 2.0</span></span>

<span data-ttu-id="37bed-104">Questo articolo evidenzia le modifiche più significative apportate ad ASP.NET Core 2.0, con collegamenti alla relativa documentazione.</span><span class="sxs-lookup"><span data-stu-id="37bed-104">This article highlights the most significant changes in ASP.NET Core 2.0, with links to relevant documentation.</span></span>

## <a name="razor-pages"></a><span data-ttu-id="37bed-105">Razor Pagine</span><span class="sxs-lookup"><span data-stu-id="37bed-105">Razor Pages</span></span>

<span data-ttu-id="37bed-106">Razor Pages è una nuova funzionalità di ASP.NET Core MVC che rende più semplice e più produttivo gli scenari di codifica della pagina.</span><span class="sxs-lookup"><span data-stu-id="37bed-106">Razor Pages is a new feature of ASP.NET Core MVC that makes coding page-focused scenarios easier and more productive.</span></span>

<span data-ttu-id="37bed-107">Per altre informazioni, vedere l'introduzione e l'esercitazione:</span><span class="sxs-lookup"><span data-stu-id="37bed-107">For more information, see the introduction and tutorial:</span></span>

* [<span data-ttu-id="37bed-108">Introduzione alle Razor pagine</span><span class="sxs-lookup"><span data-stu-id="37bed-108">Introduction to Razor Pages</span></span>](xref:razor-pages/index)
* [<span data-ttu-id="37bed-109">Introduzione alle Razor pagine</span><span class="sxs-lookup"><span data-stu-id="37bed-109">Get started with Razor Pages</span></span>](xref:tutorials/razor-pages/razor-pages-start)

## <a name="aspnet-core-metapackage"></a><span data-ttu-id="37bed-110">Metapacchetto ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="37bed-110">ASP.NET Core metapackage</span></span>

<span data-ttu-id="37bed-111">Un nuovo metapacchetto ASP.NET Core include tutti i pacchetti creati e supportati dai team di ASP.NET Core ed Entity Framework Core, con le relative dipendenze interne e di terze parti.</span><span class="sxs-lookup"><span data-stu-id="37bed-111">A new ASP.NET Core metapackage includes all of the packages made and supported by the ASP.NET Core and Entity Framework Core teams, along with their internal and 3rd-party dependencies.</span></span> <span data-ttu-id="37bed-112">Non è più necessario scegliere le singole funzionalità di ASP.NET Core in base al pacchetto.</span><span class="sxs-lookup"><span data-stu-id="37bed-112">You no longer need to choose individual ASP.NET Core features by package.</span></span> <span data-ttu-id="37bed-113">Tutte le funzionalità sono incluse nel pacchetto [Microsoft.AspNetCore.All](https://www.nuget.org/packages/Microsoft.AspNetCore.All).</span><span class="sxs-lookup"><span data-stu-id="37bed-113">All features are included in the [Microsoft.AspNetCore.All](https://www.nuget.org/packages/Microsoft.AspNetCore.All) package.</span></span> <span data-ttu-id="37bed-114">I modelli predefiniti usano questo pacchetto.</span><span class="sxs-lookup"><span data-stu-id="37bed-114">The default templates use this package.</span></span>

<span data-ttu-id="37bed-115">Per altre informazioni, vedere [Microsoft.AspNetCore.All metapackage for ASP.NET Core 2.0](xref:fundamentals/metapackage) (Metapacchetto Microsoft.AspNetCore.All per ASP.NET Core 2.0).</span><span class="sxs-lookup"><span data-stu-id="37bed-115">For more information, see [Microsoft.AspNetCore.All metapackage for ASP.NET Core 2.0](xref:fundamentals/metapackage).</span></span>

## <a name="runtime-store"></a><span data-ttu-id="37bed-116">Archivio di runtime</span><span class="sxs-lookup"><span data-stu-id="37bed-116">Runtime Store</span></span>

<span data-ttu-id="37bed-117">Le applicazioni che usano il metapacchetto `Microsoft.AspNetCore.All` sfruttano automaticamente il nuovo archivio di runtime di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="37bed-117">Applications that use the `Microsoft.AspNetCore.All` metapackage automatically take advantage of the new .NET Core Runtime Store.</span></span> <span data-ttu-id="37bed-118">L'archivio contiene tutti gli asset di runtime necessari per eseguire le applicazioni ASP.NET Core 2.0.</span><span class="sxs-lookup"><span data-stu-id="37bed-118">The Store contains all the runtime assets needed to run ASP.NET Core 2.0 applications.</span></span> <span data-ttu-id="37bed-119">Quando si usa il metapacchetto `Microsoft.AspNetCore.All`, con l'applicazione non vengono distribuiti asset dai pacchetti NuGet di riferimento di ASP.NET Core poiché sono già presenti nel sistema di destinazione.</span><span class="sxs-lookup"><span data-stu-id="37bed-119">When you use the `Microsoft.AspNetCore.All` metapackage, no assets from the referenced ASP.NET Core NuGet packages are deployed with the application because they already reside on the target system.</span></span> <span data-ttu-id="37bed-120">Gli asset contenuti nell'archivio di runtime vengono anche precompilati per migliorare i tempi di avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="37bed-120">The assets in the Runtime Store are also precompiled to improve application startup time.</span></span>

<span data-ttu-id="37bed-121">Per altre informazioni, vedere l'articolo relativo all'[archivio dei pacchetti di runtime](/dotnet/core/deploying/runtime-store)</span><span class="sxs-lookup"><span data-stu-id="37bed-121">For more information, see [Runtime store](/dotnet/core/deploying/runtime-store)</span></span>

## <a name="net-standard-20"></a><span data-ttu-id="37bed-122">.NET Standard 2.0</span><span class="sxs-lookup"><span data-stu-id="37bed-122">.NET Standard 2.0</span></span>

<span data-ttu-id="37bed-123">I pacchetti di ASP.NET Core 2.0 hanno come destinazione .NET Standard 2.0.</span><span class="sxs-lookup"><span data-stu-id="37bed-123">The ASP.NET Core 2.0 packages target .NET Standard 2.0.</span></span> <span data-ttu-id="37bed-124">Ai pacchetti possono fare riferimento altre librerie .NET Standard 2.0 ed è possibile eseguire i pacchetti in implementazioni di .NET compatibili con .NET Standard 2.0, tra cui .NET Core 2.0 e .NET Framework 4.6.1.</span><span class="sxs-lookup"><span data-stu-id="37bed-124">The packages can be referenced by other .NET Standard 2.0 libraries, and they can run on .NET Standard 2.0-compliant implementations of .NET, including .NET Core 2.0 and .NET Framework 4.6.1.</span></span> 

<span data-ttu-id="37bed-125">Il metapacchetto `Microsoft.AspNetCore.All` è riservato esclusivamente a .NET Core 2.0, poiché è destinato all'uso con l'archivio di runtime di .NET Core 2.0.</span><span class="sxs-lookup"><span data-stu-id="37bed-125">The `Microsoft.AspNetCore.All` metapackage targets .NET Core 2.0 only, because it's intended to be used with the .NET Core 2.0 Runtime Store.</span></span>

## <a name="configuration-update"></a><span data-ttu-id="37bed-126">Aggiornamento della configurazione</span><span class="sxs-lookup"><span data-stu-id="37bed-126">Configuration update</span></span>

<span data-ttu-id="37bed-127">In ASP.NET Core 2.0 viene aggiunta per impostazione predefinita un'istanza di `IConfiguration` al contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="37bed-127">An `IConfiguration` instance is added to the services container by default in ASP.NET Core 2.0.</span></span> <span data-ttu-id="37bed-128">`IConfiguration` nel contenitore dei servizi semplifica per le applicazioni il recupero dei valori di configurazione dal contenitore.</span><span class="sxs-lookup"><span data-stu-id="37bed-128">`IConfiguration` in the services container makes it easier for applications to retrieve configuration values from the container.</span></span>

<span data-ttu-id="37bed-129">Per informazioni sullo stato della documentazione prevista, vedere l'[argomento su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/3387).</span><span class="sxs-lookup"><span data-stu-id="37bed-129">For information about the status of planned documentation, see the [GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/3387).</span></span>

## <a name="logging-update"></a><span data-ttu-id="37bed-130">Aggiornamento della registrazione</span><span class="sxs-lookup"><span data-stu-id="37bed-130">Logging update</span></span>

<span data-ttu-id="37bed-131">In ASP.NET Core 2.0 la registrazione è incorporata nel sistema di inserimento delle dipendenze per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="37bed-131">In ASP.NET Core 2.0, logging is incorporated into the dependency injection (DI) system by default.</span></span> <span data-ttu-id="37bed-132">Si aggiungono i provider e si configurano i filtri nel file *Program.cs* anziché nel file *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="37bed-132">You add providers and configure filtering in the *Program.cs* file instead of in the *Startup.cs* file.</span></span> <span data-ttu-id="37bed-133">E l'oggetto `ILoggerFactory` predefinito supporta i filtri in modo tale da consentire l'uso di un unico approccio flessibile sia per il filtraggio tra provider, sia per il filtraggio di un provider specifico.</span><span class="sxs-lookup"><span data-stu-id="37bed-133">And the default `ILoggerFactory` supports filtering in a way that lets you use one flexible approach for both cross-provider filtering and specific-provider filtering.</span></span>

<span data-ttu-id="37bed-134">Per altre informazioni, vedere l'[introduzione alla registrazione in ASP.NET Core](xref:fundamentals/logging/index).</span><span class="sxs-lookup"><span data-stu-id="37bed-134">For more information, see [Introduction to Logging](xref:fundamentals/logging/index).</span></span>

## <a name="authentication-update"></a><span data-ttu-id="37bed-135">Aggiornamento dell'autenticazione</span><span class="sxs-lookup"><span data-stu-id="37bed-135">Authentication update</span></span>

<span data-ttu-id="37bed-136">Un nuovo modello di autenticazione rende più semplice configurare l'autenticazione per un'applicazione che usa l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="37bed-136">A new authentication model makes it easier to configure authentication for an application using DI.</span></span>

<span data-ttu-id="37bed-137">I nuovi modelli sono disponibili per la configurazione dell'autenticazione per le app Web e le API Web che usano [Azure AD B2C](https://azure.microsoft.com/services/active-directory-b2c/).</span><span class="sxs-lookup"><span data-stu-id="37bed-137">New templates are available for configuring authentication for web apps and web APIs using [Azure AD B2C](https://azure.microsoft.com/services/active-directory-b2c/).</span></span>

<span data-ttu-id="37bed-138">Per informazioni sullo stato della documentazione prevista, vedere l'[argomento su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/3054).</span><span class="sxs-lookup"><span data-stu-id="37bed-138">For information about the status of planned documentation, see the [GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/3054).</span></span>

## <a name="identity-update"></a><span data-ttu-id="37bed-139">Aggiornamento di Identity</span><span class="sxs-lookup"><span data-stu-id="37bed-139">Identity update</span></span>

<span data-ttu-id="37bed-140">È stata semplificata la creazione di API Web sicure con Identity in ASP.NET Core 2,0.</span><span class="sxs-lookup"><span data-stu-id="37bed-140">We've made it easier to build secure web APIs using Identity in ASP.NET Core 2.0.</span></span> <span data-ttu-id="37bed-141">È possibile acquisire i token di accesso per accedere alle API Web usando la libreria di autenticazione [MSAL](https://www.nuget.org/packages/Microsoft.Identity.Client).</span><span class="sxs-lookup"><span data-stu-id="37bed-141">You can acquire access tokens for accessing your web APIs using the [Microsoft Authentication Library (MSAL)](https://www.nuget.org/packages/Microsoft.Identity.Client).</span></span>

<span data-ttu-id="37bed-142">Per altre informazioni sulle modifiche apportate all'autenticazione nella versione 2.0, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="37bed-142">For more information on authentication changes in 2.0, see the following resources:</span></span>

* <span data-ttu-id="37bed-143">[Account confirmation and password recovery in ASP.NET Core](xref:security/authentication/accconfirm) (Conferma dell'account e recupero della password in ASP.NET Core)</span><span class="sxs-lookup"><span data-stu-id="37bed-143">[Account confirmation and password recovery in ASP.NET Core](xref:security/authentication/accconfirm)</span></span>
* [<span data-ttu-id="37bed-144">Abilitare la generazione di codice a matrice per le app di autenticazione in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="37bed-144">Enable QR Code generation for authenticator apps in ASP.NET Core</span></span>](xref:security/authentication/identity-enable-qrcodes)
* [<span data-ttu-id="37bed-145">Eseguire la migrazione dell'autenticazione e Identity al ASP.NET Core 2,0</span><span class="sxs-lookup"><span data-stu-id="37bed-145">Migrate Authentication and Identity to ASP.NET Core 2.0</span></span>](xref:migration/1x-to-2x/identity-2x)

## <a name="spa-templates"></a><span data-ttu-id="37bed-146">Modelli SPA</span><span class="sxs-lookup"><span data-stu-id="37bed-146">SPA templates</span></span>

<span data-ttu-id="37bed-147">Sono disponibili modelli di progetto di applicazione a pagina singola, o SPA (Single-Page Application), per Angular, Aurelia, Knockout.js, React.js e React.js con Redux.</span><span class="sxs-lookup"><span data-stu-id="37bed-147">Single Page Application (SPA) project templates for Angular, Aurelia, Knockout.js, React.js, and React.js with Redux are available.</span></span> <span data-ttu-id="37bed-148">Il modello Angular è stato aggiornato alla versione 4.</span><span class="sxs-lookup"><span data-stu-id="37bed-148">The Angular template has been updated to Angular 4.</span></span> <span data-ttu-id="37bed-149">I modelli Angular e React sono disponibili per impostazione predefinita. Per informazioni su come ottenere gli altri modelli, vedere [Creare un nuovo progetto di applicazione a pagina singola](xref:client-side/spa-services#create-a-new-project).</span><span class="sxs-lookup"><span data-stu-id="37bed-149">The Angular and React templates are available by default; for information about how to get the other templates, see [Create a new SPA project](xref:client-side/spa-services#create-a-new-project).</span></span> <span data-ttu-id="37bed-150">Per informazioni su come compilare un'applicazione a pagina singola in ASP.NET Core, vedere <xref:client-side/spa-services>.</span><span class="sxs-lookup"><span data-stu-id="37bed-150">For information about how to build a SPA in ASP.NET Core, see <xref:client-side/spa-services>.</span></span>

## <a name="kestrel-improvements"></a><span data-ttu-id="37bed-151">Miglioramenti di Kestrel</span><span class="sxs-lookup"><span data-stu-id="37bed-151">Kestrel improvements</span></span>

<span data-ttu-id="37bed-152">Il server Web Kestrel include nuove funzionalità che lo rendono più adatto all'uso come server con connessione Internet.</span><span class="sxs-lookup"><span data-stu-id="37bed-152">The Kestrel web server has new features that make it more suitable as an Internet-facing server.</span></span> <span data-ttu-id="37bed-153">Sono state aggiunte diverse opzioni di configurazione dei vincoli in una nuova proprietà della classe `KestrelServerOptions`, `Limits`.</span><span class="sxs-lookup"><span data-stu-id="37bed-153">A number of server constraint configuration options are added in the `KestrelServerOptions` class's new `Limits` property.</span></span> <span data-ttu-id="37bed-154">Aggiungere limiti per gli elementi seguenti:</span><span class="sxs-lookup"><span data-stu-id="37bed-154">Add limits for the following:</span></span>

* <span data-ttu-id="37bed-155">Numero massimo di connessioni client</span><span class="sxs-lookup"><span data-stu-id="37bed-155">Maximum client connections</span></span>
* <span data-ttu-id="37bed-156">Dimensione massima del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="37bed-156">Maximum request body size</span></span>
* <span data-ttu-id="37bed-157">Velocità minima dei dati del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="37bed-157">Minimum request body data rate</span></span>

<span data-ttu-id="37bed-158">Per altre informazioni, vedere [Introduzione all'implementazione di server Web Kestrel in ASP.NET Core](xref:fundamentals/servers/kestrel).</span><span class="sxs-lookup"><span data-stu-id="37bed-158">For more information, see [Kestrel web server implementation in ASP.NET Core](xref:fundamentals/servers/kestrel).</span></span>

## <a name="weblistener-renamed-to-httpsys"></a><span data-ttu-id="37bed-159">WebListener rinominato in HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="37bed-159">WebListener renamed to HTTP.sys</span></span>

<span data-ttu-id="37bed-160">I pacchetti `Microsoft.AspNetCore.Server.WebListener` e `Microsoft.Net.Http.Server` sono state uniti in un nuovo pacchetto `Microsoft.AspNetCore.Server.HttpSys`.</span><span class="sxs-lookup"><span data-stu-id="37bed-160">The packages `Microsoft.AspNetCore.Server.WebListener` and `Microsoft.Net.Http.Server` have been merged into a new package `Microsoft.AspNetCore.Server.HttpSys`.</span></span> <span data-ttu-id="37bed-161">Gli spazi dei nomi sono stati aggiornati di conseguenza.</span><span class="sxs-lookup"><span data-stu-id="37bed-161">The namespaces have been updated to match.</span></span>

<span data-ttu-id="37bed-162">Per altre informazioni, vedere l'introduzione all'[implementazione del server Web HTTP.sys in ASP.NET Core](xref:fundamentals/servers/httpsys).</span><span class="sxs-lookup"><span data-stu-id="37bed-162">For more information, see [HTTP.sys web server implementation in ASP.NET Core](xref:fundamentals/servers/httpsys).</span></span>

## <a name="enhanced-http-header-support"></a><span data-ttu-id="37bed-163">Supporto ottimizzato per le intestazioni HTTP</span><span class="sxs-lookup"><span data-stu-id="37bed-163">Enhanced HTTP header support</span></span>

<span data-ttu-id="37bed-164">Quando si usa MVC per trasmettere un oggetto `FileStreamResult` o `FileContentResult`, è ora possibile impostare un `ETag` o una data `LastModified` per il contenuto trasmesso.</span><span class="sxs-lookup"><span data-stu-id="37bed-164">When using MVC to transmit a `FileStreamResult` or a `FileContentResult`, you now have the option to set an `ETag` or a `LastModified` date on the content you transmit.</span></span> <span data-ttu-id="37bed-165">È possibile impostare questi valori per il contenuto restituito con codice simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="37bed-165">You can set these values on the returned content with code similar to the following:</span></span>

```csharp
var data = Encoding.UTF8.GetBytes("This is a sample text from a binary array");
var entityTag = new EntityTagHeaderValue("\"MyCalculatedEtagValue\"");
return File(data, "text/plain", "downloadName.txt", lastModified: DateTime.UtcNow.AddSeconds(-5), entityTag: entityTag);
```

<span data-ttu-id="37bed-166">Il file restituito ai visitatori presenta le intestazioni HTTP appropriate per i `ETag` valori e `LastModified` .</span><span class="sxs-lookup"><span data-stu-id="37bed-166">The file returned to your visitors has the appropriate HTTP headers for the `ETag` and `LastModified` values.</span></span>

<span data-ttu-id="37bed-167">Se il visitatore di un'applicazione richiede contenuto con un'intestazione di richiesta di intervallo, ASP.NET Core lo riconosce e gestisce l'intestazione.</span><span class="sxs-lookup"><span data-stu-id="37bed-167">If an application visitor requests content with a Range Request header, ASP.NET Core recognizes the request and handles the header.</span></span> <span data-ttu-id="37bed-168">Se il contenuto richiesto può essere recapitato parzialmente, ASP.NET Core ignora e considera le varie parti in modo appropriato restituendo solo il set di byte richiesto.</span><span class="sxs-lookup"><span data-stu-id="37bed-168">If the requested content can be partially delivered, ASP.NET Core appropriately skips and returns just the requested set of bytes.</span></span> <span data-ttu-id="37bed-169">Non è necessario scrivere gestori speciali nei metodi per adattare o gestire questa funzionalità, poiché è gestita automaticamente.</span><span class="sxs-lookup"><span data-stu-id="37bed-169">You don't need to write any special handlers into your methods to adapt or handle this feature; it's automatically handled for you.</span></span>

## <a name="hosting-startup-and-application-insights"></a><span data-ttu-id="37bed-170">Avvio in hosting e Application Insights</span><span class="sxs-lookup"><span data-stu-id="37bed-170">Hosting startup and Application Insights</span></span>

<span data-ttu-id="37bed-171">Gli ambienti di hosting sono ora in grado di inserire le dipendenze aggiuntive dei pacchetti e di eseguire codice durante l'avvio dell'applicazione, senza che per l'applicazione sia necessario accettare una dipendenza o chiamare metodi in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="37bed-171">Hosting environments can now inject extra package dependencies and execute code during application startup, without the application needing to explicitly take a dependency or call any methods.</span></span> <span data-ttu-id="37bed-172">Questa funzionalità può essere usata per consentire ad alcuni ambienti di attivare le proprie funzionalità esclusive senza che sia necessario indicarlo anticipatamente all'applicazione.</span><span class="sxs-lookup"><span data-stu-id="37bed-172">This feature can be used to enable certain environments to "light-up" features unique to that environment without the application needing to know ahead of time.</span></span> 

<span data-ttu-id="37bed-173">In ASP.NET Core 2.0 questa funzionalità viene usata per abilitare automaticamente la diagnostica di Application Insights durante il debug in Visual Studio e, dopo aver scelto questa opzione, durante l'esecuzione in Servizi app di Azure.</span><span class="sxs-lookup"><span data-stu-id="37bed-173">In ASP.NET Core 2.0, this feature is used to automatically enable Application Insights diagnostics when debugging in Visual Studio and (after opting in) when running in Azure App Services.</span></span> <span data-ttu-id="37bed-174">Di conseguenza, i modelli di progetto non aggiungono più i pacchetti e il codice di Application Insights per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="37bed-174">As a result, the project templates no longer add Application Insights packages and code by default.</span></span>

<span data-ttu-id="37bed-175">Per informazioni sullo stato della documentazione prevista, vedere l'[argomento su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/3389).</span><span class="sxs-lookup"><span data-stu-id="37bed-175">For information about the status of planned documentation, see the [GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/3389).</span></span>

## <a name="automatic-use-of-anti-forgery-tokens"></a><span data-ttu-id="37bed-176">Uso automatico dei token antifalsificazione</span><span class="sxs-lookup"><span data-stu-id="37bed-176">Automatic use of anti-forgery tokens</span></span>

<span data-ttu-id="37bed-177">ASP.NET Core ha sempre agevolato la codifica HTML del contenuto per impostazione predefinita, ma con la nuova versione è stato fatto un passo avanti nella prevenzione degli attacchi basati sulla falsificazione della richiesta tra siti (XSRF).</span><span class="sxs-lookup"><span data-stu-id="37bed-177">ASP.NET Core has always helped HTML-encode content by default, but with the new version an extra step is taken to help prevent cross-site request forgery (XSRF) attacks.</span></span> <span data-ttu-id="37bed-178">ASP.NET Core ora genera token antifalsificazione per impostazione predefinita e li convalida per le pagine e le azioni POST dei form senza configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="37bed-178">ASP.NET Core will now emit anti-forgery tokens by default and validate them on form POST actions and pages without extra configuration.</span></span>

<span data-ttu-id="37bed-179">Per altre informazioni, vedere <xref:security/anti-request-forgery>.</span><span class="sxs-lookup"><span data-stu-id="37bed-179">For more information, see <xref:security/anti-request-forgery>.</span></span>

## <a name="automatic-precompilation"></a><span data-ttu-id="37bed-180">Precompilazione automatica</span><span class="sxs-lookup"><span data-stu-id="37bed-180">Automatic precompilation</span></span>

<span data-ttu-id="37bed-181">Razor per impostazione predefinita, la pre-compilazione viene abilitata durante la pubblicazione, riducendo le dimensioni dell'output di pubblicazione e l'ora di avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="37bed-181">Razor view pre-compilation is enabled during publish by default, reducing the publish output size and application startup time.</span></span>

<span data-ttu-id="37bed-182">Per ulteriori informazioni, vedere la pagina relativa [ Razor alla visualizzazione della compilazione e della precompilazione in ASP.NET Core](xref:mvc/views/view-compilation).</span><span class="sxs-lookup"><span data-stu-id="37bed-182">For more information, see [Razor view compilation and precompilation in ASP.NET Core](xref:mvc/views/view-compilation).</span></span>

## <a name="razor-support-for-c-71"></a><span data-ttu-id="37bed-183">Razor supporto per C# 7,1</span><span class="sxs-lookup"><span data-stu-id="37bed-183">Razor support for C# 7.1</span></span>

<span data-ttu-id="37bed-184">Il Razor motore di visualizzazione è stato aggiornato per funzionare con il nuovo compilatore Roslyn.</span><span class="sxs-lookup"><span data-stu-id="37bed-184">The Razor view engine has been updated to work with the new Roslyn compiler.</span></span> <span data-ttu-id="37bed-185">Questo include il supporto per funzionalità di C# 7.1 tra cui le espressioni predefinite, i nomi di tupla dedotti e i criteri di ricerca con i generics.</span><span class="sxs-lookup"><span data-stu-id="37bed-185">That includes support for C# 7.1 features like Default Expressions, Inferred Tuple Names, and Pattern-Matching with Generics.</span></span> <span data-ttu-id="37bed-186">Per usare C# 7.1 nel progetto, aggiungere la proprietà seguente nel file di progetto e quindi ricaricare la soluzione:</span><span class="sxs-lookup"><span data-stu-id="37bed-186">To use C# 7.1 in your project, add the following property in your project file and then reload the solution:</span></span>

```xml
<LangVersion>latest</LangVersion>
```

<span data-ttu-id="37bed-187">Per informazioni sullo stato delle funzionalità di C# 7.1, vedere il [repository GitHub per Roslyn](https://github.com/dotnet/roslyn/blob/main/docs/Language%20Feature%20Status.md).</span><span class="sxs-lookup"><span data-stu-id="37bed-187">For information about the status of C# 7.1 features, see [the Roslyn GitHub repository](https://github.com/dotnet/roslyn/blob/main/docs/Language%20Feature%20Status.md).</span></span>

## <a name="other-documentation-updates-for-20"></a><span data-ttu-id="37bed-188">Altri aggiornamenti alla documentazione per la versione 2.0</span><span class="sxs-lookup"><span data-stu-id="37bed-188">Other documentation updates for 2.0</span></span>

* [<span data-ttu-id="37bed-189">Profili di pubblicazione di Visual Studio per la distribuzione di app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="37bed-189">Visual Studio publish profiles for ASP.NET Core app deployment</span></span>](xref:host-and-deploy/visual-studio-publish-profiles)
* [<span data-ttu-id="37bed-190">Gestione delle chiavi</span><span class="sxs-lookup"><span data-stu-id="37bed-190">Key Management</span></span>](xref:security/data-protection/implementation/key-management)
* [<span data-ttu-id="37bed-191">Configurare l'autenticazione di Facebook</span><span class="sxs-lookup"><span data-stu-id="37bed-191">Configure Facebook authentication</span></span>](xref:security/authentication/facebook-logins)
* [<span data-ttu-id="37bed-192">Configurare l'autenticazione di Twitter</span><span class="sxs-lookup"><span data-stu-id="37bed-192">Configure Twitter authentication</span></span>](xref:security/authentication/twitter-logins)
* [<span data-ttu-id="37bed-193">Configurare l'autenticazione di Google</span><span class="sxs-lookup"><span data-stu-id="37bed-193">Configure Google authentication</span></span>](xref:security/authentication/google-logins)
* [<span data-ttu-id="37bed-194">Configurazione dell'autenticazione di account Microsoft</span><span class="sxs-lookup"><span data-stu-id="37bed-194">Configure Microsoft Account authentication</span></span>](xref:security/authentication/microsoft-logins)

## <a name="migration-guidance"></a><span data-ttu-id="37bed-195">Indicazioni sulla migrazione</span><span class="sxs-lookup"><span data-stu-id="37bed-195">Migration guidance</span></span>

<span data-ttu-id="37bed-196">Per indicazioni su come eseguire la migrazione delle applicazioni ASP.NET Core 1.x ad ASP.NET Core 2.0, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="37bed-196">For guidance on how to migrate ASP.NET Core 1.x applications to ASP.NET Core 2.0, see the following resources:</span></span>

* [<span data-ttu-id="37bed-197">Eseguire la migrazione da ASP.NET Core 1.x alla versione 2.0</span><span class="sxs-lookup"><span data-stu-id="37bed-197">Migrate from ASP.NET Core 1.x to ASP.NET Core 2.0</span></span>](xref:migration/1x-to-2x/index)
* [<span data-ttu-id="37bed-198">Eseguire la migrazione dell'autenticazione e Identity al ASP.NET Core 2,0</span><span class="sxs-lookup"><span data-stu-id="37bed-198">Migrate Authentication and Identity to ASP.NET Core 2.0</span></span>](xref:migration/1x-to-2x/identity-2x)

## <a name="additional-information"></a><span data-ttu-id="37bed-199">Informazioni aggiuntive</span><span class="sxs-lookup"><span data-stu-id="37bed-199">Additional Information</span></span>

<span data-ttu-id="37bed-200">Per l'elenco completo delle modifiche, vedere le [note sulla versione di ASP.NET Core 2.0](https://github.com/dotnet/aspnetcore/releases/tag/2.0.0).</span><span class="sxs-lookup"><span data-stu-id="37bed-200">For the complete list of changes, see the [ASP.NET Core 2.0 Release Notes](https://github.com/dotnet/aspnetcore/releases/tag/2.0.0).</span></span>

<span data-ttu-id="37bed-201">Per essere aggiornati sull'avanzamento del lavoro e sui piani dei team di sviluppo di ASP.NET Core, partecipare alle riunioni in [ASP.NET Community Standup](https://live.asp.net/).</span><span class="sxs-lookup"><span data-stu-id="37bed-201">To connect with the ASP.NET Core development team's progress and plans, tune in to the [ASP.NET Community Standup](https://live.asp.net/).</span></span>
