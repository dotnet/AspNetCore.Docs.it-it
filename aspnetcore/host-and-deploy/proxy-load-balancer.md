---
title: Configurare ASP.NET Core per l'uso di server proxy e servizi di bilanciamento del carico
author: rick-anderson
description: Informazioni sulla configurazione per le app ospitate dietro server proxy e servizi di bilanciamento del carico, che spesso nascondono informazioni importanti sulle richieste.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: host-and-deploy/proxy-load-balancer
ms.openlocfilehash: e63821743dad565b51d5c2360dcc2fbf0632754f
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530047"
---
# <a name="configure-aspnet-core-to-work-with-proxy-servers-and-load-balancers"></a>Configurare ASP.NET Core per l'uso di server proxy e servizi di bilanciamento del carico

Di [Chris Ross](https://github.com/Tratcher)

::: moniker range=">= aspnetcore-3.0"

Nella configurazione consigliata per ASP.NET Core, l'app è ospitata mediante IIS e il modulo ASP.NET Core, Nginx o Apache. Server proxy, servizi di bilanciamento del carico e altre appliance di rete spesso nascondono informazioni sulla richiesta prima che questa raggiunga l'app:

* Quando le richieste HTTPS vengono trasmesse tramite proxy su HTTP, lo schema originale (HTTPS) viene perso e deve essere inoltrato in un'intestazione.
* Poiché un'app riceve una richiesta dal proxy e non dall'effettiva origine su Internet o nella rete aziendale, anche l'indirizzo IP di origine del client deve essere inoltrato in un'intestazione.

Queste informazioni potrebbero essere importanti per l'elaborazione delle richieste, ad esempio per i reindirizzamenti, l'autenticazione, la generazione di collegamenti, la valutazione dei criteri e la georilevazione dei client.

## <a name="forwarded-headers"></a>Intestazioni inoltrate

Per convenzione, i proxy inoltrano le informazioni nelle intestazioni HTTP.

| Intestazione | Descrizione |
| ------ | ----------- |
| X-Forwarded-For | Contiene informazioni sul client che ha avviato la richiesta e sui proxy successivi in una catena di proxy. Questo parametro può contenere indirizzi IP (e, facoltativamente, numeri di porta). In una catena di server proxy, il primo parametro indica il client in cui è stata eseguita inizialmente la richiesta. Seguono gli identificatori dei proxy successivi. L'ultimo proxy della catena non è incluso nell'elenco dei parametri. L'indirizzo IP dell'ultimo proxy e, facoltativamente, un numero di porta sono disponibili come indirizzo IP remoto a livello di trasporto. |
| X-Forwarded-Proto | Il valore dello schema di origine (HTTP/HTTPS). Il valore può essere anche un elenco di schemi, se la richiesta ha attraversato più proxy. |
| X-Forwarded-Host | Il valore originale del campo dell'intestazione host. I proxy in genere non modificano l'intestazione host. Vedere l'[avviso di sicurezza Microsoft CVE-2018-0787](https://github.com/aspnet/Announcements/issues/295) per informazioni su una vulnerabilità di elevazione dei privilegi che interessa i sistemi in cui il proxy non convalida o limita le intestazioni host a valori di riferimento noti. |

Il middleware delle intestazioni inoltri ( <xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersMiddleware> ) legge queste intestazioni e compila i campi associati in <xref:Microsoft.AspNetCore.Http.HttpContext> .

Il middleware aggiorna:

* [HttpContext. Connection. RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress): set usando il `X-Forwarded-For` valore dell'intestazione. Impostazioni aggiuntive influiscono sul modo in cui il middleware imposta `RemoteIpAddress`. Per informazioni dettagliate, vedere [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options).
* [HttpContext. Request. Scheme](xref:Microsoft.AspNetCore.Http.HttpRequest.Scheme): impostare usando il `X-Forwarded-Proto` valore dell'intestazione.
* [HttpContext. Request. host](xref:Microsoft.AspNetCore.Http.HttpRequest.Host): impostare usando il `X-Forwarded-Host` valore dell'intestazione.

È possibile configurare le [impostazioni predefinite](#forwarded-headers-middleware-options) del middleware delle intestazioni inoltrate. Le impostazioni predefinite sono le seguenti:

* È presente *un solo proxy* tra l'app e l'origine delle richieste.
* Sono configurati indirizzi di loopback solo per reti e proxy noti.
* Le intestazioni inoltrate sono denominate `X-Forwarded-For` e `X-Forwarded-Proto`.

Non tutte le appliance di rete aggiungono le intestazioni `X-Forwarded-For` e `X-Forwarded-Proto` senza alcuna configurazione aggiuntiva. Se le richieste trasmesse tramite proxy non contengono queste intestazioni quando raggiungono l'app, fare riferimento alle indicazioni del produttore dell'appliance. Se l'appliance usa nomi di intestazione diversi da `X-Forwarded-For` e `X-Forwarded-Proto`, impostare le opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> e <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> in modo che corrispondano ai nomi di intestazione usati dall'appliance. Per altre informazioni, vedere [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options) e [Configurazione per un proxy che usa nomi di intestazione diversi](#configuration-for-a-proxy-that-uses-different-header-names).

## <a name="iisiis-express-and-aspnet-core-module"></a>IIS/IIS Express e modulo Core ASP.NET

Il middleware delle intestazioni inoltrate è abilitato per impostazione predefinita dal [middleware di integrazione IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) quando l'app è ospitata [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro IIS e il modulo ASP.NET Core. Il middleware delle intestazioni inoltrate viene attivato per l'esecuzione prima nella pipeline del middleware con una configurazione con restrizioni specifica per il modulo ASP.NET Core per motivi di attendibilità delle intestazioni inoltrate, ad esempio lo [spoofing degli indirizzi IP](https://www.iplocation.net/ip-spoofing). Il middleware è configurato per inoltrare le intestazioni `X-Forwarded-For` e `X-Forwarded-Proto` ed è limitato a un singolo proxy localhost. Se è richiesta una configurazione aggiuntiva, vedere [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options).

## <a name="other-proxy-server-and-load-balancer-scenarios"></a>Altri scenari con server proxy e servizi di bilanciamento del carico

Se non si usa il [middleware di integrazione IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) per l'hosting [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), il middleware delle intestazioni inoltrate non è abilitato per impostazione predefinita. Il middleware delle intestazioni inoltrate deve essere abilitato per consentire a un'app di inoltrare le intestazioni inoltrate del processo con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*>. Dopo aver abilitato il middleware, se non sono specificate opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> per il middleware, le intestazioni [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) predefinite sono [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders).

Configurare il middleware con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> per l'inoltro delle intestazioni `X-Forwarded-For` e `X-Forwarded-Proto` in `Startup.ConfigureServices`.

<a name="fhmo"></a>

### <a name="forwarded-headers-middleware-order"></a>Ordine middleware intestazioni inoltri

Il middleware delle intestazioni con inoltri deve essere eseguito prima di altri middleware. Questo ordine garantisce che il middleware basato sulle intestazioni inoltrate possa usare i valori di intestazione per l'elaborazione. Il middleware delle intestazioni con inoltri può essere eseguito dopo la diagnostica e la gestione degli errori, ma deve essere eseguito prima di chiamare `UseHsts` :

[!code-csharp[](~/host-and-deploy/proxy-load-balancer/3.1samples/Startup.cs?name=snippet&highlight=13-17,25,30)]

In alternativa, chiamare `UseForwardedHeaders` prima di diagnostica:

[!code-csharp[](~/host-and-deploy/proxy-load-balancer/3.1samples/Startup2.cs?name=snippet)]

> [!NOTE]
> Se non vengono specificate <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> in `Startup.ConfigureServices` o direttamente nel metodo di estensione con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*>, le intestazioni predefinite da inoltrare sono [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders). La proprietà <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> deve essere configurata con le intestazioni da inoltrare.

## <a name="nginx-configuration"></a>Configurazione Nginx

Per inoltrare le intestazioni `X-Forwarded-For` e `X-Forwarded-Proto`, vedere <xref:host-and-deploy/linux-nginx#configure-nginx>. Per altre informazioni, vedere [NGINX: Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/) (NGINX: Uso dell'intestazione Forwarded).

## <a name="apache-configuration"></a>Configurazione Apache

`X-Forwarded-For` viene aggiunta automaticamente. Vedere [Apache Module mod_proxy: Reverse Proxy Request Headers](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html#x-headers) (Modulo Apache mod_proxy: Intestazioni delle richieste del proxy inverso). Per informazioni su come inoltrare l'intestazione `X-Forwarded-Proto`, vedere <xref:host-and-deploy/linux-apache#configure-apache>.

## <a name="forwarded-headers-middleware-options"></a>Opzioni del middleware delle intestazioni inoltrate

<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> controllano il comportamento del middleware delle intestazioni inoltrate. L'esempio seguente modifica i valori predefiniti:

* Limitare il numero di voci nelle intestazioni inoltrate a `2`.
* Aggiungere un indirizzo proxy noto di `127.0.10.1`.
* Modificare il nome dell'intestazione inoltrata dal nome predefinito `X-Forwarded-For` a `X-Forwarded-For-My-Custom-Header-Name`.

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardLimit = 2;
    options.KnownProxies.Add(IPAddress.Parse("127.0.10.1"));
    options.ForwardedForHeaderName = "X-Forwarded-For-My-Custom-Header-Name";
});
```

| Opzione | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> | Limita gli host mediante l'intestazione `X-Forwarded-Host` ai valori specificati.<ul><li>I valori vengono confrontati tramite ordinal-ignore-case.</li><li>I numeri di porta devono essere esclusi.</li><li>Se l'elenco è vuoto, sono consentiti tutti gli host.</li><li>Un carattere jolly di primo livello `*` consente tutti gli host non vuoti.</li><li>I caratteri jolly per i sottodomini sono consentiti, ma non corrispondono al dominio radice. Ad esempio, `*.contoso.com` corrisponde al sottodominio `foo.contoso.com`, ma non al dominio radice `contoso.com`.</li><li>I nomi host Unicode sono consentiti ma vengono convertiti in [Punycode](https://tools.ietf.org/html/rfc3492) per la corrispondenza.</li><li>Gli [indirizzi IPv6](https://tools.ietf.org/html/rfc4291) devono includere le parentesi quadre di delimitazione ed essere in [formato convenzionale](https://tools.ietf.org/html/rfc4291#section-2.2), ad esempio `[ABCD:EF01:2345:6789:ABCD:EF01:2345:6789]`. Per gli indirizzi IPv6 non viene applicata la distinzione tra maiuscole e minuscole per la verifica dell'uguaglianza logica tra i diversi formati e non viene eseguita alcuna canonizzazione.</li><li>La mancata limitazione degli host consentiti potrebbe permettere a un utente malintenzionato di eseguire lo spoofing dei collegamenti generati dal servizio.</li></ul>Il valore predefinito è un insieme `IList<string>` vuoto. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XForwardedForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedForHeaderName). Questa opzione viene usata quando il proxy o il server d'inoltro non usa l'intestazione `X-Forwarded-For` ma usa un'altra intestazione per inoltrare le informazioni.<br><br>Il valore predefinito è `X-Forwarded-For`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> | Identifica i server d'inoltro che devono essere elaborati. Vedere [ForwardedHeaders Enum](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) (Enumerazione ForwardedHeaders) per l'elenco dei campi applicabili. I valori tipici assegnati a questa proprietà sono `ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto`.<br><br>Il valore predefinito è [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders). |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHostHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XForwardedHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedHostHeaderName). Questa opzione viene usata quando il proxy o il server d'inoltro non usa l'intestazione `X-Forwarded-Host` ma usa un'altra intestazione per inoltrare le informazioni.<br><br>Il valore predefinito è `X-Forwarded-Host`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XForwardedProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedProtoHeaderName). Questa opzione viene usata quando il proxy o il server d'inoltro non usa l'intestazione `X-Forwarded-Proto` ma usa un'altra intestazione per inoltrare le informazioni.<br><br>Il valore predefinito è `X-Forwarded-Proto`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardLimit> | Limita il numero di voci nelle intestazioni elaborate. Impostare su `null` per disabilitare il limite, ma solo se è configurato `KnownProxies` o `KnownNetworks`. L'impostazione di un valore non `null` è una precauzione (ma non una garanzia) per proteggersi da proxy non configurati correttamente e richieste dannose provenienti da canali laterali in rete.<br><br>Il middleware delle intestazioni inoltrate elabora le intestazioni in ordine inverso da destra a sinistra. Se viene usato il valore predefinito (`1`), viene elaborato solo il valore più a destra delle intestazioni, a meno che non venga aumentato il valore di `ForwardLimit`.<br><br>Il valore predefinito è `1`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks> | Intervalli di indirizzi delle reti note da cui accettare le intestazioni inoltrate. Specificare gli intervalli IP usando la notazione CIDR (Classless Interdomain Routing).<br><br>Se il server usa socket dual mode, gli indirizzi IPv4 vengono forniti in un formato IPv6 (ad esempio, `10.0.0.1` in IPv4 rappresentato in IPv6 come `::ffff:10.0.0.1`). Vedere [IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*). Determinare se questo formato è richiesto esaminando [HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*).<br><br>Il valore predefinito è un oggetto `IList`\<<xref:Microsoft.AspNetCore.HttpOverrides.IPNetwork>> contenente una singola voce per `IPAddress.Loopback`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies> | Indirizzi dei proxy noti da cui accettare le intestazioni inoltrate. Usare `KnownProxies` per specificare le corrispondenze esatte degli indirizzi IP.<br><br>Se il server usa socket dual mode, gli indirizzi IPv4 vengono forniti in un formato IPv6 (ad esempio, `10.0.0.1` in IPv4 rappresentato in IPv6 come `::ffff:10.0.0.1`). Vedere [IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*). Determinare se questo formato è richiesto esaminando [HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*).<br><br>Il valore predefinito è un oggetto `IList`\<<xref:System.Net.IPAddress>> contenente una singola voce per `IPAddress.IPv6Loopback`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalForHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XOriginalForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalForHeaderName).<br><br>Il valore predefinito è `X-Original-For`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalHostHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XOriginalHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalHostHeaderName).<br><br>Il valore predefinito è `X-Original-Host`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalProtoHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XOriginalProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalProtoHeaderName).<br><br>Il valore predefinito è `X-Original-Proto`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.RequireHeaderSymmetry> | Richiedere il numero di valori di intestazione da sincronizzare tra le intestazioni [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) in fase di elaborazione.<br><br>Il valore predefinito in ASP.NET Core 1.x è `true`. Il valore predefinito in ASP.NET Core 2.0 o versione successiva è `false`. |

## <a name="scenarios-and-use-cases"></a>Scenari e casi d'uso

### <a name="when-it-isnt-possible-to-add-forwarded-headers-and-all-requests-are-secure"></a>Quando non è possibile aggiungere le intestazioni inoltrate e tutte le richieste sono sicure

In alcuni casi, potrebbe non essere possibile aggiungere le intestazioni inoltrate alle richieste trasmesse tramite proxy all'app. Se il proxy impone che tutte le richieste esterne pubbliche siano HTTPS, lo schema può essere impostato manualmente in `Startup.Configure` prima di usare qualsiasi tipo di middleware:

```csharp
app.Use((context, next) =>
{
    context.Request.Scheme = "https";
    return next();
});
```

Questo codice può essere disabilitato tramite una variabile di ambiente o un'altra impostazione di configurazione in un ambiente di sviluppo o di gestione temporanea.

### <a name="deal-with-path-base-and-proxies-that-change-the-request-path"></a>Gestire la base del percorso e i proxy che modificano il percorso della richiesta

Alcuni proxy passano il percorso senza modifiche, ma con un percorso di base dell'app che deve essere rimosso per consentire il corretto funzionamento del routing. Il middleware [UsePathBaseExtensions.UsePathBase](xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*) suddivide il percorso in [HttpRequest.Path](xref:Microsoft.AspNetCore.Http.HttpRequest.Path) e il percorso di base dell'app in [HttpRequest.PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase).

Se `/foo` è il percorso di base dell'app per un percorso proxy passato come `/foo/api/1`, il middleware imposta `Request.PathBase` su `/foo` e `Request.Path` su `/api/1` con il comando seguente:

```csharp
app.UsePathBase("/foo");
```

Il percorso originale e la base del percorso vengono riapplicati quando il middleware viene chiamato nuovamente in direzione inversa. Per altre informazioni dell'elaborazione degli ordini del middleware, vedere <xref:fundamentals/middleware/index>.

Se il proxy taglia il percorso (ad esempio, inoltrando `/foo/api/1` a `/api/1`), correggere i reindirizzamenti e i collegamenti impostando la proprietà [PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase) della richiesta:

```csharp
app.Use((context, next) =>
{
    context.Request.PathBase = new PathString("/foo");
    return next();
});
```

Se il proxy aggiunge i dati del percorso, eliminare parte del percorso per correggere i reindirizzamenti e i collegamenti usando <xref:Microsoft.AspNetCore.Http.PathString.StartsWithSegments*> ed eseguendo l'assegnazione alla proprietà <xref:Microsoft.AspNetCore.Http.HttpRequest.Path>:

```csharp
app.Use((context, next) =>
{
    if (context.Request.Path.StartsWithSegments("/foo", out var remainder))
    {
        context.Request.Path = remainder;
    }

    return next();
});
```

### <a name="configuration-for-a-proxy-that-uses-different-header-names"></a>Configurazione per un proxy che usa nomi di intestazione diversi

Se il proxy non usa intestazioni denominate `X-Forwarded-For` e `X-Forwarded-Proto` per inoltrare l'indirizzo proxy o la porta e le informazioni dello scherma originali, impostare le opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> e <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> in modo che corrispondano ai nomi di intestazione usati dal proxy:

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedForHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-For_Header";
    options.ForwardedProtoHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-Proto_Header";
});
```

## <a name="forward-the-scheme-for-linux-and-non-iis-reverse-proxies"></a>Inoltrare lo schema per Linux e proxy inversi non IIS

Con le app che chiamano <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*> e <xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*> il sito si ritrova in un ciclo infinito se distribuito in un servizio app Linux di Azure, in una macchina virtuale Linux di Azure o dietro eventuali altri proxy inversi diversi da IIS. TLS viene terminato dal proxy inverso e Kestrel non viene a conoscenza dello schema di richiesta corretto. Si verificano errori anche per OAuth e OIDC in questa configurazione perché generano reindirizzamenti non corretti. <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> aggiunge e configura il middleware delle intestazioni inoltrate in caso di esecuzione dietro IIS, ma non esiste alcuna configurazione automatica corrispondente per Linux (integrazione di Apache o Nginx).

Per inoltrare lo schema dal proxy in scenari non IIS, aggiungere e configurare il middleware delle intestazioni inoltrate. In `Startup.ConfigureServices` usare il codice seguente:

```csharp
// using Microsoft.AspNetCore.HttpOverrides;

if (string.Equals(
    Environment.GetEnvironmentVariable("ASPNETCORE_FORWARDEDHEADERS_ENABLED"), 
    "true", StringComparison.OrdinalIgnoreCase))
{
    services.Configure<ForwardedHeadersOptions>(options =>
    {
        options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | 
            ForwardedHeaders.XForwardedProto;
        // Only loopback proxies are allowed by default.
        // Clear that restriction because forwarders are enabled by explicit 
        // configuration.
        options.KnownNetworks.Clear();
        options.KnownProxies.Clear();
    });
}
```

## <a name="certificate-forwarding"></a>Inoltro di certificati 

### <a name="azure"></a>Azure

Per configurare app Azure servizio per l'invio di certificati, vedere [configurare l'autenticazione reciproca TLS per il servizio app Azure](/azure/app-service/app-service-web-configure-tls-mutual-auth). Le linee guida seguenti riguardano la configurazione dell'app ASP.NET Core.

In `Startup.Configure` aggiungere il codice seguente prima della chiamata a `app.UseAuthentication();` :

```csharp
app.UseCertificateForwarding();
```


Configurare il middleware di invio del certificato per specificare il nome dell'intestazione usato da Azure. In `Startup.ConfigureServices` aggiungere il codice seguente per configurare l'intestazione da cui il middleware compila un certificato:

```csharp
services.AddCertificateForwarding(options =>
    options.CertificateHeader = "X-ARR-ClientCert");
```

### <a name="other-web-proxies"></a>Altri proxy Web

Se viene usato un proxy che non è IIS o app Azure Application Request Routing (ARR) del servizio, configurare il proxy in modo che inoltri il certificato ricevuto in un'intestazione HTTP. In `Startup.Configure` aggiungere il codice seguente prima della chiamata a `app.UseAuthentication();` :

```csharp
app.UseCertificateForwarding();
```

Configurare il middleware di invio del certificato per specificare il nome dell'intestazione. In `Startup.ConfigureServices` aggiungere il codice seguente per configurare l'intestazione da cui il middleware compila un certificato:

```csharp
services.AddCertificateForwarding(options =>
    options.CertificateHeader = "YOUR_CERTIFICATE_HEADER_NAME");
```

Se il proxy non prevede la codifica Base64 del certificato (come nel caso di nginx), impostare l' `HeaderConverter` opzione. Si consideri l'esempio seguente in `Startup.ConfigureServices`:

```csharp
services.AddCertificateForwarding(options =>
{
    options.CertificateHeader = "YOUR_CUSTOM_HEADER_NAME";
    options.HeaderConverter = (headerValue) => 
    {
        var clientCertificate = 
           /* some conversion logic to create an X509Certificate2 */
        return clientCertificate;
    }
});
```

## <a name="troubleshoot"></a>Risolvere problemi

Quando le intestazioni non vengono inoltrate come previsto, abilitare la [registrazione](xref:fundamentals/logging/index). Se i log non forniscono informazioni sufficienti per risolvere il problema, enumerare le intestazioni delle richieste ricevute dal server. Usare il middleware inline per scrivere le intestazioni di richiesta in una risposta dell'app o per registrare le intestazioni. 

Per scrivere le intestazioni nella risposta dell'app, aggiungere il middleware inline di terminale seguente immediatamente dopo la chiamata a <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> in `Startup.Configure`:

```csharp
app.Run(async (context) =>
{
    context.Response.ContentType = "text/plain";

    // Request method, scheme, and path
    await context.Response.WriteAsync(
        $"Request Method: {context.Request.Method}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Scheme: {context.Request.Scheme}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Path: {context.Request.Path}{Environment.NewLine}");

    // Headers
    await context.Response.WriteAsync($"Request Headers:{Environment.NewLine}");

    foreach (var header in context.Request.Headers)
    {
        await context.Response.WriteAsync($"{header.Key}: " +
            $"{header.Value}{Environment.NewLine}");
    }

    await context.Response.WriteAsync(Environment.NewLine);

    // Connection: RemoteIp
    await context.Response.WriteAsync(
        $"Request RemoteIp: {context.Connection.RemoteIpAddress}");
});
```

È possibile scrivere nei log invece che nel corpo della risposta. La scrittura nei log permette il normale funzionamento del sito durante il debug.

Per scrivere nei log invece che nel corpo della risposta:

* Inserire `ILogger<Startup>` nella classe `Startup` come descritto in [Creare log in Startup](xref:fundamentals/logging/index#create-logs-in-startup).
* Aggiungere il middleware inline seguente immediatamente dopo la chiamata a <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> in `Startup.Configure`.

```csharp
app.Use(async (context, next) =>
{
    // Request method, scheme, and path
    _logger.LogDebug("Request Method: {Method}", context.Request.Method);
    _logger.LogDebug("Request Scheme: {Scheme}", context.Request.Scheme);
    _logger.LogDebug("Request Path: {Path}", context.Request.Path);

    // Headers
    foreach (var header in context.Request.Headers)
    {
        _logger.LogDebug("Header: {Key}: {Value}", header.Key, header.Value);
    }

    // Connection: RemoteIp
    _logger.LogDebug("Request RemoteIp: {RemoteIpAddress}", 
        context.Connection.RemoteIpAddress);

    await next();
});
```

Durante l'elaborazione, i valori di `X-Forwarded-{For|Proto|Host}` vengono spostati in `X-Original-{For|Proto|Host}`. Se sono presenti più valori in una determinata intestazione, il middleware delle intestazioni inoltrate elabora le intestazioni in ordine inverso da destra a sinistra. Il valore predefinito di `ForwardLimit` è `1` (uno) e, pertanto, viene elaborato solo il valore più a destra delle intestazioni, a meno che non venga aumentato il valore di `ForwardLimit`.

L'indirizzo IP remoto originale della richiesta deve corrispondere a una voce negli elenchi `KnownProxies` o `KnownNetworks` prima dell'elaborazione delle intestazioni inoltrate. Questo riduce lo spoofing delle intestazioni, non accettando server di inoltro da proxy non attendibili. Quando viene rilevato un proxy sconosciuto, la registrazione indica l'indirizzo del proxy:

```console
September 20th 2018, 15:49:44.168 Unknown proxy: 10.0.0.100:54321
```

Nell'esempio precedente, 10.0.0.100 è un server proxy. Se il server è un proxy attendibile, aggiungere l'indirizzo IP del server a `KnownProxies` (o aggiungere una rete attendibile a `KnownNetworks`) in `Startup.ConfigureServices`. Per altre informazioni, vedere la sezione [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options).

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.100"));
});
```

> [!IMPORTANT]
> Consentire solo a reti e proxy attendibili di inoltrare le intestazioni. In caso contrario, possono verificarsi attacchi di [spoofing degli indirizzi IP](https://www.iplocation.net/ip-spoofing).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:host-and-deploy/web-farm>
* [Avviso di sicurezza Microsoft CVE-2018-0787: vulnerabilità di elevazione dei privilegi di ASP.NET Core](https://github.com/aspnet/Announcements/issues/295)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Nella configurazione consigliata per ASP.NET Core, l'app è ospitata mediante IIS e il modulo ASP.NET Core, Nginx o Apache. Server proxy, servizi di bilanciamento del carico e altre appliance di rete spesso nascondono informazioni sulla richiesta prima che questa raggiunga l'app:

* Quando le richieste HTTPS vengono trasmesse tramite proxy su HTTP, lo schema originale (HTTPS) viene perso e deve essere inoltrato in un'intestazione.
* Poiché un'app riceve una richiesta dal proxy e non dall'effettiva origine su Internet o nella rete aziendale, anche l'indirizzo IP di origine del client deve essere inoltrato in un'intestazione.

Queste informazioni potrebbero essere importanti per l'elaborazione delle richieste, ad esempio per i reindirizzamenti, l'autenticazione, la generazione di collegamenti, la valutazione dei criteri e la georilevazione dei client.

## <a name="forwarded-headers"></a>Intestazioni inoltrate

Per convenzione, i proxy inoltrano le informazioni nelle intestazioni HTTP.

| Intestazione | Descrizione |
| ------ | ----------- |
| X-Forwarded-For | Contiene informazioni sul client che ha avviato la richiesta e sui proxy successivi in una catena di proxy. Questo parametro può contenere indirizzi IP (e, facoltativamente, numeri di porta). In una catena di server proxy, il primo parametro indica il client in cui è stata eseguita inizialmente la richiesta. Seguono gli identificatori dei proxy successivi. L'ultimo proxy della catena non è incluso nell'elenco dei parametri. L'indirizzo IP dell'ultimo proxy e, facoltativamente, un numero di porta sono disponibili come indirizzo IP remoto a livello di trasporto. |
| X-Forwarded-Proto | Il valore dello schema di origine (HTTP/HTTPS). Il valore può essere anche un elenco di schemi, se la richiesta ha attraversato più proxy. |
| X-Forwarded-Host | Il valore originale del campo dell'intestazione host. I proxy in genere non modificano l'intestazione host. Vedere l'[avviso di sicurezza Microsoft CVE-2018-0787](https://github.com/aspnet/Announcements/issues/295) per informazioni su una vulnerabilità di elevazione dei privilegi che interessa i sistemi in cui il proxy non convalida o limita le intestazioni host a valori di riferimento noti. |

Il middleware delle intestazioni inoltrate, dal pacchetto [Microsoft.AspNetCore.HttpOverrides](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides/), legge le intestazioni e compila i campi associati in <xref:Microsoft.AspNetCore.Http.HttpContext>.

Il middleware aggiorna:

* [HttpContext. Connection. RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress): set usando il `X-Forwarded-For` valore dell'intestazione. Impostazioni aggiuntive influiscono sul modo in cui il middleware imposta `RemoteIpAddress`. Per informazioni dettagliate, vedere [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options).
* [HttpContext. Request. Scheme](xref:Microsoft.AspNetCore.Http.HttpRequest.Scheme): impostare usando il `X-Forwarded-Proto` valore dell'intestazione.
* [HttpContext. Request. host](xref:Microsoft.AspNetCore.Http.HttpRequest.Host): impostare usando il `X-Forwarded-Host` valore dell'intestazione.

È possibile configurare le [impostazioni predefinite](#forwarded-headers-middleware-options) del middleware delle intestazioni inoltrate. Le impostazioni predefinite sono le seguenti:

* È presente *un solo proxy* tra l'app e l'origine delle richieste.
* Sono configurati indirizzi di loopback solo per reti e proxy noti.
* Le intestazioni inoltrate sono denominate `X-Forwarded-For` e `X-Forwarded-Proto`.

Non tutte le appliance di rete aggiungono le intestazioni `X-Forwarded-For` e `X-Forwarded-Proto` senza alcuna configurazione aggiuntiva. Se le richieste trasmesse tramite proxy non contengono queste intestazioni quando raggiungono l'app, fare riferimento alle indicazioni del produttore dell'appliance. Se l'appliance usa nomi di intestazione diversi da `X-Forwarded-For` e `X-Forwarded-Proto`, impostare le opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> e <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> in modo che corrispondano ai nomi di intestazione usati dall'appliance. Per altre informazioni, vedere [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options) e [Configurazione per un proxy che usa nomi di intestazione diversi](#configuration-for-a-proxy-that-uses-different-header-names).

## <a name="iisiis-express-and-aspnet-core-module"></a>IIS/IIS Express e modulo Core ASP.NET

Il middleware delle intestazioni inoltrate è abilitato per impostazione predefinita dal [middleware di integrazione IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) quando l'app è ospitata [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro IIS e il modulo ASP.NET Core. Il middleware delle intestazioni inoltrate viene attivato per l'esecuzione prima nella pipeline del middleware con una configurazione con restrizioni specifica per il modulo ASP.NET Core per motivi di attendibilità delle intestazioni inoltrate, ad esempio lo [spoofing degli indirizzi IP](https://www.iplocation.net/ip-spoofing). Il middleware è configurato per inoltrare le intestazioni `X-Forwarded-For` e `X-Forwarded-Proto` ed è limitato a un singolo proxy localhost. Se è richiesta una configurazione aggiuntiva, vedere [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options).

## <a name="other-proxy-server-and-load-balancer-scenarios"></a>Altri scenari con server proxy e servizi di bilanciamento del carico

Se non si usa il [middleware di integrazione IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) per l'hosting [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), il middleware delle intestazioni inoltrate non è abilitato per impostazione predefinita. Il middleware delle intestazioni inoltrate deve essere abilitato per consentire a un'app di inoltrare le intestazioni inoltrate del processo con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*>. Dopo aver abilitato il middleware, se non sono specificate opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> per il middleware, le intestazioni [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) predefinite sono [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders).

Configurare il middleware con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> per l'inoltro delle intestazioni `X-Forwarded-For` e `X-Forwarded-Proto` in `Startup.ConfigureServices`. Richiamare il metodo <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> in `Startup.Configure` prima di chiamare altro middleware:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();

    services.Configure<ForwardedHeadersOptions>(options =>
    {
        options.ForwardedHeaders = 
            ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    });
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseForwardedHeaders();

    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
    }

    app.UseStaticFiles();
    // In ASP.NET Core 1.x, replace the following line with: app.UseIdentity();
    app.UseAuthentication();
    app.UseMvc();
}
```

> [!NOTE]
> Se non vengono specificate <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> in `Startup.ConfigureServices` o direttamente nel metodo di estensione con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*>, le intestazioni predefinite da inoltrare sono [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders). La proprietà <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> deve essere configurata con le intestazioni da inoltrare.

## <a name="nginx-configuration"></a>Configurazione Nginx

Per inoltrare le intestazioni `X-Forwarded-For` e `X-Forwarded-Proto`, vedere <xref:host-and-deploy/linux-nginx#configure-nginx>. Per altre informazioni, vedere [NGINX: Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/) (NGINX: Uso dell'intestazione Forwarded).

## <a name="apache-configuration"></a>Configurazione Apache

`X-Forwarded-For` viene aggiunta automaticamente. Vedere [Apache Module mod_proxy: Reverse Proxy Request Headers](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html#x-headers) (Modulo Apache mod_proxy: Intestazioni delle richieste del proxy inverso). Per informazioni su come inoltrare l'intestazione `X-Forwarded-Proto`, vedere <xref:host-and-deploy/linux-apache#configure-apache>.

## <a name="forwarded-headers-middleware-options"></a>Opzioni del middleware delle intestazioni inoltrate

<xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> controllano il comportamento del middleware delle intestazioni inoltrate. L'esempio seguente modifica i valori predefiniti:

* Limitare il numero di voci nelle intestazioni inoltrate a `2`.
* Aggiungere un indirizzo proxy noto di `127.0.10.1`.
* Modificare il nome dell'intestazione inoltrata dal nome predefinito `X-Forwarded-For` a `X-Forwarded-For-My-Custom-Header-Name`.

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardLimit = 2;
    options.KnownProxies.Add(IPAddress.Parse("127.0.10.1"));
    options.ForwardedForHeaderName = "X-Forwarded-For-My-Custom-Header-Name";
});
```

| Opzione | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> | Limita gli host mediante l'intestazione `X-Forwarded-Host` ai valori specificati.<ul><li>I valori vengono confrontati tramite ordinal-ignore-case.</li><li>I numeri di porta devono essere esclusi.</li><li>Se l'elenco è vuoto, sono consentiti tutti gli host.</li><li>Un carattere jolly di primo livello `*` consente tutti gli host non vuoti.</li><li>I caratteri jolly per i sottodomini sono consentiti, ma non corrispondono al dominio radice. Ad esempio, `*.contoso.com` corrisponde al sottodominio `foo.contoso.com`, ma non al dominio radice `contoso.com`.</li><li>I nomi host Unicode sono consentiti ma vengono convertiti in [Punycode](https://tools.ietf.org/html/rfc3492) per la corrispondenza.</li><li>Gli [indirizzi IPv6](https://tools.ietf.org/html/rfc4291) devono includere le parentesi quadre di delimitazione ed essere in [formato convenzionale](https://tools.ietf.org/html/rfc4291#section-2.2), ad esempio `[ABCD:EF01:2345:6789:ABCD:EF01:2345:6789]`. Per gli indirizzi IPv6 non viene applicata la distinzione tra maiuscole e minuscole per la verifica dell'uguaglianza logica tra i diversi formati e non viene eseguita alcuna canonizzazione.</li><li>La mancata limitazione degli host consentiti potrebbe permettere a un utente malintenzionato di eseguire lo spoofing dei collegamenti generati dal servizio.</li></ul>Il valore predefinito è un insieme `IList<string>` vuoto. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XForwardedForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedForHeaderName). Questa opzione viene usata quando il proxy o il server d'inoltro non usa l'intestazione `X-Forwarded-For` ma usa un'altra intestazione per inoltrare le informazioni.<br><br>Il valore predefinito è `X-Forwarded-For`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders> | Identifica i server d'inoltro che devono essere elaborati. Vedere [ForwardedHeaders Enum](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders) (Enumerazione ForwardedHeaders) per l'elenco dei campi applicabili. I valori tipici assegnati a questa proprietà sono `ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto`.<br><br>Il valore predefinito è [ForwardedHeaders.None](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeaders). |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHostHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XForwardedHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedHostHeaderName). Questa opzione viene usata quando il proxy o il server d'inoltro non usa l'intestazione `X-Forwarded-Host` ma usa un'altra intestazione per inoltrare le informazioni.<br><br>Il valore predefinito è `X-Forwarded-Host`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XForwardedProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XForwardedProtoHeaderName). Questa opzione viene usata quando il proxy o il server d'inoltro non usa l'intestazione `X-Forwarded-Proto` ma usa un'altra intestazione per inoltrare le informazioni.<br><br>Il valore predefinito è `X-Forwarded-Proto`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardLimit> | Limita il numero di voci nelle intestazioni elaborate. Impostare su `null` per disabilitare il limite, ma solo se è configurato `KnownProxies` o `KnownNetworks`. L'impostazione di un valore non `null` è una precauzione (ma non una garanzia) per proteggersi da proxy non configurati correttamente e richieste dannose provenienti da canali laterali in rete.<br><br>Il middleware delle intestazioni inoltrate elabora le intestazioni in ordine inverso da destra a sinistra. Se viene usato il valore predefinito (`1`), viene elaborato solo il valore più a destra delle intestazioni, a meno che non venga aumentato il valore di `ForwardLimit`.<br><br>Il valore predefinito è `1`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks> | Intervalli di indirizzi delle reti note da cui accettare le intestazioni inoltrate. Specificare gli intervalli IP usando la notazione CIDR (Classless Interdomain Routing).<br><br>Se il server usa socket dual mode, gli indirizzi IPv4 vengono forniti in un formato IPv6 (ad esempio, `10.0.0.1` in IPv4 rappresentato in IPv6 come `::ffff:10.0.0.1`). Vedere [IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*). Determinare se questo formato è richiesto esaminando [HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*). Per altre informazioni, vedere la sezione [Configurazione per un indirizzo IPv4 rappresentato come indirizzo IPv6](#configuration-for-an-ipv4-address-represented-as-an-ipv6-address).<br><br>Il valore predefinito è un oggetto `IList`\<<xref:Microsoft.AspNetCore.HttpOverrides.IPNetwork>> contenente una singola voce per `IPAddress.Loopback`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies> | Indirizzi dei proxy noti da cui accettare le intestazioni inoltrate. Usare `KnownProxies` per specificare le corrispondenze esatte degli indirizzi IP.<br><br>Se il server usa socket dual mode, gli indirizzi IPv4 vengono forniti in un formato IPv6 (ad esempio, `10.0.0.1` in IPv4 rappresentato in IPv6 come `::ffff:10.0.0.1`). Vedere [IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*). Determinare se questo formato è richiesto esaminando [HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*). Per altre informazioni, vedere la sezione [Configurazione per un indirizzo IPv4 rappresentato come indirizzo IPv6](#configuration-for-an-ipv4-address-represented-as-an-ipv6-address).<br><br>Il valore predefinito è un oggetto `IList`\<<xref:System.Net.IPAddress>> contenente una singola voce per `IPAddress.IPv6Loopback`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalForHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XOriginalForHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalForHeaderName).<br><br>Il valore predefinito è `X-Original-For`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalHostHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XOriginalHostHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalHostHeaderName).<br><br>Il valore predefinito è `X-Original-Host`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.OriginalProtoHeaderName> | Usare l'intestazione specificata da questa proprietà anziché quella specificata da [ForwardedHeadersDefaults.XOriginalProtoHeaderName](xref:Microsoft.AspNetCore.HttpOverrides.ForwardedHeadersDefaults.XOriginalProtoHeaderName).<br><br>Il valore predefinito è `X-Original-Proto`. |
| <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.RequireHeaderSymmetry> | Richiedere il numero di valori di intestazione da sincronizzare tra le intestazioni [ForwardedHeadersOptions.ForwardedHeaders](xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedHeaders) in fase di elaborazione.<br><br>Il valore predefinito in ASP.NET Core 1.x è `true`. Il valore predefinito in ASP.NET Core 2.0 o versione successiva è `false`. |

## <a name="scenarios-and-use-cases"></a>Scenari e casi d'uso

### <a name="when-it-isnt-possible-to-add-forwarded-headers-and-all-requests-are-secure"></a>Quando non è possibile aggiungere le intestazioni inoltrate e tutte le richieste sono sicure

In alcuni casi, potrebbe non essere possibile aggiungere le intestazioni inoltrate alle richieste trasmesse tramite proxy all'app. Se il proxy impone che tutte le richieste esterne pubbliche siano HTTPS, lo schema può essere impostato manualmente in `Startup.Configure` prima di usare qualsiasi tipo di middleware:

```csharp
app.Use((context, next) =>
{
    context.Request.Scheme = "https";
    return next();
});
```

Questo codice può essere disabilitato tramite una variabile di ambiente o un'altra impostazione di configurazione in un ambiente di sviluppo o di gestione temporanea.

### <a name="deal-with-path-base-and-proxies-that-change-the-request-path"></a>Gestire la base del percorso e i proxy che modificano il percorso della richiesta

Alcuni proxy passano il percorso senza modifiche, ma con un percorso di base dell'app che deve essere rimosso per consentire il corretto funzionamento del routing. Il middleware [UsePathBaseExtensions.UsePathBase](xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*) suddivide il percorso in [HttpRequest.Path](xref:Microsoft.AspNetCore.Http.HttpRequest.Path) e il percorso di base dell'app in [HttpRequest.PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase).

Se `/foo` è il percorso di base dell'app per un percorso proxy passato come `/foo/api/1`, il middleware imposta `Request.PathBase` su `/foo` e `Request.Path` su `/api/1` con il comando seguente:

```csharp
app.UsePathBase("/foo");
```

Il percorso originale e la base del percorso vengono riapplicati quando il middleware viene chiamato nuovamente in direzione inversa. Per altre informazioni dell'elaborazione degli ordini del middleware, vedere <xref:fundamentals/middleware/index>.

Se il proxy taglia il percorso (ad esempio, inoltrando `/foo/api/1` a `/api/1`), correggere i reindirizzamenti e i collegamenti impostando la proprietà [PathBase](xref:Microsoft.AspNetCore.Http.HttpRequest.PathBase) della richiesta:

```csharp
app.Use((context, next) =>
{
    context.Request.PathBase = new PathString("/foo");
    return next();
});
```

Se il proxy aggiunge i dati del percorso, eliminare parte del percorso per correggere i reindirizzamenti e i collegamenti usando <xref:Microsoft.AspNetCore.Http.PathString.StartsWithSegments*> ed eseguendo l'assegnazione alla proprietà <xref:Microsoft.AspNetCore.Http.HttpRequest.Path>:

```csharp
app.Use((context, next) =>
{
    if (context.Request.Path.StartsWithSegments("/foo", out var remainder))
    {
        context.Request.Path = remainder;
    }

    return next();
});
```

### <a name="configuration-for-a-proxy-that-uses-different-header-names"></a>Configurazione per un proxy che usa nomi di intestazione diversi

Se il proxy non usa intestazioni denominate `X-Forwarded-For` e `X-Forwarded-Proto` per inoltrare l'indirizzo proxy o la porta e le informazioni dello scherma originali, impostare le opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedForHeaderName> e <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.ForwardedProtoHeaderName> in modo che corrispondano ai nomi di intestazione usati dal proxy:

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedForHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-For_Header";
    options.ForwardedProtoHeaderName = "Header_Name_Used_By_Proxy_For_X-Forwarded-Proto_Header";
});
```

