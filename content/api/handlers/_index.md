---
title: "HTTP Handler/Func"
draft: true
---

Although it's very convenient to use gRPC + gRPC-Gateway to serve RESTful APIs, you sometime want to use prebuilt HTTP Handler/HandlerFunc(s) or similar.

Mortar provides 2 `fx.Group`(s) for that: [ExternalHTTPHandlers](https://github.com/go-masonry/mortar/blob/master/providers/groups/alias.go#L26) and [ExternalHTTPHandlerFunctions](https://github.com/go-masonry/mortar/blob/master/providers/groups/alias.go#L29), one for `http.Handler` and the other for `http.HandlerFunc`.

{{%notice info%}}To better understand Mortar groups, read [here](/fx/groups){{%/notice%}}

### Registering `http.Handler`

To register a new `http.Handler`, you need to define how it's going to be served == **Pattern**:

1. Create a [HTTPHandlerPatternPair](https://pkg.go.dev/github.com/go-masonry/mortar/providers/types#HTTPHandlerPatternPair):
   
   ```golang
   var customHandler http.Handler ...

   func createHandlerPair() types.HTTPHandlerPatternPair {
    return types.HTTPHandlerPatternPair{
        Pattern:"/your/pattern",
        Handler: customHandler,
    }
   }
   ```

2. Add this pair to the `ExternalHTTPHandlers` group:
   
   ```golang
   var customHandlerOption = fx.Provide(
       fx.Annotated{
           Group: groups.ExternalHTTPHandlers,
           Target: createHandlerPair, // function pointer, will be called by Uber-FX
       }
   )
   ```

3. All that's left is to register it into the `Uber-FX` graph:

   ```golang
   return fx.New(
    ...
    customHandlerOption,
    ...
   )
   ```

### Registering `http.HandlerFunc`

Registering `http.HandlerFunc` is very much similar to [registering `http.Handler`](#registering-httphandler).

Actually, you need to follow the exact same guide and replace **Handler** with **HandlerFunc**:

{{< tabs >}}
    {{< tab "Handler">}}
```golang
var customHandler http.Handler ...

func createHandlerPair() types.HTTPHandlerPatternPair {
 return types.HTTPHandlerPatternPair{
     Pattern:"/your/pattern",
     Handler: customHandler,
 }
}

var customHandlerOption = fx.Provide(
    fx.Annotated{
        Group: groups.ExternalHTTPHandlers,
        Target: createHandlerPair, // function pointer, will be called by Uber-FX
    }
) 
return fx.New(
 ...
 customHandlerOption,
 ...
)
```
    {{< /tab >}}

    {{< tab "HandlerFunc">}}
```golang
var customHandlerFunc http.HandlerFunc ...

func createHandlerFuncPair() types.HTTPHandlerFuncPatternPair {
 return types.HTTPHandlerFuncPatternPair{
     Pattern:"/your/pattern",
     Handler: customHandlerFunc,
 }
}

var customHandlerFuncOption = fx.Provide(
    fx.Annotated{
        Group: groups.ExternalHTTPHandlerFunctions,
        Target: createHandlerFuncPair, // function pointer, will be called by Uber-FX
    }
) 
return fx.New(
 ...
 customHandlerFuncOption,
 ...
)
```
    {{< /tab >}}
{{< /tabs >}}
