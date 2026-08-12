<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/colinmoynes/vulnly-core/main/assets/icons/vulnly-wordmark-dark.png">
  <img src="https://raw.githubusercontent.com/colinmoynes/vulnly-core/main/assets/icons/vulnly-wordmark.png" alt="Vulnly" width="320">
</picture>

Generate self-contained HTML vulnerability reports from Cloudsmith, Trivy, Grype and Snyk.
Every report is a single file that opens with no network access.

![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)
![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)

## Installation

Install from PyPI (Currently in beta):

```bash
pip install vulnly
```

## Features

- **Fully offline reports** — a single HTML file with fonts, charting library and logo embedded. No CDN, no web fonts, no outbound requests when opened, so reports work on airgapped hosts and keep working once archived
- Supports multiple scanner sources: **Cloudsmith**, **Trivy**, **Grype**, and **Snyk** (auto-detected or via `--source` flag)
- **Repository summary reports** — automatically detects Cloudsmith repo-level summaries and generates a dedicated overview report with package status breakdown, aggregate vulnerability counts, and per-package detail table
- Dark and light colour themes via `--theme` flag, plus an in-report toggle so readers can switch without regenerating
- Interactive doughnut charts showing severity or status distribution (Chart.js, bundled)
- Severity stat cards (Total, Critical, High, Medium, Low, Unknown)
- Auto-generated executive summary and action-required alert box
- Client-side search and severity filters, with the table paginated so large reports open quickly
- Automatic linking for CVE IDs (NVD) and GHSA IDs (GitHub Advisories)
- Customisable logo — supply via `--logo` flag or drop an image into `assets/`
- Accepts input from a file or stdin (`-`), making it easy to pipe from other tools
- Input validation with helpful warnings for malformed data
- Zero external dependencies — uses only the Python standard library

## Quick Start

```bash
vulnly sample_data.json
```

This reads `sample_data.json` and writes the report to `reports/` (the default output directory).

### Specify the scanner source

The tool auto-detects the input format, but you can be explicit:

```bash
vulnly scan.json --source trivy
vulnly scan.json --source grype
vulnly scan.json --source snyk
vulnly scan.json --source cloudsmith
```

If `--source` is specified and the input doesn't match the expected format, the tool exits with a clear error message.

### Repository summary reports

Cloudsmith repo-level summary JSON is automatically detected and generates a dedicated repository overview report:

```bash
vulnly repo_summary.json
```

The output includes package status cards (vulnerable / no issues / not scanned), aggregate vulnerability severity breakdown, a status distribution chart, and a searchable/filterable package table.

### Specify a custom output path

```bash
vulnly sample_data.json -o my_report.html
```

### Custom logo

Use the `--logo` flag to supply a custom logo image. It will be embedded directly in the HTML report header as a base64 data URI:

```bash
vulnly sample_data.json --logo my-company-logo.png
```

**Logo requirements:**

| Constraint | Limit |
|---|---|
| Max file size | 2 MB |
| Max dimensions | 512 × 512 px |
| Recommended size | 128 × 128 px to 256 × 256 px |
| Supported formats | `.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.webp`, `.ico` |

If a custom logo exceeds the file size or dimension limits, a warning is printed and the default Vulnly logo is used instead.

If no `--logo` is provided, the built-in Vulnly logo is used.

### Theme

Choose the theme the report opens in — dark is the default:

```bash
vulnly sample_data.json --theme light
vulnly sample_data.json --theme dark
```

Every report also carries both palettes, so readers can switch with the
toggle in the report header regardless of which theme it was generated in.
Their choice is remembered per browser.

### Read from stdin

Pipe JSON directly from another command:

```bash
cat scan_output.json | vulnly -
```

#### Cloudsmith CLI

Pipe vulnerability scan results directly from the Cloudsmith CLI:

```bash
cloudsmith vulnerabilities WORKSPACE/REPO/PACKAGE_IDENTIFIER --output-format json | vulnly -
```

#### Trivy

Scan an image with Trivy and pipe directly to Vulnly:

```bash
trivy image -f json nginx:latest | vulnly --source trivy -
```

#### Grype