### <a name="configuration-for-an-ipv4-address-represented-as-an-ipv6-address"></a>Configurazione per un indirizzo IPv4 rappresentato come indirizzo IPv6

Se il server usa socket dual mode, gli indirizzi IPv4 vengono forniti in un formato IPv6 (ad esempio, `10.0.0.1` in IPv4 rappresentato in IPv6 come `::ffff:10.0.0.1` o `::ffff:a00:1`). Vedere [IPAddress.MapToIPv6](xref:System.Net.IPAddress.MapToIPv6*). Determinare se questo formato è richiesto esaminando [HttpContext.Connection.RemoteIpAddress](xref:Microsoft.AspNetCore.Http.ConnectionInfo.RemoteIpAddress*).

Nell'esempio seguente un indirizzo di rete che fornisce intestazioni inoltrate viene aggiunto all'elenco `KnownNetworks` in formato IPv6.

Indirizzo IPv4: `10.11.12.1/8`

Indirizzo IPv6 convertito: `::ffff:10.11.12.1`  
Lunghezza del prefisso convertito: 104

È anche possibile specificare l'indirizzo in formato esadecimale (`10.11.12.1` rappresentato in IPv6 come `::ffff:0a0b:0c01`). Durante la conversione di un indirizzo IPv4 in IPv6, aggiungere 96 alla lunghezza del prefisso CIDR (`8` nell'esempio) per tenere conto del prefisso IPv6 aggiuntivo `::ffff:` (8 + 96 = 104). 

