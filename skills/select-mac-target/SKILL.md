---
name: select-mac-target
description: >
  Resolve the correct Xcode destination and scheme for a macOS test run.
  Use inside macos-tester agent instead of select-iphone/ipad-simulator.
  macOS does not use a simulator — the app runs directly on the host machine.
  Confirms the target with the user before returning.
tools:
  - Bash
---

# Skill: select-mac-target

## Input
- `project_path` — path to the `.xcodeproj` file
- `test_cases` — list of test cases (used to check Mac Catalyst vs native macOS)

## Step 1: List available schemes

```bash
xcodebuildmcp simulator list-schemes --project-path <path.xcodeproj>
```

Look for:
- A scheme ending in ` (Mac)` or `macOS` — this is a native macOS target
- A scheme with `Catalyst` in the name — this is an iPad app running on Mac via Mac Catalyst
- A scheme with `Designed for iPad` — this runs iPad binary on Apple Silicon without Catalyst

If multiple Mac-compatible schemes exist, ask the user which one to use.

## Step 2: Confirm destination string

For native macOS or Mac Catalyst:
```
-destination 'platform=macOS,arch=arm64'
```

For Designed for iPad on Apple Silicon:
```
-destination 'platform=macOS,variant=Mac Catalyst'
```

## Step 3: macOS-specific considerations

Inform the user about macOS differences relevant to the test run:

- **No simulator:** The app runs directly on this Mac. System state (login items, permissions, notifications) may affect test results.
- **Accessibility permissions:** macOS requires Accessibility access for axe to interact with UI. Verify in `System Settings > Privacy & Security > Accessibility` that Terminal (or the shell running axe) is listed and enabled.
- **Window focus:** macOS apps can lose focus to other windows. If a tap fails with "element not found", check whether the app window is frontmost.
- **No tab bar navigation:** macOS apps use menu bars and toolbars instead of tab bars. Test cases using `navigate: <tab>` steps must be adapted — the `run-test-case` skill skips the reset-to-Dashboard step on macOS automatically.
- **Menu bar actions:** Steps that require menu bar access (e.g. `File > New`) are not expressible as YAML `tap` steps — flag these to the user and ask for manual verification or skip.

## Step 4: Present recommendation

e.g.:

> "I found a native macOS scheme 'FocusLens (Mac)'. I'll use `-destination 'platform=macOS,arch=arm64'`. Accessibility permission for axe is required — please confirm it's enabled in System Settings before I proceed."

**Wait for user confirmation before returning.**

## Output
- `destination` — xcodebuild destination string
- `scheme` — scheme name to use
- `device_name` — `"Mac (macOS <version>)"` for the report
- `platform` — always `"macos"`
