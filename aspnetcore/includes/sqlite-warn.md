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
ms.openlocfilehash: cf20722e8c8669fb17af8db032d4064ca2be2f4c
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552457"
---
> [!NOTE]
> 
> <span data-ttu-id="12519-101">**Limitazioni di SQLite**</span><span class="sxs-lookup"><span data-stu-id="12519-101">**SQLite limitations**</span></span>
>
> <span data-ttu-id="12519-102">Questa esercitazione usa la funzionalità *migrazioni* di Entity Framework Core laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="12519-102">This tutorial uses the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="12519-103">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="12519-103">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="12519-104">Tuttavia, le migrazioni eseguono solo i tipi di modifiche supportate dal motore di database e le funzionalità di modifica dello schema di SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="12519-104">However, migrations only does the kinds of changes that the database engine supports, and SQLite's schema change capabilities are limited.</span></span> <span data-ttu-id="12519-105">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="12519-105">For example, adding a column is supported, but removing a column is not supported.</span></span> <span data-ttu-id="12519-106">Se si crea una migrazione per rimuovere una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="12519-106">If a migration is created to remove a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> 
>
> <span data-ttu-id="12519-107">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="12519-107">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="12519-108">Il codice verrebbe inserito nei metodi `Up` e `Down` per una migrazione e comporterebbe le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="12519-108">The code would go in the `Up` and `Down` methods for a migration and would involve:</span></span>
>
> * <span data-ttu-id="12519-109">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="12519-109">Creating a new table.</span></span>
> * <span data-ttu-id="12519-110">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="12519-110">Copying data from the old table to the new table.</span></span>
> * <span data-ttu-id="12519-111">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="12519-111">Dropping the old table.</span></span>
> * <span data-ttu-id="12519-112">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="12519-112">Renaming the new table.</span></span>
>
> <span data-ttu-id="12519-113">La scrittura di codice specifico del database di questo tipo esula dall'ambito di questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="12519-113">Writing database-specific code of this type is outside the scope of this tutorial.</span></span> <span data-ttu-id="12519-114">Al contrario, in questa esercitazione viene eliminato e ricreato il database ogni volta che un tentativo di applicare una migrazione ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="12519-114">Instead, this tutorial drops and re-creates the database whenever an attempt to apply a migration would fail.</span></span> <span data-ttu-id="12519-115">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="12519-115">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="12519-116">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="12519-116">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="12519-117">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="12519-117">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="12519-118">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="12519-118">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="12519-119">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="12519-119">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)