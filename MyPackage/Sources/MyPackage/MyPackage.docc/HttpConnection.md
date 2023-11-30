# HttpConnection

Wrapper arround ``HttpClient`` with ability to decode response in exepected type.

## Overview

`HttpConnection` conforms to `HttpConnectable` which has one generic method ``HttpConnection/send(_:progress:)`` with parameters of ``Request`` and `ProgressHandler`.
 Generic deserialization functionality of `Request` allows us to define the type of response we expect to receive for each request that passes through it.

## Creating and sending requests

First you need to create `Request` with underlaying `HttpRequest`, with expected type of returned response. 

```swift
let httpRequest = HttpRequest.generic(route: .get("person"))

let request = Request<Person>(underlyingRequest: httpRequest)
```
Then instantiate `HttpConnection` using ``HttpClient`` and call ``HttpConnection/send(_:progress:)`` method with created `request`:

```swift 
let client = HttpClientImpl(configuration: config)

let connection = HttpConnection(client: client)

let response = try await connection.send(request)
```
Returned `response` is decoded into a `Person` object.
