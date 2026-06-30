# Plugins

## Introduction

In Bonyan, every feature of the application is implemented as a plugin. A plugin is an independent unit with one implementation and one manifest. The architecture does not concern itself with the internal structure of a plugin. What matters is that the plugin declares itself through its manifest and communicates with the rest of the system through the Registry and the Event Bus.

Plugins do not know each other directly. If a plugin needs a capability from another plugin, it gets the service address from the Registry and calls it directly.

## Plugin Types

A plugin's type is declared in its manifest and exists for understanding and documentation purposes. The Core does not make decisions based on type. The startup order of plugins is defined in the project configuration file and the developer is responsible for its correctness.

### Infrastructure

Provides base services that other plugins depend on. Must be listed before other plugins in the configuration file.

Examples: storage, routing, state management, logging, internationalization.

### Product

Implements the core business logic of the application. Without these plugins the application cannot fulfill its primary purpose.

Examples in a finance application: transactions, accounts, categories.

### Feature

Provides optional capabilities the application can work without.

Examples: search, sharing, advanced charts, CSV import.

### Platform

Handles compatibility with the runtime environment. Each platform has a different implementation but exposes the same interface.

Examples: IndexedDB for browser, SQLite for mobile and desktop, FileSystem.

### Transport

Manages message transfer between the system and the outside world.

Examples: HTTP, WebSocket, push notifications.

## Manifest

Every plugin has a manifest. The manifest is the only source of truth for understanding a plugin. The Core gets acquainted with a plugin exclusively through its manifest at startup. Anything not declared in the manifest does not exist from the system's perspective.

Full details of the manifest structure are in `manifest.md`.

## Folder Structure

Every plugin lives in its own folder. The Core expects two specific files in the root of that folder:

**manifest.json:** The plugin's manifest, which the Core reads at startup.

**index.js:** The plugin's entry point, through which the Core communicates with the plugin's implementation.

The rest of the folder's internal structure is up to the developer. Bonyan's architecture does not concern itself with what is inside the plugin.

## Entry Point

At startup, the Core gives the plugin's entry point access to three things: the services declared in the manifest's `dependencies`, the ability to publish and listen to events via the Event Bus, and the plugin's final configuration. The Core provides only these three things and nothing else. The plugin, in turn, returns to the Core the services it declared in `provides` so they can be registered in the Registry.

The exact shape of this exchange — file names, function signatures, and code style — is an implementation detail and belongs in the best-practices document, not in this one.

## User Interface

The user interface does not live inside a plugin. This rule has no exceptions.

The UI layer receives data through plugin services. This separation means the same plugin can be used in a minimal application and in a complex dashboard application. The logic is identical, the appearance is different.

## Plugin Lifecycle

### Startup

The Core starts plugins in the order defined in the configuration file. For each plugin, these steps are followed:

1. The plugin's manifest is read and validated.
2. All required dependencies are verified to exist in the Registry.
3. The plugin's default configuration is merged with the configuration defined in the project configuration file. Project values take priority.
4. The plugin is registered and its services are added to the Registry.
5. The `pluginname:ready` event is published so others know this plugin is available.

### Startup failure

If a plugin encounters an error during any of these steps, the Core deactivates it, logs the reason, and publishes the `core:plugin-failed` event. Startup of the remaining plugins continues.

### Critical error during runtime

If a plugin encounters a critical error during runtime, it reports it to the Core. The Core deactivates the plugin, publishes the `core:plugin-crashed` event, and the system continues with degraded capability.

Ordinary operational errors must be handled inside the plugin itself and must not leak outside.

### Shutdown

When the application shuts down, the Core shuts down plugins in reverse startup order:

1. The `core:shutdown` event is published so the plugin has a chance to prepare.
2. The plugin releases its resources.
3. The plugin saves any necessary state.
4. The plugin's services are removed from the Registry.
5. The `pluginname:stopped` event is published.

## Dependencies

Each plugin declares in its manifest which services it needs. Dependencies come in two kinds:

**Required dependency** means the plugin cannot function at all without this service. If the service is not available, the plugin is deactivated.

**Optional dependency** means the plugin uses the service if it exists. If it does not exist, the plugin works with reduced capability.

A plugin may only use services it has declared in its manifest `dependencies` and may only provide services it has declared in its manifest `provides`.

## Configuration

A plugin may have default configuration. This is declared in the plugin's manifest. The developer can override these values in the project configuration file. Project values always take priority.

Example: a plugin that communicates with an external API defines its default timeout in the manifest. The developer can change this value in the project configuration file.

## Portability

A plugin is self-contained and can be used in another project without modification, provided that the dependencies declared in its manifest are also available in that project.

This is guaranteed because a plugin has no hidden dependencies. Everything a plugin needs is explicitly declared in its manifest.