# HttpInterceptor

Used for modifying or performing side effect on request and response.

## Overview

Transforms `HttpRequest` before sending it and `HttpResponse` after receiving it. Also do some stuff e.g logging when ``HttpInterceptor/willSend(_:)-2jpbn``, ``HttpInterceptor/didReceiveSuccess(_:)-70mp9`` or ``HttpInterceptor/didReceiveFailure(_:)-7qy0s`` occurs. 


## Creating a interceptor 

Create your interceptor by conforming to a HttpInterceptor and implementing the protocol's methods.

```swift
struct HeaderInterceptor: HttpInterceptor {}  
```

In case of modifying a request right before sending, you need to implement your desired modification in ``HttpInterceptor/modifyRequest(_:)-5h479`` method. 

As shown below we are creating `HeaderInterceptor` to add a header to a `URLRequest` before it is executed, by implementing ``HttpInterceptor/modifyRequest(_:)-5h479`` with the wanted modification.  
 
```swift
struct HeaderInterceptor: HttpInterceptor {
    func modifyRequest(_ request: URLRequest) -> URLRequest {
        var request = request
        request.setValue("en", forHTTPHeaderField: "Accept-Language")

        return request
        }
}
```

## Usage 

To use certain `HttpInterceptor` you just simply need to add it in array of interceptors which ``HttpConfiguration`` accepts on his instantiation.

```swift
let headerInterceptor = HeaderInterceptor()

let configuration = HttpConfiguration(
    baseUrl: URL(string: "https://www.example.com"), 
    interceptors: [headerInterceptor]
)

let client = HttpClientImpl(configuration: configuration)
```

With all done now all requests will be modified using provided interceptor's implementation and will have a specified header when the server received it. 

Similarly like this example you can use interceptors to modify the response  before returning it to a call site.  Or use one interceptor for modifying and using ``HttpInterceptor/willSend(_:)-2jpbn``, ``HttpInterceptor/didReceiveSuccess(_:)-70mp9`` or ``HttpInterceptor/didReceiveFailure(_:)-7qy0s`` methods to perform some additional side job you need.

### HttpInterceptor as response validator

As `HttpInterceptor` has ``HttpInterceptor/didReceiveSuccess(_:)-70mp9``method which is called when the response is received from the server.  
We can create an interceptor that implements only that method and use it for validating server responses.

```swift
struct ResponseValidator: HttpInterceptor { 
    func didReceiveSuccess(_ response: HttpResponse) {
        guard (200...299) ~= response.statusCode {
            throws NetworkingError.invalidResponse(response)
        }
    }
}
```
Now we can add validator interceptor along with other interceptors when `HttpConfiguration` is created, so upcoming responses are getting validated with defined logic.  

```swift
let headerInterceptor = HeaderInterceptor()
let validator = ResponseValidator()

let configuration = HttpConfiguration(
baseUrl: URL(string: "https://www.example.com"), 
interceptors: [headerInterceptor, validator]
)

let client = HttpClientImpl(configuration: configuration)
```
