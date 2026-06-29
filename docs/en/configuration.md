# Configuration

## Introduction

Every project that uses Bonyan has a central configuration file named `bootstrap.json`. This file is the only place where the application's adapters and plugins are declared. The Core reads this file as its first startup step and acts on it.

This file is read only by the Core. No plugin can access it directly.

## Configuration File Structure

The `bootstrap.json` file consists of six sections:

**architecture:** The Bonyan architecture version this project is built on.

**core:** The name and version of the Core the project uses.

**adapters:** Declares which implementation each adapter type should use.

**plugins:** The list of plugins in startup order. Order matters and the developer is responsible for its correctness.

**config:** All configuration in one place. Each plugin's configuration is identified by the plugin name as the key. Project-wide settings also go here.

**app:** Project information such as name and version.

## Configuration File Example

```json
{
  "architecture": "0.5",
  "core": {
    "name": "my-company.core",
    "version": "1.2.0"
  },
  "adapters": {
    "storage": "IndexedDBAdapter",
    "auth": "LocalAuthAdapter",
    "notification": "BrowserNotificationAdapter"
  },
  "plugins": [
    {
      "name": "my-company.logger",
      "version": "1.0.0",
      "path": "./plugins/logger"
    },
    {
      "name": "my-company.indexed-db",
      "version": "2.1.0",
      "path": "./plugins/indexed-db"
    },
    {
      "name": "my-company.auth",
      "version": "1.0.0",
      "path": "./plugins/auth"
    },
    {
      "name": "my-company.router",
      "version": "1.3.0",
      "path": "./plugins/router"
    },
    {
      "name": "my-company.transactions",
      "version": "1.0.0",
      "path": "./plugins/transactions"
    },
    {
      "name": "my-company.settings",
      "version": "1.0.1",
      "path": "./plugins/settings"
    }
  ],
  "config": {
    "my-company.indexed-db": {
      "db-name": "my-app-db"
    },
    "my-company.logger": {
      "level": "info"
    },
    "my-company.settings": {
      "theme": "dark"
    },
    "defaultLanguage": "en",
    "timezone": "UTC"
  },
  "app": {
    "name": "My Finance App",
    "version": "1.4.1"
  }
}
```

## Environment Variables

Sensitive values such as API keys, database passwords, and service tokens must not be stored as plain text in `bootstrap.json`. These values must be supplied through environment variables.

To use an environment variable in `bootstrap.json`, use the `${ENV_VAR_NAME}` format:

```json
{
  "config": {
    "my-company.payment": {
      "apiKey": "${PAYMENT_API_KEY}",
      "endpoint": "${PAYMENT_ENDPOINT}"
    }
  }
}
```

The Core reads these values from the runtime environment at startup and substitutes them.

**Data type:** All substituted values are strings. The Core does not preserve data types. Each plugin is responsible for converting configuration values to the type it needs.

**Required variable:** If a required environment variable is not defined in the runtime environment, the Core stops startup and reports a clear error:

```json
{
  "config": {
    "my-company.payment": {
      "apiKey": {
        "env": "${PAYMENT_API_KEY}",
        "required": true
      }
    }
  }
}
```

## Configuration Rules

**One configuration file.** Every project has exactly one `bootstrap.json`.

**Only the Core reads it.** This file is read only by the Core. No plugin can access it directly.

**Order matters.** The order of plugins in the `plugins` list is the order they are loaded. The developer is responsible for the correctness of this order.

**If it is not listed, it is not loaded.** If a plugin is not in the list, the Core will not load it.

**Plugin config overrides manifest defaults.** Values defined in `config` replace the default values declared in the plugin's manifest.

**Configuration does not change after startup.** The configuration file is read at startup and remains fixed until the system shuts down.

**Optional dependencies are not errors.** Optional dependencies are checked at startup but their absence does not stop the startup process.

**Sensitive values belong in environment variables.** API keys, passwords, and tokens must not appear as plain text in this file.