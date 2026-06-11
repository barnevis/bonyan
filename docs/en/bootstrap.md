# Bootstrap Configuration

## Introduction

The bootstrap configuration is the single source of truth from which the Core understands what parts a project consists of and how access to channels is configured. Without this file, the Core does not know which plugins, modules, platforms, and user interfaces must be loaded.

Every project has one bootstrap configuration file. This file is the first thing the Core reads upon execution.

## Bootstrap Configuration Format

The bootstrap configuration is a JSON file named `bootstrap.json`.

## Bootstrap Configuration Structure

The bootstrap configuration consists of eight sections:

**architecture** — The architecture version upon which this project is built.

**core** — The name and version of the Core, and the plugins the Core directly requires.

**platform** — The platform on which this project runs, along with its folder path so the Core can read its `manifest.json` and `contract.json` files.

**plugins** — The list of plugins to be loaded, in order of loading priority. Each plugin must specify its folder path in addition to its name and version.

**modules** — The list of modules to be loaded. Each module must specify its folder path in addition to its name and version.

**ui** — The user interface of this project, along with its folder path.

**channels** — Channel access rules. The developer determines which parts can access each channel based on the project's needs and security.

Each channel can be defined with one of two policies:

* **allow** — Only the named parts can access this channel. Other parts do not have access.
* **deny** — All parts can access this channel except the named parts. If `deny` is empty, all parts have access.

**Default Policy:** If a channel is not defined in the `channels` section, access to it is **closed** — no external part can access it. The developer must explicitly define any channel requiring external access in `channels`.

**config** — Configuration values for all parts that override the manifest's default values.

## Bootstrap Configuration Example

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
    "system.channel": {
      "deny": []
    },
    "my-company.ui.web.channel": {
      "allow": [
        "my-company.module.task",
        "my-company.module.notification"
      ]
    },
    "my-company.module.task.channel": {
      "deny": [
        "my-company.ui.mobile"
      ]
    }
  }
}
```

## Startup Process

After reading the configuration file, the Core performs the following steps in order:

1. Checking the architecture version
2. Loading plugins required by the Core
3. Transferring logs from internal logging to a log plugin (if present)
4. Identifying the platform defined in `bootstrap.json`
5. Validating the platform's manifest and contract
6. Loading the platform
7. Loading plugins in the listed order
8. Validating the manifest and contract of each plugin
9. Checking dependencies of each plugin (absence of an optional dependency is not an error)
10. Loading modules
11. Validating the manifest and contract of each module
12. Checking dependencies of each module (absence of an optional dependency is not an error)
13. Identifying the user interface defined in `bootstrap.json`
14. Validating the manifest and contract of the user interface
15. Creating the channels defined in the `channels` section; if a part requested a private channel in its manifest but it is not defined in `channels`, the Core creates it with default access
16. Setting channel access rules based on the `channels` section
17. Initializing the user interface
18. The system is ready

The Core's behavior in response to startup errors depends on the type of part:

* If the Core, the main platform, or the main user interface cannot be initialized, system startup stops.
* If a plugin or module does not have access to a required dependency, that part is not loaded.
* If the absence of a part prevents dependent parts from functioning, those dependent parts are not loaded either.
* The absence of an optional dependency does not stop startup.
* All startup errors must be logged with a clear message.

The goal of this behavior is to allow the system to run with degraded capabilities whenever possible, while stopping startup clearly when foundational system parts are unavailable.

## Core and Manifest

Unlike other parts, the Core does not have a `manifest.json` file. Core information is read directly from the `core` section in `bootstrap.json`. This exception is due to the fact that the Core executes before any loading mechanism and cannot wait to read its own manifest.

## Environment Variables

Sensitive values such as API keys, database passwords, and service tokens must not be stored as plain text in `bootstrap.json`. These values must be provided via environment variables.

To use an environment variable in `bootstrap.json`, the `${ENV_VAR_NAME}` format is used:

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

The Core reads these values from the execution environment at startup and replaces them. All replaced values are strings — the Core does not preserve data types. Each part is responsible for converting configuration values into its required type. If an environment variable marked as `required: true` is not defined in the execution environment, the Core halts startup and reports a clear error.

## Bootstrap Configuration Rules

* Each project has only one `bootstrap.json` file.
* This file is read exclusively by the Core.
* No module or plugin can access this file directly.
* The order of plugins in the list dictates their loading priority.
* Plugins required by the Core must also be present in the `plugins` list.
* If a part is not in the list, the Core will not load it.
* Each project has exactly one platform and one user interface.
* The `config` values in this file override the manifest's default values.
* Configuration does not change after system startup.
* Optional dependencies are checked at startup, but their absence is not an error.
* The `channels` section only specifies access rules for parts and channels; routing or interpreting messages is not the Core's responsibility.