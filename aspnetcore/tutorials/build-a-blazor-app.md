---
title: Creare un' Blazor app elenco attività
author: guardrex
description: Creare un' Blazor app dettagliata.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/14/2020
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
ms.openlocfilehash: 106e1119db777074b5eae24f5d7e216e6127ca13
ms.sourcegitcommit: 75db2f684a9302b0be7925eab586aa091c6bd19f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/01/2021
ms.locfileid: "99238300"
---
# <a name="build-a-no-locblazor-todo-list-app"></a><span data-ttu-id="255ef-103">Creare un' Blazor app elenco attività</span><span class="sxs-lookup"><span data-stu-id="255ef-103">Build a Blazor todo list app</span></span>

<span data-ttu-id="255ef-104">Di [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="255ef-104">By [Daniel Roth](https://github.com/danroth27) and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="255ef-105">Questa esercitazione illustra come creare e modificare un' Blazor app.</span><span class="sxs-lookup"><span data-stu-id="255ef-105">This tutorial shows you how to build and modify a Blazor app.</span></span> <span data-ttu-id="255ef-106">Si apprenderà come:</span><span class="sxs-lookup"><span data-stu-id="255ef-106">You learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="255ef-107">Creare un progetto di Blazor app elenco TODO</span><span class="sxs-lookup"><span data-stu-id="255ef-107">Create a todo list Blazor app project</span></span>
> * <span data-ttu-id="255ef-108">Modificare i Razor componenti</span><span class="sxs-lookup"><span data-stu-id="255ef-108">Modify Razor components</span></span>
> * <span data-ttu-id="255ef-109">Usare la gestione degli eventi e data binding nei componenti</span><span class="sxs-lookup"><span data-stu-id="255ef-109">Use event handling and data binding in components</span></span>
> * <span data-ttu-id="255ef-110">Usare il routing in un' Blazor app</span><span class="sxs-lookup"><span data-stu-id="255ef-110">Use routing in a Blazor app</span></span>

<span data-ttu-id="255ef-111">Al termine di questa esercitazione, si disporrà di un'app elenco attività.</span><span class="sxs-lookup"><span data-stu-id="255ef-111">At the end of this tutorial, you'll have a working todo list app.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="255ef-112">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="255ef-112">Prerequisites</span></span>

::: moniker range=">= aspnetcore-5.0"

[!INCLUDE[](~/includes/5.0-SDK.md)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!INCLUDE[](~/includes/3.1-SDK.md)]

::: moniker-end

## <a name="create-a-todo-list-no-locblazor-app"></a><span data-ttu-id="255ef-113">Creare un'app elenco attività Blazor</span><span class="sxs-lookup"><span data-stu-id="255ef-113">Create a todo list Blazor app</span></span>

1. <span data-ttu-id="255ef-114">Creare una nuova Blazor App denominata `TodoList` in una shell comandi:</span><span class="sxs-lookup"><span data-stu-id="255ef-114">Create a new Blazor app named `TodoList` in a command shell:</span></span>

   ```dotnetcli
   dotnet new blazorserver -o TodoList
   ```

   <span data-ttu-id="255ef-115">Il comando precedente crea una cartella denominata `TodoList` con l' `-o|--output` opzione che consente di mantenere l'app.</span><span class="sxs-lookup"><span data-stu-id="255ef-115">The preceding command creates a folder named `TodoList` with the `-o|--output` option to hold the app.</span></span> <span data-ttu-id="255ef-116">La `TodoList` cartella è la *cartella radice* del progetto.</span><span class="sxs-lookup"><span data-stu-id="255ef-116">The `TodoList` folder is the *root folder* of the project.</span></span> <span data-ttu-id="255ef-117">Passare alla `TodoList` cartella con il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="255ef-117">Change directories to the `TodoList` folder with the following command:</span></span>

   ```dotnetcli
   cd TodoList
   ```

1. <span data-ttu-id="255ef-118">Aggiungere un nuovo `Todo` Razor componente all'app usando il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="255ef-118">Add a new `Todo` Razor component to the app using the following command:</span></span>

   ```dotnetcli
   dotnet new razorcomponent -n Todo -o Pages
   ```

   <span data-ttu-id="255ef-119">L' `-n|--name` opzione nel comando precedente specifica il nome del nuovo Razor componente.</span><span class="sxs-lookup"><span data-stu-id="255ef-119">The `-n|--name` option in the preceding command specifies the name of the new Razor component.</span></span> <span data-ttu-id="255ef-120">Il nuovo componente viene creato nella cartella del progetto `Pages` con l' `-o|--output` opzione.</span><span class="sxs-lookup"><span data-stu-id="255ef-120">The new component is created in the project's `Pages` folder with the `-o|--output` option.</span></span>

   > [!IMPORTANT]
   > <span data-ttu-id="255ef-121">Razor i nomi dei file di componente richiedono una prima lettera maiuscola.</span><span class="sxs-lookup"><span data-stu-id="255ef-121">Razor component file names require a capitalized first letter.</span></span> <span data-ttu-id="255ef-122">Aprire la `Pages` cartella e verificare che il `Todo` nome del file del componente inizi con una lettera maiuscola `T` .</span><span class="sxs-lookup"><span data-stu-id="255ef-122">Open the `Pages` folder and confirm that the `Todo` component file name starts with a capital letter `T`.</span></span> <span data-ttu-id="255ef-123">Il nome del file deve essere `Todo.razor` .</span><span class="sxs-lookup"><span data-stu-id="255ef-123">The file name should be `Todo.razor`.</span></span>

1. <span data-ttu-id="255ef-124">Aprire il `Todo` componente in qualsiasi editor di file e aggiungere una `@page` Razor direttiva all'inizio del file con un URL relativo di `/todo` .</span><span class="sxs-lookup"><span data-stu-id="255ef-124">Open the `Todo` component in any file editor and add an `@page` Razor directive to the top of the file with a relative URL of `/todo`.</span></span>

   <span data-ttu-id="255ef-125">`Pages/Todo.razor`:</span><span class="sxs-lookup"><span data-stu-id="255ef-125">`Pages/Todo.razor`:</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo0.razor?highlight=1)]

   <span data-ttu-id="255ef-126">Salvare il file.`Pages/Todo.razor`</span><span class="sxs-lookup"><span data-stu-id="255ef-126">Save the `Pages/Todo.razor` file.</span></span>