```csharp
// To access IPNetwork and IPAddress, add the following namespaces:
// using System.Net;
// using Microsoft.AspNetCore.HttpOverrides;
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders =
        ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Add(new IPNetwork(
        IPAddress.Parse("::ffff:10.11.12.1"), 104));
});
```

## <a name="forward-the-scheme-for-linux-and-non-iis-reverse-proxies"></a>Inoltrare lo schema per Linux e proxy inversi non IIS

Con le app che chiamano <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*> e <xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*> il sito si ritrova in un ciclo infinito se distribuito in un servizio app Linux di Azure, in una macchina virtuale Linux di Azure o dietro eventuali altri proxy inversi diversi da IIS. TLS viene terminato dal proxy inverso e Kestrel non viene a conoscenza dello schema di richiesta corretto. Si verificano errori anche per OAuth e OIDC in questa configurazione perché generano reindirizzamenti non corretti. <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> aggiunge e configura il middleware delle intestazioni inoltrate in caso di esecuzione dietro IIS, ma non esiste alcuna configurazione automatica corrispondente per Linux (integrazione di Apache o Nginx).

Per inoltrare lo schema dal proxy in scenari non IIS, aggiungere e configurare il middleware delle intestazioni inoltrate. In `Startup.ConfigureServices` usare il codice seguente:

