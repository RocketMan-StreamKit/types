# Scoped file access (`files` API)

Requires manifest permission `FILE_ACCESS`. Addons **never** get direct `require('fs')` (that needs `ROOT`). Instead they use the global `files` object: the user approves each path at runtime, and the main process enforces scope.

## How it works

1. Declare `"FILE_ACCESS"` in `manifest.json` (user sees it at install time).
2. Call `files.requestAccess(absolutePath, 'read' | 'manage')` — a native consent dialog appears (like closing the app).
3. On approval, the grant is stored; the addon can call `readFile`, `writeFile`, `stat`, etc. only inside granted paths.
4. Folder grants include **all subfolders and files** inside.
5. `manage` access **includes read** and metadata.
6. Two consecutive user denials for the same path block further requests for **30 seconds**.
7. Subscribe to `fileAccessGranted` / `fileAccessRevoked` with `events.On`.
8. User can revoke any grant in addon settings with one click.

## Request access

```js
const result = await files.requestAccess('C:\\Games\\GTAV', 'manage');
if (!result.success) {
  console.warn(result.message, result.blockedUntil);
  return;
}

events.On('fileAccessGranted', grant => {
  console.log('Granted', grant.path, grant.type);
});

events.On('fileAccessRevoked', grant => {
  console.log('Revoked', grant.path);
});
```

## Check and list

```js
if (await files.hasAccess('C:\\Games\\GTAV\\mods\\hello.txt', 'read')) {
  const { success, data } = await files.readFile('C:\\Games\\GTAV\\mods\\hello.txt');
}

const grants = await files.listAccess();
await files.revokeAccess(grants[0].id);
```

## Operations

| Method | Required grant | Description |
| --- | --- | --- |
| `requestAccess(path, type)` | `FILE_ACCESS` | User consent dialog |
| `revokeAccess(grantId)` | `FILE_ACCESS` | Drop a grant |
| `hasAccess(path, type?)` | `FILE_ACCESS` | `true` / `false` |
| `listAccess()` | `FILE_ACCESS` | Active grants |
| `readFile(path, encoding?)` | read on path | Read text file |
| `writeFile(path, data, encoding?)` | manage on path | Write text file |
| `appendFile(path, data, encoding?)` | manage on path | Append text |
| `readdir(path)` | read on folder | List entries |
| `stat(path)` | read on path | Size, created/modified/access times |
| `exists(path)` | read on path | Existence inside scope |
| `mkdir(path)` | manage on parent/path | Create directory |
| `unlink(path)` | manage on file | Delete file |
| `rename(from, to)` | folder grant + manage on destination | Rename/move items **inside** a granted folder (not the folder root itself); destination must have manage access |
| `copyFile(from, to)` | folder grant (read) + manage on destination | Copy files from inside a granted folder; same destination rules as `rename` |

`stat` returns: `path`, `isDirectory`, `isFile`, `size`, `createdAt`, `modifiedAt`, `accessedAt`.

### `rename` and `copyFile` rules

- The **source** must lie **strictly inside** a granted folder (not on the granted folder root path).
- You **cannot** rename or copy the granted folder itself.
- The **destination** path must be covered by a grant with **manage** access (can be the same folder grant or another approved folder).
- A grant to a single file (without a parent folder grant) is **not** enough for `rename` / `copyFile`.

## Principle of least privilege

- Request `read` when you only need to inspect files.
- Request `manage` only for install/copy/write operations.
- Request the **smallest** path (a single mod folder, not the whole disk).
- Do not use `ROOT` for production game mods.

## Settings UI

Users see active grants under the addon settings page and can revoke them instantly. The `FILE_ACCESS` permission is highlighted in **yellow** at install time with an explanation of risks.
