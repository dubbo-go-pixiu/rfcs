# Title

- RFC PR: https://github.com/dubbo-go-pixiu/rfcs/pulls
- Tracking Issue: https://github.com/apache/dubbo-go-pixiu/issues/433

## Summary

The purpose of this project is to provides unified metrics metric calculation and escalation implementation and add calculations for simple metrics such as request processing latency, request processing QPS, and so on for Pixiu.

## Motivation

##### Purpose：

Provide a unified implementation of metrics calculation and reporting, and add simple metrics such as request processing delay and request processing QPS to Pixiu.

##### Expected outcome：

Added simple metrics such as request processing delay, request processing QPS and other simple metrics to Pixiu, and provided a unified metrics metrics calculation and reporting implementation


## Detailed design

This is the bulk of the RFC. Explain the design in enough detail that:

- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.
- How the feature is used.

##### Feature  implementation：

- Main logic: as an intermediate component to collect, define, calculate, externally push data indicators, and then regularly pull data.
- Main process: monitoring indicator data calculation -> monitoring indicator data collection -> monitoring indicator data reporting.

##### Corner cases dissected：

```
var (
	metricServerReqDur = metric.NewHistogramVec(&metric.HistogramVecOpts{
        ...
        // 监控指标
        // 直方图分布中，统计的桶
        Labels:
        []string{"path"},Buckets:
        []float64{5, 10, 25, 50, 100, 250, 500, 1000},
        })
	metricServerReqCodeTotal = metric.NewCounterVec(&metric.CounterVecOpts{
	...
	//
    监控指标：直接在记录指标 incr() 即可
        Labels:  []string{"path", "code"},
        })
    )
    func PromethousHandler(path string) func(http.Handler) http.Handler {
    		return func(next http.Handler) http.Handler {
    			return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                    //请求进入的时间
                    startTime := timex.Now()
                	cw := &security.WithCodeResponseWriter{Writer: w}
                    defer func() {
            //请求返回的时间
            metricServerReqDur.Observe(int64(timex.Since(startTime)/time.Millisecond), path)
            metricServerReqCodeTotal.Inc(path, strconv.Itoa(cw.Code))
            }()
        //中间件放行，执行完后续中间件和业务逻辑。重新回到这，做一个完整请求的指标上报
        // [ 🧅 ：洋葱模型 ]

            next.ServeHTTP(cw, r)
        })
    }
}
```

#### Feature is used：

`metrics collection`    `metrics calculation`  `metrics reporting`  For these three services ， each individual service provides an external interface for invocation. 

## Drawbacks

Design patterns adopted throughout the project——Chain of Responsibility

1. The **Handler** declares the interface, common for all concrete handlers. It usually contains just a single method for handling requests, but sometimes it may also have another method for setting the next handler on the chain.
2. The **Base Handler** is an optional class where you can put the boilerplate code that’s common to all handler classes. Usually, this class defines a field for storing a reference to the next handler. The clients can build a chain by passing a handler to the constructor or setter of the previous handler. The class may also implement the default handling behavior: it can pass execution to the next handler after checking for its existence.
3. **Concrete Handlers** contain the actual code for processing requests. Upon receiving a request, each handler must decide whether to process it and, additionally, whether to pass it along the chain. Handlers are usually self-contained and immutable, accepting all necessary data just once via the constructor.
4. The **Client** may compose chains just once or compose them dynamically, depending on the application’s logic. Note that a request can be sent to any handler in the chain—it doesn’t have to be the first one.

## Alternatives

- Why is this design the best in the space of possible designs?
  -  The collected data will be passed along a processing chain.
  - Handler:  A interface declares  for common for all concrete handlers.
  - Base Handler ：The clients can build a chain by passing a handler to the constructor or setter of the previous handler. The class may also implement the default handling behavior: it can pass execution to the next handler after checking for its existence.
  - Concrete Handlers：the actual code for processing requests
  - Client :  may compose chains just once or compose them dynamically.
- What other designs have been considered and what is the rationale for not
  choosing them?
  - Our program is expected to process different kinds of requests in various ways, but the exact types of requests and their sequences are unknown beforehand.
  -  Since we can link the handlers in the chain in any order, all requests will get through the chain exactly as you planned.
  -  If we provide setters for a reference field inside the handler classes, you’ll be able to insert, remove or reorder handlers dynamically. If  the set of handlers and their order are supposed to change at runtime,this design is better.

## Unresolved questions

- metrics data reporting  strategy
  - The timeliness of data reporting is crucial. How to ensure the timeliness of data reporting is very important. Real-time reporting may result in performance degradation. A good strategy is needed to balance performance and timeliness of data reporting.
  - After the event is triggered, it may not be reported immediately, but the event can be cached first, and then reported after meeting certain conditions.Whether the current sending policy is complied with and whether the last sent time interval is greater than the specified time interval Whether the number of cached events is greater than the maximum number of cached events.

