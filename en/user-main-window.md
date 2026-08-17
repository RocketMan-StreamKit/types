# Main window

The main window is the control surface you keep open during a stream. It does not list every event — that is [Latest events](./user-events.md). It does not show chat — that is the [Chat window](./user-chat.md).

## Donation timer

The left side shows remaining time, added time, and an optional limit. Values follow the format you set in [Donation timer](./user-timer.md) settings.

| Control | What it does |
| --- | --- |
| Toggle next to the title | Turns the timer on or off without wiping stored values |
| Gear on the timer title | Opens Settings → Donation timer |
| Remaining / Added / Limit | Live values (added and limit rows appear when they are not zero) |
| Clock / hourglass / calendar buttons | Open a dialog to **set**, **add**, or **subtract** remaining time, added time, or the limit |

If you joined a [cooperative sync](./user-coop-sync.md) room with timer sharing, a **Timer {ROOM_ID}** row appears. Click it to open the folder where that room’s timer files are written.

The timer can pause while the stream is offline, count up instead of down, and treat donations as subtracting time — those flags live in timer settings, not on this panel.

## Quick actions

| Button | What it does |
| --- | --- |
| **Reboot the Overlay system** | Restarts the in-app overlay window and the effect queue. Use this if effects freeze or stop appearing. |
| **Chat window** | Opens the streamer chat dashboard. |
| **Latest events widget** | Opens the live event feed. |
| **Applications** | Lists installed [application addons](./user-widgets.md) so you can open their windows. Hidden when none are installed. |

## Title bar

Every major window shares the same chrome:

- Minimize, maximize, close
- **Always on top**
- **Opacity** slider (preview while you drag)
- **Notifications** (bell) — unread badge; click an item to open the related settings (for example the catalog scrolled to an addon). Read items disappear after a few minutes once every window’s list is closed. Right-click deletes an item.
- **Settings** (main window) and **tutorial** (question-mark) on main and settings windows
- **Restart application** — use after settings that require a reload, or if something stops responding

While a connected platform reports the stream as live, the title shows elapsed stream time (`StreamKit+ | HH:mm:ss`) on the main, chat, and events windows.

## Status bar

The bottom bar shows:

- **Integration status** — colored dots for installed platform addons. Hover for details. Some badges are clickable (the addon decides what happens).
- **Viewer counts** — per platform and/or total. Click to choose how counts are shown (including offline platforms and deltas).
- **App version** on the main window

If a platform is disconnected, fix it in [Addons](./user-addons.md) rather than on this bar.
