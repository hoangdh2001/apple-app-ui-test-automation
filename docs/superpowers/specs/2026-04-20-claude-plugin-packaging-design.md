# Design: Claude Plugin Packaging for apple-ui-test-automation

**Date:** 2026-04-20  
**Status:** Approved

## Goal

Package the existing `apple-app-ui-test-automation` project as a publishable Claude Code plugin, ready for distribution via a Claude Code marketplace (GitHub-hosted).

## Approach

**Option B — Full manifest + marketplace-ready.** Add a `.claude-plugin/plugin.json` manifest and a `README.md` without changing the existing `agents/` and `skills/` directory structure.

## Directory Structure

```
apple-app-ui-test-automation/
├── .claude-plugin/
│   └── plugin.json          (new)
├── agents/                  (unchanged — 4 agents)
├── skills/                  (unchanged — 7 skills)
├── tests/
│   ├── ios-tests.yaml       (new — skeleton sample, runtime-populated)
│   └── reports/             (created at runtime by agents, gitignored)
├── .gitignore               (new)
├── CLAUDE.md                (unchanged)
└── README.md                (new)
```

**Note on `tests/`:** The `tests/` directory does not exist yet. This packaging step creates it with a skeleton `ios-tests.yaml` as a sample. The `tests/reports/` subdirectory is created at runtime by the `generate-test-report` skill and must be gitignored.

## Plugin Manifest (`.claude-plugin/plugin.json`)

Schema source: [Claude Code plugin docs](https://code.claude.com/docs/en/plugins) and [marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces).

The `"commands"` key is the standard manifest field for slash-command/skill paths per the Claude Code plugin schema. The `"agents"` key is a separate field for agent definitions.

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

**`category` note:** `"Code Quality & Testing"` is used per the Claude Code marketplace taxonomy. The `&` character is valid JSON and consistent with the documented category name.

**Versioning:** The `version` field must match the git tag on each release (e.g., tag `v0.1.0` → `"version": "0.1.0"`). Bump MINOR for new agents/skills, PATCH for fixes.

## README.md Structure

1. **Badges** — platform (iOS · iPadOS · macOS), license (MIT)
2. **One-liner** — AI-driven UI test automation for Apple apps
3. **Prerequisites** — `xcodebuildmcp`, `axe` (with install links), macOS Accessibility permission
4. **Installation** — Claude Code plugin install steps (note: verify exact command syntax against current Claude Code docs before publishing; commands shown are representative)
5. **Quick Start** — example invocations for iPhone, iPad, macOS agents
6. **Test Case YAML Schema** — full schema with step types and assertion types
7. **Report Output** — generated at runtime to `tests/reports/` (Markdown + screenshots); this directory is gitignored
8. **License** — MIT

## `.gitignore` Contents

```
tests/reports/
```

## Files to Create

| File | Action |
|---|---|
| `.claude-plugin/plugin.json` | Create new |
| `README.md` | Create new |
| `tests/ios-tests.yaml` | Create new — skeleton sample test suite |
| `.gitignore` | Create new |

## Files Unchanged

All existing `agents/` and `skills/` files remain unmodified.
