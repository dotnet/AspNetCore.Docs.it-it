---
title: Gestire utenti e gruppi in SignalR
author: bradygaster
description: Panoramica di ASP.NET Core SignalR gestione di utenti e gruppi.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 05/17/2020
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
uid: signalr/groups
ms.openlocfilehash: 8f9e91f34386bb5b4cf79bee429a39ae27c03c5e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589218"
---
# <a name="manage-users-and-groups-in-signalr"></a><span data-ttu-id="df5ef-103">Gestire utenti e gruppi in SignalR</span><span class="sxs-lookup"><span data-stu-id="df5ef-103">Manage users and groups in SignalR</span></span>

<span data-ttu-id="df5ef-104">Di [Brennan Conroy](https://github.com/BrennanConroy)</span><span class="sxs-lookup"><span data-stu-id="df5ef-104">By [Brennan Conroy](https://github.com/BrennanConroy)</span></span>

<span data-ttu-id="df5ef-105">SignalR consente l'invio di messaggi a tutte le connessioni associate a un utente specifico, oltre che ai gruppi denominati di connessioni.</span><span class="sxs-lookup"><span data-stu-id="df5ef-105">SignalR allows messages to be sent to all connections associated with a specific user, as well as to named groups of connections.</span></span>

<span data-ttu-id="df5ef-106">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/groups/sample/) [(procedura per il download)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="df5ef-106">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/groups/sample/) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

## <a name="users-in-signalr"></a><span data-ttu-id="df5ef-107">Utenti in SignalR</span><span class="sxs-lookup"><span data-stu-id="df5ef-107">Users in SignalR</span></span>

<span data-ttu-id="df5ef-108">Un singolo utente in SignalR può avere più connessioni a un'app.</span><span class="sxs-lookup"><span data-stu-id="df5ef-108">A single user in SignalR can have multiple connections to an app.</span></span> <span data-ttu-id="df5ef-109">Ad esempio, un utente potrebbe essere connesso sul desktop e sul telefono.</span><span class="sxs-lookup"><span data-stu-id="df5ef-109">For example, a user could be connected on their desktop as well as their phone.</span></span> <span data-ttu-id="df5ef-110">Ogni dispositivo ha una SignalR connessione separata, ma tutti sono associati allo stesso utente.</span><span class="sxs-lookup"><span data-stu-id="df5ef-110">Each device has a separate SignalR connection, but they're all associated with the same user.</span></span> <span data-ttu-id="df5ef-111">Se un messaggio viene inviato all'utente, tutte le connessioni associate a tale utente riceveranno il messaggio.</span><span class="sxs-lookup"><span data-stu-id="df5ef-111">If a message is sent to the user, all of the connections associated with that user receive the message.</span></span> <span data-ttu-id="df5ef-112">È possibile accedere all'identificatore utente per una connessione tramite la `Context.UserIdentifier` proprietà nell'hub.</span><span class="sxs-lookup"><span data-stu-id="df5ef-112">The user identifier for a connection can be accessed by the `Context.UserIdentifier` property in the hub.</span></span>

