---
title: "Versioning Our Solutions"
description: "Semantic versioning standards, version information requirements, and Docker image labeling practices"
category: "Development Practices"
tags: ["semver", "versioning", "docker", "OCI", "git-tags", "version-commands"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0.0"
revision_history:
  - date: "2024-11-11"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Versioning Our Solutions

## External Resources

- <https://semver.org/>
- <https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html>
- <https://github.com/opencontainers/image-spec/blob/main/annotations.md>
- <https://git-scm.com/book/en/v2/Git-Basics-Tagging>

## Versioning

Our team follows [Semantic Versioning 2.0.0](https://semver.org/) for all our projects.

Here's the deal: all production-ready applications and container images get proper semantic versioning. For test builds or anything that's not quite ready for production, we follow the pre-release versioning guidelines from [SemVer spec item 9](https://semver.org/#spec-item-9).

### What "Production-Ready" Means

An application is considered production-ready when it meets all criteria in our [Definition of Done](./deliver-solutions-that-work.md#definition-of-done). Specifically:

- **CI pipeline is green** - All tests passing (unit, E2E, integration)
- **Code review approved** - Someone else reviewed and signed off
- **Observability configured** - Logs, metrics, and traces flowing to monitoring systems
- **Runbook created and reviewed** - Operations documentation exists and has been validated
- **Deployment tested** - Successfully deployed and verified in staging environment
- **Rollback procedure documented and tested** - We know how to undo this if things go wrong
- **Security scans pass** - No unaddressed vulnerabilities

Until all these boxes are checked, use pre-release versioning (like `1.0.0-alpha.1` or `1.0.0-rc.2`) to signal the application isn't ready for production deployment.

### Command Line Version Info

Every application should be able to tell you what version when asked. Here's what we expect:

All programs should support the standard `--version` flag (following [GNU standards](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html)). The short `-v` option is nice to have, but not required.

When possible, include these details in your version output:

- **Date**: Build date in ISO 8601 format (`YYYY-MM-DD`)
  _example:_ `BUILD_DATE=$(date +"%Y-%m-%d")`
- **Commit**: Short Git commit hash (`1234abc`)
  _example:_ `COMMIT_HASH=$(git rev-parse --short HEAD)`
- **Version**: Comes from the latest git tag
  _example:_ `VERSION=$(git describe --tags --always --dirty 2>/dev/null || echo "dev")`

### Version Example

Here's what a good version output looks like:

```bash
$ someApp --version
someapp - version: 1.2.3; build date: 2024-02-02; commit: 5ab3d7e
```

### Incrementing Versions

Every new software release should bump the version according to semantic versioning rules:

- **Major** (1.0.0): Breaking changes
- **Minor** (0.1.0): New features, backwards compatible
- **Patch** (0.0.1): Bug fixes, backwards compatible

## Docker Image Labeling

All our Docker images should include these labels based on the [OCI Image Format Specification](https://github.com/opencontainers/image-spec/blob/main/annotations.md):

```json
{
  "org.opencontainers.image.base.name": "<company>/<team-name>/<app-name>",
  "org.opencontainers.image.created": "2023-06-14T15:37:46Z",
  "org.opencontainers.image.ref.name": "scratch",
  "org.opencontainers.image.title": "<app-name>",
  "org.opencontainers.image.vendor": "<company>",
  "org.opencontainers.image.version": "<semantic-version>",
  "org.opencontainers.image.source": "<url-to-git-repo>",
  "org.opencontainers.image.authors": "<team-email>"
}
```

**Important:** The `org.opencontainers.image.version` label should match your software version and align with your Git tag. Keep everything in sync!

---

[← Previous: Managing Our Source](./managing-our-source.md) | [↑ Back to How We Operate](../README.md) | [Next: Builds and Deployments →](./builds-and-deployments.md)
