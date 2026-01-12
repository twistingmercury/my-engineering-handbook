---
title: "Observability: Logging"
description: "Structured logging standards, log levels, and what to log for effective debugging without exposing sensitive data"
category: "Observability"
tags:
  [
    "logging",
    "structured-logging",
    "log-levels",
    "stdout",
    "correlation-ids",
    "PII",
    "PHI",
  ]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
date: "2024-11-11"
revision_history:
  - date: "2024-11-11"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Observability: Logging

## What to Log

Logging is one of those things that seems simple until you're debugging a production incident at 2 AM and realize your logs are useless.

Good logging requires consistency - in field names, detail levels, and structure. Without it, debugging becomes expensive and automation becomes impossible.

Some people advocate "log everything!" That sounds great until you're drowning in noise and paying for storage you don't need. Instead, be strategic about what you log and what you don't.

### Context is Everything

Compare these two log entries:

- `user login failed`
- `{"user_id":"jdoe", "event":"login", "level":"info", "timestamp":"2025-11-11T14:23:45Z", "error":"invalid_password"}`

The second one tells a story. You know who, what, when, and why. The first one just tells you something went wrong somewhere for someone.

Adding context helps you piece together what happened. But here's the catch: without unique identifiers (request IDs, trace IDs, correlation IDs), you'll get lost trying to track actions across services. In a microservice architecture, correlation is everything.

## Recommendations

### 1. Log to stdout

For containerized applications, always log to `stdout`. Let your container orchestration platform handle log collection and routing. Don't try to manage log files inside containers.

### 2. Use Structured Logging

Treat logs as operational data, not just debugging output. Use JSON or another structured format that you can query and analyze.

Structured logs enable:

- Filtering by specific fields
- Automated alerting
- Performance analysis
- Trend detection

### 3. Use the Right Log Levels

Not everything deserves the same level of attention. Here's how we use log levels:

- **Debug** - Low-level diagnostic information for development and troubleshooting
- **Info** - Normal business events (user logged in, order processed, etc.)
- **Warn** - System working correctly, but something unusual happened that might need attention
- **Error** - Something failed, but we're attempting recovery
- **Critical** - Fatal errors that prevent the application from starting or functioning

**Production default:** Set production logging to `Warn` level.

Why `Warn` instead of `Info`? Because we use distributed tracing to capture normal operational flow. Traces show you the complete request journey with timing, dependencies, and context. They're designed for "everything worked" scenarios.

Logs at `Warn` and above surface the exceptions - things that deviate from normal operation and actually need human attention. This keeps your logs focused on actionable information instead of drowning in noise about successful operations.

The combination gives you complete observability:

- **Traces** (with sampling) capture detailed request flows and performance data
- **Correlation IDs** tie logs and traces together when you need to investigate
- **Warn/Error/Critical logs** highlight anomalies and failures that need attention

You get full visibility without paying to store logs for millions of successful requests.

### 4. Include Standard Fields

Every log entry should include:

- **timestamp** (UTC in ISO 8601 format)
- **severity** (debug, info, warn, error, critical)
- **action** (what was happening)
- **component** (which service/module)
- **method** (which function)
- **error info** (if applicable)
- **input/output values** (sanitized - no PII/PHI)
- **duration** (for operations that take time)

### A Warning About Warnings

Don't overuse the `warn` level. Reserve it for things that actually need investigation to keep the system healthy - like invalid request content or approaching resource limits.

If it doesn't need investigation, log it as `info`. Too many warnings create noise, and eventually people stop paying attention to them.

## What Information to Capture

Use common sense, but follow these guidelines when deciding what to log.

### Always Log These Events

- Application startup and initialization
- Configuration parameters (version, build tag, environment)
- Configuration errors
- All `error` and `critical` level events (never suppress these)
- Shutdown requests and graceful shutdowns
- Configuration changes (especially at runtime)

### Always Include

- **When** - Timestamp in UTC (ISO 8601 format)
- **What** - Event name and severity level
- **Where** - Hostname/container ID and component/service name
- **How long** - Duration for any operation that takes meaningful time

### Include When Relevant

- **Database operations** - Connection info, query types (not full queries), result counts
- **Message queue operations** - Topics, queues, producer/consumer IDs, message counts
- **Errors** - Error type, message, stack trace (sanitized)
- **User actions** - User ID, session ID, correlation ID (no personal information)

### Never Include

This is non-negotiable:

- **PII/PHI** - Names, addresses, SSNs, medical records, phone numbers
- **Secrets** - Passwords, API keys, tokens, certificates
- **Sensitive business data** - Account numbers, credit cards, financial details
- **Full request/response bodies** - They often contain sensitive data

When in doubt, don't log it. You can always add more logging later, but you can't un-log sensitive data that's already in production.
