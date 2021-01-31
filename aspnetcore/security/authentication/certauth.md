---
title: Configurare l'autenticazione del certificato in ASP.NET Core
author: blowdart
description: Informazioni su come configurare l'autenticazione del certificato in ASP.NET Core per IIS e HTTP.sys.
monikerRange: '>= aspnetcore-3.0'
ms.author: bdorrans
ms.date: 07/16/2020
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
uid: security/authentication/certauth
ms.openlocfilehash: c862bc8bff6c4cc80696d92067e814889d6e7782
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217531"
---
# <a name="configure-certificate-authentication-in-aspnet-core"></a><span data-ttu-id="fbdac-103">Configurare l'autenticazione del certificato in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="fbdac-103">Configure certificate authentication in ASP.NET Core</span></span>

<span data-ttu-id="fbdac-104">`Microsoft.AspNetCore.Authentication.Certificate` contiene un'implementazione simile all' [autenticazione del certificato](https://tools.ietf.org/html/rfc5246#section-7.4.4) per ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="fbdac-104">`Microsoft.AspNetCore.Authentication.Certificate` contains an implementation similar to [Certificate Authentication](https://tools.ietf.org/html/rfc5246#section-7.4.4) for ASP.NET Core.</span></span> <span data-ttu-id="fbdac-105">L'autenticazione del certificato viene eseguita a livello di TLS, molto prima che venga mai ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="fbdac-105">Certificate authentication happens at the TLS level, long before it ever gets to ASP.NET Core.</span></span> <span data-ttu-id="fbdac-106">Più precisamente, si tratta di un gestore di autenticazione che convalida il certificato e quindi fornisce un evento in cui è possibile risolvere il certificato in un `ClaimsPrincipal` .</span><span class="sxs-lookup"><span data-stu-id="fbdac-106">More accurately, this is an authentication handler that validates the certificate and then gives you an event where you can resolve that certificate to a `ClaimsPrincipal`.</span></span> 

<span data-ttu-id="fbdac-107">[Configurare il server](#configure-your-server-to-require-certificates) per l'autenticazione del certificato, essere IIS, gheppio, app Web di Azure o qualsiasi altro elemento in uso.</span><span class="sxs-lookup"><span data-stu-id="fbdac-107">[Configure your server](#configure-your-server-to-require-certificates) for certificate authentication, be it IIS, Kestrel, Azure Web Apps, or whatever else you're using.</span></span>

## <a name="proxy-and-load-balancer-scenarios"></a><span data-ttu-id="fbdac-108">Scenari di proxy e bilanciamento del carico</span><span class="sxs-lookup"><span data-stu-id="fbdac-108">Proxy and load balancer scenarios</span></span>

<span data-ttu-id="fbdac-109">L'autenticazione del certificato è uno scenario con stato usato principalmente quando un proxy o un servizio di bilanciamento del carico non gestisce il traffico tra client e server.</span><span class="sxs-lookup"><span data-stu-id="fbdac-109">Certificate authentication is a stateful scenario primarily used where a proxy or load balancer doesn't handle traffic between clients and servers.</span></span> <span data-ttu-id="fbdac-110">Se si usa un proxy o un servizio di bilanciamento del carico, l'autenticazione del certificato funziona solo se il proxy o il servizio di bilanciamento del carico:</span><span class="sxs-lookup"><span data-stu-id="fbdac-110">If a proxy or load balancer is used, certificate authentication only works if the proxy or load balancer:</span></span>

* <span data-ttu-id="fbdac-111">Gestisce l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-111">Handles the authentication.</span></span>
* <span data-ttu-id="fbdac-112">Passa le informazioni di autenticazione utente all'app, ad esempio in un'intestazione di richiesta, che agisce sulle informazioni di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-112">Passes the user authentication information to the app (for example, in a request header), which acts on the authentication information.</span></span>

<span data-ttu-id="fbdac-113">Un'alternativa all'autenticazione del certificato negli ambienti in cui vengono usati proxy e servizi di bilanciamento del carico è Active Directory servizi federati (ADFS) con OpenID Connect (OIDC).</span><span class="sxs-lookup"><span data-stu-id="fbdac-113">An alternative to certificate authentication in environments where proxies and load balancers are used is Active Directory Federated Services (ADFS) with OpenID Connect (OIDC).</span></span>

## <a name="get-started"></a><span data-ttu-id="fbdac-114">Introduzione</span><span class="sxs-lookup"><span data-stu-id="fbdac-114">Get started</span></span>

<span data-ttu-id="fbdac-115">Acquisire un certificato HTTPS, applicarlo e [configurare il server](#configure-your-server-to-require-certificates) per richiedere i certificati.</span><span class="sxs-lookup"><span data-stu-id="fbdac-115">Acquire an HTTPS certificate, apply it, and [configure your server](#configure-your-server-to-require-certificates) to require certificates.</span></span>

<span data-ttu-id="fbdac-116">Nell'app Web aggiungere un riferimento al pacchetto [Microsoft. AspNetCore. Authentication. Certificate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Certificate) .</span><span class="sxs-lookup"><span data-stu-id="fbdac-116">In your web app, add a reference to the [Microsoft.AspNetCore.Authentication.Certificate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Certificate) package.</span></span> <span data-ttu-id="fbdac-117">Quindi, nel `Startup.ConfigureServices` metodo chiamare `services.AddAuthentication(CertificateAuthenticationDefaults.AuthenticationScheme).AddCertificate(...);` con le opzioni, fornendo un delegato per `OnCertificateValidated` per eseguire qualsiasi convalida supplementare sul certificato client inviato con le richieste.</span><span class="sxs-lookup"><span data-stu-id="fbdac-117">Then in the `Startup.ConfigureServices` method, call `services.AddAuthentication(CertificateAuthenticationDefaults.AuthenticationScheme).AddCertificate(...);` with your options, providing a delegate for `OnCertificateValidated` to do any supplementary validation on the client certificate sent with requests.</span></span> <span data-ttu-id="fbdac-118">Trasformare le informazioni in un oggetto `ClaimsPrincipal` e impostarle sulla `context.Principal` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="fbdac-118">Turn that information into a `ClaimsPrincipal` and set it on the `context.Principal` property.</span></span>

<span data-ttu-id="fbdac-119">Se l'autenticazione ha esito negativo, questo gestore restituisce una `403 (Forbidden)` risposta anziché un `401 (Unauthorized)` , come si può immaginare.</span><span class="sxs-lookup"><span data-stu-id="fbdac-119">If authentication fails, this handler returns a `403 (Forbidden)` response rather a `401 (Unauthorized)`, as you might expect.</span></span> <span data-ttu-id="fbdac-120">Il motivo è che l'autenticazione deve verificarsi durante la connessione TLS iniziale.</span><span class="sxs-lookup"><span data-stu-id="fbdac-120">The reasoning is that the authentication should happen during the initial TLS connection.</span></span> <span data-ttu-id="fbdac-121">Quando raggiunge il gestore, è troppo tardi.</span><span class="sxs-lookup"><span data-stu-id="fbdac-121">By the time it reaches the handler, it's too late.</span></span> <span data-ttu-id="fbdac-122">Non è possibile aggiornare la connessione da una connessione anonima a un'altra con un certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-122">There's no way to upgrade the connection from an anonymous connection to one with a certificate.</span></span>

<span data-ttu-id="fbdac-123">Aggiungere anche `app.UseAuthentication();` il `Startup.Configure` metodo.</span><span class="sxs-lookup"><span data-stu-id="fbdac-123">Also add `app.UseAuthentication();` in the `Startup.Configure` method.</span></span> <span data-ttu-id="fbdac-124">In caso contrario, `HttpContext.User` non verrà impostato su `ClaimsPrincipal` creato dal certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-124">Otherwise, the `HttpContext.User` will not be set to `ClaimsPrincipal` created from the certificate.</span></span> <span data-ttu-id="fbdac-125">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="fbdac-125">For example:</span></span>

::: moniker range=">= aspnetcore-5.0"

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate()
        // Adding an ICertificateValidationCache results in certificate auth caching the results.
        // The default implementation uses a memory cache.
        .AddCertificateCache();

    // All other service configuration
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();

    // All other app configuration
}
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate();

    // All other service configuration
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();

    // All other app configuration
}
```

::: moniker-end

<span data-ttu-id="fbdac-126">Nell'esempio precedente viene illustrata la modalità predefinita per aggiungere l'autenticazione del certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-126">The preceding example demonstrates the default way to add certificate authentication.</span></span> <span data-ttu-id="fbdac-127">Il gestore crea un'entità utente usando le proprietà comuni del certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-127">The handler constructs a user principal using the common certificate properties.</span></span>

## <a name="configure-certificate-validation"></a><span data-ttu-id="fbdac-128">Configurare la convalida del certificato</span><span class="sxs-lookup"><span data-stu-id="fbdac-128">Configure certificate validation</span></span>

<span data-ttu-id="fbdac-129">Il `CertificateAuthenticationOptions` gestore dispone di alcune convalide predefinite che rappresentano le convalide minime che è necessario eseguire su un certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-129">The `CertificateAuthenticationOptions` handler has some built-in validations that are the minimum validations you should perform on a certificate.</span></span> <span data-ttu-id="fbdac-130">Ognuna di queste impostazioni è abilitata per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="fbdac-130">Each of these settings is enabled by default.</span></span>

### <a name="allowedcertificatetypes--chained-selfsigned-or-all-chained--selfsigned"></a><span data-ttu-id="fbdac-131">AllowedCertificateTypes = concatenato, SelfSigned o tutti (concatenato | SelfSigned)</span><span class="sxs-lookup"><span data-stu-id="fbdac-131">AllowedCertificateTypes = Chained, SelfSigned, or All (Chained | SelfSigned)</span></span>

<span data-ttu-id="fbdac-132">Valore predefinito: `CertificateTypes.Chained`</span><span class="sxs-lookup"><span data-stu-id="fbdac-132">Default value: `CertificateTypes.Chained`</span></span>

<span data-ttu-id="fbdac-133">Questo controllo consente di verificare che sia consentito solo il tipo di certificato appropriato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-133">This check validates that only the appropriate certificate type is allowed.</span></span> <span data-ttu-id="fbdac-134">Se l'app usa certificati autofirmati, questa opzione deve essere impostata su `CertificateTypes.All` o `CertificateTypes.SelfSigned` .</span><span class="sxs-lookup"><span data-stu-id="fbdac-134">If the app is using self-signed certificates, this option needs to be set to `CertificateTypes.All` or `CertificateTypes.SelfSigned`.</span></span>

### <a name="validatecertificateuse"></a><span data-ttu-id="fbdac-135">ValidateCertificateUse</span><span class="sxs-lookup"><span data-stu-id="fbdac-135">ValidateCertificateUse</span></span>

<span data-ttu-id="fbdac-136">Valore predefinito: `true`</span><span class="sxs-lookup"><span data-stu-id="fbdac-136">Default value: `true`</span></span>

<span data-ttu-id="fbdac-137">Questo controllo verifica che il certificato presentato dal client disponga dell'utilizzo chiavi avanzato (EKU) di autenticazione client o non EKU affatto.</span><span class="sxs-lookup"><span data-stu-id="fbdac-137">This check validates that the certificate presented by the client has the Client Authentication extended key use (EKU), or no EKUs at all.</span></span> <span data-ttu-id="fbdac-138">Come le specifiche dicono, se non è specificato alcun EKU, tutti i EKU vengono considerati validi.</span><span class="sxs-lookup"><span data-stu-id="fbdac-138">As the specifications say, if no EKU is specified, then all EKUs are deemed valid.</span></span>

### <a name="validatevalidityperiod"></a><span data-ttu-id="fbdac-139">ValidateValidityPeriod</span><span class="sxs-lookup"><span data-stu-id="fbdac-139">ValidateValidityPeriod</span></span>

<span data-ttu-id="fbdac-140">Valore predefinito: `true`</span><span class="sxs-lookup"><span data-stu-id="fbdac-140">Default value: `true`</span></span>

<span data-ttu-id="fbdac-141">Questo controllo verifica che il certificato sia entro il periodo di validità.</span><span class="sxs-lookup"><span data-stu-id="fbdac-141">This check validates that the certificate is within its validity period.</span></span> <span data-ttu-id="fbdac-142">Per ogni richiesta, il gestore garantisce che un certificato valido al momento della presentazione non sia scaduto durante la sessione corrente.</span><span class="sxs-lookup"><span data-stu-id="fbdac-142">On each request, the handler ensures that a certificate that was valid when it was presented hasn't expired during its current session.</span></span>

### <a name="revocationflag"></a><span data-ttu-id="fbdac-143">RevocationFlag</span><span class="sxs-lookup"><span data-stu-id="fbdac-143">RevocationFlag</span></span>

<span data-ttu-id="fbdac-144">Valore predefinito: `X509RevocationFlag.ExcludeRoot`</span><span class="sxs-lookup"><span data-stu-id="fbdac-144">Default value: `X509RevocationFlag.ExcludeRoot`</span></span>

<span data-ttu-id="fbdac-145">Flag che specifica i certificati nella catena controllati per la revoca.</span><span class="sxs-lookup"><span data-stu-id="fbdac-145">A flag that specifies which certificates in the chain are checked for revocation.</span></span>

<span data-ttu-id="fbdac-146">I controlli di revoca vengono eseguiti solo quando il certificato è concatenato a un certificato radice.</span><span class="sxs-lookup"><span data-stu-id="fbdac-146">Revocation checks are only performed when the certificate is chained to a root certificate.</span></span>

### <a name="revocationmode"></a><span data-ttu-id="fbdac-147">RevocationMode</span><span class="sxs-lookup"><span data-stu-id="fbdac-147">RevocationMode</span></span>

<span data-ttu-id="fbdac-148">Valore predefinito: `X509RevocationMode.Online`</span><span class="sxs-lookup"><span data-stu-id="fbdac-148">Default value: `X509RevocationMode.Online`</span></span>

<span data-ttu-id="fbdac-149">Flag che specifica la modalità di esecuzione dei controlli di revoca.</span><span class="sxs-lookup"><span data-stu-id="fbdac-149">A flag that specifies how revocation checks are performed.</span></span>

<span data-ttu-id="fbdac-150">La specifica di un controllo online può comportare un ritardo prolungato mentre viene contattata l'autorità di certificazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-150">Specifying an online check can result in a long delay while the certificate authority is contacted.</span></span>

<span data-ttu-id="fbdac-151">I controlli di revoca vengono eseguiti solo quando il certificato è concatenato a un certificato radice.</span><span class="sxs-lookup"><span data-stu-id="fbdac-151">Revocation checks are only performed when the certificate is chained to a root certificate.</span></span>

### <a name="can-i-configure-my-app-to-require-a-certificate-only-on-certain-paths"></a><span data-ttu-id="fbdac-152">È possibile configurare l'app in modo che richieda un certificato solo su determinati percorsi?</span><span class="sxs-lookup"><span data-stu-id="fbdac-152">Can I configure my app to require a certificate only on certain paths?</span></span>

<span data-ttu-id="fbdac-153">Questa operazione non è possibile.</span><span class="sxs-lookup"><span data-stu-id="fbdac-153">This isn't possible.</span></span> <span data-ttu-id="fbdac-154">Tenere presente che lo scambio di certificati è stato eseguito dall'avvio della conversazione HTTPS, che viene eseguito dal server prima che la prima richiesta venga ricevuta sulla connessione, quindi non è possibile definire l'ambito in base a qualsiasi campo di richiesta.</span><span class="sxs-lookup"><span data-stu-id="fbdac-154">Remember the certificate exchange is done that the start of the HTTPS conversation, it's done by the server before the first request is received on that connection so it's not possible to scope based on any request fields.</span></span>

## <a name="handler-events"></a><span data-ttu-id="fbdac-155">Eventi gestore</span><span class="sxs-lookup"><span data-stu-id="fbdac-155">Handler events</span></span>

<span data-ttu-id="fbdac-156">Il gestore dispone di due eventi:</span><span class="sxs-lookup"><span data-stu-id="fbdac-156">The handler has two events:</span></span>

* <span data-ttu-id="fbdac-157">`OnAuthenticationFailed`: Viene chiamato se si verifica un'eccezione durante l'autenticazione e consente di rispondere.</span><span class="sxs-lookup"><span data-stu-id="fbdac-157">`OnAuthenticationFailed`: Called if an exception happens during authentication and allows you to react.</span></span>
* <span data-ttu-id="fbdac-158">`OnCertificateValidated`: Viene chiamato dopo che il certificato è stato convalidato, la convalida è stata superata ed è stata creata un'entità predefinita.</span><span class="sxs-lookup"><span data-stu-id="fbdac-158">`OnCertificateValidated`: Called after the certificate has been validated, passed validation and a default principal has been created.</span></span> <span data-ttu-id="fbdac-159">Questo evento consente di eseguire una convalida personalizzata e di aumentare o sostituire l'entità di protezione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-159">This event allows you to perform your own validation and augment or replace the principal.</span></span> <span data-ttu-id="fbdac-160">Gli esempi includono:</span><span class="sxs-lookup"><span data-stu-id="fbdac-160">For examples include:</span></span>
  * <span data-ttu-id="fbdac-161">Determinare se il certificato è noto ai servizi.</span><span class="sxs-lookup"><span data-stu-id="fbdac-161">Determining if the certificate is known to your services.</span></span>
  * <span data-ttu-id="fbdac-162">Creazione di un'entità personalizzata.</span><span class="sxs-lookup"><span data-stu-id="fbdac-162">Constructing your own principal.</span></span> <span data-ttu-id="fbdac-163">Si consideri l'esempio seguente in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="fbdac-163">Consider the following example in `Startup.ConfigureServices`:</span></span>

    ```csharp
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate(options =>
        {
            options.Events = new CertificateAuthenticationEvents
            {
                OnCertificateValidated = context =>
                {
                    var claims = new[]
                    {
                        new Claim(
                            ClaimTypes.NameIdentifier, 
                            context.ClientCertificate.Subject,
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer),
                        new Claim(ClaimTypes.Name,
                            context.ClientCertificate.Subject,
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer)
                    };
    
                    context.Principal = new ClaimsPrincipal(
                        new ClaimsIdentity(claims, context.Scheme.Name));
                    context.Success();
    
                    return Task.CompletedTask;
                }
            };
        });
    ```

<span data-ttu-id="fbdac-164">Se il certificato in ingresso non soddisfa la convalida aggiuntiva, chiamare `context.Fail("failure reason")` con un motivo dell'errore.</span><span class="sxs-lookup"><span data-stu-id="fbdac-164">If you find the inbound certificate doesn't meet your extra validation, call `context.Fail("failure reason")` with a failure reason.</span></span>

<span data-ttu-id="fbdac-165">Per le funzionalità reali, è probabile che si voglia chiamare un servizio registrato nell'inserimento delle dipendenze che si connette a un database o a un altro tipo di archivio utente.</span><span class="sxs-lookup"><span data-stu-id="fbdac-165">For real functionality, you'll probably want to call a service registered in dependency injection that connects to a database or other type of user store.</span></span> <span data-ttu-id="fbdac-166">Accedere al servizio usando il contesto passato al delegato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-166">Access your service by using the context passed into your delegate.</span></span> <span data-ttu-id="fbdac-167">Si consideri l'esempio seguente in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="fbdac-167">Consider the following example in `Startup.ConfigureServices`:</span></span>

```csharp
services.AddAuthentication(
    CertificateAuthenticationDefaults.AuthenticationScheme)
    .AddCertificate(options =>
    {
        options.Events = new CertificateAuthenticationEvents
        {
            OnCertificateValidated = context =>
            {
                var validationService =
                    context.HttpContext.RequestServices
                        .GetRequiredService<ICertificateValidationService>();
                
                if (validationService.ValidateCertificate(
                    context.ClientCertificate))
                {
                    var claims = new[]
                    {
                        new Claim(
                            ClaimTypes.NameIdentifier, 
                            context.ClientCertificate.Subject, 
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer),
                        new Claim(
                            ClaimTypes.Name, 
                            context.ClientCertificate.Subject, 
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer)
                    };

                    context.Principal = new ClaimsPrincipal(
                        new ClaimsIdentity(claims, context.Scheme.Name));
                    context.Success();
                }                     

                return Task.CompletedTask;
            }
        };
    });
```

<span data-ttu-id="fbdac-168">Concettualmente, la convalida del certificato è un problema di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-168">Conceptually, the validation of the certificate is an authorization concern.</span></span> <span data-ttu-id="fbdac-169">L'aggiunta di un controllo, ad esempio, di un'autorità emittente o di un'identificazione personale in un criterio di autorizzazione, anziché all'interno di `OnCertificateValidated` , è perfettamente accettabile.</span><span class="sxs-lookup"><span data-stu-id="fbdac-169">Adding a check on, for example, an issuer or thumbprint in an authorization policy, rather than inside `OnCertificateValidated`, is perfectly acceptable.</span></span>

## <a name="configure-your-server-to-require-certificates"></a><span data-ttu-id="fbdac-170">Configurare il server per richiedere i certificati</span><span class="sxs-lookup"><span data-stu-id="fbdac-170">Configure your server to require certificates</span></span>

### <a name="kestrel"></a><span data-ttu-id="fbdac-171">Kestrel</span><span class="sxs-lookup"><span data-stu-id="fbdac-171">Kestrel</span></span>

<span data-ttu-id="fbdac-172">In *Program.cs* configurare gheppio come segue:</span><span class="sxs-lookup"><span data-stu-id="fbdac-172">In *Program.cs*, configure Kestrel as follows:</span></span>

```csharp
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args)
{
    return Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            webBuilder.ConfigureKestrel(o =>
            {
                o.ConfigureHttpsDefaults(o => 
            o.ClientCertificateMode = 
                ClientCertificateMode.RequireCertificate);
            });
        });
}
```

> [!NOTE]
> <span data-ttu-id="fbdac-173">Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> non verranno applicati i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="fbdac-173">Endpoints created by calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **before** calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> won't have the defaults applied.</span></span>

### <a name="iis"></a><span data-ttu-id="fbdac-174">IIS</span><span class="sxs-lookup"><span data-stu-id="fbdac-174">IIS</span></span>

<span data-ttu-id="fbdac-175">Completare i passaggi seguenti in Gestione IIS:</span><span class="sxs-lookup"><span data-stu-id="fbdac-175">Complete the following steps in IIS Manager:</span></span>

1. <span data-ttu-id="fbdac-176">Selezionare il sito dalla scheda **connessioni** .</span><span class="sxs-lookup"><span data-stu-id="fbdac-176">Select your site from the **Connections** tab.</span></span>
1. <span data-ttu-id="fbdac-177">Fare doppio clic sull'opzione **Impostazioni SSL** nella finestra **visualizzazione funzionalità** .</span><span class="sxs-lookup"><span data-stu-id="fbdac-177">Double-click the **SSL Settings** option in the **Features View** window.</span></span>
1. <span data-ttu-id="fbdac-178">Selezionare la casella di controllo **Richiedi SSL** e selezionare il pulsante di opzione **Richiedi** nella sezione **certificati client** .</span><span class="sxs-lookup"><span data-stu-id="fbdac-178">Check the **Require SSL** checkbox, and select the **Require** radio button in the **Client certificates** section.</span></span>

![Impostazioni del certificato client in IIS](README-IISConfig.png)

### <a name="azure-and-custom-web-proxies"></a><span data-ttu-id="fbdac-180">Azure e proxy Web personalizzati</span><span class="sxs-lookup"><span data-stu-id="fbdac-180">Azure and custom web proxies</span></span>

<span data-ttu-id="fbdac-181">Vedere l' [host e distribuire la documentazione](xref:host-and-deploy/proxy-load-balancer#certificate-forwarding) per informazioni su come configurare il middleware di invio del certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-181">See the [host and deploy documentation](xref:host-and-deploy/proxy-load-balancer#certificate-forwarding) for how to configure the certificate forwarding middleware.</span></span>

### <a name="use-certificate-authentication-in-azure-web-apps"></a><span data-ttu-id="fbdac-182">Usare l'autenticazione del certificato in app Web di Azure</span><span class="sxs-lookup"><span data-stu-id="fbdac-182">Use certificate authentication in Azure Web Apps</span></span>

<span data-ttu-id="fbdac-183">Per Azure non è necessaria alcuna configurazione di invio.</span><span class="sxs-lookup"><span data-stu-id="fbdac-183">No forwarding configuration is required for Azure.</span></span> <span data-ttu-id="fbdac-184">Questa configurazione è già presente nel middleware di invio del certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-184">This is already setup in the certificate forwarding middleware.</span></span>

> [!NOTE]
> <span data-ttu-id="fbdac-185">A tale scopo, è necessario che CertificateForwardingMiddleware sia presente.</span><span class="sxs-lookup"><span data-stu-id="fbdac-185">This requires that the CertificateForwardingMiddleware is present.</span></span>

### <a name="use-certificate-authentication-in-custom-web-proxies"></a><span data-ttu-id="fbdac-186">Usare l'autenticazione del certificato in proxy Web personalizzati</span><span class="sxs-lookup"><span data-stu-id="fbdac-186">Use certificate authentication in custom web proxies</span></span>

<span data-ttu-id="fbdac-187">Il `AddCertificateForwarding` metodo viene usato per specificare:</span><span class="sxs-lookup"><span data-stu-id="fbdac-187">The `AddCertificateForwarding` method is used to specify:</span></span>

* <span data-ttu-id="fbdac-188">Nome dell'intestazione del client.</span><span class="sxs-lookup"><span data-stu-id="fbdac-188">The client header name.</span></span>
* <span data-ttu-id="fbdac-189">Modalità di caricamento del certificato (utilizzando la `HeaderConverter` proprietà).</span><span class="sxs-lookup"><span data-stu-id="fbdac-189">How the certificate is to be loaded (using the `HeaderConverter` property).</span></span>

<span data-ttu-id="fbdac-190">Nei proxy Web personalizzati il certificato viene passato come intestazione di richiesta personalizzata, ad esempio `X-SSL-CERT` .</span><span class="sxs-lookup"><span data-stu-id="fbdac-190">In custom web proxies, the certificate is passed as a custom request header, for example `X-SSL-CERT`.</span></span> <span data-ttu-id="fbdac-191">Per usarlo, configurare l'invio di certificati in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="fbdac-191">To use it, configure certificate forwarding in `Startup.ConfigureServices`:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddCertificateForwarding(options =>
    {
        options.CertificateHeader = "X-SSL-CERT";
        options.HeaderConverter = (headerValue) =>
        {
            X509Certificate2 clientCertificate = null;

            if(!string.IsNullOrWhiteSpace(headerValue))
            {
                byte[] bytes = StringToByteArray(headerValue);
                clientCertificate = new X509Certificate2(bytes);
            }

            return clientCertificate;
        };
    });
}

private static byte[] StringToByteArray(string hex)
{
    int NumberChars = hex.Length;
    byte[] bytes = new byte[NumberChars / 2];

    for (int i = 0; i < NumberChars; i += 2)
    {
        bytes[i / 2] = Convert.ToByte(hex.Substring(i, 2), 16);
    }

    return bytes;
}
```

