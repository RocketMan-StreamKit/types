# Donation timer

The timer is a countdown (or count-up) for stream goals. You **control it live** on the [main window](./user-main-window.md). **Settings → Donation timer** defines files, format, behavior, and automatic rules.

Automatic time changes from events need an active [license](./user-license.md). Manual edits on the main window always work.

## Files for OBS

Pick a **timer data folder**. StreamKit+ writes files there (including `timer.txt` and `timerSeconds.txt`) so OBS can use a Text source. If the folder is empty, files go to the app data directory.

Choose a **display format** such as `HH:mm:ss` or `mm:ss`. The same format is used in the main window and in the files.

## Behavior flags

| Option | Effect |
| --- | --- |
| **Count manual changes in added time** | Editing remaining time on the main window also updates the Added counter (on by default). |
| **Show manual changes in latest events** | Manual edits appear in [Latest events](./user-events.md) (on by default). |
| **Reverse timer mode** | Counts up instead of down. |
| **Run only during stream** | Pauses while no platform reports the stream as online (on by default). |
| **Reverse donations** | Matching donations and trigger rules **subtract** time instead of adding it. |

The **limit** on the main window compares to **added time**, not to remaining time. Use it as a cap on how much extra time donations can pile on.

## Automatic adjustments

Add rules that add (or subtract) seconds when an event matches: source addon, event type, reference amount and currency, seconds to apply. Donation amounts can scale proportionally (for example +60 seconds per a set amount).

The trigger model is the same as [overlays](./user-overlay.md): you can use **All** donation sources when you want every donation platform in one currency.

## Cooperative rooms

If you join a room with timer sharing, members get a `timer-{ROOM_ID}` folder inside their timer folder. See [Cooperative sync](./user-coop-sync.md).
