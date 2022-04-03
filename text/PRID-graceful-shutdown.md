# Title

Gracefully shuts down for dubbo-go-pixiu

## Summary

Make sure the pixiu can be shutdown gracefully

## Motivation

Minimize traffic loss when Pixiu is closing

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail that:

- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.
- How the feature is used.

### Prepare

1. Configure of the gracefully shuts down

```yaml
  shutdown_config:
    timeout: "10s"
    step_timeout: "10s"
    reject_policy: "immediacy"
```

2. Implement a graceful policy for each listener

* Http/Https listener Http provides a way to get offline gracefully.

```go
timeout, cancelFunc := context.WithTimeout(context.Background(), 10*time.Second)
defer cancelFunc()
//shut down with timeout
err := server.Shutdown(timeout)
//if context exceeded, then force shutdown
if err != nil {
    logger.Error("Http2ListenerService Shutdown error %s", err)
    _ = server.Close()
}
```

* Dubbo/Triple/Grpc listener

1. Create a counter for each listener
2. Counter +1 when requesting, counter -1 when response returned
3. When the graceful shutdown is started, the polling counter is started, and once the counter is 0, the listener should close now
4. if context exceeded, then force shutdown

## Drawbacks

## Alternatives
Listening for shutdown signals

## Unresolved questions
