---
title: ASP.NET Core Blazor WebAssembly con gruppi e ruoli di Azure Active Directory
author: guardrex
description: Informazioni su come configurare Blazor WebAssembly per l'uso di gruppi e ruoli di Azure Active Directory.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 01/24/2021
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
uid: blazor/security/webassembly/aad-groups-roles
ms.openlocfilehash: b725a60a310be23f7ceb626d4c543d0df6fadf62
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394759"
---
# <a name="azure-active-directory-aad-groups-administrator-roles-and-app-roles"></a><span data-ttu-id="66db2-103">Gruppi di Azure Active Directory (AAD), ruoli di amministratore e ruoli dell'app</span><span class="sxs-lookup"><span data-stu-id="66db2-103">Azure Active Directory (AAD) groups, Administrator Roles, and App Roles</span></span>

<span data-ttu-id="66db2-104">Azure Active Directory (AAD) fornisce diversi approcci di autorizzazione che possono essere combinati con ASP.NET Core Identity :</span><span class="sxs-lookup"><span data-stu-id="66db2-104">Azure Active Directory (AAD) provides several authorization approaches that can be combined with ASP.NET Core Identity:</span></span>

* <span data-ttu-id="66db2-105">Gruppi</span><span class="sxs-lookup"><span data-stu-id="66db2-105">Groups</span></span>
  * <span data-ttu-id="66db2-106">Sicurezza</span><span class="sxs-lookup"><span data-stu-id="66db2-106">Security</span></span>
  * <span data-ttu-id="66db2-107">Microsoft 365</span><span class="sxs-lookup"><span data-stu-id="66db2-107">Microsoft 365</span></span>
  * <span data-ttu-id="66db2-108">Distribuzione</span><span class="sxs-lookup"><span data-stu-id="66db2-108">Distribution</span></span>
* <span data-ttu-id="66db2-109">Ruoli</span><span class="sxs-lookup"><span data-stu-id="66db2-109">Roles</span></span>
  * <span data-ttu-id="66db2-110">Ruoli di amministratore di AAD</span><span class="sxs-lookup"><span data-stu-id="66db2-110">AAD Administrator Roles</span></span>
  * <span data-ttu-id="66db2-111">Ruoli dell'app</span><span class="sxs-lookup"><span data-stu-id="66db2-111">App Roles</span></span>

<span data-ttu-id="66db2-112">Le indicazioni fornite in questo articolo sono valide per gli Blazor WebAssembly scenari di distribuzione di AAD descritti negli argomenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="66db2-112">The guidance in this article applies to the Blazor WebAssembly AAD deployment scenarios described in the following topics:</span></span>

