---
title: "Observability: Heartbeats, Readiness, and Liveness"
description: "Health check endpoint standards for auto-healing, auto-scaling, and Kubernetes readiness/liveness probes"
category: "Observability"
tags:
  [
    "health-checks",
    "heartbeat",
    "kubernetes",
    "readiness",
    "liveness",
    "auto-healing",
    "monitoring",
  ]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
date: "2024-11-11"
revision_history:
  - date: "2024-11-11"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Observability: Heartbeats, Readiness, and Liveness

## Other Resources

- [Kubernetes: Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

## What is a Heartbeat?

Every service needs a way to answer the question "Are you okay?" That's what heartbeat endpoints do.

But a good heartbeat does more than just respond with a 200. It checks that the service can actually do its job - can it reach the database? Are its dependencies available? Is it configured correctly?

This information serves multiple purposes:

- **Auto-healing** - Kubernetes can restart unhealthy containers
- **Auto-scaling** - Only send traffic to healthy instances
- **Troubleshooting** - Ops teams can quickly identify what's broken
- **Alerting** - Automated monitoring can catch problems before users notice

The heartbeat endpoint doubles as readiness and liveness checks in Kubernetes, so design it carefully.

## How to Define the health check

Every service exposes a heartbeat endpoint at `/ops/health`. Here's the standard OpenAPI spec we use:

```yaml
openapi: 3.0.3
info:
  title: Service Heartbeat API
  description: Standard health check endpoint for all services
  version: 1.0.0
paths:
  /ops/health:
    get:
      summary: Get service health status
      description: Returns the current health status of the service and its dependencies
      responses:
        "200":
          description: Service health information
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/HealthResponse"
        "503":
          description: Service unavailable or critical health issue
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/HealthResponse"
components:
  schemas:
    HealthResponse:
      type: object
      required:
        - Status
        - URL
        - Machine
        - UtcDateTime
      properties:
        Status:
          type: string
          enum: [OK, WARNING, ERROR]
          description: Overall health status
        Message:
          type: string
          description: Optional details about current health condition
        URL:
          type: string
          format: uri
          description: The actual endpoint URL that responded
        Machine:
          type: string
          description: Hostname or container name (never IP address)
        UtcDateTime:
          type: string
          format: date-time
          description: ISO 8601 timestamp when response was generated
        RequestDuration:
          type: number
          minimum: 0
          description: How long the health check took (milliseconds)
        Dependencies:
          type: array
          description: Health status of direct dependencies
          items:
            $ref: "#/components/schemas/DependencyHealth"
    DependencyHealth:
      type: object
      required:
        - Status
        - URL
        - UtcDateTime
      properties:
        Status:
          type: string
          enum: [OK, WARNING, CRITICAL]
        URL:
          type: string
          format: uri
        UtcDateTime:
          type: string
          format: date-time
        RequestDuration:
          type: number
          minimum: 0
```
