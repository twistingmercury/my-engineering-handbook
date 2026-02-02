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
- `{"user_id":"jdoe", "event":"login", "level":"warn", "timestamp":"2025-11-11T14:23:45Z", "trace_id":"4bf92f3577b34da6a3ce929d0e0e4736", "span_id":"00f067aa0ba902b7", "error":"invalid_password"}`

The second one tells a story. You know who, what, when, and why. The first one just tells you something went wrong somewhere for someone.

Adding context helps you piece together what happened. But here's the catch: without unique identifiers, you'll get lost trying to track actions across services. In a microservice architecture, correlation is everything.

The OpenTelemetry `trace_id` serves as the primary correlation identifier - it's shared across all spans in a distributed trace, allowing you to track a request's journey through your entire system. The `span_id` identifies the specific operation within that trace. Together, they link your logs to distributed traces for complete request visibility.

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
- **Trace context** (`trace_id` and `span_id`) ties logs and traces together when you need to investigate
- **Warn/Error/Critical logs** highlight anomalies and failures that need attention

You get full visibility without paying to store logs for millions of successful requests.

### 4. Include Standard Fields

Every log entry should include:

- **timestamp** (UTC in ISO 8601 format)
- **severity** (debug, info, warn, error, critical)
- **action** (what was happening)
- **component** (which service/module)
- **method** (which function)
- **trace_id** (32 lowercase hex characters - when trace context is active)
- **span_id** (16 lowercase hex characters - when trace context is active)
- **error info** (if applicable)
- **input/output values** (sanitized - no PII/PHI)
- **duration** (for operations that take time)

The `trace_id` and `span_id` fields enable correlation with distributed traces per the OpenTelemetry/W3C Trace Context standard. Include these fields whenever trace context is active to link logs with their corresponding trace spans. See the distributed tracing documentation for full trace context details.

### A Warning About Warnings

Don't overuse the `warn` level. Reserve it for things that actually need investigation to keep the system healthy - like invalid request content or approaching resource limits.

If it doesn't need investigation, log it as `info`. Too many warnings create noise, and eventually people stop paying attention to them.

#### Resource Limit Thresholds

When logging warnings about resource limits, use these graduated severity thresholds:

- **75% utilization** = `Warn` - Approaching limit, investigate soon
- **80% utilization** = `Error` - Critical threshold, action needed
- **90% utilization** = `Critical` - Imminent failure risk

Apply these thresholds to:

- **Memory** - Container or process memory limit
- **CPU** - CPU quota measured over 5-minute average
- **Disk** - Volume capacity for persistent storage
- **Database connections** - Connection pool size
- **File descriptors** - Process ulimit for open files

**Logging vs. Alerting**: Log immediately when crossing these thresholds—this captures transient spikes for forensic analysis. However, only trigger alerts or page on-call if the condition is sustained for 5+ minutes. Brief spikes that resolve quickly shouldn't wake anyone up.

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
- **User actions** - Internal user ID (UUIDs, database IDs), session ID (never log the personal information these IDs reference: usernames, emails, SSNs, etc.)

#### Sanitizing Stack Traces

When logging stack traces, "sanitized" means removing information that could expose system internals or sensitive data while keeping the diagnostic value.

**Remove from stack traces:**

- Absolute file paths - Use relative paths from project root instead (e.g., `src/handlers/auth.go:42` instead of `/home/jsmith/company-app/src/handlers/auth.go:42`)
- Usernames and home directory paths - Replace with generic markers (e.g., `/home/jsmith/` becomes `$HOME/`)
- Environment variable values that appear in stack context
- Memory addresses - These provide minimal debugging value and can expose internal state

**Keep in stack traces:**

- Function and method names
- Line numbers
- Error messages
- Relative file paths from project root
- Call hierarchy showing the execution flow

**Additional guidance:**

- Truncate stack traces to 50 frames maximum to prevent log bloat
- Consider using structured logging libraries that handle sanitization automatically
- For Go, libraries like `github.com/pkg/errors` provide stack traces that are easier to sanitize programmatically
- Always test your sanitization logic to ensure it doesn't accidentally remove critical debugging information

### Never Include

This is non-negotiable:

- **PII/PHI** - Names, addresses, SSNs, medical records, phone numbers
- **Secrets** - Passwords, API keys, tokens, certificates
- **Sensitive business data** - Account numbers, credit cards, financial details
- **Full request/response bodies** - They often contain sensitive data

For comprehensive data protection guidelines, see [Data Protection](./security-in-development.md#data-protection).

When in doubt, don't log it. You can always add more logging later, but you can't un-log sensitive data that's already in production.

## Operational Log Review

Logs are only valuable if someone looks at them. On-call engineers should review logs daily as part of standard operational duties.

**Daily review focus areas:**

- **Error and Critical logs** - Investigate all errors and critical events, even if they didn't trigger alerts
- **Warning patterns** - Look for repeated warnings that might indicate emerging problems
- **Anomalies** - Unusual patterns in log volume, timing, or content
- **Resource warnings** - Memory, CPU, disk, or connection pool warnings that could indicate capacity issues

This proactive review often catches problems before they become incidents. It also builds operational awareness of normal vs. abnormal system behavior.

**How to review efficiently:**

- Start with the highest severity logs (Critical, then Error, then Warn)
- Look for patterns across services, not just individual log entries
- Use your log aggregation tool's filtering and grouping features to identify trends
- Document any findings or concerns in your team's runbook or incident tracking system

Daily log review takes 15-30 minutes and significantly improves system reliability by catching issues early. Teams that practice this consistently see far fewer midnight and early morning fires—problems get addressed during business hours before they escalate.
