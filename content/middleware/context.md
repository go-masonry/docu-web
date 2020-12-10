---
title: "Context"
date: 2020-12-06T10:35:11+02:00
draft: true
---

If you are not familiar with `context.Context` please read [this](https://golang.org/pkg/context/) and [that](https://blog.golang.org/context) first.

From context documentation package

{{% alert theme="info" %}} *Incoming requests to a server should create a Context, and outgoing calls to servers should accept a Context.* {{%/alert %}}

Since we are building a gRPC Web Service it's part of the design.
Everything gRPC related already have a `context.Context` as the first argument.

{{% panel header="gRPC" %}}

- [client](https://github.com/grpc/grpc-go/blob/master/call.go#L29)

    ```golang
    func (cc *ClientConn) Invoke(ctx context.Context, method string, args, reply interface{}, opts ...CallOption) error
    ```

- [interceptors](https://github.com/grpc/grpc-go/blob/master/interceptor.go)

    ```golang
    type UnaryClientInterceptor func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, invoker UnaryInvoker, opts ...CallOption) error

    type UnaryServerInterceptor func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (resp interface{}, err error)
    ```

- Also all the [generated](https://github.com/golang/protobuf/blob/master/protoc-gen-go/grpc/grpc.go) server endpoints have `context.Context` as the first argument.
{{%/panel%}}

### Storage

One of the "drawbacks" in GO is a lack of [storage](https://github.com/golang/go/issues/21355) per go routine.
You can "solve" this drawback by

- {{< icon name="fa-bomb" >}} By having a sync `map[goroutine-id]storage`, but it's probably not a practical idea.
- {{< icon name="fa-thumbs-up" >}} Propagate `context.Context` as the first parameter for every public function.

Actually the second option is largely encouraged and became a kind of a standard.

- [Tracing](https://github.com/opentracing/opentracing-go/blob/master/gocontext.go) uses context to store a `traceId`, `spanId` and other information.
- [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) reverse proxy injects selected `http.Request` headers into the context.

In mortar most Interfaces have a way to leverage on `context.Context` and depending on the package there are different `ContextExtractor` definitions.

Let's look at Logging as an example

### Logging Example

All Logger methods have a `context.Context` as it's first argument

```golang
Debug(ctx context.Context, format string, args ...interface{})
Info(ctx context.Context, format string, args ...interface{})
```

Meaning that every log entry have a context...

Internally just before we send the log to the implementing library we [iterate](https://github.com/go-masonry/mortar/blob/master/logger/context_logger.go#L82) on every provided `ContextExtractor` and add all the extracted key->value pairs to the log entry.

```golang
func (c *contextAwareLogEntry) enrich(ctx context.Context) (logger log.Fields) {
	defer func() {
		if r := recover(); r != nil {
			c.innerLogger.WithField("__panic__", r).Error(ctx, "one of the context extractors panicked")
			logger = c.innerLogger
		}
	}()
	logger = c.innerLogger
	for _, extractor := range c.contextExtractors {
		for k, v := range extractor(ctx) {
			logger = logger.WithField(k, v)
		}
	}
	return
}
```

One of such `ContextExtractor` is provided by the [bjaeger](https://github.com/go-masonry/bjaeger/blob/master/utils.go#L37) wrapper.

```golang
func extractFromSpanContext(ctx jaeger.SpanContext) map[string]interface{} {
	var output = make(map[string]interface{}, 4)
	output[SpanIDKey] = ctx.SpanID().String()
	output[ParentSpanIDKey] = ctx.ParentID().String()
	output[SampledKey] = ctx.IsSampled()
	if traceID := ctx.TraceID(); traceID.IsValid() {
		output[TraceIDKey] = traceID.String()
	}
	return output
}
```

If there is a Trace Span within the Context we will include it's information in the log entry.