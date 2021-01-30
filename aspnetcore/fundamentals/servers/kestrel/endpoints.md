---
title: Configurare gli endpoint per il server Web ASP.NET Core gheppio
author: rick-anderson
description: Informazioni sulla configurazione degli endpoint con gheppio, il server Web multipiattaforma per ASP.NET Core.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel/endpoints
ms.openlocfilehash: f9d82409f4b31a5564c7cdfa48beb303d784e213
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057148"
---
# <a name="configure-endpoints-for-the-aspnet-core-kestrel-web-server"></a>Configurare gli endpoint per il server Web ASP.NET Core gheppio

Per impostazione predefinita, ASP.NET Core è associato a:

* `http://localhost:5000`
* `https://localhost:5001` (quando è presente un certificato di sviluppo locale)

Specificare gli URL usando gli elementi seguenti:

* La variabile di ambiente `ASPNETCORE_URLS`.
* L'argomento della riga di comando `--urls`.
* La chiave di configurazione dell'host `urls`.
* Il metodo di estensione `UseUrls`.

Il valore specificato usando i metodi seguenti può essere uno o più endpoint HTTP e HTTPS (HTTPS se è disponibile un certificato predefinito). Configurare il valore come un elenco delimitato da punto e virgola (ad esempio, `"Urls": "http://localhost:8000;http://localhost:8001"`).

Per altre informazioni su questi approcci, vedere [URL del server](xref:fundamentals/host/web-host#server-urls) e [Override della configurazione](xref:fundamentals/host/web-host#override-configuration).

Viene creato un certificato di sviluppo nei casi seguenti:

* Quando [.NET SDK](/dotnet/core/sdk) è installato.
* Quando per creare un certificato viene usato [lo strumento dev-certs](xref:aspnetcore-2.1#https).

Alcuni browser richiedono la concessione di autorizzazioni esplicite per considerare attendibile il certificato di sviluppo locale.

I modelli di progetto consentono di configurare le app per l'esecuzione su HTTPS per impostazione predefinita e includono il [reindirizzamento HTTPS e il supporto HSTS](xref:security/enforcing-ssl).

Chiamare i metodi <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> oppure <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket%2A> su <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> per configurare le porte e i prefissi URL per Kestrel.

Anche `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` e la variabile di ambiente `ASPNETCORE_URLS` funzionano, ma con le limitazioni indicate più avanti nella sezione (deve essere disponibile un certificato predefinito per la configurazione dell'endopoint HTTPS).

`KestrelServerOptions` configurazione

## <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults (azione \<ListenOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint specificato. Se si chiama `ConfigureEndpointDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });
});
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults%2A> non verranno applicati i valori predefiniti.

## <a name="configureiconfiguration"></a>Configure(IConfiguration)

Crea un loader di configurazione per la configurazione di Kestrel che accetta un elemento <xref:Microsoft.Extensions.Configuration.IConfiguration> come input. L'ambito della configurazione deve essere la sezione di configurazione per Kestrel.

`CreateDefaultBuilder` chiama `Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "Https": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    }
  }
}
```

## <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults (azione \<HttpsConnectionAdapterOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint HTTPS. Se si chiama `ConfigureHttpsDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        // certificate is an X509Certificate2
        listenOptions.ServerCertificate = certificate;
    });
});
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults%2A> non verranno applicati i valori predefiniti.

## <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

Configura Kestrel per l'uso di HTTPS.

Estensioni `ListenOptions.UseHttps`:

* `UseHttps`: Configurare gheppio per l'uso di HTTPS con il certificato predefinito. Genera un'eccezione se non è stato configurato alcun certificato predefinito.
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

Parametri `ListenOptions.UseHttps`:

