---
title: "Flexible Application Configuration"
description: "Configuration best practices using environment variables, config files, and CLI flags for containerized applications"
category: "Operations"
tags: ["configuration", "12-factor", "environment-variables", "secrets", "docker", "kubernetes"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0.0"
revision_history:
  - date: "2025-09-29"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Flexible Application Configuration

## Other Resources

- [The Twelve-factor App: III. Config](https://12factor.net/config)

## Why Configuration Matters

Your app should get its settings from different places and work smoothly whether it's on your laptop or in production. This is really important for things to run well in container setups.

We're loosely following the [12-factor app configuration guidelines](https://12factor.net/config) that emphasize storing config in the environment and separating it from code.

## The Three Ways Apps Should Get Their Config

Your application should support all three of these configuration methods, because different situations call for different approaches:

### 1. Environment Variables

This is your bread and butter for containerized apps. Environment variables are:

- **Perfect for Docker/Kubernetes** – Easy to inject at runtime
- **Great for secrets** – Can be populated from secret stores
- **Cloud-native friendly** – Every orchestration platform handles them well

### 2. Configuration Files

Config files are great for complex setups and local development:

- **Better for complex config** – Nested structures, arrays, etc.
- **Version controllable** – Non-sensitive defaults can be checked in (never credentials or environment-specific values)
- **Human readable** – Easy to understand and modify

### 3. Command Line Flags

Command line flags are perfect for overrides and debugging:

- **Great for overrides** – Quick changes without rebuilding
- **Perfect for debugging** – Temporary config changes
- **Explicit and visible** – Easy to see what's different

## Configuration Precedence (Most Important Wins)

When your app starts up, it should check for configuration in this order. **Later sources override earlier ones**:

1. **Built-in defaults** – Safe values for development and testing
2. **Configuration files** – More complex structured config
3. **Environment variables** – Container-friendly overrides
4. **Command line flags** – Explicit overrides and debugging

> **Note on 12-Factor App alignment**: The [12-Factor App](https://12factor.net/config) emphasizes environment variables as the primary config source for production. This precedence order supports local development workflows where config files provide convenience. In containerized production environments, environment variables should contain all necessary configuration rather than relying on defaults or config files.

**Important:** Your built-in defaults should _never_ be production-ready values. They should be obviously development values that would fail safely in production. Examples of safe defaults:

- **Database URLs**: `localhost:5432`, `127.0.0.1:3306` – Will fail with "connection refused" if accidentally used in production
- **External APIs**: Mock/stub endpoints or dedicated sandbox environments – Prevents accidental production API calls during development
- **Timeouts**: Conservative values like `30s` instead of `5s` – Gives more breathing room for local debugging
- **Feature flags**: All experimental features OFF by default – Prevents untested features from running in production

**Guiding principle**: Defaults should fail loudly in production if accidentally used (connection refused, not silent data corruption). This fail-fast approach catches configuration mistakes early rather than causing subtle production issues.

## Docker & Kubernetes Best Practices

### For Docker Containers

- **Use environment variables** for most runtime config
- **Bake default config files** into your container image
- **Mount custom config files** as volumes when needed
- **Don't hardcode anything** that might change between environments

### For Kubernetes Deployments

- **ConfigMaps** for non-sensitive configuration data
- **Secrets** for sensitive data like passwords and API keys
- **Environment variables** to expose ConfigMap and Secret values
- **Init containers** for complex configuration setup if needed

### Handling Sensitive Configuration

Never, ever put secrets in your code or regular config files. Here's how to handle sensitive data properly:

#### Configuration Types and Where They Belong

Different types of configuration data have different security and lifecycle requirements. Here's where each type should live:

| Configuration Type | Storage Method | Examples | Version Controlled? |
|-------------------|----------------|----------|-------------------|
| **Non-sensitive settings** | Config files in repo | Port numbers, log levels, timeouts, feature flags, retry counts | Yes |
| **Environment-specific settings** | ConfigMaps / Environment variables | Service URLs, database names, queue names, external API endpoints | No (managed per environment) |
| **Credentials & secrets** | Secrets Manager / Kubernetes Secrets | Passwords, API keys, certificates, encryption keys, access tokens | Never |

#### What Counts as Sensitive?

- Database passwords and connection strings
- API keys and access tokens
- Encryption keys and certificates
- Third-party service credentials

#### How to Handle Secrets

- **Use a secrets management service** (AWS Secrets Manager, Azure Key Vault, etc.)
- **Kubernetes Secrets** for container deployments
- **Environment variables** populated at runtime from secure sources
- **Never log sensitive values** – redact them in logs and error messages

### Configuration Validation

Your app should validate its configuration at startup and fail fast if something's wrong:

- **Check required values** are present
- **Validate formats** (URLs, ports, timeouts, etc.)
- **Test connections** where possible (database, external APIs)
- **Provide clear error messages** when validation fails

#### What "Fail Fast" Means in Practice

When configuration validation fails, your application should follow these specific behaviors:

- **Exit with code 1** – Signal a fatal error to the container runtime
- **Log at CRITICAL level** – Include specific field names identifying what's wrong (e.g., "DATABASE_URL is missing", "PORT must be between 1-65535, got: 99999")
- **Validate ALL configuration** – Report all validation failures in one go, not just the first error encountered
- **No retry loops** – Don't attempt to retry configuration validation; if config is wrong, it won't fix itself
- **Let the orchestrator handle restarts** – Kubernetes and other orchestrators will restart your container based on their policies

This approach ensures configuration problems are caught immediately at startup rather than causing mysterious failures later during runtime. The orchestrator can then apply its restart policies while you investigate and fix the configuration.

---

[← Previous: Observability: Health Checks](./observability-heartbeats-readiness-liveness.md) | [↑ Back to Home](index.md) | [Next: Scale & High Availability →](./scale-and-high-availability.md)
