---
title: "Tooling Standards"
description: "Approved cloud-native technologies and tooling patterns for portable, cloud-agnostic development"
category: "Standards"
tags:
  [
    "tooling",
    "cloud-native",
    "docker",
    "kubernetes",
    "standards",
    "governance",
    "portability",
  ]
audience: "Software Engineers, DevOps Engineers, Architects"
version: "1.0"
date: "2026-02-02"
revision_history:
  - date: "2026-02-02"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Tooling Standards

## Purpose

This document defines approved technologies and patterns for cloud-native development. The goal is clear: build solutions that run anywhere without vendor lock-in.

Our philosophy: **cloud-agnostic, cloud-native**. Choose portable technologies and document patterns, not specific vendor products. If you can't move your workload to another cloud provider in a reasonable timeframe, you're doing it wrong.

## Principles

**Patterns over products**: We document the capabilities you need, not which vendor's product to use. Need a message queue? Great - use Service Bus on Azure, SQS on AWS, or Pub/Sub on GCP. The pattern (asynchronous messaging) matters more than the brand name.

**Portability first**: Technologies in this guide work on any major cloud provider. If a tool only runs on one cloud, it doesn't belong here.

**Managed services preferred**: Let cloud providers handle operational complexity so you can focus on business logic. Self-hosting databases, message queues, or caches should be the exception, not the default. Valid exceptions include: regulatory requirements (data residency), cost-prohibitive managed pricing at scale, or specific performance requirements unavailable in managed offerings. Document exceptions in an ADR.

**Standards-based**: Choose technologies built on open standards and interoperable protocols. Avoid proprietary APIs when standard alternatives exist.

## Core Cloud-Native Technologies

These are specific, approved technologies that work everywhere. They're portable standards, not vendor products.

