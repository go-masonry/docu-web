---
title: "Groups"
draft: true
---

`Uber-FX` [group](https://pkg.go.dev/go.uber.org/fx#hdr-Value_Groups) is a feature that allows you to consume and produce
multiple values of the same type. This makes it easier to influence/configure different instances.

Mortar has different [groups](https://pkg.go.dev/github.com/go-masonry/mortar/providers/groups), but we will focus on one of them here.

{{% panel header="Internal HTTP Handlers" %}}

```golang
// InternalHTTPHandlers - Internal Http Handlers group. Mortar comes with several internal handlers, you can add yours.
InternalHTTPHandlers = partial.FxGroupInternalHTTPHandlers

//InternalHTTPHandlerFunctions - Internal Http Handler Functions group. Similar toInternalHttpHandlers but for functions
InternalHTTPHandlerFunctions = partial.FxGroupInternalHTTPHandlerFunctions
```

{{%/panel%}}

### Build Information

Similar to everything else in `Uber-FX`, to register a new instance into a `fx.Group` you need to create a Constructor.

We will look at one of the internal handlers: [Build Information](https://github.com/go-masonry/mortar/blob/master/handlers/self.go#L38).

{{% panel header="Build Information"%}}

In this example we want to register a new **internal** [`"/<path>/<pattern>"` -> `http.HandlerFunc`] pair.

1. Let's start by defining a constructor that will return `http.HandlerFunc`:

    ```golang
    func (s *selfHandlerDeps) BuildInfo() http.HandlerFunc {
      return func(w http.ResponseWriter, req *http.Request) {
      information := mortar.GetBuildInformation(true) // using true here will insert a default value if there is none
      if err := json.NewEncoder(w).Encode(information); err != nil {
        w.WriteHeader(http.StatusInternalServerError)
        s.Logger.WithError(err).Warn(nil, "failed to serve build info")
      }
      }
    }
    ```

2. Create an `HTTPHandlerPatternPair`.
   This is the part where you define on what **path** this handler will be served:
    ```golang
    // SelfHandlers this service information handlers
    func SelfHandlers(deps selfHandlerDeps) []partial.HTTPHandlerPatternPair {
      return []partial.HTTPHandlerPatternPair{
      {Pattern: "/self/build", Handler: deps.BuildInfo()},
      ...
      }
    }
    ```
3. [Tell](https://github.com/go-masonry/mortar/blob/master/providers/handlers.go#L45) `Uber-FX` that we want this instance of `HTTPHandlerPatternPair` added to the `groups.InternalHTTPHandlerFunctions` group.
    ```golang
    func InternalSelfHandlersFxOption() fx.Option {
      return fx.Provide(
      fx.Annotated{
        Group:  groups.InternalHTTPHandlers + ",flatten",
        Target: handlers.SelfHandlers,
      })
    }
    ```

4. Lastly, in your application you need to provide the above `InternalSelfHandlersFxOption()` option to `Uber-FX` graph,
   [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/main.go#L37) you can see how it's done in the demo.
    ```golang
    return fx.New(
      ...
      InternalSelfHandlersFxOption(),
      ...
    )
    ```

{{%/panel%}}
