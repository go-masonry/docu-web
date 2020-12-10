---
title: "Uber-FX"
date: 2020-12-06T10:35:11+02:00
draft: true
---

## Inversion of Control

If you are unfamiliar with the concept please read about [it](https://en.wikipedia.org/wiki/Inversion_of_control).

Mortar is heavily based on this principle. There are different libraries to achieve that in Go.

{{%notice success%}}
Mortar uses [`Uber-FX`](https://github.com/uber-go/fx) and you're strongly encouraged to read all about it.

**Well actually you kinda have to.**
{{%/notice%}}

To summarize, this is what you need to understand about IoC frameworks

- You are not the one creating the dependencies, `Uber-FX` does it for you. Just tell it how.
- Every dependency is a Singleton, hence that same instance reused everywhere.
- Once your dependencies defined as an Interface it's really easy to swap it. Especially during tests.
- There is no magic, only implicits.