| Technology        | Purpose                               | Version | Documentation                                        |
| ----------------- | ------------------------------------- | ------- | ---------------------------------------------------- |
| **Docker**        | Container runtime and image format    | 24.0+   | [Docker Docs](https://docs.docker.com/)              |
| **Kubernetes**    | Container orchestration               | 1.28+   | [Kubernetes Docs](https://kubernetes.io/docs/)       |
| **Helm**          | Kubernetes package manager            | 3.12+   | [Helm Docs](https://helm.sh/docs/)                   |
| **Envoy**         | Service mesh proxy and load balancer  | 1.28+   | [Envoy Docs](https://www.envoyproxy.io/docs)         |
| **OpenTelemetry** | Observability (traces, metrics, logs) | 1.0+    | [OpenTelemetry Docs](https://opentelemetry.io/docs/) |
| **OPA**           | Policy enforcement and authorization  | 0.60+   | [OPA Docs](https://www.openpolicyagent.org/docs/)    |

**Docker** is required for all services. It's the container standard - if your code doesn't run in a container, it doesn't deploy to our infrastructure. All services must be containerized and stateless regardless of architecture pattern (microservices or monolith).

**Kubernetes** is our orchestration platform. Whether you're running on AKS (Azure), EKS (AWS), or GKE (Google Cloud), the YAML manifests and Helm charts work the same way.

**Envoy** handles service-to-service communication. Use it as a sidecar proxy for traffic management, load balancing, and observability. It's the data plane for Istio, Consul Connect, and other service meshes.

**OpenTelemetry** is the observability standard. It captures traces, metrics, and logs with vendor-neutral APIs and exports them to any backend (Jaeger, Prometheus, Datadog, etc.). See our observability guides for implementation details.

**OPA** enforces policy decisions. Use it for authorization logic, admission control in Kubernetes, and any decision-making that should be separate from application code.

## Managed Services: Pattern-Based

These are capabilities you should use via managed cloud services. Pick what fits your cloud provider, but the pattern is required.

### Databases

| Pattern                          | When to Use                                            | Examples (not prescriptive)                                         |
| -------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------- |
| **Managed relational database**  | Structured data, ACID transactions, SQL queries        | Azure SQL Database, Amazon RDS (PostgreSQL/MySQL), Google Cloud SQL |
| **Managed document database**    | Semi-structured data, flexible schemas, JSON documents | Azure Cosmos DB, Amazon DocumentDB, Google Firestore                |
| **Managed time-series database** | Metrics, IoT data, high-write workloads                | Azure Data Explorer, Amazon Timestream, Google Cloud Bigtable       |

**Key requirements:**

- Automated backups and point-in-time recovery
- High availability across availability zones
- Encryption at rest and in transit
- Connection pooling support
- Read replicas for scaling read-heavy workloads

Do not self-host databases unless you have a specific technical requirement that managed services can't satisfy. Managing database infrastructure is undifferentiated heavy lifting.

### Messaging

| Pattern               | When to Use                                            | Examples (not prescriptive)                                |
| --------------------- | ------------------------------------------------------ | ---------------------------------------------------------- |
| **Message queue**     | Task distribution, command processing, async workflows | Azure Service Bus, Amazon SQS, Google Cloud Tasks          |
| **Event streaming**   | Event sourcing, real-time analytics, log aggregation   | Azure Event Hubs, Amazon Kinesis, Google Cloud Pub/Sub     |
| **Pub/Sub messaging** | Event broadcasting, fan-out patterns                   | Azure Service Bus Topics, Amazon SNS, Google Cloud Pub/Sub |

**Key requirements:**

- At-least-once delivery guarantees
- Dead-letter queue support
- Message retention for replay
- Ordering guarantees (when needed)
- Idempotent message handling in consumers

See [Scale and High Availability](./scale-and-high-availability.md) for patterns on handling message queue failures and backpressure.

### Secrets Management

| Pattern               | When to Use                                                 | Examples (not prescriptive)                                 |
| --------------------- | ----------------------------------------------------------- | ----------------------------------------------------------- |
| **Managed key vault** | API keys, connection strings, certificates, encryption keys | Azure Key Vault, AWS Secrets Manager, Google Secret Manager |

**Key requirements:**

- Automatic secret rotation support
- Access auditing and logging
- Integration with Kubernetes (CSI driver or external secrets operator)
- Role-based access control (RBAC)
- Encryption key management (customer-managed keys)

Never commit secrets to source control. Never store secrets in environment variables unless they're injected at runtime from a vault. See [Flexible Application Configuration](./flexible-application-configuration.md) and [Security in Development](./security-in-development.md) for implementation guidance.

### Caching

| Pattern           | When to Use                                         | Examples (not prescriptive)                                                 |
| ----------------- | --------------------------------------------------- | --------------------------------------------------------------------------- |
| **Managed Redis** | Session storage, distributed caching, rate limiting | Azure Cache for Redis, Amazon ElastiCache (Redis), Google Cloud Memorystore |

**Key requirements:**

- High availability (replication across zones)
- Persistence options for durability
- TLS encryption for data in transit
- Eviction policies appropriate for use case
- Connection pooling in client libraries

See [Scale and High Availability](./scale-and-high-availability.md) for caching strategies and cache invalidation patterns.

## Development and Collaboration

These are capabilities every team needs. Choose tools that fit your organization, but make sure they provide these patterns.

### Version Control

| Capability Required      | Why                                    | Examples                                      |
| ------------------------ | -------------------------------------- | --------------------------------------------- |
| **Git-based repository** | Source control, branching, code review | GitHub, Bitbucket, Azure DevOps Repos, GitLab |

**Key requirements:**

- Pull request/merge request workflow
- Code review tools and approvals
- Protected branches (main/master)
- Webhook support for CI/CD integration
- Branch policies (require reviews, status checks)

See [Managing Our Source](./managing-our-source.md) for branching strategies and workflow.

### Documentation

| Capability Required                | Why                                               | Examples                                                    |
| ---------------------------------- | ------------------------------------------------- | ----------------------------------------------------------- |
| **Wiki or documentation platform** | Architectural decisions, runbooks, team knowledge | Confluence, Notion, GitBook, GitHub Wiki, Azure DevOps Wiki |

**Key requirements:**

- Search functionality
- Version history
- Access control (public vs. team vs. private)
- Markdown support
- Link validation

See [Importance of Documentation](./importance-of-documentation.md) for documentation standards.

### Issue Tracking

| Capability Required            | Why                                               | Examples                                                 |
| ------------------------------ | ------------------------------------------------- | -------------------------------------------------------- |
| **Issue and project tracking** | Feature planning, bug tracking, sprint management | Jira, GitHub Issues, Azure Boards, Linear, GitLab Issues |

**Key requirements:**

- Backlog management
- Sprint/iteration planning
- Integration with VCS (link commits to issues)
- Custom workflows (if needed)
- Labels/tags for categorization

### CI/CD Pipeline

| Capability Required     | Why                            | Examples                                                |
| ----------------------- | ------------------------------ | ------------------------------------------------------- |
| **Pipeline automation** | Build, test, deploy automation | GitHub Actions, Azure Pipelines, GitLab CI/CD, CircleCI |

**Key requirements:**

- Triggered by source control events (push, PR)
- Matrix builds (test multiple versions/platforms)
- Artifact storage and versioning
- Deployment approvals and gates
- Secret injection (vault integration)

See [Builds and Deployments](./builds-and-deployments.md) for pipeline patterns and [Versioning Our Solutions](./versioning-our-solutions.md) for artifact versioning.

### Metrics and Monitoring

| Capability Required               | Why                                       | Examples                                                                           |
| --------------------------------- | ----------------------------------------- | ---------------------------------------------------------------------------------- |
| **Prometheus-compatible metrics** | Application and infrastructure monitoring | Prometheus, Datadog, Grafana Cloud, Azure Monitor, AWS CloudWatch (with exporters) |

**Key requirements:**

- PromQL query language support (or equivalent)
- Alerting rules and notification routing
- Dashboard creation and templating
- Long-term metrics storage
- Integration with OpenTelemetry

See [Observability: Metrics](./observability-metrics.md) for instrumentation patterns and [Observability: Distributed Tracing](./observability-distributed-tracing.md) for OpenTelemetry implementation.

## Local Development Tools

Developers need minimal tooling to build and test locally. These tools should be installed on every development machine.

**Required tools:**

- **Docker Desktop** or **Podman**: Container runtime for local development and testing
- **kubectl**: Kubernetes CLI for interacting with clusters
- **Helm**: Kubernetes package manager for deploying charts
- **Git**: Version control client
- **Language-specific SDK**: Go 1.21+, .NET 8+, Node.js 20+, Python 3.11+ (as needed)

**Recommended tools:**

- **k9s** or **Lens**: Kubernetes cluster UI/TUI for easier debugging
- **jq**: JSON processing for working with API responses and logs
- **curl** or **httpie**: HTTP client for API testing
- **yq**: YAML processing for working with Kubernetes manifests

**Local environment:**

Use Docker Compose for running dependencies locally (databases, message queues, caches). Every repository should include a `docker-compose.yml` file that spins up required services with sensible defaults.

Example local stack:

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: localdev
      POSTGRES_USER: myapp
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  azurite: # Azure Storage emulator
    image: mcr.microsoft.com/azure-storage/azurite
    ports:
      - "10000:10000" # Blob
      - "10001:10001" # Queue
```

Developers should be able to run `docker compose up` and have a working local environment. Configuration should use environment variables or local config files (never committed) that point to these local services.

## Tool Governance

Technology choices aren't permanent. Tools move through lifecycle states based on adoption, support, and business needs.

### Lifecycle States

**Evaluating**: We're testing this technology on non-critical projects to see if it fits our needs.

- No production usage yet
- Spike projects and proof-of-concepts only
- Document findings and decision criteria
- Time-boxed evaluation period (typically 3-6 months)

**Approved**: This technology is vetted and ready for production use.

- Documented in this guide
- Support and training available
- Used in production by at least one team
- Maintained by vendor/community with regular updates

**Deprecated**: We're phasing this technology out, but existing usage is supported.

- No new projects should use this
- Existing projects should plan migration
- Security patches and critical fixes only
- End-of-life date communicated

**Retired**: This technology is no longer supported.

- No exceptions for new usage
- Existing usage must be migrated
- Documentation archived
- Lessons learned captured

### Proposing New Tools

Have a technology that should be in this guide? Here's how to propose it:

1. **Document the need**: What problem does this solve that current tools don't?
2. **Evaluate portability**: Does it work on all major cloud providers? Is it based on open standards?
3. **Assess operational impact**: What's the learning curve? What new skills are needed? What's the operational overhead?
4. **Pilot it**: Use it in a non-critical project and document the experience.
5. **Present findings**: Share results with architecture review board (or equivalent) including pros, cons, and migration costs.
6. **Decide together**: Tool adoption affects everyone - make it a team decision, not an individual one.

### Annual Review

Every year, we review this guide to ensure it stays relevant. Questions we ask:

- Are version requirements still current?
- Have better alternatives emerged?
- Are deprecated tools ready for retirement?
- Should any approved tools move to deprecated status?
- What new technologies should we evaluate?

Technology moves fast - our standards should evolve thoughtfully, not chaotically.

## Cross-References

Implementation details for these tools and patterns are documented elsewhere:

- [Observability: Metrics](./observability-metrics.md) - Prometheus instrumentation and RED/USE methods
- [Observability: Distributed Tracing](./observability-distributed-tracing.md) - OpenTelemetry implementation and sampling
- [Observability: Logging](./observability-logging.md) - Structured logging and aggregation patterns
- [Scale and High Availability](./scale-and-high-availability.md) - Caching strategies and message queue patterns
- [Flexible Application Configuration](./flexible-application-configuration.md) - Configuration management and secrets handling
- [Security in Development](./security-in-development.md) - Security tooling and policy enforcement with OPA
- [Builds and Deployments](./builds-and-deployments.md) - CI/CD pipeline patterns
- [Versioning Our Solutions](./versioning-our-solutions.md) - Artifact versioning and release management
- [Managing Our Source](./managing-our-source.md) - Git workflows and branching strategies
- [Deliver Solutions That Work](./deliver-solutions-that-work.md) - Testing strategies and quality assurance

## Summary

Good tooling standards balance consistency with flexibility. Be specific about portable standards (Docker, Kubernetes, OpenTelemetry) and pattern-based about cloud services (managed databases, message queues).

The goal isn't standardization for its own sake - it's enabling teams to build reliable, portable, cloud-native solutions without reinventing the wheel every time.

Choose patterns over products. Build for portability. Let managed services handle undifferentiated heavy lifting. Focus on business value, not infrastructure complexity.
