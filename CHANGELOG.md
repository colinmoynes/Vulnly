# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.0.0b9] - 2026-08-12

### Security
- Fixed stored cross-site scripting via scanner-supplied text. Advisory titles and package names come from vulnerability feeds and were not escaped before being written into the report, so crafted text could execute when the report was opened. A closing `</script>` sequence in a title also broke the report itself, leaving the vulnerability table empty.
- Fixed path traversal in advisory links. A crafted advisory identifier could point a row's link at an arbitrary page on `nvd.nist.gov` or `github.com` instead of the real advisory. Identifiers are now validated and fall back to an inactive link.

### Added
- Vulnerability table now loads 50 rows at a time with a **Load more** button, so reports with thousands of findings open quickly. Printing or exporting to PDF still includes every row.
- Reports carry a theme toggle in the header, so readers can switch between dark and light without regenerating. The choice is remembered per browser, and `--theme` still sets the theme a report opens in.
- Unknown-severity findings now appear in the severity cards, distribution chart, legend and filter buttons.

### Changed
- Report detail pills are simplified: Format, Scan Target and Package Size are removed, and Repository is renamed **Path** and shown only when it differs from the package name.

### Fixed
- Clean scans no longer report vulnerabilities. A report with no findings showed a red "Vulnerabilities Detected" banner and an action-required alert; it now shows a green no-vulnerabilities state.
- The "Immediate Action Required" alert now appears only when Critical or High severity findings are present. Scans with only Medium and Low findings show a review-recommended notice instead.
- Severity counts now reconcile. Unknown-severity findings were counted in the total but omitted from the severity cards and chart, so totals did not match the breakdown. A scan containing only unknown-severity findings also reported a total and then said no vulnerabilities were found.
- Snyk and Grype reports no longer show a blank scan date. Neither scanner records a scan timestamp, so reports read "Security scan completed on ." — they now fall back to the report date.
- Alert text no longer names severities with no findings, such as "0 Critical and 2 High".
- Reports no longer show duplicated paths such as `nginx/nginx` for image scans.
- Repository summaries no longer fail when a scanner reports a non-numeric package count.

### Removed
- The `TEMPLATE_FILE`, `TEMPLATES` and `REPO_SUMMARY_TEMPLATES` module attributes. Each report type now has a single theme-aware template. Code calling `generate_html()` or `generate_repo_summary_html()` directly should use `REPORT_TEMPLATE` / `REPO_SUMMARY_TEMPLATE` and pass `theme="light"` where a light report is wanted. The command-line interface is unaffected.

## [1.0.0b8] - 2026-04-12

### Added
- Snyk scanner support with auto-detection

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
