# Claude Plugin Packaging Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Package the existing apple-app-ui-test-automation project as a Claude Code plugin ready for GitHub-based marketplace distribution.

**Architecture:** Add four new files (`.claude-plugin/plugin.json`, `README.md`, `tests/ios-tests.yaml`, `.gitignore`) without touching any existing `agents/` or `skills/` content. No code to compile or test — verification is done by inspecting file contents and structure.

**Tech Stack:** JSON (plugin manifest), YAML (test cases), Markdown (README), plain text (.gitignore)

**Spec:** `docs/superpowers/specs/2026-04-20-claude-plugin-packaging-design.md`

---

## File Map

| File | Status | Responsibility |
|---|---|---|
| `.claude-plugin/plugin.json` | Create | Plugin manifest — name, version, metadata, agent/command paths |
| `README.md` | Create | User-facing docs — install, quick start, YAML schema, report output |
| `tests/ios-tests.yaml` | Create | Skeleton sample test suite for new users |
| `.gitignore` | Create | Exclude `tests/reports/` and `.DS_Store` from git |

---

### Task 1: Create plugin manifest

**Files:**
- Create: `.claude-plugin/plugin.json`

- [ ] **Step 1: Create the `.claude-plugin/` directory and write `plugin.json`**

```json
{
  "name": "apple-ui-test-automation",
  "version": "0.1.0",
  "description": "AI-driven UI test automation for iOS, iPadOS, and macOS apps — runs on simulators and real targets via xcodebuildmcp and AXe CLI.",
  "author": {
    "name": "hoangdh2001",
    "url": "https://github.com/hoangdh2001"
  },
  "repository": "https://github.com/hoangdh2001/apple-app-ui-test-automation",
  "homepage": "https://github.com/hoangdh2001/apple-app-ui-test-automation#readme",
  "license": "MIT",
  "keywords": [
    "ios",
    "macos",
    "xcode",
    "ui-testing",
    "automation",
    "simulator",
    "xctest",
    "apple"
  ],
  "category": "Code Quality & Testing",
  "agents": [
    "./agents/ios-ui-automation-tester.md",
    "./agents/iphone-tester.md",
    "./agents/ipad-tester.md",
    "./agents/macos-tester.md"
  ],
  "commands": [
    "./skills/verify-cli-env/SKILL.md",
    "./skills/load-test-cases/SKILL.md",
    "./skills/select-iphone-simulator/SKILL.md",
    "./skills/select-ipad-simulator/SKILL.md",
    "./skills/select-mac-target/SKILL.md",
    "./skills/run-test-case/SKILL.md",
    "./skills/generate-test-report/SKILL.md"
  ]
}
```

Write to: `.claude-plugin/plugin.json`

- [ ] **Step 2: Verify the manifest is valid JSON and all paths resolve**

Run:
```bash
cd "/Users/hoangdo/Documents/personal/my plugin/apple-app-ui-test-automation"
python3 -m json.tool .claude-plugin/plugin.json > /dev/null && echo "JSON valid" && python3 -c "
import json, os
d = json.load(open('.claude-plugin/plugin.json'))
for p in d['agents'] + d['commands']:
    print(p, 'EXISTS' if os.path.exists(p) else 'MISSING')
"
```

Expected: `JSON valid`, all agents and commands print `EXISTS`.

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "feat: add Claude Code plugin manifest"
```

---

### Task 2: Create `.gitignore`

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Write `.gitignore`**

```
tests/reports/
.DS_Store
**/.DS_Store
```

Write to: `.gitignore`

- [ ] **Step 2: Verify**

Run:
```bash
cd "/Users/hoangdo/Documents/personal/my plugin/apple-app-ui-test-automation"
cat .gitignore
```

Expected: three lines — `tests/reports/`, `.DS_Store`, `**/.DS_Store`.

- [ ] **Step 3: Commit**

```bash
git add .gitignore
git commit -m "chore: add gitignore for reports and DS_Store"
```

---

### Task 3: Create skeleton test suite

**Files:**
- Create: `tests/ios-tests.yaml`

- [ ] **Step 1: Write skeleton `ios-tests.yaml`**

```yaml
suite: Sample iOS Test Suite
version: 1.0
cases:
  - id: launch-and-verify-home
    name: "Launch app and verify home screen"
    feature: "Home"
    steps:
      - navigate: Home
    expect:
      - ui_contains: "Home"

  - id: navigate-to-settings
    name: "Navigate to Settings tab"
    feature: "Settings"
    steps:
      - navigate: Settings
    expect:
      - ui_contains: "Settings"

  - id: tap-and-verify
    name: "Tap a button and verify result"
    feature: "Home > Actions"
    steps:
      - navigate: Home
      - tap: "My Button"
    expect:
      - ui_contains: "Success"
      - element_not_exists: "Error"
```

Write to: `tests/ios-tests.yaml`

- [ ] **Step 2: Verify YAML is valid**

Run:
```bash
cd "/Users/hoangdo/Documents/personal/my plugin/apple-app-ui-test-automation"
python3 -c "import yaml; d=yaml.safe_load(open('tests/ios-tests.yaml')); print('YAML valid, cases:', len(d['cases']))"
```

Expected: `YAML valid, cases: 3`

If `yaml` not found: `pip3 install pyyaml` first, or check structure manually with `cat tests/ios-tests.yaml`.

- [ ] **Step 3: Commit**

```bash
git add tests/ios-tests.yaml
git commit -m "feat: add skeleton sample test suite"
```

---

### Task 4: Create README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write `README.md`**

```markdown
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
```

Write to: `README.md`

- [ ] **Step 2: Verify file exists and is non-empty**

Run:
```bash
wc -l "/Users/hoangdo/Documents/personal/my plugin/apple-app-ui-test-automation/README.md"
```

Expected: 70+ lines.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add marketplace-ready README"
```

---

### Task 5: Final verification

- [ ] **Step 1: Verify complete file structure**

Run:
```bash
cd "/Users/hoangdo/Documents/personal/my plugin/apple-app-ui-test-automation"
ls -la .claude-plugin/
ls -la tests/
ls .gitignore README.md
```

Expected:
- `.claude-plugin/plugin.json` exists
- `tests/ios-tests.yaml` exists
- `.gitignore` exists
- `README.md` exists
- `agents/` and `skills/` are unchanged

- [ ] **Step 2: Confirm agents/ and skills/ are untouched**

Run:
```bash
cd "/Users/hoangdo/Documents/personal/my plugin/apple-app-ui-test-automation"
ls agents/
ls skills/
```

Expected: 4 `.md` files in `agents/`, 7 subdirectories in `skills/` — all unchanged.

- [ ] **Step 3: Confirm no reports directory tracked**

Run:
```bash
cd "/Users/hoangdo/Documents/personal/my plugin/apple-app-ui-test-automation"
cat .gitignore
```

Expected: `tests/reports/` on first line.