* `filename` è il percorso e il nome file di un file di certificato, relativo alla directory che contiene i file di contenuto dell'app.
* `password` è la password richiesta per accedere ai dati del certificato X.509.
* `configureOptions` è un elemento `Action` per configurare `HttpsConnectionAdapterOptions`. Restituisce `ListenOptions`.
* `storeName` è l'archivio certificati da cui caricare il certificato.
* `subject` è il nome dell'oggetto del certificato.
* `allowInvalid` indica se devono essere considerati i certificati non validi, come ad esempio i certificati autofirmati.
* `location` è il percorso dell'archivio da cui caricare il certificato.
* `serverCertificate` è il certificato X.509.

Nell'ambiente di produzione, HTTPS deve essere configurato in modo esplicito. Come minimo, è necessario specificare un certificato predefinito.

Configurazioni supportate descritte in seguito:

* Nessuna configurazione
* Sostituire il certificato predefinito della configurazione
* Modificare le impostazioni predefinite nel codice

### <a name="no-configuration"></a>Nessuna configurazione

Kestrel è in ascolto su `http://localhost:5000` e `https://localhost:5001` (se è disponibile un certificato predefinito).

<a name="configuration"></a>

### <a name="replace-the-default-certificate-from-configuration"></a>Sostituire il certificato predefinito della configurazione

È disponibile per Kestrel uno schema di configurazione delle impostazioni delle app HTTPS predefinito. Configurare più endpoint, inclusi gli URL e i certificati da usare, da un file su disco o da un archivio certificati.

Nell'esempio seguente *appsettings.json* :