* [<span data-ttu-id="66db2-113">App autonoma con account Microsoft</span><span class="sxs-lookup"><span data-stu-id="66db2-113">Standalone with Microsoft Accounts</span></span>](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [<span data-ttu-id="66db2-114">App autonoma con AAD</span><span class="sxs-lookup"><span data-stu-id="66db2-114">Standalone with AAD</span></span>](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [<span data-ttu-id="66db2-115">App ospitata con AAD</span><span class="sxs-lookup"><span data-stu-id="66db2-115">Hosted with AAD</span></span>](xref:blazor/security/webassembly/hosted-with-azure-active-directory)

<span data-ttu-id="66db2-116">Le linee guida dell'articolo forniscono istruzioni per le app client e server:</span><span class="sxs-lookup"><span data-stu-id="66db2-116">The article's guidance provides instructions for client and server apps:</span></span>

* <span data-ttu-id="66db2-117">**Client**: app autonome Blazor WebAssembly o **`Client`** app di una soluzione ospitata Blazor .</span><span class="sxs-lookup"><span data-stu-id="66db2-117">**CLIENT**: Standalone Blazor WebAssembly apps or the **`Client`** app of a hosted Blazor solution.</span></span>
* <span data-ttu-id="66db2-118">**Server**: autonomo ASP.NET Core API del server/app per le API Web o l' **`Server`** app di una soluzione ospitata Blazor .</span><span class="sxs-lookup"><span data-stu-id="66db2-118">**SERVER**: Standalone ASP.NET Core server API/web API apps or the **`Server`** app of a hosted Blazor solution.</span></span>

## <a name="scopes"></a><span data-ttu-id="66db2-119">Ambiti</span><span class="sxs-lookup"><span data-stu-id="66db2-119">Scopes</span></span>

<span data-ttu-id="66db2-120">Per consentire [Microsoft Graph chiamate API](/graph/use-the-api) per i dati del profilo utente, dell'assegnazione di ruolo e delle appartenenze ai gruppi, il **client** viene configurato con ( `https://graph.microsoft.com/User.Read` ) [API Graph autorizzazione (ambito)](/graph/permissions-reference) nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="66db2-120">To permit [Microsoft Graph API](/graph/use-the-api) calls for user profile, role assignment, and group membership data, the **CLIENT** is configured with (`https://graph.microsoft.com/User.Read`) [Graph API permission (scope)](/graph/permissions-reference) in the Azure portal.</span></span>

<span data-ttu-id="66db2-121">Un'app **Server** che chiama API Graph per i dati di appartenenza a gruppi e ruoli viene fornita `GroupMember.Read.All` ( `https://graph.microsoft.com/GroupMember.Read.All` ) [API Graph autorizzazione (ambito)](/graph/permissions-reference) nella portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="66db2-121">A **SERVER** app that calls Graph API for role and group membership data is provided `GroupMember.Read.All` (`https://graph.microsoft.com/GroupMember.Read.All`) [Graph API permission (scope)](/graph/permissions-reference) in the Azure portal.</span></span>

<span data-ttu-id="66db2-122">Questi ambiti sono necessari in aggiunta agli ambiti richiesti negli scenari di distribuzione di AAD descritti negli argomenti elencati nella prima sezione di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="66db2-122">These scopes are required in addition to the scopes required in AAD deployment scenarios described by the topics listed in the first section of this article.</span></span>

> [!NOTE]
> <span data-ttu-id="66db2-123">Le parole "autorizzazione" e "ambito" vengono utilizzate in modo interscambiabile nella portale di Azure e in diversi set di documentazione esterni e Microsoft.</span><span class="sxs-lookup"><span data-stu-id="66db2-123">The words "permission" and "scope" are used interchangeably in the Azure portal and in various Microsoft and external documentation sets.</span></span> <span data-ttu-id="66db2-124">Questo articolo usa la parola "scope" in tutto per le autorizzazioni assegnate a un'app nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="66db2-124">This article uses the word "scope" throughout for the permissions assigned to an app in the Azure portal.</span></span>

## <a name="group-membership-claims-attribute"></a><span data-ttu-id="66db2-125">Attributo delle attestazioni di appartenenza a gruppo</span><span class="sxs-lookup"><span data-stu-id="66db2-125">Group Membership Claims attribute</span></span>

<span data-ttu-id="66db2-126">Nel manifesto dell'app nella portale di Azure per le app **client** e **Server** , impostare l' [ `groupMembershipClaims` attributo](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute) su `All` .</span><span class="sxs-lookup"><span data-stu-id="66db2-126">In the app's manifest in the the Azure portal for **CLIENT** and **SERVER** apps, set the [`groupMembershipClaims` attribute](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute) to `All`.</span></span> <span data-ttu-id="66db2-127">Il valore `All` risultante è il recupero di tutti i gruppi di sicurezza, i gruppi di distribuzione e i ruoli di cui l'utente connesso è membro.</span><span class="sxs-lookup"><span data-stu-id="66db2-127">A value of `All` results in obtaining all of the security groups, distribution groups, and roles that the signed-in user is a member of.</span></span>

1. <span data-ttu-id="66db2-128">Aprire la registrazione portale di Azure dell'app.</span><span class="sxs-lookup"><span data-stu-id="66db2-128">Open the app's Azure portal registration.</span></span>
1. <span data-ttu-id="66db2-129">Selezionare **Gestisci**  >  **manifesto** nella barra laterale.</span><span class="sxs-lookup"><span data-stu-id="66db2-129">Select **Manage** > **Manifest** in the sidebar.</span></span>
1. <span data-ttu-id="66db2-130">Trovare l' `groupMembershipClaims` attributo.</span><span class="sxs-lookup"><span data-stu-id="66db2-130">Find the `groupMembershipClaims` attribute.</span></span>
1. <span data-ttu-id="66db2-131">Impostare il valore su `All`.</span><span class="sxs-lookup"><span data-stu-id="66db2-131">Set the value to `All`.</span></span>
1. <span data-ttu-id="66db2-132">Fare clic sul pulsante **Salva**.</span><span class="sxs-lookup"><span data-stu-id="66db2-132">Select the **Save** button.</span></span>

```json
"groupMembershipClaims": "All",
```

## <a name="custom-user-account"></a><span data-ttu-id="66db2-133">Account utente personalizzato</span><span class="sxs-lookup"><span data-stu-id="66db2-133">Custom user account</span></span>

<span data-ttu-id="66db2-134">Assegnare gli utenti ai gruppi di sicurezza AAD e ai ruoli di amministratore di AAD nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="66db2-134">Assign users to AAD security groups and AAD Administrator Roles in the Azure portal.</span></span>

<span data-ttu-id="66db2-135">Gli esempi in questo articolo:</span><span class="sxs-lookup"><span data-stu-id="66db2-135">The examples in this article:</span></span>

* <span data-ttu-id="66db2-136">Si supponga che un utente sia assegnato al ruolo di *amministratore fatturazione* di AAD nel tenant portale di Azure AAD per l'autorizzazione ad accedere ai dati dell'API del server.</span><span class="sxs-lookup"><span data-stu-id="66db2-136">Assume that a user is assigned to the AAD *Billing Administrator* role in the Azure portal AAD tenant for authorization to access server API data.</span></span>
* <span data-ttu-id="66db2-137">Usare i [criteri di autorizzazione](xref:security/authorization/policies) per controllare l'accesso all'interno delle app **client** e **server** .</span><span class="sxs-lookup"><span data-stu-id="66db2-137">Use [authorization policies](xref:security/authorization/policies) to control access within the **CLIENT** and **SERVER** apps.</span></span>

<span data-ttu-id="66db2-138">Nell'app **client** Estendi <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> per includere le proprietà per:</span><span class="sxs-lookup"><span data-stu-id="66db2-138">In the **CLIENT** app, extend <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> to include properties for:</span></span>

* <span data-ttu-id="66db2-139">`Roles`: Matrice dei ruoli dell'app AAD (analizzata nella sezione [ruoli app](#app-roles) )</span><span class="sxs-lookup"><span data-stu-id="66db2-139">`Roles`: AAD App Roles array (covered in the [App Roles](#app-roles) section)</span></span>
* <span data-ttu-id="66db2-140">`Wids`: Ruoli di amministratore di AAD in [attestazioni ID note ( `wids` )](/azure/active-directory/develop/access-tokens#payload-claims)</span><span class="sxs-lookup"><span data-stu-id="66db2-140">`Wids`: AAD Administrator Roles in [well-known IDs claims (`wids`)](/azure/active-directory/develop/access-tokens#payload-claims)</span></span>
* <span data-ttu-id="66db2-141">`Oid`: [Attestazione dell'identificatore di oggetto `oid` ](/azure/active-directory/develop/id-tokens#payload-claims) non modificabile () (identifica in modo univoco un utente all'interno e tra i tenant)</span><span class="sxs-lookup"><span data-stu-id="66db2-141">`Oid`: Immutable [object identifier claim (`oid`)](/azure/active-directory/develop/id-tokens#payload-claims) (uniquely identifies a user within and across tenants)</span></span>

<span data-ttu-id="66db2-142">Assegnare una matrice vuota a ogni proprietà della matrice in modo che il controllo `null` non sia necessario quando queste proprietà vengono usate nei `foreach` cicli.</span><span class="sxs-lookup"><span data-stu-id="66db2-142">Assign an empty array to each array property so that checking for `null` isn't required when these properties are used in `foreach` loops.</span></span>

<span data-ttu-id="66db2-143">`CustomUserAccount.cs`:</span><span class="sxs-lookup"><span data-stu-id="66db2-143">`CustomUserAccount.cs`:</span></span>

```csharp
using System;
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("roles")]
    public string[] Roles { get; set; } = Array.Empty<string>();

    [JsonPropertyName("wids")]
    public string[] Wids { get; set; } = Array.Empty<string>();

    [JsonPropertyName("oid")]
    public string Oid { get; set; }
}
```

<span data-ttu-id="66db2-144">Aggiungere un riferimento al pacchetto per il file di progetto dell'app **client** [`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph) .</span><span class="sxs-lookup"><span data-stu-id="66db2-144">Add a package reference to the **CLIENT** app's project file for [`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph).</span></span>

<span data-ttu-id="66db2-145">Aggiungere le classi e la configurazione dell'utilità Graph SDK nella sezione *Graph SDK* dell' <xref:blazor/security/webassembly/graph-api#graph-sdk> articolo.</span><span class="sxs-lookup"><span data-stu-id="66db2-145">Add the Graph SDK utility classes and configuration in the *Graph SDK* section of the <xref:blazor/security/webassembly/graph-api#graph-sdk> article.</span></span> <span data-ttu-id="66db2-146">Nella `GraphClientExtensions` classe specificare l' `User.Read` ambito per il token di accesso nel `AuthenticateRequestAsync` Metodo:</span><span class="sxs-lookup"><span data-stu-id="66db2-146">In the `GraphClientExtensions` class, specify the `User.Read` scope for the access token in the `AuthenticateRequestAsync` method:</span></span>

```csharp
var result = await TokenProvider.RequestAccessToken(
    new AccessTokenRequestOptions()
    {
        Scopes = new[] { "https://graph.microsoft.com/User.Read" }
    });
```

<span data-ttu-id="66db2-147">Aggiungere la factory di account utente personalizzata seguente all'app **client** .</span><span class="sxs-lookup"><span data-stu-id="66db2-147">Add the following custom user account factory to the **CLIENT** app.</span></span> <span data-ttu-id="66db2-148">La Factory utente personalizzata viene usata per stabilire:</span><span class="sxs-lookup"><span data-stu-id="66db2-148">The custom user factory is used to establish:</span></span>

* <span data-ttu-id="66db2-149">Attestazioni del ruolo app ( `appRole` ) (descritte nella sezione [ruoli app](#app-roles) )</span><span class="sxs-lookup"><span data-stu-id="66db2-149">App Role claims (`appRole`) (covered in the [App Roles](#app-roles) section)</span></span>
* <span data-ttu-id="66db2-150">Attestazioni ruolo amministratore AAD ( `directoryRole` )</span><span class="sxs-lookup"><span data-stu-id="66db2-150">AAD Administrator Role claims (`directoryRole`)</span></span>
* <span data-ttu-id="66db2-151">Un esempio di attestazione dati del profilo utente per il numero di cellulare dell'utente ( `mobilePhone` )</span><span class="sxs-lookup"><span data-stu-id="66db2-151">An example user profile data claim for the user's mobile phone number (`mobilePhone`)</span></span>
* <span data-ttu-id="66db2-152">Attestazioni del gruppo AAD ( `directoryGroup` )</span><span class="sxs-lookup"><span data-stu-id="66db2-152">AAD Group claims (`directoryGroup`)</span></span>

<span data-ttu-id="66db2-153">`CustomAccountFactory.cs`:</span><span class="sxs-lookup"><span data-stu-id="66db2-153">`CustomAccountFactory.cs`:</span></span>

```csharp
using System;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Graph;

public class CustomAccountFactory
    : AccountClaimsPrincipalFactory<CustomUserAccount>
{
    private readonly ILogger<CustomAccountFactory> logger;
    private readonly IServiceProvider serviceProvider;

    public CustomAccountFactory(IAccessTokenProviderAccessor accessor,
        IServiceProvider serviceProvider,
        ILogger<CustomAccountFactory> logger)
        : base(accessor)
    {
        this.serviceProvider = serviceProvider;
        this.logger = logger;
    }
    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        CustomUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            foreach (var role in account.Roles)
            {
                userIdentity.AddClaim(new Claim("appRole", role));
            }

            foreach (var wid in account.Wids)
            {
                userIdentity.AddClaim(new Claim("directoryRole", wid));
            }

            try
            {
                var graphClient = ActivatorUtilities
                    .CreateInstance<GraphServiceClient>(serviceProvider);

                var requestMe = graphClient.Me.Request();
                var user = await requestMe.GetAsync();

                if (user != null)
                {
                    userIdentity.AddClaim(new Claim("mobilePhone",
                        user.MobilePhone));
                }

                var requestMemberOf = graphClient.Users[account.Oid].MemberOf;
                var memberships = await requestMemberOf.Request().GetAsync();

                if (memberships != null)
                {
                    foreach (var entry in memberships)
                    {
                        if (entry.ODataType == "#microsoft.graph.group")
                        {
                            userIdentity.AddClaim(
                                new Claim("directoryGroup", entry.Id));
                        }
                    }
                }
            }
            catch (ServiceException exception)
            {
                logger.LogError("Graph API service failure: {Message}",
                    exception.Message);
            }
        }

        return initialUser;
    }
}
```

<span data-ttu-id="66db2-154">Il codice precedente non include le appartenenze transitive.</span><span class="sxs-lookup"><span data-stu-id="66db2-154">The preceding code doesn't include transitive memberships.</span></span> <span data-ttu-id="66db2-155">Se l'app richiede attestazioni di appartenenza a gruppi dirette e transitive, sostituire la `MemberOf` Proprietà ( `IUserMemberOfCollectionWithReferencesRequestBuilder` ) con `TransitiveMemberOf` ( `IUserTransitiveMemberOfCollectionWithReferencesRequestBuilder` ).</span><span class="sxs-lookup"><span data-stu-id="66db2-155">If the app requires direct and transitive group membership claims, replace the `MemberOf` property (`IUserMemberOfCollectionWithReferencesRequestBuilder`) with `TransitiveMemberOf` (`IUserTransitiveMemberOfCollectionWithReferencesRequestBuilder`).</span></span>

<span data-ttu-id="66db2-156">Il codice precedente ignora le attestazioni di appartenenza a gruppi ( `groups` ) che sono ruoli `#microsoft.graph.directoryRole` di amministratore di AAD (tipo) perché i valori GUID restituiti dalla Identity piattaforma Microsoft 2,0 sono **ID entità** del ruolo amministratore di AAD e non [**ID modello di ruolo**](/azure/active-directory/roles/permissions-reference#role-template-ids).</span><span class="sxs-lookup"><span data-stu-id="66db2-156">The preceding code ignores group membership claims (`groups`) that are AAD Administrator Roles (`#microsoft.graph.directoryRole` type) because the GUID values returned by the Microsoft Identity Platform 2.0 are AAD Administrator Role **entity IDs** and not [**Role Template IDs**](/azure/active-directory/roles/permissions-reference#role-template-ids).</span></span> <span data-ttu-id="66db2-157">Gli ID entità non sono stabili tra i tenant nella Identity piattaforma Microsoft 2,0 e non devono essere usati per creare criteri di autorizzazione per gli utenti nelle app.</span><span class="sxs-lookup"><span data-stu-id="66db2-157">Entity IDs aren't stable across tenants in Microsoft Identity Platform 2.0 and shouldn't be used to create authorization policies for users in apps.</span></span> <span data-ttu-id="66db2-158">Usare sempre gli **ID dei modelli di ruolo** per i ruoli di amministratore di AAD **forniti dalle `wids` attestazioni**.</span><span class="sxs-lookup"><span data-stu-id="66db2-158">Always use **Role Template IDs** for AAD Administrator Roles **provided by `wids` claims**.</span></span>

<span data-ttu-id="66db2-159">Nell' `Program.Main` app **client** configurare l'autenticazione MSAL per l'uso della factory dell'account utente personalizzata.</span><span class="sxs-lookup"><span data-stu-id="66db2-159">In `Program.Main` of the **CLIENT** app, configure the MSAL authentication to use the custom user account factory.</span></span>

<span data-ttu-id="66db2-160">`Program.cs`:</span><span class="sxs-lookup"><span data-stu-id="66db2-160">`Program.cs`:</span></span>

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Extensions.Configuration;

...

builder.Services.AddMsalAuthentication<RemoteAuthenticationState,
    CustomUserAccount>(options =>
{
    ...
})
.AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, CustomUserAccount,
    CustomAccountFactory>();

...

builder.Services.AddGraphClient();
```

## <a name="authorization-configuration"></a><span data-ttu-id="66db2-161">Configurazione dell'autorizzazione</span><span class="sxs-lookup"><span data-stu-id="66db2-161">Authorization configuration</span></span>

<span data-ttu-id="66db2-162">Nell'app **client** creare un [criterio](xref:security/authorization/policies) per ogni [ruolo app](#app-roles), ruolo amministratore AAD o gruppo di sicurezza in `Program.Main` .</span><span class="sxs-lookup"><span data-stu-id="66db2-162">In the **CLIENT** app, create a [policy](xref:security/authorization/policies) for each [App Role](#app-roles), AAD Administrator Role, or security group in `Program.Main`.</span></span> <span data-ttu-id="66db2-163">L'esempio seguente crea un criterio per il ruolo di *amministratore fatturazione* di AAD:</span><span class="sxs-lookup"><span data-stu-id="66db2-163">The following example creates a policy for the AAD *Billing Administrator* role:</span></span>

```csharp
builder.Services.AddAuthorizationCore(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("directoryRole", 
            "b0f54661-2d74-4c50-afa3-1ec803f12efe"));
});
```

<span data-ttu-id="66db2-164">Per l'elenco completo degli ID per i ruoli di amministratore di AAD, vedere [ID dei modelli di ruolo](/azure/active-directory/roles/permissions-reference#role-template-ids) nella documentazione di Azure.</span><span class="sxs-lookup"><span data-stu-id="66db2-164">For the complete list of IDs for AAD Administrator Roles, see [Role template IDs](/azure/active-directory/roles/permissions-reference#role-template-ids) in the Azure documentation.</span></span> <span data-ttu-id="66db2-165">Per ulteriori informazioni sui criteri di autorizzazione, vedere <xref:security/authorization/policies> .</span><span class="sxs-lookup"><span data-stu-id="66db2-165">For more information on authorization policies, see <xref:security/authorization/policies>.</span></span>

<span data-ttu-id="66db2-166">Negli esempi seguenti l'app **client** usa il criterio precedente per autorizzare l'utente.</span><span class="sxs-lookup"><span data-stu-id="66db2-166">In the following examples, the **CLIENT** app uses the preceding policy to authorize the user.</span></span>

<span data-ttu-id="66db2-167">Il [ `AuthorizeView` componente](xref:blazor/security/index#authorizeview-component) funziona con i criteri:</span><span class="sxs-lookup"><span data-stu-id="66db2-167">The [`AuthorizeView` component](xref:blazor/security/index#authorizeview-component) works with the policy:</span></span>

```razor
<AuthorizeView Policy="BillingAdministrator">
    <Authorized>
        <p>
            The user is in the 'Billing Administrator' AAD Administrator Role
            and can see this content.
        </p>
    </Authorized>
    <NotAuthorized>
        <p>
            The user is NOT in the 'Billing Administrator' role and sees this
            content.
        </p>
    </NotAuthorized>
</AuthorizeView>
```

<span data-ttu-id="66db2-168">L'accesso a un intero componente può essere basato sui criteri mediante una [ `[Authorize]` direttiva attribute](xref:blazor/security/index#authorize-attribute) ( <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ):</span><span class="sxs-lookup"><span data-stu-id="66db2-168">Access to an entire component can be based on the policy using an [`[Authorize]` attribute directive](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>):</span></span>

```razor
@page "/"
@using Microsoft.AspNetCore.Authorization
@attribute [Authorize(Policy = "BillingAdministrator")]
```

<span data-ttu-id="66db2-169">Se l'utente non è connesso, viene reindirizzato alla pagina di accesso di AAD e quindi di nuovo al componente dopo l'accesso.</span><span class="sxs-lookup"><span data-stu-id="66db2-169">If the user isn't logged in, they're redirected to the AAD sign-in page and then back to the component after they sign in.</span></span>

<span data-ttu-id="66db2-170">Un controllo dei criteri può essere [eseguito anche nel codice con la logica procedurale](xref:blazor/security/index#procedural-logic).</span><span class="sxs-lookup"><span data-stu-id="66db2-170">A policy check can also be [performed in code with procedural logic](xref:blazor/security/index#procedural-logic).</span></span>

<span data-ttu-id="66db2-171">`Pages/CheckPolicy.razor`:</span><span class="sxs-lookup"><span data-stu-id="66db2-171">`Pages/CheckPolicy.razor`:</span></span>

```razor
@page "/checkpolicy"
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<h1>Check Policy</h1>

<p>This component checks a policy in code.</p>

<button @onclick="CheckPolicy">Check 'BillingAdministrator' policy</button>

<p>Policy Message: @policyMessage</p>

@code {
    private string policyMessage = "Check hasn't been made yet.";

    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async void CheckPolicy()
    {
        var user = (await authenticationStateTask).User;

        if ((await AuthorizationService.AuthorizeAsync(user, 
            "BillingAdministrator")).Succeeded)
        {
            policyMessage = "Yes! The 'BillingAdministrator' policy is met.";
        }
        else
        {
            policyMessage = "No! 'BillingAdministrator' policy is NOT met.";
        }
    }
}
```

## <a name="authorize-server-apiweb-api-access"></a><span data-ttu-id="66db2-172">Autorizzare l'accesso API Server/API Web</span><span class="sxs-lookup"><span data-stu-id="66db2-172">Authorize server API/web API access</span></span>

<span data-ttu-id="66db2-173">Un'app per le API **Server** può autorizzare gli utenti ad accedere a endpoint API protetti con [criteri di autorizzazione](xref:security/authorization/policies) per i gruppi di sicurezza, i ruoli di amministratore di AAD e i ruoli dell'app quando un token di accesso contiene `groups` `wids` `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` attestazioni, e.</span><span class="sxs-lookup"><span data-stu-id="66db2-173">A **SERVER** API app can authorize users to access secure API endpoints with [authorization policies](xref:security/authorization/policies) for security groups, AAD Administrator Roles, and App Roles when an access token contains `groups`, `wids`, and `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` claims.</span></span> <span data-ttu-id="66db2-174">L'esempio seguente crea un criterio per il ruolo di *amministratore fatturazione* di AAD in `Startup.ConfigureServices` usando le `wids` attestazioni (ID noto/ID modello di ruolo):</span><span class="sxs-lookup"><span data-stu-id="66db2-174">The following example creates a policy for the AAD *Billing Administrator* role in `Startup.ConfigureServices` using the `wids` (well-known IDs/Role Template IDs) claims:</span></span>

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("wids", "b0f54661-2d74-4c50-afa3-1ec803f12efe"));
});
```

<span data-ttu-id="66db2-175">Per l'elenco completo degli ID per i ruoli di amministratore di AAD, vedere [ID dei modelli di ruolo](/azure/active-directory/roles/permissions-reference#role-template-ids) nella documentazione di Azure.</span><span class="sxs-lookup"><span data-stu-id="66db2-175">For the complete list of IDs for AAD Administrator Roles, see [Role template IDs](/azure/active-directory/roles/permissions-reference#role-template-ids) in the Azure documentation.</span></span> <span data-ttu-id="66db2-176">Per ulteriori informazioni sui criteri di autorizzazione, vedere <xref:security/authorization/policies> .</span><span class="sxs-lookup"><span data-stu-id="66db2-176">For more information on authorization policies, see <xref:security/authorization/policies>.</span></span>

<span data-ttu-id="66db2-177">L'accesso a un controller nell'app **Server** può essere basato sull'uso di un [ `[Authorize]` attributo](xref:security/authorization/simple) con il nome del criterio (documentazione API: <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ).</span><span class="sxs-lookup"><span data-stu-id="66db2-177">Access to a controller in the **SERVER** app can be based on using an [`[Authorize]` attribute](xref:security/authorization/simple) with the name of the policy (API documentation: <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>).</span></span>

<span data-ttu-id="66db2-178">Nell'esempio seguente viene limitato l'accesso ai dati di fatturazione da `BillingDataController` ad amministratori della fatturazione di Azure con un nome di criterio `BillingAdministrator` :</span><span class="sxs-lookup"><span data-stu-id="66db2-178">The following example limits access to billing data from the `BillingDataController` to Azure Billing Administrators with a policy name of `BillingAdministrator`:</span></span>

```csharp
...
using Microsoft.AspNetCore.Authorization;

[Authorize(Policy = "BillingAdministrator")]
[ApiController]
[Route("[controller]")]
public class BillingDataController : ControllerBase
{
    ...
}
```

<span data-ttu-id="66db2-179">Per altre informazioni, vedere <xref:security/authorization/policies>.</span><span class="sxs-lookup"><span data-stu-id="66db2-179">For more information, see <xref:security/authorization/policies>.</span></span>

## <a name="app-roles"></a><span data-ttu-id="66db2-180">Ruoli dell'app</span><span class="sxs-lookup"><span data-stu-id="66db2-180">App Roles</span></span>

<span data-ttu-id="66db2-181">Per configurare l'app nel portale di Azure per fornire attestazioni di appartenenza ai ruoli dell'app, vedere [procedura: aggiungere i ruoli dell'app nell'applicazione e riceverli nel token](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) nella documentazione di Azure.</span><span class="sxs-lookup"><span data-stu-id="66db2-181">To configure the app in the Azure portal to provide App Roles membership claims, see [How to: Add app roles in your application and receive them in the token](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) in the Azure documentation.</span></span>

<span data-ttu-id="66db2-182">Nell'esempio seguente si presuppone che le app **client** e **Server** siano configurate con due ruoli e che i ruoli siano assegnati a un utente di test:</span><span class="sxs-lookup"><span data-stu-id="66db2-182">The following example assumes that the **CLIENT** and **SERVER** apps are configured with two roles, and the roles are assigned to a test user:</span></span>

* `admin`
* `developer`

> [!NOTE]
> <span data-ttu-id="66db2-183">Quando si sviluppa un' Blazor WebAssembly app ospitata o una coppia client-server di app autonome (un'app autonoma Blazor WebAssembly e un'api di ASP.NET Core Server autonoma o un'app per le API Web), la `appRoles` Proprietà Manifest sia del client che del server portale di Azure le registrazioni dell'app devono includere gli stessi ruoli configurati.</span><span class="sxs-lookup"><span data-stu-id="66db2-183">When developing a hosted Blazor WebAssembly app or a client-server pair of standalone apps (a standalone Blazor WebAssembly app and a standalone ASP.NET Core server API/web API app), the `appRoles` manifest property of both the client and the server Azure portal app registrations must include the same configured roles.</span></span> <span data-ttu-id="66db2-184">Dopo aver stabilito i ruoli nel manifesto dell'app client, copiarli interamente nel manifesto dell'app Server.</span><span class="sxs-lookup"><span data-stu-id="66db2-184">After establishing the roles in the client app's manifest, copy them in their entirety to the server app's manifest.</span></span> <span data-ttu-id="66db2-185">Se non si esegue il mirroring del manifesto `appRoles` tra le registrazioni dell'app client e server, le attestazioni dei ruoli non vengono stabilite per gli utenti autenticati dell'API Server/API Web, anche se il relativo token di accesso ha le attestazioni dei ruoli corrette.</span><span class="sxs-lookup"><span data-stu-id="66db2-185">If you don't mirror the manifest `appRoles` between the client and server app registrations, role claims aren't established for authenticated users of the server API/web API, even if their access token has the correct roles claims.</span></span>