1. <span data-ttu-id="255ef-127">Aggiungere il componente `Todo` alla barra di spostamento.</span><span class="sxs-lookup"><span data-stu-id="255ef-127">Add the `Todo` component to the navigation bar.</span></span>

   <span data-ttu-id="255ef-128">Il `NavMenu` componente viene usato nel layout dell'app.</span><span class="sxs-lookup"><span data-stu-id="255ef-128">The `NavMenu` component is used in the app's layout.</span></span> <span data-ttu-id="255ef-129">I layout sono componenti che consentono di evitare la duplicazione del contenuto in un'app.</span><span class="sxs-lookup"><span data-stu-id="255ef-129">Layouts are components that allow you to avoid duplication of content in an app.</span></span> <span data-ttu-id="255ef-130">Il `NavLink` componente fornisce un segnale nell'interfaccia utente dell'app quando l'URL del componente viene caricato dall'app.</span><span class="sxs-lookup"><span data-stu-id="255ef-130">The `NavLink` component provides a cue in the app's UI when the component URL is loaded by the app.</span></span>

   <span data-ttu-id="255ef-131">Nell'elenco non ordinato ( `<ul>...</ul>` ) del `NavMenu` componente aggiungere i seguenti elementi elenco ( `<li>...</li>` ) e `NavLink` componente per il componente `Todo` .</span><span class="sxs-lookup"><span data-stu-id="255ef-131">In the unordered list (`<ul>...</ul>`) of the `NavMenu` component, add the following list item (`<li>...</li>`) and `NavLink` component for the `Todo` component.</span></span>

   <span data-ttu-id="255ef-132">In `Shared/NavMenu.razor`:</span><span class="sxs-lookup"><span data-stu-id="255ef-132">In `Shared/NavMenu.razor`:</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/NavMenu.razor?highlight=5-9)]

   <span data-ttu-id="255ef-133">Salvare il file.`Shared/NavMenu.razor`</span><span class="sxs-lookup"><span data-stu-id="255ef-133">Save the `Shared/NavMenu.razor` file.</span></span>

