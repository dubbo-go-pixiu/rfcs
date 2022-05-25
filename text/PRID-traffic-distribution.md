# Title

Piuxiu provides the ability to distribute traffic

## Summary

The gateway provides the ability to distribute traffic, which makes it easier to build service operation and maintenance capabilities such as canary testing and blue-green release.

## Motivation

It can control the distribution of traffic, perform operations such as cutting flow, and use it for canary release.

### Canary release

After the grayscale release starts, a new version of the application is started first, but the traffic is not directly cut, but the testers test the new version online, and the new version of the application launched is our canary.

### A/B testing

A/B testing routes traffic to new versions based on meta information requested by users, a grayscale publishing strategy based on request content matching. Only requests matching certain rules will be directed to the new version.

### Based on service weight

An integer (0 - 100) percentage of random requests that will be routed to the route specified in the canary. A weight of 0 means that the canary rule will not send any requests to the service in the canary entry. A weight of 100 means that all requests will be sent to the gateway specified in.

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail that:

- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.
- How the feature is used.

### Prepare

1. Configure of the canary

```go
const (
	canaryByHeader CanaryHeaders = "canary-by-header"
	canaryWeight                 = "canary-weight"
	canaryByCookie               = "canary-by-cookie"
)

type (
	CanaryHeaders string

	Headers struct {
		Value string `yaml:"value" json:"value" mapstructure:"value"` // header value
	}

	Config struct {
		CanaryByHeader map[CanaryHeaders]Headers
	}
)
```

2. Implement different strategies for CanaryHeaders

* **canary-by-header** Traffic segmentation based on Request Header
* **canary-weight** Traffic segmentation based on service weight
* **canary-by-cookie** Cookie-based traffic segmentation

## Drawbacks

## Alternatives

## Unresolved questions

