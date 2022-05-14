# Title

Piuxiu provides the ability to distribute traffic

## Summary

The gateway provides the ability to distribute traffic, which makes it easier to build service operation and maintenance capabilities such as canary testing and blue-green release.

## Motivation

It can control the distribution of traffic, perform operations such as cutting flow, and use it for canary release.

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

