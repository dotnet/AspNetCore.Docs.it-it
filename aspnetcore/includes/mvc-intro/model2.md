---
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
ms.openlocfilehash: 44cef0c1491cc609dd9b821910014ec88ecaad81
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551458"
---
::: moniker range=">= aspnetcore-3.0"

<a name="dc"></a>

<span data-ttu-id="357dd-101">Creare una cartella *Data*.</span><span class="sxs-lookup"><span data-stu-id="357dd-101">Create a *Data* folder.</span></span>

<span data-ttu-id="357dd-102">Aggiungere la classe `MvcMovieContext` seguente alla cartella *Data*:</span><span class="sxs-lookup"><span data-stu-id="357dd-102">Add the following `MvcMovieContext` class to the *Data* folder:</span></span>  

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/zDocOnly/MvcMovieContext.cs?name=snippet)]

<span data-ttu-id="357dd-103">Il codice precedente crea una proprietà `DbSet` per il set di entità.</span><span class="sxs-lookup"><span data-stu-id="357dd-103">The preceding code creates a `DbSet` property for the entity set.</span></span> <span data-ttu-id="357dd-104">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella.</span><span class="sxs-lookup"><span data-stu-id="357dd-104">In Entity Framework terminology, an entity set typically corresponds to a database table, and an entity corresponds to a row in the table.</span></span>

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a><span data-ttu-id="357dd-105">Aggiungere una stringa di connessione del database</span><span class="sxs-lookup"><span data-stu-id="357dd-105">Add a database connection string</span></span>

<span data-ttu-id="357dd-106">Aggiungere una stringa di connessione al *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="357dd-106">Add a connection string to the *appsettings.json* file:</span></span>

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings_SQLite.json?highlight=10-12)]

### <a name="add-nuget-packages-and-ef-tools"></a><span data-ttu-id="357dd-107">Aggiungere i pacchetti NuGet e gli strumenti EF</span><span class="sxs-lookup"><span data-stu-id="357dd-107">Add NuGet packages and EF tools</span></span>

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

<a name="reg"></a>

### <a name="register-the-database-context"></a><span data-ttu-id="357dd-108">Registrare il contesto del database</span><span class="sxs-lookup"><span data-stu-id="357dd-108">Register the database context</span></span>

<span data-ttu-id="357dd-109">Aggiungere le istruzioni `using` seguenti all'inizio di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="357dd-109">Add the following `using` statements at the top of *Startup.cs*:</span></span>

```csharp
using MvcMovie.Data;
using Microsoft.EntityFrameworkCore;
```

<span data-ttu-id="357dd-110">Registrare il contesto del database con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="357dd-110">Register the database context with the [dependency injection](xref:fundamentals/dependency-injection) container in `Startup.ConfigureServices`.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_UseSqlite&highlight=6-7)]

<span data-ttu-id="357dd-111">Compilare il progetto per controllare se sono presenti errori del compilatore.</span><span class="sxs-lookup"><span data-stu-id="357dd-111">Build the project as a check for compiler errors.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="357dd-112">Aggiungere la classe `MvcMovieContext` seguente alla cartella *Models*:</span><span class="sxs-lookup"><span data-stu-id="357dd-112">Add the following `MvcMovieContext` class to the *Models* folder:</span></span>  

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Data/MvcMovieContext.cs)]

<span data-ttu-id="357dd-113">Il codice precedente crea una proprietà `DbSet` per il set di entità.</span><span class="sxs-lookup"><span data-stu-id="357dd-113">The preceding code creates a `DbSet` property for the entity set.</span></span> <span data-ttu-id="357dd-114">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella.</span><span class="sxs-lookup"><span data-stu-id="357dd-114">In Entity Framework terminology, an entity set typically corresponds to a database table, and an entity corresponds to a row in the table.</span></span>

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a><span data-ttu-id="357dd-115">Aggiungere una stringa di connessione del database</span><span class="sxs-lookup"><span data-stu-id="357dd-115">Add a database connection string</span></span>

<span data-ttu-id="357dd-116">Aggiungere una stringa di connessione al *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="357dd-116">Add a connection string to the *appsettings.json* file:</span></span>

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/appsettings_SQLite.json?highlight=8-10)]

### <a name="add-required-nuget-packages"></a><span data-ttu-id="357dd-117">Aggiungere i pacchetti NuGet necessari</span><span class="sxs-lookup"><span data-stu-id="357dd-117">Add required NuGet packages</span></span>

<span data-ttu-id="357dd-118">Eseguire il comando seguente dell'interfaccia della riga di comando di .NET Core CLI per aggiungere SQLite e CodeGeneration.Design al progetto:</span><span class="sxs-lookup"><span data-stu-id="357dd-118">Run the following .NET Core CLI command to add SQLite and CodeGeneration.Design  to the project:</span></span>

```dotnetcli
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
```

<span data-ttu-id="357dd-119">Il pacchetto `Microsoft.VisualStudio.Web.CodeGeneration.Design` è necessario per lo scaffolding.</span><span class="sxs-lookup"><span data-stu-id="357dd-119">The `Microsoft.VisualStudio.Web.CodeGeneration.Design` package is required for scaffolding.</span></span>

<a name="reg"></a>

### <a name="register-the-database-context"></a><span data-ttu-id="357dd-120">Registrare il contesto del database</span><span class="sxs-lookup"><span data-stu-id="357dd-120">Register the database context</span></span>

<span data-ttu-id="357dd-121">Aggiungere le istruzioni `using` seguenti all'inizio di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="357dd-121">Add the following `using` statements at the top of *Startup.cs*:</span></span>

```csharp
using MvcMovie.Models;
using Microsoft.EntityFrameworkCore;
```

<span data-ttu-id="357dd-122">Registrare il contesto del database con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="357dd-122">Register the database context with the [dependency injection](xref:fundamentals/dependency-injection) container in `Startup.ConfigureServices`.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

<span data-ttu-id="357dd-123">Compilare il progetto per il controllo di eventuali errori.</span><span class="sxs-lookup"><span data-stu-id="357dd-123">Build the project as a check for errors.</span></span>
::: moniker-end