Scan an image with Grype and pipe directly to Vulnly:

```bash
grype nginx:latest -o json | vulnly --source grype -
```

#### Snyk

Test a container image with Snyk and pipe directly to Vulnly:

```bash
snyk container test nginx:latest --json | vulnly --source snyk -
```

<img src="https://raw.githubusercontent.com/colinmoynes/vulnly/main/assets/report-demo.jpg">

## Usage

```
usage: vulnly [-h] [-o OUTPUT]
              [--logo LOGO] [--theme {dark,light}]
              [--source SOURCE]
              input

Generate an HTML vulnerability report from a JSON input file.

positional arguments:
  input                 Path to the JSON file containing vulnerability data
                        (use '-' for stdin)

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT, --output OUTPUT
                        Output HTML file path (default: auto-generated in
                        reports/ subfolder)
  --logo LOGO           Path to a custom logo image (max 512×512px, 2 MB;
                        embedded as base64 in the report)
  --theme {dark,light}  Report colour theme (default: dark)
  --source SOURCE       Scanner source format: cloudsmith, trivy, grype, snyk
                        (auto-detected if omitted)
```

### Environment variables

| Variable | Effect |
| --- | --- |
| `VULNLY_NO_UPDATE_CHECK` | Set to any value to skip the PyPI version check. |

The version check is skipped automatically when stderr is not a terminal, so
it makes no network request in CI or when output is piped.

## Offline reports

A generated report is a single HTML file that makes **no network requests when
opened**. Everything it needs is embedded:

| Asset | Source | Licence |
| --- | --- | --- |
| Charting | Chart.js 4.4.0, bundled and inlined | MIT |
| Typography | Inter, variable font covering weights 400–800 | SIL OFL 1.1 |
| Logo | Embedded as a data URI | — |

This means reports render identically on airgapped hosts and in restricted
browser environments, and keep rendering years later when a CDN URL would have
moved. It also means opening a report tells no third party that you did.

CVE and GHSA identifiers still link out to NVD and GitHub Advisories, but those
are ordinary links — nothing is fetched unless the reader clicks one.

The generator itself contacts the network only for its PyPI version check,
which can be disabled with `VULNLY_NO_UPDATE_CHECK` as described above.

## Input JSON Format

The generator accepts **multiple input formats** — it auto-detects which one you provide, or you can specify explicitly with `--source`.

### 1. Cloudsmith package format

Pass raw JSON output from the Cloudsmith CLI. The tool extracts namespace, repository, package metadata, scan target, and all vulnerabilities automatically.

The expected structure has a top-level `data` object containing `package`, `scans`, etc.:

```json
{
    "data": {
        "created_at": "2025-06-09T08:39:20.441354Z",
        "identifier": "TZR5N4HaO7nTclhz",
        "package": {
            "name": "log4j",
            "url": "https://example.com/packages/my-org/java/wAoMy00juV6N/",
            "version": "28a05d8e..."
        },
        "scans": [
            {
                "target": "Java",
                "type": "jar",
                "results": [
                    {
                        "vulnerability_id": "CVE-2021-45046",
                        "severity": "Critical",
                        "title": "DoS in log4j 2.x...",
                        "package_name": "org.apache.logging.log4j:log4j-core",
                        "affected_version": { "version": "2.8.1" },
                        "fixed_version": { "version": "2.16.0, 2.12.2" },
                        "cvss_scores": null
                    }
                ]
            }
        ]
    }
}
```

### 2. Cloudsmith repository summary format

Pass the repo-level summary JSON from Cloudsmith. This is auto-detected and produces a repository overview report instead of a per-package vulnerability report:

```json
{
    "data": {
        "owner": "my-org",
        "packages": [
            {
                "package": "cloudsmith.io/jdk:9ea72e62...",
                "slug_perm": "XXVmdsZn7OEh",
                "status": "vulnerable",
                "vulnerabilities": {
                    "critical": 0,
                    "high": 0,
                    "low": 1,
                    "medium": 2,
                    "unknown": 0
                }
            },
            {
                "package": "cloudsmith.io/jdk:5659dd01...",
                "slug_perm": "Ft30zuSymLol",
                "status": "no_issues_found",
                "vulnerabilities": {
                    "critical": 0,
                    "high": 0,
                    "low": 0,
                    "medium": 0,
                    "unknown": 0
                }
            }
        ],
        "repository": "chainguard"
    }
}
```

