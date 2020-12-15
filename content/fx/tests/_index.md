---
title: "Tests"
date: 2020-12-05T13:18:51+02:00
draft: true
---

Testing with `Uber-FX` makes it possible to test different logic while mocking parts of the DI graph.

### Constructor per Type

While it's [possible](https://pkg.go.dev/go.uber.org/fx?readme=expanded#example-package) to register several instances using one constructor function, If possible **avoid this**.

{{%panel header="Official Uber-FX documentation"%}}

```golang
// Functions may also return multiple objects. For example, we could combine
// NewHandler and NewLogger into a single function:
//
//   func NewHandlerAndLogger() (*log.Logger, http.Handler, error)
//
// Fx also understands this idiom, and would treat NewHandlerAndLogger as the
// constructor for both the *log.Logger and http.Handler types. Just like
// constructors for a single type, NewHandlerAndLogger would be called at most
// once, and both the handler and the logger would be cached and reused as
// necessary.
```

{{%/panel%}}

{{%alert danger%}} Make sure the constructor function returns only ONE Parameter and an optional Error.{{%/alert%}}

Let's explain the reason behind that. During tests, sometimes you will want to **swap** a real dependency with a fake/mocked one.
Currently, the real practical way to do it is **NOT calling** the real constructor function that creates this dependency, but call a different one that returns the same type.
If your constructor function will return several types, you can't really **swap only one** of them. You will need to swap **all of them** since you can't call the real constructor.