---
title: "Telemetry"
draft: true
---

{{%notice success%}} This section is a work in progress {{%/notice%}}

This section is about collecting telemetry **data** from your services and software. This data is usually forwarded to different analytic tools such as:

- Jaeger
- Prometheus
- EKS

Mortar has different middleware to help you with that.
It defines some `interfaces` and different middleware helpers, but not the implementations.
Let's break them into 3 different topics:

- Logging
- Monitoring
- Tracing

Each topic is covered differently, but essentially they are somewhat connected.
