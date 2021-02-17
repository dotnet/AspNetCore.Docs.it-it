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
> Per questa esercitazione si userà la funzionalità *migrazioni* di Entity Framework Core laddove possibile. Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati. Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate. Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata. Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo. A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite. In caso di modifiche dello schema, il database viene invece eliminato e ricreato.
>
>La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella. La ricompilazione della tabella comporta:
>
>* Creazione di una nuova tabella.
>* Copia dei dati dalla vecchia tabella alla nuova tabella.
>* Eliminazione della tabella precedente.
>* Ridenominazione della nuova tabella.
>
>Per altre informazioni, vedere le risorse seguenti:
>
> * [Limitazioni del provider di database SQLite per EF Core](/ef/core/providers/sqlite/limitations)
> * [Personalizzare il codice di migrazione](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [Seeding dei dati](/ef/core/modeling/data-seeding)
  * [Istruzione ALTER TABLE di SQLite](https://sqlite.org/lang_altertable.html)