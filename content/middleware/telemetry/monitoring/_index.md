---
title: "Monitoring"
draft: true
---

Monitoring helps us understand different insights from our Application.

Similar to Logging, Mortar Monitoring is all about the Interfaces. Implementations can be different

* [Prometheus](https://github.com/go-masonry/bprometheus)
* Datadog
* StatsD

```golang
type Metrics interface {
    // Counter creates or loads a counter with possible predefined tags
    Counter(name, desc string) TagsAwareCounter
    // Gauge creates or loads a gauge with possible predefined tags
    Gauge(name, desc string) TagsAwareGauge
    // Histogram creates or loads a histogram with possible predefined tags
    Histogram(name, desc string, buckets Buckets) TagsAwareHistogram
    // Timer creates or loads a timer with possible predefined tags
    Timer(name, desc string) TagsAwareTimer
    // WithTags sets custom tags to be included if possible in every Metric
    WithTags(tags Tags) Metrics
}
```

When you use an Instance of this interface, Mortar uses an internal cache to remember every metric you create (with Labels/Tags).

{{% alert warning%}}Metric name and labels/tags are used to create a Unique metric ID. Their order is insignificant.{{%/alert%}}

## Usage

Inject, create counter, increment.

```golang
import "github.com/go-masonry/mortar/interfaces/monitor"

type serviceStruct struct {
    fx.In

    Metrics monitor.Metrics `optional:"true"`
}
func (s *serviceStruct) Do(ctx context.Context, req *proto.Request) {
    if s.Metrics != nil {
        counter := s.Metrics.Counter("do_counter","Count number of time Do called")
        counter.Inc()
    }
}
```

**Metrics** variable was injected with an `optional:"true"` tag.
This tells Uber-Fx to ignore it if it wasn't provided, leaving it nil.

In the example above we created a counter `do_counter`, it's a **Singleton**.
Next time when this code will be called or you will create a counter named `do_counter` somewhere else, Mortar will return the same instance.

## Tags/Labels

You can also add labels/tags to every metric.
This is what you need to remember.

* You need to **Declare** Tags with their Default Values, **before** you create a Metric.

    ```golang
    counter := w.deps.Metrics.WithTags(monitor.Tags{
        "color":   "blue", // default static value
        "success": fmt.Sprintf("%t", err == nil), // the value will change every time this called. BUT not the metric name nor tags/labels keys. 
    }).Counter("paint_desired_color_counter", "New paint color for car")
    counter.Add(1)
    ```

    In this case Mortar will create (if not exists) a Counter identified **internally** by `paint_desired_color_counter_color_success`.
* You **shouldn't** dynamically add Tags to a metric after it was created.
  
    {{% alert warning %}}This feature depends on the implementation, Prometheus disallows{{% /alert %}}

    ```golang
     // Counter was created without explicit tags/labels
     counter:=w.deps.Metrics.Counter("accept_car_counter", "Accepting a new car")
     // Don't try to add new Tags to a predefined Metric
     counter.WithTags(monitor.Tags{
      "color": car.GetColor(),
     }).Inc()
    ```

    If you using Prometheus, you will probably get an Error similar to this one:

    ```shell
    1:54PM WRN .../go/pkg/mod/github.com/go-masonry/mortar@v0.2.1/monitoring/types.go:21 > monitoring error error="inconsistent label cardinality: expected 1 label values but got 2 in prometheus.Labels{\"color\":\"blue\", \"service\":\"workshop\"}" app=workshop git=9f00a9c host=Tals-Mac-mini.lan version=v1.2.3
    ```

    The reason is that only one label is expected: `service`.
    It was added as a [static label](#static-tags).

* You can extract values from a Context to override **Declared** Values.

    ```golang
    counter.WithContext(ctx).Inc()
    ```

    For that you need to register Context Extractors for Metrics first, similar to [Logging Context Extractors](/middleware/telemetry/logging/#registering-a-contextextractor).

### Static Tags

You can also add constant/static labels to every metric **Implicitly**.

To do that you need to add them in your [configuration file](https://github.com/go-masonry/mortar/blob/master/interfaces/cfg/keys/keys.go#L89).

Mortar will then [automatically register](https://github.com/go-masonry/mortar/blob/master/constructors/monitor.go#L35) all the tags and will add them to every metric you create.

## Setup

To enable Monitoring you need to do a couple of things

1. Configure your Monitoring implementation of [choice](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/metrics.go#L23).

   ```golang
   // PrometheusBuilder returns a monitor.Builder that is implemented by Prometheus
    func PrometheusBuilder(cfg cfg.Config) monitor.Builder {
        name := cfg.Get(confkeys.ApplicationName).String()
        return bprometheus.Builder().SetNamespace(name)
    }
   ```

2. Create Uber-Fx [options](https://github.com/go-masonry/mortar-demo/blob/master/workshop/app/mortar/metrics.go#L13).

    ```golang
    // PrometheusFxOption registers prometheus
    func PrometheusFxOption() fx.Option {
        return fx.Options(
            providers.MonitorFxOption(),
            // Add a gRPC interceptor that times every gRPC incoming request, their respective metric name is: <application name>_grpc_<method>_bucket
            // Example:
            //  # HELP workshop_grpc_Check time api calls for /mortar.health.v1.Health/Check
            //  # TYPE workshop_grpc_Check histogram
            //  workshop_grpc_Check_bucket{code="0",service="workshop",le="0.005"} 2
            //  workshop_grpc_Check_bucket{code="0",service="workshop",le="0.01"} 2
            providers.MonitorGRPCInterceptorFxOption(), 
            bprometheus.PrometheusInternalHandlerFxOption(), // This one exposes http://localhost:5382/metrics
            fx.Provide(PrometheusBuilder),
        )
    }
    ```

3. [Provide](https://github.com/go-masonry/mortar-demo/blob/master/workshop/main.go#L34) everything.

   ```golang
    fx.New(
        mortar.PrometheusFxOption(),                              // Prometheus
    )
   ```

## Prometheus Implementation

If you chose to use Prometheus you can call `http://localhost:5382/metrics` to get a list of all the known metrics.

```http
GET /metrics HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:5382

# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.6741e-05
go_gc_duration_seconds{quantile="0.25"} 8.1574e-05
go_gc_duration_seconds{quantile="0.5"} 0.000104209
go_gc_duration_seconds{quantile="0.75"} 0.000121561
go_gc_duration_seconds{quantile="1"} 0.000194707
go_gc_duration_seconds_sum 0.005753907
go_gc_duration_seconds_count 56
...
```
