---
title: BlazorIsolamento ASP.NET Core CSS
author: daveabrock
description: Scopri in che modo l'isolamento CSS ti permette di definire l'ambito di CSS per i tuoi componenti, semplificando i tuoi CSS ed evitando conflitti con altri componenti o librerie.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/20/2020
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
uid: blazor/components/css-isolation
ms.openlocfilehash: 0748f606314963788e6733ca2ae2ca2123d839b3
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99529982"
---
# <a name="aspnet-core-blazor-css-isolation"></a>BlazorIsolamento ASP.NET Core CSS

Di [Dave Brock](https://twitter.com/daveabrock)

L'isolamento CSS semplifica l'impronta CSS di un'app impedendo le dipendenze sugli stili globali e consente di evitare conflitti di stile tra componenti e librerie.

## <a name="enable-css-isolation"></a>Abilita isolamento CSS 

Per definire stili specifici del componente, creare un `.razor.css` file corrispondente al nome del `.razor` file per il componente nella stessa cartella. Il `.razor.css` file è un *file CSS con ambito*. 

Per un `Example` componente in un `Example.razor` file, creare un file insieme al componente denominato `Example.razor.css` . Il `Example.razor.css` file deve risiedere nella stessa cartella del `Example` componente ( `Example.razor` ). Il `Example` nome di base del file **non** fa distinzione tra maiuscole e minuscole.

`Pages/Example.razor`:

```razor
@page "/example"

<h1>Scoped CSS Example</h1>
```

`Pages/Example.razor.css`:

```css
h1 { 
    color: brown;
    font-family: Tahoma, Geneva, Verdana, sans-serif;
}
```

**Gli stili definiti in `Example.razor.css` vengono applicati solo all'output del componente di cui è stato eseguito il rendering `Example` .** L'isolamento CSS viene applicato agli elementi HTML nel Razor file corrispondente. Qualsiasi `h1` dichiarazione CSS definita altrove nell'app non è in conflitto con gli `Example` stili del componente.

> [!NOTE]
> Per garantire l'isolamento dello stile quando si verifica la creazione di bundle, l'importazione di CSS nei Razor blocchi di codice non è supportata.

## <a name="css-isolation-bundling"></a>Aggregazione di isolamento CSS

L'isolamento CSS si verifica in fase di compilazione. Durante questo processo, Blazor riscrive i selettori CSS in modo che corrispondano al markup sottoposto a rendering dal componente. Questi stili CSS riscritti sono aggregati e prodotti come asset statico in `{PROJECT NAME}.styles.css` , dove il segnaposto `{PROJECT NAME}` è il nome del pacchetto o del prodotto a cui si fa riferimento.

Per impostazione predefinita, a questi file statici viene fatto riferimento dal percorso radice dell'app. Nell'app, fare riferimento al file in bundle controllando il riferimento all'interno del `<head>` tag del codice HTML generato:

```html
<link href="ProjectName.styles.css" rel="stylesheet">
```

All'interno del file in bundle, ogni componente è associato a un identificatore di ambito. Per ogni componente con stile, viene aggiunto un attributo HTML con il formato `b-<10-character-string>` . L'identificatore è univoco e diverso per ogni app. Nel componente sottoposto a rendering `Counter` Blazor Accoda un identificatore di ambito all' `h1` elemento:

```html
<h1 b-3xxtam6d07>
```

Il `ProjectName.styles.css` file usa l'identificatore di ambito per raggruppare una dichiarazione di stile con il relativo componente. Nell'esempio seguente viene fornito lo stile per l' `<h1>` elemento precedente:

```css
/* /Pages/Counter.razor.rz.scp.css */
h1[b-3xxtam6d07] {
    color: brown;
}
```

In fase di compilazione, viene creato un bundle di progetto con la convenzione `{STATIC WEB ASSETS BASE PATH}/Project.lib.scp.css` , dove il segnaposto `{STATIC WEB ASSETS BASE PATH}` è il percorso di base degli asset Web statici.

Se vengono usati altri progetti, ad esempio pacchetti NuGet o [ Razor librerie di classi](xref:blazor/components/class-libraries), il file in bundle:

* Fa riferimento agli stili utilizzando le importazioni CSS.
* Non viene pubblicato come asset Web statico dell'app che utilizza gli stili.

## <a name="child-component-support"></a>Supporto componenti figlio

Per impostazione predefinita, l'isolamento CSS si applica solo al componente associato al formato `{COMPONENT NAME}.razor.css` , in cui il segnaposto `{COMPONENT NAME}` è in genere il nome del componente. Per applicare le modifiche a un componente figlio, usare il `::deep` combinatore con tutti gli elementi discendenti nel file del componente padre `.razor.css` . Il `::deep` combinatore Seleziona elementi *discendenti* dell'identificatore di ambito generato da un elemento. 

Nell'esempio seguente viene illustrato un componente padre denominato `Parent` con un componente figlio denominato `Child` .

`Pages/Parent.razor`:

```razor
@page "/parent"

<div>
    <h1>Parent component</h1>

    <Child />
</div>
```

`Shared/Child.razor`:

```razor
<h1>Child Component</h1>
```

Aggiornare la `h1` dichiarazione in `Parent.razor.css` con il `::deep` combinatore per indicare che la `h1` dichiarazione di stile deve essere applicata al componente padre e ai relativi elementi figlio.

`Pages/Parent.razor.css`:

```css
::deep h1 { 
    color: red;
}
```

Lo `h1` stile è ora applicabile ai `Parent` `Child` componenti e senza la necessità di creare un file CSS con ambito separato per il componente figlio.

Il `::deep` combinatore funziona solo con gli elementi discendenti. Il markup seguente applica gli `h1` stili ai componenti come previsto. L'identificatore di ambito del componente padre viene applicato all' `div` elemento, in modo che il browser sappia ereditare gli stili dal componente padre.

`Pages/Parent.razor`:

```razor
<div>
    <h1>Parent</h1>

    <Child />
</div>
```

Tuttavia, escludendo l' `div` elemento viene rimossa la relazione discendente. Nell'esempio seguente lo stile **non** viene applicato al componente figlio.

`Pages/Parent.razor`:

```razor
<h1>Parent</h1>

<Child />
```

## <a name="css-preprocessor-support"></a>Supporto per il preprocessore CSS

I preprocessori CSS sono utili per migliorare lo sviluppo CSS utilizzando funzionalità quali variabili, nidificazione, moduli, mixin ed ereditarietà. Sebbene l'isolamento CSS non supporti in modo nativo i preprocessori CSS come Sass o less, l'integrazione dei preprocessori CSS è facile purché la compilazione del preprocessore venga eseguita prima che Blazor riscriva i selettori CSS durante il processo di compilazione. Usando Visual Studio, ad esempio, configurare la compilazione del preprocessore esistente come attività **prima della compilazione** in Visual Studio Task Runner Explorer.

Molti pacchetti NuGet di terze parti, ad esempio [delegate. SassBuilder](https://www.nuget.org/packages/Delegate.SassBuilder), possono compilare i file Sass/scss all'inizio del processo di compilazione prima che si verifichi l'isolamento CSS e non è necessaria alcuna configurazione aggiuntiva.

## <a name="css-isolation-configuration"></a>Configurazione dell'isolamento CSS

L'isolamento CSS è progettato per l'uso predefinito, ma fornisce la configurazione per alcuni scenari avanzati, ad esempio quando sono presenti dipendenze da strumenti o flussi di lavoro esistenti.

### <a name="customize-scope-identifier-format"></a>Personalizzare il formato dell'identificatore dell'ambito

Per impostazione predefinita, gli identificatori di ambito usano il formato `b-<10-character-string>` . Per personalizzare il formato dell'identificatore di ambito, aggiornare il file di progetto in un modello desiderato:

```xml
<ItemGroup>
  <None Update="Pages/Example.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

Nell'esempio precedente, il CSS generato per `Example.Razor.css` modifica l'identificatore di ambito da `b-<10-character-string>` a `my-custom-scope-identifier` .

Usare gli identificatori di ambito per ottenere l'ereditarietà con file CSS con ambito. Nell'esempio di file di progetto seguente un `BaseComponent.razor.css` file contiene stili comuni tra i componenti. Un `DerivedComponent.razor.css` file eredita questi stili.

```xml
<ItemGroup>
  <None Update="Pages/BaseComponent.razor.css" CssScope="my-custom-scope-identifier" />
  <None Update="Pages/DerivedComponent.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

Usare l'operatore jolly ( `*` ) per condividere gli identificatori di ambito tra più file:

```xml
<ItemGroup>
  <None Update="Pages/*.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

### <a name="change-base-path-for-static-web-assets"></a>Modificare il percorso di base per gli asset Web statici

Il `scoped.styles.css` file viene generato alla radice dell'app. Nel file di progetto, utilizzare la `StaticWebAssetBasePath` proprietà per modificare il percorso predefinito. L'esempio seguente posiziona il `scoped.styles.css` file e il resto degli asset dell'app nel `_content` percorso:

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>_content/$(PackageId)</StaticWebAssetBasePath>
</PropertyGroup>
```

### <a name="disable-automatic-bundling"></a>Disabilitare la creazione automatica di bundle

Per rifiutare esplicitamente la modalità di Blazor pubblicazione e caricamento dei file con ambito in fase di esecuzione, utilizzare la `DisableScopedCssBundling` Proprietà. Quando si usa questa proprietà, altri strumenti o processi sono responsabili dell'acquisizione dei file CSS isolati dalla `obj` Directory e della pubblicazione e del caricamento in fase di esecuzione:

```xml
<PropertyGroup>
  <DisableScopedCssBundling>true</DisableScopedCssBundling>
</PropertyGroup>
```

## <a name="razor-class-library-rcl-support"></a>Razor supporto della libreria di classi (RCL)

Quando una [ Razor libreria di classi (RCL)](xref:razor-pages/ui-class) fornisce stili isolati, l' `<link>` attributo del tag `href` punta a `{STATIC WEB ASSET BASE PATH}/{ASSEMBLY NAME}.bundle.scp.css` , dove i segnaposto sono:

* `{STATIC WEB ASSET BASE PATH}`: Percorso di base dell'asset Web statico.
* `{ASSEMBLY NAME}`: Nome dell'assembly della libreria di classi.

Nell'esempio seguente:

* Il percorso di base dell'asset Web statico è `_content/ClassLib` .
* Il nome dell'assembly della libreria di classi è `ClassLib` .

`wwwroot/index.html` ( Blazor WebAssembly ) o `Pages_Host.cshtml` (Blazor Server):

```html
<link href="_content/ClassLib/ClassLib.bundle.scp.css" rel="stylesheet">
```

Per ulteriori informazioni sulle librerie di componenti e RCL, vedere:

* <xref:razor-pages/ui-class>
* <xref:blazor/components/class-libraries>.
