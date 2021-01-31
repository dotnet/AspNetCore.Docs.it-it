<span data-ttu-id="196fe-101">I trailer HTTP sono simili alle intestazioni HTTP, con la differenza che vengono inviati dopo l'invio del corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="196fe-101">HTTP Trailers are similar to HTTP Headers, except they are sent after the response body is sent.</span></span> <span data-ttu-id="196fe-102">Per IIS e HTTP.sys sono supportati solo i trailer di risposta HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="196fe-102">For IIS and HTTP.sys, only HTTP/2 response trailers are supported.</span></span>

```csharp
if (httpContext.Response.SupportsTrailers())
{
    httpContext.Response.DeclareTrailer("trailername"); 

    // Write body
    httpContext.Response.WriteAsync("Hello world");

    httpContext.Response.AppendTrailer("trailername", "TrailerValue");
}
```

<span data-ttu-id="196fe-103">Nell'esempio di codice precedente:</span><span class="sxs-lookup"><span data-stu-id="196fe-103">In the preceding example code:</span></span>

* <span data-ttu-id="196fe-104">`SupportsTrailers` garantisce che i trailer siano supportati per la risposta.</span><span class="sxs-lookup"><span data-stu-id="196fe-104">`SupportsTrailers` ensures that trailers are supported for the response.</span></span>
* <span data-ttu-id="196fe-105">`DeclareTrailer` aggiunge il nome del trailer specificato all' `Trailer` intestazione della risposta.</span><span class="sxs-lookup"><span data-stu-id="196fe-105">`DeclareTrailer` adds the given trailer name to the `Trailer` response header.</span></span> <span data-ttu-id="196fe-106">La dichiarazione dei trailer di una risposta è facoltativa, ma consigliata.</span><span class="sxs-lookup"><span data-stu-id="196fe-106">Declaring a response's trailers is optional, but recommended.</span></span> <span data-ttu-id="196fe-107">Se `DeclareTrailer` viene chiamato, deve essere prima dell'invio delle intestazioni di risposta.</span><span class="sxs-lookup"><span data-stu-id="196fe-107">If `DeclareTrailer` is called, it must be before the response headers are sent.</span></span>
* <span data-ttu-id="196fe-108">`AppendTrailer` Accoda il trailer.</span><span class="sxs-lookup"><span data-stu-id="196fe-108">`AppendTrailer` appends the trailer.</span></span>
