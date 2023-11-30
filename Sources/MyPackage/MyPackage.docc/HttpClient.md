# ``HttpClient``

HttpClient module provides API to send network request and decode response. 

## Overview

`HttpClient` provides API for a way to send a request to the server and receive a response from it. 
Also provides a ``HttpConnectable`` protocol which can decode response in expected type.  

## Creating a client

For creating `HttpClient` you need to start by creating `HttpConfiguration`,  then instantiate a client using created configuration: 

```swift
let configuration = HttpConfiguration(baseUrl: URL(string: "https://www.example.com"))

let client = HttpClientImpl(configuration: configuration)
```

You can customize the client using ``HttpConfiguration`` by passing an array of ``HttpClient/HttpInterceptor`` which can modify request before sending it and response right after receiving. Example of this is given in ``HttpClient/HttpInterceptor``.


## Sending requests

For sending request, use `send(_ :progress:)` that accepts ``HttpRequest`` and ``ProgressHandler`` closure.


```swift
let httpRequest = HttpRequest.generic(route: .get("/stations"))
let response = try await client.send(httpRequest)
```

Returned response will be type of ``HttpResponse``. 

### Download data 

For downloading data you can use ``HttpClient/HttpClient/send(_:progress:)`` method as well but with different `HttpRequestKind`. 

`progress` parameter is useful for download kind of requests where closure provides `bytesWritten` and `totalBytesToDownload`  so you can use it for progress bar as data comes.

```swift
let httpRequest = HttpRequest.download(route: .get("/invoice"))
let response = try await client.send(httpRequest) { bytesWritten, totalBytesToDownload in 
// Fill your progress bar with data
myProgress: Float = (Float)totalBytesWritten / (Float) totalBytesToDownload 
viewModel.progress = myProgress
}
```

`HttpResponse` for download request will have property `location` set to URL location of downloaded content. So you can use downloaded content right away or move it to other desired location.

```swift
let response = try await client.send(downloadRequest, progress: nil)

if let url = response.location, let contents =  FileManager.default.contents(atPath: url.relativePath) {
    // Use `contents` here  or  move it at new location
    // ...
}
```

Note: `HttpResponse`'s `location` property will always be `nil` for generic request.