* Impostare `AllowInvalid` su `true` per consentire l'utilizzo di certificati non validi, ad esempio certificati autofirmati.
* Qualsiasi endpoint HTTPS che non specifica un certificato ( `HttpsDefaultCert` nell'esempio seguente) esegue il fallback al certificato definito in `Certificates:Default` o al certificato di sviluppo.

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },
      "HttpsInlineCertAndKeyFile": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Path": "<path to .pem/.crt file>",
          "KeyPath": "<path to .key file>",
          "Password": "<certificate password>"
        }
      },
      "HttpsInlineCertStore": {
        "Url": "https://localhost:5003",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },
      "HttpsDefaultCert": {
        "Url": "https://localhost:5004"
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

Note di schema:

* I nomi degli endpoint non fanno [distinzione tra maiuscole e](xref:fundamentals/configuration/index#configuration-keys-and-values)minuscole. Ad esempio, la sintassi di `HTTPS` and `Https` sono equivalenti.
* Il parametro `Url` è obbligatorio per ogni endpoint. Il formato per questo parametro è uguale a quello del parametro di configurazione di primo livello `Urls`, con la differenza che si limita a un singolo valore.
* Questi endpoint sostituiscono quelli definiti nella configurazione di primo livello `Urls`, non vi si aggiungono. Gli endpoint definiti nel codice tramite `Listen` si aggiungono agli endpoint definiti nella sezione di configurazione.
* La sezione `Certificate` è facoltativa. Se la `Certificate` sezione non è specificata, vengono usati i valori predefiniti definiti in `Certificates:Default` . Se non sono disponibili impostazioni predefinite, viene utilizzato il certificato di sviluppo. Se non sono presenti valori predefiniti e il certificato di sviluppo non è presente, il server genera un'eccezione e non viene avviato.
* La `Certificate` sezione supporta più [origini di certificati](#certificate-sources).
* È possibile definire un numero qualsiasi di endpoint nella [configurazione](xref:fundamentals/configuration/index) purché non causino conflitti di porta.

#### <a name="certificate-sources"></a>Origini certificati

È possibile configurare i nodi certificato per caricare i certificati da diverse origini:

* `Path` e `Password` per caricare i file con *estensione pfx* .
* `Path`, `KeyPath` e `Password` per caricare i file con estensione *PEM*. / *CRT* e *. Key* .
* `Subject` e da `Store` caricare dall'archivio certificati.

Ad esempio, il `Certificates:Default` certificato può essere specificato come:

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

#### <a name="configurationloader"></a>ConfigurationLoader

`options.Configure(context.Configuration.GetSection("{SECTION}"))` restituisce un oggetto <xref:Microsoft.AspNetCore.Server.Kestrel.KestrelConfigurationLoader> con un metodo `.Endpoint(string name, listenOptions => { })` che può essere usato per integrare le impostazioni dell'endpoint configurato:

```csharp
webBuilder.UseKestrel((context, serverOptions) =>
{
    serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
        .Endpoint("HTTPS", listenOptions =>
        {
            listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
        });
});
```

`KestrelServerOptions.ConfigurationLoader` è possibile accedere direttamente a per continuare a scorrere sul caricatore esistente, ad esempio quello fornito da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> .

* La sezione di configurazione per ogni endpoint è disponibile nelle opzioni del `Endpoint` metodo in modo che sia possibile leggere le impostazioni personalizzate.
* È possibile caricare più configurazioni chiamando ancora `options.Configure(context.Configuration.GetSection("{SECTION}"))` con un'altra sezione. Viene usata solo l'ultima configurazione, a meno che `Load` sia stato chiamato esplicitamente in istanze precedenti. Il metapacchetto non chiama `Load` in modo che la sezione di configurazione predefinita venga sostituita.
* `KestrelConfigurationLoader` riflette la famiglia di API `Listen` da `KestrelServerOptions` all'overload di `Endpoint`, in modo che gli endpoint di codice e di configurazione possano essere configurati nella stessa posizione. Questi overload non usano nomi e usano solo le impostazioni predefinite della configurazione.

### <a name="change-the-defaults-in-code"></a>Modificare le impostazioni predefinite nel codice

`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` possono essere usati per modificare le impostazioni predefinite per `ListenOptions` e `HttpsConnectionAdapterOptions`, inclusa l'esecuzione dell'override del certificato predefinito specificato nello scenario precedente. `ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` devono essere chiamati prima che venga configurato qualsiasi endpoint.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });

    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.SslProtocols = SslProtocols.Tls12;
    });
});
```

## <a name="configure-endpoints-using-server-name-indication"></a>Configurare gli endpoint con Indicazione nome server

[L'Indicazione nome server (SNI)](https://tools.ietf.org/html/rfc6066#section-3) può essere usata per ospitare più domini sulla stessa porta e indirizzo IP. Perché SNI funzioni è necessario che il client invii al server il nome host per la sessione protetta durante l'handshake TLS in modo che il server possa specificare il certificato corretto. Il client usa il certificato specificato per la comunicazione crittografata con il server durante la sessione protetta che segue l'handshake TLS.

Kestrel supporta SNI tramite il callback `ServerCertificateSelector`. Il callback viene richiamato una volta per ogni connessione per consentire all'app di controllare il nome host e selezionare il certificato appropriato. Il codice di callback seguente può essere usato nella `ConfigureWebHostDefaults` chiamata al metodo del file *Program.cs* di un progetto:

```csharp
//using System.Security.Cryptography.X509Certificates;
//using Microsoft.AspNetCore.Server.Kestrel.Https;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ListenAnyIP(5005, listenOptions =>
    {
        listenOptions.UseHttps(httpsOptions =>
        {
            var localhostCert = CertificateLoader.LoadFromStoreCert(
                "localhost", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var exampleCert = CertificateLoader.LoadFromStoreCert(
                "example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var subExampleCert = CertificateLoader.LoadFromStoreCert(
                "sub.example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var certs = new Dictionary<string, X509Certificate2>(StringComparer.OrdinalIgnoreCase)
            {
                { "localhost", localhostCert },
                { "example.com", exampleCert },
                { "sub.example.com", subExampleCert },
            };            

            httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
            {
                if (name != null && certs.TryGetValue(name, out var cert))
                {
                    return cert;
                }

                return exampleCert;
            };
        });
    });
});
```

Il supporto SNI richiede:

* In esecuzione su Framework di destinazione `netcoreapp2.1` o versione successiva. In `net461` o versioni successive, il callback viene richiamato, ma `name` è sempre `null` . L'elemento `name` è `null` anche se il client non specifica il parametro del nome host nell'handshake TLS.
* Esecuzione di tutti i siti Web nella stessa istanza di Kestrel. Kestrel supporta la condivisione di un indirizzo IP e di una porta tra più istanze solo con un proxy inverso.

## <a name="ssltls-protocols"></a>Protocolli SSL/TLS

I protocolli SSL sono protocolli usati per crittografare e decrittografare il traffico tra due peer, tradizionalmente un client e un server.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.SslProtocols = SslProtocols.Tls13;
    });
});
```

