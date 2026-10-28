# DeadPoly Pterodactyl Egg v2.0.0

## Breaking changes

- Replaces the original feature-heavy egg with a smaller configuration that follows the successful gameplay test.
- Launches the nested server executable directly instead of the root bootstrap executable.
- Uses `DEADPOLY_AUTO_UPDATE` for the custom updater; `AUTO_UPDATE` must be `0` to disable the image updater.
- Removes automatic INI rewriting, background update monitoring, API-triggered restarts, unused blacklist fields, mod ZIP staging, and unsupported-setting placeholders.
- Existing servers must update their saved startup command and variable values; importing the egg alone is insufficient.

## Fixes

- Uses the GitHub noreply author address.
- Forces Windows-platform downloads during installation and startup updates.
- Sets `HOME=/mnt/server` for installation and `HOME=/home/container` at runtime.
- Uses plain SteamCMD download URLs, pipeline error handling, and nested-executable existence checks.
- Avoids overwriting existing files while staging the shipped configuration template.
- Removes the default `-nosteam` argument.

## Logging and startup status

- Removes full Wine exception/DLL tracing and Bash execution tracing.
- Retains application debug strings through `WINEDEBUG=-all,+debugstr`.
- Captures Wine output in `wine-output.log`.
- Forwards selected lines from `DeadPoly/Saved/Logs/DeadPoly.log` to the Pterodactyl console.
- Filters verbose EOS, HTTP, plugin, configuration, and heartbeat output while retaining session readiness, errors, the listening-port line, and world startup.
- Detects `Successfully created session`, matching the readiness message observed during a successful startup and player-connection test.

## Hardware requirements

- Minimum: 2 CPU cores and 6 GB RAM.
- Recommended: 4 CPU cores and 8 GB or more RAM for larger player counts.
- The dedicated server was observed using approximately 5.1 GB RAM with no players connected.

## Documentation

- Rewrites downloads, migration, variables, and firewall instructions to match this egg.
- Documents UDP 7777 as observed; query port 7778 is unverified rather than a required TCP/UDP port.
- Clarifies Linux/Wine hosting versus native Windows hosting and distinguishes egg format from panel version.

## Validation status

The maintainer confirmed player connection and gameplay using the nested executable during troubleshooting. Startup detection was confirmed working with the filtered log-forwarding startup command. Fresh-install regression testing, stop/save persistence, update-on-start, and external connectivity still require validation.

## Upgrade

Back up saves and configuration. Import or update the egg, replace the existing server startup override with the new egg default, set `AUTO_UPDATE=0` and `DEADPOLY_AUTO_UPDATE=true`, and test a complete start/join/stop/start cycle.
