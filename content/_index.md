---
draft: true
hide:
    - toc
---

Mortar is a Go framework/library for building gRPC (and REST) web services.

It has out-of-the-box support for configuration, application metrics, logging, tracing, profiling, dependency injection and more.

The key features are:

- [Dependency Injection](/fx) using [Uber-FX](https://github.com/uber-go/fx).
  - Out-of-the-box defaults that work, however you can [replace everything](/mortar/bricks).
- [Middleware concept](/middleware) is almost everywhere.
  - [Build-in](/middleware) Server/Client Interceptors both for gRPC/HTTP, you can bring your own.
  - [Telemetry](/middleware/telemetry) - Metrics, Tracing and Logging all connected.
- Pimped gRPC and HTTP [clients](/clients).
- Increases Developer Velocity
- DevOps friendly and Production ready.
  - [Builder pattern](/mortar/builders)
  - Profiling, Debug, Build Info, Configuration and [more](https://github.com/go-masonry/mortar/tree/master/handlers).
  - Create a production ready service in 15min using this [service template](https://github.com/go-masonry/mortar-template).

{{%notice success%}}
If you want to better understand Mortar features, clone the [Demo](https://github.com/go-masonry/mortar-demo) repository and follow the README there.
{{%/notice%}}
