# Getting started (for streamers)

StreamKit+ is a desktop control center for live streams. It turns viewer actions — donations, subscriptions, follows, chat, raids, channel points, and more — into overlays, sounds, widgets, in-game reactions, and keyboard automations.

This section is for **streamers**. It describes how to use the app, not how to write addons. Addon developers should start at [Getting started](./getting-started.md).

**Supported platforms:** Linux, Windows, macOS.  
**Interface languages:** English, Russian, Ukrainian.

## First launch

1. Open StreamKit+. The **main window** shows the donation timer, quick actions, and integration status.
2. Open **Settings** from the gear icon in the title bar.
3. In **Main**, pick your interface language and display currency.
4. In **License & updates**, activate a license if you want automatic reactions (overlays, sounds, timer rules, hotkeys, game actions). Viewing chat and events still works without a license.
5. Open **Addon catalog**, install a streaming platform addon (for example Twitch) and a donation addon if you use one.
6. Enable each addon, sign in or paste tokens as requested, then **Save**.
7. Confirm the status bar on the main window shows a connected integration.
8. Configure [overlays](./user-overlay.md), [sounds](./user-sounds.md), and the [timer](./user-timer.md) as needed.
9. Open **Chat window** and **Latest events** from the main window during the stream.

An interactive tutorial (question-mark icon in the title bar) walks through the main window and the current settings section. You can skip it and replay it later from the same icon.

## What the app is made of

| Area | What it is |
| --- | --- |
| [Main window](./user-main-window.md) | Timer, shortcuts, status, notifications |
| [Latest events](./user-events.md) | Live feed of donations, follows, subs, and other activity |
| [Chat window](./user-chat.md) | Combined chat from connected platforms |
| [Settings](./user-settings.md) | Every module below lives in the settings sidebar |
| [Addons and catalog](./user-addons.md) | Install and connect platforms, effects, widgets, games |
| [Overlay](./user-overlay.md) | On-screen / OBS effects when events match |
| [Widgets and applications](./user-widgets.md) | Persistent OBS panels and extra in-app windows |
| [Donation timer](./user-timer.md) | Countdown that writes files for OBS |
| [Sound effects](./user-sounds.md) | Alert sounds on events |
| [Hotkeys](./user-hotkeys.md) | Keyboard macros on events |
| [Cooperative sync](./user-coop-sync.md) | Share overlay and hotkey effects with other PCs |
| [Game integrations](./user-games.md) | Viewer actions that affect a game |
| [Text to speech](./user-tts.md) | Read messages aloud |
| [LLM Access](./user-llm.md) | AI profiles for addons that need a language model |
| [License and updates](./user-license.md) | Entitlement, app updates, release notes |
| [Backups](./user-backup.md) | Zip snapshots of config, addons, and assets |

## Addons in plain language

The core app is a hub. Specific platforms and effects come from **addons** you install yourself:

- **Streaming / donation addons** bring events and chat into StreamKit+.
- **Overlay, widget, application, and game addons** react to those events or add extra UI.

You install them from the catalog, from a folder, by repository ID, or by dropping a folder/zip onto the window. Before install you see the permissions the addon asks for. Enable, disable, and configure each one in the matching settings section.

## License

Without an active license you can still connect platforms, watch chat and events, and configure everything. Automatic **triggers** (play overlay, play sound, add timer seconds, run hotkeys, run game actions) and **creating a cooperative sync room** require a license. Joining someone else’s room does not.

See [License and updates](./user-license.md).

## Windows remember their place

Settings, chat, and latest-events remember position, size, always-on-top, and whether they were open. They reopen on the next launch if you left them open. Settings also restores the last sidebar section.
