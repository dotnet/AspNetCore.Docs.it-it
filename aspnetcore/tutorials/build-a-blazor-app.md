---
title: Creare un' Blazor app elenco attività
author: guardrex
description: Creare un' Blazor app dettagliata.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 02/12/2021
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
uid: tutorials/build-a-blazor-app
ms.openlocfilehash: d984023a1c46c5383d47a1634c54e61747b83d60
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101210"
---
# <a name="build-a-blazor-todo-list-app"></a>Creare un' Blazor app elenco attività

Questa esercitazione illustra come creare e modificare un' Blazor app. Si apprenderà come:

> [!div class="checklist"]
> * Creare un progetto di Blazor app elenco TODO
> * Modificare i Razor componenti
> * Usare la gestione degli eventi e data binding nei componenti
> * Usare il routing in un' Blazor app

Al termine di questa esercitazione, si disporrà di un'app elenco attività.

## <a name="prerequisites"></a>Prerequisiti

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE[](~/includes/5.0-SDK.md)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!INCLUDE[](~/includes/3.1-SDK.md)]

::: moniker-end

## <a name="create-a-todo-list-blazor-app"></a>Creare un'app elenco attività Blazor

1. Creare una nuova Blazor App denominata `TodoList` in una shell comandi:

   ```dotnetcli
   dotnet new blazorserver -o TodoList
   ```

   Il comando precedente crea una cartella denominata `TodoList` con l' `-o|--output` opzione che consente di mantenere l'app. La `TodoList` cartella è la *cartella radice* del progetto. Passare alla `TodoList` cartella con il comando seguente:

   ```dotnetcli
   cd TodoList
   ```

1. Aggiungere un nuovo `Todo` Razor componente all'app usando il comando seguente:

   ```dotnetcli
   dotnet new razorcomponent -n Todo -o Pages
   ```

   L' `-n|--name` opzione nel comando precedente specifica il nome del nuovo Razor componente. Il nuovo componente viene creato nella cartella del progetto `Pages` con l' `-o|--output` opzione.

   > [!IMPORTANT]
   > Razor i nomi dei file di componente richiedono una prima lettera maiuscola. Aprire la `Pages` cartella e verificare che il `Todo` nome del file del componente inizi con una lettera maiuscola `T` . Il nome del file deve essere `Todo.razor` .

1. Aprire il `Todo` componente in qualsiasi editor di file e aggiungere una `@page` Razor direttiva all'inizio del file con un URL relativo di `/todo` .

   `Pages/Todo.razor`:

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo0.razor?highlight=1)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo0.razor?highlight=1)]

   ::: moniker-end

   Salvare il file.`Pages/Todo.razor`

1. Aggiungere il componente `Todo` alla barra di spostamento.

   Il `NavMenu` componente viene usato nel layout dell'app. I layout sono componenti che consentono di evitare la duplicazione del contenuto in un'app. Il `NavLink` componente fornisce un segnale nell'interfaccia utente dell'app quando l'URL del componente viene caricato dall'app.

   Nell'elenco non ordinato ( `<ul>...</ul>` ) del `NavMenu` componente aggiungere i seguenti elementi elenco ( `<li>...</li>` ) e `NavLink` componente per il componente `Todo` .

   In `Shared/NavMenu.razor`:

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/build-a-blazor-app/NavMenu.razor?highlight=5-9)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/build-a-blazor-app/NavMenu.razor?highlight=5-9)]

   ::: moniker-end

   Salvare il file.`Shared/NavMenu.razor`

1. Compilare ed eseguire l'app eseguendo il [`dotnet watch run`](xref:tutorials/dotnet-watch) comando nella shell dei comandi dalla `TodoList` cartella. Quando l'app è in esecuzione, visita la nuova pagina todo selezionando il **`Todo`** collegamento nella barra di spostamento dell'app, che carica la pagina in `/todo` .

   Lasciare l'app in esecuzione nella shell dei comandi. Ogni volta che un file viene salvato, l'app viene ricompilata automaticamente. Il browser perde temporaneamente la connessione all'app durante la compilazione e il riavvio. La pagina nel browser viene ricaricata automaticamente quando viene ristabilita la connessione.

1. Aggiungere un `TodoItem.cs` file alla radice del progetto (la `TodoList` cartella) in cui archiviare una classe che rappresenta un elemento todo. Usare il codice C# seguente per la classe `TodoItem`.

   `TodoItem.cs`:

   ::: moniker range=">= aspnetcore-5.0"

   [!code-csharp[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/build-a-blazor-app/TodoItem.cs)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-csharp[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/build-a-blazor-app/TodoItem.cs)]

   ::: moniker-end

   > [!NOTE]
   > Se si usa Visual Studio per creare il `ToDoItem.cs` file e la `ToDoItem` classe, usare uno degli approcci seguenti:
   >
   > * Rimuovere lo spazio dei nomi generato da Visual Studio per la classe.
   > * Usare il pulsante **copia** nel blocco di codice precedente e sostituire l'intero contenuto del file generato da Visual Studio.

