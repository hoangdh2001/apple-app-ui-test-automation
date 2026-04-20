---
name: load-test-cases
description: >
  Detect test mode (ad-hoc vs regression) and return a list of test cases to run.
  Used by all platform agents (iphone-tester, ipad-tester, macos-tester).
  For regression: reads tests/ios-tests.yaml or infers cases from the codebase.
  For ad-hoc: generates a minimal test plan from the user's description.
tools:
  - Bash
  - Read
  - Glob
---

# Skill: load-test-cases

## Input
- `user_message` — the original request from the user
- `project_root` — absolute path to the directory containing the `.xcodeproj` file

## Step 1: Detect mode

| User says | Mode |
|-----------|------|
| "test [feature X]", "kiểm tra [X]", "thử [X]" | Ad-hoc |
| "run regression", "chạy regression", "test all", "test toàn bộ" | Regression |

## Step 2a: Ad-hoc mode

Generate a minimal test plan from the user's description:
- What screen/tab does this feature live on?
- What interactions are needed to reach it?
- What is the expected result?

Return the generated cases and proceed immediately. No file read needed.

## Step 2b: Regression mode

Check if the YAML file exists:

```bash
ls <project_root>/tests/ios-tests.yaml 2>/dev/null && echo "FOUND" || echo "NOT FOUND"
```

**If FOUND:** Read and parse it. Return all entries under `cases:`.

**If NOT FOUND:** Infer from the codebase:

```bash
ls <project_root>/<AppName>/Features/
```

For each feature directory:
1. Identify the primary user action (what does a user *do* with this feature?)
2. Generate one test case covering its core happy path

Present the inferred cases to the user and **wait for confirmation** before returning.

## Test Case YAML Schema

```yaml
suite: Suite Name
version: 1.0
cases:
  - id: kebab-case-id
    name: "Human readable name"
    feature: "Tab > Section"
    steps:
      - navigate: <tab name>      # tap tab bar item by accessibility label
      - tap: "<element label>"    # tap UI element by accessibility label
      - select: "<option>"        # tap item in picker/context menu
      - type: "<text>"            # type text into focused field
    expect:
      - ui_contains: "<label>"
      - element_not_exists: "<label>"
```

## Step Translation: YAML verbs → axe commands

| YAML step | axe command |
|-----------|-------------|
| `navigate: Settings` | `tap --label Settings --wait-timeout 5` |
| `tap: "Start Focus"` | `tap --label 'Start Focus' --wait-timeout 5` |
| `select: "Dark"` | `tap --label 'Dark' --wait-timeout 3` |
| `type: "hello"` | `type 'hello'` |

If `--label` fails with "multiple matches": run `axe describe-ui --udid <UDID>` and use coordinate-based tap.

## Output
List of test case objects ready for the agent to pass into `run-test-case`.
