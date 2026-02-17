---
title: "Builds and Deployments"
description: "Container-based CI/CD pipeline architecture, multi-stage Docker builds, and deployment strategies for services and tools"
category: "Development Practices"
tags:
  ["docker", "ci-cd", "builds", "deployments", "e2e-tests", "kubernetes", "AKS"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0.0"
revision_history:
  - date: "2025-10-02"
    author: "Jeremy K. Johnson"
    changes: "Initial release"
  - date: "2025-12-15"
    author: "Jeremy K. Johnson"
    changes: Add examples for the docker builds and e2e tests.
---

# Builds and Deployments

## Containers are Central to our CI Build Pipeline

> The actual code for the examples are currently in Go. However, the focus isn't the code; the focus is on using containers to manage the build and end-to-end testing. These principles apply to any language: .NET, Python, Kotlin/Java...you get the idea.

Using Docker and Docker Compose for our build process does a very special thing for us: It makes the build environment predictable and uniform. That means the build we run locally is the same build that runs within our CI pipeline. No more "it works on my machine" headaches.

This may seem obvious for services that are hosted in a container, but it holds even for things like command-line tools that are installed locally.

For services and processes that are natural fits for hosting in a container, we use a multi-stage Docker build. For a locally installed tool, we don't need a multi-stage build; we just copy the compiled binary out of the build container: [tools ci build Dockerfile](https://github.com/twistingmercury/docker-ci-builds/blob/develop/cli/build/Dockerfile)

## Basic Build Architecture

All CI builds will have four basic parts that come together to make it all work:

1. **The CI build definition**: This can be a definition in Azure DevOps, AWS, GitHub, etc. The pipeline technology really doesn't matter.
2. **A Dockerfile**: This executes the actual compilation, and if required, creates the final image.
3. **End-to-End Test suite**: These are the end-to-end tests, a.k.a. `e2e` tests, that perform testing as if it were a user of the tool, or a client of the service.
4. **An orchestrator script**: This is the "glue" that brings it all together. For services, we typically use a `build.sh` script. For CLI tools, a `Makefile` works well. The orchestrator ensures all necessary tooling is installed, runs linting, code analysis, unit tests, and then calls `docker build...`. Assuming all steps pass, it then executes the end-to-end tests.

> **Example Projects**
>
> **Service example:** [api/](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api) - uses `build.sh` orchestration
>
> **CLI tool example**: [cli/](https://github.com/twistingmercury/docker-ci-builds/tree/develop/cli) - uses `Makefile` orchestration

### The CI Build Definition

This file is typically a `yaml` file that defines the build for the pipeline. It will have sections where you'll configure the OS (we use Linux), pull down the code, set up any authentication contexts, and after the build script (the orchestrator script) has completed, publish any artifacts to wherever they may need to go, such as test results, Docker Images, code artifacts, etc. The details of what actually gets published from project to project may change, but the responsibility of the yaml build definitions is the same. See the example workflows: [api-ci.yaml](https://github.com/twistingmercury/docker-ci-builds/blob/develop/.github/workflows/api-ci.yaml) and [cli-ci.yaml](https://github.com/twistingmercury/docker-ci-builds/blob/develop/.github/workflows/cli-ci.yaml).

### The Dockerfile

The Dockerfile is core to the build. Within it, the actual binary is compiled within an environment that is 100% under your control. The benefit is that it will build anywhere since there are no external dependencies. Everything it needs to build is within the Docker container. Compilation tools? There. Supporting tools like code generators, linters, and analyzers? There. The build experience is the same regardless of where it is run!

The [api/build/](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api/build) directory shows a multi-stage build example. The [Dockerfile](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/build/Dockerfile) handles compilation within controlled build stages, installing linters and security scanners as part of the build. The [build.sh](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/build/build.sh) orchestrator script coordinates the overall process: building the Docker image, running the end-to-end tests, and preparing artifacts for the CI pipeline.

### End-to-End Test Suite

> **IMPORTANT!**
>
> Developing solid end-to-end tests will probably take just as long as implementing the solution you're developing. So, keep this in mind. You need to factor this in when starting a new project or adding functionality to an existing project.

For the philosophy and testing approach behind E2E tests, see [End-to-end Testing](./deliver-solutions-that-work.md#end-to-end-testing) in the quality standards guide.

This is a little more complex to dive into, so we won't go into the details too far. For a look at what end-to-end tests look like in detail, please refer to the end-to-end tests for the API example: [api/tests/e2e/](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api/tests/e2e)

These tests are written as consumers of the API and don't rely upon any of the code from the API project. They only rely upon what a client would have access to, such as API documentation. In the example API, the E2E tests are written in Go using the standard testing package.

The tools or language you use to write the tests are secondary. As long as the tests are conducted from a consumer's perspective, based on the documentation the caller would have, it is fine.

The typical structure of the end-to-end tests are contained within a `tests/` directory at the project's root:

- [docker-compose.yaml](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/tests/docker-compose.yaml): Sets up the infrastructure the tests need. For the API example, this includes the API service, any dependent services (databases, collectors), and the test container itself.
- [e2e/Dockerfile](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/tests/e2e/Dockerfile): Builds a Docker image containing the tests and any runner scripts needed. When a container is created using this image, testing starts.
- [e2e/test-runner.sh](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/tests/e2e/test-runner.sh): The entrypoint script that coordinates the test run. It ensures all needed resources are available before executing the tests, since some containers may take time to be ready.

## Deployments

We value and strive to be able to perform zero-downtime deployments. We use release pipelines to help manage this. The [api-ci.yaml](https://github.com/twistingmercury/docker-ci-builds/blob/develop/.github/workflows/api-ci.yaml) workflow demonstrates this pattern.

The one thing we don't automate right away is deployment...at first; a human needs to "push the button". We want the be very explicit about when a deployment takes place. This is so we can monitor the deployment to make sure we can rollback quickly if needed. Once we're confident and have a history of deployments going well, we can work on automating the `CD` side of the equation.

We define "confident" as: 20 consecutive successful production deployments over a minimum of 30 days with zero rollbacks. This threshold ensures we have sufficient evidence that our deployment process, monitoring, and rollback procedures are reliable before removing human oversight.

### Hosting

You can't talk about deployments without talking briefly about hosting. Our ideal is Kubernetes. We prefer to avoid virtual machines. We also avoid using PaaS services like Azure App Services, or AWS Elastic Beanstalk, though there will be exceptions. We may have some older software deployed on these platforms, but we shouldn't target these hosting options for new apps and services. Containers is the word!

---

[← Previous: Versioning Our Solutions](./versioning-our-solutions.md) | [↑ Back to Home](index.md) | [Next: Observability: Logging →](./observability-logging.md)
