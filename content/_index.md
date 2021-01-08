---
title: "Mortar Documentation"
date: 2020-12-10T13:08:23+02:00
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
- Increases Developer Velocity
- DevOps friendly and Production ready.
  - [Builder pattern](/mortar/builders)
  - Profiling, Debug, Build Info, Configuration and [more](https://github.com/go-masonry/mortar/tree/master/handlers).

The simplest way to start is to clone the [Demo](https://github.com/go-masonry/mortar-demo) repository.

{{%notice secondary%}}This documentation is a WIP{{%/notice%}}
