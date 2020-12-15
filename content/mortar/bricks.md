---
title: "Bricks"
date: 2020-12-10T13:08:23+02:00
draft: true
hide:
 - nextpage
---

When we build software, most of us always try to rely on something we or others have built before us.
Since we don't want to reinvent the wheel, **again**.

Go's standard library is an excellent example here.
There are `strings`, `time`, `http` and many other build-in libraries that we use.
While this example is great, it doesn't scale to 3rd party libraries.

Let's look at Logger libraries for example, there are:

- [Logrus](https://github.com/sirupsen/logrus)
- [Apex](https://github.com/apex/log)
- [Zap](https://github.com/uber-go/zap)
- [Zerolog](https://github.com/rs/zerolog)
- ...

You are encouraged to have a look at each library, but I can assure you, all of their APIs are different from each other.
Mortar after all is a library, and its purpose to be used in many projects.
That is why we defined different [interfaces](https://pkg.go.dev/github.com/go-masonry/mortar/interfaces).

{{%alert%}}We define a Brick as an implementation of an interface defined in Mortar using ann external library{{%/alert%}}

- [Mortar Logger](https://github.com/go-masonry/mortar/blob/master/interfaces/log/interfaces.go)
    - [Implementation using Zerolog](https://github.com/go-masonry/bzerolog)
- [Mortar Config](https://github.com/go-masonry/mortar/blob/master/interfaces/cfg/interfaces.go)
    - [Implementation using Viper](https://github.com/go-masonry/bviper)
- [Mortar Tracing](https://github.com/go-masonry/mortar/blob/master/interfaces/trace/interfaces.go)
    {{%notice%}} This is a special case, since [open tracing](https://github.com/opentracing/opentracing-go) is already an abstraction.{{%/notice%}}
    - [Implementation using Jaeger](https://github.com/go-masonry/bjaeger)
- ...

To easily differentiate an actual library from its Brick wrapper, every Brick package starts with a `b`.

- **b**viper
- **b**zerolog
- ...

{{%alert warning%}}As a rule, every Brick is based on Mortar itself, but not on any other Brick{{%/alert%}}

Meaning you can't import `bzerolog` from within `bjaeger`, for example.

Check their `go.mod` files to make sure.

This rule is here to ensure that every Brick can be easily swapped.
