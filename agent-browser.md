---
name: agent-browser
description: Use when automating a browser, testing a web UI, filling forms, clicking elements, taking screenshots, or verifying page state. Uses the agent-browser CLI (Rust, CDP-based). Always follow the snapshot→ref workflow — never guess CSS selectors.
---

# Agent Browser Automation

## Core Workflow (always follow this order)

```bash
# 1. Open the page
agent-browser open https://example.com

# 2. Snapshot interactive elements to get stable refs
agent-browser snapshot -i --json

# 3. Interact using the @eN refs from the snapshot
agent-browser click @e2
agent-browser fill @e3 "hello@example.com"

# 4. Re-snapshot after any page change to get fresh refs
agent-browser snapshot -i --json
```

**Refs (@e1, @e2...) are the primary interaction primitive.** Never guess CSS selectors — always snapshot first.

---

## Snapshot Options

```bash
agent-browser snapshot -i          # Interactive elements only (most useful)
agent-browser snapshot -i --json   # JSON output with structured refs
agent-browser snapshot -c          # Compact format
agent-browser snapshot -d 3        # Limit depth
agent-browser snapshot -s "#main"  # Scope to a container
```

---

## Interaction Commands

```bash
# Click / double-click
agent-browser click @e1
agent-browser dblclick @e1

# Fill forms (clears first)
agent-browser fill @e2 "text value"

# Type (appends, no clear)
agent-browser type @e2 "text"

# Keyboard shortcuts
agent-browser press Enter
agent-browser press "Control+a"
agent-browser press Tab

# Checkboxes / dropdowns
agent-browser check @e3
agent-browser select @e4 "Option Value"

# Upload files
agent-browser upload @e5 ./file.pdf
```

---

## Semantic Locators (when refs aren't available)

```bash
agent-browser find role button click --name "Submit"
agent-browser find label "Email" fill "test@test.com"
agent-browser find text "Sign in" click
agent-browser find placeholder "Search..." fill "query"
agent-browser find testid "submit-btn" click
```

---

## Waiting

```bash
agent-browser wait @e1                        # Wait for element visibility
agent-browser wait --text "Welcome"           # Wait for text on page
agent-browser wait --url "**/dashboard"       # Wait for URL pattern
agent-browser wait --load networkidle         # Wait for network quiet
agent-browser wait --fn "window.ready===true" # Wait for JS condition
agent-browser wait 500                        # Fixed millisecond delay
```

---

## Reading Page State

```bash
agent-browser get text @e1          # Text content
agent-browser get value @e2         # Input value
agent-browser get url               # Current URL
agent-browser get title             # Page title
agent-browser is visible @e1        # true/false
agent-browser is enabled @e2        # true/false
```

---

## Screenshots & Visual Verification

```bash
agent-browser screenshot result.png           # Viewport screenshot
agent-browser screenshot --full result.png    # Full-page
agent-browser screenshot --annotate           # Add numbered labels ([1],[2]...)
agent-browser diff screenshot --baseline before.png  # Pixel diff
```

Annotated screenshots number elements matching `@e1`, `@e2` refs — useful for confirming what was clicked.

---

## Batch Execution (for multi-step flows)

```bash
# Inline
agent-browser batch "open https://example.com" "snapshot -i" "click @e2"

# JSON via stdin (best for complex flows)
echo '[
  ["open", "https://example.com"],
  ["snapshot", "-i", "--json"],
  ["fill", "@e2", "user@test.com"],
  ["click", "@e5"],
  ["wait", "--text", "Success"],
  ["screenshot", "after.png"]
]' | agent-browser batch --json

# Stop on first error
agent-browser batch --bail "open ..." "click @e1"
```

---

## Network & Mocking

```bash
agent-browser network route "**/api/auth" --body '{"token":"fake"}'
agent-browser network route "*.png" --abort            # Block images
agent-browser network requests --type fetch            # View requests
agent-browser network har start && <actions> && agent-browser network har stop trace.har
```

---

## Sessions & Auth State

```bash
# Isolated sessions (parallel agents)
agent-browser --session agent1 open site-a.com
agent-browser --session agent2 open site-b.com

# Save / restore authenticated state
agent-browser state save auth.json
agent-browser state load auth.json

# Reuse existing Chrome profile (already logged in)
agent-browser --profile Default open gmail.com
```

---

## JSON Output (programmatic)

Append `--json` to any command for structured output:

```bash
agent-browser snapshot -i --json
agent-browser get text @e1 --json
agent-browser is visible @e2 --json
# → {"success": true, "data": {...}}
```

---

## Tabs

```bash
agent-browser tab new https://docs.example.com --label docs
agent-browser tab docs        # Switch to labelled tab
agent-browser tab             # List all tabs
agent-browser tab close docs
```

---

## Debugging

```bash
agent-browser console               # View JS console output
agent-browser errors                # Uncaught exceptions
agent-browser inspect               # Open Chrome DevTools
agent-browser --headed open <url>   # Show browser window
agent-browser diff snapshot         # Compare before/after DOM
```

---

## Setup

```bash
agent-browser install               # Download Chrome for Testing
agent-browser doctor                # Diagnose environment
agent-browser doctor --fix          # Auto-repair
```

---

## Key Rules

1. **Always snapshot before interacting** — refs go stale after navigation or re-renders
2. **Re-snapshot after every click that causes a page change**
3. **Use `wait` before reading state** — don't assume instant DOM updates
4. **Prefer `find role` / `find label` over CSS selectors** when refs aren't available
5. **Use `batch --json` for multi-step flows** — avoids repeated process startup cost
6. **`--json` everywhere** when parsing output programmatically
