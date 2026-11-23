# GitHub MCP Integration

## 1. Overview

This project integrates GitHub MCP Server with GitHub Copilot to inspect repository information and Continuous Integration (CI) execution results.

The integration allows an AI assistant to interact with GitHub through available MCP tools.

MCP stands for Model Context Protocol. It provides a standardized way for AI applications to connect with external tools and services.

## 2. Architecture

```text
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
GitHub Services
    |
    +---- Repository Information
    |
    +---- Branches and Commits
    |
    +---- Pull Requests
    |
    +---- GitHub Actions Workflows
    |
    +---- Workflow Jobs and Logs
```

## 3. MCP Configuration

The GitHub MCP server was configured in the VS Code workspace using:

`.vscode/mcp.json`

Configuration:

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

GitHub Copilot Agent Mode was used to connect to the configured MCP server and invoke available GitHub tools.

## 4. Operations Tested

The following operations were successfully tested using GitHub MCP tools:

| Operation | Purpose |
|---|---|
| Repository inspection | Read repository information |
| Branch listing | Inspect available branches |
| Commit listing | Review repository history |
| Pull request inspection | Review pull request information |
| Workflow run listing | Retrieve GitHub Actions runs |
| Workflow run details | Inspect individual workflow runs |
| Job details | Retrieve individual job information |
| Job log retrieval | Inspect CI and Trivy output |

The GitHub MCP integration successfully retrieved workflow runs, job details, and Trivy job logs.

## 5. Trivy Results Inspection

GitHub MCP tools were used to inspect two GitHub Actions workflow runs and retrieve their job logs.

### Successful main-branch run

- Workflow run ID: `45122522740`
- Job ID: `137519332139`
- Result: Successful
- Configured severity threshold: HIGH and CRITICAL

The workflow passed because no findings matching the configured HIGH and CRITICAL failure threshold caused the scan to fail.

The successful result does not establish that the image had zero LOW or MEDIUM findings.

### Failed test-branch run

- Workflow run ID: `45127532371`
- Job ID: `137535227464`
- Result: Failed
- Total vulnerability findings: 71
- LOW findings: 12
- MEDIUM findings: 59
- HIGH findings: 0
- CRITICAL findings: 0
- Configured severity threshold: LOW, MEDIUM, HIGH, and CRITICAL

### Detailed findings

| Target | Total | LOW | MEDIUM | HIGH | CRITICAL |
|---|---:|---:|---:|---:|---:|
| Ubuntu 26.04 | 70 | 12 | 58 | 0 | 0 |
| `usr/bin/pebble` | 1 | 0 | 1 | 0 | 0 |
| **Total** | **71** | **12** | **59** | **0** | **0** |

The stricter test configuration included LOW and MEDIUM vulnerabilities in the failure threshold. Therefore, Trivy returned a non-zero exit code and the GitHub Actions job failed.

These counts represent vulnerability findings, not necessarily unique CVEs. Results can change with image contents and vulnerability database updates.

This inspection demonstrated how GitHub MCP tools can retrieve CI logs and help an AI assistant analyze security scan results.

## 6. Responsibilities of Each Component

It is important to distinguish between the responsibilities of the tools used in this project.

| Component | Responsibility |
|---|---|
| GitHub Actions | Executes the CI workflow |
| Docker | Builds and runs the container image |
| Trivy | Scans the image for vulnerabilities |
| GitHub MCP Server | Provides GitHub tools to an AI client |
| GitHub Copilot | Uses MCP tools to inspect and analyze GitHub information |
| Branch Protection | Enforces required status checks before normal merging |

GitHub MCP does not perform Docker image vulnerability scanning. It provides access to GitHub information, including the workflow logs containing Trivy results.

## 7. Security Considerations

MCP tools can have different permission levels depending on their configuration and the tools available.

During this project, Copilot was instructed to perform repository and workflow inspection without modifying files or repository settings.

The integration was used to retrieve information and analyze CI results.

No repository modifications were made during the MCP verification exercise.

## 8. Conclusion

The GitHub MCP integration was successfully tested with GitHub Copilot.

It demonstrated how an AI assistant can inspect repository information, retrieve workflow execution details, access job logs, and analyze security scan results through GitHub tools.

This integration complements the CI pipeline by making workflow and security information accessible for AI-assisted inspection.