```csharp
// using Microsoft.AspNetCore.HttpOverrides;

if (string.Equals(
    Environment.GetEnvironmentVariable("ASPNETCORE_FORWARDEDHEADERS_ENABLED"), 
    "true", StringComparison.OrdinalIgnoreCase))
{
    services.Configure<ForwardedHeadersOptions>(options =>
    {
        options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | 
            ForwardedHeaders.XForwardedProto;
        // Only loopback proxies are allowed by default.
        // Clear that restriction because forwarders are enabled by explicit 
        // configuration.
        options.KnownNetworks.Clear();
        options.KnownProxies.Clear();
    });
}
```

## <a name="troubleshoot"></a>Risolvere problemi

Quando le intestazioni non vengono inoltrate come previsto, abilitare la [registrazione](xref:fundamentals/logging/index). Se i log non forniscono informazioni sufficienti per risolvere il problema, enumerare le intestazioni delle richieste ricevute dal server. Usare il middleware inline per scrivere le intestazioni di richiesta in una risposta dell'app o per registrare le intestazioni. 

Per scrivere le intestazioni nella risposta dell'app, aggiungere il middleware inline di terminale seguente immediatamente dopo la chiamata a <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> in `Startup.Configure`:

```csharp
app.Run(async (context) =>
{
    context.Response.ContentType = "text/plain";

    // Request method, scheme, and path
    await context.Response.WriteAsync(
        $"Request Method: {context.Request.Method}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Scheme: {context.Request.Scheme}{Environment.NewLine}");
    await context.Response.WriteAsync(
        $"Request Path: {context.Request.Path}{Environment.NewLine}");

    // Headers
    await context.Response.WriteAsync($"Request Headers:{Environment.NewLine}");

    foreach (var header in context.Request.Headers)
    {
        await context.Response.WriteAsync($"{header.Key}: " +
            $"{header.Value}{Environment.NewLine}");
    }

    await context.Response.WriteAsync(Environment.NewLine);

    // Connection: RemoteIp
    await context.Response.WriteAsync(
        $"Request RemoteIp: {context.Connection.RemoteIpAddress}");
});
```

