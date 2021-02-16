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
ms.openlocfilehash: 6435d39b6d442f0d4643d77d9111d11b50781544
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100536296"
---
Il `:` separatore non funziona con le chiavi gerarchiche delle variabili di ambiente in tutte le piattaforme. `__`, il doppio carattere di sottolineatura, è:

* Supportato da tutte le piattaforme. Il separatore, ad esempio, `:` non è supportato da [bash](https://linuxhint.com/bash-environment-variables/), ma `__` è.
* Sostituito automaticamente da `:`