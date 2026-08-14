# Alerts (`alerts`)

**No permission required**

Coordinates on-screen alert playback across addons (donations, subscriptions, and similar overlays). A donation platform addon reports when its alert starts and ends; other addons (for example a music player) can pause themselves while any alert is playing.

This API does **not** render UI in StreamKit+. It only shares a global “is any alert playing?” flag between addon workers.

## Concepts

| Term | Meaning |
| --- | --- |
| Producer | Addon that displays an alert — calls `alerts.started()` / `alerts.ended()` |
| Consumer | Addon that reacts — calls `alerts.isActive()` and/or `alerts.onChange(...)` |
| Global active | `true` while **at least one** alert is in progress from **any** addon |

Counts are **reference-counted per addon**: call `ended()` once for each `started()`. Intermediate changes (1→2 or 2→1 active alerts) do **not** notify subscribers — only the edges `0 → 1+` and `→ 0` do.

## `alerts.started()`

Marks that **this** addon began displaying an on-screen alert.

```js
alerts.started();
// … play donation / sub alert …
alerts.ended();
```

## `alerts.ended()`

Marks that **this** addon finished one alert. Safe to call when the count is already zero (no-op).

```js
alerts.ended();
```

## `alerts.isActive()`

Returns whether any addon currently reports an active alert.

```js
const res = await alerts.isActive();
if (res.success && res.active) {
  // pause music, hide HUD, etc.
}
```

Returns `{ success: true, active: boolean }`.

## `alerts.onChange(cb)`

Subscribes to global alert-display state changes. Returns `{ Destroy }` to unsubscribe.

| Payload | When |
| --- | --- |
| `{ active: true }` | First alert started anywhere (was idle → now playing) |
| `{ active: false }` | Last alert finished (all addons idle) |

```js
const sub = alerts.onChange(({ active }) => {
  if (active) {
    pauseMusic();
  } else {
    resumeMusic();
  }
});

// Optional: sync current state on load
const state = await alerts.isActive();
if (state.success && state.active) {
  pauseMusic();
}

// later
sub.Destroy();
```

`onChange` does **not** fire with the current value at subscribe time — call `isActive()` if you need the initial state.

## Lifecycle cleanup

When an addon is **disabled**, **uninstalled**, **reloaded** (`api.restart()` / settings restart), or stopped after a **crash loop**, StreamKit+ clears every active alert attributed to that addon.

If that clearance makes the global flag go from active → inactive, all other running addons receive `{ active: false }` through `onChange`.

Producers do not need to call `ended()` during `addon:prepare-stop` for correctness (the host clears the counts), but calling `ended()` before stop is still fine.

## Full example: donation producer + music consumer

```js
// Donation / alert platform addon
events.On('onDonationAlert', async ({ body }) => {
  alerts.started();
  try {
    await playAlertMedia(body);
  } finally {
    alerts.ended();
  }
});
```

```js
// Music player addon
let pausedForAlert = false;

alerts.onChange(({ active }) => {
  if (active && !pausedForAlert) {
    pausedForAlert = true;
    pausePlayback();
  } else if (!active && pausedForAlert) {
    pausedForAlert = false;
    resumePlayback();
  }
});
```
