# Backups

**Settings → Backups** writes zip archives of your StreamKit+ setup so you can move PCs or undo a bad change.

Each backup includes the main config, registered storage, installed addon files, local sound files, addon file-field assets, and timer output files.

## Schedule

| Setting | What it does |
| --- | --- |
| **Interval** | How often an automatic backup runs (default every 24 hours) |
| **Max copies** | How many zips to keep (1–100, default 4). Older copies are rotated out. |
| **Backup folder** | Custom directory, or the default under app data (`storage_backups`) |

Save backup settings before the scheduler picks up a new interval or folder.

## Manual actions

- **Create backup now** — snapshot immediately
- **Open backup folder** — in the system file manager
- **Restore from ZIP** — pick an archive

Restore checks values against the current settings schema, ignores unknown keys, puts addon and asset files back (creating folders as needed), then **restarts StreamKit+** after you confirm. That replaces the current config — make a fresh backup first if you might want to go back.

The list on the page shows date, name, and size of existing copies.
