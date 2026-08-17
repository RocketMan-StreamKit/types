# Settings

Open Settings from the gear on the [main window](./user-main-window.md) title bar. The sidebar lists every module. The window remembers the last section you had open.

Most sections share a **Save / Reset** bar at the bottom. Save applies changes; Reset discards unsaved edits. Some detail screens (an overlay, a sound, a hotkey preset, an addon) have their own save bar.

A few options require an **application restart** (hardware acceleration, high-performance GPU, backup API / proxy). The UI warns you when that is the case. Use **Restart application** in the title bar after saving.

Each settings section can replay its own tutorial from the guide icon in the title bar.

## Sidebar map

| Section | Guide |
| --- | --- |
| Main | This page |
| License & updates | [License and updates](./user-license.md) |
| Addon catalog / Integrations | [Addons and catalog](./user-addons.md) |
| Overlay system | [Overlay](./user-overlay.md) |
| Widgets / Internal applications | [Widgets and applications](./user-widgets.md) |
| Donation timer | [Donation timer](./user-timer.md) |
| Sound effects | [Sound effects](./user-sounds.md) |
| Hotkey integrations | [Hotkeys](./user-hotkeys.md) |
| Cooperative sync | [Cooperative sync](./user-coop-sync.md) |
| Game integrations | [Game integrations](./user-games.md) |
| Text to speech | [Text to speech](./user-tts.md) |
| LLM Access | [LLM Access](./user-llm.md) |
| Chat window | [Chat window](./user-chat.md) |
| Update information | [License and updates](./user-license.md) |
| Backups | [Backups](./user-backup.md) |
| Interface | This page |

## Main

Global options for the whole app.

| Setting | What it does |
| --- | --- |
| **Language** | Menus, settings, and notifications (English, Russian, Ukrainian) |
| **Currency** | Display currency for donations and amounts. Search by name or code. |
| **Hardware acceleration** | Uses the GPU for the app UI. Turn it off if a game is already loading the GPU. Restart required. |
| **High-performance GPU** | On laptops, prefers the discrete GPU. Shown only when hardware acceleration is on. Restart required. |
| **Discord status** | Rich Presence while Discord is running |
| **Developer mode** | Extra diagnostics in packaged builds (always on in development builds) |
| **Backup API server** | Uses a fallback API host when the main one is unreachable. Restart required. |
| **Open logs folder** | Opens the log directory in the system file manager |
| **Reset window positions** | Restores default window layout |
| **Linux desktop shortcut** | Packaged Linux builds can register StreamKit+ in KDE, GNOME, and other application menus |

## Interface

**Settings → Interface** controls color schemes.

- Built-in schemes include system, light, dark, and named presets (for example Dark Moon, Cyberpunk, Matrix).
- **Custom schemes**: create, apply, export, or import a palette as a base64 string. Each preset has color pickers plus typography and corner radius.
- The preview updates across open windows **before** you save. Reset or leaving without saving clears the preview.

Export and import are useful for sharing a theme between PCs. If the exported file was made in another StreamKit+ version, the app may ask you to confirm before importing.
