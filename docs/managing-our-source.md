---
title: "Managing Our Source"
description: "Trunk-based development strategy, branching guidelines, and code review practices for source control"
category: "Development Practices"
tags: ["git", "trunk-based-development", "branching", "pull-requests", "code-review", "ci-cd"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
date: "2024-11-11"
revision_history:
  - date: "2025-10-06"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Managing Our Source

## External Resources

- <https://trunkbaseddevelopment.com/>
- <https://git-scm.com/book/en/v2/Git-Basics-Tagging>

## Branching Strategy

Let's talk about how we handle branching in Supporting Services.

We use Git for source control, and we've adopted a [trunk-based development](https://trunkbaseddevelopment.com/) approach. Don't worry if you're not familiar with it - we'll walk through everything you need to know.

### How It Works

Think of our branching strategy like this: we have one main highway (the main branch) where all the stable code lives, and everyone creates their own on-ramps (feature branches) to get their work merged back onto that highway.

### The Main Branch (Our Trunk)

This is where all the stable, production-ready code lives. Every time something gets merged here, it automatically triggers a build that creates and pushes a Docker image. Pretty neat, right? This means our Docker images always reflect the latest stable code.

### Our Build Pipeline

Before anything gets merged back to main, it goes through our automated pipeline. This includes:

- Linting (keeping our code clean)
- Vulnerability and security checks (staying safe)
- Unit tests and end-to-end tests (making sure everything works)

This catches issues early and ensures only tested, working code makes it to main.

### Development Branches

When you're working on a new feature or fixing a bug, you'll create your own branch from main. These branches are meant to be:

- **Short-lived** - don't let them hang around too long
- **Focused** - one task per branch
- **Independent** - they shouldn't interfere with what others are doing

### Code Reviews & Pull Requests

Once you've finished your work, create a pull request to merge back into main. This kicks off our code review process where your teammates will take a look and make sure everything looks good. Only after passing review and all tests does your PR get merged.

### Staying Up to Date

Here's a pro tip: regularly merge changes from main into your development branch. This keeps you current with everyone else's work and helps avoid those dreaded merge conflicts.

## Branch Naming Guidelines

Keep it simple and consistent:

**Feature branches:** `feature/<jira-ticket-number>`

- Example: `feature/pe-1234`

**Bug fix branches:** `bug/<jira-ticket-number>`

- Example: `bug/pe-1235`

## A Few Important Things to Remember

- **Deploy independently:** Each branch should generally be able to merge and deploy on its own. We know there might be edge cases, and we'll handle those as they come up.
- **Keep it ephemeral:** Once your feature or bug fix is merged and successfully deployed to production, delete that branch. Don't let old branches pile up!
- **Ready for deployment:** When your branch gets merged to main, the expectation is that the code is ready to go live. At that point, we'll [tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) the codebase with the version number for deployment.

And that's it! Questions? Don't hesitate to reach out to the team.
