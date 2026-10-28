# DeadPoly Pterodactyl Egg

![Release](https://img.shields.io/github/v/release/Azzy-tm/deadpoly-pterodactyl-egg?style=flat&label=Release)
![Downloads](https://img.shields.io/github/downloads/Azzy-tm/deadpoly-pterodactyl-egg/total?style=flat&label=Downloads)
![License](https://img.shields.io/github/license/Azzy-tm/deadpoly-pterodactyl-egg?style=flat&label=License)
![Game](https://img.shields.io/badge/DeadPoly-Wine-red?style=flat)

A Pterodactyl egg for hosting the **DeadPoly** Windows dedicated server through **Wine** on a Linux node.

- **Egg version:** `2.0.0`
- **Maintainer:** [Azzy-tm](https://github.com/Azzy-tm)
- **Steam App ID:** `2208380`
- **Server executable:** `DeadPoly/Binaries/Win64/DeadPolyServer.exe`
- **Runtime image:** `ghcr.io/parkervcp/yolks:wine_staging`
- **Egg schema:** `PTDL_v2`

## Repository

- **GitHub:** https://github.com/Azzy-tm/deadpoly-pterodactyl-egg
- **Issues:** https://github.com/Azzy-tm/deadpoly-pterodactyl-egg/issues
- **Egg download:** https://raw.githubusercontent.com/Azzy-tm/deadpoly-pterodactyl-egg/main/deadpoly-egg.json

## Compatibility and testing

This egg targets a Linux Pterodactyl node running Docker and the Wine Staging image. It is not a native Windows-host installer.

During testing on the maintainer's Ubuntu node, the egg installed the Windows server files, launched the nested executable through Wine, created an online session, bound UDP 7777, allowed a player to connect and play, and was detected by Pterodactyl as running.

External router access, long-term save persistence, and a complete fresh-install regression pass still require additional testing.

## Hardware requirements

| Resource | Minimum | Recommended |
|---|---:|---:|
| CPU cores | 2 | 4 |
| RAM | 6 GB | 8 GB or more |
| Disk | Enough for Steam server files, the Wine prefix, logs, and saves | SSD strongly recommended |

- **2 CPU cores** are enough for a small server or light usage.
- **4 CPU cores** are recommended for more players or heavier world activity.
- The dedicated server was observed using approximately **5.1 GB RAM with no players connected**, so **6 GB RAM is the practical minimum**.
- Allocate **8 GB or more** for multiple players, frequent saves, additional services on the same host, or headroom during updates.

## Features

- Installs the official DeadPoly Windows dedicated server using anonymous SteamCMD.
- Launches `DeadPoly/Binaries/Win64/DeadPolyServer.exe` directly through Wine.
- Runs the Windows-specific SteamCMD update and validation before launch when enabled.
- Stages DeadPoly's shipped configuration template without overwriting existing files.
- Captures Wine output in `wine-output.log`.
- Forwards selected lines from `DeadPoly/Saved/Logs/DeadPoly.log` to the Pterodactyl console.
- Detects server readiness using the successful session-creation message.

This egg does not automatically rewrite INI files, monitor for updates while running, trigger API restarts, install mods, or manage blacklists. Server name, player limits, private mode, admins, allow lists, and bans must be configured manually.

## Download on Linux / Ubuntu

### Option 1: Download only the egg file

Install `curl` and `jq`:

```bash
sudo apt update
sudo apt install -y curl jq
```

Download the egg:

```bash
curl -fL -o deadpoly-egg.json \
  https://raw.githubusercontent.com/Azzy-tm/deadpoly-pterodactyl-egg/main/deadpoly-egg.json
```

Verify the JSON and egg-format marker:

```bash
jq -e 'type == "object" and .meta.version == "PTDL_v2"' deadpoly-egg.json
```

Expected output:

```text
true
```

This check confirms valid JSON and the egg-format marker. It is not a complete Pterodactyl panel-schema validation or proof of runtime compatibility.

### Option 2: Clone the repository

```bash
sudo apt update
sudo apt install -y git
git clone https://github.com/Azzy-tm/deadpoly-pterodactyl-egg.git
cd deadpoly-pterodactyl-egg
```

The egg file is `deadpoly-egg.json`. To update an unmodified clone later:

```bash
git pull --ff-only
```

Preserve local changes before updating.

### Option 3: Download the latest release

```bash
sudo apt update
sudo apt install -y curl jq
curl -fL -o deadpoly-egg.json \
  https://github.com/Azzy-tm/deadpoly-pterodactyl-egg/releases/latest/download/deadpoly-egg.json
jq -e 'type == "object" and .meta.version == "PTDL_v2"' deadpoly-egg.json
```

Expected output:

```text
true
```

## Installation

1. In the Pterodactyl Admin Area, open **Nests**.
2. Create or select a nest for DeadPoly.
3. Choose **Add Egg**, then **Import Egg**.
4. Upload `deadpoly-egg.json`.
5. Create a server using the **DeadPoly** egg and the **Wine Staging** Docker image.
6. Assign at least **2 CPU cores** and **6 GB RAM**. Use **4 cores** and **8 GB or more** for larger player counts.
7. Assign a primary game allocation. The default game port is UDP `7777`.
8. Set `AUTO_UPDATE=0` and `DEADPOLY_AUTO_UPDATE=true`.
9. Start the server and allow installation to finish.
10. Review the generated configuration, test joining, and configure access control before opening the server to other players.

There are no separate helper scripts to upload. Startup is embedded in the egg.

## Upgrade from v1.x

This is a breaking upgrade. Back up `DeadPoly/Saved`, configuration files, and any additional save locations first.

1. Stop the server.
2. Import or update the v2.0.0 egg.
3. Assign the new egg to the existing server if a new egg was imported.
4. Replace the server's saved startup override with the new egg's default startup command. Updating an egg does not necessarily replace an existing override.
5. Set `AUTO_UPDATE=0`.
6. Set `DEADPOLY_AUTO_UPDATE=true`, `QUERY_PORT=7778`, and leave `EXTRA_ARGS` empty initially.
7. Start the server.
8. Confirm this message appears:

   ```text
   [startup] DeadPoly egg v2.0.0: launching nested server with filtered log forwarding.
   ```

9. Confirm Pterodactyl marks the server as running, a player can join, and saves survive a stop/start cycle.

The v2.0.0 startup command does not invoke the old `deadpoly-config.sh`, `deadpoly-update-monitor.sh`, or `deadpoly-start.sh` files. Existing files are not automatically deleted.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `AUTO_UPDATE` | `0` | Disables the container image's built-in updater. Keep at `0`. |
| `DEADPOLY_AUTO_UPDATE` | `true` | Runs SteamCMD update and validation for the Windows server files before each start. |
| `QUERY_PORT` | `7778` | Compatibility launch argument. No listener was observed on this port during testing. |
| `EXTRA_ARGS` | empty | Optional whitespace-separated extra launch arguments. Shell quoting is not evaluated. |
| `EGG_VERSION` | `2.0.0` | Egg release label. |

The image's built-in updater may print that updates are disabled. That is expected; `DEADPOLY_AUTO_UPDATE=true` controls the Windows-specific update performed by this egg.

## Access control

Stop the server before editing configuration. Inspect the shipped template under `DeadPoly/Saved/1 RENAME Config`, if present, and generated files under `DeadPoly/Saved/Config`.

This egg does not write server name, player limits, private mode, admin lists, allow lists, or bans. Follow the configuration instructions shipped with your installed DeadPoly build.

DeadPoly uses the in-game Trust-menu ID, not a SteamID64 or player name:

1. Launch DeadPoly.
2. Open your inventory.
3. Select **TRUSTS**.
4. Copy the value shown as **My ID**.

Use that ID for admin, allow-list, and blacklist configuration where applicable. Do not assume comma-separated IDs are accepted; follow the format used by your installed server build.

## Startup detection and logging

The startup command launches the nested executable through Wine, captures Wine output in `wine-output.log`, and streams selected lines from `DeadPoly/Saved/Logs/DeadPoly.log` to the Pterodactyl console.

The console filter shows:

- Successful session creation.
- Unreal Engine errors.
- The game network listener line.
- World startup completion.

It suppresses high-volume verbose EOS, HTTP, plugin, configuration, and heartbeat messages. Some harmless EOS overlay and social-UI warnings may still appear because a headless dedicated server under Wine has no desktop overlay or signed-in local user.

The readiness pattern is:

```text
Successfully created session
```

This is an online-session indicator, not a guarantee of map readiness or external connectivity. Recheck it after game updates.

If the panel remains in the starting state, inspect `wine-output.log` and `DeadPoly/Saved/Logs/DeadPoly.log`, then test connectivity before reinstalling.

## Ports and router forwarding

In the tested setup, the game bound UDP `7777`. No listener was observed on port `7778` or TCP `7777`. Docker publishing TCP and UDP does not prove that the game uses both protocols.

| Purpose | Default | Protocol | Evidence |
|---|---:|---|---|
| Game traffic | `7777` | UDP | Bound by the running game process during testing |
| Query argument | `7778` | Unverified | No listener observed; not a confirmed forwarding requirement |

For hosting behind a router, forward external UDP `7777` to UDP `7777` on the Ubuntu node's LAN address. Reserve that address in DHCP. Use the assigned game port instead if changed.

Check Pterodactyl allocations, host/container firewall rules, and any upstream NAT or ISP CGNAT. Docker-published traffic can bypass ordinary UFW incoming rules. Do not disable the firewall, expose the host through DMZ, or forward unexplained Wine ports. Test joining from outside your LAN to validate forwarding.

## Backups and shutdown

Back up `DeadPoly/Saved`, configuration, and any additional save locations used by your build.

The egg sends Ctrl+C on stop. Graceful shutdown and save persistence still require testing. Verify that a save survives a full stop/start cycle before relying on this deployment.

## Versioning

This egg follows [Semantic Versioning](https://semver.org/):

- **Patch** (`2.0.1`): Bug fixes, documentation updates, or small corrections.
- **Minor** (`2.1.0`): New backward-compatible variables or features.
- **Major** (`3.0.0`): Breaking changes, major startup/install changes, or support for a new official mod system.

## Release information and license

- See [CHANGELOG.md](CHANGELOG.md) for the complete release history, including v1.0.0 and v2.0.0.
- See [RELEASE_NOTES.md](RELEASE_NOTES.md) for highlights and upgrade information for the latest release.
- The current release is v2.0.0.

This project is licensed under the [MIT License](LICENSE). The DeadPoly game, its assets, and Steam-distributed dedicated-server files are not covered by this repository's license and remain subject to their own terms.
