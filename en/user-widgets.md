# Widgets and applications

These are two addon types that add extra UI rather than a one-shot scare effect.

## Widgets

**Settings → Widgets.** Widgets are persistent web pages: chat panels, donation goals, polls, first-message alerts, and similar.

Each widget can:

- expose a **URL for an OBS Browser Source**
- optionally show inside the in-app overlay
- choose **where sound may play** (direct link, app window, and/or OBS embed). Silent targets keep video playing at volume zero.

Install from the [catalog](./user-addons.md), open the widget, set display and sound flags, configure the addon’s own options, then Save. Add the widget URL to OBS like any other browser source.

## Internal applications

**Settings → Internal applications.** Application addons open as **separate StreamKit+ windows** — extra tools that stay inside the app.

Open them from:

- the **Applications** button on the [main window](./user-main-window.md) (shown only when at least one is installed), or
- **Open** above the addon in Settings → Internal applications.

If a window cannot open, the app explains why (disabled, crashed, banned, missing files).

## Shared addon editor

Widget, application, overlay, and game addons use the same settings editor: generated fields from the addon, enable/disable, update, uninstall. Save with the bar at the bottom of the detail view. See [Addons and catalog](./user-addons.md) for permissions and updates.