1. Tornare al `Todo` componente ed eseguire le attività seguenti:

   * Aggiungere un campo per gli elementi todo nel `@code` blocco. Il componente `Todo` usa questo campo per mantenere lo stato dell'elenco attività.
   * Aggiungere il markup per l'elenco non ordinato e un ciclo `foreach` per eseguire il rendering di ogni elemento Todo come elemento di elenco (`<li>`).

   `Pages/Todo.razor`:

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo2.razor?highlight=5-10,13)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo2.razor?highlight=5-10,13)]

   ::: moniker-end

1. L'app richiede elementi dell'interfaccia utente per l'aggiunta di voci todo all'elenco. Aggiungere un input di testo (`<input>`) e un pulsante (`<button>`) sotto l'elenco non ordinato (`<ul>...</ul>`):

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo3.razor?highlight=12-13)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo3.razor?highlight=12-13)]

   ::: moniker-end

1. Salvare il `TodoItem.cs` file e il `Pages/Todo.razor` file aggiornato. Nella shell dei comandi l'app viene ricompilata automaticamente quando i file vengono salvati. Il browser perde temporaneamente la connessione all'app e quindi ricarica la pagina quando viene ristabilita la connessione.

1. Quando il **`Add todo`** pulsante è selezionato, non accade nulla perché un gestore eventi non è associato al pulsante.

1. Aggiungere un `AddTodo` metodo al `Todo` componente e registrare il metodo per il pulsante usando l' `@onclick` attributo. Il metodo C# `AddTodo` viene chiamato quando viene selezionato il pulsante:

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo4.razor?highlight=2,7-10)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo4.razor?highlight=2,7-10)]

   ::: moniker-end

1. Per ottenere il titolo del nuovo elemento todo, aggiungere un `newTodo` campo stringa nella parte superiore del `@code` blocco:

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo5.razor?highlight=3)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo5.razor?highlight=3)]

   ::: moniker-end

   Modificare l' `<input>` elemento di testo da associare all' `newTodo` `@bind` attributo:

   ```razor
   <input placeholder="Something todo" @bind="newTodo" />
   ```

1. Aggiornare il metodo `AddTodo` per aggiungere l'elemento `TodoItem` con il titolo specificato all'elenco. Cancellare il valore dell'input di testo impostando `newTodo` su una stringa vuota:

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo6.razor?highlight=19-26)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo6.razor?highlight=19-26)]

   ::: moniker-end

1. Salvare il file.`Pages/Todo.razor` L'app viene ricompilata automaticamente nella shell dei comandi. La pagina viene ricaricata nel browser dopo la riconnessione del browser all'app.

1. Il testo del titolo per ogni elemento Todo può essere reso modificabile e una casella di controllo può consentire all'utente di tenere traccia degli elementi completati. Aggiungere un input casella di controllo per ogni elemento attività e associarne il valore alla proprietà `IsDone`. Passare `@todo.Title` a un `<input>` elemento associato a `todo.Title` con `@bind` :

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo7.razor?name=snippet&highlight=4-7)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo7.razor?name=snippet&highlight=4-7)]

   ::: moniker-end

1. Aggiornare l' `<h3>` intestazione per visualizzare un conteggio del numero di elementi todo non completi ( `IsDone` is `false` ).

   ```razor
   <h3>Todo (@todos.Count(todo => !todo.IsDone))</h3>
   ```

1. Componente completato `Todo` ( `Pages/Todo.razor` ):

   ::: moniker range=">= aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo1.razor)]

   ::: moniker-end

   ::: moniker range="< aspnetcore-5.0"

   [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/build-a-blazor-app/Todo1.razor)]

   ::: moniker-end

1. Salvare il file.`Pages/Todo.razor` L'app viene ricompilata automaticamente nella shell dei comandi. La pagina viene ricaricata nel browser dopo la riconnessione del browser all'app.

1. Aggiungere elementi, modificare elementi e contrassegnare gli elementi todo eseguiti per testare il componente.

1. Al termine, arrestare l'app nella shell dei comandi. Molte shell dei comandi accettano il comando della tastiera <kbd>CTRL</kbd> + <kbd>c</kbd> per arrestare un'app.

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione sono state illustrate le procedure per:

> [!div class="checklist"]
> * Creare un progetto di Blazor app elenco TODO
> * Modificare i Razor componenti
> * Usare la gestione degli eventi e data binding nei componenti
> * Usare il routing in un' Blazor app

Informazioni sugli strumenti per ASP.NET Core Blazor :

> [!div class="nextstepaction"]
> <xref:blazor/tooling>
