# HUB-16514 — Silent app: Design platform-specific screenshot toggle (macOS only)

**Ticket:** https://netsoftllc.atlassian.net/browse/HUB-16514
**Status:** Discovery
**Assignee:** victorio.borges
**Related:** HUB-16513

---

## Problem

Admins need a way to disable screenshots specifically for macOS without affecting Windows/Linux members. The current workaround — disabling screenshots globally — unintentionally cuts off all platforms.

## Scope

- Platform-level screenshot toggle in Settings → Silent app
- Controls for disabling screenshots per OS (macOS, Windows, Linux)
- Handle users with mixed-platform setups (e.g., one member on both macOS and Windows)

---

## Current System (from code)

### Screenshot settings (org-level, global)

All live under **Activity → Screenshots** in the settings menu — not under "Silent app". Three tabs:

| Setting | Key | Values |
|---|---|---|
| Frequency | `client_screenshot_frequency` | 0 (None) … 10x; default 1x |
| Blur | `client_screenshot_blur` | 0 = Off, 1 = On |
| Member deletion | `delete_screens_allowed` | Boolean; default true |

Feature-gated: `:screenshots`, `:screenshots_blurring`, `:screenshots_deletion`.

### Activity-level flags (per-activity record)

Three bit flags on `Activity`:
- `FLAG_SUPPORTS_SCREENSHOT` — client device supports capture
- `FLAG_WANTS_SCREENSHOT` — org requested screenshots
- `FLAG_DISABLED_SCREENSHOT` — screenshots disabled for this record

### Platform setting that already exists

`client_platforms_allowed` — controls **which platforms can track time** (All / Desktop only / Mobile only). Lives under **Timer apps**, not Screenshots. Not screenshot-specific, but the data shape is relevant.

### Client API

Settings exposed to the desktop client via `EXPOSED_ORGANIZATION_SETTINGS` in `app/api/hubstaff/client/v1.rb:58–68`. Any new platform-specific screenshot toggle would need to be added here to reach the client app.

### Enforcement

`user_organization.rb:627–629` — `screenshot_setting_enabled?` checks feature flag + frequency != 0. No platform branching today.

### Priority hierarchy (proposed)

A screenshot is taken only if all three layers say yes:

```
Org frequency > 0 (global not off)
    AND
OS of the current session = ON
    AND
Member setting = ON
```

Any layer set to off blocks the screenshot for that session. The OS toggle applies to the **device session**, not the member — so the same member can have screenshots on Windows but not on macOS if the macOS toggle is off. This aligns with how `FLAG_DISABLED_SCREENSHOT` already works (stamped per activity record, not per member).

Mixed-platform example:
- Member logs in on macOS → macOS toggle = OFF → no screenshots
- Same member logs in on Windows → Windows toggle = ON → screenshots taken

The OS toggle acts as a **ceiling**: it can only restrict, never override a member setting that is already off.

---

### What does NOT exist today

- No macOS/Windows/Linux-specific screenshot toggle
- No per-member screenshot override (only org-wide)
- No per-project screenshot override

### Key files

| File | Purpose |
|---|---|
| `app/models/organization.rb:2405–2620` | All settings definitions |
| `app/view_models/organizations/settings_menus.rb:440–474` | UI menu structure |
| `app/api/hubstaff/client/v1.rb:58–68` | Client API exports |
| `app/models/activity.rb:161–304` | Activity-level screenshot flags |
| `app/models/user_organization.rb:627–629` | `screenshot_setting_enabled?` |
