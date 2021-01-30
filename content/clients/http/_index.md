---
title: "HTTP Client"
draft: true
weight: 75
---

Mortar HTTP Client is no other than [`*http.Client`](https://golang.org/pkg/net/http/#Client).

Similar to [gRPC Client](/clients/grpc) Mortar makes `http.Client` very convenient to configure and adds [Interceptors](/clients/interceptors) capabilities.

```golang
// HTTPClientBuilder is a builder interface to build http.Client with interceptors
type HTTPClientBuilder interface {
 AddInterceptors(...HTTPClientInterceptor) HTTPClientBuilder
 WithPreconfiguredClient(*http.Client) HTTPClientBuilder
 Build() *http.Client
}

// NewHTTPClientBuilder REST HTTP builder
//
// Useful when you want to create several *http.Client with different options
type NewHTTPClientBuilder func() HTTPClientBuilder
```

As you can see above, there is a builder and a function type alias `NewHTTPClientBuilder`.
Using `NewHTTPClientBuilder` it's possible to create different **instances** of `http.Client` which have a common set of Interceptors,
but can be individually configured with additional ones.
It's very similar to what we have with [gRPC](/clients/grpc/#customizing-grpc-connection).

```golang
// HTTPClientBuilder creates an injectable http.Client builder that can be predefined with Interceptors
//
// This function returns a closure that will always create a new builder. That way every usage can add different
// interceptors without influencing others
func HTTPClientBuilder(deps httpClientBuilderDeps) clientInt.NewHTTPClientBuilder {
 return func() clientInt.HTTPClientBuilder {
  return client.HTTPClientBuilder().AddInterceptors(deps.Interceptors...)
 }
}
```

{{%notice%}}
The following code snippet is [defined in Mortar](https://github.com/go-masonry/mortar/blob/master/constructors/partial/httpclient.go#L27).
`HTTPClientBuilder` function is already in use if you register `HttpClientFxOptions`.
{{%/notice%}}

Now you know how it's defined in Mortar, but how do you use it to create an `http.Client` ?

### Example

For a complete working example look [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/controllers/workshop.go#L59).

1. Register

   ```golang
   import (
       "github.com/go-masonry/mortar/providers"
       "go.uber.org/fx"
   )

   func HttpClientFxOptions() fx.Option {
       return fx.Options(
           providers.HTTPClientBuildersFxOption(), // client builders for gRPC and HTTP
       )
   }
   ```

2. Inject

    ```golang
    import "github.com/go-masonry/mortar/interfaces/http/client"
    ...
    type yourDeps struct {
        fx.In

        HTTPClientBuilder client.NewHTTPClientBuilder
    }
    ```

3. Use

   At this point it is possible to add additional Interceptors using `deps.HTTPClientBuilder()...` that will influence only this client instance.

   ```golang
   // Create a client instance
   var client *http.Client = deps.HTTPClientBuilder().Build()
   var httpReq *http.Request = makeHTTPRequest()
 
   response, err := w.client.Do(httpReq)
   ```
  
