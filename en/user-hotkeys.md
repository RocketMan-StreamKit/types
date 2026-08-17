# Hotkey integrations

**Settings → Hotkey integrations** runs keyboard scenarios when a dashboard event fires: delays, blocking keys, simulated presses, taps, and typing text.

Typical uses: switch an OBS scene from a hotkey the app presses, fire a game input, or type a chat command — optionally only while a chosen program is in the foreground.

Automatic dispatch needs an active [license](./user-license.md). Some anti-cheat systems may flag simulated input — use this only in games and tools that allow it.

## Keyboard agent

StreamKit+ does not inject keys by itself. A small **keyboard agent** must be running.

- Start or stop it from this settings section.
- **Auto-start** launches the agent with the app.
- On Windows, **Run as administrator** is on by default (needed for some elevated games).
- Status should read connected / ready before presets will fire.
- On Linux, foreground-app detection expects an X11 session. On macOS, OS security may block the agent entirely.

The agent stops when StreamKit+ exits.

## Presets

Each preset has:

- Triggers (same model as [overlays](./user-overlay.md))
- Optional **process gating**: always, only when an executable is in the foreground, or only while it is running
- A **scenario**: an ordered list of delay / block key / unblock / key down / key up / tap / type text (US QWERTY)

**Test current settings** runs the scenario after a short delay (default 5 seconds) so you can switch to the target window.

## Folders

Group presets into folders. Optional auto-assignment rules match a substring in the preset name or a target process executable. Export or import a folder (all presets and rules) as a single base64 string — useful for moving a setup between PCs.

## Cooperative sync

If you are in a [room](./user-coop-sync.md), hotkey presets can run on members too. Each member needs the keyboard agent running locally. Sounds and game triggers are not synced.
