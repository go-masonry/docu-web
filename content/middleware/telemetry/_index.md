---
title: "Telemetry"
draft: true
---

This section is about collecting telemetry **data** from your services and software. This data is usually forwarded to different analytic tools such as:

- Jaeger
- Prometheus
- ELK

Mortar has different middleware to help you with that.
It defines some `interfaces` and different middleware helpers, but not the implementations.
Let's break them into 3 different topics:

- [Logging](/middleware/telemetry/logging)
- [Monitoring](/middleware/telemetry/monitoring)
- [Tracing](/middleware/telemetry/tracing)

Each topic is covered differently, but essentially they are somewhat connected.
