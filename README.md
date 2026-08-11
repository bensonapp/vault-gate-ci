[English](README.md) | [Русский](README.ru.md)

---

# Vault Gate CI: DevSecOps Pipeline

An automated, multi-stage security CI/CD pipeline built with GitHub Actions. This project demonstrates the integration of security controls into the software development lifecycle (SDLC) using a vulnerable REST API (VAmPI) as the test target.

## Architecture & Tools

The pipeline is triggered automatically on every push to the `main` branch and implements three layers of automated security testing:

1. **SAST (Static Application Security Testing)**
   * **Tool:** Semgrep
   * **Purpose:** Scans the Python/FastAPI source code for logic flaws, hardcoded secrets, and insecure configurations before the application is built.

2. **SCA (Software Composition Analysis)**
   * **Tool:** Trivy
   * **Purpose:** Analyzes project dependencies to detect known vulnerabilities (CVEs) in third-party libraries. Reports are exported and saved as CI/CD artifacts.

3. **DAST (Dynamic Application Security Testing)**
   * **Tool:** Docker & OWASP ZAP (Baseline Scan)
   * **Purpose:** The API is built and deployed in an ephemeral Docker container. OWASP ZAP performs passive dynamic scanning against the running target (`http://localhost:5000`) to detect misconfigurations and runtime vulnerabilities (e.g., missing security headers, CSRF). ZAP automatically creates GitHub Issues for identified risks.

## Security Findings & Triage

### 1. Pipeline Architecture & Orchestration
The pipeline successfully initializes the runner, performs static analysis, checks dependencies, builds the Docker image, and executes dynamic attacks against the running container.

![Successful CI/CD Run](images/success-pipeline.png)

### 2. Security Gate Execution
The pipeline enforces a strict build block (exit-code: 1) upon detecting `HIGH` or `CRITICAL` vulnerabilities in project dependencies. Reports are preserved as CI/CD artifacts.

![Trivy Gate](images/trivy-gate.jpg)

### 3. Automated Bug Tracker Integration
Upon completion of the DAST scan, OWASP ZAP automatically converts identified vulnerabilities into GitHub Issues for the development team.

![ZAP Issues](images/zap-issues.png)

### Key Vulnerabilities Identified
* **Dependency Risks (SCA):** Identified several outdated Python packages with High/Critical CVEs. *Mitigation:* Pin dependencies in `requirements.txt` to patched versions.
* **Runtime Risks (DAST):** OWASP ZAP reported missing Anti-CSRF tokens and missing Content Security Policy (CSP) headers. *Mitigation:* Implement strict CORS policies and security headers via FastAPI middleware.

## How to Run
Check the `.github/workflows/sec-pipeline.yml` file for the complete GitHub Actions configuration. To trigger the pipeline, push a commit to the `main` branch.
