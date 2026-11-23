# Security Scanning and Validation

## 1. Overview

This project integrates Trivy into the GitHub Actions CI pipeline to scan Docker images for known vulnerabilities.

The purpose is to identify security issues before changes are allowed to merge into the main branch.

## 2. Trivy Integration

Trivy is integrated using the GitHub Action:

`aquasecurity/trivy-action@v0.36.0`

The scanner checks the locally built Docker image for:

- Operating system vulnerabilities.
- Application library vulnerabilities.

The scan runs after the Docker image has been built.

## 3. Security Configuration

The current CI configuration is:

| Setting | Value |
|---|---|
| Scanner | Trivy |
| Image | `java-hello-world:${{ github.sha }}` |
| Output format | Table |
| Vulnerability types | `os,library` |
| Severity threshold | `HIGH,CRITICAL` |
| Exit code on matching findings | `1` |
| Ignore unfixed vulnerabilities | `false` |

### Severity levels

| Severity | Meaning |
|---|---|
| CRITICAL | Extremely serious vulnerability |
| HIGH | Serious vulnerability |
| MEDIUM | Moderate vulnerability |
| LOW | Lower-severity vulnerability |

The current main-branch pipeline fails when HIGH or CRITICAL findings are detected.

LOW and MEDIUM findings do not fail the current main-branch scan, although they can still be reported.

## 4. How the Pipeline Handles Scan Results

The Trivy action uses:

`exit-code: '1'`

This means that when findings match the configured severity threshold, Trivy returns a non-zero exit code.

GitHub Actions marks the security scan step as failed, and subsequent steps that depend on normal success are skipped.

If no findings match the configured threshold, the scan passes and the workflow continues to the Docker container test.

## 5. Security Validation Experiment

A temporary test branch named `test-branch-protection` was created to validate the security policy.

The normal configuration:

`HIGH,CRITICAL`

was temporarily changed to:

`LOW,MEDIUM,HIGH,CRITICAL`

This made the pipeline fail when LOW and MEDIUM vulnerabilities were detected.

The test pull request was not merged.

### Results from the inspected workflow runs

| Metric | Successful main run | Failed test run |
|---|---:|---:|
| Workflow run ID | `45122522740` | `45127532371` |
| Job ID | `137519332139` | `137535227464` |
| Total findings | Not independently established | 71 |
| LOW | Not independently established | 12 |
| MEDIUM | Not independently established | 59 |
| HIGH | Not independently established | 0 |
| CRITICAL | Not independently established | 0 |
| Configured threshold | `HIGH,CRITICAL` | All severities |
| Trivy result | Passed | Failed |

### Detailed results from the failed test scan

| Target | Total | LOW | MEDIUM | HIGH | CRITICAL |
|---|---:|---:|---:|---:|---:|
| Ubuntu 26.04 | 70 | 12 | 58 | 0 | 0 |
| `usr/bin/pebble` | 1 | 0 | 1 | 0 | 0 |
| **Total** | **71** | **12** | **59** | **0** | **0** |

The failed scan identified vulnerabilities in operating system packages and the Go standard library bundled in the `pebble` binary.

The findings represent vulnerability instances, not necessarily unique CVEs. A single CVE can affect multiple packages.

The successful main run passed under the HIGH and CRITICAL threshold. This does not establish that the image was completely vulnerability-free; it establishes that no findings matching the configured failure threshold caused the scan to fail.

Vulnerability results can change with image contents, Trivy versions, and vulnerability database updates.

## 6. Branch Protection

Branch protection was configured for the `main` branch.

The required status check is:

`Build, Test and Security Scan`

The test pull request demonstrated the following behavior:

1. The stricter Trivy scan detected matching findings.
2. The security scan failed.
3. The required GitHub Actions status check failed.
4. GitHub blocked the normal pull request merge.

This validates the connection between security scanning and merge protection.

## 7. Security Benefits

This implementation demonstrates:

- Automated Docker image vulnerability scanning.
- Severity-based security enforcement.
- Early detection of known vulnerabilities.
- CI pipeline failure when configured security thresholds are exceeded.
- Pull request merge protection through required status checks.

## 8. Current Limitations

The current project has some security improvements that could be considered in a future iteration:

- Pin GitHub Actions to immutable commit SHAs.
- Pin Docker base images by digest.
- Run the application as a non-root user.
- Upload Trivy results in SARIF format to GitHub Code Scanning.
- Enable GitHub secret scanning and push protection where available.

These are optional enhancements and are not part of the current implementation.

## 9. Conclusion

Trivy is integrated into the CI pipeline, and its exit status controls the workflow result.

The stricter-threshold experiment demonstrated that matching vulnerabilities cause the pipeline to fail. Branch protection then prevents the normal merge process when the required security check fails.

GitHub MCP tools were also used to inspect workflow runs, retrieve job details, and access Trivy logs, demonstrating AI-assisted CI/CD security analysis.