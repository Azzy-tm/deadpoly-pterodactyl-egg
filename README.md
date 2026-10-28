# DeadPoly Pterodactyl Egg

![Release](https://img.shields.io/github/v/release/Azzy-tm/deadpoly-pterodactyl-egg?style=flat&label=Release)
![Downloads](https://img.shields.io/github/downloads/Azzy-tm/deadpoly-pterodactyl-egg/total?style=flat&label=Downloads)
![License](https://img.shields.io/github/license/Azzy-tm/deadpoly-pterodactyl-egg?style=flat&label=License)

Hosts the Windows DeadPoly dedicated server through Wine on a Linux Pterodactyl node.

- Egg version: `2.0.1`
- Maintainer: [Azzy-tm](https://github.com/Azzy-tm)
- Steam server App ID: `2208380`
- Executable: `DeadPoly/Binaries/Win64/DeadPolyServer.exe`
- Runtime image: `ghcr.io/parkervcp/yolks:wine_staging`
- Egg export format: `PTDL_v2` (not a panel version requirement)

## Repository and release information

- [Repository](https://github.com/Azzy-tm/deadpoly-pterodactyl-egg)
- [Report an issue](https://github.com/Azzy-tm/deadpoly-pterodactyl-egg/issues)
- [Latest published release](https://github.com/Azzy-tm/deadpoly-pterodactyl-egg/releases/latest)
- [CHANGELOG.md](CHANGELOG.md): complete historical record, including previous releases.
- [RELEASE_NOTES.md](RELEASE_NOTES.md): highlights and upgrade information for the latest release.

The repository copy and latest published release may differ until a new release is published.

## Compatibility and testing

This egg targets a Linux Pterodactyl node with Docker and Wings using the Wine Staging image. It is not a native Windows-host installer. A Windows desktop can download the JSON and administer the server through the panel.

The maintainer confirmed v2.0.1 startup on the existing server. Earlier testing confirmed a player could connect and play using the nested executable. The v2.0.1 JSON, startup wrapper, embedded startup script, and installation script passed static parsing/syntax checks.

Fresh installation, graceful shutdown, save persistence across stop/start, external connectivity, and readiness timing with the revised command still need dedicated regression testing. Successful startup alone does not verify all of these behaviors.

The Wine Staging image uses a floating tag and can change independently of this egg.

## Hardware requirements

| Resource | Minimum | Guidance |
|---|---:|---|
| CPU | 2 cores | 4 cores may be needed with more players or heavier world activity |
| RAM | 6 GB | Allow additional headroom as player count and world activity increase |
| Storage | Space for game files, Wine prefix, logs, saves, and backups | No measured minimum capacity established |

The maintainer observed approximately 5.1 GB RAM usage with no players connected. These requirements are based on that setup, not a player-count benchmark or a guarantee of maximum usage.

CPU and RAM limits are configured in Pterodactyl's server resource settings. The egg does not allocate or enforce them through environment variables.

## Features

- Anonymous SteamCMD installation of Windows server files.
- Direct launch of the nested server executable through Wine.
- Optional Windows-specific SteamCMD update and validation before every start.
- Configuration-template staging without overwriting existing files.
- Filtered forwarding of the Unreal game log to the panel console.
- Session-creation readiness matching.
- Previous game-log preservation as `DeadPoly.log.previous`.

No automatic INI rewriting, background Steam update monitor, API-triggered restart, blacklist automation, mod ZIP installation, or mod loader is included. Configure gameplay and access control manually using the instructions supplied with the installed game build.

## Download

### Option 1: Download the repository egg

On Debian/Ubuntu, install the download and JSON-checking utilities if needed:

```bash
sudo apt update
sudo apt install -y curl jq
```

```bash
curl -fL -o deadpoly-egg.json \
  https://raw.githubusercontent.com/Azzy-tm/deadpoly-pterodactyl-egg/main/deadpoly-egg.json
jq -e 'type == "object" and .meta.version == "PTDL_v2"' deadpoly-egg.json
```

Expected output: `true`. This checks JSON parsing and the format marker, not full panel compatibility or runtime behavior.

### Option 2: Clone the repository

```bash
sudo apt update
sudo apt install -y git
git clone https://github.com/Azzy-tm/deadpoly-pterodactyl-egg.git
cd deadpoly-pterodactyl-egg
```

The import file is `deadpoly-egg.json`. To update an unmodified clone:

```bash
git pull --ff-only
```

Preserve local edits before updating.

### Option 3: Download the latest release asset

```bash
curl -fL -o deadpoly-egg.json \
  https://github.com/Azzy-tm/deadpoly-pterodactyl-egg/releases/latest/download/deadpoly-egg.json
jq -e 'type == "object" and .meta.version == "PTDL_v2"' deadpoly-egg.json
```

This URL requires an asset named `deadpoly-egg.json` on the latest published release. It may return an older version until v2.0.1 is published. Check the downloaded egg's description and `EGG_VERSION` before importing.

## Import and installation

1. In the panel administration area, select or create a nest and import `deadpoly-egg.json`.
2. Create a server using the DeadPoly egg and Wine Staging image.
3. Configure at least 2 CPU cores and 6 GB RAM. Consider 4 cores for more players.
4. Assign the primary game allocation. The tested default is UDP `7777`.
5. Keep `AUTO_UPDATE=0` and `DEADPOLY_AUTO_UPDATE=true`. Set `QUERY_PORT=7778` and initially leave `EXTRA_ARGS` empty.
6. Allow the installation task to complete, then start the server.
7. Inspect the generated configuration and test joining before inviting other players.

`SERVER_PORT` comes from the primary allocation; do not create a duplicate variable. Startup and installation are embedded in the egg; no separate helper scripts need uploading.

## Upgrade from v2.0.0 or v1.x

Back up `DeadPoly/Saved`, configuration, and any additional save locations before changing the egg.

1. Stop the server.
2. Import or update the v2.0.1 egg. If imported as a new egg, assign it to the server.
3. Replace the existing server's saved Startup Command with the new egg default. Updating the egg may not replace an existing startup override.
4. Set `AUTO_UPDATE=0`, `DEADPOLY_AUTO_UPDATE=true`, and `QUERY_PORT=7778`. Leave `EXTRA_ARGS` empty initially.
5. Check that the displayed `EGG_VERSION` is `2.0.1`; refresh the saved server value if it remains older.
6. Confirm the egg's Start Configuration is:

```json
{
  "done": "Successfully created session"
}
```

7. Start the server and check for:

```text
[startup] DeadPoly egg v2.0.1: launching nested server with filtered log forwarding.
```

8. Verify joining, readiness timing, a normal stop/start, and save persistence.

Do not reinstall solely to change the startup command. Existing installations require `.steamcmd/steamcmd.sh` and the nested executable.

### Changes from v1.x

The old automatic INI rewriting, update monitor, API restart integration, blacklist reference fields, mod ZIP staging, and unsupported-setting placeholders are removed. Old helper files are not invoked by this egg and are not automatically deleted. Review previously generated configuration manually rather than assuming it has been corrected.

## Startup variables

| Variable | Default | Purpose |
|---|---|---|
| `AUTO_UPDATE` | `0` | Disables the container image's built-in updater; keep at `0` |
| `DEADPOLY_AUTO_UPDATE` | `true` | Updates and validates Windows server files before each start |
| `QUERY_PORT` | `7778` | Compatibility launch argument; listener not observed in the tested setup |
| `EXTRA_ARGS` | empty | Whitespace-separated extra launch arguments; shell quoting is not evaluated |
| `EGG_VERSION` | `2.0.1` | Egg release label |

## Updates and restarts

With `DEADPOLY_AUTO_UPDATE=true`, the startup command runs Windows-specific SteamCMD update/validation before launching the game. Keep this enabled if a restart should pick up available updates.

The container may print that updates are disabled because `AUTO_UPDATE=0`. That message refers to the image's separate updater, not the egg's `DEADPOLY_AUTO_UPDATE` path.

This egg does not check for new builds while the game is running and does not request automatic restarts through the panel API. A game update alone does not trigger a restart. Stop/start or restart the server when you want the startup update check to run.

SteamCMD output remains visible separately from the filtered game output. A successful SteamCMD validation does not prove that the game subsequently started or that a newer build was available.

## Configuration and access control

Stop the server before editing configuration. Inspect the shipped template under `DeadPoly/Saved/1 RENAME Config`, if present, and generated files under `DeadPoly/Saved/Config`.

The installer copies template files only where files do not already exist. It does not rewrite existing configuration. The earlier setup used `DeadPoly/Saved/Config/WindowsServer/Game.ini`; verify the paths and syntax used by your installed build.

Server name, player limit, private mode, admin IDs, allow lists, and bans are not managed by startup variables in v2.0.1. Follow the installed game's configuration instructions. Do not assume that the old v1.x comma-separated environment-variable syntax applies to game configuration files.

## Logging and readiness

The startup command captures Wine stdout/stderr in `wine-output.log`. It forwards selected lines from `DeadPoly/Saved/Logs/DeadPoly.log` to the panel console:

- Successful session creation.
- Lines containing `Error:`.
- The `LogNet:` listening-port line.
- The `LogWorld: Bringing World` startup line.

Warnings and most verbose plugin, configuration, EOS, HTTP, and heartbeat messages remain in the game log but are not forwarded by this filter. Filtering output does not fix or dismiss the underlying messages. Inspect the full logs when diagnosing a problem.

Before each launch, an existing `DeadPoly.log` is copied to `DeadPoly.log.previous`, replacing the previous backup, and the current log is cleared. `wine-output.log` is also replaced on each launch. These are not long-term log archives; copy them elsewhere before restarting if you need to retain them.

The readiness matcher is:

```text
Successfully created session
```

The command constructs the filter phrase from separate pieces so the echoed startup command does not itself contain the complete matcher. A session-creation message is not proof of external player connectivity. Confirm the panel becomes Running after the real game message, not merely when the command is displayed.

If startup fails, inspect both log files and the full console before reinstalling.

## Ports and network access

| Purpose | Default | Evidence |
|---|---:|---|
| Game traffic | UDP `7777` | Observed listening during testing |
| Query argument | `7778` | No listener observed; protocol/use not confirmed |

Assign and permit the actual game port in the panel and network configuration. Do not assume query port `7778` requires TCP/UDP forwarding solely because a launch variable exists.

If hosting behind a router, forward the assigned UDP game port to the node's LAN address and verify any upstream firewall/NAT restrictions. Test from outside the LAN before claiming external connectivity is working. Do not expose unrelated Wine ports or disable the firewall as a workaround.

## Backups and shutdown

Back up `DeadPoly/Saved`, configuration, and any additional save locations used by your build.

The egg's stop setting is Ctrl+C (`^C`). The revised wrapper includes interrupt/termination forwarding to the Wine process. This is not a guarantee of application-level graceful shutdown or a completed save. Verify that a world change survives a normal stop/start cycle and avoid force-stopping during saves.

## Version history and license

See [CHANGELOG.md](CHANGELOG.md) for the historical record and [RELEASE_NOTES.md](RELEASE_NOTES.md) for the latest release's highlights and migration information.

The repository's [LICENSE](LICENSE) governs the egg. DeadPoly, its assets, Steam-delivered files, and third-party runtime components remain subject to their respective terms.