1. <span data-ttu-id="255ef-134">Compilare ed eseguire l'app eseguendo il [`dotnet watch run`](/aspnet/core/tutorials/dotnet-watch) comando nella shell dei comandi dalla `TodoList` cartella.</span><span class="sxs-lookup"><span data-stu-id="255ef-134">Build and run the app by executing the [`dotnet watch run`](/aspnet/core/tutorials/dotnet-watch) command in the command shell from the `TodoList` folder.</span></span> <span data-ttu-id="255ef-135">Quando l'app è in esecuzione, visita la nuova pagina todo selezionando il **`Todo`** collegamento nella barra di spostamento dell'app, che carica la pagina in `/todo` .</span><span class="sxs-lookup"><span data-stu-id="255ef-135">After the app is running, visit the new Todo page by selecting the **`Todo`** link in the app's navigation bar, which loads the page at `/todo`.</span></span>

   <span data-ttu-id="255ef-136">Lasciare l'app in esecuzione nella shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="255ef-136">Leave the app running the command shell.</span></span> <span data-ttu-id="255ef-137">Ogni volta che un file viene salvato, l'app viene ricompilata automaticamente.</span><span class="sxs-lookup"><span data-stu-id="255ef-137">Each time a file is saved, the app is automatically rebuilt.</span></span> <span data-ttu-id="255ef-138">Il browser perde temporaneamente la connessione all'app durante la compilazione e il riavvio.</span><span class="sxs-lookup"><span data-stu-id="255ef-138">The browser temporarily loses its connection to the app while compiling and restarting.</span></span> <span data-ttu-id="255ef-139">La pagina nel browser viene ricaricata automaticamente quando viene ristabilita la connessione.</span><span class="sxs-lookup"><span data-stu-id="255ef-139">The page in the browser is automatically reloaded when the connection is re-established.</span></span>

1. <span data-ttu-id="255ef-140">Aggiungere un `TodoItem.cs` file alla radice del progetto (la `TodoList` cartella) in cui archiviare una classe che rappresenta un elemento todo.</span><span class="sxs-lookup"><span data-stu-id="255ef-140">Add a `TodoItem.cs` file to the root of the project (the `TodoList` folder) to hold a class that represents a todo item.</span></span> <span data-ttu-id="255ef-141">Usare il codice C# seguente per la classe `TodoItem`.</span><span class="sxs-lookup"><span data-stu-id="255ef-141">Use the following C# code for the `TodoItem` class.</span></span>

   <span data-ttu-id="255ef-142">`TodoItem.cs`:</span><span class="sxs-lookup"><span data-stu-id="255ef-142">`TodoItem.cs`:</span></span>

   [!code-csharp[](build-a-blazor-app/samples_snapshot/TodoItem.cs)]
   
   > [!NOTE]
   > <span data-ttu-id="255ef-143">Se si usa Visual Studio per creare il `ToDoItem.cs` file e la `ToDoItem` classe, usare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="255ef-143">If using Visual Studio to create the `ToDoItem.cs` file and `ToDoItem` class, use either of the following approaches:</span></span>
   >
   > * <span data-ttu-id="255ef-144">Rimuovere lo spazio dei nomi generato da Visual Studio per la classe.</span><span class="sxs-lookup"><span data-stu-id="255ef-144">Remove the namespace that Visual Studio generates for the class.</span></span>
   > * <span data-ttu-id="255ef-145">Usare il pulsante **copia** nel blocco di codice precedente e sostituire l'intero contenuto del file generato da Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="255ef-145">Use the **Copy** button in the preceding code block and replace the entire contents of the file that Visual Studio generates.</span></span>

1. <span data-ttu-id="255ef-146">Tornare al `Todo` componente ed eseguire le attività seguenti:</span><span class="sxs-lookup"><span data-stu-id="255ef-146">Return to the `Todo` component and perform the following tasks:</span></span>

   * <span data-ttu-id="255ef-147">Aggiungere un campo per gli elementi todo nel `@code` blocco.</span><span class="sxs-lookup"><span data-stu-id="255ef-147">Add a field for the todo items in the `@code` block.</span></span> <span data-ttu-id="255ef-148">Il componente `Todo` usa questo campo per mantenere lo stato dell'elenco attività.</span><span class="sxs-lookup"><span data-stu-id="255ef-148">The `Todo` component uses this field to maintain the state of the todo list.</span></span>
   * <span data-ttu-id="255ef-149">Aggiungere il markup per l'elenco non ordinato e un ciclo `foreach` per eseguire il rendering di ogni elemento Todo come elemento di elenco (`<li>`).</span><span class="sxs-lookup"><span data-stu-id="255ef-149">Add unordered list markup and a `foreach` loop to render each todo item as a list item (`<li>`).</span></span>

   <span data-ttu-id="255ef-150">`Pages/Todo.razor`:</span><span class="sxs-lookup"><span data-stu-id="255ef-150">`Pages/Todo.razor`:</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo2.razor?highlight=5-10,13)]

