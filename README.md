# Java Docker CI/CD Security Pipeline

A simple Java application demonstrating an automated CI security pipeline using **GitHub Actions, Docker, Trivy, and GitHub MCP Server**.

The project compiles and tests a Java application, builds a Docker image, scans the image for vulnerabilities, and validates the container before changes can be merged into the `main` branch.

## Project Objectives

- Build a basic Java Hello World application.
- Containerize the application using Docker.
- Automate compilation, testing, and Docker image building with GitHub Actions.
- Scan Docker images for security vulnerabilities using Trivy.
- Prevent pull requests from merging when required CI security checks fail.
- Use GitHub MCP tools with GitHub Copilot to inspect repositories, workflow runs, jobs, and logs.

## Architecture

```text
Developer
    |
    v
GitHub Repository
    |
    +---- Push to main
    |
    +---- Pull Request targeting main
                |
                v
        GitHub Actions CI Pipeline
                |
                +---- Checkout Repository
                |
                +---- Set Up Java 21
                |
                +---- Compile Java Application
                |
                +---- Test Java Application
                |
                +---- Build Docker Image
                |
                +---- Scan Image with Trivy
                |
                +---- Test Docker Container
                |
                v
        Required Status Check
                |
          +-----+-----+
          |           |
        Passed      Failed
          |           |
       Eligible    Merge Blocked
       for merge   by branch protection


Separate Inspection Workflow:

Developer
    |
    v
VS Code
    |
    v
GitHub Copilot (Agent Mode)
    |
    v
GitHub MCP Server
    |
    v
GitHub Repository and Actions Information
    |
    +---- Workflow Runs
    +---- Job Details
    +---- Workflow Logs
```

## Technology Stack

| Technology | Purpose |
|---|---|
| Java 21 | Application development |
| Docker | Application containerization |
| Multi-stage Dockerfile | Separate compilation and runtime environments |
| GitHub Actions | CI automation |
| Trivy | Docker image vulnerability scanning |
| GitHub MCP Server | Access GitHub repository and Actions information |
| GitHub Copilot | Interact with GitHub MCP tools |
| Git | Source code version control |

## Project Structure

```text
java-github-actions-security-pipeline/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── docs/
│   ├── architecture.md
│   ├── security.md
│   └── github-mcp.md
│
├── screenshots/
│
├── src/
│   └── HelloWorld.java
│
├── Dockerfile
├── .dockerignore
├── .gitignore
└── README.md
```

## Application

The application is written in Java and prints:

```text
Hello World!
Welcome to GitHub Actions CI/CD Pipeline.
```

## Run Locally

### Prerequisites

- Java Development Kit (JDK) 21
- Docker
- Git

### 1. Clone the repository

```bash
git clone https://github.com/Dhanikji/java-github-actions-security-pipeline.git

cd java-github-actions-security-pipeline
```

### 2. Compile the Java application

```bash
mkdir -p out
javac -d out src/HelloWorld.java
```

### 3. Run the application

```bash
java -cp out HelloWorld
```

### 4. Build the Docker image

```bash
docker build -t java-hello-world:v1 .
```

### 5. Run the Docker container

```bash
docker run --rm java-hello-world:v1
```

## CI Pipeline

The GitHub Actions workflow is located at:

`.github/workflows/ci.yml`

The pipeline automatically runs on:

- Pushes to the `main` branch.
- Pull requests targeting the `main` branch.

### Pipeline stages

1. **Checkout:** Retrieves the repository source code.
2. **Java Setup:** Configures Java 21.
3. **Compilation:** Compiles the Java application.
4. **Application Testing:** Verifies the expected application output.
5. **Docker Build:** Builds the Docker image.
6. **Security Scan:** Scans the image using Trivy.
7. **Container Testing:** Runs the image and verifies its output.

The Docker image is built locally on the GitHub Actions runner. This project does not publish the image to a container registry or deploy it.

## Docker Security Scanning

Trivy scans the Docker image for operating system and library vulnerabilities.

Current CI configuration:

| Setting | Value |
|---|---|
| Severity threshold | `HIGH,CRITICAL` |
| Exit code | `1` |
| Vulnerability types | `os,library` |
| Ignore unfixed vulnerabilities | `false` |
| Output format | Table |

The pipeline fails when Trivy detects findings matching the configured severity threshold.

LOW and MEDIUM findings do not fail the current main-branch pipeline.

### Security validation experiment

A temporary test branch was used to validate the security policy.

The severity threshold was changed from:

`HIGH,CRITICAL`

to:

`LOW,MEDIUM,HIGH,CRITICAL`

The inspected Trivy report identified:

| Severity | Findings |
|---|---:|
| LOW | 12 |
| MEDIUM | 59 |
| HIGH | 0 |
| CRITICAL | 0 |
| **Total** | **71** |

The stricter scan failed because LOW and MEDIUM findings were included in the failure threshold.

The successful main-branch scan passed under its HIGH and CRITICAL threshold. This does not mean the image had zero vulnerabilities.

## Pull Request Protection

Branch protection is configured on `main`.

The required status check is:

`Build, Test and Security Scan`

The test pull request demonstrated the following:

1. The stricter Trivy scan detected matching findings.
2. The security scan failed.
3. The required status check failed.
4. GitHub blocked the normal pull request merge.

The test pull request was not merged.

This demonstrated that security scan results can control merge eligibility through GitHub branch protection.

## GitHub MCP Integration

GitHub MCP Server was connected to GitHub Copilot to inspect repository and GitHub Actions information.

The integration was tested for:

- Listing workflow runs.
- Retrieving workflow run details.
- Retrieving job details.
- Retrieving GitHub Actions job logs.
- Comparing successful and failed Trivy scan results.

**Important:** Trivy performs the vulnerability scanning. GitHub MCP provides access to GitHub information; it does not perform the image scan itself.

See [GitHub MCP Documentation](docs/github-mcp.md).

## Documentation

- [Architecture](docs/architecture.md)
- [Security and Trivy](docs/security.md)
- [GitHub MCP Integration](docs/github-mcp.md)

## Project Scope

This project demonstrates Continuous Integration and security validation.

It does not currently include:

- Continuous Deployment (CD).
- Docker image publishing.
- Kubernetes deployment.
- Production hosting.

## Author

Sachin Singh Dhanik

GitHub: [Dhanikji](https://github.com/Dhanikji)
