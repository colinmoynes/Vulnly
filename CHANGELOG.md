# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.0.0b5] - 2025-04-12

### Added

- Snyk scanner support with auto-detection
- Repository summary reports for Cloudsmith repo-level summaries with package status breakdown, aggregate vulnerability counts, and per-package detail table
- Light colour theme via `--theme` flag
- Interactive doughnut charts showing severity or status distribution (powered by Chart.js)
- Severity stat cards (Total, Critical, High, Medium, Low)
- Auto-generated executive summary and action-required alert box
- Client-side search and filter buttons for the CVE / package table
- Automatic linking for CVE IDs (NVD) and GHSA IDs (GitHub Advisories)
- Customisable logo via `--logo` flag or `assets/` directory
- Input from stdin (`-`) for piping from other tools
- Input validation with helpful warnings for malformed data

## [1.0.0b4] - 2025-03-15

### Added

- Grype scanner support with auto-detection

## [1.0.0b3] - 2025-02-20

### Added

- Trivy scanner support with auto-detection

## [1.0.0b2] - 2025-01-25

### Added

- Dark theme HTML report template
- Cloudsmith scanner support

## [1.0.0b1] - 2025-01-10

### Added

- Initial beta release
- Self-contained HTML vulnerability report generation
- Cloudsmith JSON input support
- Zero external dependencies — uses only the Python standard library
