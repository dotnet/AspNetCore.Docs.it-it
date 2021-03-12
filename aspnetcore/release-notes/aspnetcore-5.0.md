---
title: Novità di ASP.NET Core 5,0
author: rick-anderson
description: Informazioni sulle nuove funzionalità di ASP.NET Core 5,0.
ms.author: riande
ms.custom: mvc
ms.date: 10/29/2020
no-loc:
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
- Kestrel
uid: aspnetcore-5.0
ms.openlocfilehash: 64e068194744d6c5495d8ecaf31076c7a1c1f349
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585799"
---
# <a name="whats-new-in-aspnet-core-50"></a>Novità di ASP.NET Core 5,0

In questo articolo vengono evidenziate le modifiche più significative in ASP.NET Core 5,0 con i collegamenti alla documentazione pertinente.

## <a name="aspnet-core-mvc-and-razor-improvements"></a>ASP.NET Core MVC e Razor miglioramenti

### <a name="model-binding-datetime-as-utc"></a>Associazione di modelli DateTime come ora UTC

L'associazione di modelli ora supporta l'associazione di stringhe di ora UTC a `DateTime` . Se la richiesta contiene una stringa di ora UTC, l'associazione di modelli lo associa a un formato UTC `DateTime` . Ad esempio, la stringa temporale seguente viene associata all'ora UTC `DateTime` : `https://example.com/mycontroller/myaction?time=2019-06-14T02%3A30%3A04.0576719Z`

### <a name="model-binding-and-validation-with-c-9-record-types"></a>Associazione di modelli e convalida con i tipi di record C# 9

