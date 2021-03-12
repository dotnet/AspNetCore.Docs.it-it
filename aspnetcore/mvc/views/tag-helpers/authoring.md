---
title: Creare helper tag in ASP.NET Core
author: rick-anderson
description: Informazioni su come creare helper tag in ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: mvc/views/tag-helpers/authoring
ms.openlocfilehash: e0c9f450e4eded49694cbbb0e9fa2614a221ab14
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586826"
---
# <a name="author-tag-helpers-in-aspnet-core"></a><span data-ttu-id="00f1c-103">Creare helper tag in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="00f1c-103">Author Tag Helpers in ASP.NET Core</span></span>

<span data-ttu-id="00f1c-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="00f1c-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="00f1c-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/authoring/sample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="00f1c-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/tag-helpers/authoring/sample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="get-started-with-tag-helpers"></a><span data-ttu-id="00f1c-106">Guida introduttiva agli helper tag</span><span class="sxs-lookup"><span data-stu-id="00f1c-106">Get started with Tag Helpers</span></span>

<span data-ttu-id="00f1c-107">Questa esercitazione rappresenta un'introduzione alla programmazione di helper tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-107">This tutorial provides an introduction to programming Tag Helpers.</span></span> <span data-ttu-id="00f1c-108">[Introduzione agli helper tag](intro.md) descrive i vantaggi offerti dagli helper tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-108">[Introduction to Tag Helpers](intro.md) describes the benefits that Tag Helpers provide.</span></span>

<span data-ttu-id="00f1c-109">Un helper tag è una classe che implementa l'interfaccia `ITagHelper`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-109">A tag helper is any class that implements the `ITagHelper` interface.</span></span> <span data-ttu-id="00f1c-110">Quando si crea un helper tag, tuttavia, in genere lo si deriva da `TagHelper`. In questo modo si ottiene l'accesso al metodo `Process`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-110">However, when you author a tag helper, you generally derive from `TagHelper`, doing so gives you access to the `Process` method.</span></span>

1. <span data-ttu-id="00f1c-111">Creare un nuovo progetto ASP.NET Core denominato **AuthoringTagHelpers**.</span><span class="sxs-lookup"><span data-stu-id="00f1c-111">Create a new ASP.NET Core project called **AuthoringTagHelpers**.</span></span> <span data-ttu-id="00f1c-112">Per questo progetto non sarà necessaria l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="00f1c-112">You won't need authentication for this project.</span></span>

1. <span data-ttu-id="00f1c-113">Creare una cartella che contenga gli helper tag denominata *TagHelpers*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-113">Create a folder to hold the Tag Helpers called *TagHelpers*.</span></span> <span data-ttu-id="00f1c-114">La cartella *TagHelpers\*\*non* è obbligatoria, ma rappresenta una convenzione sensata.</span><span class="sxs-lookup"><span data-stu-id="00f1c-114">The *TagHelpers* folder is *not* required, but it's a reasonable convention.</span></span> <span data-ttu-id="00f1c-115">È ora possibile iniziare a scrivere alcuni helper tag semplici.</span><span class="sxs-lookup"><span data-stu-id="00f1c-115">Now let's get started writing some simple tag helpers.</span></span>

## <a name="a-minimal-tag-helper"></a><span data-ttu-id="00f1c-116">Un helper tag piccolissimo</span><span class="sxs-lookup"><span data-stu-id="00f1c-116">A minimal Tag Helper</span></span>

<span data-ttu-id="00f1c-117">In questa sezione verrà scritto un helper tag che aggiorna un tag di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="00f1c-117">In this section, you write a tag helper that updates an email tag.</span></span> <span data-ttu-id="00f1c-118">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="00f1c-118">For example:</span></span>

```html
<email>Support</email>
```

<span data-ttu-id="00f1c-119">Il server userà l'helper tag di posta elettronica creato per convertire il markup nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-119">The server will use our email tag helper to convert that markup into the following:</span></span>

