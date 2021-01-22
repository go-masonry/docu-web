---
title: "Logging"
draft: true
---

Logging is extremely important, that is why the Go standard library has [one](https://golang.org/pkg/log/).
However, there are better alternatives, and it's up to you to choose the one you want.
You will need to wrap it to implement *Mortar Logger Interface* and you're good to go.
We've [already done](https://github.com/go-masonry/bzerolog) that for [zerolog](https://github.com/rs/zerolog).

One of the goals for *Mortar Logger Interface* was for it to be flexible, but most importantly it had to include `context.Context`, you'll see later why.
Before we go into the details let us start with some usage examples:

- Simple logging similar to `fmt.Printf`:
  
  ```golang
  logger.Debug(ctx, "there were %d different attributes in the request", 5)
  ```

- Structured style logging:

  ```golang
  // Add error, if err != nil it will be shown in the log
  logger.WithError(err).Debug(ctx, "calculation finished")
  // Add Fields, error
  logger.
    WithError(err).
    WithField("method", req.Method).
    WithField("status", req.StatusCode).
    Debug(ctx, "request processed")
  ```

Output depends on the implementation, but most of the libraries will output logs in JSON/Console format.

## Context

In our opinion you should always have `context.Context` as the first parameter for every **public function** you define and **Logger** is not an exception.
You can read all the reasons for that [here](/middleware/context).

Since every `Logger` function has a `context.Context`, we can extract different information from it (if configured) and *ADD* it to the log entry.
When you setup a `Logger` instance to be used everywhere in your application, you can provide different `ContextExtractor`s and they will enrich your log entry.
So what is this mysterious `ContextExtractor`?

{{% notice info%}}**ContextExtractor**
```golang
type ContextExtractor func(ctx context.Context) map[string]interface{}
```
{{%/notice%}}

It's a function that accepts a `context.Context` (which is a map) and outputs a `JSON` style map.
Each key in the map will be added to the log entry.

This opens a lot of different possibilities:

- Add static fields such as [Host, Application Name/Version, etc].
- Add **Tracing** information.
  
  > {{% alert theme="info" %}}If configured properly through your entire infrastructure, this feature allows you to aggregate **ALL the logs** related to a single request across **ALL services**.{{% /alert %}}

- Add dynamic fields from a request, for example `X-Forwarded-For`

### Registering a `ContextExtractor`

Since Mortar is built with `Uber-FX`, it uses the `fx.Group` feature to register different `ContextExtractor`s regardless of where they are defined.
Here you can see what are Mortar Logger's dependencies:

```golang
type loggerDeps struct {
	fx.In

	Config            cfg.Config
	LoggerBuilder     logInt.Builder            `optional:"true"`
	ContextExtractors []logInt.ContextExtractor `group:"loggerContextExtractors"`
}
```

Look at the Group Identifier `group:"loggerContextExtractors"` which also has a [group alias](https://github.com/go-masonry/mortar/blob/master/providers/groups/alias.go#L41) for reference purposes.

{{% notice info%}}**"loggerContextExtractors"**
```golang
// LoggerContextExtractors -  Default Logger Context extractors group. Add custom extractors to enrich your logs from context.Context
LoggerContextExtractors = constructors.FxGroupLoggerContextExtractors
```
{{%/notice%}}

Every `ContextExtractor` known to `Uber-FX` which is labeled with `loggerContextExtractors` will find its way there and will be registered.

{{%panel theme="info" header="Example: showing how an FX Option is defined in external library"%}}

1. Here is an example from a [Jaeger package](https://github.com/go-masonry/bjaeger/blob/master/utils.go#L25):
  
    ```golang
    // TraceInfoContextExtractorFxOption is a preconfigured fx.Option that will allow adding trace info to log entry
    func TraceInfoContextExtractorFxOption() fx.Option {
        return fx.Provide(
            fx.Annotated{

                Group: groups.LoggerContextExtractors, // <<-- Group Name
                
                Target: func() log.ContextExtractor {
                    return TraceInfoExtractorFromContext
                },
            },
        )
    }
    ```

2. Demo application shows how it's registered [`app/mortar/logger.go`](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/logger.go#L20):

    ```golang [main.go]
    func LoggerFxOption() fx.Option {
        return fx.Options(
            ...
            bjaeger.TraceInfoContextExtractorFxOption(),
        )
    }
    ```

3. [`main.go`](https://github.com/go-masonry/mortar-demo/blob/master/workshop/main.go#L32) in Demo application, starting it all:

    ```golang  
    func createApplication(configFilePath string, additionalFiles []string) *fx.App {
            ...
            mortar.LoggerFxOption(),                                  // Logger
            mortar.TracerFxOption(),                                  // Jaeger tracing
            ...
        )
    }
    ```

{{%/panel%}}

Once everything is configured and provided you should see Tracing information in every log, regardless if it's sampled or not.

```shell
6:48PM DBG app/services/workshop.go:48 > accepting car app=workshop car={"body_style":2,"color":"blue","number":"12345679","owner":"me myself"} git=9f00a9c host=Tals-Mac-mini.lan parentSpanId=0 sampled=true spanId=4b71de0edc5b6d18 traceId=4b71de0edc5b6d18 version=v1.2.3

6:48PM DBG app/controllers/workshop.go:50 > car accepted app=workshop git=9f00a9c host=Tals-Mac-mini.lan parentSpanId=0 sampled=true spanId=4b71de0edc5b6d18 traceId=4b71de0edc5b6d18 version=v1.2.3
```

As you see above there are **2 logs**, one is called from `app/services/...` the other from `app/controllers/...`.

Both of these logs have the same `traceId=4b71de0edc5b6d18`.
If you'll [forward](/middleware/telemetry/tracing/#clients) this tracing information to another remote service you will see the same `traceId=4b71de0edc5b6d18` in that service logs as well.

Now you can aggregate them in your Logging solution and you will see all the logs related to that `traceId` from different services.
