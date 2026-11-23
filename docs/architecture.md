# Project Architecture

## 1. Overview

This project implements an automated Continuous Integration (CI) and security validation pipeline for a basic Java application.

The architecture combines Java, Docker, GitHub Actions, Trivy, GitHub MCP Server, and GitHub Copilot.

The main objective is to automate application compilation, testing, containerization, vulnerability scanning, and pull request validation.

## 2. High-Level Architecture

```text
                 Developer
                     |
                     | Git Push / Pull Request
                     v
              GitHub Repository
                     |
                     v
              GitHub Actions
                     |
          +----------+----------+
          |                     |
          v                     v
     Java Setup            Docker Build
          |                     |
          v                     v
     Java Compile          Docker Image
          |                     |
          v                     v
     Java Tests           Trivy Security Scan
                                |
                                v
                       Container Test
                                |
                                v
                       Required Status Check
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
                  Passed                  Failed
                    |                       |
                    v                       v
              Eligible for             Merge Blocked
                  Merge               by Branch Rules
                                            
                                            
              GitHub MCP Server
                     |
                     v
               GitHub Copilot
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Workflow    Job       Job Logs
        Runs     Details
```

## 3. Component Responsibilities

### Java Application

The application is developed using Java 21.

Its purpose is to demonstrate compilation, execution, and output validation in the CI pipeline.

Source file:

`src/HelloWorld.java`

### Docker

Docker packages the Java application into a container image.

The project uses a multi-stage Dockerfile:

- Build stage: Uses the Java 21 JDK to compile the application.
- Runtime stage: Uses the Java 21 JRE to execute the compiled application.

This separates compilation tools from the runtime environment.

### GitHub Actions

GitHub Actions automates the CI process.

The workflow is defined in:

`.github/workflows/ci.yml`

It executes when:

- Code is pushed to `main`.
- A pull request targets `main`.

The workflow runs on a GitHub-hosted Ubuntu runner.

### Trivy

Trivy scans the locally built Docker image for operating system and library vulnerabilities.

The configured failure threshold is HIGH and CRITICAL.

If matching vulnerabilities are detected, the scan exits with code 1 and the workflow fails.

### Branch Protection

Branch protection rules are configured for `main`.

The workflow status check `Build, Test and Security Scan` is required before merging.

If the required check fails, GitHub blocks the normal merge process.

### GitHub MCP Server

GitHub MCP Server exposes GitHub repository and Actions functionality to an AI client.

GitHub Copilot uses the MCP integration to inspect:

- Repository information.
- Workflow runs.
- Job details.
- Workflow logs.
- Security scan results available in those logs.

MCP is used for inspection and analysis. Trivy remains responsible for scanning the Docker image.

## 4. CI Execution Flow

1. A developer pushes code or opens a pull request targeting `main`.
2. GitHub Actions starts the workflow.
3. The repository is checked out.
4. Java 21 is configured.
5. The Java source code is compiled.
6. The application output is tested.
7. Docker builds the application image.
8. Trivy scans the image.
9. If the security scan passes, the Docker container is executed and tested.
10. GitHub reports the workflow status.
11. Branch protection uses the required status check to determine merge eligibility.

## 5. Security and Merge Validation

The project uses a severity-based security policy.

The normal main-branch configuration fails on HIGH and CRITICAL vulnerabilities.

A separate test branch was used to change the threshold to all four severities:

`LOW,MEDIUM,HIGH,CRITICAL`

This caused the scan to fail because matching LOW and MEDIUM findings were detected.

The failed required status check demonstrated that branch protection prevents normal merging when the CI pipeline fails.

## 6. GitHub MCP Inspection Flow

GitHub Copilot connects to GitHub MCP Server and invokes available GitHub tools.

The tested operations include:

- Listing workflow runs.
- Retrieving workflow run information.
- Retrieving job information.
- Retrieving job logs.
- Comparing successful and failed security scans.

This provides a way to inspect CI execution and security results without manually navigating every Actions page.

## 7. Architecture Scope

The current implementation focuses on Continuous Integration and security validation.

It does not include:

- Continuous Deployment.
- Container registry publishing.
- Kubernetes deployment.
- Production hosting.

These capabilities are outside the current project scope.