```html
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

<span data-ttu-id="00f1c-120">In altre parole, un tag di ancoraggio che dà come risultato un collegamento di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="00f1c-120">That is, an anchor tag that makes this an email link.</span></span> <span data-ttu-id="00f1c-121">È necessario eseguire questa operazione se si sta scrivendo un motore di blog e si vuole inviare posta elettronica di marketing o di supporto ad altri contatti, tutti nello stesso dominio.</span><span class="sxs-lookup"><span data-stu-id="00f1c-121">You might want to do this if you are writing a blog engine and need it to send email for marketing, support, and other contacts, all to the same domain.</span></span>

1. <span data-ttu-id="00f1c-122">Aggiungere la classe `EmailTagHelper` seguente alla cartella *TagHelpers*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-122">Add the following `EmailTagHelper` class to the *TagHelpers* folder.</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1EmailTagHelperCopy.cs)]

   * <span data-ttu-id="00f1c-123">Gli helper tag usano una convenzione di denominazione che interessa gli elementi del nome della classe di base (meno la parte *TagHelper* del nome della classe).</span><span class="sxs-lookup"><span data-stu-id="00f1c-123">Tag helpers use a naming convention that targets elements of the root class name (minus the *TagHelper* portion of the class name).</span></span> <span data-ttu-id="00f1c-124">In questo esempio, il nome radice di **EmailTagHelper** è *posta elettronica*, quindi il `<email>` tag verrà assegnato come destinazione.</span><span class="sxs-lookup"><span data-stu-id="00f1c-124">In this example, the root name of **EmailTagHelper** is *email*, so the `<email>` tag will be targeted.</span></span> <span data-ttu-id="00f1c-125">Questa convenzione di denominazione dovrebbe funzionare per la maggior parte degli helper tag. Più avanti verrà illustrato come eseguirne l'override.</span><span class="sxs-lookup"><span data-stu-id="00f1c-125">This naming convention should work for most tag helpers, later on I'll show how to override it.</span></span>

   * <span data-ttu-id="00f1c-126">La classe `EmailTagHelper` deriva da `TagHelper`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-126">The `EmailTagHelper` class derives from `TagHelper`.</span></span> <span data-ttu-id="00f1c-127">La classe `TagHelper` mette a disposizione metodi e proprietà per la scrittura di helper tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-127">The `TagHelper` class provides methods and properties for writing Tag Helpers.</span></span>

   * <span data-ttu-id="00f1c-128">Il metodo `Process` sottoposto a override controlla ciò che l'helper tag fa quando viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="00f1c-128">The overridden `Process` method controls what the tag helper does when executed.</span></span> <span data-ttu-id="00f1c-129">La classe `TagHelper` fornisce anche una versione asincrona (`ProcessAsync`) con gli stessi parametri.</span><span class="sxs-lookup"><span data-stu-id="00f1c-129">The `TagHelper` class also provides an asynchronous version (`ProcessAsync`) with the same parameters.</span></span>

   * <span data-ttu-id="00f1c-130">Il parametro context di `Process` (e `ProcessAsync`) contiene informazioni associate all'esecuzione del tag HTML corrente.</span><span class="sxs-lookup"><span data-stu-id="00f1c-130">The context parameter to `Process` (and `ProcessAsync`) contains information associated with the execution of the current HTML tag.</span></span>

   * <span data-ttu-id="00f1c-131">Il parametro output di `Process` (e `ProcessAsync`) contiene un elemento HTML con stato rappresentativo dell'origine originale usato per generare un tag e del contenuto HTML.</span><span class="sxs-lookup"><span data-stu-id="00f1c-131">The output parameter to `Process` (and `ProcessAsync`) contains a stateful HTML element representative of the original source used to generate an HTML tag and content.</span></span>

   * <span data-ttu-id="00f1c-132">Il nome della classe usata ha il suffisso **TagHelper**, che *non* è obbligatorio, ma il cui uso è considerato una convenzione basata su procedure consigliate.</span><span class="sxs-lookup"><span data-stu-id="00f1c-132">Our class name has a suffix of **TagHelper**, which is *not* required, but it's considered a best practice convention.</span></span> <span data-ttu-id="00f1c-133">È possibile dichiarare la classe come:</span><span class="sxs-lookup"><span data-stu-id="00f1c-133">You could declare the class as:</span></span>

   ```csharp
   public class Email : TagHelper
   ```

1. <span data-ttu-id="00f1c-134">Per rendere la `EmailTagHelper` classe disponibile a tutte le Razor Visualizzazioni, aggiungere la `addTagHelper` direttiva al file *views/_ViewImports. cshtml* :</span><span class="sxs-lookup"><span data-stu-id="00f1c-134">To make the `EmailTagHelper` class available to all our Razor views, add the `addTagHelper` directive to the *Views/_ViewImports.cshtml* file:</span></span>

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopyEmail.cshtml?highlight=2,3)]

   <span data-ttu-id="00f1c-135">Il codice qui sopra usa la sintassi con caratteri jolly per specificare che tutti gli helper tag in questo assembly saranno disponibili.</span><span class="sxs-lookup"><span data-stu-id="00f1c-135">The code above uses the wildcard syntax to specify all the tag helpers in our assembly will be available.</span></span> <span data-ttu-id="00f1c-136">La prima stringa dopo `@addTagHelper` specifica l'helper tag da caricare (usare "\*" per indicare tutti gli helper tag), e la seconda stringa "AuthoringTagHelpers" specifica l'assembly in cui si trova l'helper tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-136">The first string after `@addTagHelper` specifies the tag helper to load (Use "\*" for all tag helpers), and the second string "AuthoringTagHelpers" specifies the assembly the tag helper is in.</span></span> <span data-ttu-id="00f1c-137">Si noti anche che la seconda riga introduce gli helper tag di ASP.NET Core MVC usando la sintassi con caratteri jolly (gli helper sono descritti in [Introduzione agli helper Tag](intro.md)). Si tratta della `@addTagHelper` direttiva che rende l'helper tag disponibile per la Razor visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="00f1c-137">Also, note that the second line brings in the ASP.NET Core MVC tag helpers using the wildcard syntax (those helpers are discussed in [Introduction to Tag Helpers](intro.md).) It's the `@addTagHelper` directive that makes the tag helper available to the Razor view.</span></span> <span data-ttu-id="00f1c-138">In alternativa, è possibile specificare il nome completo dell'helper tag, come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="00f1c-138">Alternatively, you can provide the fully qualified name (FQN) of a tag helper as shown below:</span></span>

```csharp
@using AuthoringTagHelpers
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.EmailTagHelper, AuthoringTagHelpers
```

<!--
the following snippet uses TagHelpers3 and should use TagHelpers (not the 3)
    [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImports.cshtml?highlight=3&range=1-3)]
-->

<span data-ttu-id="00f1c-139">Per aggiungere un helper tag a una visualizzazione usando un nome completo, aggiungere prima il nome completo (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`) e quindi il **nome dell'assembly** (*AuthoringTagHelpers*, non necessariamente `namespace`).</span><span class="sxs-lookup"><span data-stu-id="00f1c-139">To add a tag helper to a view using a FQN, you first add the FQN (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`), and then the **assembly name** (*AuthoringTagHelpers*, not necessarily the `namespace`).</span></span> <span data-ttu-id="00f1c-140">La maggior parte degli sviluppatori preferisce usare la sintassi con caratteri jolly.</span><span class="sxs-lookup"><span data-stu-id="00f1c-140">Most developers will prefer to use the wildcard syntax.</span></span> <span data-ttu-id="00f1c-141">[Introduzione agli helper tag](intro.md) descrive in dettaglio la sintassi di aggiunta, rimozione, gerarchia degli helper tag, nonché la sintassi con caratteri jolly.</span><span class="sxs-lookup"><span data-stu-id="00f1c-141">[Introduction to Tag Helpers](intro.md) goes into detail on tag helper adding, removing, hierarchy, and wildcard syntax.</span></span>

