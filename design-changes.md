# Design Changes — HUB-16514

---

# Screenshots per OS

A new sub-settings tab added inside Screenshot Settings. It follows the existing settings-page structure: section label, description, banner, then controls.

## Page structure

- **Label:** Screenshots per OS
- **Description:** Control whether screenshots are captured on each operating system.
- **Banner (blue, top of controls):**
	- OS settings override member settings. 
	- Turning off an operating system stops screenshot capture for everyone on that platform — even members with screenshots enabled.
- **Controls:** One toggle row per OS — macOS, Windows, Linux

## Toggle behavior

Each OS row has a toggle. When toggled:

- The row description updates dynamically:
  - On: "Taking screenshots for all members on [OS]"
  - Off: "Not taking screenshots for members on [OS]"
- The row dims (opacity) when disabled to signal the inactive state
- A confirmation modal appears before the change is committed

## Confirmation modal

Adds friction to a high-impact action — turning an OS on or off affects all members on that platform.

- Uses the Zone confirmation modal pattern (native `<dialog>`, warning/yellow variant for both states)
- The OS name is bold in the message body
- Cancel reverts the toggle; Confirm commits it

### Turning off

- **Title:** Disable screenshots on [OS]?
- **Message:** You're turning off screenshot capture for all members on **[OS]**.
  No screenshots will be taken regardless of individual member settings.
- **Confirm button:** Disable [OS]

### Turning on

- **Title:** Enable screenshots on [OS]?
- **Message:** You're turning on screenshot capture for all members on **[OS]**. Screenshots will resume based on each member's individual settings.
- **Confirm button:** Enable [OS]

## Business rules

- Backend detects the OS from the incoming request and applies the matching setting — the client never sees the raw value before the override
- A member using the same account on multiple OSes gets per-OS behavior (e.g., screenshots disabled on macOS, still active on Windows)
- Overrides the member-level screenshot frequency setting
- Default state: all existing and new users start with all OSes enabled
- Applies to all org members — Silent and Standard app users alike, not gated by tracker type
- No user-level override for now — deferred as a future improvement


---

# Screenshot Frequency

## Added note below description

A one-line note was added just below the section description ("Control the number of screenshots taken in a 10 minute period."), on a new line:

> OS settings override individual member frequency. [View OS settings →]

The link redirects the user to the Screenshots per OS tab. This addresses the support ticket risk flagged in Orlynn's feedback — admins who land on Screenshot Frequency will see a clear pointer to the OS-level override.

---

# Screenshot Thumbnails

Two explorations are in progress. The approach for the new OS-disabled state has not been decided yet.

## Option A — OS-specific label (granular)

The disabled reason names the specific OS:
- "Screenshots disabled / on macOS"
- "Screenshots disabled / on Windows"
- "Screenshots disabled / on Linux"

Gives the viewer precise context about why there's no screenshot.

## Option B — General label

A single generic label regardless of which OS triggered it:
- "Screenshots disabled"

Simpler, consistent with the existing "Screenshot frequency 0" pattern.

## States covered

| State | Description |
|---|---|
| Existing — frequency 0 | Screenshot frequency set to 0, no capture |
| New — OS disabled (general) | OS toggled off, general label |
| New — OS disabled (macOS) | OS toggled off, named OS |
| New — OS disabled (Windows) | OS toggled off, named OS |
| New — OS disabled (Linux) | OS toggled off, named OS |

> Decision pending: which label approach to use. The thumbnail explorer in the prototype shows both options side by side.

---

# Open Questions

---

# Iterations
