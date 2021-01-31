I trailer HTTP sono simili alle intestazioni HTTP, con la differenza che vengono inviati dopo l'invio del corpo della risposta. Per IIS e HTTP.sys sono supportati solo i trailer di risposta HTTP/2.

```csharp
if (httpContext.Response.SupportsTrailers())
{
    httpContext.Response.DeclareTrailer("trailername"); 

    // Write body
    httpContext.Response.WriteAsync("Hello world");

    httpContext.Response.AppendTrailer("trailername", "TrailerValue");
}
```

Nell'esempio di codice precedente:

* `SupportsTrailers` garantisce che i trailer siano supportati per la risposta.
* `DeclareTrailer` aggiunge il nome del trailer specificato all' `Trailer` intestazione della risposta. La dichiarazione dei trailer di una risposta è facoltativa, ma consigliata. Se `DeclareTrailer` viene chiamato, deve essere prima dell'invio delle intestazioni di risposta.
* `AppendTrailer` Accoda il trailer.
