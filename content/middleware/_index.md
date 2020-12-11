---
title: "Middleware"
date: 2020-12-06T10:35:11+02:00
draft: true
weight: 90
---

It's hard to define Middleware, but in Mortar context you can think of it as **Proxy/Interceptors**.
Some of them are bundled as part of the libraries Mortar uses, others defined by Mortar.
  
  - gRPC package already has a way to register different Interceptors
    for [Client](https://github.com/grpc/grpc-go/blob/master/interceptor.go#L43)
    and [Server](https://github.com/grpc/grpc-go/blob/master/interceptor.go#L84).
    Mortar already comes with some.

    - Server Interceptors
      - [Logger](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/server/logger.go#L23) `UnaryServerInterceptor`, making it easier to log gRPC request and response.
      - [Monitoring](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/server/monitor.go#L30) `UnaryServerInterceptor`, this one can create automatic Histogram/Timer metric for each gRPC method.
      - [Tracing](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/trace/server.go#L13) `UnaryServerInterceptor`, Injects or Uses tracing information.
    - Client Interceptors
      - [Tracing](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/trace/client.go#L18) `UnaryClientInterceptor`, creates a Client span and passes Tracing Information to the next hop.
      - [In->Out](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/client/headers.go#L23) `UnaryClientInterceptor`, this one can be used to copy specific headers from `grpc.Incoming` to `grpc.Outgoing`

  - HTTP interceptors
    - Server HTTP Middleware
    {{%notice secondary%}}There are no `http.Handler` Interceptors bundled with Mortar. It's better to use the gRPC layer for that, but you can always add your own.{{%/notice%}}
    - Client
    {{%notice success%}}Mortar introduces a way to add interceptors to `http.Client`, [here](https://github.com/go-masonry/mortar/blob/master/interfaces/http/client/interfaces.go#L21) you can see how it's defined.{{%/notice%}}
      - [Tracing](https://github.com/go-masonry/mortar/blob/master/middleware/interceptors/trace/client.go#L44) `HTTPClientInterceptor`, similar to the gRPC one, but for HTTP.
      - In Workshop Demo there is a special [TEST](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/controllers/workshop_test.go#L164) Interceptor that returns a custom response.

### Other types of Middleware

Network Interceptors are not the only ones you can use in Mortar.
There are other types of Middleware.

They rely on `context.Context` passed to most methods, read [here](/middleware/context) about that.
