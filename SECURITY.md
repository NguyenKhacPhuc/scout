# Security policy

Scout drives real devices, handles test credentials and can proxy network traffic, so
we take security reports seriously.

## Reporting a vulnerability

Please **do not open a public issue**. Report privately through
[GitHub private vulnerability reporting](https://github.com/NguyenKhacPhuc/scout/security/advisories/new).
Include the steps to reproduce the problem and its impact. You'll get an
acknowledgement within a few days.

These issues are in scope:

- Credentials leaking into model requests, traces, results, screenshots or logs.
- A way to bypass the purchase, consent, credential or app-boundary guards.
- The MCP server, local bridge or dashboard exposing files or actions beyond what they
  document.
- Sandbox or recording traffic escaping its declared hosts.

## Supported versions

Scout is pre-1.0. Security fixes go into the latest release.
