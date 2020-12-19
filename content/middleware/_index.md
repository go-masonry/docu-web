---
title: "Middleware"
date: 2020-12-06T10:35:11+02:00
draft: true
weight: 90
---

It's hard to define middleware, but in Mortar's context you can think of it as **proxy/interceptors**.
Some of them are bundled as part of the libraries Mortar uses, others defined by Mortar.
  
  - The gRPC package already has a way to register different interceptors
    for [client](https://pkg.go.dev/google.golang.org/grpc#UnaryClientInterceptor)
    and [server](https://pkg.go.dev/google.golang.org/grpc#UnaryServerInterceptor).
    Mortar already comes with some:

    - Server interceptors:
      - [Logger](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/server/logger.go#L23) `UnaryServerInterceptor`, making it easier to log gRPC request and response.
      - [Monitoring](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/server/monitor.go#L30) `UnaryServerInterceptor`, this one can create automatic Histogram/Timer metric for each gRPC method.
      - [Tracing](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/trace/server.go#L13) `UnaryServerInterceptor`, injects or uses tracing information.
    - Client interceptors:
      - [Tracing](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/trace/client.go#L18) `UnaryClientInterceptor`, creates a client span and passes tracing information to the next hop.
      - [In->Out](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/client/headers.go#L23) `UnaryClientInterceptor`, this one can be used to copy specific headers from `grpc.Incoming` to `grpc.Outgoing`.

  - HTTP interceptors:
    - Server HTTP middleware.
    {{%notice secondary%}}There are no `http.Handler` interceptors bundled with Mortar. It's better to use the gRPC layer for that, but you can always add your own.{{%/notice%}}
    - Client.
    {{%notice success%}}Mortar introduces a way to add interceptors to `http.Client`, [here](https://pkg.go.dev/github.com/go-masonry/mortar/interfaces/http/client#HTTPClientInterceptor) you can see how it's defined.{{%/notice%}}
      - [Tracing](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/trace/client.go#L44) `HTTPClientInterceptor`, similar to the gRPC one, but for HTTP.
      - Request/Response [dump interceptor](https://pkg.go.dev/github.com/go-masonry/mortar@v0.1.6/middleware/interceptors/client#DumpRESTClientInterceptor), for debugging purposes.
      - In Workshop Demo there is a special [TEST](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/controllers/workshop_test.go#L164) Interceptor that returns a custom response.

### Other types of middleware

Network interceptors are not the only ones you can use in Mortar.
There are other types of middleware.

They rely on `context.Context` passed to most methods, read [here](/middleware/context) about that.