È possibile scrivere nei log invece che nel corpo della risposta. La scrittura nei log permette il normale funzionamento del sito durante il debug.

Per scrivere nei log invece che nel corpo della risposta:

* Inserire `ILogger<Startup>` nella classe `Startup` come descritto in [Creare log in Startup](xref:fundamentals/logging/index#create-logs-in-startup).
* Aggiungere il middleware inline seguente immediatamente dopo la chiamata a <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> in `Startup.Configure`.

```csharp
app.Use(async (context, next) =>
{
    // Request method, scheme, and path
    _logger.LogDebug("Request Method: {Method}", context.Request.Method);
    _logger.LogDebug("Request Scheme: {Scheme}", context.Request.Scheme);
    _logger.LogDebug("Request Path: {Path}", context.Request.Path);

    // Headers
    foreach (var header in context.Request.Headers)
    {
        _logger.LogDebug("Header: {Key}: {Value}", header.Key, header.Value);
    }

    // Connection: RemoteIp
    _logger.LogDebug("Request RemoteIp: {RemoteIpAddress}", 
        context.Connection.RemoteIpAddress);

    await next();
});
```

Durante l'elaborazione, i valori di `X-Forwarded-{For|Proto|Host}` vengono spostati in `X-Original-{For|Proto|Host}`. Se sono presenti più valori in una determinata intestazione, il middleware delle intestazioni inoltrate elabora le intestazioni in ordine inverso da destra a sinistra. Il valore predefinito di `ForwardLimit` è `1` (uno) e, pertanto, viene elaborato solo il valore più a destra delle intestazioni, a meno che non venga aumentato il valore di `ForwardLimit`.

L'indirizzo IP remoto originale della richiesta deve corrispondere a una voce negli elenchi `KnownProxies` o `KnownNetworks` prima dell'elaborazione delle intestazioni inoltrate. Questo riduce lo spoofing delle intestazioni, non accettando server di inoltro da proxy non attendibili. Quando viene rilevato un proxy sconosciuto, la registrazione indica l'indirizzo del proxy:

```console
September 20th 2018, 15:49:44.168 Unknown proxy: 10.0.0.100:54321
```

Nell'esempio precedente, 10.0.0.100 è un server proxy. Se il server è un proxy attendibile, aggiungere l'indirizzo IP del server a `KnownProxies` (o aggiungere una rete attendibile a `KnownNetworks`) in `Startup.ConfigureServices`. Per altre informazioni, vedere la sezione [Opzioni del middleware delle intestazioni inoltrate](#forwarded-headers-middleware-options).

```csharp
services.Configure<ForwardedHeadersOptions>(options =>
{
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.100"));
});
```

> [!IMPORTANT]
> Consentire solo a reti e proxy attendibili di inoltrare le intestazioni. In caso contrario, possono verificarsi attacchi di [spoofing degli indirizzi IP](https://www.iplocation.net/ip-spoofing).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:host-and-deploy/web-farm>
* [Avviso di sicurezza Microsoft CVE-2018-0787: vulnerabilità di elevazione dei privilegi di ASP.NET Core](https://github.com/aspnet/Announcements/issues/295)

::: moniker-end