The report includes status cards (vulnerable, no issues, not scanned), aggregate severity totals, a doughnut chart of package status, and a filterable package table.

Package status values: `vulnerable`, `no_issues_found`, `no_scan`.

### 3. Trivy format

Pass Trivy's JSON output (`trivy image -f json`). The tool maps `ArtifactName`, `Results[].Vulnerabilities[]`, CVSS scores, and fix versions:

```bash
trivy image -f json nginx:latest > trivy-output.json
vulnly trivy-output.json
```

### 4. Grype format

Pass Grype's JSON output (`grype -o json`). The tool maps `matches[]`, artifact metadata, CVSS scores, and fix versions:

```bash
grype nginx:latest -o json > grype-output.json
vulnly grype-output.json
```

### 5. Snyk format

Pass Snyk's JSON output (`snyk container test --json`). The tool maps `vulnerabilities[]`, CVSS scores, CVE identifiers, and fix versions. Duplicate vulnerability paths are automatically deduplicated:

```bash
snyk container test nginx:latest --json > snyk-output.json
vulnly snyk-output.json
```

### 6. Simplified format

A flat JSON structure is also supported for custom integrations:

```json
{
    "scan_date": "2026-03-13",
    "repository": "my-org/production-repo",
    "package_name": "my-application",
    "package_version": "1.4.2",
    "package_format": "Docker",
    "scan_target": "debian 9.4",
    "package_size": "~77.4 MB",
    "scan_id": "TZR5N4HaO7nTclhz",
    "vulnerabilities": [
        {
            "severity": "critical",
            "identifier": "CVE-2026-1234",
            "package": "openssl",
            "affected_version": "1.1.1t",
            "fixed_version": "1.1.1u",
            "title": "Heap buffer overflow in OpenSSL",
            "cvss": 9.8
        }
    ]
}
```

#### Top-level fields

| Field | Required | Description |
|---|---|---|
| `scan_date` | Yes | Date the scan was performed (`YYYY-MM-DD`) |
| `repository` | Yes | Namespace/repository path (e.g. `my-org/production-repo`) |
| `package_name` | Yes | Name of the scanned package |
| `package_version` | Yes | Version of the scanned package |
| `package_format` | No | Package format (e.g. `Docker`, `Maven`). Defaults to `Unknown` |
| `scan_target` | No | Scan target OS/platform (e.g. `debian 9.4`). Defaults to `Unknown` |
| `package_size` | No | Human-readable package size (e.g. `~77.4 MB`). Defaults to `Unknown` |
| `scan_id` | No | Unique scan identifier. Defaults to `N/A` |
| `vulnerabilities` | Yes | Array of vulnerability objects |

#### Vulnerability object fields

| Field | Required | Description |
|---|---|---|
| `severity` | Yes | One of `critical`, `high`, `medium`, `low` (case-insensitive) |
| `identifier` | Yes | CVE ID or other identifier (e.g. `GHSA-...`). CVE IDs are auto-linked to NVD; GHSA IDs are linked to GitHub Advisories |
| `package` | Yes | Name of the affected dependency |
| `affected_version` | No | The vulnerable version |
| `fixed_version` | No | The version containing the fix, or `"N/A"` / omit if no fix is available |
| `title` | No | Short description of the vulnerability. Auto-generated if omitted |
| `cvss` | No | CVSS v3 score as a number (e.g. `9.8`). Displayed as `N/A` if omitted |

## Requirements

- Python 3.10+

## Terminal Output

The script prints a severity breakdown after generating the report:

```
Report generated: reports/nginx_nginx_c755eb98e755_grype_vulnerability_report.html (162 vulnerabilities)
  HIGH: 13  MEDIUM: 29  LOW: 7
```

When no `-o` is specified, reports are saved to the `reports/` subfolder. This directory is git-ignored by default.

## License

This project is licensed under the Apache License 2.0 — see the [LICENSE](LICENSE) file for details.
