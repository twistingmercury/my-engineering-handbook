---
title: "Builds and Deployments"
description: "Container-based CI/CD pipeline architecture, multi-stage Docker builds, and deployment strategies for services and tools"
category: "Development Practices"
tags:
  ["docker", "ci-cd", "builds", "deployments", "e2e-tests", "kubernetes", "AKS"]
audience: "Software Engineers, DevOps Engineers"
version: "1.0"
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

Using Docker and Docker Compose for our build process does a very special thing for us: It makes the build environment predictable and uniform. That means the build we run locally is the same build that runs within our CI pipeline. No more "it works on my machine" nonsense.

This may seem obvious for services that are hosted in a container, but it holds even for things like command-line tools that are installed locally.

For services and processes that are natural fits for hosting in a container, we use a multi-stage Docker build. For a locally installed tool, we don't need a multi-stage build; we just copy the compiled binary out of the build container: [tools ci build Dockerfile](https://github.com/twistingmercury/docker-ci-builds/blob/develop/cli/build/Dockerfile)

## Basic Build Architecture

All CI builds will have four basic parts that come together to make it all work:

1. **The CI build definition**: This can be a definition in Azure DevOps, AWS, GitHub, etc. The pipeline technology really doesn't matter.
2. **A Dockerfile**: This executes the actual compilation, and if required, creates the final image.
3. **End-to-End Test suite**: These are the end-to-end tests, a.k.a. `e2e` tests, that perform testing as if it were a user of the tool, or a client of the service.
4. **An orchestrator script**: This POSIX-compliant script is the "glue" that brings it all together. We typically call the file `build.sh`. This file will make sure all necessary tooling is installed into the build environment, run linting, code analysis, the unit tests, and then call `docker build...`. Assuming all the steps pass, it will then execute the end-to-end tests.

> **Example Projects**
>
> **Service example:** [docker-ci/builds/api](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api)
>
> **Tool example**: [docker-ci/cli](https://github.com/twistingmercury/docker-ci-builds/tree/develop/cli)

### The CI Build Definition

This file is typically a `yaml` file that defines the build for the pipeline. It will have sections where you'll configure the OS (we use Linux), pull down the code, set up any authentication contexts, and after the build script (the orchestrator script) has completed, publish any artifacts to wherever they may need to go, such as test results, Docker Images, code artifacts, etc. The details of what actually gets published from project to project may change, but the responsibility of the yaml build definitions is the same. Here is an example from the [docker-ci/builds/api](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api) project:

### The Dockerfile

The Dockerfile is core to the build. Within it, the actual binary is compiled within an environment that is 100% under your control. The benefit is that it will build anywhere since there are no external dependencies. Everything it needs to build is within the Docker container. Compilation tools? There. Supporting tools like code generators, linters, and analyzers? There. The build experience is the same regardless of where it is run!

Here is an example of a multi-stage build from the [docker-ci/builds/api/build](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api/build) project:

1. Ensure the necessary authorization is granted.
2. Install any required tools the script will need to orchestrate the build; [linters, security scanners, etc](https://github.com/twistingmercury/docker-ci-builds/blob/41c3034cc6b0ac6714db5bc38f32313dca389faa/api/build/Dockerfile#L9).
3. Build the Docker [image](https://github.com/twistingmercury/docker-ci-builds/blob/1d99ff0962563bc7adf0ce51cb6a9779b8c95619/api/build/build.sh#L14C1-L28C2).
4. Run the end-to-end [tests](https://github.com/twistingmercury/docker-ci-builds/blob/1d99ff0962563bc7adf0ce51cb6a9779b8c95619/api/build/build.sh#L30).
5. Prepare any files for any further action that may need to be managed by the CI pipeline. These are usually things like pushing the Docker image to a private repository, publishing analysis results, etc.

### End-to-End Test Suite

> **IMPORTANT!**
>
> Developing solid end-to-end tests will probably take just as long as implementing the solution you're developing. So, keep this in mind. You need to factor this in when starting a new project or adding functionality to an existing project.

For the philosophy and testing approach behind E2E tests, see [End-to-end Testing](./deliver-solutions-that-work.md#end-to-end-testing) in the quality standards guide.

This is a little more complex to dive into, so we won't go into the details too far. For a look at what end-to-end tests look like in detail, please refer to the end-to-end test for the API example: [docker-ci/builds/api/tests](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api/tests/e2e)

These tests are written as consumers of the API and don't rely upon any of the code from the API project. It only relies upon what a client may have access to, such as API documentation like swagger.json. In the example API, `BATS` and `curl` are the core tools to write the tests, and `SQLCMD` is used to validate any database changes that should be expected. Responses are evaluated against what is documented in the OpenAPI specification (swagger.json).

When writing the end to end tests the tools or language to write the tests are secondary. As long as the result is that the tests are conducted from a consumer's perspective, based on the documentation the caller would have, it is fine.

The typical structure of the end-to-end tests are like so, contained within a directory named `tests` at the project's root.

- [.conf] directory: *optional, as needed* This contains any necessary files, like configurations, certs for testing, etc., that the testing infrastructure will need. In this example, we needed a cert for the OTEL collector, and a configuration for it as well.
- [Dockerfile](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/tests/e2e/Dockerfile): This builds a docker image containing the tests and any runner scripts needed. When a container is created using the image, the testing starts.
- [docker-compose.yaml](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/tests/docker-compose.yaml): This sets up the infrastructure the tests will needs. In the case of SAMPLE MGMT API, we need to set up the following:
  - `otel_collector`: This is so that traces from the SAMPLE MGMT API will have a working endpoint, and not create any unnecessary errors.
  - `project_databases`: This is a custom image that we use that contain the database accessed by SAMPLE MGMT API.
  - `project_api`: This will be the image that was created during the build.
  - `m_tests`: This is the image that contains the tests that will be executed.
- [test-runner.sh](https://github.com/twistingmercury/docker-ci-builds/blob/develop/api/tests/e2e/test-runner.sh): This script is the entrypoint to the container and coordinates the test run. Some containers may take a little time to be ready, so this script will ensure that all needed resources are available before executing the tests.

## Deployments

We value and strive to be able to perform zero-downtime deployments. We use release pipelines to help manage this. Continuing with using [docker-ci/builds/api](https://github.com/twistingmercury/docker-ci-builds/tree/develop/api) as an example, here is it's pipeline definition: [api-ci.yaml](https://github.com/twistingmercury/docker-ci-builds/blob/develop/.github/workflows/api-ci.yaml)

The one thing we DO NOT automate is deployment...at first; a human needs to "push the button". We want the be very explicit about when a deployment takes place. This is so we can monitor the deployment to make sure we can rollback quickly if needed. Once we're confident and have a history of deployments going well, we can work on automating the `CD` side of the equation.

We define "confident" as: 20 consecutive successful production deployments over a minimum of 30 days with zero rollbacks. This threshold ensures we have sufficient evidence that our deployment process, monitoring, and rollback procedures are reliable before removing human oversight.

### Hosting

You can't talk about deployments without talking briefly about hosting. Our ideal is Kubernetes. We do NOT want to use virtual machines. We also avoid using PaaS services like Azure App Services, or AWS Elastic Beanstalk, though there will be exceptions. We may have some older software deployed on these platforms, but we shouldn't target these hosting options for new apps and services. Containers is the word!
