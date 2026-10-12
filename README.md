# Scout

<!-- mcp-name: io.github.NguyenKhacPhuc/scout -->

[![CI](https://github.com/NguyenKhacPhuc/scout/actions/workflows/ci.yml/badge.svg)](https://github.com/NguyenKhacPhuc/scout/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/scout-mobile)](https://pypi.org/project/scout-mobile/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/NguyenKhacPhuc/scout/blob/main/LICENSE)

**AI-driven mobile testing that your coding agent can run.** Describe a flow in plain
language. Scout then drives a real Android or iOS app through Appium, checks the result
against checkpoints you defined, and keeps the evidence: screenshots, a step-by-step
trace, model decisions and screen recordings.

Scout runs as an [MCP](https://modelcontextprotocol.io) server. You can use it from
Claude Desktop, Claude Code, Codex or any other MCP client. There is also an optional
local dashboard for test libraries, runs and review.

```text
You:   On the connected Pixel, sign in with my test account and confirm the
       account settings screen appears.
Agent: run_mobile_test_from_prompt → Passed
       required checkpoints: login_form ✓ (during run), account_screen ✓ (final)
       evidence: screenshots, redacted trace, model decisions
```

## Why Scout

- **A test passes only when it is verified.** The agent saying it is "done" is not
  enough. Scout checks each required checkpoint against what is actually on screen.
  - If a check can't be verified, the result is `Inconclusive`.
  - If a starting condition isn't met, the result is `Blocked`.
  - Device and model failures are reported as `Execution error`.
- **Evidence you can share.** Every run saves `result.json`, a redacted action trace,
  the model's decisions with their confidence scores, safe screenshots and MP4 clips.
  You can export them as a ZIP for a PR or a ticket.
- **Explore first, then repeat the test.** Explore an app from a prompt, then save the
  flow as a versioned test case with typed actions. Rerun it, combine cases into suites,
  and compare results across Android and iOS.
- **Safety by default.** Scout refuses purchases, transfers, subscriptions and account
  deletion. It stops before legal or privacy consent unless you explicitly allow it.
  It keeps credentials out of model requests and saved output, and it restores any
  device settings it changed.
- **Network sandbox.** Replay synthetic HTTP/WebSocket fixtures through mitmproxy, or
  record real traffic and edit it into fixtures. Scenario matrices then run one case
  against several backend variants, for example error and empty states.

## Quickstart (MCP server)

**Prerequisites**

- [uv](https://docs.astral.sh/uv/) (or Python 3.11+ with pip).
- For Android: `adb`, plus an authorized device or emulator.
- Appium 3 with the driver for your platform:

  ```sh
  npm install -g appium
  appium driver install uiautomator2    # Android
  appium driver install xcuitest        # iOS (macOS only)
  ```

  Scout starts Appium on `127.0.0.1:4723` when it isn't already running.

**1. Add the server to your MCP client.**

For Claude Code:

```sh
claude mcp add scout -- uvx scout-mobile
```

For Claude Desktop, add this under `mcpServers` in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "scout": { "command": "uvx", "args": ["scout-mobile"] }
  }
}
```

**2. Configure a model.** Put the key in `~/.scout/.env`. Scout reads it locally, so
the key never goes through your chat:

```sh
mkdir -p ~/.scout
curl -fsSL https://raw.githubusercontent.com/NguyenKhacPhuc/scout/main/.env.example -o ~/.scout/.env
```

Then fill in **one** of the two options:

- **TypeSafe Jev (default).** Set `TYPESAFE_API_KEY`. This is the full feature set,
  including model-judged `classification` checkpoints.
- **Any OpenAI-compatible chat model.** Set `SCOUT_DECISION_AGENT=llm`, plus
  `TEXT_MODEL_API_KEY`, `TEXT_MODEL_BASE_URL` and `TEXT_MODEL`. The model then chooses
  actions. Checkpoints with `text_present` / `text_absent` work without TypeSafe.
  `classification` checkpoints still need `TYPESAFE_API_KEY`.

For sign-in tests, also set `SCOUT_TEST_EMAIL` and `SCOUT_TEST_PASSWORD` for a
disposable test account.

**3. Ask your agent.** For example: *"List my Android devices. Then use Scout to open
com.example.app, go to Settings, and confirm the version number is shown."* Run
`list_android_devices`, `search_android_packages` and `get_test_case_template`
first, so the agent picks the right device and app and defines good checkpoints.

Runs and evidence are stored in `~/.scout/runner-output/`. Set `SCOUT_HOME` to
store them somewhere else.

## Full setup (dashboard + MCP from source)

The dashboard is a local web app for workspaces, test libraries, suites, run history,
evidence review, the sandbox and scenario matrices. It ships with the source checkout:

```sh
git clone https://github.com/NguyenKhacPhuc/scout.git && cd scout
uv sync
npm ci --prefix tooling/appium          # project-local Appium; see docs/appium-trial.md
npm ci --prefix dashboard
cp .env.example .env                    # then fill in values locally
```

Point your MCP client at the checkout. Prompt-driven runs start the dashboard and the
device bridge on demand:

```sh
claude mcp add scout -- uv run --directory /path/to/scout scout-mcp
```

You can also start them yourself: `cd dashboard && npm run dev` for the dashboard and
`uv run python -m scout_mobile.local_server` for the device bridge. See the
[dashboard setup](https://github.com/NguyenKhacPhuc/scout/blob/main/dashboard/README.md).

The CLI runs an exported case directly:

```sh
uv run scout --case case.json --package com.example.app --serial DEVICE_SERIAL
uv run scout nodes        # print the run pipeline
```

## How it works

Each run is a pipeline of inspectable, pluggable nodes in three phases:

1. **Setup** runs once. It takes a per-device lease, checks the app is installed,
   launches it, and checks the starting conditions on the opening screen.
2. **Step loop** repeats until the run ends:
   - Scout observes the screen through the native accessibility tree.
   - A model picks one operation and one target it actually observed.
   - Guards check for app-boundary, consent, purchase and credential problems.
   - Scout re-checks the target and acts on it.
   - When the model reports DONE, checkpoints decide the verdict.
3. **Teardown** restores device settings and writes the evidence.

Run `scout nodes` to print the live pipeline. Plugins can insert, replace or disable
nodes that aren't locked. Safety nodes are locked and can't be removed or replaced. See
[architecture](https://github.com/NguyenKhacPhuc/scout/blob/main/docs/architecture.md).

## Documentation

| Guide | Covers |
| --- | --- |
| [MCP tools](https://github.com/NguyenKhacPhuc/scout/blob/main/docs/mcp.md) | Every tool, prompt-driven runs, saved cases, suites, evidence |
| [Android / Appium](https://github.com/NguyenKhacPhuc/scout/blob/main/docs/appium-trial.md) | Appium install, ADB fallback, measurements |
| [iOS](https://github.com/NguyenKhacPhuc/scout/blob/main/docs/ios-appium.md) | XCUITest, WebDriverAgent signing, simulators and devices |
| [Network sandbox](https://github.com/NguyenKhacPhuc/scout/blob/main/docs/sandbox.md) | Fixtures, recording, proxy routing, test CA |
| [Test accounts](https://github.com/NguyenKhacPhuc/scout/blob/main/docs/test-account.md) | Keeping credentials local and redacted |
| [Architecture](https://github.com/NguyenKhacPhuc/scout/blob/main/docs/architecture.md) | Layers, pipeline nodes, plugins |
| [Agent rules](https://github.com/NguyenKhacPhuc/scout/blob/main/AGENTS.md) | Instructions to copy into your app repo's `AGENTS.md` / `CLAUDE.md` |

The dashboard also has a searchable in-app guide at `/docs`.

## Status

Scout is **alpha**: interfaces may change between minor versions. Android is
exercised on emulators and physical devices. iOS runs through XCUITest on macOS;
see the iOS guide for its limitations. Missing or unverifiable evidence is reported as inconclusive,
never as a pass. Please [open an issue](https://github.com/NguyenKhacPhuc/scout/issues)
if a screen or flow doesn't work as you expect.

## Contributing

Contributions are welcome. See
[CONTRIBUTING.md](https://github.com/NguyenKhacPhuc/scout/blob/main/CONTRIBUTING.md),
and look for issues labelled `good first issue` or `help wanted`.

## License

[MIT](https://github.com/NguyenKhacPhuc/scout/blob/main/LICENSE). The runner adapts the
observe–choose–act design of [Jev Ultrafast](https://github.com/browser-use/jev-ultrafast)
(MIT); see [third-party notices](https://github.com/NguyenKhacPhuc/scout/blob/main/THIRD_PARTY_NOTICES.md).