1. <span data-ttu-id="00f1c-142">Aggiornare il markup nel file *Views/Home/Contact.cshtml* con queste modifiche:</span><span class="sxs-lookup"><span data-stu-id="00f1c-142">Update the markup in the *Views/Home/Contact.cshtml* file with these changes:</span></span>

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. <span data-ttu-id="00f1c-143">Eseguire l'app e usare il browser preferito per visualizzare l'origine HTML e verificare che i tag di posta elettronica siano stati sostituiti dal markup di ancoraggio, ad esempio, `<a>Support</a>`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-143">Run the app and use your favorite browser to view the HTML source so you can verify that the email tags are replaced with anchor markup (For example, `<a>Support</a>`).</span></span> <span data-ttu-id="00f1c-144">*Support* e *Marketing* sono visualizzati come collegamenti, ma non hanno un attributo `href` che li renda funzionali.</span><span class="sxs-lookup"><span data-stu-id="00f1c-144">*Support* and *Marketing* are rendered as a links, but they don't have an `href` attribute to make them functional.</span></span> <span data-ttu-id="00f1c-145">Questo problema verrà corretto nella prossima sezione.</span><span class="sxs-lookup"><span data-stu-id="00f1c-145">We'll fix that in the next section.</span></span>

## <a name="setattribute-and-setcontent"></a><span data-ttu-id="00f1c-146">SetAttribute e SetContent</span><span class="sxs-lookup"><span data-stu-id="00f1c-146">SetAttribute and SetContent</span></span>

<span data-ttu-id="00f1c-147">In questa sezione la classe `EmailTagHelper` verrà aggiornata in modo che crei un tag di ancoraggio valido per la posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="00f1c-147">In this section, we'll update the `EmailTagHelper` so that it will create a valid anchor tag for email.</span></span> <span data-ttu-id="00f1c-148">Il modulo verrà aggiornato in modo da ottenere le informazioni da una Razor vista (sotto forma di `mail-to` attributo) e usarle per generare l'ancoraggio.</span><span class="sxs-lookup"><span data-stu-id="00f1c-148">We'll update it to take information from a Razor view (in the form of a `mail-to` attribute) and use that in generating the anchor.</span></span>

<span data-ttu-id="00f1c-149">Aggiornare la classe `EmailTagHelper` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-149">Update the `EmailTagHelper` class with the following:</span></span>

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?range=6-22)]

* <span data-ttu-id="00f1c-150">I nomi della classe e delle proprietà per gli helper tag, scritti secondo la convenzione Pascal, vengono convertiti nel formato [kebab case](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101) corrispondente.</span><span class="sxs-lookup"><span data-stu-id="00f1c-150">Pascal-cased class and property names for tag helpers are translated into their [kebab case](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101).</span></span> <span data-ttu-id="00f1c-151">Per usare l'attributo `MailTo`, pertanto, si userà l'equivalente `<email mail-to="value"/>`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-151">Therefore, to use the `MailTo` attribute, you'll use `<email mail-to="value"/>` equivalent.</span></span>

* <span data-ttu-id="00f1c-152">L'ultima riga imposta il contenuto completato per l'helper tag dalle minime funzioni.</span><span class="sxs-lookup"><span data-stu-id="00f1c-152">The last line sets the completed content for our minimally functional tag helper.</span></span>

* <span data-ttu-id="00f1c-153">La riga evidenziata illustra la sintassi per l'aggiunta di attributi:</span><span class="sxs-lookup"><span data-stu-id="00f1c-153">The highlighted line shows the syntax for adding attributes:</span></span>

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?highlight=6&range=14-21)]

<span data-ttu-id="00f1c-154">Questo approccio funziona per l'attributo "href" a condizione che non esista nella raccolta di attributi.</span><span class="sxs-lookup"><span data-stu-id="00f1c-154">That approach works for the attribute "href" as long as it doesn't currently exist in the attributes collection.</span></span> <span data-ttu-id="00f1c-155">È anche possibile usare il metodo `output.Attributes.Add` per aggiungere un attributo di helper tag alla fine della raccolta di attributi di tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-155">You can also use the `output.Attributes.Add` method to add a tag helper attribute to the end of the collection of tag attributes.</span></span>

1. <span data-ttu-id="00f1c-156">Aggiornare il markup nel file *Views/Home/Contact.cshtml* con queste modifiche:</span><span class="sxs-lookup"><span data-stu-id="00f1c-156">Update the markup in the *Views/Home/Contact.cshtml* file with these changes:</span></span>

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/ContactCopy.cshtml?highlight=15,16)]

1. <span data-ttu-id="00f1c-157">Eseguire l'app e verificare che generi collegamenti corretti.</span><span class="sxs-lookup"><span data-stu-id="00f1c-157">Run the app and verify that it generates the correct links.</span></span>

<a name="self-closing"></a>

   > [!NOTE]
   > <span data-ttu-id="00f1c-158">Se si scrive il tag di posta elettronica a chiusura automatica (`<email mail-to="Rick" />`), anche l'output finale è a chiusura automatica.</span><span class="sxs-lookup"><span data-stu-id="00f1c-158">If you were to write the email tag self-closing (`<email mail-to="Rick" />`), the final output would also be self-closing.</span></span> <span data-ttu-id="00f1c-159">Per consentire la scrittura del tag con solo un tag di inizio ( `<email mail-to="Rick">` ) è necessario contrassegnare la classe con quanto segue:</span><span class="sxs-lookup"><span data-stu-id="00f1c-159">To enable the ability to write the tag with only a start tag (`<email mail-to="Rick">`) you must mark the class with the following:</span></span>
   >
   > [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailVoid.cs?highlight=1&range=6-10)]

   <span data-ttu-id="00f1c-160">Con un helper tag di posta elettronica a chiusura automatica, l'output è `<a href="mailto:Rick@contoso.com" />`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-160">With a self-closing email tag helper, the output would be `<a href="mailto:Rick@contoso.com" />`.</span></span> <span data-ttu-id="00f1c-161">I tag di ancoraggio a chiusura automatica non costituiscono codice HTML valido ed è quindi necessario evitare di crearli, ma può essere necessario creare un helper tag a chiusura automatica.</span><span class="sxs-lookup"><span data-stu-id="00f1c-161">Self-closing anchor tags are not valid HTML, so you wouldn't want to create one, but you might want to create a tag helper that's self-closing.</span></span> <span data-ttu-id="00f1c-162">Gli helper tag impostano il tipo della proprietà `TagMode` dopo la lettura di un tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-162">Tag helpers set the type of the `TagMode` property after reading a tag.</span></span>

### <a name="processasync"></a><span data-ttu-id="00f1c-163">ProcessAsync</span><span class="sxs-lookup"><span data-stu-id="00f1c-163">ProcessAsync</span></span>

