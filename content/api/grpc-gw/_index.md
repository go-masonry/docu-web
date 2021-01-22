---
title: "gRPC-Gateway"
draft: true
---

Mortar comes with a [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway), which is a reverse-proxy that translates a RESTful HTTP API into gRPC.
We will show how you should register its handlers, after you generate them from the proto files.

### Register `grpc-gateway` Handlers

{{%notice%}}Before reading this part, get yourself familiar with the gRPC API [counterpart](/api/grpc).{{%/notice%}}

If you've read the gRPC part, you simply need to add one [function](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L49) to `Uber-FX` graph.
This function should return a slice of [GRPCGatewayGeneratedHandlers](https://pkg.go.dev/github.com/go-masonry/mortar/interfaces/http/server#GRPCGatewayGeneratedHandlers).

1. Register your `grpc-gateway` generated Handlers:
   {{%notice warning "Important" %}}Each handler function must register itself on the provided `runtime.ServeMux`{{%/notice%}}

   ```golang
   func workshopGRPCGatewayHandlers() []serverInt.GRPCGatewayGeneratedHandlers {
       return []serverInt.GRPCGatewayGeneratedHandlers{
           // Register workshop REST API
           func(mux *runtime.ServeMux, endpoint string) error {
               return workshop.RegisterWorkshopHandlerFromEndpoint(context.Background(), mux, endpoint, []grpc.DialOption{grpc.WithInsecure()})
           },
           // Any additional gRPC gateway registrations should be called here
       }
   }
   ```

2. Now tell `Uber-FX` about it and make sure it's added to the [GRPCGatewayGeneratedHandlers](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L33) group.

   {{%notice%}}
   However, in this case our function doesn't return a single value, but an array of them.
   By default `Uber-FX` will treat it as an Array of Arrays `[][]GRPCGatewayGeneratedHandlers`.
   Hence we need to [**flatten**](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L34) our value.
   {{%/notice%}}

   ```golang
   // GRPC Gateway Generated Handlers registration
   fx.Provide(fx.Annotated{
    // "flatten" does this [][]serverInt.GRPCGatewayGeneratedHandlers -> []serverInt.GRPCGatewayGeneratedHandlers
    Group:  groups.GRPCGatewayGeneratedHandlers + ",flatten",
    Target: workshopGRPCGatewayHandlers,
   })
   ```

3. Finally, you need to add the above `fx.Option` to `Uber-FX` graph, as shown [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/main.go#L39).

## REST Enabled

You now have an RESTful reverse-proxy that automatically translates REST API calls to their gRPC counterparts.
Following this guide, your REST API will be exposed on `5381` port.

```http
POST /v1/workshop/cars HTTP/1.1
Accept: application/json, */*;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Content-Length: 88
Content-Type: application/json
Host: localhost:5381

{
    "body_style": "HATCHBACK",
    "color": "blue",
    "number": "12345679",
    "owner": "me myself"
}


HTTP/1.1 200 OK
Content-Length: 2
Content-Type: application/json
Date: Thu, 21 Jan 2021 11:54:47 GMT
Grpc-Metadata-Content-Type: application/grpc

{}
```

As you can see above **grpc-gateway** did a great job.
