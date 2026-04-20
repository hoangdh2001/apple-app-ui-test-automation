# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin providing AI-driven UI test automation for Apple platforms (iOS, iPadOS, macOS). It has no build system — it is a collection of agent and skill markdown files that Claude Code loads and executes.

## Required CLI Dependencies

```bash
# Xcode build orchestration
xcodebuildmcp --version   # must be present

# Apple UI automation (tap, swipe, assert, screenshot)
axe --version             # must be present
```

Install AXe from: https://github.com/cameroncooke/AXe

macOS requires Accessibility permission: System Settings → Privacy & Security → Accessibility → grant Terminal/shell.

## Plugin Architecture

```
agents/          # Entry points — each agent orchestrates a full test run
skills/          # Reusable units called by agents, one responsibility each
tests/           # Test definitions (YAML) and generated reports (Markdown)
```

**Agents** hold state across phases and call skills sequentially. **Skills** are stateless units with defined inputs, outputs, and error handling. Users invoke agents; agents invoke skills.

### Agents

| File | Purpose |
|---|---|
| `ios-ui-automation-tester.md` | Universal orchestrator — detects iPhone vs iPad |
| `iphone-tester.md` | iPhone-specific; selects SE/mini vs Pro Max based on test type |
| `ipad-tester.md` | iPad-specific; handles split-view, popovers, multi-device runs |
| `macos-tester.md` | macOS/Mac Catalyst; no simulator, manages Accessibility permissions |

### Skills

| File | Responsibility |
|---|---|
| `verify-cli-env/SKILL.md` | Pre-flight check for `xcodebuildmcp` and `axe` |
| `load-test-cases/SKILL.md` | Parse `tests/ios-tests.yaml` or generate ad-hoc plan from user description |
| `select-iphone-simulator/SKILL.md` | List simulators, recommend by test type, wait for user confirmation |
| `select-ipad-simulator/SKILL.md` | Same as above but iPad-only; supports multi-device runs |
| `select-mac-target/SKILL.md` | Detect native vs Catalyst scheme, return Xcode destination string |
| `run-test-case/SKILL.md` | Reset app → execute steps via `axe batch` → assert → retry on failure |
| `generate-test-report/SKILL.md` | Write Markdown report to `tests/reports/YYYY-MM-DD-HH-MM-<platform>-<suite>.md` |

## 7-Phase Workflow (all agents)

1. **Env check** — `verify-cli-env`
2. **Project discovery** — find `.xcodeproj` and schemes
3. **Load tests** — `load-test-cases`
4. **Select device** — platform-specific selector skill
5. **Build & launch** — via `xcodebuildmcp`
6. **Run tests** — `run-test-case` for each case sequentially
7. **Report** — `generate-test-report`

## Test Case YAML Schema

Regression tests live in `tests/ios-tests.yaml`:

```yaml
suite: Suite Name
version: 1.0
cases:
  - id: kebab-case-id
    name: "Human readable name"
    feature: "Tab > Section"
    steps:
      - navigate: <tab name>           # tap tab bar by label
      - tap: "<element label>"
      - select: "<option>"
      - type: "<text>"
    expect:
      - ui_contains: "<label>"
      - element_not_exists: "<label>"
```

## Device Selection Heuristics

**iPhone:** theme/appearance → mid-size (iPhone 15); layout-sensitive → both SE (375pt) and Pro Max (430pt); general → latest available.

**iPad:** general → iPad Pro (largest canvas); compact → iPad mini (744pt); split-view → 12.9" or 11"; navigation/popovers → both orientations.

**macOS:** native macOS, Mac Catalyst, or Designed for iPad on Apple Silicon — no simulator.

## Key Conventions

- Test results: `pass`, `flaky`, or `fail` with screenshot paths and root cause notes.
- Screenshots stored at `tests/reports/screenshots/`.
- Agents wait for user confirmation before device selection and before destructive actions.
- Each skill documents its exact inputs and outputs at the top of its SKILL.md — read those before editing.