<span data-ttu-id="00f1c-164">In questa sezione verrà scritto un helper di posta elettronica asincrono.</span><span class="sxs-lookup"><span data-stu-id="00f1c-164">In this section, we'll write an asynchronous email helper.</span></span>

1. <span data-ttu-id="00f1c-165">Sostituire la classe `EmailTagHelper` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-165">Replace the `EmailTagHelper` class with the following code:</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelper.cs?range=6-17)]

   <span data-ttu-id="00f1c-166">**Note:**</span><span class="sxs-lookup"><span data-stu-id="00f1c-166">**Notes:**</span></span>

   * <span data-ttu-id="00f1c-167">Questa versione usa il metodo `ProcessAsync` asincrono.</span><span class="sxs-lookup"><span data-stu-id="00f1c-167">This version uses the asynchronous `ProcessAsync` method.</span></span> <span data-ttu-id="00f1c-168">`GetChildContentAsync` asincrono restituisce un `Task` contenente `TagHelperContent`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-168">The asynchronous `GetChildContentAsync` returns a `Task` containing the `TagHelperContent`.</span></span>

   * <span data-ttu-id="00f1c-169">Usare il parametro `output` per ottenere il contenuto dell'elemento HTML.</span><span class="sxs-lookup"><span data-stu-id="00f1c-169">Use the `output` parameter to get contents of the HTML element.</span></span>

1. <span data-ttu-id="00f1c-170">Apportare le seguenti modifiche al file *Views/Home/Contact.cshtml*, in modo che l'helper tag possa ottenere la posta elettronica di destinazione.</span><span class="sxs-lookup"><span data-stu-id="00f1c-170">Make the following change to the *Views/Home/Contact.cshtml* file so the tag helper can get the target email.</span></span>

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. <span data-ttu-id="00f1c-171">Eseguire l'app e verificare che generi collegamenti di posta elettronica validi.</span><span class="sxs-lookup"><span data-stu-id="00f1c-171">Run the app and verify that it generates valid email links.</span></span>

### <a name="removeall-precontentsethtmlcontent-and-postcontentsethtmlcontent"></a><span data-ttu-id="00f1c-172">RemoveAll, PreContent.SetHtmlContent e PostContent.SetHtmlContent</span><span class="sxs-lookup"><span data-stu-id="00f1c-172">RemoveAll, PreContent.SetHtmlContent and PostContent.SetHtmlContent</span></span>

1. <span data-ttu-id="00f1c-173">Aggiungere la classe `BoldTagHelper` seguente alla cartella *TagHelpers*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-173">Add the following `BoldTagHelper` class to the *TagHelpers* folder.</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/BoldTagHelper.cs)]

   * <span data-ttu-id="00f1c-174">L'attributo `[HtmlTargetElement]` passa un parametro di attributo che specifica che qualsiasi elemento HTML che contenga un attributo HTML denominato "bold" corrisponderà e che quindi verrà eseguito il metodo di override `Process` nella classe.</span><span class="sxs-lookup"><span data-stu-id="00f1c-174">The `[HtmlTargetElement]` attribute passes an attribute parameter that specifies that any HTML element that contains an HTML attribute named "bold" will match, and the `Process` override method in the class will run.</span></span> <span data-ttu-id="00f1c-175">Nell'esempio, il metodo `Process` rimuove l'attributo "bold" e racchiude il markup tra `<strong></strong>`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-175">In our sample, the `Process` method removes the "bold" attribute and surrounds the containing markup with `<strong></strong>`.</span></span>

   * <span data-ttu-id="00f1c-176">Dato che non si deve sostituire il contenuto esistente del tag, è necessario scrivere il tag `<strong>` di apertura con il metodo `PreContent.SetHtmlContent` e il tag `</strong>` di chiusura con il metodo `PostContent.SetHtmlContent`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-176">Because you don't want to replace the existing tag content, you must write the opening `<strong>` tag with the `PreContent.SetHtmlContent` method and the closing `</strong>` tag with the `PostContent.SetHtmlContent` method.</span></span>

1. <span data-ttu-id="00f1c-177">Modificare la visualizzazione *About.cshtml* in modo che contenga un valore di attributo `bold`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-177">Modify the *About.cshtml* view to contain a `bold` attribute value.</span></span> <span data-ttu-id="00f1c-178">Il codice completo è illustrato di seguito.</span><span class="sxs-lookup"><span data-stu-id="00f1c-178">The completed code is shown below.</span></span>

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutBoldOnly.cshtml?highlight=7)]

1. <span data-ttu-id="00f1c-179">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="00f1c-179">Run the app.</span></span> <span data-ttu-id="00f1c-180">È possibile usare il browser preferito per controllare l'origine e verificare il markup.</span><span class="sxs-lookup"><span data-stu-id="00f1c-180">You can use your favorite browser to inspect the source and verify the markup.</span></span>

   <span data-ttu-id="00f1c-181">L'attributo `[HtmlTargetElement]` qui sopra ha come destinazione solo il markup HTML che fornisce un nome di attributo corrispondente a "bold".</span><span class="sxs-lookup"><span data-stu-id="00f1c-181">The `[HtmlTargetElement]` attribute above only targets HTML markup that provides an attribute name of "bold".</span></span> <span data-ttu-id="00f1c-182">L'elemento `<bold>` non è stato modificato dall'helper tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-182">The `<bold>` element wasn't modified by the tag helper.</span></span>

1. <span data-ttu-id="00f1c-183">Impostare come commento la riga dell'attributo `[HtmlTargetElement]`. Per impostazione predefinita, userà quindi tag `<bold>`, vale a dire il markup HTML nel formato `<bold>`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-183">Comment out the `[HtmlTargetElement]` attribute line and it will default to targeting `<bold>` tags, that is, HTML markup of the form `<bold>`.</span></span> <span data-ttu-id="00f1c-184">Tenere presente che la convenzione di denominazione predefinita corrisponderà al nome della classe **Bold** TagHelper per i tag `<bold>`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-184">Remember, the default naming convention will match the class name **Bold** TagHelper to `<bold>` tags.</span></span>

1. <span data-ttu-id="00f1c-185">Eseguire l'applicazione e verificare che il tag `<bold>` venga elaborato dall'helper tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-185">Run the app and verify that the `<bold>` tag is processed by the tag helper.</span></span>

