---
name: run-test-case
description: >
  Execute a single UI test case and return its result.
  Works across iPhone, iPad, and macOS — pass the platform param to adjust behavior.
  Requires the app to already be built and running before calling this skill.
  Returns: pass, flaky, or fail — with screenshot path, logs, and root cause on failure.
tools:
  - Bash
---

# Skill: run-test-case

## Input
- `test_case` — one parsed test case object (id, name, feature, steps, expect)
- `platform` — `"iphone"` | `"ipad"` | `"macos"`
- `udid` — simulator UDID *(omit for macOS)*
- `bundle_id` — app bundle identifier
- `report_dir` — path to write screenshots, e.g. `tests/reports/screenshots`

---

## Step 1: Reset app state

**iPhone / iPad:**

Navigate to a known base state before running:

```bash
axe tap --label Dashboard --udid <UDID> --wait-timeout 3
```

If this fails (modal blocking, crash):
```bash
xcodebuildmcp simulator stop --simulator-id <UDID> --bundle-id <bundle>
xcodebuildmcp simulator launch-app --simulator-id <UDID> --bundle-id <bundle>
```
Then poll until interactive (max 30s, retry every 3s):
```bash
axe describe-ui --udid <UDID>
# Success: root element has "type": "Application"
```

**macOS:** Skip reset. macOS apps do not have a tab bar to navigate to. Proceed directly to Step 2.

---

## Step 2: Execute interactions

Translate all YAML steps into a single axe batch call via stdin.

**iPhone / iPad:**
```bash
axe batch --udid <UDID> --stdin << 'EOF'
tap --label <label> --wait-timeout 5
type '<text>'
EOF
```

**macOS:**
```bash
axe batch --stdin << 'EOF'
tap --label <label> --wait-timeout 5
type '<text>'
EOF
```

Rules:
- Always use `--stdin`, not `--step` or `--file`
- Add `--wait-timeout` to every `tap` and `navigate` step

---

## Step 3: Verify expected state

Run `describe-ui` in a **separate** call (batch is fire-and-forget):

**iPhone / iPad:** `axe describe-ui --udid <UDID>`
**macOS:** `axe describe-ui`

For each `expect` entry:
- `ui_contains: "X"` → assert element with `AXLabel == "X"` exists
- `element_not_exists: "X"` → assert NO element with `AXLabel == "X"` exists

If all pass, capture screenshot and return ✅:

**iPhone / iPad:** `axe screenshot --udid <UDID> --output <report_dir>/<id>-pass.png`
**macOS:** `axe screenshot --output <report_dir>/<id>-pass.png`

---

## Step 4: Handle failure

If any assertion fails:

1. Capture error screenshot:
   - iPhone/iPad: `axe screenshot --udid <UDID> --output <report_dir>/<id>-error.png`
   - macOS: `axe screenshot --output <report_dir>/<id>-error.png`

2. Start log capture:
   ```bash
   xcodebuildmcp logging start-simulator-log-capture \
     --simulator-id <UDID> \       # omit on macOS
     --bundle-id <bundle> \
     --capture-console true
   # Save the returned logSessionId
   ```

3. Retry the **failing step only** with `--wait-timeout 5`, then re-run `describe-ui` and assert again.

4. Stop log capture:
   ```bash
   xcodebuildmcp logging stop-simulator-log-capture --log-session-id <logSessionId>
   ```

5. Determine result:
   - Retry passed → ⚠️ flaky
   - Retry failed → ❌ fail

6. Suggest root cause from logs:
   - "Element not found" → label changed, wrong screen, animation still running
   - "Multiple matches" → use coordinate-based tap fallback
   - Crash in logs → report crash type

---

## Output

```
{
  id: string
  name: string
  feature: string
  result: "pass" | "flaky" | "fail"
  duration_s: number
  screenshot: string
  error?: string
  logs?: string
  suggested_cause?: string
}
```
