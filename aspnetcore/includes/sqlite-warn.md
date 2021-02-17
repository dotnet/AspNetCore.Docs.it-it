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
> **Limitazioni di SQLite**
>
> Questa esercitazione usa la funzionalità *migrazioni* di Entity Framework Core laddove possibile. Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati. Tuttavia, le migrazioni eseguono solo i tipi di modifiche supportate dal motore di database e le funzionalità di modifica dello schema di SQLite sono limitate. Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione di una colonna non è supportata. Se si crea una migrazione per rimuovere una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo. 
>
> La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella. Il codice verrebbe inserito nei metodi `Up` e `Down` per una migrazione e comporterebbe le operazioni seguenti:
>
> * Creazione di una nuova tabella.
> * Copia dei dati dalla vecchia tabella alla nuova tabella.
> * Eliminazione della tabella precedente.
> * Ridenominazione della nuova tabella.
>
> La scrittura di codice specifico del database di questo tipo esula dall'ambito di questa esercitazione. Al contrario, in questa esercitazione viene eliminato e ricreato il database ogni volta che un tentativo di applicare una migrazione ha esito negativo. Per altre informazioni, vedere le risorse seguenti:
>
> * [Limitazioni del provider di database SQLite per EF Core](/ef/core/providers/sqlite/limitations)
> * [Personalizzare il codice di migrazione](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [Seeding dei dati](/ef/core/modeling/data-seeding)
> * [Istruzione ALTER TABLE di SQLite](https://sqlite.org/lang_altertable.html)