# apple-ui-test-automation

![Platforms](https://img.shields.io/badge/platforms-iOS%20%C2%B7%20iPadOS%20%C2%B7%20macOS-blue)
![License](https://img.shields.io/badge/license-MIT-green)

AI-driven UI test automation for iOS, iPadOS, and macOS apps — powered by Claude Code, xcodebuildmcp, and AXe CLI.

## Prerequisites

| Tool | Install |
|---|---|
| `xcodebuildmcp` | `npm install -g xcodebuildmcp@latest` |
| `axe` | [github.com/cameroncooke/AXe](https://github.com/cameroncooke/AXe) |

**macOS only:** Grant Accessibility permission to your terminal — System Settings → Privacy & Security → Accessibility.

Verify both are installed:
```bash
xcodebuildmcp --version
axe --version
```

## Installation

> **Note:** Verify exact Claude Code plugin commands against the [official docs](https://code.claude.com/docs/en/plugins) before use — command syntax may vary by version.

```
/plugin marketplace add https://github.com/hoangdh2001/apple-app-ui-test-automation
/plugin install apple-ui-test-automation
```

## Quick Start

Open a Claude Code session in your Xcode project directory, then invoke an agent:

```
# iPhone
Use the iphone-tester agent to run a regression suite on my app

# iPad
Use the ipad-tester agent to test split-view behavior

# macOS
Use the macos-tester agent to run the full test suite

# Auto-detect iPhone/iPad
Use the ios-ui-automation-tester agent
```

Each agent runs a 7-phase workflow: environment check → project discovery → load tests → select device → build & launch → run tests → generate report.

## Test Case YAML Schema

Define regression tests in `tests/ios-tests.yaml`:

```yaml
suite: Suite Name
version: 1.0
cases:
  - id: kebab-case-id
    name: "Human readable name"
    feature: "Tab > Section"
    steps:
      - navigate: <tab name>       # tap tab bar by label
      - tap: "<element label>"     # tap UI element by label
      - select: "<option>"         # tap picker or menu item
      - type: "<text>"             # type into focused field
    expect:
      - ui_contains: "<label>"     # assert element exists
      - element_not_exists: "<label>"  # assert element absent
```

A skeleton example is provided in `tests/ios-tests.yaml`.

## Report Output

After each run, a Markdown report is generated at:

```
tests/reports/YYYY-MM-DD-HH-MM-<platform>-<suite>.md
```

Screenshots are stored in `tests/reports/screenshots/`. The `tests/reports/` directory is gitignored.

## License

MIT — © hoangdh2001
