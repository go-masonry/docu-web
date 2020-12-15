---
title: "Dependency Pattern"
date: 2020-12-05T13:18:51+02:00
draft: true
---

We are using this pattern when creating a new `Uber-FX` dependency, it makes it easy adding new external dependencies later when your application evolves.
To explain this better, we will do this with a `Notifier` example.

### Preparation

1. Define our `Notifier` interface:

    ```golang
    type Notifier interface{
        Alert(ctx context.Context, msg string)
    }
    ```

2. Create a dependency "container" for all future dependencies of the `Notifier` implementation and embed `fx.In` into it.
   This will *mark* it and will tell `Uber-FX` to inject all the requested dependencies:

    ```golang
    // It's not public as you can see
    type notifierDeps struct {
        fx.In
    }
    ```

3. Create an implementation struct:

    ```golang
    // Notice again that's not public
    type notifier struct{
        deps notifierDeps
    }
    ```

4. Create a Constructor function that will return a `Notifier` implementation as a type:

    ```golang
    func CreateNotifier(deps notifierDeps) (Notifier,error) {
        return &notifier(deps:deps), nil
    }
    ```

5. Finally, implement it:

    ```golang
    func (n *notifier) Alert(ctx context.Context, msg string) {
        // alert someone
    }
    ```

### Usage

Now, suppose you want to log every time you alert someone. All you need to do is:

1. Add a `log.Logger` dependency to the `notifierDeps` struct:

    ```golang
    type notifierDeps struct {
        fx.In

        Logger log.Logger
    }
    ```

2. Use it:

    ```golang
    func (n *notifier) Alert(ctx context.Context, msg string) {
        n.deps.Logger.WithField("msg", msg).Debug(ctx, "alerting")
        // alert someone
    }
    ```

### Tests

You are happily using `Notifier` in your application, but what about tests?
You know, to test logic that has `Notifier` as a dependency. `Notifier` will probably call an external service, which is not available during tests.
One way to do it, is to use [gomock](https://github.com/golang/mock).

1. You can add a comment above the `Notifier` interface:

    ```golang
    //go:generate mockgen -source=notifier.go -destination=mock/notifier_mock.go
    type Notifier interface{
        Alert(ctx context.Context, msg string)
    }
    ```

2. Execute `go generate ./...` and it will generate all the mocks in your application.
3. Use the generated mock in your tests.
   {{%alert warning%}}Remember not to call a real Notifier Constructor{{%/alert%}}

Mortar includes mocks of all of its interfaces in their respective directories, for example a [Logger](https://github.com/go-masonry/mortar/tree/master/interfaces/log/mock) mock.

You can find several test examples [here](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/controllers/workshop_test.go).
