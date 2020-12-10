---
title: "Telemetry"
date: 2020-12-06T09:43:50+02:00
draft: true
---

This section is about collecting telemetry **data** from your services and software. This data is usually forwarded to different analytic tools such as

- Jaeger
- Prometheus
- EKS

Mortar have different middleware to help you with that.
It defines some `Interfaces` and different middleware helpers, but not the implementations.
Let's break them into 3 different topics

- Logging
- Monitoring
- Tracing

Each topic is covered differently, but essentially they are somewhat connected.