<span data-ttu-id="df5ef-113">Per impostazione predefinita, SignalR utilizza l'oggetto `ClaimTypes.NameIdentifier` dall'oggetto associato alla `ClaimsPrincipal` connessione come identificatore utente.</span><span class="sxs-lookup"><span data-stu-id="df5ef-113">By default, SignalR uses the `ClaimTypes.NameIdentifier` from the `ClaimsPrincipal` associated with the connection as the user identifier.</span></span> <span data-ttu-id="df5ef-114">Per personalizzare questo comportamento, vedere [usare le attestazioni per personalizzare la gestione delle identità](xref:signalr/authn-and-authz#use-claims-to-customize-identity-handling).</span><span class="sxs-lookup"><span data-stu-id="df5ef-114">To customize this behavior, see [Use claims to customize identity handling](xref:signalr/authn-and-authz#use-claims-to-customize-identity-handling).</span></span>

<span data-ttu-id="df5ef-115">Inviare un messaggio a un utente specifico passando l'identificatore utente alla `User` funzione in un metodo Hub, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="df5ef-115">Send a message to a specific user by passing the user identifier to the `User` function in a hub method, as shown in the following example:</span></span>

> [!NOTE]
> <span data-ttu-id="df5ef-116">L'identificatore utente distingue tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="df5ef-116">The user identifier is case-sensitive.</span></span>

[!code-csharp[Configure service](groups/sample/Hubs/ChatHub.cs?range=29-32)]

## <a name="groups-in-signalr"></a><span data-ttu-id="df5ef-117">Gruppi in SignalR</span><span class="sxs-lookup"><span data-stu-id="df5ef-117">Groups in SignalR</span></span>

<span data-ttu-id="df5ef-118">Un gruppo è una raccolta di connessioni associate a un nome.</span><span class="sxs-lookup"><span data-stu-id="df5ef-118">A group is a collection of connections associated with a name.</span></span> <span data-ttu-id="df5ef-119">I messaggi possono essere inviati a tutte le connessioni in un gruppo.</span><span class="sxs-lookup"><span data-stu-id="df5ef-119">Messages can be sent to all connections in a group.</span></span> <span data-ttu-id="df5ef-120">I gruppi sono il metodo consigliato per l'invio a una connessione o a più connessioni perché i gruppi sono gestiti dall'applicazione.</span><span class="sxs-lookup"><span data-stu-id="df5ef-120">Groups are the recommended way to send to a connection or multiple connections because the groups are managed by the application.</span></span> <span data-ttu-id="df5ef-121">Una connessione può essere un membro di più gruppi.</span><span class="sxs-lookup"><span data-stu-id="df5ef-121">A connection can be a member of multiple groups.</span></span> <span data-ttu-id="df5ef-122">I gruppi sono ideali per qualcosa di simile a un'applicazione di chat, in cui ogni stanza può essere rappresentata come un gruppo.</span><span class="sxs-lookup"><span data-stu-id="df5ef-122">Groups are ideal for something like a chat application, where each room can be represented as a group.</span></span> <span data-ttu-id="df5ef-123">Le connessioni vengono aggiunte o rimosse dai gruppi tramite `AddToGroupAsync` i `RemoveFromGroupAsync` metodi e.</span><span class="sxs-lookup"><span data-stu-id="df5ef-123">Connections are added to or removed from groups via the `AddToGroupAsync` and `RemoveFromGroupAsync` methods.</span></span>

[!code-csharp[Hub methods](groups/sample/Hubs/ChatHub.cs?range=15-27)]

<span data-ttu-id="df5ef-124">L'appartenenza al gruppo non viene mantenuta quando si riconnette una connessione.</span><span class="sxs-lookup"><span data-stu-id="df5ef-124">Group membership isn't preserved when a connection reconnects.</span></span> <span data-ttu-id="df5ef-125">La connessione deve essere nuovamente aggiunta al gruppo quando viene ristabilita.</span><span class="sxs-lookup"><span data-stu-id="df5ef-125">The connection needs to rejoin the group when it's re-established.</span></span> <span data-ttu-id="df5ef-126">Non è possibile contare i membri di un gruppo, poiché queste informazioni non sono disponibili se l'applicazione viene ridimensionata in più server.</span><span class="sxs-lookup"><span data-stu-id="df5ef-126">It's not possible to count the members of a group, since this information is not available if the application is scaled to multiple servers.</span></span>

<span data-ttu-id="df5ef-127">Per proteggere l'accesso alle risorse durante l'uso di gruppi, usare la funzionalità di [autenticazione e autorizzazione](xref:signalr/authn-and-authz) in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="df5ef-127">To protect access to resources while using groups, use [authentication and authorization](xref:signalr/authn-and-authz) functionality in ASP.NET Core.</span></span> <span data-ttu-id="df5ef-128">Se un utente viene aggiunto a un gruppo solo quando le credenziali sono valide per il gruppo, i messaggi inviati a tale gruppo passeranno solo agli utenti autorizzati.</span><span class="sxs-lookup"><span data-stu-id="df5ef-128">If a user is added to a group only when the credentials are valid for that group, messages sent to that group will only go to authorized users.</span></span> <span data-ttu-id="df5ef-129">Tuttavia, i gruppi non sono una funzionalità di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="df5ef-129">However, groups are not a security feature.</span></span> <span data-ttu-id="df5ef-130">Le attestazioni di autenticazione hanno funzionalità che non vengono eseguite dai gruppi, ad esempio la scadenza e la revoca.</span><span class="sxs-lookup"><span data-stu-id="df5ef-130">Authentication claims have features that groups do not, such as expiry and revocation.</span></span> <span data-ttu-id="df5ef-131">Se l'autorizzazione dell'utente per accedere al gruppo viene revocata, l'app deve rimuovere l'utente dal gruppo in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="df5ef-131">If a user's permission to access the group is revoked, the app must remove the user from the group explicitly.</span></span>

> [!NOTE]
> <span data-ttu-id="df5ef-132">I nomi dei gruppi fanno distinzione tra maiuscole e minuscole</span><span class="sxs-lookup"><span data-stu-id="df5ef-132">Group names are case-sensitive.</span></span>

## <a name="related-resources"></a><span data-ttu-id="df5ef-133">Risorse correlate</span><span class="sxs-lookup"><span data-stu-id="df5ef-133">Related resources</span></span>

* [<span data-ttu-id="df5ef-134">Operazioni preliminari</span><span class="sxs-lookup"><span data-stu-id="df5ef-134">Get started</span></span>](xref:tutorials/signalr)
* [<span data-ttu-id="df5ef-135">Hub</span><span class="sxs-lookup"><span data-stu-id="df5ef-135">Hubs</span></span>](xref:signalr/hubs)
* [<span data-ttu-id="df5ef-136">Pubblicare in Azure</span><span class="sxs-lookup"><span data-stu-id="df5ef-136">Publish to Azure</span></span>](xref:signalr/publish-to-azure-web-app)