> [!NOTE]
> <span data-ttu-id="66db2-186">Sebbene non sia possibile assegnare ruoli a gruppi senza un account di Azure AD Premium, è possibile assegnare ruoli agli utenti e ricevere un'attestazione dei ruoli per gli utenti con un account Azure standard.</span><span class="sxs-lookup"><span data-stu-id="66db2-186">Although you can't assign roles to groups without an Azure AD Premium account, you can assign roles to users and receive a roles claim for users with a standard Azure account.</span></span> <span data-ttu-id="66db2-187">Le indicazioni fornite in questa sezione non richiedono un account AAD Premium.</span><span class="sxs-lookup"><span data-stu-id="66db2-187">The guidance in this section doesn't require an AAD Premium account.</span></span>
>
> <span data-ttu-id="66db2-188">Nel portale di Azure vengono assegnati più ruoli aggiungendo **_nuovamente un utente_** per ogni assegnazione di ruolo aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="66db2-188">Multiple roles are assigned in the Azure portal by **_re-adding a user_** for each additional role assignment.</span></span>

<span data-ttu-id="66db2-189">Il `CustomAccountFactory` illustrato nella sezione [account utente personalizzato](#custom-user-account) è configurato per agire su un' `roles` attestazione con un valore di matrice JSON.</span><span class="sxs-lookup"><span data-stu-id="66db2-189">The `CustomAccountFactory` shown in the [Custom user account](#custom-user-account) section is set up to act on a `roles` claim with a JSON array value.</span></span> <span data-ttu-id="66db2-190">Aggiungere e registrare nell' `CustomAccountFactory` app **client** come illustrato nella sezione [account utente personalizzato](#custom-user-account) .</span><span class="sxs-lookup"><span data-stu-id="66db2-190">Add and register the `CustomAccountFactory` in the **CLIENT** app as shown in the [Custom user account](#custom-user-account) section.</span></span> <span data-ttu-id="66db2-191">Non è necessario fornire codice per rimuovere l' `roles` attestazione originale perché viene automaticamente rimossa dal Framework.</span><span class="sxs-lookup"><span data-stu-id="66db2-191">There's no need to provide code to remove the original `roles` claim because it's automatically removed by the framework.</span></span>

<span data-ttu-id="66db2-192">In `Program.Main` di un'app **client** specificare l'attestazione denominata " `appRole` " come attestazione del ruolo per i <xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> Controlli:</span><span class="sxs-lookup"><span data-stu-id="66db2-192">In `Program.Main` of a **CLIENT** app, specify the claim named "`appRole`" as the role claim for <xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> checks:</span></span>

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.UserOptions.RoleClaim = "appRole";
});
```

> [!NOTE]
> <span data-ttu-id="66db2-193">Se si preferisce usare l' `directoryRoles` attestazione (Aggiungi ruoli di amministratore), assegnare " `directoryRoles` " a <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationUserOptions.RoleClaim?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="66db2-193">If you prefer to use the `directoryRoles` claim (ADD Administrator Roles), assign "`directoryRoles`" to the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationUserOptions.RoleClaim?displayProperty=nameWithType>.</span></span>

<span data-ttu-id="66db2-194">In `Startup.ConfigureServices` di un'app **Server** specificare l'attestazione denominata " `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` " come attestazione del ruolo per i <xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> Controlli:</span><span class="sxs-lookup"><span data-stu-id="66db2-194">In `Startup.ConfigureServices` of a **SERVER** app, specify the claim named "`http://schemas.microsoft.com/ws/2008/06/identity/claims/role`" as the role claim for <xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> checks:</span></span>

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(options =>
    {
        Configuration.Bind("AzureAd", options);
        options.TokenValidationParameters.RoleClaimType = 
            "http://schemas.microsoft.com/ws/2008/06/identity/claims/role";
    },
    options => { Configuration.Bind("AzureAd", options); });
