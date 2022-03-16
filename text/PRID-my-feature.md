# Title

Tracing for dubbo-go-pixiu

## Summary

Track requests sent to the gateway, record status information, and report it

## Motivation

As the number of services increases and the internal invocation chain becomes 
more complex, it becomes more difficult to troubleshoot problems relying solely 
on logging and performance monitoring. Providing distributed tracking system 
support will help developers visualize disliking request links.

We can send a HTTP request to pixiu gateway,the tracer will automatically trace the
invocations among internal components, such as listener and filter.

When the call is complete, you can open the Jaeger website to see trace details.

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail that:

- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.
- How the feature is used.

### Prepare
1. Jaeger and ZooKeeper are launched together via Docker-compose
2. A new Tracing structure is added to the Bootstrap structure. Tracing is shown as follows:
```go
type Tracing struct {
  Disabled bool
  //默认是HTTP
  ServiceName string
  Driver string
  Sampler Sampler
  Reporter Reporter
}
type Sampler struct {
  Type String
  Param float64
}
type Reporter struct {
  LogSpans bool
  CollectorEndpoint String
}
```
3. These need to be configured in conf.xml at startup time
### Start Jaeger
Initialize trace packages in pkg package, including trace_manager.go, jaeger, 
and api packages respectively. The jaeger package has driver.go,tracer.go(not abstracting span for now).

Add CreateDefaultTracerManager in pixiu_start.go, and provide the default registry to monitor the Tracer of HTTP.

The tracerManager structure is as follows:
```go
type TracerManager struct {
  Driver *jaegerDriver
  bootstrap *model.Bootstrap
}
```
When initializing TracerManager, you need to inject the Bootstrap configuration object and determine 
whether to create the driver.
```go
// 不同listener监听的协议请求不同
type protocolName string

type holder struct {
// key是traceID
tracers map[string]api.Tracer
}
type jagerDriver struct {
tracers map[protocolName] *holder
}

func NewTracer(protocolName string) (api.Tracer, io.Closer, error){
    
}

```
According to api.go, tracer.go within one certain package has StartSpan and GetSpanFromContext methods.

```go
type Tracer struct {
	T opentracing.Tracer
}

func (t *Tracer) StartSpan(ctx context.Context, request interface{}) Span {
  
}

func (t *Tracer) GetSpanFromContext(tx context.Context) Span {
  
}
```

For example, we can start and use span by doing these steps in DefaultHttpWorker's ServeHTTP method:
```go
tracer, closer, _ := NewTracer("HTTP") 
defer closer.Close()
rootSpan := tracer.StartSpan("http-listener")
ctx := tracer.T.ContextWithSpan(r.Context(), rootSpan)
defer rootSpan.Finish()
s.ls.FilterChain.ServeHTTP(w, r.WithContext(ctx))
	
```

## Drawbacks

When a large number of requests come in, we have to store a large number of tracers.

## Alternatives

- Jaeger has a variety of sampling strategies.
- Span creation can be implemented for different request protocols.

## Unresolved questions

Persistence of trace

More request protocol support