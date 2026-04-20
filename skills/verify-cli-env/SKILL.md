---
name: verify-cli-env
description: >
  Verify that xcodebuildmcp and axe CLIs are installed and available.
  Use at the start of any Apple platform automation session (iPhone, iPad, macOS).
  Returns CLI versions if found; stops with install instructions if either is missing.
tools:
  - Bash
---

# Skill: verify-cli-env

## Input
None. Run unconditionally at session start.

## Steps

```bash
xcodebuildmcp --version
axe --version
```

## Output

**If both present:** Return version strings and continue.

**If either missing:** Stop immediately and tell the user:
- Install xcodebuildmcp: `brew install xcodebuildmcp` or `npm install -g xcodebuildmcp@latest`
- Install axe: https://github.com/cameroncooke/AXe

Do not proceed with any other step until both CLIs are confirmed available.
