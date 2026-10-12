# Changelog

All notable changes are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/) (pre-1.0: minor versions may break).

## [0.1.0] - Unreleased

First public release.

- MCP server (`uvx scout-mobile`) for Android (Appium/UiAutomator2, ADB fallback) and
  iOS (Appium/XCUITest) testing from prompts or saved cases.
- Checkpoint verification with honest verdicts: Passed, Failed, Inconclusive, Blocked
  and Execution error.
- Redacted evidence: traces, model decisions, safe screenshots, recordings and ZIP
  export.
- Local dashboard for workspaces, test libraries, suites, run review, network sandbox
  and scenario matrices.
- Pluggable run pipeline with locked safety nodes.
- `SCOUT_HOME` workspace setting; an installed package stores data in `~/.scout`.