<span data-ttu-id="fbdac-192">Il `Startup.Configure` metodo aggiunge quindi il middleware.</span><span class="sxs-lookup"><span data-stu-id="fbdac-192">The `Startup.Configure` method then adds the middleware.</span></span> <span data-ttu-id="fbdac-193">`UseCertificateForwarding` viene chiamato prima delle chiamate a `UseAuthentication` e `UseAuthorization` :</span><span class="sxs-lookup"><span data-stu-id="fbdac-193">`UseCertificateForwarding` is called before the calls to `UseAuthentication` and `UseAuthorization`:</span></span>

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...

    app.UseRouting();

    app.UseCertificateForwarding();
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

<span data-ttu-id="fbdac-194">Una classe separata può essere usata per implementare la logica di convalida.</span><span class="sxs-lookup"><span data-stu-id="fbdac-194">A separate class can be used to implement validation logic.</span></span> <span data-ttu-id="fbdac-195">Poiché in questo esempio viene usato lo stesso certificato autofirmato, assicurarsi che sia possibile usare solo il certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-195">Because the same self-signed certificate is used in this example, ensure that only your certificate can be used.</span></span> <span data-ttu-id="fbdac-196">Verificare che le identificazioni personali del certificato client e del certificato del server corrispondano; in caso contrario, è possibile usare qualsiasi certificato e sarà sufficiente per l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-196">Validate that the thumbprints of both the client certificate and the server certificate match, otherwise any certificate can be used and will be enough to authenticate.</span></span> <span data-ttu-id="fbdac-197">Viene usato all'interno del `AddCertificate` metodo.</span><span class="sxs-lookup"><span data-stu-id="fbdac-197">This would be used inside the `AddCertificate` method.</span></span> <span data-ttu-id="fbdac-198">È anche possibile convalidare l'oggetto o l'autorità emittente qui se si usano certificati intermedi o figlio.</span><span class="sxs-lookup"><span data-stu-id="fbdac-198">You could also validate the subject or the issuer here if you're using intermediate or child certificates.</span></span>

