---
title: "The Importance of Documentation"
description: "Documentation standards, README requirements, and guidelines for keeping project documentation current and useful"
category: "Core Principles"
tags: ["documentation", "README", "CHANGELOG", "runbooks", "confluence", "operational-docs"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
date: "2024-11-11"
revision_history:
  - date: "2025-09-30"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# The Importance of Documentation

## Related Resources

- [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)

## Documentation That Actually Helps

We've all been there: you inherit a project with docs that are either missing, outdated, or flat-out wrong. It's frustrating, time-consuming, and completely avoidable.

Good documentation isn't nice-to-have - it's essential. Poor docs lead to operational nightmares, frustrated developers, and services that nobody wants to touch.

**The cardinal rule:** _Stale documentation is worse than no documentation._ If the docs lie to you, you'll waste time following bad instructions. If there are no docs, at least you know to read the code.

That's why the README gets updated with every change to the project. Not eventually. Not when you remember. Every time.

## Project README Requirements

Every project gets a README that follows this structure. No exceptions.

```markdown
# Project Name

> **Maturity Level**: [Emerging|Basic|Mature] - (a short sentence fragment for context)

---

A sentence describing the project. Two at most.

## Usage

## How it works

## Key Considerations

## Development Considerations

### Quick Start

### Building & running

### Testing

### Versioning
```

### README Guidelines

Follow these rules to keep READMEs useful:

- **No emojis** - They look unprofessional and add zero value (checkboxes are fine)
- **Working links only** - Test every link before committing. Broken links destroy credibility
- **Never duplicate docs** - Reference other docs, don't copy-paste them. DRY applies to documentation too
- **No file tree diagrams** - They're useless and become outdated instantly
- **Explain versioning** - Document how the project is versioned (usually git tags)
- **No workstation setup instructions** - Include version requirements, not "how to install Node.js"
- **Write for the reader** - What do they actually need to know?
- **Most important stuff first** - Lead with usage, bury the implementation details
- **Be specific** - "Configure the database" is worthless. "Set the DATABASE_URL environment variable" is helpful
- **Show, don't just tell** - Include examples. A code snippet is worth a thousand words

## Operational Documentation

Regular maintenance and operational tasks live in our team's OpsBook. We don't rely on people remembering to do these tasks - each one has a Jira Automation that creates tickets on schedule.

Examples of operational tasks:

- Checking logs for anomalies
- Renewing certificates before they expire
- Rotating API keys for third-party services
- Reviewing access permissions

If it needs to happen regularly, it goes in the OpsBook with automation.

## Documentation Types & When to Use Them

Different documentation serves different purposes. Use the right tool for the job.

### README.md

- **When to use:** Every project, no exceptions
- **What goes in it:** Project overview, usage instructions, development setup
- **Who it's for:** Developers, operators, anyone who needs to use or work on the project

### CHANGELOG.md

- **When to use:** Every project that has versioned releases
- **What goes in it:** What changed between versions (follow [Keep a Changelog](https://keepachangelog.com/))
- **Who it's for:** Users upgrading versions, maintainers tracking history

### Confluence Pages

- **When to use:** Process docs, architecture decisions, team knowledge, OpsBooks
- **What goes in it:** How we work, why we made specific decisions, operational procedures
- **Who it's for:** Team members, stakeholders, future team members

### Runbooks

- **When to use:** Production services that require operational support
- **What goes in it:** Troubleshooting guides, emergency procedures, monitoring dashboards
- **Who it's for:** On-call engineers, operations team, anyone responding to incidents
