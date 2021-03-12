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
# <a name="whats-new-in-aspnet-core-20"></a>Novità di ASP.NET Core 2.0

Questo articolo evidenzia le modifiche più significative apportate ad ASP.NET Core 2.0, con collegamenti alla relativa documentazione.

## <a name="razor-pages"></a>Razor Pagine

Razor Pages è una nuova funzionalità di ASP.NET Core MVC che rende più semplice e più produttivo gli scenari di codifica della pagina.

Per altre informazioni, vedere l'introduzione e l'esercitazione:

* [Introduzione alle Razor pagine](xref:razor-pages/index)
* [Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start)

## <a name="aspnet-core-metapackage"></a>Metapacchetto ASP.NET Core

Un nuovo metapacchetto ASP.NET Core include tutti i pacchetti creati e supportati dai team di ASP.NET Core ed Entity Framework Core, con le relative dipendenze interne e di terze parti. Non è più necessario scegliere le singole funzionalità di ASP.NET Core in base al pacchetto. Tutte le funzionalità sono incluse nel pacchetto [Microsoft.AspNetCore.All](https://www.nuget.org/packages/Microsoft.AspNetCore.All). I modelli predefiniti usano questo pacchetto.

Per altre informazioni, vedere [Microsoft.AspNetCore.All metapackage for ASP.NET Core 2.0](xref:fundamentals/metapackage) (Metapacchetto Microsoft.AspNetCore.All per ASP.NET Core 2.0).

## <a name="runtime-store"></a>Archivio di runtime

Le applicazioni che usano il metapacchetto `Microsoft.AspNetCore.All` sfruttano automaticamente il nuovo archivio di runtime di .NET Core. L'archivio contiene tutti gli asset di runtime necessari per eseguire le applicazioni ASP.NET Core 2.0. Quando si usa il metapacchetto `Microsoft.AspNetCore.All`, con l'applicazione non vengono distribuiti asset dai pacchetti NuGet di riferimento di ASP.NET Core poiché sono già presenti nel sistema di destinazione. Gli asset contenuti nell'archivio di runtime vengono anche precompilati per migliorare i tempi di avvio dell'applicazione.

Per altre informazioni, vedere l'articolo relativo all'[archivio dei pacchetti di runtime](/dotnet/core/deploying/runtime-store)

## <a name="net-standard-20"></a>.NET Standard 2.0

I pacchetti di ASP.NET Core 2.0 hanno come destinazione .NET Standard 2.0. Ai pacchetti possono fare riferimento altre librerie .NET Standard 2.0 ed è possibile eseguire i pacchetti in implementazioni di .NET compatibili con .NET Standard 2.0, tra cui .NET Core 2.0 e .NET Framework 4.6.1. 

Il metapacchetto `Microsoft.AspNetCore.All` è riservato esclusivamente a .NET Core 2.0, poiché è destinato all'uso con l'archivio di runtime di .NET Core 2.0.

## <a name="configuration-update"></a>Aggiornamento della configurazione

In ASP.NET Core 2.0 viene aggiunta per impostazione predefinita un'istanza di `IConfiguration` al contenitore dei servizi. `IConfiguration` nel contenitore dei servizi semplifica per le applicazioni il recupero dei valori di configurazione dal contenitore.

Per informazioni sullo stato della documentazione prevista, vedere l'[argomento su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/3387).

## <a name="logging-update"></a>Aggiornamento della registrazione

In ASP.NET Core 2.0 la registrazione è incorporata nel sistema di inserimento delle dipendenze per impostazione predefinita. Si aggiungono i provider e si configurano i filtri nel file *Program.cs* anziché nel file *Startup.cs*. E l'oggetto `ILoggerFactory` predefinito supporta i filtri in modo tale da consentire l'uso di un unico approccio flessibile sia per il filtraggio tra provider, sia per il filtraggio di un provider specifico.

Per altre informazioni, vedere l'[introduzione alla registrazione in ASP.NET Core](xref:fundamentals/logging/index).

## <a name="authentication-update"></a>Aggiornamento dell'autenticazione

Un nuovo modello di autenticazione rende più semplice configurare l'autenticazione per un'applicazione che usa l'inserimento delle dipendenze.

I nuovi modelli sono disponibili per la configurazione dell'autenticazione per le app Web e le API Web che usano [Azure AD B2C](https://azure.microsoft.com/services/active-directory-b2c/).

Per informazioni sullo stato della documentazione prevista, vedere l'[argomento su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/3054).

## <a name="identity-update"></a>Aggiornamento di Identity

È stata semplificata la creazione di API Web sicure con Identity in ASP.NET Core 2,0. È possibile acquisire i token di accesso per accedere alle API Web usando la libreria di autenticazione [MSAL](https://www.nuget.org/packages/Microsoft.Identity.Client).

Per altre informazioni sulle modifiche apportate all'autenticazione nella versione 2.0, vedere le risorse seguenti:

* [Account confirmation and password recovery in ASP.NET Core](xref:security/authentication/accconfirm) (Conferma dell'account e recupero della password in ASP.NET Core)
* [Abilitare la generazione di codice a matrice per le app di autenticazione in ASP.NET Core](xref:security/authentication/identity-enable-qrcodes)
* [Eseguire la migrazione dell'autenticazione e Identity al ASP.NET Core 2,0](xref:migration/1x-to-2x/identity-2x)

## <a name="spa-templates"></a>Modelli SPA

Sono disponibili modelli di progetto di applicazione a pagina singola, o SPA (Single-Page Application), per Angular, Aurelia, Knockout.js, React.js e React.js con Redux. Il modello Angular è stato aggiornato alla versione 4. I modelli Angular e React sono disponibili per impostazione predefinita. Per informazioni su come ottenere gli altri modelli, vedere [Creare un nuovo progetto di applicazione a pagina singola](xref:client-side/spa-services#create-a-new-project). Per informazioni su come compilare un'applicazione a pagina singola in ASP.NET Core, vedere <xref:client-side/spa-services>.

## <a name="kestrel-improvements"></a>Miglioramenti di Kestrel

Il server Web Kestrel include nuove funzionalità che lo rendono più adatto all'uso come server con connessione Internet. Sono state aggiunte diverse opzioni di configurazione dei vincoli in una nuova proprietà della classe `KestrelServerOptions`, `Limits`. Aggiungere limiti per gli elementi seguenti:

* Numero massimo di connessioni client
* Dimensione massima del corpo della richiesta
* Velocità minima dei dati del corpo della richiesta

Per altre informazioni, vedere [Introduzione all'implementazione di server Web Kestrel in ASP.NET Core](xref:fundamentals/servers/kestrel).

## <a name="weblistener-renamed-to-httpsys"></a>WebListener rinominato in HTTP.sys

I pacchetti `Microsoft.AspNetCore.Server.WebListener` e `Microsoft.Net.Http.Server` sono state uniti in un nuovo pacchetto `Microsoft.AspNetCore.Server.HttpSys`. Gli spazi dei nomi sono stati aggiornati di conseguenza.

Per altre informazioni, vedere l'introduzione all'[implementazione del server Web HTTP.sys in ASP.NET Core](xref:fundamentals/servers/httpsys).

## <a name="enhanced-http-header-support"></a>Supporto ottimizzato per le intestazioni HTTP

Quando si usa MVC per trasmettere un oggetto `FileStreamResult` o `FileContentResult`, è ora possibile impostare un `ETag` o una data `LastModified` per il contenuto trasmesso. È possibile impostare questi valori per il contenuto restituito con codice simile al seguente:

```csharp
var data = Encoding.UTF8.GetBytes("This is a sample text from a binary array");
var entityTag = new EntityTagHeaderValue("\"MyCalculatedEtagValue\"");
return File(data, "text/plain", "downloadName.txt", lastModified: DateTime.UtcNow.AddSeconds(-5), entityTag: entityTag);
```

Il file restituito ai visitatori presenta le intestazioni HTTP appropriate per i `ETag` valori e `LastModified` .

Se il visitatore di un'applicazione richiede contenuto con un'intestazione di richiesta di intervallo, ASP.NET Core lo riconosce e gestisce l'intestazione. Se il contenuto richiesto può essere recapitato parzialmente, ASP.NET Core ignora e considera le varie parti in modo appropriato restituendo solo il set di byte richiesto. Non è necessario scrivere gestori speciali nei metodi per adattare o gestire questa funzionalità, poiché è gestita automaticamente.

## <a name="hosting-startup-and-application-insights"></a>Avvio in hosting e Application Insights

Gli ambienti di hosting sono ora in grado di inserire le dipendenze aggiuntive dei pacchetti e di eseguire codice durante l'avvio dell'applicazione, senza che per l'applicazione sia necessario accettare una dipendenza o chiamare metodi in modo esplicito. Questa funzionalità può essere usata per consentire ad alcuni ambienti di attivare le proprie funzionalità esclusive senza che sia necessario indicarlo anticipatamente all'applicazione. 

In ASP.NET Core 2.0 questa funzionalità viene usata per abilitare automaticamente la diagnostica di Application Insights durante il debug in Visual Studio e, dopo aver scelto questa opzione, durante l'esecuzione in Servizi app di Azure. Di conseguenza, i modelli di progetto non aggiungono più i pacchetti e il codice di Application Insights per impostazione predefinita.

Per informazioni sullo stato della documentazione prevista, vedere l'[argomento su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/3389).

## <a name="automatic-use-of-anti-forgery-tokens"></a>Uso automatico dei token antifalsificazione

ASP.NET Core ha sempre agevolato la codifica HTML del contenuto per impostazione predefinita, ma con la nuova versione è stato fatto un passo avanti nella prevenzione degli attacchi basati sulla falsificazione della richiesta tra siti (XSRF). ASP.NET Core ora genera token antifalsificazione per impostazione predefinita e li convalida per le pagine e le azioni POST dei form senza configurazione aggiuntiva.

Per altre informazioni, vedere <xref:security/anti-request-forgery>.

## <a name="automatic-precompilation"></a>Precompilazione automatica

Razor per impostazione predefinita, la pre-compilazione viene abilitata durante la pubblicazione, riducendo le dimensioni dell'output di pubblicazione e l'ora di avvio dell'applicazione.

Per ulteriori informazioni, vedere la pagina relativa [ Razor alla visualizzazione della compilazione e della precompilazione in ASP.NET Core](xref:mvc/views/view-compilation).

## <a name="razor-support-for-c-71"></a>Razor supporto per C# 7,1

Il Razor motore di visualizzazione è stato aggiornato per funzionare con il nuovo compilatore Roslyn. Questo include il supporto per funzionalità di C# 7.1 tra cui le espressioni predefinite, i nomi di tupla dedotti e i criteri di ricerca con i generics. Per usare C# 7.1 nel progetto, aggiungere la proprietà seguente nel file di progetto e quindi ricaricare la soluzione:

```xml
<LangVersion>latest</LangVersion>
```

Per informazioni sullo stato delle funzionalità di C# 7.1, vedere il [repository GitHub per Roslyn](https://github.com/dotnet/roslyn/blob/main/docs/Language%20Feature%20Status.md).

## <a name="other-documentation-updates-for-20"></a>Altri aggiornamenti alla documentazione per la versione 2.0

* [Profili di pubblicazione di Visual Studio per la distribuzione di app ASP.NET Core](xref:host-and-deploy/visual-studio-publish-profiles)
* [Gestione delle chiavi](xref:security/data-protection/implementation/key-management)
* [Configurare l'autenticazione di Facebook](xref:security/authentication/facebook-logins)
* [Configurare l'autenticazione di Twitter](xref:security/authentication/twitter-logins)
* [Configurare l'autenticazione di Google](xref:security/authentication/google-logins)
* [Configurazione dell'autenticazione di account Microsoft](xref:security/authentication/microsoft-logins)

## <a name="migration-guidance"></a>Indicazioni sulla migrazione

Per indicazioni su come eseguire la migrazione delle applicazioni ASP.NET Core 1.x ad ASP.NET Core 2.0, vedere le risorse seguenti:

* [Eseguire la migrazione da ASP.NET Core 1.x alla versione 2.0](xref:migration/1x-to-2x/index)
* [Eseguire la migrazione dell'autenticazione e Identity al ASP.NET Core 2,0](xref:migration/1x-to-2x/identity-2x)

## <a name="additional-information"></a>Informazioni aggiuntive

Per l'elenco completo delle modifiche, vedere le [note sulla versione di ASP.NET Core 2.0](https://github.com/dotnet/aspnetcore/releases/tag/2.0.0).

Per essere aggiornati sull'avanzamento del lavoro e sui piani dei team di sviluppo di ASP.NET Core, partecipare alle riunioni in [ASP.NET Community Standup](https://live.asp.net/).
