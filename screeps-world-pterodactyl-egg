{
  "_comment": "DO NOT EDIT: FILE GENERATED AUTOMATICALLY BY PTERODACTYL PANEL - PTERODACTYL.IO",
  "meta": {
    "version": "PTDL_v2",
    "update_url": null
  },
  "exported_at": "2026-07-28T16:00:00+02:00",
  "name": "Screeps: World",
  "author": "pterodactyl_author@example.com",
  "description": "Screeps: World private server 4.3.0 for Pterodactyl. Uses Node.js 22 and a modern C++20 build toolchain. Includes screepsmod-auth, screepsmod-admin-utils, and screepsbot-zeswarm. The server runs directly in the foreground without GNU Screen, avoiding dynamic-UID getpwuid failures under Wings.",
  "features": null,
  "docker_images": {
    "Node.js 22": "ghcr.io/parkervcp/yolks:nodejs_22"
  },
  "file_denylist": [],
  "startup": "./node_modules/.bin/screeps start",
  "config": {
    "files": "{\n  \".screepsrc\": {\n    \"parser\": \"ini\",\n    \"find\": {\n      \"steam_api_key\": \"{{server.build.env.STEAM_KEY}}\",\n      \"port\": \"{{server.build.default.port}}\",\n      \"host\": \"0.0.0.0\",\n      \"password\": \"{{server.build.env.SERVER_PASSWORD}}\",\n      \"cli_port\": \"\",\n      \"cli_host\": \"127.0.0.1\",\n      \"runners_cnt\": \"{{server.build.env.RUNNERS_COUNT}}\",\n      \"runner_threads\": \"{{server.build.env.RUNNER_THREADS}}\",\n      \"processors_cnt\": \"{{server.build.env.PROCESSORS_COUNT}}\",\n      \"logdir\": \"logs\",\n      \"modfile\": \"mods.json\",\n      \"assetdir\": \"assets\",\n      \"db\": \"db.json\",\n      \"log_console\": \"true\",\n      \"log_rotate_keep\": \"5\",\n      \"storage_disabled\": \"false\",\n      \"restart_interval\": \"{{server.build.env.RESTART_INTERVAL}}\"\n    }\n  }\n}",
    "startup": "{\n  \"done\": \"Game server listening on\"\n}",
    "logs": "{}",
    "stop": "^C"
  },
  "scripts": {
    "installation": {
      "script": "#!/bin/bash\nset -euo pipefail\n\napt-get update\nDEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends build-essential python3 git ca-certificates\n\ncd /mnt/server\nexport HOME=/mnt/server\nexport PYTHON=/usr/bin/python3\n\nprintf '%s\\n' 'Installing pinned Screeps server packages with Node.js 22...'\n\ncat > package.json <<'PACKAGE_JSON'\n{\n  \"name\": \"pterodactyl-screeps-world\",\n  \"version\": \"1.0.0\",\n  \"private\": true,\n  \"description\": \"Screeps: World private server managed by Pterodactyl\",\n  \"dependencies\": {\n    \"screeps\": \"4.3.0\",\n    \"screepsbot-zeswarm\": \"1.0.3\",\n    \"screepsmod-admin-utils\": \"1.36.1\",\n    \"screepsmod-auth\": \"2.9.0\"\n  }\n}\nPACKAGE_JSON\n\nnpm install --omit=dev --no-audit --no-fund\n\nlauncher_dist=\"/mnt/server/node_modules/@screeps/launcher/init_dist\"\n\nif [ ! -f /mnt/server/.screepsrc ]; then\n  cp \"${launcher_dist}/.screepsrc\" /mnt/server/.screepsrc\nfi\n\nif [ ! -f /mnt/server/db.json ]; then\n  cp \"${launcher_dist}/db.json\" /mnt/server/db.json\nfi\n\nif [ ! -d /mnt/server/assets ]; then\n  cp -a \"${launcher_dist}/assets\" /mnt/server/assets\nfi\n\ncat > /mnt/server/mods.json <<'MODS_JSON'\n{\n  \"mods\": [\n    \"node_modules/screepsmod-auth/index.js\",\n    \"node_modules/screepsmod-admin-utils/index.js\"\n  ],\n  \"bots\": {\n    \"simplebot\": \"node_modules/screepsbot-zeswarm/src\"\n  }\n}\nMODS_JSON\n\nchmod 755 /mnt/server/node_modules/.bin/screeps\nmkdir -p /mnt/server/logs\n\nprintf '%s\\n' 'Screeps: World installation completed.'\nprintf '%s\\n' 'The first server start can take longer while the initial world data is loaded.'\n\napt-get clean",
      "container": "node:22-bookworm",
      "entrypoint": "/bin/bash"
    }
  },
  "variables": [
    {
      "name": "Steam Web API Key",
      "description": "Required for Steam authentication on a headless server. Create one at https://steamcommunity.com/dev/apikey. This value is stored by Pterodactyl as an environment variable; it is not a secret-storage system.",
      "env_variable": "STEAM_KEY",
      "default_value": "",
      "user_viewable": false,
      "user_editable": true,
      "rules": "required|string|size:32|regex:/^[A-Fa-f0-9]{32}$/",
      "field_type": "text"
    },
    {
      "name": "Server Password",
      "description": "Optional password requested when a player connects. Leave empty for no server password.",
      "env_variable": "SERVER_PASSWORD",
      "default_value": "",
      "user_viewable": true,
      "user_editable": true,
      "rules": "nullable|string|max:128",
      "field_type": "text"
    },
    {
      "name": "Runner Processes",
      "description": "Number of player-code runner processes. Keep this at 1 for a small or single-player server; multiple runners create simultaneous global environments.",
      "env_variable": "RUNNERS_COUNT",
      "default_value": "1",
      "user_viewable": true,
      "user_editable": true,
      "rules": "required|integer|min:1|max:16",
      "field_type": "text"
    },
    {
      "name": "Runner Threads",
      "description": "Worker threads per runner for executing player code. Do not set this above the CPU capacity assigned to the server.",
      "env_variable": "RUNNER_THREADS",
      "default_value": "2",
      "user_viewable": true,
      "user_editable": true,
      "rules": "required|integer|min:1|max:64",
      "field_type": "text"
    },
    {
      "name": "Processor Processes",
      "description": "Parallel room processor processes. For a small private world, 1 is usually sufficient.",
      "env_variable": "PROCESSORS_COUNT",
      "default_value": "1",
      "user_viewable": true,
      "user_editable": true,
      "rules": "required|integer|min:1|max:64",
      "field_type": "text"
    },
    {
      "name": "Worker Restart Interval",
      "description": "Seconds before Screeps recycles runner workers. The upstream default is 3600 seconds.",
      "env_variable": "RESTART_INTERVAL",
      "default_value": "3600",
      "user_viewable": true,
      "user_editable": true,
      "rules": "required|integer|min:60|max:86400",
      "field_type": "text"
    }
  ]
}
