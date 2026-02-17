---
title: "Deliver Solutions that Work as Expected"
description: "Quality standards, testing strategies, and definition of done for shipping reliable software"
category: "Core Principles"
tags: ["quality", "testing", "unit-tests", "e2e-tests", "definition-of-done", "documentation-testing"]
audience: "Software Engineers, Test Engineers"
version: "1.0.0"
revision_history:
  - date: "2025-10-02"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
---

# Deliver Solutions that Work as Expected

## Other References

- [Unit Testing Guidelines: What to Test And What Not](https://www.automatetheplanet.com/unit-testing-guidelines/)

## Overview

Quality isn't something you add at the end - it's baked in from the start.

We pick simple architectures and tools that make our code clean and maintainable. Think of it like building a house: a solid foundation makes everything else easier. And since no single approach guarantees quality, we throw multiple strategies at the problem:

- **Unit tests** verify each function works correctly in isolation
- **End-to-end tests** make sure everything works together like your users expect
- **Performance and load tests** ensure the system can handle real-world traffic
- **Data validation** catches issues before they become production incidents
- **Documentation testing** confirms our docs actually work (yes, we test the docs too)

## Definition of Done

Before you call something "done," it needs to check all these boxes. This isn't bureaucracy - it's the difference between shipping quality software and shipping future headaches.

- **README.md is current** - Engineers should understand what the project does and how to work with it. See [README Requirements](./importance-of-documentation.md#project-readme-requirements)
- **CHANGELOG.md is updated** - Document what changed in this release
- **Observability is working** - Logs and traces flow to Datadog, indexes are configured, retention policies are set
- **Monitoring and alerts are live** - We know when things break, ideally before users do
- **Runbook is written** - Operations docs exist and someone has actually followed them
- **Scripts are tested** - Any automation you're shipping has been verified to work
- **Operational tasks are scheduled** - Regular maintenance is automated (cert renewal, log cleanup, etc.)
- **CI pipeline is green** - All unit and E2E tests pass
- **PR is approved** - Someone else reviewed your code and signed off (you can't approve your own)
- **Security scans pass** - Vulnerabilities are addressed, not just acknowledged

## Unit Tests

Unit tests should be fast, isolated, and not depend on external resources (no databases, no network calls, no external file system dependencies. Local test fixtures and embedded test data are acceptable).

We aim for good coverage, but "good" doesn't mean 100%. Here's what we target:

- **75% minimum overall line coverage** - This is your baseline across the entire codebase
- **95% for critical paths** - Authentication, payment processing, data integrity, and security-sensitive code deserve extra attention
- **Document exceptions** - If you can't hit these targets, write down why. Maybe it's legacy code that's being replaced, or integration code that's better tested at the E2E level. Just explain your reasoning.

If a file has 20% coverage but the other 80% is just constants and type definitions, that's fine. Most testing frameworks let you exclude that stuff from coverage reports - use those features.

### What Not to Test

Don't waste time testing:

- 3rd party libraries and APIs (trust that they test their own code)
- Constants and configuration values
- Auto-generated code
- Plain data structures with no behavior (POJOs/POCOs, DTOs, etc.)

This isn't an exhaustive list. Use your judgment. If a test wouldn't catch a real bug or doesn't give you confidence the code works, skip it. Just document why you skipped it so future developers understand the reasoning.

## End-to-end Testing

E2E tests verify your system works the way your users (or API clients) expect it to work.

If you built a RESTful API, your E2E tests should hit every endpoint and verify every response type - success cases, error cases, edge cases, all of it. Prioritize your testing efforts: 1) All success paths, 2) Documented error responses, 3) Auth boundaries, 4) Edge cases. Use your judgment on testing every permutation of query parameters or request variations.

For details on E2E test infrastructure, Docker setup, and examples, see [End-to-End Test Suite](./builds-and-deployments.md#end-to-end-test-suite) in the builds and deployments guide.

### How E2E Tests Work

These tests run in isolated Docker environments that spin up everything your service needs: databases, message queues, dependent services, the whole stack. Here's the typical flow:

1. **Execute the action** - Use `curl` to hit your API, or run your CLI tool, or whatever your users would do. Write these tests in whatever language makes sense, but here's the key: don't import or depend on your service's code. Test it like a black box, the way a real user would.

2. **Verify the response** - Check that you got what you expected. For APIs, test all endpoints and all possible response codes (200s, 400s, 500s).

3. **Check state changes** - If the operation should have modified data, verify it in the database. Created a record? Query for it. Updated a field? Check the new value. Deleted something? Make sure it's gone.

Like unit tests, E2E tests run automatically in CI. If they fail, the build fails.

## Testing Your Instructional Docs

Here's something people often skip: testing documentation. But think about it - if your docs are wrong or confusing, the next person to work on this project is going to waste hours (or days) trying to figure it out.

> **The Gold Standard**
>
> Someone who joined the team last week should be able to follow your docs and succeed without asking for help.

You're not writing docs for yourself - you're writing for the team member who'll maintain this code six months from now (which might also be you, but future you who's forgotten everything).

### How to Test Your Docs

This is a manual process, but it doesn't take long:

1. **Cut the fluff** - Keep it short and direct. If a sentence doesn't add value, delete it.

2. **Use AI for polish** - Tools like ChatGPT or Claude can help make your writing clearer and more consistent.

3. **Get a real user** - Have a teammate (ideally someone not familiar with the project) follow your instructions. Every question they ask is a gap in your docs.

4. **Iterate until it works** - Once someone can complete the task without asking questions, your docs are ready.

---

[← Previous: Security in Development](./security-in-development.md) | [↑ Back to Home](index.md) | [Next: Importance of Documentation →](./importance-of-documentation.md)