<span data-ttu-id="00f1c-186">La decorazione di una classe con più attributi `[HtmlTargetElement]` risulta in un OR logico delle destinazioni.</span><span class="sxs-lookup"><span data-stu-id="00f1c-186">Decorating a class with multiple `[HtmlTargetElement]` attributes results in a logical-OR of the targets.</span></span> <span data-ttu-id="00f1c-187">Usando il codice riportato di seguito, ad esempio, un tag bold o un attributo bold corrisponderanno.</span><span class="sxs-lookup"><span data-stu-id="00f1c-187">For example, using the code below, a bold tag or a bold attribute will match.</span></span>

[!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zBoldTagHelperCopy.cs?highlight=1,2&range=5-15)]

<span data-ttu-id="00f1c-188">Se si aggiungono più attributi alla stessa istruzione, il runtime li gestisce come AND logici.</span><span class="sxs-lookup"><span data-stu-id="00f1c-188">When multiple attributes are added to the same statement, the runtime treats them as a logical-AND.</span></span> <span data-ttu-id="00f1c-189">Nel codice riportato di seguito, ad esempio, perché corrisponda, un elemento HTML deve essere denominato "bold" con un attributo denominato "bold" (`<bold bold />`).</span><span class="sxs-lookup"><span data-stu-id="00f1c-189">For example, in the code below, an HTML element must be named "bold" with an attribute named "bold" (`<bold bold />`) to match.</span></span>

```csharp
[HtmlTargetElement("bold", Attributes = "bold")]
   ```

<span data-ttu-id="00f1c-190">È anche possibile usare `[HtmlTargetElement]` per modificare il nome dell'elemento di destinazione.</span><span class="sxs-lookup"><span data-stu-id="00f1c-190">You can also use the `[HtmlTargetElement]` to change the name of the targeted element.</span></span> <span data-ttu-id="00f1c-191">Se ad esempio si vuole che `BoldTagHelper` abbia come destinazione tag `<MyBold>`, usare l'attributo seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-191">For example if you wanted the `BoldTagHelper` to target `<MyBold>` tags, you would use the following attribute:</span></span>

```csharp
[HtmlTargetElement("MyBold")]
```

## <a name="pass-a-model-to-a-tag-helper"></a><span data-ttu-id="00f1c-192">Passare un modello a un helper tag</span><span class="sxs-lookup"><span data-stu-id="00f1c-192">Pass a model to a Tag Helper</span></span>

1. <span data-ttu-id="00f1c-193">Aggiungere una cartella *Models*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-193">Add a *Models* folder.</span></span>

1. <span data-ttu-id="00f1c-194">Aggiungere la classe `WebsiteContext` seguente alla cartella *Models*:</span><span class="sxs-lookup"><span data-stu-id="00f1c-194">Add the following `WebsiteContext` class to the *Models* folder:</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Models/WebsiteContext.cs)]

1. <span data-ttu-id="00f1c-195">Aggiungere la classe `WebsiteInformationTagHelper` seguente alla cartella *TagHelpers*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-195">Add the following `WebsiteInformationTagHelper` class to the *TagHelpers* folder.</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/WebsiteInformationTagHelper.cs)]

   * <span data-ttu-id="00f1c-196">Come affermato in precedenza, gli helper tag convertono i nomi delle relative classi e proprietà C#, definiti secondo la convenzione Pascal, in formato [kebab case](https://wiki.c2.com/?KebabCase).</span><span class="sxs-lookup"><span data-stu-id="00f1c-196">As mentioned previously, tag helpers translates Pascal-cased C# class names and properties for tag helpers into [kebab case](https://wiki.c2.com/?KebabCase).</span></span> <span data-ttu-id="00f1c-197">Pertanto, per utilizzare `WebsiteInformationTagHelper` in Razor , si scriverà `<website-information />` .</span><span class="sxs-lookup"><span data-stu-id="00f1c-197">Therefore, to use the `WebsiteInformationTagHelper` in Razor, you'll write `<website-information />`.</span></span>

   * <span data-ttu-id="00f1c-198">Non si sta identificando in modo esplicito l'elemento di destinazione con l'attributo `[HtmlTargetElement]` e quindi verrà considerato come destinazione il valore predefinito di `website-information`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-198">You are not explicitly identifying the target element with the `[HtmlTargetElement]` attribute, so the default of `website-information` will be targeted.</span></span> <span data-ttu-id="00f1c-199">Se è stato applicato l'attributo seguente (si noti che non è nel formato kebab case ma corrisponde al nome della classe):</span><span class="sxs-lookup"><span data-stu-id="00f1c-199">If you applied the following attribute (note it's not kebab case but matches the class name):</span></span>

   ```csharp
   [HtmlTargetElement("WebsiteInformation")]
   ```

   <span data-ttu-id="00f1c-200">Il tag nel formato kebab case `<website-information />` non corrisponde.</span><span class="sxs-lookup"><span data-stu-id="00f1c-200">The kebab case tag `<website-information />` wouldn't match.</span></span> <span data-ttu-id="00f1c-201">Se si vuole usare l'attributo `[HtmlTargetElement]`, è necessario usare il formato kebab case come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="00f1c-201">If you want use the `[HtmlTargetElement]` attribute, you would use kebab case as shown below:</span></span>

   ```csharp
   [HtmlTargetElement("Website-Information")]
   ```

   * <span data-ttu-id="00f1c-202">Gli elementi a chiusura automatica non hanno contenuto.</span><span class="sxs-lookup"><span data-stu-id="00f1c-202">Elements that are self-closing have no content.</span></span> <span data-ttu-id="00f1c-203">Per questo esempio, il Razor markup userà un tag di chiusura automatica, ma l'helper Tag creerà un elemento [Section](https://www.w3.org/TR/html5/sections.html#the-section-element) , che non è a chiusura automatica e si sta scrivendo contenuto all'interno dell' `section` elemento.</span><span class="sxs-lookup"><span data-stu-id="00f1c-203">For this example, the Razor markup will use a self-closing tag, but the tag helper will be creating a [section](https://www.w3.org/TR/html5/sections.html#the-section-element) element (which isn't self-closing and you are writing content inside the `section` element).</span></span> <span data-ttu-id="00f1c-204">Per scrivere output, è pertanto necessario impostare `TagMode` su `StartTagAndEndTag`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-204">Therefore, you need to set `TagMode` to `StartTagAndEndTag` to write output.</span></span> <span data-ttu-id="00f1c-205">In alternativa, è possibile impostare come commento la riga che imposta `TagMode` e scrivere markup con un tag di chiusura.</span><span class="sxs-lookup"><span data-stu-id="00f1c-205">Alternatively, you can comment out the line setting `TagMode` and write markup with a closing tag.</span></span> <span data-ttu-id="00f1c-206">Del markup di esempio viene fornito più avanti in questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="00f1c-206">(Example markup is provided later in this tutorial.)</span></span>

   * <span data-ttu-id="00f1c-207">Il simbolo `$` (segno di dollaro) nella riga seguente usa una [stringa interpolata](/dotnet/csharp/language-reference/keywords/interpolated-strings):</span><span class="sxs-lookup"><span data-stu-id="00f1c-207">The `$` (dollar sign) in the following line uses an [interpolated string](/dotnet/csharp/language-reference/keywords/interpolated-strings):</span></span>

   ```cshtml
   $@"<ul><li><strong>Version:</strong> {Info.Version}</li>
   ```

