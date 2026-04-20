---
name: select-iphone-simulator
description: >
  List available simulators, filter to iPhone only, and recommend one based on test content.
  Use inside iphone-tester agent after load-test-cases and before build-and-launch.
  Presents recommendation with reasoning and waits for user confirmation.
tools:
  - Bash
---

# Skill: select-iphone-simulator

## Input
- `test_cases` — list of test cases (used to determine best device size)

## Step 1: List simulators

```bash
axe list-simulators
```

Filter results: **keep iPhone only** — exclude iPad, Apple Watch, Apple TV, Apple Vision Pro.
Prefer simulators that are already booted (faster startup).

## Step 2: Recommend device

| Test content | Recommended device |
|-------------|--------------------|
| Theme, appearance, color, dark mode | Any mid-size iPhone (iPhone 15) |
| Layout-sensitive (tables, lists, grids) | Test on **both** SE/mini (small) and Pro Max (large) |
| General regression | Latest available iPhone |
| WidgetKit, Live Activities | Any iPhone with iOS 16.2+ |

## Step 3: Present recommendation

State your reasoning clearly, e.g.:

> "For layout testing I recommend running on both iPhone SE (3rd gen, UDID: XXXX) and iPhone 15 Pro Max (UDID: YYYY) — the SE has the smallest screen at 375pt width, the Pro Max the largest at 430pt. Want to use these or pick a different simulator?"

**Wait for the user to confirm or choose before returning the UDID.**

## Output
- `udid` — confirmed simulator UDID (or list of UDIDs if running on multiple devices)
- `device_name` — human-readable name + iOS version, e.g. `"iPhone 15 (iOS 17.5)"`
