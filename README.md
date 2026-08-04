# Screeps: World — Pterodactyl Egg

A community [Pterodactyl](https://pterodactyl.io/) egg for running a private [Screeps: World](https://www.screeps.com/) server.

> [!IMPORTANT]
> The server and included mods are pinned to specific versions because native Screeps dependencies are sensitive to Node.js and compiler changes.

## Features

- Screeps server `4.3.0`
- Node.js 22 on Debian Bookworm
- Modern C++20 build toolchain for native modules such as `isolated-vm`
- Built-in LokiJS storage using `db.json`
- Automatic Pterodactyl port configuration
- Automatically selected, loopback-only administrative CLI port
- Foreground process management without GNU Screen
- Persistent world data across normal restarts and reinstalls
- Included community packages:
  - `screepsmod-auth@2.9.0`
  - `screepsmod-admin-utils@1.36.1`
  - `screepsbot-zeswarm@1.0.3`

The bot is installed as `simplebot`, but it is not spawned automatically.

## Requirements

- A current Pterodactyl Panel and Wings installation supporting `PTDL_v2` eggs
- Docker access to:
  - `node:22-bookworm`
  - `ghcr.io/parkervcp/yolks:nodejs_22`
- Outbound access to npm and GitHub during installation
- A 32-character [Steam Web API key](https://steamcommunity.com/dev/apikey)

Suggested allocation for a small private world:

| Resource | Suggested allocation |
| --- | ---: |
| CPU | 2 vCPU |
| Memory | 2–4 GB |
| Disk | 4 GB |

Installation compiles native Node.js modules and may temporarily use more CPU and memory than normal operation.

## Installation

1. Download [`egg-screeps-world.json`](./egg-screeps-world.json).
2. Open the Pterodactyl administration interface.
3. Open **Nests** and import the egg into an appropriate nest.
4. Create a server using the imported **Screeps: World** egg.
5. Assign one public TCP allocation for the game server.
6. Enter your Steam Web API key under **Startup**.
7. Install and start the server.

The first installation may take several minutes because Screeps contains native Node.js dependencies.

When the server is ready, the console should contain output similar to:

```text
Starting all processes. Ports: 21025 (game), 21026 (cli)
Game server listening on 0.0.0.0:21025
```

Connect through the Screeps: World Steam client using the public hostname or IP address and the Pterodactyl game allocation.

## Configuration variables

| Variable | Default | Description |
| --- | ---: | --- |
| `STEAM_KEY` | — | Required Steam Web API key. |
| `SERVER_PASSWORD` | Empty | Optional password required when connecting. |
| `RUNNERS_COUNT` | `1` | Number of player-code runner processes. Keep this at `1` for a small private server. |
| `RUNNER_THREADS` | `2` | Worker threads used to execute player code. Do not exceed the assigned CPU capacity. |
| `PROCESSORS_COUNT` | `1` | Parallel room processor processes. |
| `RESTART_INTERVAL` | `3600` | Interval in seconds before Screeps recycles workers. |

Pterodactyl writes these settings into `.screepsrc` before every start. Manual changes to managed keys may therefore be overwritten.

## Ports

The Pterodactyl primary allocation is used for the public game server.

The Screeps administrative CLI binds only to `127.0.0.1` inside the container. Its port is selected automatically to prevent a collision when Pterodactyl assigns `21026` as the public game port. No additional Pterodactyl allocation is required for the CLI.

The selected CLI port is printed during startup:

```text
Starting all processes. Ports: 21026 (game), 21027 (cli)
```

## Server files

| Path | Purpose |
| --- | --- |
| `.screepsrc` | Screeps launcher configuration managed by the egg |
| `db.json` | Persistent LokiJS world database |
| `assets/` | Static game assets |
| `mods.json` | Enabled mods and registered bots |
| `logs/` | Individual Screeps component logs |
| `package.json` | Pinned server and mod dependencies |

Back up at least `db.json`. A full Pterodactyl server backup is safer because mods and configuration may also affect the world.

## Administrative CLI

The CLI is intentionally not exposed publicly. A Wings host administrator can connect from inside the container.

First identify the CLI port from the startup log, then run:

```bash
docker exec -it <server-container> sh -lc \
  'cd /home/container && ./node_modules/.bin/screeps cli --port <cli-port>'
```

Do not publish the CLI port to the internet. It permits administrative JavaScript execution inside the Screeps server.

### Spawning the included bot

From the administrative CLI:

```javascript
bots.spawn("simplebot", "W3N1");
```

Replace `W3N1` with the target room. The egg does not spawn bots automatically, so a new private server contains no NPC opponent unless you add one.

## How it works

The egg:

1. Uses `node:22-bookworm` as the installation environment.
2. Installs a modern compiler, Python 3, Git, and the pinned npm dependencies.
3. Creates the initial world database and assets from the Screeps launcher package.
4. Configures `.screepsrc` using Pterodactyl's INI parser.
5. Starts `./node_modules/.bin/screeps start` as the foreground container process.

The Screeps launcher then manages several child processes:

- `storage` — LokiJS database and inter-process communication
- `backend` — game API, sockets, authentication, and administrative CLI
- `engine_main` — central game engine work
- `engine_runner` — player-code execution
- `engine_processor` — room processing

Running the launcher in the foreground is deliberate. Older eggs placed it inside GNU Screen, which fails when Screen cannot resolve Pterodactyl's dynamically assigned container UID.

## Updating

The Screeps server and mods are intentionally pinned in the installation script. Do not blindly replace the versions with `latest`: Screeps native modules have strict Node.js and compiler compatibility requirements.

Before updating:

1. Create a Pterodactyl backup.
2. Review upstream Node.js requirements and changelogs.
3. Update the pinned versions in the egg.
4. Test installation and startup on a disposable server.
5. Verify that the backend, storage, runners, and processors remain running.

A reinstall preserves an existing `.screepsrc`, `db.json`, and `assets/` directory, but it recreates `package.json` and `mods.json`. Back up custom package or mod changes first.

## Troubleshooting

### ``assetdir` option is not defined`

This indicates an incomplete `.screepsrc`, usually left by an older egg revision. Update to the current egg or add:

```ini
logdir = logs
modfile = mods.json
assetdir = assets
db = db.json
```

### `EADDRINUSE ... 0.0.0.0:21026`

An older egg revision configured both the game server and administrative CLI to use port `21026`. In `.screepsrc`, change:

```ini
cli_port = 21026
```

to:

```ini
cli_port =
```

Restart the complete server afterward.

### `getpwuid() can't identify your account!`

This error is normally produced by GNU Screen when the Pterodactyl container UID has no matching passwd entry. This egg does not use Screen. Confirm that the server startup command is:

```bash
./node_modules/.bin/screeps start
```

### `Unexpected token ?` or unsupported `-std=c++20`

The server was installed using an obsolete Node.js or compiler image. Reinstall using the images defined by the current egg.

### Docker container-name conflict

If Wings reports that the server UUID is already in use, inspect the exact named container on the Wings host before removing anything:

```bash
docker ps -a --filter "name=^/<server-uuid>$"
```

Only remove the stale container after confirming that it belongs to the affected Pterodactyl server. Do not delete the server volume.

## Security notes

- Pterodactyl environment variables are not a dedicated secrets manager. Protect access to the Panel, Wings host, backups, and `.screepsrc`.
- The administrative CLI is deliberately loopback-only.
- `SERVER_PASSWORD` controls access to the game server but does not replace normal host and Panel security.
- Mods execute inside every relevant Screeps process. Review changes before upgrading them.
- Keep regular backups of `db.json`.

## Limitations

- Uses the built-in LokiJS storage rather than MongoDB and Redis.
- Does not expose the administrative CLI through Pterodactyl.
- Does not automatically migrate custom mods or package changes during reinstall.
- Has not been tested across every CPU architecture, Wings release, or third-party Docker configuration.

## Reporting problems

When opening an issue, include:

- Pterodactyl Panel version
- Wings version
- Host CPU architecture
- Egg revision or commit
- Complete installation log
- Complete startup log from the first failure
- Any manual changes to `.screepsrc`, `mods.json`, the startup command, or Docker image

Remove Steam keys, passwords, public IP addresses, and other secrets before posting logs.

## Upstream projects

- [Screeps standalone server](https://github.com/screeps/screeps)
- [Screepers launcher](https://github.com/screepers/screeps-launcher)
- [Pterodactyl](https://github.com/pterodactyl/panel)
- [Pterodactyl-compatible Node.js images](https://github.com/DedicatedMC/yolks)
- [ScreepsMods](https://github.com/ScreepsMods)

