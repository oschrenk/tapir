# tapir, or Typed API descRiptions

With tapir you can describe HTTP API endpoints as immutable Scala values. Each endpoint can contain a number of 
input parameters, error-output parameters, and normal-output parameters. An endpoint specification can be 
interpreted as:

* a server, given the "business logic": a function, which computes output parameters based on input parameters. 
  Currently supported: 
  * [Akka HTTP](server/akkahttp.md) `Route`s/`Directive`s.
  * [Http4s](server/http4s.md) `HttpRoutes[F]`
  * [Finatra](server/finatra.md) `http.Controller`
* a client, which is a function from input parameters to output parameters. Currently supported: [sttp](sttp.md).
* documentation. Currently supported: [OpenAPI](openapi.md).

Tapir is licensed under Apache2, the source code is [available on GitHub](https://github.com/softwaremill/tapir).

Depending on how you prefer to explore the library, take a look at one of the [examples](examples.md) or read on
for a more detailed description of how tapir works!

## Code teaser

```scala
import sttp.tapir._
import sttp.tapir.json.circe._
import io.circe.generic.auto._

type Limit = Int
type AuthToken = String
case class BooksFromYear(genre: String, year: Int)
case class Book(title: String)


// Define an endpoint

val booksListing: Endpoint[(BooksFromYear, Limit, AuthToken), String, List[Book], Any] = 
  endpoint
    .get
    .in(("books" / path[String]("genre") / path[Int]("year")).mapTo(BooksFromYear))
    .in(query[Limit]("limit").description("Maximum number of books to retrieve"))
    .in(header[AuthToken]("X-Auth-Token"))
    .errorOut(stringBody)
    .out(jsonBody[List[Book]])


// Generate OpenAPI documentation

import sttp.tapir.docs.openapi._
import sttp.tapir.openapi.circe.yaml._

val docs = booksListing.toOpenAPI("My Bookshop", "1.0")
println(docs.toYaml)


// Convert to akka-http Route

import sttp.tapir.server.akkahttp._
import akka.http.scaladsl.server.Route
import scala.concurrent.Future

def bookListingLogic(bfy: BooksFromYear,
                     limit: Limit,
                     at: AuthToken): Future[Either[String, List[Book]]] =
  Future.successful(Right(List(Book("The Sorrows of Young Werther"))))
val booksListingRoute: Route = booksListing.toRoute((bookListingLogic _).tupled)


// Convert to sttp Request

import sttp.tapir.client.sttp._
import sttp.client3._

val booksListingRequest: Request[DecodeResult[Either[String, List[Book]]], Any] = booksListing
  .toSttpRequest(uri"http://localhost:8080")
  .apply((BooksFromYear("SF", 2016), 20, "xyz-abc-123"))
```

## Other sttp projects

sttp is a family of Scala HTTP-related projects, and currently includes:

* [sttp client](https://github.com/softwaremill/sttp): the Scala HTTP client you always wanted!
* sttp tapir: this project
* [sttp model](https://github.com/softwaremill/sttp-model): simple HTTP model classes (used by client & tapir)

## Sponsors

Development and maintenance of sttp tapir is sponsored by [SoftwareMill](https://softwaremill.com), a software development and consulting company. We help clients scale their business through software. Our areas of expertise include backends, distributed systems, blockchain, machine learning and data analytics.

[![](https://files.softwaremill.com/logo/logo.png "SoftwareMill")](https://softwaremill.com)

# Table of contents

```eval_rst
.. toctree::
   :maxdepth: 2
   :caption: Getting started

   quickstart
   examples
   goals

.. toctree::
   :maxdepth: 2
   :caption: Endpoints

   endpoint/basics
   endpoint/ios
   endpoint/statuscodes
   endpoint/codecs
   endpoint/customtypes
   endpoint/validation
   endpoint/json
   endpoint/forms
   endpoint/auth
   endpoint/integrations
   endpoint/zio

.. toctree::
   :maxdepth: 2
   :caption: Server interpreters

   server/akkahttp
   server/http4s
   server/finatra
   server/play
   server/vertx
   server/options
   server/logic
   server/errors
   server/debugging

.. toctree::
   :maxdepth: 2
   :caption: Client interpreters
   
   sttp

.. toctree::
   :maxdepth: 2
   :caption: Documentation interpreters

   openapi

.. toctree::
   :maxdepth: 2
   :caption: Testing

   testing

.. toctree::
   :maxdepth: 2
   :caption: Other subjects

   other_interpreters
   mytapir
   troubleshooting
   contributing

