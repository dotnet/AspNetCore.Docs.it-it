---
title: Creare un client e un server gRPC .NET Core in ASP.NET Core
author: juntaoluo
description: Questa esercitazione illustra come creare un servizio gRPC e un client gRPC in ASP.NET Core. Informazioni su come creare un progetto di servizio gRPC, modificare un file con estensione proto e aggiungere una chiamata con flusso bidirezionale.
ms.author: johluo
ms.date: 10/23/2020
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
uid: tutorials/grpc/grpc-start
ms.openlocfilehash: fed25eeacf57504810d41fcf002dcaa9927a21af
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588601"
---
# <a name="tutorial-create-a-grpc-client-and-server-in-aspnet-core"></a>Esercitazione: creare un client e un server gRPC in ASP.NET Core

A cura di [John Luo](https://github.com/juntaoluo)

Questa esercitazione illustra come creare un client [gRPC](https://grpc.io/docs/guides/) .NET Core e un server gRPC ASP.NET Core.

Al termine, si otterrà un client gRPC che comunica con il servizio Greeter gRPC.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/grpc/grpc-start/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).

In questa esercitazione:

> [!div class="checklist"]
> * Creare un server gRPC.
> * Creare un client gRPC.
> * Testare il servizio client gRPC con il servizio Greeter gRPC.

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* [Visual Studio per Mac versione 8,7 o successiva](/visualstudio/releasenotes/vs2019-mac-relnotes)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]
---

## <a name="create-a-grpc-service"></a>Creare un servizio gRPC

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Avviare Visual Studio e selezionare **Crea un nuovo progetto**. In alternativa, dal menu **File** di Visual Studio scegliere **Nuovo** > **Progetto**.
* Nella finestra di dialogo **Crea un nuovo progetto** selezionare **servizio GRPC** e selezionare **Avanti**:

  ![Crea una finestra di dialogo nuovo progetto in Visual Studio](~/tutorials/grpc/grpc-start/static/cnp.png)

* Denominare il progetto **GrpcGreeter**. È importante denominare il progetto *GrpcGreeter* in modo che gli spazi dei nomi corrispondano quando si copia e incolla il codice.
* Selezionare **Crea**.
* Nella finestra di dialogo **Crea un nuovo servizio gRPC**:
  * È selezionato il modello **Servizio gRPC**.
  * Selezionare **Crea**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Modificare le directory ( `cd` ) in una cartella per il progetto.
* Eseguire i comandi seguenti:

  ```dotnetcli
  dotnet new grpc -o GrpcGreeter
  code -r GrpcGreeter
  ```

  * Il comando `dotnet new` crea un nuovo servizio gRPC nella cartella *GrpcGreeter*.
  * Il comando `code` apre la cartella *GrpcGreeter* in una nuova istanza di Visual Studio Code.

  Viene visualizzata una finestra di dialogo con le **risorse necessarie per la compilazione e il debug non è presente in ' GrpcGreeter '. Aggiungerli?**
* Selezionare **Sì**.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Avviare Visual Studio per Mac e selezionare **Crea un nuovo progetto**. In alternativa, dal menu **File** di Visual Studio scegliere **Nuovo** > **Progetto**.
* Nella finestra di dialogo **Crea un nuovo progetto** selezionare **Web e**  >  **app** console  >  **servizio gRPC** e selezionare **Avanti**:

  ![Crea una finestra di dialogo nuovo progetto in macOS](~/tutorials/grpc/grpc-start/static/cnp-mac.png)

* Selezionare **.NET Core 3,1** per il Framework di destinazione e selezionare **Avanti**.
* Denominare il progetto **GrpcGreeter**. È importante denominare il progetto *GrpcGreeter* in modo che gli spazi dei nomi corrispondano quando si copia e incolla il codice.
* Selezionare **Crea**.
---

### <a name="run-the-service"></a>Eseguire il servizio

  [!INCLUDE[](~/includes/run-the-app.md)]

I log indicano che il servizio è in ascolto su `https://localhost:5001`.

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

> [!NOTE]
> Il modello gRPC è configurato per l'uso di [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246). i client gRPC devono usare HTTPS per chiamare il server.
>
> macOS non supporta ASP.NET Core gRPC con TLS. Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva. Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).

