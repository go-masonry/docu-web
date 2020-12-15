---
title: "Listeners"
date: 2020-12-10T13:08:23+02:00
draft: true
hide:
 - nextpage
---

We assume that you, like us, don't expose your services to the world without setting up at least a Load Balancer before it/them.
Historically, when we started building webservices, our infrastructure was expecting a single port to forward all the traffic to it from the LoadBalancer.
When we wanted to use gRPC API, as well as REST, we still had to expose everything under one port.
Fortunately, we weren't the first, and we used this excellent [cmux](https://github.com/soheilhy/cmux) library to solve that problem.

Now, our services were hosted on AWS using [ELB](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-listener-config.html).
Back then, ELB didn't support HTTP/2. However, from Nov 2020 AWS [have gRPC support](https://aws.amazon.com/blogs/aws/new-application-load-balancer-support-for-end-to-end-http-2-and-grpc/)...

gRPC is based on HTTP/2 and that presented some headache... Long story short, we switched to [Envoy](https://www.envoyproxy.io/) and made the following changes:

Our services now have had 3 listeners and 3 ports respectively:

- gRPC
- External REST, which is a reverse-proxy to gRPC using [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway).
- [Internal REST](#internal-rest) which exposes different useful information about the service.

While the first 2 were exposed to the Internet, the third one (**Internal**) wasn't.

## Mortar

Mortar is designed with that in mind. Essentially, you create 3 web services with 3 ports they listen on.
If that setup can't work for you, you can still can do everything with [1 listener and 1 port](#one-listener-one-port).

Let's look at some benefits this approach has.

### External gRPC and REST

Basically, you treat gRPC and External REST as one (but with 2 different ports).
If your Load Balancer doesn't support HTTP/2, it sure does support HTTP/1.
It also can probably give you some [benefits](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html#listener-configuration) if you use an HTTP listener when routing traffic to the External REST port.

### Internal REST

When your services are deployed in some cloud, you need different ways to "look" into them:

- What is their version or even better, Git commit.
- [Configuration and environment](https://github.com/go-masonry/mortar/blob/master/handlers/self.go#L48) values.
    {{%notice success%}} Mortar has a [simple](https://github.com/go-masonry/mortar/blob/master/handlers/self.go#L65) way to obfuscate passwords, secret, tokens, etc.{{%/notice%}}
- Sometimes you even want to [profile](https://golang.org/pkg/net/http/pprof/) your services by using [provided handlers](https://github.com/go-masonry/mortar/blob/master/handlers/profile.go#L15):
    - Runtime Stats (Memory, CPU, ...).
    - Flags provided when your service was started:

        ```shell
        service config <path to file> --answer-to-everythig=42
        ```

    - Or [Debug](https://github.com/go-masonry/mortar/blob/master/handlers/debug.go#L45) them.

Using Internal Handlers and their dedicated listener and port, you can set up a simple but very effective traffic rule,
 by allowing only office/vpn traffic to have access to it.

## One Listener, One Port

While multi-listener/port approach is great (well, at least for us), sometimes it's not possible.
You can still achieve everything using 1 listener and 1 port.
Look at the test example [here](https://github.com/go-masonry/mortar/blob/master/http/server/oneport_test.go).
