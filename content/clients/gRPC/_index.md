---
title: "gRPC Client"
draft: true
weight: 75
---

[**gRPC**](https://grpc.io/docs/languages/go/quickstart/) Clients are usually generated by Protobuf compilers such as `protoc` or `buf`.
The connection itself is generic and this is where Mortar enters.

```golang
// GRPCClientConnectionWrapper is a convenience wrapper to support predefined dial Options
// provided by GRPCClientConnectionBuilder
type GRPCClientConnectionWrapper interface {
    // Context can be nil
    Dial(ctx context.Context, target string, extraOptions ...grpc.DialOption) (grpc.ClientConnInterface, error)
}

// GRPCClientConnectionBuilder is a convenience builder to gather []grpc.DialOption
type GRPCClientConnectionBuilder interface {
    AddOptions(opts ...grpc.DialOption) GRPCClientConnectionBuilder
    Build() GRPCClientConnectionWrapper
}
```

{{%panel header="**GRPCClientConnectionWrapper** explained" theme="info"%}}
`GRPCClientConnectionWrapper.Dial` method is very similar to one defined in [gRPC client](https://pkg.go.dev/google.golang.org/grpc#DialContext).

```golang
func DialContext(ctx context.Context, target string, opts ...DialOption) (conn *ClientConn, err error)
```

However, the return value from gRPC method is `*grpc.ClientConn` which is a struct.
This makes it difficult to mock if needed.
Lately gRPC project added `grpc.ClientConnInterface` to help them in tests, but since it's an Interface it was borrowed...
{{%/panel%}}

Mortar provides a Builder interface that allows you to add different Options before creating a connection.
While you can create one yourself, Mortar comes with predefined [Client Builders](https://pkg.go.dev/github.com/go-masonry/mortar/providers#HTTPClientBuildersFxOption).
You just need to make sure they're registered/provided in Uber-Fx.

{{%notice%}}`HTTPClientBuildersFxOption` is used for both gRPC and HTTP Clients.{{%/notice%}}

Once provided you should inject the **`client.GRPCClientConnectionBuilder`**.

### Customizing gRPC connection

While you should add all the **gRPC Client Interceptors** using a provided Uber-Fx group [`GRPCUnaryClientInterceptors`](https://pkg.go.dev/github.com/go-masonry/mortar/providers/groups),
the reason you inject the **`client.GRPCClientConnectionBuilder`** is the ability to add custom options for a particular connection.

* Ignore TLS by adding `grpc.WithInsecure()` only when connecting to server **X**.
* Block until the underlying connection is up using `grpc.WithBlock()` when establishing a connection against service **Y**.
* More options can be found [here](https://pkg.go.dev/google.golang.org/grpc#DialOption).

### Example

We will use our Demo -> SubWorkshop as an example.

1. Register/Provide HTTP Clients [dependencies](https://github.com/go-masonry/mortar-demo/blob/master/subworkshop/app/mortar/http.go#L8).

   ```golang
    import (
        "github.com/go-masonry/mortar/providers"
        "go.uber.org/fx"
    )

    func HttpClientFxOptions() fx.Option {
        return fx.Options(
            providers.HTTPClientBuildersFxOption(), // client builders for gRPC and HTTP
        )
    }
   ```

2. Inject [builder](https://github.com/go-masonry/mortar-demo/blob/master/subworkshop/app/controllers/subworkshop.go#L21).

   ```golang
    type subWorkshopControllerDeps struct {
     fx.In

     GRPCClientBuilder client.GRPCClientConnectionBuilder
    }
   ```

3. Create a [connection](https://github.com/go-masonry/mortar-demo/blob/master/subworkshop/app/controllers/subworkshop.go#L45).

    ```golang
    wrapper := s.deps.GRPCClientBuilder.Build()
    conn, err := wrapper.Dial(ctx, "localhost:5381", grpc.WithInsecure())
    if err != nil {
        return fmt.Errorf("can't connect to %s, %w", "localhost:5381", err)
    }
    ```

4. Use this connection to create a [gRPC client](https://github.com/go-masonry/mortar-demo/blob/master/subworkshop/app/controllers/subworkshop.go#L52).

   ```golang
    grpcClient := protopackage.NewServiceClient(conn)
    resp, err := grpcClient.DoSomething(ctx, &protopackage.ServiceRequest{})
    if err != nil {
        return err
    }
    fmt.Printf("Response: %s", resp)
   ```