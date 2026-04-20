---
name: generate-test-report
description: >
  Generate a Markdown test report from completed test case results.
  Used by all platform agents after all test cases have been executed.
  Writes the report to tests/reports/ and returns the file path.
tools:
  - Bash
  - Write
---

# Skill: generate-test-report

## Input
- `results` — list of result objects from `run-test-case`
- `platform` — `"iphone"` | `"ipad"` | `"macos"` *(shown in report header)*
- `device_name` — e.g. `"iPhone 15 (iOS 17.5)"` or `"Mac (macOS 15.0)"`
- `scheme` — Xcode scheme used
- `build_duration_s` — total build time in seconds
- `suite_name` — e.g. `"regression"` or `"focus-session"`
- `project_root` — path to project root (for git command)

## Step 1: Collect metadata

```bash
git -C <project_root> rev-parse --short HEAD
xcodebuildmcp --version
axe --version
```

Compute:
- Total test duration = sum of `duration_s` across all results
- Counts: total, passed (✅), flaky (⚠️), failed (❌)
- Timestamp: current datetime as `YYYY-MM-DD HH:MM`

## Step 2: Write report

Output path: `<project_root>/tests/reports/YYYY-MM-DD-HH-MM-<platform>-<suite-name>.md`

### Report template

```markdown
# iOS Test Report — <platform> — YYYY-MM-DD HH:MM

## Summary
| | |
|---|---|
| Platform | <platform> |
| Device | <device_name> |
| Scheme | <scheme> |
| Build time | <Xs> |
| Test duration | <Xs> |
| **Total** | <N> |
| ✅ Passed | <X> |
| ⚠️ Flaky (passed on retry) | <X> |
| ❌ Failed | <Y> |

---

## Test Cases

### ✅ <id> — <name> (<duration>s)
> <feature>

![Screenshot](screenshots/<id>-pass.png)

---

### ❌ <id> — <name> (<duration>s)
> <feature>

**Error:** <what failed>
**Retry:** <Passed on retry | Failed>

**Logs:**
```
<captured log output>
```

**Screenshot:** ![Error state](screenshots/<id>-error.png)
**Suggested cause:** <root cause analysis>

---

## Environment
- Built: YYYY-MM-DD HH:MM
- Commit: `<short sha>`
- xcodebuildmcp: `<version>`
- axe: `<version>`
```

## Output
Absolute path to the written report file.
