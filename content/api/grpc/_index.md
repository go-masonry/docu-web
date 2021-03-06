---
title: "gRPC"
draft: true
---

Mortar is mostly about **gRPC (and REST)** web services. We will explain how you should register multiple gRPC/REST APIs.
To make it easier, we will use [mortar-demo](https://github.com/go-masonry/mortar-demo/tree/master/workshop) in our example.

### Protobuf

Before you implement any gRPC service related code, you first need to define the API by writing [proto](https://github.com/go-masonry/mortar-demo/blob/master/workshop/api/workshop.proto) files.
Once that's done, you will [call](https://github.com/go-masonry/mortar-demo/blob/master/workshop/Makefile#L14) `protoc` and generate your `<name>_grpc.pb.go`, `<name>.pb.go` and `<name>.pb.gw.go` [files](https://github.com/go-masonry/mortar-demo/tree/master/workshop/api).

We will be using the Workshop API example.

{{<highlight proto "hl_lines=2">}}
service Workshop {
  rpc AcceptCar(Car) returns (google.protobuf.Empty);
  rpc PaintCar(PaintCarRequest) returns (google.protobuf.Empty);
  rpc RetrieveCar(RetrieveCarRequest) returns (Car);
  rpc CarPainted(PaintFinishedRequest) returns (google.protobuf.Empty);
}
{{</highlight>}}
### Implementing

Once we have our gRPC server interfaces generated, we need to implement them.
In our case, we need to implement [this](https://github.com/go-masonry/mortar-demo/blob/master/workshop/api/workshop_grpc.pb.go#L75) generated interface.

{{<highlight go "lineos=table,hl_lines=5">}}
// WorkshopServer is the server API for Workshop service.
// All implementations must embed UnimplementedWorkshopServer
// for forward compatibility
type WorkshopServer interface {
 AcceptCar(context.Context, *Car) (*empty.Empty, error)
 PaintCar(context.Context, *PaintCarRequest) (*empty.Empty, error)
 RetrieveCar(context.Context, *RetrieveCarRequest) (*Car, error)
 CarPainted(context.Context, *PaintFinishedRequest) (*empty.Empty, error)
 mustEmbedUnimplementedWorkshopServer()
}
{{</highlight>}}

You can see how it's done in [app/services/workshop.go](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/services/workshop.go).
Here is one of the methods:

{{<highlight go>}}
...
func (w *workshopImpl) AcceptCar(ctx context.Context, car *workshop.Car) (*empty.Empty, error) {
 if err := w.deps.Validations.AcceptCar(ctx, car); err != nil {
  return nil, err
 }
 w.deps.Logger.WithField("car", car).Debug(ctx, "accepting car")
 return w.deps.Controller.AcceptCar(ctx, car)
}
...
{{</highlight>}}

### Registering

Once we have the implementation covered, we need to register it. There are several steps you need to cover:

1. Create a function that will return [GRPCServerAPI](https://pkg.go.dev/github.com/go-masonry/mortar/interfaces/http/server#GRPCServerAPI).
   {{%notice warning "Important"%}}In this function you must register the gRPC API implementation on the provided `grpc.Server`{{%/notice%}}

   {{<highlight go>}}
    func workshopGRPCServiceAPIs(deps workshopServiceDeps) serverInt.GRPCServerAPI {
     return func(srv *grpc.Server) {
      workshop.RegisterWorkshopServer(srv, deps.Workshop)
      // Any additional gRPC Implementations should be called here
     }
    }
   {{</highlight>}}

    You can look [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L42) to understand how it's done in our workshop example.

2. Next, add it to the `groups.GRPCServerAPIs` group as shown [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/workshop.go#L25):
   {{%notice info%}}To better understand Mortar groups read [here](/fx/groups){{%/notice%}}

   {{<highlight go>}}
   return fx.Options(
    // GRPC Service APIs registration
    fx.Provide(fx.Annotated{
     Group:  groups.GRPCServerAPIs,
     Target: workshopGRPCServiceAPIs,
    }),
   ...
   {{</highlight>}}

   This way you can register multiple gRPC API implementations, and they will all be registered in [one place](https://github.com/go-masonry/mortar/blob/master/constructors/partial/httpserver.go#L89).

3. Now we need to add this option to the `Uber-FX` graph, as shown [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/main.go#L39):
   
   {{<highlight go>}}
   return fx.New(
    ...
    mortar.WorkshopAPIsAndOtherDependenciesFxOption(),
    ...
   )
   {{</highlight>}}
