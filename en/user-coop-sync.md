# Cooperative sync

**Settings → Cooperative sync** lets a licensed host share **overlay** and **hotkey** effects with other StreamKit+ installs. When a trigger fires on the host, members play the matching effect at roughly the same time (the app compensates for ping).

Sounds and game actions stay on the **host only**. Members do not need a license to receive effects. Creating a room **does** require an active [license](./user-license.md).

## Connect

| Action | Who | Notes |
| --- | --- | --- |
| **Device name** | Everyone | Shown to others in the room |
| **Create room** | Host | Optional custom id (letters and digits, up to 10) or a random id |
| **Join room** | Member | Enter the host’s room id |
| **Leave** | Anyone | Turns off auto-reconnect for this session |

After a successful create or join, the session is restored on the next launch: the host recreates the same room, members keep trying to join until it appears. **Leave** or a host **kick** turns that auto-connect off.

## Host options (while you are the host)

| Setting | Effect |
| --- | --- |
| **Send trigger signals** | Everyone / one random member / everyone except the host (host then skips local overlay and hotkey) |
| **Share donation timer** | Members write `timer-{ROOM_ID}` files inside their [timer](./user-timer.md) folder |
| **Members list** | Ping and kick |

Members see a clickable **Timer {ROOM_ID}** row on the [main window](./user-main-window.md) when timer sharing is on.

## Matching addons

Members should have the same overlay, info-overlay, and game addons as the host (same id, version, and enabled state). Mismatches show as a warning. You can still install, update, enable, or remove addons while connected — including installing a missing catalog addon from this page.

Overlay volume uses each member’s **local** overlay volume, not the host’s.

If a batch includes hotkeys and a member has no [keyboard agent](./user-hotkeys.md) running, that member sees a warning.

## Typical flow

1. Host: license active → Create room → share the id in voice chat or Discord.
2. Members: install the same overlay (and game) addons → Join room.
3. Host streams as usual. Overlay and hotkey triggers fan out to the room.
4. After a restart, wait for the host’s room to come back; members reconnect automatically.