1. <span data-ttu-id="00f1c-208">Aggiungere il markup seguente alla visualizzazione *About.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-208">Add the following markup to the *About.cshtml* view.</span></span> <span data-ttu-id="00f1c-209">Il markup evidenziato visualizza le informazioni sul sito Web.</span><span class="sxs-lookup"><span data-stu-id="00f1c-209">The highlighted markup displays the web site information.</span></span>

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?highlight=1,4-8, 18-999)]

   > [!NOTE]
   > <span data-ttu-id="00f1c-210">Nel Razor markup mostrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="00f1c-210">In the Razor markup shown below:</span></span>
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?range=18-18)]
   >
   > <span data-ttu-id="00f1c-211">Razor sa `info` che l'attributo è una classe, non una stringa, e si vuole scrivere il codice C#.</span><span class="sxs-lookup"><span data-stu-id="00f1c-211">Razor knows the `info` attribute is a class, not a string, and you want to write C# code.</span></span> <span data-ttu-id="00f1c-212">Qualsiasi attributo di helper tag non di tipo stringa deve essere scritto senza il carattere `@`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-212">Any non-string tag helper attribute should be written without the `@` character.</span></span>

1. <span data-ttu-id="00f1c-213">Eseguire l'app e passare alla visualizzazione About per visualizzare le informazioni sul sito Web.</span><span class="sxs-lookup"><span data-stu-id="00f1c-213">Run the app, and navigate to the About view to see the web site information.</span></span>

   > [!NOTE]
   > <span data-ttu-id="00f1c-214">È possibile usare il markup seguente con un tag di chiusura e rimuovere la riga con `TagMode.StartTagAndEndTag` nell'helper tag:</span><span class="sxs-lookup"><span data-stu-id="00f1c-214">You can use the following markup with a closing tag and remove the line with `TagMode.StartTagAndEndTag` in the tag helper:</span></span>
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutNotSelfClosing.cshtml?range=20-21)]

## <a name="condition-tag-helper"></a><span data-ttu-id="00f1c-215">Helper tag di condizione</span><span class="sxs-lookup"><span data-stu-id="00f1c-215">Condition Tag Helper</span></span>

<span data-ttu-id="00f1c-216">L'helper tag di condizione esegue il rendering dell'output quando riceve un valore true.</span><span class="sxs-lookup"><span data-stu-id="00f1c-216">The condition tag helper renders output when passed a true value.</span></span>

1. <span data-ttu-id="00f1c-217">Aggiungere la classe `ConditionTagHelper` seguente alla cartella *TagHelpers*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-217">Add the following `ConditionTagHelper` class to the *TagHelpers* folder.</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/ConditionTagHelper.cs)]

1. <span data-ttu-id="00f1c-218">Sostituire il contenuto del file *Views/Home/Index.cshtml* con il markup seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-218">Replace the contents of the *Views/Home/Index.cshtml* file with the following markup:</span></span>

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Index.cshtml)]

1. <span data-ttu-id="00f1c-219">Sostituire il metodo `Index` nel controller `Home` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-219">Replace the `Index` method in the `Home` controller with the following code:</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Controllers/HomeController.cs?range=9-18)]

1. <span data-ttu-id="00f1c-220">Eseguire l'app e passare alla home page.</span><span class="sxs-lookup"><span data-stu-id="00f1c-220">Run the app and browse to the home page.</span></span> <span data-ttu-id="00f1c-221">Non verrà eseguito il rendering del markup in `div` condizionale.</span><span class="sxs-lookup"><span data-stu-id="00f1c-221">The markup in the conditional `div` won't be rendered.</span></span> <span data-ttu-id="00f1c-222">Accodare la stringa di query `?approved=true` all'URL (ad esempio, `http://localhost:1235/Home/Index?approved=true`).</span><span class="sxs-lookup"><span data-stu-id="00f1c-222">Append the query string `?approved=true` to the URL (for example, `http://localhost:1235/Home/Index?approved=true`).</span></span> <span data-ttu-id="00f1c-223">`approved`è impostato su true e il markup condizionale verrà visualizzato.</span><span class="sxs-lookup"><span data-stu-id="00f1c-223">`approved` is set to true and the conditional markup will be displayed.</span></span>

> [!NOTE]
> <span data-ttu-id="00f1c-224">Usare l'operatore [nameof](/dotnet/csharp/language-reference/keywords/nameof) per specificare l'attributo di destinazione, anziché specificare una stringa, come è stato fatto per l'helper tag bold:</span><span class="sxs-lookup"><span data-stu-id="00f1c-224">Use the [nameof](/dotnet/csharp/language-reference/keywords/nameof) operator to specify the attribute to target rather than specifying a string as you did with the bold tag helper:</span></span>
>
> [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zConditionTagHelperCopy.cs?highlight=1,2,5&range=5-18)]
>
> <span data-ttu-id="00f1c-225">L'operatore [nameof](/dotnet/csharp/language-reference/keywords/nameof) proteggerà il codice nel caso venisse effettuato il refactoring (potrebbe essere necessario modificare il nome in `RedCondition`).</span><span class="sxs-lookup"><span data-stu-id="00f1c-225">The [nameof](/dotnet/csharp/language-reference/keywords/nameof) operator will protect the code should it ever be refactored (we might want to change the name to `RedCondition`).</span></span>

