# Contributing to Scout

Thanks for helping. Bug reports, docs fixes, new device quirks, pipeline plugins and
dashboard work are all welcome.

## Development setup

```sh
git clone https://github.com/NguyenKhacPhuc/scout.git && cd scout
uv sync --extra sandbox
npm ci --prefix dashboard        # only for dashboard work
npm ci --prefix tooling/appium   # only for on-device runs; see docs/appium-trial.md
cp .env.example .env             # fill locally; never commit it
```

## Running the checks

The Python checks are synthetic contract tests. They need no device and make no model
calls. Run them the same way CI does:

```sh
for f in checks/test_*.py; do PYTHONPATH=checks uv run python -m unittest "checks.$(basename "${f%.py}")"; done
```

Run the dashboard type-check with:

```sh
cd dashboard && npx tsc --noEmit -p .
```

If a change affects on-device behaviour, say in the PR which device, OS version and app
you ran it on. Keep synthetic checks and on-device runs clearly separate: a passing
synthetic check is not evidence of device behaviour.

## Ground rules

- **Safety nodes stay locked.** Don't weaken the purchase, consent, credential or
  app-boundary guards, or make them optional by default. New risky capabilities must
  require an explicit, per-run authorization.
- **No real app data.** Fixtures, tests and docs use synthetic package IDs such as
  `com.example.app`, synthetic accounts and synthetic payloads. Never include real
  credentials, device serials, internal hostnames or another company's app contracts.
- **Verdicts must stay honest.** Missing or unverifiable evidence must produce
  `Inconclusive` or `Blocked`, never `Passed`.
- Match the style of the surrounding code. Keep changes focused, and give them a test
  when behaviour changes.

## Pull requests

1. Fork, then create a branch from `main`.
2. Make the change, add or update a check, and run the checks above.
3. Open a PR that explains what changed, why, and how you verified it.

## Releasing (maintainers)

1. Bump `version` in `pyproject.toml` and both versions in `server.json`, and update
   `CHANGELOG.md`.
2. Merge to `main`, then tag and push: `git tag v0.2.0 && git push origin v0.2.0`.
3. The Release workflow builds the package and publishes it to PyPI (trusted
   publishing). It then creates the GitHub release and publishes `server.json` to the
   MCP Registry.
