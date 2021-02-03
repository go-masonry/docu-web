---
title: "Tracing"
draft: true
---

_[Distributed tracing](https://opentracing.io/docs/overview/what-is-tracing/), also called distributed request tracing, is a method used to profile and monitor applications, especially those built using a microservices architecture. Distributed tracing helps pinpoint where failures occur and what causes poor performance._

{{<mermaid>}}
graph LR
    A(A)
    B(B)
    C(C)
    D(D)
    E(E)
    F(F)
    A -->|0.3ms| B
    A --> C
    B --> E
    C -->|1s| D
    C --> F
    E -->|0.5s| D
{{< /mermaid >}}

Mortar has you covered, but we haven't reinvented a wheel.
Instead of defining a new Interface, you use a standard `opentracing.Tracer` Interface.
It's defined [here](github.com/opentracing/opentracing-go).

```golang
type Tracer interface {
    StartSpan(operationName string, opts ...StartSpanOption) Span
    Inject(sm SpanContext, format interface{}, carrier interface{}) error
    Extract(format interface{}, carrier interface{}) (SpanContext, error)
}
```

## Usage

It's best to `StartSpan` in a most outer layer of your Application.
Since you will probably build a gRPC web service, **gRPC ServerInterceptor** is that spot.

Mortar have everything predefined [already](#predefined-interceptors).

## Predefined Interceptors

Most likely you will not have to add anything, but use [these predefined Interceptors](https://pkg.go.dev/github.com/go-masonry/mortar/middleware/interceptors/trace).

### Server

If you examine [grpc.UnaryServerInterceptor](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/trace/server.go#L13)
you can see that this Interceptor calls `opentracing.StartSpanFromContextWithTracer` which _starts and returns a span with `operationName` using a span found within the context as a ChildOfRef.
If that doesn't exist it creates a root span.
It also returns a context.Context object built around the returned span._

That's great for **gRPC communication**, since everything will be found inside a `Context`.
But what if your REST API is called ?
Given that REST API is a reverse-proxy to your gRPC API, unless properly treated, the above code will create a new Server Span even if there is Tracing Information (usually found within HTTP Headers).
To fix that we need to **inject** any tracing information into our Context.

{{%notice%}}Unless you really have to, it's best to handle everything on the gRPC layer.{{%/notice%}}

If your REST API is implemented by gRPC-Gateway, you should use [this](https://pkg.go.dev/github.com/go-masonry/mortar/providers#GRPCGatewayMetadataTraceCarrierFxOption) Metadata Trace Carrier.

```golang
func HttpServerFxOptions() fx.Option {
 return fx.Options(
  providers.GRPCTracingUnaryServerInterceptorFxOption(),
  providers.GRPCGatewayMetadataTraceCarrierFxOption(), // read it's documentation to understand better
 )
}
```

{{%notice warning%}}
Knowing what HTTP headers have this value depends on the Implementing Tracing library, Jaeger GO client knows how to do that :)
{{%/notice%}}

{{%notice%}}
To better understand how `MetadataTraceCarrierOption` works [read this](https://pkg.go.dev/github.com/go-masonry/mortar/http/server#MetadataTraceCarrierOption)
{{%/notice%}}

You can look at a working example [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/http.go#L17).

### Clients

You should get yourself familiar with [Mortar Clients](/clients) first.

When calling remote services you need to pass your current Tracing information forward.
Basically you have 2 options

1. Create a Client Span for every remote call and then pass it.
2. Just pass the Tracing Information forward.

Mortar supports the first (with Client Span) option out-of-the-box.

1. [gRPC Client Interceptor](https://pkg.go.dev/github.com/go-masonry/mortar/middleware/interceptors/trace#TracerGRPCClientInterceptor)
2. [HTTP Client Interceptor](https://pkg.go.dev/github.com/go-masonry/mortar/middleware/interceptors/trace#TracerRESTClientInterceptor)

In order to use them, they must be provided to Uber-Fx first.

```golang
func HttpClientFxOptions() fx.Option {
 return fx.Options(
  providers.HTTPClientBuildersFxOption(), // client builders
  providers.TracerGRPCClientInterceptorFxOption(),
  providers.TracerRESTClientInterceptorFxOption(),
 )
}
```

Once provided you can use Mortar Clients as any other gRPC Clients.

* gRPC Client example can be found [here](https://github.com/go-masonry/mortar-demo/blob/master/subworkshop/app/controllers/subworkshop.go#L21)
* HTTP Client example can be found [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/controllers/workshop.go#L24)

## Adding dynamic info to current Span

Sometime you need to add a `Custom Tag` or a `Custom Log` to the current Span.

It's really easy and it's not related to Mortar.

* To add a custom Tag:

    ```golang
    if span := opentracing.SpanFromContext(ctx); span != nil {
        span.SetTag("custom", "tag")
    }
    ```

* To add a custom Log:

    ```golang
    if span := opentracing.SpanFromContext(ctx); span != nil {
        span.LogFields(spanLog.String("custom", "key"))
    }
    ```

Here is what it should look like in Jaeger.

<!-- > ![Tags](/images/tagslogs.png) -->

{{< container-image path="images/tagslogs.png" fit=true  alt="Gray cat" >}}
