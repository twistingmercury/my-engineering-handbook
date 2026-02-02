---
title: "Observability: Distributed Tracing"
description: "OpenTelemetry implementation guide for tracking requests across microservices with sampling strategies and correlation"
category: "Observability"
tags:
  [
    "distributed-tracing",
    "opentelemetry",
    "spans",
    "sampling",
    "correlation-ids",
    "microservices",
  ]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
revision_history:
  - date: "2026-01-30"
    author: "Jeremy K. Johnson"
    changes: "Added Trace Context section defining OpenTelemetry trace_id and span_id as correlation standard; updated correlation mechanism explanation"
  - date: "2024-11-11"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Observability: Distributed Tracing

## Other Resources

[OpenTelemetry Instrumentation Guides](https://opentelemetry.io/docs/instrumentation/)

## What is Distributed Tracing?

We rarely have an isolated singular service. Services use other services; a single user request might touch 5, 10, or even 20 different services. Distributed tracing tracks that request's entire journey, showing you exactly where time is spent and where things go wrong. Think of it like GPS tracking for your requests - you can see the complete path from start to finish, including any detours or traffic jams along the way. Without distributed tracing, debugging microservices is like trying to solve a mystery with half the clues missing. You might know something went wrong, but good luck figuring out where or why.

Distributed tracing gives you:

- **End-to-end visibility**: See exactly how requests flow through your system
- **Performance insights**: Spot slow services, database calls, or network issues
- **Error context**: When something breaks, see exactly where and what caused it
- **Dependency mapping**: Understand which services depend on what

For tracing to work, each service needs to pass the trace context to the next service. This happens automatically with most modern tracing libraries - they add headers to HTTP requests, message queue messages, etc.

## Trace Context

OpenTelemetry uses two core identifiers to correlate distributed requests:

- **`trace_id`**: A unique identifier for the entire request journey (32 lowercase hex characters)
- **`span_id`**: A unique identifier for each operation within that trace (16 lowercase hex characters)

These ARE your correlation IDs. They tie logs, traces, and metrics together across all services involved in a request.

OpenTelemetry follows the [W3C Trace Context](https://www.w3.org/TR/trace-context/) standard for propagation. Services pass trace context via the `traceparent` HTTP header using the format:

```text
{version}-{trace_id}-{span_id}-{trace_flags}
```

Example:

```text
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

When a service receives this header, it continues the same trace by creating child spans with the same `trace_id` but new `span_id` values. This creates the parent-child relationship that builds your complete trace tree.

For more details on OpenTelemetry traces, see the [OpenTelemetry Traces documentation](https://opentelemetry.io/docs/concepts/signals/traces/).

## What to Trace

### Always Trace

- **External calls**: HTTP requests, database queries, message publishing
- **Service boundaries**: Incoming requests and outgoing responses
- **Critical business operations**: Payment processing, user authentication, etc.

### Consider Tracing

- **Expensive operations**: File I/O, complex calculations, third-party API calls
- **Error-prone areas**: Places where things often go wrong
- **Performance bottlenecks**: Operations you're trying to optimize

#### Defining "Expensive Operations"

What counts as "expensive" varies by operation type. These thresholds are industry-standard guidelines to help you decide when custom tracing adds value:

| Operation Type             | Threshold    | Source                                                                                                                 |
| -------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------- |
| User-facing API response   | >200ms       | [Google SRE Workbook](https://sre.google/workbook/implementing-slos/)                                                  |
| Database query             | >100ms       | [PostgreSQL Wiki](https://wiki.postgresql.org/wiki/Logging_Difficult_Queries)                                          |
| Cache operation (Redis)    | >10ms        | [Redis Latency Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/latency-monitor/)      |
| External API call          | >500ms       | [AWS Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/framework/ops_observability_dist_trace.html) |
| Memory-intensive operation | >50MB        | Industry practice                                                                                                      |
| All errors                 | Always trace | [OpenTelemetry Best Practices](https://opentelemetry.io/docs/concepts/sampling/)                                       |

These are starting points, not rigid rules. Adjust based on your service's performance profile and user expectations. A background batch job might have different thresholds than a real-time API.

### Don't Over-Trace

Tracing adds overhead, so be selective. You don't need to trace every function call - focus on the operations that matter for debugging and performance monitoring.

## Implementation with OpenTelemetry

OpenTelemetry is the industry standard for distributed tracing. It works with all major programming languages and integrates with most observability platforms.

### Auto-Instrumentation

Most languages offer auto-instrumentation that automatically traces common operations like:

- HTTP requests (both incoming and outgoing)
- Database calls
- Message queue operations
- Popular framework operations

Start with auto-instrumentation - it gives you 80% of what you need with minimal effort.

### Custom Instrumentation

For business-specific operations, add custom spans around important code sections. This is where you'll trace things like:

- Complex business logic
- Internal service calls
- Custom integrations

### Sampling

Don't trace every single request - it's expensive and usually unnecessary. Use sampling to trace a representative subset:

- **Head-based sampling**: Decide at the start of a trace
- **Tail-based sampling**: Decide after seeing the complete trace (useful for keeping all error traces)

Start with a 10% sampling and adjust based on your traffic volume and needs.

**Sampling Rate Adjustment Criteria:**

- **10% sampling**: Default for traffic under 1,000 requests per minute
- **1-5% sampling**: For traffic between 1,000-10,000 requests per minute
- **0.1-1% sampling**: For traffic above 10,000 requests per minute
- **100% error traces**: Always capture traces that contain errors regardless of sampling rate

**Review Cadence:**

- Review sampling rates monthly under normal operations
- After a release, review sampling effectiveness daily for 5 days to ensure you're capturing enough data to identify issues without overwhelming your tracing infrastructure

### Span Naming

- Use descriptive, consistent names: `GET /users/{id}` not `GET /users/12345`
- Include the operation type: `db.query.select_user`
- Keep names stable - don't include dynamic values

## How Tracing and Logging Work Together

Tracing and logging serve different purposes and complement each other:

**Traces are for normal operations.** When everything works as expected, traces capture the complete request flow: which services were called, how long each step took, what dependencies were involved. This is your "everything worked" data.

**Logs are for anomalies.** When something unusual happens - a business rule violation, an approaching resource limit, an unexpected error - logs surface it for human attention.

This is why we set production log level to `Warn` instead of `Info`. We don't need to log every successful request because traces already capture that information. Logs focus on deviations from normal operation.

**Trace context ties them together.** When a trace is active, your logs should include the `trace_id` and `span_id` fields. This correlation mechanism lets you:

1. Find the relevant log entry (the anomaly that needs attention)
2. Pull up the associated trace using the `trace_id` (the full context of what happened)
3. Understand both what went wrong and why

Most OpenTelemetry SDKs automatically inject `trace_id` and `span_id` into your logging context when using structured logging. See the [Observability: Logging](observability-logging.md) guide for details on structured log format and trace correlation.

This approach gives you complete observability while keeping costs reasonable. You're not paying to store logs for millions of successful requests when traces already capture that data more efficiently.