### <a name="avoid-tag-helper-conflicts"></a><span data-ttu-id="00f1c-226">Evitare conflitti di helper tag</span><span class="sxs-lookup"><span data-stu-id="00f1c-226">Avoid Tag Helper conflicts</span></span>

<span data-ttu-id="00f1c-227">In questa sezione verrà scritta una coppia di helper tag a collegamento automatico.</span><span class="sxs-lookup"><span data-stu-id="00f1c-227">In this section, you write a pair of auto-linking tag helpers.</span></span> <span data-ttu-id="00f1c-228">Il primo sostituirà il markup contenente un URL che inizia con HTTP con un tag di ancoraggio HTML contenente lo stesso URL (e che quindi genera un collegamento all'URL).</span><span class="sxs-lookup"><span data-stu-id="00f1c-228">The first will replace markup containing a URL starting with HTTP to an HTML anchor tag containing the same URL (and thus yielding a link to the URL).</span></span> <span data-ttu-id="00f1c-229">Il secondo eseguirà la stessa operazione per un URL che inizia con WWW.</span><span class="sxs-lookup"><span data-stu-id="00f1c-229">The second will do the same for a URL starting with WWW.</span></span>

<span data-ttu-id="00f1c-230">Poiché questi due helper sono strettamente correlati e in futuro potrebbero essere sottoposti a refactoring, verranno mantenuti nello stesso file.</span><span class="sxs-lookup"><span data-stu-id="00f1c-230">Because these two helpers are closely related and you may refactor them in the future, we'll keep them in the same file.</span></span>

1. <span data-ttu-id="00f1c-231">Aggiungere la classe `AutoLinkerHttpTagHelper` seguente alla cartella *TagHelpers*.</span><span class="sxs-lookup"><span data-stu-id="00f1c-231">Add the following `AutoLinkerHttpTagHelper` class to the *TagHelpers* folder.</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=7-19)]

   >[!NOTE]
   ><span data-ttu-id="00f1c-232">La classe `AutoLinkerHttpTagHelper` ha come destinazioni elementi `p` e usa [Regex](/dotnet/standard/base-types/regular-expression-language-quick-reference) per creare l'ancoraggio.</span><span class="sxs-lookup"><span data-stu-id="00f1c-232">The `AutoLinkerHttpTagHelper` class targets `p` elements and uses [Regex](/dotnet/standard/base-types/regular-expression-language-quick-reference) to create the anchor.</span></span>

1. <span data-ttu-id="00f1c-233">Aggiungere il markup seguente alla fine del file *Views/Home/Contact.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="00f1c-233">Add the following markup to the end of the *Views/Home/Contact.cshtml* file:</span></span>

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=19)]

1. <span data-ttu-id="00f1c-234">Eseguire l'app e verificare che l'helper tag esegua correttamente il rendering dell'ancoraggio.</span><span class="sxs-lookup"><span data-stu-id="00f1c-234">Run the app and verify that the tag helper renders the anchor correctly.</span></span>

1. <span data-ttu-id="00f1c-235">Aggiornare la classe `AutoLinker` per includere `AutoLinkerWwwTagHelper` che convertirà il testo www in un tag di ancoraggio contenente anch'esso il testo www originale.</span><span class="sxs-lookup"><span data-stu-id="00f1c-235">Update the `AutoLinker` class to include the `AutoLinkerWwwTagHelper` which will convert www text to an anchor tag that also contains the original www text.</span></span> <span data-ttu-id="00f1c-236">Il codice aggiornato è evidenziato di seguito:</span><span class="sxs-lookup"><span data-stu-id="00f1c-236">The updated code is highlighted below:</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?highlight=15-34&range=7-34)]

1. <span data-ttu-id="00f1c-237">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="00f1c-237">Run the app.</span></span> <span data-ttu-id="00f1c-238">Si noti che il testo www è visualizzato come collegamento, ma il testo HTTP non lo è.</span><span class="sxs-lookup"><span data-stu-id="00f1c-238">Notice the www text is rendered as a link but the HTTP text isn't.</span></span> <span data-ttu-id="00f1c-239">Se si inserisce un punto di interruzione in entrambe le classi, è possibile notare che la classe helper tag HTTP viene eseguita per prima.</span><span class="sxs-lookup"><span data-stu-id="00f1c-239">If you put a break point in both classes, you can see that the HTTP tag helper class runs first.</span></span> <span data-ttu-id="00f1c-240">Il problema è che l'output dell'helper tag viene memorizzato nella cache. Quando viene eseguito l'helper tag WWW, quest'ultimo sovrascrive l'output dell'helper tag HTTP memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="00f1c-240">The problem is that the tag helper output is cached, and when the WWW tag helper is run, it overwrites the cached output from the HTTP tag helper.</span></span> <span data-ttu-id="00f1c-241">Più avanti in questa esercitazione verrà illustrato come controllare l'ordine di esecuzione degli helper tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-241">Later in the tutorial we'll see how to control the order that tag helpers run in.</span></span> <span data-ttu-id="00f1c-242">È possibile correggere il problema con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-242">We'll fix the code with the following:</span></span>

   [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10,21,22,26&range=8-37)]

   > [!NOTE]
   > <span data-ttu-id="00f1c-243">Nella prima edizione degli helper tag a collegamento automatico, per ottenere il contenuto della destinazione è stato usato il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-243">In the first edition of the auto-linking tag helpers, you got the content of the target with the following code:</span></span>
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=12)]
   >
   > <span data-ttu-id="00f1c-244">In altre parole, si effettua una chiamata a `GetChildContentAsync` usando `TagHelperOutput` passato al metodo `ProcessAsync`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-244">That is, you call `GetChildContentAsync` using the `TagHelperOutput` passed into the `ProcessAsync` method.</span></span> <span data-ttu-id="00f1c-245">Come affermato in precedenza, poiché l'output viene memorizzato nella cache, l'ultimo helper eseguito prevale.</span><span class="sxs-lookup"><span data-stu-id="00f1c-245">As mentioned previously, because the output is cached, the last tag helper to run wins.</span></span> <span data-ttu-id="00f1c-246">Il problema è stato risolto con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="00f1c-246">You fixed that problem with the following code:</span></span>
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?range=34-35)]
   >
   > <span data-ttu-id="00f1c-247">Il codice precedente controlla se il contenuto è stato modificato e, in caso affermativo, ottiene il contenuto dal buffer di output.</span><span class="sxs-lookup"><span data-stu-id="00f1c-247">The code above checks to see if the content has been modified, and if it has, it gets the content from the output buffer.</span></span>

