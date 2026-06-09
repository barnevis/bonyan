# Bootstrap Configuration

## Introduction

The bootstrap configuration is the only authoritative source from which the core learns what parts a project consists of and how messages are routed between buses. Without this file, the core does not know which plugins, modules, platforms, and UIs to load.

Every project has one bootstrap configuration file. This file is the first thing the core reads after it runs.

## Format

The bootstrap configuration is a JSON file named `bootstrap.json`.

## Structure

The bootstrap configuration consists of eight sections:

**architecture** — The architecture version this project is built on.

**core** — The name and version of the core and the plugins the core directly requires.

**platform** — The platform this project runs on, along with its folder path so the core can read its `manifest.json` and `contract.json`.

**plugins** — The list of plugins to load, in load order priority. Each plugin must specify its name, version, and folder path.

**modules** — The list of modules to load. Each module must specify its name, version, and folder path.

**ui** — The UI of this project, along with its folder path.

**channels** — Access rules for buses. The developer determines which parts can access each bus based on project needs and security requirements.

Each bus can be defined with one of two policies:

* **allow** — Only the named parts can access this bus. All other parts are denied.
* **deny** — All parts can access this bus except the named ones. If `deny` is empty, all parts have access.

**Default policy:** If a bus is not defined in the `channels` section, access to it is **closed** — no external part can access it. The developer must explicitly define in `channels` any bus that requires external access.

**config** — Configuration values for all parts that take priority over manifest defaults.

## Example

```json
{
  "architecture": "1.1",

  "core": {
    "name": "my-company.core.main",
    "version": "1.2.0",
    "plugins": [
      {
        "name": "my-company.plugin.logger",
        "version": "1.0.0"
      }
    ]
  },

  "platform": {
    "name": "my-company.platform.browser",
    "version": "1.0.0",
    "path": "./platform"
  },

  "plugins": [
    {
      "name": "my-company.plugin.logger",
      "version": "1.0.0",
      "path": "./plugins/logger"
    },
    {
      "name": "my-company.plugin.indexed-db",
      "version": "2.1.0",
      "path": "./plugins/indexed-db"
    },
    {
      "name": "my-company.plugin.auth",
      "version": "1.0.0",
      "path": "./plugins/auth"
    },
    {
      "name": "my-company.plugin.router",
      "version": "1.3.0",
      "path": "./plugins/router"
    }
  ],

  "modules": [
    {
      "name": "my-company.module.task",
      "version": "1.0.0",
      "path": "./modules/task"
    },
    {
      "name": "my-company.module.chat",
      "version": "1.2.0",
      "path": "./modules/chat"
    },
    {
      "name": "my-company.module.notification",
      "version": "1.0.0",
      "path": "./modules/notification"
    }
  ],

  "ui": {
    "name": "my-company.ui.web",
    "version": "1.0.0",
    "path": "./ui/web"
  },

  "config": {
    "my-company.plugin.indexed-db": {
      "db-name": "my-app-db"
    },
    "my-company.plugin.logger": {
      "level": "info"
    },
    "my-company.ui.web": {
      "theme": "dark",
      "language": "fa"
    }
  },

  "channels": {
    "system.bus": {
      "deny": []
    },
    "my-company.ui.web.bus": {
      "allow": [
        "my-company.module.task",
        "my-company.module.notification"
      ]
    },
    "my-company.module.task.bus": {
      "deny": [
        "my-company.ui.mobile"
      ]
    }
  }
}
```

## Startup Process

After reading the configuration file, the core performs the following steps in order:

1. Check architecture version
2. Load core-required plugins
3. If a logger plugin exists, forward logs from the internal logger to it
4. Load the platform defined in `bootstrap.json`
5. Validate the platform's manifest and contract
6. Start the platform
7. Load plugins in list order
8. Validate each plugin's manifest and contract
9. Check each plugin's dependencies (missing optional dependencies are not an error)
10. Load modules
11. Validate each module's manifest and contract
12. Check each module's dependencies (missing optional dependencies are not an error)
13. Load the UI defined in `bootstrap.json`
14. Validate the UI's manifest and contract
15. Create buses defined in the `channels` section; if a part declared a private bus in its manifest but it is not defined in `channels`, the core creates it with default access
16. Set bus access rules based on the `channels` section
17. Start the UI
18. System is ready

If an error occurs at any step, the core halts startup, logs the error with the internal logger, and announces a clear message.

## Core and Manifest

Unlike other parts, the core does not have a `manifest.json` file. The core's information is read directly from the `core` section in `bootstrap.json`. This exception exists because the core runs before any loading mechanism and cannot wait to read its own manifest.

## Environment Variables

Sensitive values such as API keys, database passwords, and service tokens must not be stored as plain text in `bootstrap.json`. These values must be supplied through environment variables.

To use an environment variable in `bootstrap.json`, use the `${ENV_VAR_NAME}` pattern:

```json
{
  "config": {
    "my-company.plugin.auth": {
      "api-key": "${AUTH_API_KEY}",
      "secret": "${AUTH_SECRET}"
    }
  }
}
```

The core reads these values from the injected environment source at startup and substitutes them. All substituted values are strings — the core does not preserve type. Each part is responsible for converting configuration values to the expected type before use. If an environment variable for a `required: true` config key is not set, the core halts startup and announces a clear error naming the missing variable.

## Rules

* Every project has exactly one `bootstrap.json` file
* This file is only read by the core
* No module or plugin may directly access this file
* The order of plugins in the list is the order they are loaded
* Plugins required by the core must also exist in the plugins list
* If a part is not in the list, the core does not load it
* Every project has exactly one platform and one UI
* Config values in this file take priority over manifest defaults
* Configuration does not change after system startup
* Optional dependencies are checked at startup but their absence is not an error
* The `channels` section determines which private bus messages of each part are forwarded to the system bus
