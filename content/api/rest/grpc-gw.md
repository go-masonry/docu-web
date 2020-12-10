---
title: "gRPC-Gateway"
date: 2020-12-09T10:41:15+02:00
draft: true
---

Mortar comes with [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) which is a reverse-proxy that translates a RESTful HTTP API into gRPC.
We will show how you should register it's handlers after you generate them from the proto files.

### Register `grpc-gateway` Handlers

{{%notice%}}Before reading this part get yourself familiar with the gRPC API [counterpart](/api/grpc).{{%/notice%}}

If you read the gRPC part, you simply need to add one [function](https://github.com/go-masonry/mortar-demo/blob/4a178a4540e185341a99388786a26d229422729c/workshop/app/mortar/workshop.go#L49) to `Uber-FX` graph.
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