I [tipi di record C# 9](/dotnet/csharp/whats-new/csharp-9#record-types) possono essere usati con l'associazione di modelli in un controller MVC o in una Razor pagina. I tipi di record rappresentano un modo efficace per modellare i dati trasmessi in rete.

Il codice seguente, ad esempio, `PersonController` Usa il `Person` tipo di record con associazione di modelli e convalida dei moduli:

```csharp
public record Person([Required] string Name, [Range(0, 150)] int Age);

public class PersonController
{
   public IActionResult Index() => View();

   [HttpPost]
   public IActionResult Index(Person person)
   {
          // ...
   }
}
```

Ecco il file `Person/Index.cshtml`:

```cshtml
@model Person

Name: <input asp-for="Model.Name" />
<span asp-validation-for="Model.Name" />

Age: <input asp-for="Model.Age" />
<span asp-validation-for="Model.Age" />
```

### <a name="improvements-to-dynamicroutevaluetransformer"></a>Miglioramenti apportati a DynamicRouteValueTransformer

ASP.NET Core 3,1 è stato introdotto <xref:Microsoft.AspNetCore.Mvc.Routing.DynamicRouteValueTransformer> come un modo per usare l'endpoint personalizzato per selezionare dinamicamente un'azione del controller MVC o una Razor pagina. ASP.NET Core 5,0 le app possono passare lo stato a un oggetto `DynamicRouteValueTransformer` e filtrare il set di endpoint scelti.

### <a name="miscellaneous"></a>Varie

* L'attributo [[compare]](xref:System.ComponentModel.DataAnnotations.CompareAttribute) può essere applicato alle proprietà di un Razor modello di pagina.
* I parametri e le proprietà associati dal corpo sono considerati necessari per impostazione predefinita. <!-- Review: How is this different from 3.1
The validation system in .NET Core 3.0 and later treats non-nullable parameters or bound properties as if they had a [Required] attribute.
see https://docs.microsoft.com/aspnet/core/mvc/models/validation?view=aspnetcore-3.1   
-->

## <a name="web-api"></a>API Web

### <a name="openapi-specification-on-by-default"></a>Specifica OpenAPI su per impostazione predefinita

[Openapi Specification](http://spec.openapis.org/oas/v3.0.3) è uno standard di settore per la descrizione delle API HTTP e l'integrazione in processi aziendali complessi o con terze parti. OpenAPI è ampiamente supportato da tutti i provider di servizi cloud e da molti registri API. Le app che emettono documenti OpenAPI da API Web hanno una varietà di nuove opportunità in cui è possibile usare tali API. In collaborazione con i gestori del progetto open source [Swashbuckle. AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/), il modello API ASP.NET Core contiene una dipendenza NuGet da [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore). Swashbuckle è un noto pacchetto NuGet open source che genera in modo dinamico i documenti di OpenAPI. Swashbuckle esegue questa operazione introspezione sui controller API e generando il documento OpenAPI in fase di esecuzione oppure in fase di compilazione usando l'interfaccia della riga di comando di Swashbuckle.

In ASP.NET Core 5,0, i modelli di API Web abilitano il supporto OpenAPI per impostazione predefinita. Per disabilitare OpenAPI:

* Dalla riga di comando:

    ```dotnetcli
    dotnet new webapi --no-openapi true
    ```
* Da Visual Studio: deselezionare **Abilita supporto openapi**.

Tutti i file con *estensione csproj* creati per i progetti di API Web contengono il riferimento al pacchetto NuGet [Swashbuckle. AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/) .

```xml
<ItemGroup>
    <PackageReference Include="Swashbuckle.AspNetCore" Version="5.5.1" />
</ItemGroup>
```

Il codice generato dal modello contiene codice in `Startup.ConfigureServices` che attiva la generazione di documenti openapi:

[!code-csharp[](~/release-notes/sample/StartupSwagger.cs?name=snippet)]

Il `Startup.Configure` metodo aggiunge il middleware Swashbuckle, che Abilita:

* Processo di generazione del documento.
* Pagina dell'interfaccia utente di spavalderia per impostazione predefinita in modalità di sviluppo.

Il codice generato dal modello non esporrà accidentalmente la descrizione dell'API durante la pubblicazione nell'ambiente di produzione.

[!code-csharp[](~/release-notes/sample/StartupSwagger.cs?name=snippet2)]

#### <a name="azure-api-management-import"></a>Importazione di gestione API di Azure

Quando ASP.NET Core progetti API abilitano OpenAPI, Visual Studio 2019 versione 16,8 e successive pubblicano automaticamente un passaggio aggiuntivo nel flusso di pubblicazione. Gli sviluppatori che usano [gestione API di Azure](xref:tutorials/publish-to-azure-api-management-using-vs) hanno la possibilità di importare automaticamente le API in gestione API di Azure durante il flusso di pubblicazione:

![Importazione e pubblicazione di gestione API di Azure](~/release-notes/static/publish-to-apim.png)

### <a name="better-launch-experience-for-web-api-projects"></a>Migliore esperienza di avvio per i progetti API Web

Con OpenAPI abilitato per impostazione predefinita, l'esperienza di avvio dell'app (F5) per gli sviluppatori di API Web migliora significativamente. Con ASP.NET Core 5,0, il modello API Web è preconfigurato per il caricamento della pagina dell'interfaccia utente di spavalderia. La pagina dell'interfaccia utente di spavalderia fornisce sia la documentazione aggiunta per l'API pubblicata che consente di testare le API con un solo clic.

![visualizzazione di spavalderia/index.html](~/release-notes/static/swagger-ui-page-rc1.png)

## Blazor

### <a name="performance-improvements"></a>Miglioramenti delle prestazioni

Per .NET 5 sono stati apportati miglioramenti significativi alle Blazor WebAssembly prestazioni in fase di esecuzione con un particolare interesse per il rendering dell'interfaccia utente complesso e la serializzazione JSON. Nei test delle prestazioni, Blazor WebAssembly in .NET 5 è più veloce da due a tre volte per la maggior parte degli scenari. Per altre informazioni, vedere [ASP.NET Blog: aggiornamenti ASP.NET Core in .NET 5 Release Candidate 1](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-5-release-candidate-1/#blazor-webassembly-performance-improvements).

### <a name="css-isolation"></a>Isolamento CSS

Blazor supporta ora la definizione di stili CSS che hanno come ambito un determinato componente. Gli stili CSS specifici del componente consentono di ragionare più facilmente sugli stili in un'applicazione ed evitare effetti collaterali indesiderati degli stili globali. Per altre informazioni, vedere <xref:blazor/components/css-isolation>.

### <a name="new-inputfile-component"></a>Nuovo `InputFile` componente

Il `InputFile` componente consente di leggere uno o più file selezionati da un utente per il caricamento. Per altre informazioni, vedere <xref:blazor/file-uploads>.

### <a name="new-inputradio-and-inputradiogroup-components"></a>Nuovi `InputRadio` `InputRadioGroup` componenti e

Blazor dispone di componenti predefiniti `InputRadio` e `InputRadioGroup` che semplificano data binding ai gruppi di pulsanti di opzione con convalida integrata. Per altre informazioni, vedere <xref:blazor/forms-validation>.

### <a name="component-virtualization"></a>Virtualizzazione di componenti

Migliorare le prestazioni percepite del rendering dei componenti utilizzando il Blazor supporto della virtualizzazione incorporato del Framework. Per altre informazioni, vedere <xref:blazor/components/virtualization>.

### <a name="ontoggle-event-support"></a>`ontoggle` supporto degli eventi

Blazor gli eventi supportano ora l' `ontoggle` evento DOM. Per altre informazioni, vedere <xref:blazor/components/event-handling#event-argument-types>.

### <a name="set-ui-focus-in-blazor-apps"></a>Impostare lo stato attivo dell'interfaccia utente nelle Blazor app

Usare il `FocusAsync` Metodo convenience sui riferimenti a elementi per impostare lo stato attivo dell'interfaccia utente su tale elemento. Per altre informazioni, vedere <xref:blazor/components/event-handling#focus-an-element>.

### <a name="custom-validation-class-attributes"></a>Attributi della classe di convalida personalizzata

I nomi delle classi di convalida personalizzate sono utili per l'integrazione con Framework CSS, ad esempio bootstrap. Per altre informazioni, vedere <xref:blazor/forms-validation#custom-validation-class-attributes>.

### <a name="iasyncdisposable-support"></a>Supporto di IAsyncDisposable

Blazor i componenti supportano ora l' <xref:System.IAsyncDisposable> interfaccia per la versione asincrona delle risorse allocate.

### <a name="javascript-isolation-and-object-references"></a>Isolamento JavaScript e riferimenti a oggetti

Blazor Abilita l'isolamento JavaScript nei [moduli JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)standard. Per altre informazioni, vedere <xref:blazor/call-javascript-from-dotnet#blazor-javascript-isolation-and-object-references>.

### <a name="form-components-support-display-name"></a>I componenti del modulo supportano il nome visualizzato

I seguenti componenti predefiniti supportano i nomi visualizzati con il `DisplayName` parametro:

* `InputDate`
* `InputNumber`
* `InputSelect`

Per altre informazioni, vedere <xref:blazor/forms-validation#display-name-support>.

### <a name="catch-all-route-parameters"></a>Parametri di route catch-all

I parametri di route catch-all, che acquisiscono percorsi tra più limiti di cartella, sono supportati nei componenti di. Per altre informazioni, vedere <xref:blazor/fundamentals/routing#catch-all-route-parameters>.

### <a name="debugging-improvements"></a>Miglioramenti apportati al debug

Il debug Blazor WebAssembly delle app è stato migliorato in ASP.NET Core 5,0. Inoltre, il debug è ora supportato in Visual Studio per Mac. Per altre informazioni, vedere <xref:blazor/debug>.

### <a name="microsoft-identity-v20-and-msal-v20"></a>Microsoft Identity v 2.0 e MSAL v 2.0

Blazor la protezione ora usa Microsoft Identity v 2.0 ( [`Microsoft.Identity.Web`](https://www.nuget.org/packages/Microsoft.Identity.Web) e [`Microsoft.Identity.Web.UI`](https://www.nuget.org/packages/Microsoft.Identity.Web.UI) ) e MSAL v 2.0. Per ulteriori informazioni, vedere gli argomenti relativi alla [ Blazor sicurezza e al Identity nodo](xref:blazor/security/index).

### <a name="protected-browser-storage-for-blazor-server"></a>Archivio browser protetto per Blazor Server

Blazor Server le app possono ora usare il supporto incorporato per l'archiviazione dello stato dell'app nel browser protetto da manomissioni usando la protezione dei dati ASP.NET Core. I dati possono essere archiviati in un archivio del browser locale o in una sessione di archiviazione. Per altre informazioni, vedere <xref:blazor/state-management>.

### <a name="blazor-webassembly-prerendering"></a>Blazor WebAssembly tempistiche

L'integrazione dei componenti è stata migliorata tra i modelli di hosting e Blazor WebAssembly le app possono ora eseguire il prerendering dell'output nel server. <!-- UNCOMMENT AFTER https://github.com/dotnet/AspNetCore.Docs/pull/19887 MERGES: For more information, see <xref:blazor/components/integrate-components-into-razor-pages-and-mvc-apps> and <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>. -->

### <a name="trimminglinking-improvements"></a>Miglioramenti del taglio/collegamento

Blazor WebAssembly esegue il taglio e IL collegamento del linguaggio intermedio (IL) durante una compilazione per eliminare IL IL di il non necessario dagli assembly di output dell'app. Con il rilascio di ASP.NET Core 5,0, Blazor WebAssembly esegue un taglio migliorato con altre opzioni di configurazione. Per ulteriori informazioni, vedere <xref:blazor/host-and-deploy/configure-trimmer> e [Opzioni di trimming](/dotnet/core/deploying/trimming-options).

### <a name="browser-compatibility-analyzer"></a>Analizzatore compatibilità browser

Blazor WebAssembly le app sono destinate alla superficie di attacco dell'API .NET completa, ma non tutte le API .NET sono supportate nel webassembly a causa dei vincoli del sandbox del browser. Le API non supportate vengono generate <xref:System.PlatformNotSupportedException> durante l'esecuzione su webassembly. Un analizzatore della compatibilità della piattaforma avvisa lo sviluppatore quando l'app usa API non supportate dalle piattaforme di destinazione dell'app. Per altre informazioni, vedere <xref:blazor/components/class-libraries#browser-compatibility-analyzer-for-blazor-webassembly>.

### <a name="lazy-load-assemblies"></a>Assembly di caricamento lazy

Blazor WebAssembly le prestazioni di avvio delle app possono essere migliorate posticipando il caricamento di alcuni assembly di applicazioni fino a quando non sono necessarie. Per altre informazioni, vedere <xref:blazor/webassembly-lazy-load-assemblies>.

### <a name="updated-globalization-support"></a>Supporto per la globalizzazione aggiornato

Il supporto per la globalizzazione è disponibile per la Blazor WebAssembly basata sui componenti internazionali per Unicode (ICU). Per altre informazioni, vedere <xref:blazor/globalization-localization>.

## <a name="grpc"></a>gRPC

In [gRPC](https://grpc.io/)sono stati apportati numerosi miglioramenti alla conformità. Per ulteriori informazioni, vedere la pagina relativa ai [miglioramenti delle prestazioni di gRPC in .NET 5](https://devblogs.microsoft.com/aspnet/grpc-performance-improvements-in-net-5/).

Per ulteriori informazioni su gRPC, vedere <xref:grpc/index> .

## SignalR

### <a name="signalr-hub-filters"></a>SignalR Filtri Hub

SignalR I filtri Hub, detti pipeline dell'hub in ASP.NET SignalR , sono una funzionalità che consente l'esecuzione del codice prima e dopo la chiamata dei metodi dell'hub. L'esecuzione di codice prima e dopo la chiamata dei metodi dell'hub è simile al modo in cui middleware è in grado di eseguire il codice prima e dopo una richiesta HTTP. Gli utilizzi comuni includono la registrazione, la gestione degli errori e la convalida degli argomenti.

Per altre informazioni, vedere [use Hub filters in SignalR ASP.NET Core ](xref:signalr/hub-filters).

### <a name="signalr-parallel-hub-invocations"></a>SignalR chiamate dell'hub parallelo

ASP.NET Core SignalR è ora in grado di gestire le chiamate dell'hub parallelo. Il comportamento predefinito può essere modificato per consentire ai client di richiamare più di un metodo Hub alla volta:

[!code-csharp[](~/release-notes/sample/StartupSignalRhubs.cs?name=snippet)]

### <a name="added-messagepack-support-in-signalr-java-client"></a>Aggiunto il supporto di MessagePack in SignalR Java client

Un nuovo pacchetto, [com. Microsoft. SignalR. MessagePack](https://mvnrepository.com/artifact/com.microsoft.signalr.messagepack), aggiunge il supporto MessagePack al SignalR client Java. Per usare il protocollo dell'hub MessagePack, aggiungere `.withHubProtocol(new MessagePackHubProtocol())` al generatore di connessioni:

```java
HubConnection hubConnection = HubConnectionBuilder.create(
                           "http://localhost:53353/MyHub")
               .withHubProtocol(new MessagePackHubProtocol())
               .build();
```

<!--
See [Update SignalR code](xref:migration/31-to-50#signalr) for migration instructions.
-->

## Kestrel

* Endpoint ricaricabili tramite la configurazione: è Kestrel in grado di rilevare le modifiche apportate alla configurazione passata a [ KestrelServerOptions.ConfigUre](xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Configure%2A) e di annullare l'associazione da endpoint esistenti ed eseguire il binding ai nuovi endpoint senza richiedere il riavvio dell'applicazione quando il `reloadOnChange` parametro è `true` . Per impostazione predefinita <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults%2A> , quando si usa o, viene <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> Kestrel associato alla sottosezione di configurazione [" Kestrel "](https://github.com/dotnet/aspnetcore/blob/7e9e03b70124784b1de5564c573bd65cdaccbfcc/src/DefaultBuilder/src/WebHost.cs#L226) con `reloadOnChange` abilitato. Le app devono `reloadOnChange: true` essere passate quando `KestrelServerOptions.Configure` si chiama manualmente per ottenere endpoint ricaricabili.
* Miglioramenti delle intestazioni di risposta HTTP/2. Per ulteriori informazioni, vedere [miglioramenti delle prestazioni](#performance-improvements) nella sezione successiva.
* Supporto per altri tipi di endpoint nel trasporto di socket: l'aggiunta alla nuova API introdotta in <xref:System.Net.Sockets?displayProperty=nameWithType> , il trasporto predefinito dei socket in [Kestrel](xref:fundamentals/servers/kestrel) consente il binding sia agli handle di file esistenti che ai socket di dominio UNIX. Il supporto per l'associazione agli handle di file esistenti consente di utilizzare l' `Systemd` integrazione esistente senza richiedere il `libuv` trasporto.
* Decodifica dell'intestazione personalizzata in Kestrel : le app possono specificare quali <xref:System.Text.Encoding> usare per interpretare le intestazioni in ingresso in base al nome dell'intestazione anziché al valore predefinito UTF-8. Impostare il commutatore  <!--<xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.RequestHeaderEncodingSelector> --> `Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.RequestHeaderEncodingSelector` Proprietà per specificare la codifica da usare:
 
  ```csharp
  public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                options.RequestHeaderEncodingSelector = encoding =>
                {
                      return encoding switch
                      {
                          "Host" => System.Text.Encoding.Latin1,
                          _ => System.Text.Encoding.UTF8,
                      };
                };
            });
            webBuilder.UseStartup<Startup>();
        });
  ```

### <a name="kestrel-endpoint-specific-options-via-configuration"></a>Kestrel opzioni specifiche dell'endpoint tramite configurazione

È stato aggiunto il supporto per la configurazione delle Kestrel Opzioni specifiche dell'endpoint tramite la [configurazione](xref:fundamentals/configuration/index). Le configurazioni specifiche dell'endpoint includono:

* Protocolli HTTP utilizzati
* Protocolli TLS usati
* Certificato selezionato
* Modalità certificato client

La configurazione consente di specificare il certificato selezionato in base al nome del server specificato. Il nome del server fa parte dell'estensione Indicazione nome server (SNI) al protocollo TLS come indicato dal client. Kestrella configurazione di supporta anche un prefisso con caratteri jolly nel nome host.

Nell'esempio seguente viene illustrato come specificare specifici dell'endpoint utilizzando un file di configurazione:

```json
{
  "Kestrel": {
    "Endpoints": {
      "EndpointName": {
        "Url": "https://*",
        "Sni": {
          "a.example.org": {
            "Protocols": "Http1AndHttp2",
            "SslProtocols": [ "Tls11", "Tls12"],
            "Certificate": {
              "Path": "testCert.pfx",
              "Password": "testPassword"
            },
            "ClientCertificateMode" : "NoCertificate"
          },
          "*.example.org": {
            "Certificate": {
              "Path": "testCert2.pfx",
              "Password": "testPassword"
            }
          },
          "*": {
            // At least one sub-property needs to exist per
            // SNI section or it cannot be discovered via
            // IConfiguration
            "Protocols": "Http1",
          }
        }
      }
    }
  }
}
```

Indicazione nome server (SNI) è un'estensione TLS per includere un dominio virtuale come parte della negoziazione SSL. Ciò significa che il nome di dominio virtuale, o un nome host, può essere usato per identificare l'endpoint di rete.

## <a name="performance-improvements"></a>Miglioramenti delle prestazioni

### <a name="http2"></a>HTTP/2

* Riduzioni significative nelle allocazioni nel percorso del codice HTTP/2.
* Supporto per la [compressione dinamica HPack](https://tools.ietf.org/html/rfc7541) delle intestazioni di risposta http/2 in [Kestrel](xref:fundamentals/servers/kestrel) . Per altre informazioni, vedere [dimensioni della tabella di intestazione](xref:fundamentals/servers/kestrel/options#header-table-size) e [HPACK: l'assassino invisibile (Feature) di http/2](https://blog.cloudflare.com/hpack-the-silent-killer-feature-of-http-2/).
* Invio di frame PING HTTP/2: HTTP/2 dispone di un meccanismo per l'invio di frame PING per garantire che una connessione inattiva sia ancora funzionante. Garantire una connessione valida è particolarmente utile quando si lavora con flussi di lunga durata che spesso sono inattivi, ma solo a intermittenza, ad esempio i flussi gRPC. Le app possono inviare frame PING periodici in impostando i [Kestrel](xref:fundamentals/servers/kestrel) limiti per <xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions> :

   ```csharp
   public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                options.Limits.Http2.KeepAlivePingInterval =
                                               TimeSpan.FromSeconds(10);
                options.Limits.Http2.KeepAlivePingTimeout =
                                               TimeSpan.FromSeconds(1);
            });
            webBuilder.UseStartup<Startup>();
        });
   ```
   <!-- review: KeepAlivePingInterval not found in RC1. Try testing with RC1. See https://github.com/dotnet/aspnetcore/pull/22565/files see C:/Users/riande/source/repos/WebApplication128/WebApplication128 -->

### <a name="containers"></a>Contenitori

Prima di .NET 5,0, la compilazione e la pubblicazione di un *Dockerfile* per un'app ASP.NET Core richiedevano l'estrazione dell'intera .NET Core SDK e dell'immagine ASP.NET Core. Con questa versione, il pull dei byte delle immagini dell'SDK viene ridotto e i byte estratti per l'immagine di ASP.NET Core vengono eliminati in gran parte. Per altre informazioni, vedere [questo commento sul problema GitHub](https://github.com/dotnet/dotnet-docker/issues/1814#issuecomment-625294750).

## <a name="authentication-and-authorization"></a>Autenticazione e autorizzazione

### <a name="azure-active-directory-authentication-with-microsoftidentityweb"></a>Azure Active Directory l'autenticazione con Microsoft. Identity . Web

I modelli di progetto ASP.NET Core ora si integrano con <xref:Microsoft.Identity.Web?displayProperty=fullName> per gestire l'autenticazione con [Azure Active Directory](/azure/active-directory/fundamentals/active-directory-whatis) (Azure ad). [Microsoft. Identity . Il pacchetto Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) fornisce:

* Un'esperienza migliore per l'autenticazione tramite Azure AD.
* Un modo più semplice per accedere alle risorse di Azure per conto degli utenti, tra cui [Microsoft Graph](/graph/overview). Vedere [Microsoft. Identity . Web Sample](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2), che inizia con un account di accesso di base e passa attraverso il multi-tenant, usando le API di Azure, usando Microsoft Graph e proteggendo le proprie API. `Microsoft.Identity.Web` è disponibile insieme a .NET 5.

### <a name="allow-anonymous-access-to-an-endpoint"></a>Consenti accesso anonimo a un endpoint

Il `AllowAnonymous` metodo di estensione consente l'accesso anonimo a un endpoint:

[!code-csharp[](~/release-notes/sample/StartupAnonEndpoint.cs?name=snippet)]

### <a name="custom-handling-of-authorization-failures"></a>Gestione personalizzata degli errori di autorizzazione

La gestione personalizzata degli errori di autorizzazione è ora più semplice con la nuova interfaccia [IAuthorizationMiddlewareResultHandler](https://github.com/dotnet/aspnetcore/blob/v5.0.0-rc.1.20451.17/src/Security/Authorization/Policy/src/IAuthorizationMiddlewareResultHandler.cs) richiamata dal [middleware](xref:fundamentals/middleware/index)di [autorizzazione](xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A) . L'implementazione predefinita rimane invariata, ma un gestore personalizzato può essere registrato in [Dependency Injection, che consente risposte HTTP personalizzate a seconda del motivo per cui l'autorizzazione non è riuscita. Vedere [questo esempio](https://github.com/dotnet/aspnetcore/blob/main/src/Security/samples/CustomAuthorizationFailureResponse/Authorization/SampleAuthorizationMiddlewareResultHandler.cs) che illustra l'uso di `IAuthorizationMiddlewareResultHandler` .

### <a name="authorization-when-using-endpoint-routing"></a>Autorizzazione quando si usa il routing degli endpoint

L'autorizzazione quando si usa il routing degli endpoint ora riceve `HttpContext` invece l'istanza dell'endpoint. Ciò consente al middleware di autorizzazione di accedere a `RouteData` e ad altre proprietà di `HttpContext` che non erano accessibili tramite la <xref:Microsoft.AspNetCore.Http.Endpoint> classe. L'endpoint può essere recuperato dal contesto utilizzando il [contesto. GetEndPoint](xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint%2A).

### <a name="role-based-access-control-with-kerberos-authentication-and-ldap-on-linux"></a>Controllo degli accessi in base al ruolo con autenticazione Kerberos e LDAP in Linux

Vedere [autenticazione Kerberos e controllo degli accessi in base al ruolo (RBAC)](xref:security/authentication/windowsauth#rbac)

## <a name="api-improvements"></a>Miglioramenti delle API

### <a name="json-extension-methods-for-httprequest-and-httpresponse"></a>Metodi di estensione JSON per HttpRequest e HttpResponse

I dati JSON possono essere letti e scritti da un oggetto `HttpRequest` e `HttpResponse` usando i <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> nuovi `WriteAsJsonAsync` metodi di estensione e. Questi metodi di estensione usano il [System.Text.Jssul](xref:System.Text.Json) serializzatore per gestire i dati JSON. Il nuovo `HasJsonContentType` metodo di estensione può anche controllare se una richiesta ha un tipo di contenuto JSON.

I metodi di estensione JSON possono essere combinati con il [routing degli endpoint](xref:fundamentals/routing) per creare API JSON in uno stile di programmazione che chiamiamo ***route al codice***. Si tratta di una nuova opzione per gli sviluppatori che desiderano creare API JSON di base in modo semplice. Ad esempio, un'app Web che dispone solo di un numero limitato di endpoint può scegliere di usare il routing al codice anziché le funzionalità complete di ASP.NET Core MVC:

```csharp
endpoints.MapGet("/weather/{city:alpha}", async context =>
{
    var city = (string)context.Request.RouteValues["city"];
    var weather = GetFromDatabase(city);

    await context.Response.WriteAsJsonAsync(weather);
});
```

Per altre informazioni sui nuovi metodi di estensione JSON e sul routing al codice, vedere [questa mostra .NET](https://channel9.msdn.com/Shows/On-NET/ASPNET-Core-Series-Route-to-Code).

### <a name="systemdiagnosticsactivity"></a>System. Diagnostics. Activity

Il formato predefinito per <xref:System.Diagnostics.Activity?displayProperty=fullName> ora è il formato W3C. In questo modo, il supporto per la traccia distribuita viene reso ASP.NET Core interoperabile con più Framework per impostazione predefinita.

### <a name="frombodyattribute"></a>FromBodyAttribute

<xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute> supporta ora la configurazione di un'opzione che consente di considerare questi parametri o proprietà come facoltativi:

```csharp
public IActionResult Post([FromBody(EmptyBodyBehavior = EmptyBodyBehavior.Allow)]
                          MyModel model)
{
    ...
}
```

## <a name="miscellaneous-improvements"></a>Miglioramenti vari

È stata avviata l'applicazione di [annotazioni Nullable](/dotnet/csharp/nullable-references#attributes-describe-apis) a ASP.NET Core assembly. Si prevede di aggiungere annotazioni alla maggior parte della superficie API pubblica comune del Framework .NET 5. <!-- Review: what's the impact of this? How does it work? Need more info.  Check the link I added -->

### <a name="control-startup-class-activation"></a>Controllare l'attivazione della classe di avvio

<xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseStartup%2A>È stato aggiunto un overload aggiuntivo che consente a un'app di fornire un metodo factory per controllare l' `Startup` attivazione della classe. Il controllo dell' `Startup` attivazione della classe è utile per passare parametri aggiuntivi a `Startup` che vengono inizializzati insieme all'host:

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var logger = CreateLogger();
        var host = Host.CreateDefaultBuilder()
            .ConfigureWebHost(builder =>
            {
                builder.UseStartup(context => new Startup(logger));
            })
            .Build();

        await host.RunAsync();
    }
}
```

### <a name="auto-refresh-with-dotnet-watch"></a>Aggiornamento automatico con DotNet Watch

In .NET 5 l'esecuzione di [DotNet Watch](xref:tutorials/dotnet-watch) in un progetto di ASP.NET Core avvia il browser predefinito e aggiorna automaticamente il browser in base alle modifiche apportate al codice. Ciò significa che è possibile:

* Aprire un progetto ASP.NET Core in un editor di testo.
* Eseguire `dotnet watch`.
* Concentrarsi sulle modifiche al codice mentre gli strumenti gestiscono la ricompilazione, il riavvio e il ricaricamento dell'app.

### <a name="console-logger-formatter"></a>Formattatore del logger della console

Sono stati apportati miglioramenti al provider di log della console nella `Microsoft.Extensions.Logging` libreria. Gli sviluppatori possono ora implementare un oggetto personalizzato `ConsoleFormatter` per esercitare il controllo completo sulla formattazione e la colorazione dell'output della console. Le API del formattatore consentono la formattazione avanzata implementando un subset delle sequenze di escape VT-100. VT-100 è supportato dalla maggior parte dei terminali moderni. Il logger della console può analizzare sequenze di escape su terminali non supportati che consentono agli sviluppatori di creare un singolo formattatore per tutti i terminali.

### <a name="json-console-logger"></a>Logger console JSON

Oltre al supporto per formattatori personalizzati, è stato aggiunto anche un formattatore JSON incorporato che emette log JSON strutturati nella console. Il codice seguente illustra come passare dal logger predefinito a JSON:

[!code-csharp[](~/release-notes/sample/ProgramJsonLog.cs?name=snippet)]

I messaggi di log emessi alla console sono in formato JSON:

```json
{
  "EventId": 0,
  "LogLevel": "Information",
  "Category": "Microsoft.Hosting.Lifetime",
  "Message": "Now listening on: https://localhost:5001",
  "State": {
    "Message": "Now listening on: https://localhost:5001",
    "address": "https://localhost:5001",
    "{OriginalFormat}": "Now listening on: {address}"
  }
}
```
