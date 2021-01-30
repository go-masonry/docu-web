---
title: "Interceptors"
draft: true
weight: 75
---

Interceptors are a powerful tool that enables you to log, monitor, mutate, redirect and sometimes even fail a request or response.
The beauty of interceptors is, that to the user everything looks as a regular **Client**.

With Client Interceptors you can:

* Dump request and/or response to a Log
* Alter your requests before they sent and/or responses before they returned.
* Add Tracing Information
* Collect metrics about your remote calls.

Mortar comes with some useful client Interceptors both for gRPC and HTTP, they are mentioned [here](/middleware).

## HTTP Client Interceptor

Mortar introduces a way to add Interceptors to a default `http.Client`.

If we look at `http.Client` struct we will find a `RoundTripper` it is the Interface we exploit.

```golang
// Transport specifies the mechanism by which individual
// HTTP requests are made.
// If nil, DefaultTransport is used.
type RoundTripper interface {
    // RoundTrip executes a single HTTP transaction, returning
    // a Response for the provided Request.
    // ...
    RoundTrip(*Request) (*Response, error)
}
```

Mortar defines `HTTPHandler` and `HTTPClientInterceptor` type aliases in [`mortar/interfaces/http/client/interfaces.go`](https://github.com/go-masonry/mortar/blob/master/interfaces/http/client/interfaces.go).

They mimic their [gRPC counterparts](https://pkg.go.dev/google.golang.org/grpc#UnaryClientInterceptor).

```golang
// HTTPHandler is just an alias to http.RoundTriper.RoundTrip function
type HTTPHandler func(*http.Request) (*http.Response, error)

// HTTPClientInterceptor is a user defined function that can alter a request before it's sent
// and/or alter a response before it's returned to the caller
type HTTPClientInterceptor func(*http.Request, HTTPHandler) (*http.Response, error)
```

That's everything we need to enable HTTP Client interceptors.

### Dump

When we create an HTTP Request we use different Structs and Functions, but we don't set `Content-Length` Header for example.
[Dumping Utilities](https://golang.org/pkg/net/http/httputil/#DumpRequest) allows you to see what is actually sent and what is actually received.
Mortar comes with HTTP Client Interceptor that dumps(logs) the actual **request** and **response**.

```golang
func DumpRESTClientInterceptor(deps dumpHTTPDeps) client.HTTPClientInterceptor {
 return func(req *http.Request, handler client.HTTPHandler) (*http.Response, error) {
  reqBody, err := httputil.DumpRequestOut(req, true)
  deps.Logger.WithError(err).Debug(req.Context(), "Request:\n%s\n", reqBody)
  res, err := handler(req)
  if err == nil {
   resBody, dumpErr := httputil.DumpResponse(res, true)
   deps.Logger.WithError(dumpErr).Debug(req.Context(), "Response:\n%s\n", resBody)
  }
  return res, err
 }
}
```

## GRPC Client Interceptor

GRPC Interceptors are part of the `gRPC` [package](https://pkg.go.dev/google.golang.org/grpc#UnaryClientInterceptor) already.
Besides mentioning it Mortar have nothing to add.

## Registering

Registering your Interceptors is very similar to everything else in Mortar, using Uber-Fx option.
Once created you need to add them to their respectful group.

* HTTP Client Interceptors group is called **restClientInterceptors** and referenced by `RESTClientInterceptors` defined [here](https://pkg.go.dev/github.com/go-masonry/mortar/providers/groups).

    ```golang
    // TracerRESTClientInterceptorFxOption adds REST trace client interceptor to the graph
    func TracerRESTClientInterceptorFxOption() fx.Option {
        return fx.Provide(
            fx.Annotated{
                Group:  groups.RESTClientInterceptors,
                Target: trace.TracerRESTClientInterceptor,
            })
    }
    ```

* GRPC Unary Client Interceptors group is called **grpcUnaryClientInterceptors** and referenced by `GRPCUnaryClientInterceptors` defined [here](https://pkg.go.dev/github.com/go-masonry/mortar/providers/groups).

    ```golang
    // TracerGRPCClientInterceptorFxOption adds grpc trace client interceptor to the graph
    func TracerGRPCClientInterceptorFxOption() fx.Option {
        return fx.Provide(
            fx.Annotated{
                Group:  groups.GRPCUnaryClientInterceptors,
                Target: trace.TracerGRPCClientInterceptor,
            })
    }
    ```

You can look [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/http.go#L8) to see how the above two are used.