```

> [!NOTE]
> <span data-ttu-id="66db2-195">Se si preferisce usare l' `wids` attestazione (Aggiungi ruoli di amministratore), assegnare " `wids` " a <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.RoleClaimType?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="66db2-195">If you prefer to use the `wids` claim (ADD Administrator Roles), assign "`wids`" to the <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.RoleClaimType?displayProperty=nameWithType>.</span></span>

<span data-ttu-id="66db2-196">A questo punto sono funzionali gli approcci di autorizzazione per i componenti.</span><span class="sxs-lookup"><span data-stu-id="66db2-196">Component authorization approaches are functional at this point.</span></span> <span data-ttu-id="66db2-197">Qualsiasi meccanismo di autorizzazione nei componenti dell'app **client** può usare il `admin` ruolo per autorizzare l'utente:</span><span class="sxs-lookup"><span data-stu-id="66db2-197">Any of the authorization mechanisms in components of the **CLIENT** app can use the `admin` role to authorize the user:</span></span>

* [<span data-ttu-id="66db2-198">`AuthorizeView` componente</span><span class="sxs-lookup"><span data-stu-id="66db2-198">`AuthorizeView` component</span></span>](xref:blazor/security/index#authorizeview-component)

  ```razor
  <AuthorizeView Roles="admin">
  ```

* <span data-ttu-id="66db2-199">[ `[Authorize]` attribute](xref:blazor/security/index#authorize-attribute) (direttiva <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ) ()</span><span class="sxs-lookup"><span data-stu-id="66db2-199">[`[Authorize]` attribute directive](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)</span></span>

  ```razor
  @attribute [Authorize(Roles = "admin")]
  ```

* [<span data-ttu-id="66db2-200">Logica procedurale</span><span class="sxs-lookup"><span data-stu-id="66db2-200">Procedural logic</span></span>](xref:blazor/security/index#procedural-logic)

  ```csharp
  var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
  var user = authState.User;

  if (user.IsInRole("admin")) { ... }
  ```

<span data-ttu-id="66db2-201">Sono supportati più test di ruolo:</span><span class="sxs-lookup"><span data-stu-id="66db2-201">Multiple role tests are supported:</span></span>

* <span data-ttu-id="66db2-202">Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con il `AuthorizeView` componente:</span><span class="sxs-lookup"><span data-stu-id="66db2-202">Require that the user be in **either** the `admin` **or** `developer` role with the `AuthorizeView` component:</span></span>

  ```razor
  <AuthorizeView Roles="admin, developer">
      ...
  </AuthorizeView>
  ```

* <span data-ttu-id="66db2-203">Richiedere **che l'utente sia nei** `admin` ruoli **e** `developer` con il `AuthorizeView` componente:</span><span class="sxs-lookup"><span data-stu-id="66db2-203">Require that the user be in **both** the `admin` **and** `developer` roles with the `AuthorizeView` component:</span></span>

  ```razor
  <AuthorizeView Roles="admin">
      <AuthorizeView Roles="developer">
          ...
      </AuthorizeView>
  </AuthorizeView>
  ```

* <span data-ttu-id="66db2-204">Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con l' `[Authorize]` attributo:</span><span class="sxs-lookup"><span data-stu-id="66db2-204">Require that the user be in **either** the `admin` **or** `developer` role with the `[Authorize]` attribute:</span></span>

  ```razor
  @attribute [Authorize(Roles = "admin, developer")]
  ```

* <span data-ttu-id="66db2-205">Richiedere **che l'utente sia nei** `admin` ruoli **e** `developer` con l' `[Authorize]` attributo:</span><span class="sxs-lookup"><span data-stu-id="66db2-205">Require that the user be in **both** the `admin` **and** `developer` roles with the `[Authorize]` attribute:</span></span>

  ```razor
  @attribute [Authorize(Roles = "admin")]
  @attribute [Authorize(Roles = "developer")]
  ```

* <span data-ttu-id="66db2-206">Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con codice procedurale:</span><span class="sxs-lookup"><span data-stu-id="66db2-206">Require that the user be in **either** the `admin` **or** `developer` role with procedural code:</span></span>

  ```razor
  @code {
      private async Task DoSomething()
      {
          var authState = await AuthenticationStateProvider
              .GetAuthenticationStateAsync();
          var user = authState.User;

          if (user.IsInRole("admin") || user.IsInRole("developer"))
          {
              ...
          }
          else
          {
              ...
          }
      }
  }
  ```

* <span data-ttu-id="66db2-207">È necessario che l'utente sia in **entrambi** i `admin` ruoli **e** `developer` con il codice procedurale modificando l' [OR condizionale ( `||` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) in un [and condizionale ( `&&` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="66db2-207">Require that the user be in **both** the `admin` **and** `developer` roles with procedural code by changing the [conditional OR (`||`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) to a [conditional AND (`&&`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) in the preceding example:</span></span>

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  ```

