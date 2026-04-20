---
name: select-ipad-simulator
description: >
  List available simulators, filter to iPad only, and recommend one based on test content.
  Use inside ipad-tester agent after load-test-cases and before build-and-launch.
  Includes iPad-specific checks: split-view, slide-over, multitasking layout.
  Presents recommendation with reasoning and waits for user confirmation.
tools:
  - Bash
---

# Skill: select-ipad-simulator

## Input
- `test_cases` — list of test cases (used to determine best device size and orientation)

## Step 1: List simulators

```bash
axe list-simulators
```

Filter results: **keep iPad only** — exclude iPhone, Apple Watch, Apple TV, Apple Vision Pro.
Prefer simulators that are already booted.

## Step 2: Recommend device

| Test content | Recommended device |
|-------------|--------------------|
| General regression | Latest iPad Pro (largest canvas) |
| Compact layout, sidebars | iPad mini (smallest iPad, 744pt width) |
| Split-view / multitasking | iPad Pro 12.9" or iPad Pro 11" — these support all multitasking modes |
| Navigation, popovers | Any iPad — test both portrait and landscape |

## Step 3: iPad-specific considerations

Before confirming the simulator, inform the user about relevant iPad behaviors the tests may trigger:

- **Split-view:** If any test case involves navigating between sections, the app may enter split-view mode on iPad. Mention if tests should be run in full-screen mode (`Settings > Home Screen & Multitasking > Allow Multiple Apps` off) to isolate behavior.
- **Slide-over:** Elements accessible via drag may not appear in the AX tree if a slide-over panel is open. Check for unexpected AX elements.
- **Keyboard shortcuts:** iPad supports hardware keyboard shortcuts — ensure test steps don't assume software keyboard is always visible.
- **Popover vs modal:** On iPad, sheets often present as popovers anchored to a button, not full-screen modals. `ui_contains` assertions should match the popover's AX label, not a navigation title.

## Step 4: Present recommendation

State your reasoning clearly, e.g.:

> "For split-view testing I recommend iPad Pro 12.9" (UDID: XXXX, iOS 17.5) — it supports all three multitasking modes. Want to use this, or test on iPad mini as well for compact layout coverage?"

**Wait for the user to confirm or choose before returning the UDID.**

## Output
- `udid` — confirmed simulator UDID (or list of UDIDs)
- `device_name` — human-readable name + iOS version, e.g. `"iPad Pro 12.9-inch (iOS 17.5)"`