```csharp
using System.IO;
using System.Security.Cryptography.X509Certificates;

namespace AspNetCoreCertificateAuthApi
{
    public class MyCertificateValidationService
    {
        public bool ValidateCertificate(X509Certificate2 clientCertificate)
        {
            // Do not hardcode passwords in production code
            // Use thumbprint or key vault
            var cert = new X509Certificate2(
                Path.Combine("sts_dev_cert.pfx"), "1234");

            if (clientCertificate.Thumbprint == cert.Thumbprint)
            {
                return true;
            }

            return false;
        }
    }
}
```

#### <a name="implement-an-httpclient-using-a-certificate-and-the-httpclienthandler"></a><span data-ttu-id="fbdac-199">Implementare un HttpClient usando un certificato e HttpClientHandler</span><span class="sxs-lookup"><span data-stu-id="fbdac-199">Implement an HttpClient using a certificate and the HttpClientHandler</span></span>

<span data-ttu-id="fbdac-200">L'oggetto `HttpClientHandler` può essere aggiunto direttamente nel costruttore della `HttpClient` classe.</span><span class="sxs-lookup"><span data-stu-id="fbdac-200">The `HttpClientHandler` could be added directly in the constructor of the `HttpClient` class.</span></span> <span data-ttu-id="fbdac-201">È necessario prestare attenzione durante la creazione di istanze di `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="fbdac-201">Care should be taken when creating instances of the `HttpClient`.</span></span> <span data-ttu-id="fbdac-202">`HttpClient`Invierà quindi il certificato a ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="fbdac-202">The `HttpClient` will then send the certificate with each request.</span></span>