<span data-ttu-id="66db2-208">Qualsiasi meccanismo di autorizzazione nei controller dell'app **Server** può usare il `admin` ruolo per autorizzare l'utente:</span><span class="sxs-lookup"><span data-stu-id="66db2-208">Any of the authorization mechanisms in controllers of the **SERVER** app can use the `admin` role to authorize the user:</span></span>

* <span data-ttu-id="66db2-209">[ `[Authorize]` attribute](xref:blazor/security/index#authorize-attribute) (direttiva <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ) ()</span><span class="sxs-lookup"><span data-stu-id="66db2-209">[`[Authorize]` attribute directive](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)</span></span>

  ```csharp
  [Authorize(Roles = "admin")]
  ```

* [<span data-ttu-id="66db2-210">Logica procedurale</span><span class="sxs-lookup"><span data-stu-id="66db2-210">Procedural logic</span></span>](xref:blazor/security/index#procedural-logic)

  ```csharp
  if (User.IsInRole("admin")) { ... }
  ```

<span data-ttu-id="66db2-211">Sono supportati più test di ruolo:</span><span class="sxs-lookup"><span data-stu-id="66db2-211">Multiple role tests are supported:</span></span>

* <span data-ttu-id="66db2-212">Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con l' `[Authorize]` attributo:</span><span class="sxs-lookup"><span data-stu-id="66db2-212">Require that the user be in **either** the `admin` **or** `developer` role with the `[Authorize]` attribute:</span></span>

  ```csharp
  [Authorize(Roles = "admin, developer")]
  ```

* <span data-ttu-id="66db2-213">Richiedere **che l'utente sia nei** `admin` ruoli **e** `developer` con l' `[Authorize]` attributo:</span><span class="sxs-lookup"><span data-stu-id="66db2-213">Require that the user be in **both** the `admin` **and** `developer` roles with the `[Authorize]` attribute:</span></span>

  ```csharp
  [Authorize(Roles = "admin")]
  [Authorize(Roles = "developer")]
  ```

* <span data-ttu-id="66db2-214">Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con codice procedurale:</span><span class="sxs-lookup"><span data-stu-id="66db2-214">Require that the user be in **either** the `admin` **or** `developer` role with procedural code:</span></span>

  ```csharp
  static readonly string[] scopeRequiredByApi = new string[] { "API.Access" };

  ...

  [HttpGet]
  public IEnumerable<ReturnType> Get()
  {
      HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);

      if (User.IsInRole("admin") || User.IsInRole("developer"))
      {
          ...
      }
      else
      {
          ...
      }

      return ...
  }
  ```

* <span data-ttu-id="66db2-215">È necessario che l'utente sia in **entrambi** i `admin` ruoli **e** `developer` con il codice procedurale modificando l' [OR condizionale ( `||` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) in un [and condizionale ( `&&` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="66db2-215">Require that the user be in **both** the `admin` **and** `developer` roles with procedural code by changing the [conditional OR (`||`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) to a [conditional AND (`&&`)](/dotnet/csharp/language-reference/operators/boolean-logical-operators) in the preceding example:</span></span>

  ```csharp
  if (User.IsInRole("admin") && User.IsInRole("developer"))
  ```

## <a name="additional-resources"></a><span data-ttu-id="66db2-216">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="66db2-216">Additional resources</span></span>

* [<span data-ttu-id="66db2-217">ID del modello di ruolo (documentazione di Azure)</span><span class="sxs-lookup"><span data-stu-id="66db2-217">Role template IDs (Azure documentation)</span></span>](/azure/active-directory/roles/permissions-reference#role-template-ids)
* [<span data-ttu-id="66db2-218">`groupMembershipClaims` attribute (documentazione di Azure)</span><span class="sxs-lookup"><span data-stu-id="66db2-218">`groupMembershipClaims` attribute (Azure documentation)</span></span>](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute)
* [<span data-ttu-id="66db2-219">Procedura: aggiungere i ruoli dell'app nell'applicazione e riceverli nel token (documentazione di Azure)</span><span class="sxs-lookup"><span data-stu-id="66db2-219">How to: Add app roles in your application and receive them in the token (Azure documentation)</span></span>](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)
* [<span data-ttu-id="66db2-220">Ruoli applicazione (documentazione di Azure)</span><span class="sxs-lookup"><span data-stu-id="66db2-220">Application roles (Azure documentation)</span></span>](/azure/architecture/multitenant-identity/app-roles)
* <xref:security/authorization/claims>
* <xref:security/authorization/roles>
* <xref:blazor/security/index>
