---
title: Password hash in ASP.NET Core
author: rick-anderson
description: Informazioni su come eseguire l'hashing delle password usando le API di protezione dei dati ASP.NET Core.
ms.author: riande
ms.date: 10/14/2016
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
uid: security/data-protection/consumer-apis/password-hashing
ms.openlocfilehash: b88abd076febae95f8990bcd730ce0d4517e92d2
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587762"
---
# <a name="hash-passwords-in-aspnet-core"></a><span data-ttu-id="f0ddb-103">Password hash in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="f0ddb-103">Hash passwords in ASP.NET Core</span></span>

<span data-ttu-id="f0ddb-104">La base di codice di protezione dei dati include un pacchetto *Microsoft. AspNetCore. Cryptography. Key Derivation* che contiene funzioni di derivazione della chiave crittografica.</span><span class="sxs-lookup"><span data-stu-id="f0ddb-104">The data protection code base includes a package *Microsoft.AspNetCore.Cryptography.KeyDerivation* which contains cryptographic key derivation functions.</span></span> <span data-ttu-id="f0ddb-105">Questo pacchetto è un componente autonomo e non ha dipendenze dal resto del sistema di protezione dei dati.</span><span class="sxs-lookup"><span data-stu-id="f0ddb-105">This package is a standalone component and has no dependencies on the rest of the data protection system.</span></span> <span data-ttu-id="f0ddb-106">Può essere usato completamente in modo indipendente.</span><span class="sxs-lookup"><span data-stu-id="f0ddb-106">It can be used completely independently.</span></span> <span data-ttu-id="f0ddb-107">L'origine esiste insieme alla base del codice di protezione dei dati per praticità.</span><span class="sxs-lookup"><span data-stu-id="f0ddb-107">The source exists alongside the data protection code base as a convenience.</span></span>

<span data-ttu-id="f0ddb-108">Il pacchetto offre attualmente un metodo `KeyDerivation.Pbkdf2` che consente l'hashing di una password usando l' [algoritmo PBKDF2](https://tools.ietf.org/html/rfc2898#section-5.2).</span><span class="sxs-lookup"><span data-stu-id="f0ddb-108">The package currently offers a method `KeyDerivation.Pbkdf2` which allows hashing a password using the [PBKDF2 algorithm](https://tools.ietf.org/html/rfc2898#section-5.2).</span></span> <span data-ttu-id="f0ddb-109">Questa API è molto simile al [tipo Rfc2898DeriveBytes](/dotnet/api/system.security.cryptography.rfc2898derivebytes)esistente del .NET Framework, ma sono disponibili tre distinzioni importanti:</span><span class="sxs-lookup"><span data-stu-id="f0ddb-109">This API is very similar to the .NET Framework's existing [Rfc2898DeriveBytes type](/dotnet/api/system.security.cryptography.rfc2898derivebytes), but there are three important distinctions:</span></span>

1. <span data-ttu-id="f0ddb-110">Il `KeyDerivation.Pbkdf2` metodo supporta l'utilizzo di più PRFs (attualmente `HMACSHA1` , `HMACSHA256` e `HMACSHA512` ), mentre il `Rfc2898DeriveBytes` tipo supporta solo `HMACSHA1` .</span><span class="sxs-lookup"><span data-stu-id="f0ddb-110">The `KeyDerivation.Pbkdf2` method supports consuming multiple PRFs (currently `HMACSHA1`, `HMACSHA256`, and `HMACSHA512`), whereas the `Rfc2898DeriveBytes` type only supports `HMACSHA1`.</span></span>

2. <span data-ttu-id="f0ddb-111">Il `KeyDerivation.Pbkdf2` metodo rileva il sistema operativo corrente e tenta di scegliere l'implementazione più ottimizzata della routine, garantendo in alcuni casi prestazioni molto migliori.</span><span class="sxs-lookup"><span data-stu-id="f0ddb-111">The `KeyDerivation.Pbkdf2` method detects the current operating system and attempts to choose the most optimized implementation of the routine, providing much better performance in certain cases.</span></span> <span data-ttu-id="f0ddb-112">In Windows 8 offre circa 10 volte la velocità effettiva di `Rfc2898DeriveBytes` .</span><span class="sxs-lookup"><span data-stu-id="f0ddb-112">(On Windows 8, it offers around 10x the throughput of `Rfc2898DeriveBytes`.)</span></span>

3. <span data-ttu-id="f0ddb-113">Il `KeyDerivation.Pbkdf2` metodo richiede che il chiamante specifichi tutti i parametri (Salt, PRF e count di iterazione).</span><span class="sxs-lookup"><span data-stu-id="f0ddb-113">The `KeyDerivation.Pbkdf2` method requires the caller to specify all parameters (salt, PRF, and iteration count).</span></span> <span data-ttu-id="f0ddb-114">Il `Rfc2898DeriveBytes` tipo fornisce i valori predefiniti per questi.</span><span class="sxs-lookup"><span data-stu-id="f0ddb-114">The `Rfc2898DeriveBytes` type provides default values for these.</span></span>

[!code-csharp[](password-hashing/samples/passwordhasher.cs)]

<span data-ttu-id="f0ddb-115">Per un caso d'uso reale, vedere il [codice sorgente](https://github.com/dotnet/AspNetCore/blob/main/src/Identity/Extensions.Core/src/PasswordHasher.cs) per il ASP.NET Core Identity `PasswordHasher` tipo di.</span><span class="sxs-lookup"><span data-stu-id="f0ddb-115">See the [source code](https://github.com/dotnet/AspNetCore/blob/main/src/Identity/Extensions.Core/src/PasswordHasher.cs) for ASP.NET Core Identity's `PasswordHasher` type for a real-world use case.</span></span>
