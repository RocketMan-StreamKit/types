# Overlay system

Overlays are visual (and often audio) effects that play when a dashboard event matches a trigger: screen flash, blur, a jump-scare, a custom web effect, and similar.

They can play:

- in a **fullscreen transparent window** on a monitor you choose, and/or
- in **OBS** (or another tool) via a Browser Source.

Effects in the queue play **one after another** — they do not overlap.

Automatic playback needs an active [license](./user-license.md). You can still configure overlays and test them without one.

## Global overlay settings

**Settings → Overlay system** (list view, not inside a single addon):

| Control | What it does |
| --- | --- |
| **Overlay monitor** | Which display hosts the in-app overlay. **Identify displays** shows each monitor number for a few seconds. |
| **Show overlay in taskbar** | Turn on if OBS window capture cannot find the overlay without a taskbar entry. |
| **OBS browser source URL** | Copy this URL into an OBS Browser Source (typically 1920×1080 or your canvas size). |

After big overlay changes, use **Reboot the Overlay system** on the [main window](./user-main-window.md).

## Categories

- **Overlay effects** — the scare / visual addons. You can also **create a simple overlay** (audio and/or video file, no extra code).
- **Info overlays** — name and message cards shown with an effect. You can pick a specific info addon or **random**.

## Per-effect settings

Open an overlay addon (or a simple overlay you created):

- Enable, update, uninstall
- **Test playback**
- **Where it may appear**: app window and/or OBS
- **Where sound may play** for each of those targets. If sound is off for a target, video still plays but audio is forced silent.
- Volume, duration, and (for simple overlays) loop / hide at end
- Whether to show an info card with the viewer’s name and message
- **Triggers** — when this effect should run (see below)

Save from the overlay detail bar.

## Triggers

The same idea is used for overlays, [sounds](./user-sounds.md), [timer](./user-timer.md) rules, [hotkeys](./user-hotkeys.md), and [games](./user-games.md):

1. Choose the **source addon** (or **All** donation sources to react to every installed donation platform in the selected currency — you get a warning if a platform cannot use that currency).
2. Choose the **event type** (donation, follow, subscription, custom action such as a channel point reward, …).
3. Fill in amount, tier, currency, or the custom value the addon provides.

Platform addons must be enabled before their event types appear in the lists.

## Typical setup

1. Install overlay addons from the [catalog](./user-addons.md), or create a simple media overlay.
2. Paste the OBS URL into a Browser Source if you want the effect on stream.
3. Pick the monitor for the in-app overlay.
4. Add triggers, set volume and duration, Save.
5. Fire a test event or use **Test playback**, then watch [Latest events](./user-events.md) for overlay chips.