```csharp
private async Task<JsonDocument> GetApiDataUsingHttpClientHandler()
{
    var cert = new X509Certificate2(Path.Combine(_environment.ContentRootPath, "sts_dev_cert.pfx"), "1234");
    var handler = new HttpClientHandler();
    handler.ClientCertificates.Add(cert);
    var client = new HttpClient(handler);
     
    var request = new HttpRequestMessage()
    {
        RequestUri = new Uri("https://localhost:44379/api/values"),
        Method = HttpMethod.Get,
    };
    var response = await client.SendAsync(request);
    if (response.IsSuccessStatusCode)
    {
        var responseContent = await response.Content.ReadAsStringAsync();
        var data = JsonDocument.Parse(responseContent);
        return data;
    }
 
    throw new ApplicationException($"Status code: {response.StatusCode}, Error: {response.ReasonPhrase}");
}
```

#### <a name="implement-an-httpclient-using-a-certificate-and-a-named-httpclient-from-ihttpclientfactory"></a><span data-ttu-id="fbdac-203">Implementare un HttpClient usando un certificato e un HttpClient denominato da IHttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="fbdac-203">Implement an HttpClient using a certificate and a named HttpClient from IHttpClientFactory</span></span> 

<span data-ttu-id="fbdac-204">Nell'esempio seguente viene aggiunto un certificato client a un oggetto `HttpClientHandler` utilizzando la `ClientCertificates` proprietà del gestore.</span><span class="sxs-lookup"><span data-stu-id="fbdac-204">In the following example, a client certificate is added to a `HttpClientHandler` using the `ClientCertificates` property from the handler.</span></span> <span data-ttu-id="fbdac-205">Questo gestore può quindi essere utilizzato in un'istanza denominata di un oggetto `HttpClient` utilizzando il `ConfigurePrimaryHttpMessageHandler` metodo.</span><span class="sxs-lookup"><span data-stu-id="fbdac-205">This handler can then be used in a named instance of an `HttpClient` using the `ConfigurePrimaryHttpMessageHandler` method.</span></span> <span data-ttu-id="fbdac-206">Questa impostazione è configurata in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="fbdac-206">This is setup in `Startup.ConfigureServices`:</span></span>

