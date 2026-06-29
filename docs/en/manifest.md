# Manifest

## Introduction

The manifest is the single source of truth for understanding a plugin. Every plugin must have a manifest. The Core, other plugins, and anyone working with the plugin learns what it is, what services it provides, what events it publishes, and what it needs — exclusively through its manifest. Anything not declared in the manifest does not exist from the system's perspective.

## Manifest Structure

Every manifest consists of six sections.

### Identity

Defines the basic information about the plugin:

**name:** The unique name of the plugin in `vendor.name` format. No two plugins in the same system may share the same name.

**version:** The plugin's version. Full details are in `versioning.md`.

**architecture:** The Bonyan architecture version this plugin was written for. The Core uses this value to check compatibility.

**description:** A short explanation of what the plugin does. This is for humans, not for the Core.

### Type

Declares the plugin's type. Five values are possible:

**infrastructure:** Provides base services that other plugins depend on. Examples: storage, routing, state management, logging, internationalization.

**product:** Implements the core business logic of the application. Without these plugins the application cannot fulfill its primary purpose.

**feature:** Provides optional capabilities the application can work without. Examples: search, sharing, charts.

**platform:** Handles compatibility with the runtime environment. Examples: IndexedDB, SQLite, FileSystem.

**transport:** Manages message transfer between the system and the outside world. Examples: HTTP, WebSocket.

### Dependencies

The services the plugin needs are declared here. Dependencies come in two kinds:

**Required dependency:** The plugin cannot function without this service. If the service is not available at startup, the plugin is deactivated.

**Optional dependency:** The plugin uses the service if it exists. If it does not exist, the plugin works with reduced capability.

A plugin may only use services declared here. Any request to an undeclared service is rejected by the Core.

### Provided Services

If the plugin provides a service for others to use, it must be declared here. For each service:

**name:** The service name as it will be registered in the Registry.

**description:** A short explanation of what the service does.

If the plugin provides no services, this section is left empty.

### Published Events

The events the plugin publishes via the Event Bus are declared here. For each event:

**name:** The event name in `pluginname:eventname` format.

**description:** A short explanation of when this event is published.

This declaration lets other plugins know which events they can listen to without reading the plugin's code.

If the plugin publishes no events, this section is left empty.

### Default Configuration

If the plugin is configurable, its default values are defined here. The developer can override these values in the project configuration file. Project values always take priority.

If the plugin has no configuration, this section is left empty.

## Manifest Example

A real example of the manifest for a transactions plugin in a finance application:

```json
{
  "name": "myfinance.transactions",
  "version": "1.2.0",
  "architecture": "1.0",
  "description": "Financial transaction management",
  "type": "product",
  "dependencies": {
    "required": [
      "myfinance.storage.service",
      "myfinance.accounts.service"
    ],
    "optional": [
      "myfinance.notifications.service"
    ]
  },
  "provides": [
    {
      "name": "myfinance.transactions.service",
      "description": "Access to transaction data and logic"
    }
  ],
  "events": [
    {
      "name": "transactions:saved",
      "description": "Published when a new transaction is successfully saved"
    },
    {
      "name": "transactions:deleted",
      "description": "Published when a transaction is deleted"
    }
  ],
  "config": {
    "maxTransactionsPerPage": 50,
    "defaultCurrency": "USD"
  }
}
```

## Manifest Rules

**The manifest is the single source of truth.** The Core will not start a plugin that has an incomplete or invalid manifest.

**Hidden dependencies are not allowed.** If a plugin needs a service that is not declared in the manifest, this is an architectural error. The Core rejects such access.

**The manifest does not change at runtime.** The manifest is read at startup and remains fixed until the plugin shuts down. A plugin cannot add new dependencies or access rights while running.

**Versioning is mandatory.** Every change to a plugin must be accompanied by a version update. Details are in `versioning.md`.