1. <span data-ttu-id="255ef-151">L'app richiede elementi dell'interfaccia utente per l'aggiunta di voci todo all'elenco.</span><span class="sxs-lookup"><span data-stu-id="255ef-151">The app requires UI elements for adding todo items to the list.</span></span> <span data-ttu-id="255ef-152">Aggiungere un input di testo (`<input>`) e un pulsante (`<button>`) sotto l'elenco non ordinato (`<ul>...</ul>`):</span><span class="sxs-lookup"><span data-stu-id="255ef-152">Add a text input (`<input>`) and a button (`<button>`) below the unordered list (`<ul>...</ul>`):</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo3.razor?highlight=12-13)]

1. <span data-ttu-id="255ef-153">Salvare il `TodoItem.cs` file e il `Pages/Todo.razor` file aggiornato.</span><span class="sxs-lookup"><span data-stu-id="255ef-153">Save the `TodoItem.cs` file and the updated `Pages/Todo.razor` file.</span></span> <span data-ttu-id="255ef-154">Nella shell dei comandi l'app viene ricompilata automaticamente quando i file vengono salvati.</span><span class="sxs-lookup"><span data-stu-id="255ef-154">In the command shell, the app is automatically rebuilt when the files are saved.</span></span> <span data-ttu-id="255ef-155">Il browser perde temporaneamente la connessione all'app e quindi ricarica la pagina quando viene ristabilita la connessione.</span><span class="sxs-lookup"><span data-stu-id="255ef-155">The browser temporarily loses its connection to the app and then reloads the page when the connection is re-established.</span></span>

1. <span data-ttu-id="255ef-156">Quando il **`Add todo`** pulsante è selezionato, non accade nulla perché un gestore eventi non è associato al pulsante.</span><span class="sxs-lookup"><span data-stu-id="255ef-156">When the **`Add todo`** button is selected, nothing happens because an event handler isn't attached to the button.</span></span>

1. <span data-ttu-id="255ef-157">Aggiungere un `AddTodo` metodo al `Todo` componente e registrare il metodo per il pulsante usando l' `@onclick` attributo.</span><span class="sxs-lookup"><span data-stu-id="255ef-157">Add an `AddTodo` method to the `Todo` component and register the method for the button using the `@onclick` attribute.</span></span> <span data-ttu-id="255ef-158">Il metodo C# `AddTodo` viene chiamato quando viene selezionato il pulsante:</span><span class="sxs-lookup"><span data-stu-id="255ef-158">The `AddTodo` C# method is called when the button is selected:</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo4.razor?highlight=2,7-10)]

1. <span data-ttu-id="255ef-159">Per ottenere il titolo del nuovo elemento todo, aggiungere un `newTodo` campo stringa nella parte superiore del `@code` blocco:</span><span class="sxs-lookup"><span data-stu-id="255ef-159">To get the title of the new todo item, add a `newTodo` string field at the top of the `@code` block:</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo5.razor?highlight=3)]

   <span data-ttu-id="255ef-160">Modificare l' `<input>` elemento di testo da associare all' `newTodo` `@bind` attributo:</span><span class="sxs-lookup"><span data-stu-id="255ef-160">Modify the text `<input>` element to bind `newTodo` with the `@bind` attribute:</span></span>

   ```razor
   <input placeholder="Something todo" @bind="newTodo" />
   ```

1. <span data-ttu-id="255ef-161">Aggiornare il metodo `AddTodo` per aggiungere l'elemento `TodoItem` con il titolo specificato all'elenco.</span><span class="sxs-lookup"><span data-stu-id="255ef-161">Update the `AddTodo` method to add the `TodoItem` with the specified title to the list.</span></span> <span data-ttu-id="255ef-162">Cancellare il valore dell'input di testo impostando `newTodo` su una stringa vuota:</span><span class="sxs-lookup"><span data-stu-id="255ef-162">Clear the value of the text input by setting `newTodo` to an empty string:</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo6.razor?highlight=19-26)]

1. <span data-ttu-id="255ef-163">Salvare il file.`Pages/Todo.razor`</span><span class="sxs-lookup"><span data-stu-id="255ef-163">Save the `Pages/Todo.razor` file.</span></span> <span data-ttu-id="255ef-164">L'app viene ricompilata automaticamente nella shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="255ef-164">The app is automatically rebuilt in the command shell.</span></span> <span data-ttu-id="255ef-165">La pagina viene ricaricata nel browser dopo la riconnessione del browser all'app.</span><span class="sxs-lookup"><span data-stu-id="255ef-165">The page reloads in the browser after the browser reconnects to the app.</span></span>

