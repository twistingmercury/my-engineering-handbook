---
title: "Observability: Distributed Tracing"
description: "OpenTelemetry implementation guide for tracking requests across microservices with sampling strategies and correlation"
category: "Observability"
tags: ["distributed-tracing", "opentelemetry", "spans", "sampling", "correlation-ids", "microservices"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
date: "2024-11-11"
revision_history:
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

## What to Trace

### Always Trace

- **External calls**: HTTP requests, database queries, message publishing
- **Service boundaries**: Incoming requests and outgoing responses
- **Critical business operations**: Payment processing, user authentication, etc.

### Consider Tracing

- **Expensive operations**: File I/O, complex calculations, third-party API calls
- **Error-prone areas**: Places where things often go wrong
- **Performance bottlenecks**: Operations you're trying to optimize

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

### Span Naming

- Use descriptive, consistent names: `GET /users/{id}` not `GET /users/12345`
- Include the operation type: `db.query.select_user`
- Keep names stable - don't include dynamic values

## How Tracing and Logging Work Together

Tracing and logging serve different purposes and complement each other:

**Traces are for normal operations.** When everything works as expected, traces capture the complete request flow: which services were called, how long each step took, what dependencies were involved. This is your "everything worked" data.

**Logs are for anomalies.** When something unusual happens - a business rule violation, an approaching resource limit, an unexpected error - logs surface it for human attention.

This is why we set production log level to `Warn` instead of `Info`. We don't need to log every successful request because traces already capture that information. Logs focus on deviations from normal operation.

**Correlation IDs tie them together.** When you need to investigate an issue, correlation IDs let you:

1. Find the relevant log entry (the anomaly that needs attention)
2. Pull up the associated trace (the full context of what happened)
3. Understand both what went wrong and why

This approach gives you complete observability while keeping costs reasonable. You're not paying to store logs for millions of successful requests when traces already capture that data more efficiently.

## Revision History

