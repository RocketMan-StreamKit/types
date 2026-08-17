# Addons and catalog

StreamKit+ does not hard-code Twitch, DonationAlerts, or a particular overlay. Those come from **addons** you install. This page covers the catalog and **platform** addons (streaming and donation). Other types have their own guides: [overlay](./user-overlay.md), [widgets and applications](./user-widgets.md), [games](./user-games.md).

## Addon catalog

**Settings → Addon catalog** lists recommended addons from the online catalog.

- Filter by category and search by name.
- Download counts and star ratings (ratings need an active [license](./user-license.md)).
- **Install** downloads and registers the addon, then asks you to review **permissions**.
- **Install by ID** accepts a catalog id or a GitHub repository URL.
- The full public catalog is also on [rocketman-streams.com](https://rocketman-streams.com/#addons).
- You can open an already installed addon’s folder from the catalog list.

Local folder or zip installs (including drag-and-drop onto the window) are limited to **10 MB**. If the addon needs other addons, the confirmation dialog can install those first.

If the app is older than the addon’s required version, install and update are blocked until you update StreamKit+.

## Streaming and donation addons

**Settings → Integrations** groups:

- **Streaming platforms** — chat, follows, subs, bits, raids, channel points, and similar events
- **Donation platforms** — tips from services such as DonationAlerts or DonatePay

Typical flow:

1. Install from the catalog.
2. Open the addon, fill in tokens or complete sign-in, **Save**.
3. **Enable** the addon.
4. Check the [main window](./user-main-window.md) status bar for a connected state.

Hover a locked enable switch to see why it cannot turn on (required addons missing or disabled, or a catalog ban).

## Day-to-day management

- Enable and disable without uninstalling. Uninstall **keeps** the addon’s saved settings, so a later reinstall restores tokens and preferences.
- Check for catalog updates from settings; update when a newer version is available.
- A one-time notification appears when a **new** addon shows up in the catalog.
- Banned catalog addons are disabled automatically and cannot be re-enabled.
- Running addons can show live CPU and RAM under the name. The process id is shown only in developer mode.
- If an addon crashes in a loop, auto-restart stops until you start it again from settings.

## Permissions you may see

You approve these at install time. Common ones:

| Permission (plain language) | Meaning |
| --- | --- |
| Network | The addon may call websites and APIs |
| Dashboard events | It can post events and register trigger types |
| File access | It may read or write only folders you approve (plus an automatic temp folder) |
| Text to speech | It may speak through your [TTS](./user-tts.md) engine |
| LLM | It may send prompts through your [LLM](./user-llm.md) profiles (keys stay in StreamKit+) |
| Chat send | It can send messages through chat-capable addons |

Revoke file grants later in that addon’s settings if you change your mind.