Il valore predefinito, `SslProtocols.None` , fa sì che gheppio usi le impostazioni predefinite del sistema operativo per scegliere il protocollo migliore. A meno che non esista un motivo specifico per selezionare un protocollo, usare il valore predefinito.
## <a name="connection-logging"></a>Registrazione connessione

Chiamare <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging%2A> per creare log a livello di debug per la comunicazione a livello di byte in una connessione. La registrazione delle connessioni è utile per la risoluzione dei problemi nella comunicazione di basso livello, ad esempio durante la crittografia TLS e dietro i proxy. Se `UseConnectionLogging` viene inserito prima `UseHttps` , viene registrato il traffico crittografato. Se `UseConnectionLogging` viene inserito dopo `UseHttps` , viene registrato il traffico decrittografato. Si tratta del middleware di [connessione](#connection-middleware)incorporato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

## <a name="bind-to-a-tcp-socket"></a>Associazione a un socket TCP

Il metodo <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen%2A> esegue l'associazione a un socket TCP e un'espressione lambda per le opzioni consente di configurare un certificato X.509:

[!code-csharp[](samples/5.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=12-18)]

L'esempio configura HTTPS per un endpoint con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>. Usare la stessa API per configurare altre impostazioni di Kestrel per endpoint specifici.

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

## <a name="bind-to-a-unix-socket"></a>Associazione a un socket Unix

Impostare l'ascolto su un socket Unix con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket%2A> per migliorare le prestazioni con Nginx, come visualizzato nell'esempio seguente:

[!code-csharp[](samples/5.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* Nel file di configurazione nginx impostare la `server`  >  `location`  >  `proxy_pass` voce su `http://unix:/tmp/{KESTREL SOCKET}:/;` . `{KESTREL SOCKET}` nome del socket fornito a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket%2A> (ad esempio, `kestrel-test.sock` nell'esempio precedente).
* Verificare che il socket sia scrivibile da nginx (ad esempio, `chmod go+w /tmp/kestrel-test.sock` ).

## <a name="port-0"></a>Porta 0

Quando si specifica il numero di porta `0`, Kestrel esegue l'associazione dinamica a una porta disponibile. Nell'esempio seguente viene illustrato come determinare il gheppio della porta associato in fase di esecuzione:

[!code-csharp[](samples/5.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

Quando si esegue l'app, l'output della finestra della console indica la porta dinamica in cui l'app può essere raggiunta:

```console
Listening on the following addresses: http://127.0.0.1:48508
```

## <a name="limitations"></a>Limitazioni

Configurare gli endpoint con i metodi seguenti:

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls%2A>
* L'argomento della riga di comando `--urls`
* La chiave di configurazione dell'host `urls`
* La variabile di ambiente `ASPNETCORE_URLS`

Questi metodi sono utili se si vuole che il codice funzioni con server diversi da Kestrel. Tenere presenti, tuttavia, le limitazioni seguenti:

* Non è possibile usare HTTPS con questi approcci a meno che non venga fornito un certificato predefinito nella configurazione dell'endpoint HTTPS, ad esempio usando la `KestrelServerOptions` configurazione o un file di configurazione, come illustrato in precedenza in questo articolo.
* Quando sia l'approccio `Listen` che l'approccio `UseUrls` vengono usati contemporaneamente, gli endpoint `Listen` eseguono l'override degli endpoint `UseUrls`.

## <a name="iis-endpoint-configuration"></a>Configurazione dell'endpoint IIS

Quando si usa IIS, le associazioni di URL per le associazioni di override di IIS vengono impostate mediante `Listen` o `UseUrls`. Per ulteriori informazioni, vedere [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module).

## <a name="listenoptionsprotocols"></a>ListenOptions.Protocols

La proprietà `Protocols` stabilisce i protocolli HTTP (`HttpProtocols`) abilitati in un endpoint di connessione o per il server. Assegnare un valore alla proprietà `Protocols` dall'enumerazione `HttpProtocols`.

| Valore di enumerazione `HttpProtocols` | Protocollo di connessione consentito |
| -------------------------- | ----------------------------- |
| `Http1`                    | Solo HTTP/1.1. Può essere usato con o senza TLS. |
| `Http2`                    | Solo HTTP/2. Può essere usato senza TLS solo se il client supporta una [modalità di conoscenza pregressa](https://tools.ietf.org/html/rfc7540#section-3.4). |
| `Http1AndHttp2`            | HTTP/1.1 e HTTP/2. HTTP/2 richiede che il client selezioni HTTP/2 nell'handshake TLS [(Application-Layer Protocol negotiation) ALPN](https://tools.ietf.org/html/rfc7301#section-3) in caso contrario, il valore predefinito per la connessione è HTTP/1.1. |

Il `ListenOptions.Protocols` valore predefinito per qualsiasi endpoint è `HttpProtocols.Http1AndHttp2` .

Restrizioni relative a TLS per HTTP/2:

* TLS versione 1.2 o successiva
* Rinegoziazione disabilitata
* Compressione disabilitata
* Dimensioni minime per lo scambio di chiavi temporanee:
  * Curva ellittica Diffie-Hellman (ECDHE) &lbrack; [RFC4492](https://www.ietf.org/rfc/rfc4492.txt) &rbrack; : minimo 224 bit
  * Diffie-Hellman campo finito (DHE) &lbrack; `TLS12` &rbrack; : minimo 2048 bit
* Il pacchetto di crittografia non è consentito. 

`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`&lbrack;`TLS-ECDHE`&rbrack; con la curva ellittica P-256 &lbrack; `FIPS186` &rbrack; è supportata per impostazione predefinita.

Nell'esempio seguente sono consentite connessioni HTTP/1.1 e HTTP/2 sulla porta 8000. Le connessioni sono protette da TLS con un certificato incluso:

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

In Linux <xref:System.Net.Security.CipherSuitesPolicy> è possibile usare per filtrare gli handshake TLS in base alla connessione:

```csharp
// using System.Net.Security;
// using Microsoft.AspNetCore.Hosting;
// using Microsoft.AspNetCore.Server.Kestrel.Core;
// using Microsoft.Extensions.DependencyInjection;
// using Microsoft.Extensions.Hosting;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.OnAuthenticate = (context, sslOptions) =>
        {
            sslOptions.CipherSuitesPolicy = new CipherSuitesPolicy(
                new[]
                {
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
                    // ...
                });
        };
    });
});
```

## <a name="connection-middleware"></a>Middleware di connessione

Il middleware di connessione personalizzato può filtrare gli handshake TLS in base alla connessione per specifiche crittografie, se necessario.

L'esempio seguente genera un'eccezione <xref:System.NotSupportedException> per qualsiasi algoritmo crittografico non supportato dall'app. In alternativa, definire e confrontare [ITlsHandshakeFeature. CipherAlgorithm](xref:Microsoft.AspNetCore.Connections.Features.ITlsHandshakeFeature.CipherAlgorithm) con un elenco di pacchetti di crittografia accettabili.

Non viene utilizzata alcuna crittografia con un algoritmo di crittografia [CipherAlgorithmType. null](xref:System.Security.Authentication.CipherAlgorithmType) .

```csharp
// using System.Net;
// using Microsoft.AspNetCore.Connections;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.UseTlsFilter();
    });
});
```

```csharp
using System;
using System.Security.Authentication;
using Microsoft.AspNetCore.Connections.Features;

namespace Microsoft.AspNetCore.Connections
{
    public static class TlsFilterConnectionMiddlewareExtensions
    {
        public static IConnectionBuilder UseTlsFilter(
            this IConnectionBuilder builder)
        {
            return builder.Use((connection, next) =>
            {
                var tlsFeature = connection.Features.Get<ITlsHandshakeFeature>();

                if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
                {
                    throw new NotSupportedException("Prohibited cipher: " +
                        tlsFeature.CipherAlgorithm);
                }

                return next();
            });
        }
    }
}
```

Il filtro della connessione può essere configurato anche tramite un' <xref:Microsoft.AspNetCore.Connections.IConnectionBuilder> espressione lambda:

```csharp
// using System;
// using System.Net;
// using System.Security.Authentication;
// using Microsoft.AspNetCore.Connections;
// using Microsoft.AspNetCore.Connections.Features;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.Use((context, next) =>
        {
            var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

            if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
            {
                throw new NotSupportedException(
                    $"Prohibited cipher: {tlsFeature.CipherAlgorithm}");
            }

            return next();
        });
    });
});
```

## <a name="set-the-http-protocol-from-configuration"></a>Imposta il protocollo HTTP dalla configurazione

`CreateDefaultBuilder` chiama `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.

Nell' *appsettings.json* esempio seguente viene stabilito http/1.1 come protocollo di connessione predefinito per tutti gli endpoint:

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1"
    }
  }
}
```

Nell' *appsettings.json* esempio seguente viene stabilito il protocollo di connessione HTTP/1.1 per un endpoint specifico:

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1"
      }
    }
  }
}
```

I protocolli specificati nei valori di override del codice sono impostati dalla configurazione.

## <a name="url-prefixes"></a>Prefissi URL

Se si usa `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` o la variabile di ambiente `ASPNETCORE_URLS`, i prefissi URL possono avere uno dei formati seguenti.

Sono validi solo i prefissi URL HTTP. Kestrel non supporta HTTPS quando la configurazione delle associazioni URL viene eseguita con `UseUrls`.

* Indirizzo IPv4 con numero di porta

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` è un caso speciale che esegue l'associazione a tutti gli indirizzi IPv4.

* Indirizzo IPv6 con numero di porta

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  `[::]` è l'equivalente IPv6 di `0.0.0.0` per IPv4.

* Nome host con numero di porta

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  I nomi host, `*` e `+` non sono casi particolari. Tutto ciò che non è riconosciuto come un indirizzo IP o un elemento `localhost` valido esegue l'associazione a tutti gli IP IPv4 e IPv6. Per associare nomi host diversi a app ASP.NET Core diverse sulla stessa porta, usare [HTTP.sys](xref:fundamentals/servers/httpsys) o un server proxy inverso. Gli esempi di server proxy inversi includono IIS, nginx o Apache.

  > [!WARNING]
  > Una configurazione che prevede un proxy inverso richiede il [filtro host](xref:fundamentals/servers/kestrel/host-filtering).

* Nome host `localhost` con numero di porta o IP di loopback con numero di porta

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  Se è specificato `localhost`, Kestrel tenta l'associazione alle interfacce di loopback sia IPv4 che IPv6. Se la porta richiesta è in uso da parte di un altro servizio in una delle due interfacce di loopback, Kestrel non viene avviato. Se una delle due interfacce di loopback non è disponibile per qualsiasi altro motivo (in genere perché IPv6 non è supportato), Kestrel registra un avviso.
