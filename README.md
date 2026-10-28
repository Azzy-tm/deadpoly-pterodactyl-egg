# DeadPoly Pterodactyl Egg

A Pterodactyl egg for hosting the **DeadPoly** Windows dedicated server through **Wine**.

- **Egg version:** `1.0.0` (Initial release)
- **Maintainer:** [Azzy-tm](https://github.com/Azzy-tm)
- **Steam App ID:** `2208380`
- **Server executable:** `DeadPolyServer.exe`
- **Runtime:** Wine via `ghcr.io/parkervcp/yolks:wine_staging`
- **Pterodactyl schema:** `PTDL_v2`

## Compatibility

This egg is made for **Pterodactyl 2.x**. It will likely work on Pterodactyl 1.x as well because it uses the `PTDL_v2` egg format, but v1 compatibility has not been formally tested.

If you try it on Pterodactyl 1.x, please report back whether installation, startup, and update monitoring work correctly.

## Repository

- **GitHub:** https://github.com/Azzy-tm/deadpoly-pterodactyl-egg
- **Issues:** https://github.com/Azzy-tm/deadpoly-pterodactyl-egg/issues
- **Egg download:** https://raw.githubusercontent.com/Azzy-tm/deadpoly-pterodactyl-egg/main/deadpoly-egg.json

## Features

- Installs the official DeadPoly dedicated server using SteamCMD.
- Runs the Windows server binary under Wine.
- Automatically stages DeadPoly's shipped configuration template.
- Manages server name, player limit, private mode, admins, and allow-list IDs.
- Includes a manual blacklist field for banned Trust-menu IDs.
- Optional Steam update checking, with a configurable interval that defaults to 60 minutes.
- Optional automatic Pterodactyl restart when a new Steam build is detected.
- Future-proof `mods/` folder.
- Optional mod ZIP download and extraction.
- Clear placeholders for settings that DeadPoly does not currently support.

## Requirements

- Pterodactyl Panel 2.x with Wings.
- A node able to pull `ghcr.io/parkervcp/yolks:wine_staging`.
- At least **4–6 GB RAM** recommended for the server.
- At least **5 GB** of disk space; **10 GB or more** is recommended once saves, logs, backups, and future mods are considered.
- Two port allocations:
  - Primary game port: `7777` UDP by default.
  - Query port: `7778` TCP/UDP by default.

## Installation

1. In the Pterodactyl Admin Area, go to **Nests**.
2. Create or select a nest for DeadPoly.
3. Choose **Add Egg** and then **Import Egg**.
4. Upload `deadpoly-egg.json`.
5. Create a new server using the **DeadPoly** egg and the **Wine Staging** Docker image.
6. Allocate the game port and query port.
7. Start the server to run the installation.
8. After the first installation completes, stop the server and review `DeadPoly/Saved/Config/WindowsServer/Game.ini`.
9. Set your admin and allow-list IDs, then start the server again.

## Startup variables

| Variable | Default | Editable | Description |
|---|---:|---:|---|
| `SERVER_NAME` | `DeadPoly Server` | Yes | Name shown in the server browser. |
| `MAX_PLAYERS` | `16` | Yes | Maximum simultaneous players. |
| `QUERY_PORT` | `7778` | Yes | Secondary query/info port. |
| `PRIVATE` | `False` | Yes | Restricts joining to allowed IDs. |
| `ADMIN_IDS` | empty | Yes | Comma-separated Trust-menu IDs with admin access. |
| `ALLOWED_IDS` | empty | Yes | Comma-separated Trust-menu IDs allowed to join a private server. |
| `BLACKLIST_IDS` | empty | Yes | Comma-separated Trust-menu IDs to blacklist. Managed manually; see Blacklists below. |
| `EXTRA_ARGS` | empty | Yes | Optional extra DeadPolyServer.exe launch arguments. |
| `AUTO_UPDATE` | `true` | Yes | Runs SteamCMD validation/update on every startup. |
| `AUTO_UPDATE_CHECK` | `true` | Yes | Checks Steam for newer server builds while running. |
| `UPDATE_CHECK_INTERVAL` | `60` | Yes | Minutes between update checks. |
| `PANEL_URL` | empty | Yes | Panel base URL used for automatic restarts. |
| `PTERO_API_KEY` | empty | Yes | Client API key with Control permission. |
| `MODS_URL` | empty | Yes | Direct URL to a `.zip` containing mod files. |
| `MODS_AUTO_EXTRACT` | `false` | Yes | Extracts `MODS_URL` into `mods/` during installation. |
| `STEAM_WORKSHOP_MODS` | `Currently unavailable` | No | Placeholder; Workshop server mods are not supported yet. |
| `ZOMBIE_SPAWN_RATE` | `Currently unavailable` | No | Placeholder; not configurable through DeadPoly server INI settings. |
| `LOOT_SPAWN_RATE` | `Currently unavailable` | No | Placeholder; not configurable through DeadPoly server INI settings. |
| `SLEEPING` | `Currently unavailable` | No | Placeholder; not configurable through DeadPoly server INI settings. |
| `DAY_LENGTH` | `Currently unavailable` | No | Placeholder; not configurable through DeadPoly server INI settings. |
| `DIFFICULTY` | `Currently unavailable` | No | Placeholder; not configurable through DeadPoly server INI settings. |
| `EGG_VERSION` | `1.0.0` | No | Internal version of this egg. |

## Access control

### Finding Trust-menu IDs

DeadPoly uses the in-game Trust-menu ID, not a SteamID64 or player name.

1. Launch DeadPoly.
2. Open your inventory.
3. Select **TRUSTS**.
4. Copy the value shown as **My ID**.

Use that ID for `ADMIN_IDS`, `ALLOWED_IDS`, and `BLACKLIST_IDS`.

### Whitelist

To make the server private:

1. Set `PRIVATE=True`.
2. Add permitted Trust-menu IDs to `ALLOWED_IDS`, separated by commas.
3. Restart the server.

Example:

```text
ALLOWED_IDS=FRIEND_TRUST_MENU_ID,ANOTHER_FRIEND_TRUST_MENU_ID
```

The egg writes these as quoted entries under `[Allowed]` in `Game.ini`.

### Admins

Add Trust-menu IDs to `ADMIN_IDS`, separated by commas:

```text
ADMIN_IDS=YOUR_TRUST_MENU_ID,ANOTHER_TRUST_MENU_ID
```

The egg writes these as quoted entries under `[Admins]` in `Game.ini`.

### Blacklists

DeadPoly supports blacklisting players, but the exact public ban-entry format in `Engine.ini` is not officially documented.

For that reason, `BLACKLIST_IDS` is a manual reference field:

- The egg does not automatically write it into `Engine.ini`.
- Use the in-game admin ban feature where possible.
- If manual editing is necessary, inspect existing entries at the bottom of `DeadPoly/Saved/Config/WindowsServer/Engine.ini` and follow their exact format.

## Automatic update restarts

The egg can monitor Steam app `2208380` while the server is running.

1. Ensure `AUTO_UPDATE_CHECK=true` (enabled by default).
2. Set `PANEL_URL` to your panel's base URL, for example `https://panel.example.com`.
3. Create a Pterodactyl **Client API key** with Control permission for the server.
4. Put that key into `PTERO_API_KEY`.
5. Optionally change `UPDATE_CHECK_INTERVAL`; the default is 60 minutes.

When Steam reports a new build ID, the egg requests a server restart through the Pterodactyl Client API.

### Important

The update monitor detects updates and restarts the server; it does not download the update by itself.

To ensure the new files are installed after the restart, use one of these options:

- Set `AUTO_UPDATE=true`, which runs SteamCMD validation/update during every startup.
- Create a Pterodactyl scheduled task that runs the **Update Server** action before or after restarts.

Otherwise, the server may restart repeatedly whenever Steam reports a new build but the installed files remain unchanged.

## Mods

DeadPoly does not currently have an official server-side mod system or Steam Workshop support. This egg includes a future-proof staging folder:

```text
mods/
```

You can manually place future mod files there, or provide a direct ZIP URL:

```text
MODS_URL=https://example.com/deadpoly-mods.zip
MODS_AUTO_EXTRACT=true
```

The egg downloads and extracts the archive into `mods/` during installation. It does not force DeadPoly to load those files, because no official server-side mod-loading path currently exists.

## Configuration

After the first installation, the main configuration file is:

```text
DeadPoly/Saved/Config/WindowsServer/Game.ini
```

Example:

```ini
[Server]
servername=My DeadPoly Server
maxplayers=16
Private=False

[Admins]
Admin="YOUR_TRUST_MENU_ID"

[Allowed]
Allow="FRIEND_TRUST_MENU_ID"
```

Admin and allow-list IDs are the player's in-game Trust-menu ID and must be quoted.

## Ports and firewall

Allocate both ports in Pterodactyl:

| Purpose | Default port | Protocol |
|---|---:|---|
| Game traffic | `7777` | UDP |
| Query/info traffic | `7778` | TCP/UDP |

If the server does not appear in the browser, verify both allocations, check the console for Wine or Unreal Engine errors, and test the query port from outside the server network.

## Backups

Back up this directory regularly:

```text
DeadPoly/Saved
```

It contains world/save data and configuration. Avoid force-stopping the server during an active save; DeadPoly documentation warns that closing without confirming a save can cause a rollback.

## Versioning

This egg follows [Semantic Versioning](https://semver.org/):

- **Patch** (`1.0.1`): Bug fixes, documentation updates, or small corrections.
- **Minor** (`1.1.0`): New backward-compatible variables or features.
- **Major** (`2.0.0`): Breaking changes, major startup/install changes, or support for a new official mod system.

The current egg version is shown in the `EGG_VERSION` startup variable and in the egg description.

See [CHANGELOG.md](CHANGELOG.md) for release history.

## License

This project is licensed under the [MIT License](LICENSE).

The DeadPoly game, its assets, and Steam-distributed dedicated-server files are not covered by this repository's license and remain subject to their own terms.