```csharp
var clientCertificate = 
    new X509Certificate2(
      Path.Combine(_environment.ContentRootPath, "sts_dev_cert.pfx"), "1234");
 
var handler = new HttpClientHandler();
handler.ClientCertificates.Add(clientCertificate);
 
services.AddHttpClient("namedClient", c =>
{
}).ConfigurePrimaryHttpMessageHandler(() => handler);
```

<span data-ttu-id="fbdac-207">`IHttpClientFactory`Può quindi essere utilizzato per ottenere l'istanza denominata con il gestore e il certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-207">The `IHttpClientFactory` can then be used to get the named instance with the handler and the certificate.</span></span> <span data-ttu-id="fbdac-208">Il `CreateClient` metodo con il nome del client definito nella `Startup` classe viene utilizzato per ottenere l'istanza di.</span><span class="sxs-lookup"><span data-stu-id="fbdac-208">The `CreateClient` method with the name of the client defined in the `Startup` class is used to get the instance.</span></span> <span data-ttu-id="fbdac-209">La richiesta HTTP può essere inviata utilizzando il client come richiesto.</span><span class="sxs-lookup"><span data-stu-id="fbdac-209">The HTTP request can be sent using the client as required.</span></span>

```csharp
private readonly IHttpClientFactory _clientFactory;
 
public ApiService(IHttpClientFactory clientFactory)
{
    _clientFactory = clientFactory;
}
 
private async Task<JsonDocument> GetApiDataWithNamedClient()
{
    var client = _clientFactory.CreateClient("namedClient");
 
    var request = new HttpRequestMessage()
    {
        RequestUri = new Uri("https://localhost:44379/api/values"),
        Method = HttpMethod.Get,
    };
    var response = await client.SendAsync(request);
    if (response.IsSuccessStatusCode)
    {
        var responseContent = await response.Content.ReadAsStringAsync();
        var data = JsonDocument.Parse(responseContent);
        return data;
    }
 
    throw new ApplicationException($"Status code: {response.StatusCode}, Error: {response.ReasonPhrase}");
}
```

<span data-ttu-id="fbdac-210">Se il certificato corretto viene inviato al server, vengono restituiti i dati.</span><span class="sxs-lookup"><span data-stu-id="fbdac-210">If the correct certificate is sent to the server, the data is returned.</span></span> <span data-ttu-id="fbdac-211">Se non viene inviato alcun certificato o il certificato errato, viene restituito un codice di stato HTTP 403.</span><span class="sxs-lookup"><span data-stu-id="fbdac-211">If no certificate or the wrong certificate is sent, an HTTP 403 status code is returned.</span></span>

### <a name="create-certificates-in-powershell"></a><span data-ttu-id="fbdac-212">Creare certificati in PowerShell</span><span class="sxs-lookup"><span data-stu-id="fbdac-212">Create certificates in PowerShell</span></span>

<span data-ttu-id="fbdac-213">La creazione dei certificati è la parte più difficile per la configurazione di questo flusso.</span><span class="sxs-lookup"><span data-stu-id="fbdac-213">Creating the certificates is the hardest part in setting up this flow.</span></span> <span data-ttu-id="fbdac-214">È possibile creare un certificato radice usando il `New-SelfSignedCertificate` cmdlet di PowerShell.</span><span class="sxs-lookup"><span data-stu-id="fbdac-214">A root certificate can be created using the `New-SelfSignedCertificate` PowerShell cmdlet.</span></span> <span data-ttu-id="fbdac-215">Quando si crea il certificato, usare una password complessa.</span><span class="sxs-lookup"><span data-stu-id="fbdac-215">When creating the certificate, use a strong password.</span></span> <span data-ttu-id="fbdac-216">È importante aggiungere il `KeyUsageProperty` parametro e il `KeyUsage` parametro come illustrato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-216">It's important to add the `KeyUsageProperty` parameter and the `KeyUsage` parameter as shown.</span></span>

#### <a name="create-root-ca"></a><span data-ttu-id="fbdac-217">Crea CA radice</span><span class="sxs-lookup"><span data-stu-id="fbdac-217">Create root CA</span></span>

```powershell
New-SelfSignedCertificate -DnsName "root_ca_dev_damienbod.com", "root_ca_dev_damienbod.com" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(20) -FriendlyName "root_ca_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\root_ca_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath root_ca_dev_damienbod.crt
```

> [!NOTE]
> <span data-ttu-id="fbdac-218">Il `-DnsName` valore del parametro deve corrispondere alla destinazione di distribuzione dell'app.</span><span class="sxs-lookup"><span data-stu-id="fbdac-218">The `-DnsName` parameter value must match the deployment target of the app.</span></span> <span data-ttu-id="fbdac-219">Ad esempio, "localhost" per lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="fbdac-219">For example, "localhost" for development.</span></span>

#### <a name="install-in-the-trusted-root"></a><span data-ttu-id="fbdac-220">Installare nella radice attendibile</span><span class="sxs-lookup"><span data-stu-id="fbdac-220">Install in the trusted root</span></span>

