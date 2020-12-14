---
title: "gRPC-Gateway"
date: 2020-12-09T10:41:15+02:00
draft: true
---

Mortar comes with a [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway) which is a reverse-proxy that translates a RESTful HTTP API into gRPC.
We will show how you should register its handlers, after you generate them from the proto files.

### Register `grpc-gateway` Handlers

{{%notice%}}Before reading this part, get yourself familiar with the gRPC API [counterpart](/api/grpc).{{%/notice%}}

If you read the gRPC part, you simply need to add one [function](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L49) to `Uber-FX` graph.
This function should return a slice of [GRPCGatewayGeneratedHandlers](https://github.com/go-masonry/mortar/blob/master/interfaces/http/server/interfaces.go#L55).

1. Register your `grpc-gateway` generated Handlers
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

2. Now tell `Uber-FX` about it and make sure it's added to [GRPCGatewayGeneratedHandlers](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L33) group.
   
   {{%notice%}}
   However, in this case our function doesn't return a single value, but an array of them.
   By default `Uber-FX` will treat it as an Array of Arrays `[][]GRPCGatewayGeneratedHandlers`.
   Hence we need to [**flatten**](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L34) our value.
   {{%/notice%}}

   ```golang
   // GRPC Gateway Generated Handlers registration
   fx.Provide(fx.Annotated{
    Group:  groups.GRPCGatewayGeneratedHandlers + ",flatten", // "flatten" does this [][]serverInt.GRPCGatewayGeneratedHandlers -> []serverInt.GRPCGatewayGeneratedHandlers
    Target: workshopGRPCGatewayHandlers,
   })
   ```

3. Finally you need to add the above `fx.Option` to `Uber-FX` graph, as shown [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/main.go#L39)
