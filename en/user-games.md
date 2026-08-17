# Game integrations

**Settings → Game integrations** is for addons that change what happens **inside a game** when a viewer event fires: spawn a mob, run a script, send a command, and similar.

This is meant for single-player games or multiplayer titles **without** strict anti-cheat. Third-party game addons may need extra OS permissions. Use them only if you trust the addon and the game allows external input.

Automatic in-game actions need an active [license](./user-license.md).

## How to use

1. Install a game addon from the [catalog](./user-addons.md).
2. Enable it and fill in any game-specific settings (process name, connection, …).
3. Map **input triggers**: dashboard event → action the addon registered.
4. Save, then test with a low-stakes event while the game is running.

The trigger UI is the same idea as [overlays](./user-overlay.md): source addon, event type, amount or custom value.

## Cooperative sync

[Cooperative sync](./user-coop-sync.md) compares game addon versions between host and members and may warn on mismatch. Game triggers themselves are **not** sent to members — they run on the host only. Overlay and hotkey effects are what get synced.