1. <span data-ttu-id="255ef-166">Il testo del titolo per ogni elemento Todo può essere reso modificabile e una casella di controllo può consentire all'utente di tenere traccia degli elementi completati.</span><span class="sxs-lookup"><span data-stu-id="255ef-166">The title text for each todo item can be made editable, and a check box can help the user keep track of completed items.</span></span> <span data-ttu-id="255ef-167">Aggiungere un input casella di controllo per ogni elemento attività e associarne il valore alla proprietà `IsDone`.</span><span class="sxs-lookup"><span data-stu-id="255ef-167">Add a check box input for each todo item and bind its value to the `IsDone` property.</span></span> <span data-ttu-id="255ef-168">Passare `@todo.Title` a un `<input>` elemento associato a `todo.Title` con `@bind` :</span><span class="sxs-lookup"><span data-stu-id="255ef-168">Change `@todo.Title` to an `<input>` element bound to `todo.Title` with `@bind`:</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo7.razor?highlight=4-7)]

1. <span data-ttu-id="255ef-169">Aggiornare l' `<h3>` intestazione per visualizzare un conteggio del numero di elementi todo non completi ( `IsDone` is `false` ).</span><span class="sxs-lookup"><span data-stu-id="255ef-169">Update the `<h3>` header to show a count of the number of todo items that aren't complete (`IsDone` is `false`).</span></span>

   ```razor
   <h3>Todo (@todos.Count(todo => !todo.IsDone))</h3>
   ```

1. <span data-ttu-id="255ef-170">Componente completato `Todo` ( `Pages/Todo.razor` ):</span><span class="sxs-lookup"><span data-stu-id="255ef-170">The completed `Todo` component (`Pages/Todo.razor`):</span></span>

   [!code-razor[](build-a-blazor-app/samples_snapshot/Todo1.razor)]

1. <span data-ttu-id="255ef-171">Salvare il file.`Pages/Todo.razor`</span><span class="sxs-lookup"><span data-stu-id="255ef-171">Save the `Pages/Todo.razor` file.</span></span> <span data-ttu-id="255ef-172">L'app viene ricompilata automaticamente nella shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="255ef-172">The app is automatically rebuilt in the command shell.</span></span> <span data-ttu-id="255ef-173">La pagina viene ricaricata nel browser dopo la riconnessione del browser all'app.</span><span class="sxs-lookup"><span data-stu-id="255ef-173">The page reloads in the browser after the browser reconnects to the app.</span></span>

1. <span data-ttu-id="255ef-174">Aggiungere elementi, modificare elementi e contrassegnare gli elementi todo eseguiti per testare il componente.</span><span class="sxs-lookup"><span data-stu-id="255ef-174">Add items, edit items, and mark todo items done to test the component.</span></span>

1. <span data-ttu-id="255ef-175">Al termine, arrestare l'app nella shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="255ef-175">When finished, shut down the app in the command shell.</span></span> <span data-ttu-id="255ef-176">Molte shell dei comandi accettano il comando della tastiera <kbd>CTRL</kbd> + <kbd>c</kbd> per arrestare un'app.</span><span class="sxs-lookup"><span data-stu-id="255ef-176">Many command shells accept the keyboard command <kbd>Ctrl</kbd>+<kbd>c</kbd> to stop an app.</span></span>

## <a name="next-steps"></a><span data-ttu-id="255ef-177">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="255ef-177">Next steps</span></span>

<span data-ttu-id="255ef-178">In questa esercitazione sono state illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="255ef-178">In this tutorial, you learned how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="255ef-179">Creare un progetto di Blazor app elenco TODO</span><span class="sxs-lookup"><span data-stu-id="255ef-179">Create a todo list Blazor app project</span></span>
> * <span data-ttu-id="255ef-180">Modificare i Razor componenti</span><span class="sxs-lookup"><span data-stu-id="255ef-180">Modify Razor components</span></span>
> * <span data-ttu-id="255ef-181">Usare la gestione degli eventi e data binding nei componenti</span><span class="sxs-lookup"><span data-stu-id="255ef-181">Use event handling and data binding in components</span></span>
> * <span data-ttu-id="255ef-182">Usare il routing in un' Blazor app</span><span class="sxs-lookup"><span data-stu-id="255ef-182">Use routing in a Blazor app</span></span>

<span data-ttu-id="255ef-183">Informazioni sugli strumenti per ASP.NET Core Blazor :</span><span class="sxs-lookup"><span data-stu-id="255ef-183">Learn about tooling for ASP.NET Core Blazor:</span></span>

> [!div class="nextstepaction"]
> <xref:blazor/tooling>