<span data-ttu-id="fbdac-221">Il certificato radice deve essere considerato attendibile nel sistema host.</span><span class="sxs-lookup"><span data-stu-id="fbdac-221">The root certificate needs to be trusted on your host system.</span></span> <span data-ttu-id="fbdac-222">Un certificato radice che non è stato creato da un'autorità di certificazione non sarà considerato attendibile per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="fbdac-222">A root certificate which was not created by a certificate authority won't be trusted by default.</span></span> <span data-ttu-id="fbdac-223">Il collegamento seguente illustra come eseguire questa operazione in Windows:</span><span class="sxs-lookup"><span data-stu-id="fbdac-223">The following link explains how this can be accomplished on Windows:</span></span>

https://social.msdn.microsoft.com/Forums/SqlServer/5ed119ef-1704-4be4-8a4f-ef11de7c8f34/a-certificate-chain-processed-but-terminated-in-a-root-certificate-which-is-not-trusted-by-the

#### <a name="intermediate-certificate"></a><span data-ttu-id="fbdac-224">Certificato intermedio</span><span class="sxs-lookup"><span data-stu-id="fbdac-224">Intermediate certificate</span></span>

<span data-ttu-id="fbdac-225">È ora possibile creare un certificato intermedio dal certificato radice.</span><span class="sxs-lookup"><span data-stu-id="fbdac-225">An intermediate certificate can now be created from the root certificate.</span></span> <span data-ttu-id="fbdac-226">Questa operazione non è necessaria per tutti i casi d'uso, ma potrebbe essere necessario creare molti certificati o attivare o disabilitare gruppi di certificati.</span><span class="sxs-lookup"><span data-stu-id="fbdac-226">This isn't required for all use cases, but you might need to create many certificates or need to activate or disable groups of certificates.</span></span> <span data-ttu-id="fbdac-227">Il `TextExtension` parametro è necessario per impostare la lunghezza del percorso nei vincoli di base del certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-227">The `TextExtension` parameter is required to set the path length in the basic constraints of the certificate.</span></span>

<span data-ttu-id="fbdac-228">Il certificato intermedio può quindi essere aggiunto al certificato intermedio attendibile nel sistema host Windows.</span><span class="sxs-lookup"><span data-stu-id="fbdac-228">The intermediate certificate can then be added to the trusted intermediate certificate in the Windows host system.</span></span>

```powershell
$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint of the root..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "intermediate_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "intermediate_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature -TextExtension @("2.5.29.19={text}CA=1&pathlength=1")

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\intermediate_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath intermediate_dev_damienbod.crt
```

#### <a name="create-child-certificate-from-intermediate-certificate"></a><span data-ttu-id="fbdac-229">Crea certificato figlio da certificato intermedio</span><span class="sxs-lookup"><span data-stu-id="fbdac-229">Create child certificate from intermediate certificate</span></span>

<span data-ttu-id="fbdac-230">Un certificato figlio può essere creato dal certificato intermedio.</span><span class="sxs-lookup"><span data-stu-id="fbdac-230">A child certificate can be created from the intermediate certificate.</span></span> <span data-ttu-id="fbdac-231">Si tratta dell'entità finale e non è necessario creare più certificati figlio.</span><span class="sxs-lookup"><span data-stu-id="fbdac-231">This is the end entity and doesn't need to create more child certificates.</span></span>

```powershell
$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint from the Intermediate certificate..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com"

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath child_a_dev_damienbod.crt
```

#### <a name="create-child-certificate-from-root-certificate"></a><span data-ttu-id="fbdac-232">Crea certificato figlio dal certificato radice</span><span class="sxs-lookup"><span data-stu-id="fbdac-232">Create child certificate from root certificate</span></span>

<span data-ttu-id="fbdac-233">È anche possibile creare un certificato figlio direttamente dal certificato radice.</span><span class="sxs-lookup"><span data-stu-id="fbdac-233">A child certificate can also be created from the root certificate directly.</span></span> 

```powershell
$rootcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint from the root cert..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $rootcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com"

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath child_a_dev_damienbod.crt
```

#### <a name="example-root---intermediate-certificate---certificate"></a><span data-ttu-id="fbdac-234">Esempio di certificato radice-intermedio-certificato</span><span class="sxs-lookup"><span data-stu-id="fbdac-234">Example root - intermediate certificate - certificate</span></span>

```powershell
$mypwdroot = ConvertTo-SecureString -String "1234" -Force -AsPlainText
$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

New-SelfSignedCertificate -DnsName "root_ca_dev_damienbod.com", "root_ca_dev_damienbod.com" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(20) -FriendlyName "root_ca_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature

Get-ChildItem -Path cert:\localMachine\my\0C89639E4E2998A93E423F919B36D4009A0F9991 | Export-PfxCertificate -FilePath C:\git\root_ca_dev_damienbod.pfx -Password $mypwdroot

Export-Certificate -Cert cert:\localMachine\my\0C89639E4E2998A93E423F919B36D4009A0F9991 -FilePath root_ca_dev_damienbod.crt

$rootcert = ( Get-ChildItem -Path cert:\LocalMachine\My\0C89639E4E2998A93E423F919B36D4009A0F9991 )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $rootcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature -TextExtension @("2.5.29.19={text}CA=1&pathlength=1")

Get-ChildItem -Path cert:\localMachine\my\BA9BF91ED35538A01375EFC212A2F46104B33A44 | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\BA9BF91ED35538A01375EFC212A2F46104B33A44 -FilePath child_a_dev_damienbod.crt

$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\BA9BF91ED35538A01375EFC212A2F46104B33A44 )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_b_from_a_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_b_from_a_dev_damienbod.com" 

Get-ChildItem -Path cert:\localMachine\my\141594A0AE38CBBECED7AF680F7945CD51D8F28A | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_b_from_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\141594A0AE38CBBECED7AF680F7945CD51D8F28A -FilePath child_b_from_a_dev_damienbod.crt
```

<span data-ttu-id="fbdac-235">Quando si utilizzano i certificati radice, intermedi o figlio, è possibile convalidare i certificati utilizzando l'identificazione personale o PublicKey come richiesto.</span><span class="sxs-lookup"><span data-stu-id="fbdac-235">When using the root, intermediate, or child certificates, the certificates can be validated using the Thumbprint or PublicKey as required.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Security.Cryptography.X509Certificates;

namespace AspNetCoreCertificateAuthApi
{
    public class MyCertificateValidationService 
    {
        public bool ValidateCertificate(X509Certificate2 clientCertificate)
        {
            return CheckIfThumbprintIsValid(clientCertificate);
        }

