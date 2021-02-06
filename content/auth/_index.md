---
title: "JWT Authentication"
draft: true
weight: 95
---

In case you decide to use [JWT](https://jwt.io) as your Auth* of choice, Mortar have a simple [Token Extractor](https://pkg.go.dev/github.com/go-masonry/mortar/interfaces/auth/jwt#TokenExtractor).

```golang
type TokenExtractor interface {
	// FromContext should try to extract a token from the Context using `ContextExtractor`
	FromContext(ctx context.Context) (Token, error)
	// FromString accepts a JWT token in form of a string
	//	xxxxx.yyyyy.zzzzz
	FromString(str string) (Token, error)
}
```

## Mortar and gRPC-Gateway

Usually you pass the JWT Token via the `Authorization` header

```http
Authorization: bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Now, you are encouraged to use [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway), but in this case you don't really have an access to HTTP Request Headers.
Fear not, gRPC-Gateway and Mortar have your covered.

* By default gRPC-Gateway will transform this special `Authorization` header into `grpcgateway-authorization` and put it in the Context.
* Mortar have a [`DefaultJWTTokenExtractor`](https://pkg.go.dev/github.com/go-masonry/mortar/constructors#DefaultJWTTokenExtractor) that takes care of that.

## Example

You only need to register [Token Extractor](https://pkg.go.dev/github.com/go-masonry/mortar/interfaces/auth/jwt#TokenExtractor) by providing [`DefaultJWTTokenExtractor`](https://pkg.go.dev/github.com/go-masonry/mortar/constructors#DefaultJWTTokenExtractor) constructor to Uber-Fx.

```golang
import (
 "github.com/go-masonry/mortar/constructors"
 "go.uber.org/fx"
)
...
fx.Provide(constructors.DefaultJWTTokenExtractor)
...
```

Once provided, you can inject `TokenExtractor` and use it

```golang
type authDeps struct {
 fx.In

 TokenExtractor jwt.TokenExtractor
}
...

func (impl *authImpl) CheckJWT(ctx context.Context) error {
 // we only log for error here
 token, err := impl.deps.TokenExtractor.FromContext(ctx)
 if err != nil {
     return fmt.Errorf("no jwt token found, %w",err)
 }
 m, err = token.Map()
 if err != nil {
   return fmt.Errorf("failed to produce a map from authorization header, %w", err)
 }
 ...
 ...
 return nil
}

```

Mortar [service template](https://github.com/go-masonry/mortar-template) have a simple working example. Look at [app/validations/auth.go](https://github.com/go-masonry/mortar-template/blob/master/app/validations/auth.go).
