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
ms.openlocfilehash: e8c5bd00178aefb91ab0e7066c5490ceba315530
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551405"
---
> [!NOTE]
> <span data-ttu-id="9843a-101">Per questa esercitazione si userà la funzionalità *migrazioni* di Entity Framework Core laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="9843a-101">For this tutorial you use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="9843a-102">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="9843a-102">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="9843a-103">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="9843a-103">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="9843a-104">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="9843a-104">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="9843a-105">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="9843a-105">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="9843a-106">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="9843a-106">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="9843a-107">In caso di modifiche dello schema, il database viene invece eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="9843a-107">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="9843a-108">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="9843a-108">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="9843a-109">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="9843a-109">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="9843a-110">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9843a-110">Creating a new table.</span></span>
>* <span data-ttu-id="9843a-111">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9843a-111">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="9843a-112">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="9843a-112">Dropping the old table.</span></span>
>* <span data-ttu-id="9843a-113">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9843a-113">Renaming the new table.</span></span>
>
><span data-ttu-id="9843a-114">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="9843a-114">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="9843a-115">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="9843a-115">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="9843a-116">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="9843a-116">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="9843a-117">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="9843a-117">Data seeding</span></span>](/ef/core/modeling/data-seeding)
  * [<span data-ttu-id="9843a-118">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="9843a-118">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)