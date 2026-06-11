# Design Changes — HUB-16514

## How It Works

- Backend detects the OS from the incoming request and applies the matching setting — the client never sees the raw value before the override
- A member using the same account on multiple OSes gets per-OS behavior (e.g., screenshots disabled on macOS, still active on Windows)
- Overrides the member-level screenshot setting
- Default state: all existing and new users start with everything enabled
- Use "macOS" not "Mac" in copy

## Decisions

- Setting lives in Screenshot Settings (not Silent app config) — keeps all screenshot controls in one place, avoids fragmentation over time
- Applies to all org members, Silent and Standard app users alike — not gated by tracker type
- No user-level override for now — deferred as a future improvement
- Linux/Windows rows: nice to have, out of scope for this task

## Orlynn's feedback (2026-06-11)

- When an admin toggles an OS off, the confirmation or callout should clarify: "Members with multiple devices may still have screenshots active on other platforms" and "Re-enabling will resume capture automatically"
- High-confidence support ticket risk: admin disables macOS → forgets → goes to Screenshot frequency → sees it's set → assumes screenshots are running → contacts support
- The screenshot thumbnail already handles frequency 0 with a label ("Screenshot frequency 0") — we need an equivalent for OS-disabled, and a combined state when both are true
- Suggested approach: modal on toggle-off, or a stronger inline alert; thumbnail needs at minimum two new states: OS disabled (per platform) and OS disabled + frequency 0

## Open Questions

## Iterations