1. <span data-ttu-id="00f1c-248">Eseguire l'app e verificare che i due collegamenti funzionino come previsto.</span><span class="sxs-lookup"><span data-stu-id="00f1c-248">Run the app and verify that the two links work as expected.</span></span> <span data-ttu-id="00f1c-249">Può sembrare che l'helper tag a collegamento automatico sia corretto e completo, ma presenta un problema insidioso.</span><span class="sxs-lookup"><span data-stu-id="00f1c-249">While it might appear our auto linker tag helper is correct and complete, it has a subtle problem.</span></span> <span data-ttu-id="00f1c-250">Se l'helper tag WWW viene eseguito per primo, i collegamenti www non sono corretti.</span><span class="sxs-lookup"><span data-stu-id="00f1c-250">If the WWW tag helper runs first, the www links won't be correct.</span></span> <span data-ttu-id="00f1c-251">Aggiornare il codice aggiungendo l'overload `Order` per controllare l'ordine in cui viene eseguito il tag.</span><span class="sxs-lookup"><span data-stu-id="00f1c-251">Update the code by adding the `Order` overload to control the order that the tag runs in.</span></span> <span data-ttu-id="00f1c-252">La proprietà `Order` determina l'ordine di esecuzione rispetto agli altri helper tag che hanno come destinazione lo stesso elemento.</span><span class="sxs-lookup"><span data-stu-id="00f1c-252">The `Order` property determines the execution order relative to other tag helpers targeting the same element.</span></span> <span data-ttu-id="00f1c-253">Il valore dell'ordine predefinito è zero e le istanze con valori inferiori vengono eseguite per prime.</span><span class="sxs-lookup"><span data-stu-id="00f1c-253">The default order value is zero and instances with lower values are executed first.</span></span>

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?highlight=5,6,7,8&range=8-15)]

   <span data-ttu-id="00f1c-254">Il codice precedente assicura che l'helper tag HTTP venga eseguito prima dell'helper tag WWW.</span><span class="sxs-lookup"><span data-stu-id="00f1c-254">The preceding code guarantees that the HTTP tag helper runs before the WWW tag helper.</span></span> <span data-ttu-id="00f1c-255">Modificare `Order` in `MaxValue` e verificare che il markup generato per il tag WWW non è corretto.</span><span class="sxs-lookup"><span data-stu-id="00f1c-255">Change `Order` to `MaxValue` and verify that the markup generated for the WWW tag is incorrect.</span></span>

## <a name="inspect-and-retrieve-child-content"></a><span data-ttu-id="00f1c-256">Controllare e recuperare il contenuto figlio</span><span class="sxs-lookup"><span data-stu-id="00f1c-256">Inspect and retrieve child content</span></span>

<span data-ttu-id="00f1c-257">Gli helper tag mettono a disposizione diverse proprietà per recuperare il contenuto.</span><span class="sxs-lookup"><span data-stu-id="00f1c-257">The tag helpers provide several properties to retrieve content.</span></span>

* <span data-ttu-id="00f1c-258">Il risultato di `GetChildContentAsync` può essere accodato a `output.Content`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-258">The result of `GetChildContentAsync` can be appended to `output.Content`.</span></span>
* <span data-ttu-id="00f1c-259">È possibile controllare il risultato di `GetChildContentAsync` con `GetContent`.</span><span class="sxs-lookup"><span data-stu-id="00f1c-259">You can inspect the result of `GetChildContentAsync` with `GetContent`.</span></span>
* <span data-ttu-id="00f1c-260">Se si modifica `output.Content`, il corpo di TagHelper viene eseguito, o ne viene eseguito il rendering, solo se si chiama `GetChildContentAsync`, come nell'esempio di helper tag a collegamento automatico:</span><span class="sxs-lookup"><span data-stu-id="00f1c-260">If you modify `output.Content`, the TagHelper body won't be executed or rendered unless you call `GetChildContentAsync` as in our auto-linker sample:</span></span>

[!code-csharp[](../../views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10&range=8-21)]

* <span data-ttu-id="00f1c-261">Più chiamate al metodo `GetChildContentAsync` restituiscono lo stesso valore e rieseguono nuovamente il corpo di `TagHelper` solo se viene passato un parametro false per indicare di non usare il risultato memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="00f1c-261">Multiple calls to `GetChildContentAsync` returns the same value and doesn't re-execute the `TagHelper` body unless you pass in a false parameter indicating not to use the cached result.</span></span>

## <a name="load-minified-partial-view-taghelper"></a><span data-ttu-id="00f1c-262">TagHelper per caricare visualizzazioni parziali minimizzate</span><span class="sxs-lookup"><span data-stu-id="00f1c-262">Load minified partial view TagHelper</span></span>

<span data-ttu-id="00f1c-263">Negli ambienti di produzione le prestazioni possono essere migliorate caricando visualizzazioni parziali minimizzate.</span><span class="sxs-lookup"><span data-stu-id="00f1c-263">In production environments, performance can be improved by loading minified partial views.</span></span> <span data-ttu-id="00f1c-264">Per sfruttare i vantaggi della visualizzazione parziale minimizzata nell'ambiente di produzione:</span><span class="sxs-lookup"><span data-stu-id="00f1c-264">To take advantage of minified partial view in production:</span></span>

* <span data-ttu-id="00f1c-265">Creare/configurare un processo di pre-compilazione che minimizza visualizzazioni parziali.</span><span class="sxs-lookup"><span data-stu-id="00f1c-265">Create/set up a pre-build process that minifies partial views.</span></span>
* <span data-ttu-id="00f1c-266">Usare il codice seguente per caricare le visualizzazioni parziali minimizzate in ambienti non di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="00f1c-266">Use the following code to load  minified partial views in non-development environments.</span></span>

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/MinifiedVersionTagHelper.cs?name=snippet)]
