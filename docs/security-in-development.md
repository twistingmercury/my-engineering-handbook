---
title: "Security in Our Development Process"
description: "Secure coding practices, workflow security, and data protection guidelines for building safe software"
category: "Core Principles"
tags:
  [
    "security",
    "code-review",
    "secrets-management",
    "data-protection",
    "PHI",
    "OWASP",
  ]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
date: "2024-11-11"
revision_history:
  - date: "2024-11-11"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Security in Our Development Process

## Other References

- <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- <https://owasp.org/www-project-top-ten/>

## Writing Secure Code

### Treat All Input as Hostile

Here's the thing: every piece of data coming into your application is potentially malicious until proven otherwise. Sounds paranoid? Maybe. But it's kept us out of trouble.

What this looks like in practice:

- Validate data types, lengths, and formats at your API boundaries
- Use parameterized queries for all database operations (yes, all of them)
- Sanitize anything that gets displayed back to users
- Never trust data from third-party APIs without validation

Think of it this way - would you let a stranger into your house without checking their ID? Same principle applies to your API.

### Keep Secrets Out of Source Code

This should be obvious, but we've all seen it happen: someone commits an API key to git, and suddenly your AWS bill looks like a phone number.

Here's how we handle secrets:

- Use environment variables or secret management systems (Azure Key Vault, AWS Secrets Manager, etc.)
- Add `.env` files to `.gitignore` immediately - before your first commit
- Rotate secrets regularly, especially after team changes
- Use different secrets for dev, staging, and production

**Red flags that should trigger alarm bells:**

- Hardcoded passwords or API keys anywhere in code
- `.env` files committed to git
- Production credentials shared in Slack or email

If you see any of these, stop what you're doing and fix it. Then figure out how it happened so it doesn't happen again.

### Vet Your Dependencies

Every npm install, pip install, or go get is a trust decision. Are you sure you want to run that code in production?

Before adding any third-party library:

- Check for known vulnerabilities (our CI does this automatically, but look anyway)
- Review the license terms - some licenses have implications for our business
- Consider the maintenance status - is this project actively maintained or abandonware?
- Look at the project's security track record - have they handled past issues well?

Our CI pipeline should run automated vulnerability scanning and license compliance checking on every build. Regular dependency updates aren't optional - they're part of keeping the lights on.

## Development Workflow Security

### Every Code Change Gets Reviewed

No code reaches production without another set of eyes on it. Period.

What reviewers should look for:

- Security vulnerabilities (SQL injection, XSS, authentication bypasses, etc.)
- Proper input validation
- Secrets or sensitive data accidentally committed
- Adherence to our secure coding standards

Code review isn't just about catching bugs - it's about sharing knowledge and keeping everyone sharp on security practices.

### Main Branches Are Protected

Our main branches have protection rules because we've all seen what happens when someone accidentally force pushes to main at 4:59 PM on a Friday.

Protection rules in place:

- No direct pushes - everything goes through a PR
- All CI checks must pass (no exceptions)
- Pull requests require approval from another team member
- No force pushes allowed
- Branches must be up to date before merging

These aren't just annoying hoops to jump through - they've saved us from production incidents more times than we can count.

### Automate Security Scanning

Our CI pipeline is like a security guard that never sleeps, never gets tired, and never forgets to check something.

Automated checks on every build:

- Source code vulnerability scanning
- Dependency vulnerability checking
- Docker image security scanning
- Infrastructure-as-code misconfigurations

If the scanner finds something, the build fails. Fix it before merging, not after.

## Data Protection

### PHI Never Leaves Production

Real customer data stays in production. Full stop.

Here's how we handle testing and development:

- Use synthetic test data that looks real but isn't
- Use data masking tools when you absolutely must debug something in production
- Keep separate databases for development that never, ever contain PHI

If you need production data to debug an issue, talk to the team first. There's usually a better way.

### Log Thoughtfully

Logs are for debugging, not for data collection. They should help you understand what happened without exposing sensitive information.

**What we do log:**

- Application errors and exceptions
- Performance metrics
- User actions (without personal details)

**What we never log:**

- Passwords or API keys
- Personal health information
- Credit card numbers, SSNs, or any PII
- Full request/response bodies with sensitive data

If you're not sure whether something is safe to log, assume it isn't and ask.

### Lock Down Containers

Docker images are code artifacts, and they need the same security attention as everything else we build.

Best practices we follow:

- Run as non-root users (containers don't need root permissions)
- Use minimal base images (less code = smaller attack surface)
- Never bake secrets into images (they're visible to anyone with access)
- Keep base images updated; patch vulnerabilities regularly. That mean build a new image.

## When Security Issues Happen

### Fix the Root Cause, Not Just the Symptom

When you find a security issue, don't just slap a band-aid on it and move on. We need to understand why it happened and prevent it from happening again.

Our process:

1. Fix the immediate issue
2. Understand how it happened in the first place
3. Search the codebase for similar problems
4. Update our processes to prevent recurrence
5. Document what we learned

The goal isn't to avoid blame - it's to avoid repeating mistakes.

### Escalate Quickly

Security issues don't belong in the backlog with feature requests. They get escalated immediately.

Who needs to know:

- Security team (always)
- Product owners (they need to know what's at risk)
- Infrastructure team (if it involves deployment or infrastructure)
- Compliance team (for anything involving PII or PHI)

The faster we escalate, the faster we can respond. Don't sit on security issues hoping they'll fix themselves.