        private bool CheckIfThumbprintIsValid(X509Certificate2 clientCertificate)
        {
            var listOfValidThumbprints = new List<string>
            {
                "141594A0AE38CBBECED7AF680F7945CD51D8F28A",
                "0C89639E4E2998A93E423F919B36D4009A0F9991",
                "BA9BF91ED35538A01375EFC212A2F46104B33A44"
            };

            if (listOfValidThumbprints.Contains(clientCertificate.Thumbprint))
            {
                return true;
            }

            return false;
        }
    }
}
```

<a name="occ"></a>

::: moniker range=">= aspnetcore-5.0"

## <a name="certificate-validation-caching"></a><span data-ttu-id="fbdac-236">Memorizzazione nella cache di convalida del certificato</span><span class="sxs-lookup"><span data-stu-id="fbdac-236">Certificate validation caching</span></span>

<span data-ttu-id="fbdac-237">ASP.NET Core 5,0 e versioni successive supportano la possibilità di abilitare la memorizzazione nella cache dei risultati della convalida.</span><span class="sxs-lookup"><span data-stu-id="fbdac-237">ASP.NET Core 5.0 and later versions support the ability to enable caching of validation results.</span></span> <span data-ttu-id="fbdac-238">La memorizzazione nella cache migliora in modo significativo le prestazioni dell'autenticazione del certificato, perché la convalida è un'operazione costosa.</span><span class="sxs-lookup"><span data-stu-id="fbdac-238">The caching dramatically improves performance of certificate authentication, as validation is an expensive operation.</span></span>

<span data-ttu-id="fbdac-239">Per impostazione predefinita, l'autenticazione del certificato Disabilita la memorizzazione nella cache.</span><span class="sxs-lookup"><span data-stu-id="fbdac-239">By default, certificate authentication disables caching.</span></span> <span data-ttu-id="fbdac-240">Per abilitare la memorizzazione nella cache, chiamare `AddCertificateCache` `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="fbdac-240">To enable caching, call `AddCertificateCache` in `Startup.ConfigureServices`:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
            .AddCertificate()
            .AddCertificateCache(options =>
            {
                options.CacheSize = 1024;
                options.CacheEntryExpiration = TimeSpan.FromMinutes(2);
            });
}
```

<span data-ttu-id="fbdac-241">L'implementazione della memorizzazione nella cache predefinita archivia i risultati in memoria.</span><span class="sxs-lookup"><span data-stu-id="fbdac-241">The default caching implementation stores results in memory.</span></span> <span data-ttu-id="fbdac-242">È possibile fornire la propria cache implementando `ICertificateValidationCache` e registrando l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="fbdac-242">You can provide your own cache by implementing `ICertificateValidationCache` and registering it with dependency injection.</span></span> <span data-ttu-id="fbdac-243">Ad esempio: `services.AddSingleton<ICertificateValidationCache, YourCache>()`.</span><span class="sxs-lookup"><span data-stu-id="fbdac-243">For example, `services.AddSingleton<ICertificateValidationCache, YourCache>()`.</span></span>

::: moniker-end

## <a name="optional-client-certificates"></a><span data-ttu-id="fbdac-244">Certificati client facoltativi</span><span class="sxs-lookup"><span data-stu-id="fbdac-244">Optional client certificates</span></span>

<span data-ttu-id="fbdac-245">Questa sezione fornisce informazioni per le app che devono proteggere un subset dell'app con un certificato.</span><span class="sxs-lookup"><span data-stu-id="fbdac-245">This section provides information for apps that must protect a subset of the app with a certificate.</span></span> <span data-ttu-id="fbdac-246">Ad esempio, una Razor pagina o un controller nell'app potrebbe richiedere certificati client.</span><span class="sxs-lookup"><span data-stu-id="fbdac-246">For example, a Razor Page or controller in the app might require client certificates.</span></span> <span data-ttu-id="fbdac-247">Questa operazione presenta i problemi come certificati client:</span><span class="sxs-lookup"><span data-stu-id="fbdac-247">This presents challenges as client certificates:</span></span>
  
* <span data-ttu-id="fbdac-248">Sono una funzionalità TLS, non una funzionalità HTTP.</span><span class="sxs-lookup"><span data-stu-id="fbdac-248">Are a TLS feature, not an HTTP feature.</span></span>
* <span data-ttu-id="fbdac-249">Sono negoziati per connessione e devono essere negoziati all'inizio della connessione prima che i dati HTTP siano disponibili.</span><span class="sxs-lookup"><span data-stu-id="fbdac-249">Are negotiated per-connection and must be be negotiated at the start of the connection before any HTTP data is available.</span></span> <span data-ttu-id="fbdac-250">All'inizio della connessione, è noto solo il Indicazione nome server (SNI) &dagger; .</span><span class="sxs-lookup"><span data-stu-id="fbdac-250">At the start of the connection, only the Server Name Indication (SNI)&dagger; is known.</span></span> <span data-ttu-id="fbdac-251">I certificati client e server vengono negoziati prima della prima richiesta su una connessione e le richieste in genere non sono in grado di rinegoziare.</span><span class="sxs-lookup"><span data-stu-id="fbdac-251">The client and server certificates are negotiated prior to the first request on a connection and requests generally aren't able to renegotiate.</span></span>

<span data-ttu-id="fbdac-252">Il rinegozio TLS è stato un modo obsoleto per implementare certificati client facoltativi.</span><span class="sxs-lookup"><span data-stu-id="fbdac-252">TLS renegotiation was an old way to implement optional client certificates.</span></span> <span data-ttu-id="fbdac-253">Questa operazione non è più consigliata perché:</span><span class="sxs-lookup"><span data-stu-id="fbdac-253">This is no longer recommended because:</span></span>
- <span data-ttu-id="fbdac-254">In HTTP/1.1, la rinegoziazione durante una richiesta POST può causare un deadlock in cui il corpo della richiesta riempie la finestra TCP e non è possibile ricevere i pacchetti di rinegoziazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-254">In HTTP/1.1, renegotiating during a POST request could cause a deadlock where the request body filled up the TCP window and the renegotiation packets can't be received.</span></span>
- <span data-ttu-id="fbdac-255">HTTP/2 [vieta esplicitamente](https://tools.ietf.org/html/rfc7540#section-9.2.1) la rinegoziazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-255">HTTP/2 [explicitly prohibits](https://tools.ietf.org/html/rfc7540#section-9.2.1) renegotiation.</span></span>
- <span data-ttu-id="fbdac-256">TLS 1,3 ha [rimosso](https://tools.ietf.org/html/rfc8740#section-1) il supporto per la rinegoziazione.</span><span class="sxs-lookup"><span data-stu-id="fbdac-256">TLS 1.3 has [removed](https://tools.ietf.org/html/rfc8740#section-1) support for renegotiation.</span></span>

<span data-ttu-id="fbdac-257">ASP.NET Core 5 Preview 7 e versioni successive aggiunge un supporto più pratico per i certificati client facoltativi.</span><span class="sxs-lookup"><span data-stu-id="fbdac-257">ASP.NET Core 5 preview 7 and later adds more convenient support for optional client certificates.</span></span> <span data-ttu-id="fbdac-258">Per ulteriori informazioni, vedere l' [esempio relativo ai certificati facoltativi](https://github.com/dotnet/aspnetcore/tree/9ce4a970a21bace3fb262da9591ed52359309592/src/Security/Authentication/Certificate/samples/Certificate.Optional.Sample).</span><span class="sxs-lookup"><span data-stu-id="fbdac-258">For more information, see the [Optional certificates sample](https://github.com/dotnet/aspnetcore/tree/9ce4a970a21bace3fb262da9591ed52359309592/src/Security/Authentication/Certificate/samples/Certificate.Optional.Sample).</span></span>

<span data-ttu-id="fbdac-259">L'approccio seguente supporta i certificati client facoltativi:</span><span class="sxs-lookup"><span data-stu-id="fbdac-259">The following approach supports optional client certificates:</span></span>

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="fbdac-260">Configurare l'associazione per il dominio e il sottodominio:</span><span class="sxs-lookup"><span data-stu-id="fbdac-260">Set up binding for the domain and subdomain:</span></span>
  * <span data-ttu-id="fbdac-261">Ad esempio, impostare Binding in `contoso.com` e `myClient.contoso.com` .</span><span class="sxs-lookup"><span data-stu-id="fbdac-261">For example, set up bindings on `contoso.com` and `myClient.contoso.com`.</span></span> <span data-ttu-id="fbdac-262">L' `contoso.com` host non richiede un certificato client ma lo `myClient.contoso.com` fa.</span><span class="sxs-lookup"><span data-stu-id="fbdac-262">The `contoso.com` host doesn't require a client certificate but `myClient.contoso.com` does.</span></span>
  * <span data-ttu-id="fbdac-263">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="fbdac-263">For more information, see:</span></span>
    * <span data-ttu-id="fbdac-264">[Gheppio](/fundamentals/servers/kestrel):</span><span class="sxs-lookup"><span data-stu-id="fbdac-264">[Kestrel](/fundamentals/servers/kestrel):</span></span>
      * [<span data-ttu-id="fbdac-265">ListenOptions.UseHttps</span><span class="sxs-lookup"><span data-stu-id="fbdac-265">ListenOptions.UseHttps</span></span>](xref:fundamentals/servers/kestrel/endpoints#listenoptionsusehttps)
      * <xref:Microsoft.AspNetCore.Server.Kestrel.Https.HttpsConnectionAdapterOptions.ClientCertificateMode>
      * <span data-ttu-id="fbdac-266">Si noti che il gheppio non supporta attualmente più configurazioni TLS in un'associazione. sono necessarie due associazioni con indirizzi IP o porte univoci.</span><span class="sxs-lookup"><span data-stu-id="fbdac-266">Note Kestrel does not currently support multiple TLS configurations on one binding, you'll need two bindings with unique IPs or ports.</span></span> <span data-ttu-id="fbdac-267">Vedere https://github.com/dotnet/runtime/issues/31097</span><span class="sxs-lookup"><span data-stu-id="fbdac-267">See https://github.com/dotnet/runtime/issues/31097</span></span>
    * <span data-ttu-id="fbdac-268">IIS</span><span class="sxs-lookup"><span data-stu-id="fbdac-268">IIS</span></span>
      * [<span data-ttu-id="fbdac-269">Hosting di IIS</span><span class="sxs-lookup"><span data-stu-id="fbdac-269">Hosting IIS</span></span>](xref:host-and-deploy/iis/index#create-the-iis-site)
      * [<span data-ttu-id="fbdac-270">Configurare la sicurezza in IIS</span><span class="sxs-lookup"><span data-stu-id="fbdac-270">Configure security on IIS</span></span>](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#configure-ssl-settings-2)
    * <span data-ttu-id="fbdac-271">HTTP.sys: [configurare Windows Server](xref:fundamentals/servers/httpsys#configure-windows-server)</span><span class="sxs-lookup"><span data-stu-id="fbdac-271">HTTP.sys: [Configure Windows Server](xref:fundamentals/servers/httpsys#configure-windows-server)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <span data-ttu-id="fbdac-272">Configurare l'associazione per il dominio e il sottodominio:</span><span class="sxs-lookup"><span data-stu-id="fbdac-272">Set up binding for the domain and subdomain:</span></span>
  * <span data-ttu-id="fbdac-273">Ad esempio, impostare Binding in `contoso.com` e `myClient.contoso.com` .</span><span class="sxs-lookup"><span data-stu-id="fbdac-273">For example, set up bindings on `contoso.com` and `myClient.contoso.com`.</span></span> <span data-ttu-id="fbdac-274">L' `contoso.com` host non richiede un certificato client ma lo `myClient.contoso.com` fa.</span><span class="sxs-lookup"><span data-stu-id="fbdac-274">The `contoso.com` host doesn't require a client certificate but `myClient.contoso.com` does.</span></span>
  * <span data-ttu-id="fbdac-275">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="fbdac-275">For more information, see:</span></span>
    * <span data-ttu-id="fbdac-276">[Gheppio](/fundamentals/servers/kestrel):</span><span class="sxs-lookup"><span data-stu-id="fbdac-276">[Kestrel](/fundamentals/servers/kestrel):</span></span>
      * [<span data-ttu-id="fbdac-277">ListenOptions.UseHttps</span><span class="sxs-lookup"><span data-stu-id="fbdac-277">ListenOptions.UseHttps</span></span>](xref:fundamentals/servers/kestrel#listenoptionsusehttps)
      * <xref:Microsoft.AspNetCore.Server.Kestrel.Https.HttpsConnectionAdapterOptions.ClientCertificateMode>
      * <span data-ttu-id="fbdac-278">Si noti che il gheppio non supporta attualmente più configurazioni TLS in un'associazione. sono necessarie due associazioni con indirizzi IP o porte univoci.</span><span class="sxs-lookup"><span data-stu-id="fbdac-278">Note Kestrel does not currently support multiple TLS configurations on one binding, you'll need two bindings with unique IPs or ports.</span></span> <span data-ttu-id="fbdac-279">Vedere https://github.com/dotnet/runtime/issues/31097</span><span class="sxs-lookup"><span data-stu-id="fbdac-279">See https://github.com/dotnet/runtime/issues/31097</span></span>
    * <span data-ttu-id="fbdac-280">IIS</span><span class="sxs-lookup"><span data-stu-id="fbdac-280">IIS</span></span>
      * [<span data-ttu-id="fbdac-281">Hosting di IIS</span><span class="sxs-lookup"><span data-stu-id="fbdac-281">Hosting IIS</span></span>](xref:host-and-deploy/iis/index#create-the-iis-site)
      * [<span data-ttu-id="fbdac-282">Configurare la sicurezza in IIS</span><span class="sxs-lookup"><span data-stu-id="fbdac-282">Configure security on IIS</span></span>](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#configure-ssl-settings-2)
    * <span data-ttu-id="fbdac-283">HTTP.sys: [configurare Windows Server](xref:fundamentals/servers/httpsys#configure-windows-server)</span><span class="sxs-lookup"><span data-stu-id="fbdac-283">HTTP.sys: [Configure Windows Server](xref:fundamentals/servers/httpsys#configure-windows-server)</span></span>

::: moniker-end

* <span data-ttu-id="fbdac-284">Per le richieste all'app Web che richiedono un certificato client e non dispongono di un certificato:</span><span class="sxs-lookup"><span data-stu-id="fbdac-284">For requests to the web app that require a client certificate and don't have one:</span></span>
  * <span data-ttu-id="fbdac-285">Reindirizza alla stessa pagina usando il sottodominio protetto del certificato client.</span><span class="sxs-lookup"><span data-stu-id="fbdac-285">Redirect to the same page using the client certificate protected subdomain.</span></span>
  * <span data-ttu-id="fbdac-286">Ad esempio, reindirizza a `myClient.contoso.com/requestedPage` .</span><span class="sxs-lookup"><span data-stu-id="fbdac-286">For example, redirect to `myClient.contoso.com/requestedPage`.</span></span> <span data-ttu-id="fbdac-287">Poiché la richiesta a `myClient.contoso.com/requestedPage` è un nome host diverso da `contoso.com/requestedPage` , il client stabilisce una connessione diversa e viene fornito il certificato client.</span><span class="sxs-lookup"><span data-stu-id="fbdac-287">Because the request to `myClient.contoso.com/requestedPage` is a different hostname than `contoso.com/requestedPage`, the client establishes a different connection and the client certificate is provided.</span></span>
  * <span data-ttu-id="fbdac-288">Per altre informazioni, vedere <xref:security/authorization/introduction>.</span><span class="sxs-lookup"><span data-stu-id="fbdac-288">For more information, see <xref:security/authorization/introduction>.</span></span>

<span data-ttu-id="fbdac-289">Lascia domande, commenti e altri commenti sui certificati client facoltativi in [questo](https://github.com/dotnet/AspNetCore.Docs/issues/18720) problema di discussione su GitHub.</span><span class="sxs-lookup"><span data-stu-id="fbdac-289">Leave questions, comments, and other feedback on optional client certificates in [this GitHub discussion](https://github.com/dotnet/AspNetCore.Docs/issues/18720) issue.</span></span>

<span data-ttu-id="fbdac-290">&dagger; Indicazione nome server (SNI) è un'estensione TLS per includere un dominio virtuale come parte della negoziazione SSL.</span><span class="sxs-lookup"><span data-stu-id="fbdac-290">&dagger; Server Name Indication (SNI) is a TLS extension to include a virtual domain as a part of SSL negotiation.</span></span> <span data-ttu-id="fbdac-291">Ciò significa che il nome di dominio virtuale, o un nome host, può essere usato per identificare l'endpoint di rete.</span><span class="sxs-lookup"><span data-stu-id="fbdac-291">This effectively means the virtual domain name, or a hostname, can be used to identify the network end point.</span></span>
