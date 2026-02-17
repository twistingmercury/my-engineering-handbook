---
title: "Observability: Metrics"
description: "Prometheus metrics implementation covering RED/USE methods, when to instrument, and avoiding cardinality pitfalls"
category: "Observability"
tags: ["metrics", "prometheus", "RED-method", "USE-method", "cardinality", "alerting", "dashboards"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0.0"
revision_history:
  - date: "2024-11-11"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Observability: Metrics

## Other Resources

- [Prometheus Best Practices](https://prometheus.io/docs/practices/naming/)
- [The RED Method](https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/)
- [The USE Method](https://www.brendangregg.com/usemethod.html)

## What Are Metrics?

Metrics are numerical measurements taken over time that tell you how your system is performing right now and how it's trending.

Think of metrics as your dashboard - they answer questions like:

- How many requests per second are we handling?
- What's the 95th percentile response time?
- How much memory are we using?
- What's our error rate?
- How deep is the queue?

Unlike logs (which capture discrete events) and traces (which follow individual requests), metrics aggregate data into time-series. They're designed for dashboards, alerts, and spotting trends.

## When to Expose Metrics

Not every application needs metrics. Here's when you should expose them and when you shouldn't.

### Expose Metrics For:

**Hosted services** - APIs, web applications, microservices running 24/7

- Request rates and response times
- Error rates and types
- Resource utilization
- Business KPIs

**Worker processes** - Queue consumers, batch jobs, background processors

- Queue depth and processing rate
- Job duration and success/failure rates
- Throughput and backlog
- Resource consumption

**Databases and data stores** - Connection pools, caches, message queues

- Connection pool usage
- Query performance
- Cache hit rates
- Lock contention

### Don't Bother For:

**CLI tools** - They run for seconds and then exit. Nothing to monitor.

**PoCs and prototypes** - They're temporary. Focus on proving the concept first.

**One-off scripts** - They're not long-running processes.

**Development/test utilities** - They're not production workloads.

The rule of thumb: if it runs continuously and serves production traffic, it needs metrics.

## What to Measure

Don't try to measure everything - focus on what matters. Two proven approaches help you decide what to instrument.

### The RED Method (Request-Driven Services)

For services that handle requests (APIs, web apps), measure:

- **Rate** - Requests per second (how busy are we?)
- **Errors** - Failed requests per second (what's breaking?)
- **Duration** - Response time distribution (how fast are we?)

These three metrics give you a complete picture of service health.

### The USE Method (Resource Monitoring)

For system resources (CPU, memory, disk, network), measure:

- **Utilization** - Percentage of resource capacity used
- **Saturation** - How much work is queued waiting for the resource
- **Errors** - Resource-related errors (OOM kills, disk errors, etc.)

### Business Metrics

Don't forget the metrics that matter to the business:

- Active users or sessions
- Transactions processed
- Revenue per hour
- Conversion rates
- Feature usage

These help you understand if technical performance translates to business value.

## Prometheus Implementation

Prometheus has become the de facto standard for metrics in containerized environments. Here's how to implement it correctly.

### The /metrics Endpoint

Expose metrics at `/metrics` in Prometheus exposition format:

```text
# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",status="200"} 1234
http_requests_total{method="POST",status="201"} 567

# HELP http_request_duration_seconds HTTP request duration
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.1"} 100
http_request_duration_seconds_bucket{le="0.5"} 450
http_request_duration_seconds_bucket{le="1.0"} 980
http_request_duration_seconds_sum 523.4
http_request_duration_seconds_count 1000
```

### Metric Types

Choose the right type for what you're measuring:

**Counter** - A value that only goes up (requests handled, errors encountered)

- Use for: request counts, error counts, bytes transferred
- Never use for: things that can decrease

**Gauge** - A value that can go up or down (current memory usage, active connections)

- Use for: temperatures, queue depth, current resource usage
- Can be set to any value at any time

**Histogram** - Samples observations and counts them in buckets (request duration, response size)

- Use for: response times, request sizes
- Automatically provides count, sum, and quantiles

**Summary** - Similar to histogram but calculates quantiles on the client side

- Use sparingly - histograms are usually better
- More expensive computationally

### Naming Conventions

Follow Prometheus naming conventions to keep metrics consistent:

- Use `snake_case` for metric names
- Start with the application or domain name: `http_requests_total`, `queue_depth`
- End counters with `_total`: `http_requests_total`, `errors_total`
- End duration metrics with `_seconds`: `request_duration_seconds`
- End size metrics with appropriate units: `response_size_bytes`

**Good names:**

- `api_http_requests_total`
- `worker_job_duration_seconds`
- `cache_hits_total`

**Bad names:**

- `RequestCount` (use snake_case)
- `duration` (too vague, missing unit)
- `http_requests` (counters should end with `_total`)

### Labels: Use Wisely

Labels let you slice metrics by dimensions (method, status, endpoint). But be careful - every unique combination of labels creates a new time series.

**Good label usage:**

```text
http_requests_total{method="GET", status="200", endpoint="/api/users"}
```

**Dangerous label usage (cardinality explosion):**

```text
http_requests_total{user_id="12345"}  # Don't do this!
```

If you have 10,000 users, you just created 10,000 time series for one metric. That's expensive and will kill your Prometheus server.

**Cardinality rules:**

- Labels should have bounded, finite values
- Never use IDs or unbounded strings as labels
- Typical safe labels: method, status code, endpoint (limited set), service name
- Dangerous labels: user IDs, email addresses, session IDs, request IDs

## How Metrics Complement Logs and Traces

Metrics, logs, and traces each serve different purposes. Use them together for complete observability.

**Metrics answer "What's happening now?"**

- Current request rate, error rate, latency
- Resource utilization trends
- System health at a glance
- Trigger alerts when thresholds are crossed

**Logs answer "What went wrong?"**

- Specific errors and exceptions
- Anomalies that need investigation
- Business rule violations
- Audit trail of important events

**Traces answer "Where did it go?"**

- Complete request journey through services
- Which service is slow?
- Where did the error originate?
- Dependencies and timing breakdown

### Example Investigation Workflow

1. **Alert fires** (from metrics): "Error rate > 5%"
2. **Check metrics dashboard**: Which endpoint? What status codes?
3. **Search logs**: What errors are being logged? Any patterns?
4. **Pull traces**: For failed requests, what's the full story?
5. **Root cause found**: Database connection timeout on specific query

Each layer provides different insight. You need all three.

## Common Pitfalls

### Cardinality Explosion

This is the #1 way to kill your metrics system.

**The problem:** Every unique label combination creates a new time series. Add a label with 1,000,000 possible values? That's 1,000,000 time series from a single metric -- and your Prometheus server will not be happy about it.

**The solution:**

- Keep label cardinality bounded (status codes: ~20 values, not user IDs: 1,000,000 values)
- Use sampling or aggregation for high-cardinality data
- Monitor your metrics system's resource usage

### Over-Instrumenting

Don't measure everything just because you can. Every metric has a cost:

- Memory in your application
- Network bandwidth to send metrics
- Storage in Prometheus
- Query time on dashboards

**Start with RED/USE**, then add business metrics. Add more only when you have specific questions to answer.

> **Coverage vs. Depth:** While you should avoid over-instrumenting individual services, every production service needs core metrics. See [Scale & High Availability](./scale-and-high-availability.md) for why monitoring coverage matters. The goal is complete coverage with focused instrumentation—not metrics everywhere or metrics nowhere.

### Under-Instrumenting

The opposite problem: you don't have the metrics you need when something breaks.

**Critical metrics you can't skip:**

- Request rate and error rate (for services)
- Response time distribution (p50, p95, p99)
- Resource usage (CPU, memory, connections)
- Queue depth (for async systems)

If you're oncall and can't answer "Is the service healthy?" from your dashboards, you're under-instrumented.

### Forgetting About Storage Costs

Metrics aren't free. With default Prometheus settings:

- 1,000 active time series = ~1-2 MB/hour of storage
- 1,000,000 active time series = ~1-2 GB/hour of storage

High cardinality labels can quickly make metrics expensive. Monitor your time series count and set retention policies appropriately.

---

[← Previous: Observability: Distributed Tracing](./observability-distributed-tracing.md) | [↑ Back to Home](index.md) | [Next: Observability: Health Checks →](./observability-heartbeats-readiness-liveness.md)
