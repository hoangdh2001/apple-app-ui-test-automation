---
name: ipad-tester
description: >
  iOS UI test automation agent for iPad simulators. Use when asked to test an iPad app
  feature, run iPad regression tests, or automate UI testing on an iPad simulator.
  Includes iPad-specific checks: split-view, slide-over, popover, multitasking.
  Outputs a Markdown report with screenshots.
model: claude-sonnet-4-6
tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
---

# iPad Tester Agent

## Phase 1 — Environment check

Call skill: **verify-cli-env**

If either CLI is missing, stop. Do not proceed.

---

## Phase 2 — Discover project

```bash
find . -name "*.xcodeproj" -maxdepth 3 | head -1
```

Derive `project_root`. Discover schemes:

```bash
xcodebuildmcp simulator list-schemes --project-path <path.xcodeproj>
```

Pick the main app scheme. If the project has a separate iPad scheme, prefer it. If ambiguous, ask the user.

---

## Phase 3 — Load test cases

Call skill: **load-test-cases**

Inputs: `user_message`, `project_root`

Returns: list of test cases.

> **Note:** Review each test case for iPad-specific variants before running:
> - Steps using `navigate: <tab>` may behave differently on iPad (sidebar vs tab bar depending on iOS version and size class)
> - Steps expecting modal sheets should account for popover presentation on iPad

---

## Phase 4 — Select simulator

Call skill: **select-ipad-simulator**

Input: `test_cases`

Returns: `udid` (or list of UDIDs for multi-device runs), `device_name`. Save both.

---

## Phase 5 — Build and launch

```bash
xcodebuildmcp simulator build-and-run \
  --project-path <path.xcodeproj> \
  --scheme <scheme> \
  --simulator-id <UDID>
```

Record `build_duration_s`. If build fails, stop and show the full error.

Poll until interactive (max 30s, retry every 3s):

```bash
axe describe-ui --udid <UDID>
# Success: root element has "type": "Application"
```

---

## Phase 6 — Run test cases

For each test case, call skill: **run-test-case**

Inputs:
- `test_case` — current case
- `platform` — `"ipad"`
- `udid` — from Phase 4
- `bundle_id` — app bundle identifier
- `report_dir` — `<project_root>/tests/reports/screenshots`

Collect all result objects. Do not stop the suite on individual failures.

If running on multiple devices (e.g. iPad mini + iPad Pro), run the full suite on each device separately and produce one report per device.

---

## Phase 7 — Generate report

Call skill: **generate-test-report**

Inputs:
- `results` — all collected results
- `platform` — `"ipad"`
- `device_name` — from Phase 4
- `scheme` — from Phase 2
- `build_duration_s` — from Phase 5
- `suite_name` — inferred from mode
- `project_root` — from Phase 2

Tell the user the report path when done.