### <a name="examine-the-project-files"></a>Esaminare i file di progetto

File di progetto *GrpcGreeter*:

* *greet. proto*: il file *Protos/greet. proto* definisce il `Greeter` gRPC e viene usato per generare gli asset del server gRPC. Per altre informazioni, vedere [Introduzione a gRPC](xref:grpc/index).
* Cartella dei *Servizi* : contiene l'implementazione del `Greeter` servizio.
* *appSettings.json*: contiene i dati di configurazione, ad esempio il protocollo usato da gheppio. Per altre informazioni, vedere <xref:fundamentals/configuration/index>.
* *Program.cs*: contiene il punto di ingresso per il servizio gRPC. Per altre informazioni, vedere <xref:fundamentals/host/generic-host>.
* *Startup.cs*: contiene il codice che configura il comportamento dell'app. Per altre informazioni, vedere [Avvio dell'app](xref:fundamentals/startup).

## <a name="create-the-grpc-client-in-a-net-console-app"></a>Creare il client gRPC in un'app console .NET

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Aprire una seconda istanza di Visual Studio e selezionare **Crea un nuovo progetto**.
* Nella finestra di dialogo **Crea un nuovo progetto** selezionare **App console (.NET Core)** e selezionare **Avanti**.
* Nella casella di testo **nome progetto** immettere **GrpcGreeterClient** e selezionare **Crea**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Modificare le directory ( `cd` ) in una cartella per il progetto.
* Eseguire i comandi seguenti:

  ```dotnetcli
  dotnet new console -o GrpcGreeterClient
  code -r GrpcGreeterClient
  ```

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Seguire le istruzioni riportate in [Creazione di una soluzione .NET Core completa in macOS con Visual Studio per Mac](/dotnet/core/tutorials/using-on-mac-vs-full-solution) per creare un'app console con il nome *GrpcGreeterClient*.

---

### <a name="add-required-packages"></a>Aggiungere i pacchetti necessari

Per il progetto client gRPC sono richiesti i pacchetti seguenti:

* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client), che contiene il client .NET Core.
* [Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/), che contiene le API per i messaggi protobuf per C#.
* [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/), che contiene il supporto di strumenti C# per i file protobuf. Il pacchetto di strumenti non è necessario in fase di esecuzione, quindi la dipendenza è contrassegnata come `PrivateAssets="All"`.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Installare i pacchetti tramite la Console di gestione pacchetti (PMC) o Gestisci pacchetti NuGet.

#### <a name="pmc-option-to-install-packages"></a>Opzione con la console di Gestione pacchetti per installare i pacchetti

* In Visual Studio selezionare **strumenti**  >  **Gestione pacchetti NuGet**  >  **console di gestione pacchetti**
* Dalla finestra **Console di Gestione pacchetti** eseguire `cd GrpcGreeterClient` per passare alla cartella contenente i file *GrpcGreeterClient.csproj*.
* Eseguire i comandi seguenti:

  ```powershell
  Install-Package Grpc.Net.Client
  Install-Package Google.Protobuf
  Install-Package Grpc.Tools
  ```

#### <a name="manage-nuget-packages-option-to-install-packages"></a>Opzione Gestisci pacchetti NuGet per installare i pacchetti

* Fare clic con il pulsante destro del mouse sul progetto in **Esplora soluzioni**  >  **Gestisci pacchetti NuGet**.
* Selezionare la scheda **Sfoglia**.
* Immettere **Grpc.Net.Client** nella casella di ricerca.
* Selezionare il pacchetto **Grpc.Net.Client** dalla scheda **Sfoglia** e fare clic su **Installa**.
* Ripetere la procedura per `Google.Protobuf` e `Grpc.Tools`.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Eseguire i comandi seguenti dal **terminale integrato**:

```dotnetcli
dotnet add GrpcGreeterClient.csproj package Grpc.Net.Client
dotnet add GrpcGreeterClient.csproj package Google.Protobuf
dotnet add GrpcGreeterClient.csproj package Grpc.Tools
```

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* Fare clic con il pulsante destro del mouse su progetto **GrpcGreeterClient** nel **riquadro della soluzione** e scegliere **Gestisci pacchetti NuGet**.
* Immettere **Grpc.Net.Client** nella casella di ricerca.
* Selezionare il pacchetto **Grpc .NET. client** dal riquadro dei risultati e selezionare **Aggiungi pacchetto**.
* Selezionare il pulsante **accetta** nella finestra di dialogo **Accept License** .
* Ripetere la procedura per `Google.Protobuf` e `Grpc.Tools`.

