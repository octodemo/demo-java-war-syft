# Demo Java WAR — Syft + GitHub Dependency Graph Demo

A minimal Maven WAR application with intentionally **outdated dependencies** for demonstrating:

- **Syft** SBOM generation from a packaged WAR file
- **GitHub Dependency Graph Submission API** — Syft as the authoritative source of dependency data
- **GHAS Dependabot alerts** surfaced exclusively via Syft, not native pom.xml scanning
- **CodeQL** code scanning for Java

> ⚠️ **This repository contains known-vulnerable dependencies. Do not deploy to production.**

---

## How the Demo Works

The native GitHub Dependency Graph scanner **cannot** resolve the dependencies in this repo because:

1. `pom.xml` (root) is a **bare aggregator POM** with no `<dependencies>` — just a `<modules>` declaration
2. `webapp/pom.xml` (child) declares versions as **Maven property placeholders** (`${log4j.version}`, `${spring.version}`, etc.) which the static scanner fails to resolve

The build workflow uses `anchore/sbom-action` (Syft) to scan the compiled `demo-java-war.war` and submit an SPDX SBOM directly to the **GitHub Dependency Submission API**. This populates the Dependency Graph and triggers Dependabot alerts — proving that Syft finds what the native tooling misses.

---

## Vulnerable Dependencies Included

| Library | Version | CVE | Severity | Common Name |
|---------|---------|-----|----------|-------------|
| `log4j-core` | 2.14.1 | CVE-2021-44228 | Critical | Log4Shell |
| `log4j-core` | 2.14.1 | CVE-2021-45046 | Critical | Log4Shell variant |
| `spring-webmvc` | 5.3.17 | CVE-2022-22965 | Critical | Spring4Shell |
| `commons-text` | 1.10.0 | — (patched) | — | Text4Shell |
| `jackson-databind` | 2.13.0 | CVE-2022-42003 | High | — |
| `snakeyaml` | 1.29 | CVE-2022-25857 | High | — |

---

## Prerequisites

- Java 11+
- Maven 3.8+
- [Syft](https://github.com/anchore/syft) (`brew install syft` on macOS)

---

## Build

```bash
mvn package -DskipTests
# Output: webapp/target/demo-java-war.war
```

---

## Syft SBOM Generation (local)

### Table view (quick scan)
```bash
syft scan webapp/target/demo-java-war.war -o table
```

### CycloneDX JSON
```bash
syft scan webapp/target/demo-java-war.war -o cyclonedx-json=sbom.cdx.json
```

### SPDX JSON
```bash
syft scan webapp/target/demo-java-war.war -o spdx-json=sbom.spdx.json
```

### Pipe into Grype for vulnerability matching
```bash
syft scan webapp/target/demo-java-war.war -o cyclonedx-json | grype
```

---

## GitHub Actions

On every push to `main`:

| Workflow | What it does |
|----------|--------------|
| `build-and-sbom.yml` | Builds WAR → Syft CycloneDX artifact + SPDX submission to Dependency Graph API |
| `codeql.yml` | CodeQL Java analysis, results in Security → Code scanning |

SBOM artifacts are downloadable from **Actions → latest run → Artifacts**.

---

## GHAS Features Demonstrated

### Dependabot Alerts
**Security → Dependabot alerts** — populated via Syft SBOM submission, not pom.xml scanning.

### Dependency Graph
**Insights → Dependency graph** — sourced from the `anchore/sbom-action` SPDX upload.

### CodeQL Code Scanning
**Security → Code scanning alerts** — CodeQL Java analysis results.

### Secret Scanning
Enabled by default for public repos — **Security → Secret scanning alerts**.
