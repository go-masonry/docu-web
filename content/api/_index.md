---
title: "APIs"
date: 2020-12-09T10:41:15+02:00
draft: true
---

Mortar goal is to let you easily create gRPC and REST web services.

Mortar supports

- **gRPC API**: you need to define proto files and compile them, read more [here](/api/grpc)
- **RESTful API**: can be implemented in several ways
  - Using gRPC-Gateway, read more [here](/api/rest/grpc-gw)
    - Generate reverse-proxy from [protos](https://github.com/grpc-ecosystem/grpc-gateway#usage) or via [yaml](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/grpc_api_configuration/#grpc-api-configuration).
    - {{< icon fa-bug >}} You can also add [custom](https://grpc-ecosystem.github.io/grpc-gateway/docs/operations/inject_router/#adding-custom-routes-to-the-mux) routes to it.
{{%notice info "Custom route using grpc-gateway ServeMux"%}}
You can't access `runtime.ServeMux` unless you create it yourself.

Once you do, pass it during HTTP sever build, using a dedicated [SetCustomGRPCGatewayMux](https://github.com/go-masonry/mortar/blob/master/interfaces/http/server/interfaces.go#L64) builder option.

You will also need to use your own builder instead of a [default](https://github.com/go-masonry/mortar/blob/master/constructors/partial/httpserver.go#L73) one.
{{%/notice%}}

  - Register custom HTTP Handler/HandlerFunc using a [dedicated group](/api/rest/handlers).