---

### <a name="add-greetproto"></a>Aggiungere greet.proto

* Creare una cartella *Protos* nel progetto client gRPC.
* Copiare il file *Protos\greet.proto* dal servizio Greeter gRPC al progetto client gRPC.
* Aggiornare lo spazio dei nomi all'interno del `greet.proto` file allo spazio dei nomi del progetto:

  ```
  option csharp_namespace = "GrpcGreeterClient";
  ```

* Modificare il file di progetto *GrpcGreeterClient.csproj*:

  # <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

  Fare clic con il pulsante destro del mouse sul progetto e scegliere **Modifica file di progetto**.

  # <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

  Selezionare il file *GrpcGreeterClient.csproj*.

  # <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

  Fare clic con il pulsante destro del mouse sul progetto e scegliere **Modifica file di progetto**.

  ---

* Aggiungere un gruppo di elementi con un elemento `<Protobuf>` che fa riferimento al file *greet.proto*:

  ```xml
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
  </ItemGroup>
  ```

### <a name="create-the-greeter-client"></a>Creare il client Greeter

Compilare il progetto client per creare i tipi nello `GrpcGreeter` spazio dei nomi. I tipi `GrpcGreeter` vengono generati automaticamente dal processo di compilazione.

Aggiornare il file *Program.cs* del client gRPC con il codice seguente:

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet2)]

*Program.cs* contiene la logica e il punto di ingresso per il client gRPC.

Il client Greeter viene creato come descritto di seguito:

* Creare un'istanza di `GrpcChannel` contenente le informazioni per creare la connessione al servizio gRPC.
* Usare `GrpcChannel` per costruire il client Greeter:

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=3-5)]

Il client Greeter chiama il metodo `SayHello` asincrono. Viene visualizzato il risultato della chiamata `SayHello`:

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=6-8)]

## <a name="test-the-grpc-client-with-the-grpc-greeter-service"></a>Testare il client gRPC con il servizio Greeter gRPC

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Nel servizio Greeter premere `Ctrl+F5` per avviare il server senza il debugger.
* Nel progetto `GrpcGreeterClient` premere `Ctrl+F5` per avviare il client senza il debugger.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Avviare il servizio Greeter.
* Avviare il client.


# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

* A causa del [problema http/2 TLS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)indicato in precedenza nella soluzione MacOS, è necessario aggiornare l'indirizzo del canale nel client a " http://localhost:5000 ". Aggiornare la riga 13 di **GrpcGreeterClient/Program. cs** da leggere:
  ```csharp
  using var channel = GrpcChannel.ForAddress("http://localhost:5000");
  ``` 
* Avviare il servizio Greeter.
* Avviare il client.

---

Il client invia un messaggio di saluto al servizio con un messaggio contenente il nome *GreeterClient*. Il servizio invia il messaggio "Hello GreeterClient" come risposta. La risposta "Hello GreeterClient" viene visualizzata al prompt dei comandi:

```console
Greeting: Hello GreeterClient
Press any key to exit...
```

Il servizio gRPC registra i dettagli della chiamata con esito positivo nei log scritti al prompt dei comandi:

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\GH\aspnet\docs\4\Docs\aspnetcore\tutorials\grpc\grpc-start\sample\GrpcGreeter
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST https://localhost:5001/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 78.32260000000001ms 200 application/grpc
```

> [!NOTE]
> Il codice in questo articolo richiede il certificato di sviluppo HTTPS ASP.NET Core per proteggere il servizio gRPC. Se il client gRPC .NET ha esito negativo con il messaggio `The remote certificate is invalid according to the validation procedure.` o `The SSL connection could not be established.` , il certificato di sviluppo non è attendibile. Per risolvere questo problema, vedere [chiamare un servizio gRPC con un certificato non attendibile/non valido](xref:grpc/troubleshoot#call-a-grpc-service-with-an-untrustedinvalid-certificate).

[!INCLUDE[](~/includes/gRPCazure.md)]

### <a name="next-steps"></a>Passaggi successivi

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/migration>
